---
layout: post
title: "Integrating Chef and Apache ZooKeeper for coordination in a cluster"
date: 2015-04-03 23:36:21 -0400
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [chef, chef-patterns, infrastructure-mgmt]
---
In a cluster environment services on nodes may have to be coordinated for various reasons. For e.g., when a configuration change is made to a distributed computing component like ``HDFS``, the ``HDFS`` service on all nodes shouldn't stop at the same time to restart so that the configuration takes in effect. Stopping of the service on all the nodes will end up in unavailability which is not desired to put it lightly. 

There are many options to perform orchestration/coordination with varied maturity when you manage a cluster using ``Chef``. Here we look at how ``Chef`` and ``ZooKeeper`` can work together to perform coordination of services on cluster nodes. We will use the need to control and coordinate service restart so that the service in all nodes are not stopped at the same time as the example to explain the solution.
<!--more-->
Lets take the simple case of service restart where only one instance of the service can be restarted at any time. One of the ways to accomplish this is by forcing nodes to acquire a lock to perform service restarts. The following is the solution summary 

- A lock need to be acquired by a node before it can take a restart action on the service instance running on the node.
- When a node tries to acquire a lock and if it fails since some other node is holding the lock or other reasons, the node waits and retries for a certain time.
- If the lock is acquired within the certain prescribed time, the node restarts the service.
- Once the restart is complete the node releases the lock so that other nodes can take a lock on it.
- If the lock is not acquired within the certain time, the node remembers that the restart was not successful and hence will try to restart the service next time chef client runs on the node.

If there are any misconfigurations which triggered the restart, the service will not be successfully restarted due to the misconfiguration on the node which acquired the lock first. Since the restart process failed, lock will not be released resulting in other nodes not being able to able to acquire it. This prevents misconfiguration being propagated to other nodes preventing unavailability. Also until the misconfiguration is corrected the service will not be restarted in all the nodes.

Now the key is to be able to implement lock primitive in a distributed environment and this is where [Apache ZooKeeper](https://zookeeper.apache.org/) comes into play. ``ZooKeeper`` is a high-performance coordination service which among many things provides synchronization for distributed applications. ``ZooKeeper`` is used by many Apache projects like ``HBase`` which is a distributed scalable data store. ``ZooKeeper`` is also a distributed system which means failover is handled automatically preventing unavailability.

For the use case of allowing only one node to restart a service, we can use ZooKeeper “znode” as the lock. For a service “X”, if a node is able to create a znode “X” in ZooKeeper then the service can be restarted by the node. If a node is not able to create znode “X” since the znode is already created by another node in the cluster or other reasons, then the node need to wait until the znode is removed. 

The following is a Chef [`` definition ``](https://github.com/bijugs/chef-bach/blob/master_rolling_restart/cookbooks/bcpc-hadoop/definitions/hadoop_service.rb) which implements the restart coordination logic. It is implemented as a `` definition `` so that it can be used to coordinate the restart of multiple services in a cluster. The inline comments in the code explains the logic and any reference to hadoop can be discarded since this can be used for any service. Currently this is implemented and used for a hadoop cluster.

{% codeblock %}
#
# Definition takes three parameters
# service_name: Name of the service
# dependencies: Resources typically template resources for which the service need to be restarted
# process_identifier: String which identifies the process of the service
#
define :hadoop_service, :service_name => nil, :dependencies => nil, :process_identifier => nil do

  params[:service_name] ||= params[:name]
  #
  # Service resource defined using the parameters passed
  #
  service "#{params[:service_name]}" do
    supports :status => true, :restart => true, :reload => false
    action [:enable, :start]
  end
  #
  # Checking to see whether user requested to not to use restart coordination
  #
  if node["bcpc"]["hadoop"]["skip_restart_coordination"]
    Chef::Log.info "Coordination of #{params[:service_name]} restart will be skipped as per user request."
    begin
      res = resources(service: "#{params[:service_name]}")
      if params[:dependencies]
        params[:dependencies].each do |dep|
          res.subscribes(:restart, "#{dep}", :delayed)
        end
      end
    rescue Chef::Exceptions::ResourceNotFound
      Chef::Log.info("Resource service #{params[:service_name]} not found")
    end
  else
    if !params[:process_identifier]
      Chef::Application.fatal!("hadoop_service for #{params[:service_name]} need to specify a valid value for the parameter :process_identifier")
    end
    #
    # When there is a need to restart a hadoop service, a lock need to be taken so that the restart is sequenced preventing all nodes being down at the sametime
    # If there is a failure in acquiring a lock with in a certian period, the restart is scheduled for the next run on chef-client on the node.
    # To determine whether the prev restart failed is the node attribute node[:bcpc][:hadoop][:service_name][:restart_failed] is set to true
    # This ruby block is to check whether this node attribute is set to true and if it is set then gets the hadoop service restart process in motion.
    #
    ruby_block "handle_prev_#{params[:service_name].gsub('-','_')}_restart_failure" do
      block do
        Chef::Log.info "Need to restart #{params[:service_name]} since it failed during the previous run. Another node's restart process failure is a possible reason"
      end
      action :create
      only_if { node[:bcpc][:hadoop][params[:service_name].gsub('-','_').to_sym][:restart_failed] and 
              !process_restarted_after_failure?(node[:bcpc][:hadoop][params[:service_name].gsub('-','_').to_sym][:restart_failed_time],"#{params[:process_identifier]}")}
    end
    #
    # Since string with all the zookeeper nodes is used multiple times this variable is populated once and reused reducing calls to Chef server
    #
    zk_hosts = (get_node_attributes(MGMT_IP_ATTR_SRCH_KEYS,"zookeeper_server","bcpc-hadoop").map{|zkhost| "#{zkhost['mgmt_ip']}:#{node[:bcpc][:hadoop][:zookeeper][:port]}"}).join(",")
    #
    # znode is used as the locking mechnism to control restart of services. The following code is to build the path
    # to create the znode before initiating the restart of hadoop service 
    #
    lock_znode_path = format_restart_lock_path(node[:bcpc][:hadoop][:restart_lock][:root],"#{params[:service_name]}")
    #
    # All hadoop service restart situations like changes in config files or restart due to previous failures invokes this ruby_block
    # This ruby block tries to acquire a lock and if not able to acquire the lock, sets the restart_failed node attribute to true
    #
    ruby_block "acquire_lock_to_restart_#{params[:service_name].gsub('-','_')}" do
      require 'time'
      block do
        tries = 0
        Chef::Log.info("#{node[:hostname]}: Acquring lock at #{lock_znode_path}")
        while true 
          lock = acquire_restart_lock(lock_znode_path, zk_hosts, node[:fqdn])
          if lock
            break
          else
            tries += 1
            if tries >= node[:bcpc][:hadoop][:restart_lock_acquire][:max_tries]
              failure_time = Time.now().to_s
              Chef::Log.info("Couldn't acquire lock to restart ")
              node.set[:bcpc][:hadoop][params[:service_name].gsub('-','_').to_sym][:restart_failed] = true
              node.set[:bcpc][:hadoop][params[:service_name].gsub('-','_').to_sym][:restart_failed_time] = failure_time
              node.save
              break
            end
            sleep(node[:bcpc][:hadoop][:restart_lock_acquire][:sleep_time])
          end
        end
      end
      action :nothing
      if params[:dependencies]
        params[:dependencies].each do |dep|
          subscribes :create, "#{dep}", :immediate
        end
      end
      subscribes :create, "ruby_block[handle_prev_#{params[:service_name].gsub('-','_')}_restart_failure]", :immediate
    end
    #
    # If lock is acquired by the node, ruby_block executes which is to notify service to restart
    #
    ruby_block "coordinate_#{params[:service_name].gsub('-','_')}_restart" do
      block do
        Chef::Log.info("Data node will be restarted in node #{node[:fqdn]}")
      end
      action :create
      only_if { my_restart_lock?(lock_znode_path, zk_hosts, node[:fqdn]) }
    end

    begin
      res = resources(service: "#{params[:service_name]}")
      res.subscribes(:restart, "ruby_block[coordinate_#{params[:service_name].gsub('-','_')}_restart]", :immediate)
    rescue Chef::Exceptions::ResourceNotFound
      Chef::Log.info("Resource service #{params[:service_name]} not found")
    end
    #
    # Once the service restart is complete, the following block releases the lock 
    #
    ruby_block "release_#{params[:service_name].gsub('-','_')}_restart_lock" do
      block do
        Chef::Log.info("#{node[:hostname]}: Releasing lock at #{lock_znode_path}")
        lock_rel = rel_restart_lock(lock_znode_path, zk_hosts, node[:fqdn])
        if lock_rel
          node.set[:bcpc][:hadoop][params[:service_name].gsub('-','_').to_sym][:restart_failed] = false
          node.save
        end
      end
      action :create
      only_if { my_restart_lock?(lock_znode_path, zk_hosts, node[:fqdn]) }
    end
  end

{% endcodeblock %}

The following code snippet is how it is used in recipes instead of the default Chef `` service `` resource.

{% codeblock %}
...
dep = ["template[/etc/hadoop/conf/hdfs-site.xml]",
       "template[/etc/hadoop/conf/hadoop-env.sh]"]

hadoop_service "hadoop-hdfs-datanode" do
  dependencies dep
  process_identifier "org.apache.hadoop.hdfs.server.datanode.DataNode"
end
{% endcodeblock %}

Given this framework it is easier to implement complex logic to determine restart eligibility like rack awareness, restart greater than one node or a set percentage of nodes, restart based on the state of other services in the cluster etc. Also this can be used to implement rolling upgrades of software on cluster nodes. In short there are many options and use cases which can leverage this solution.

Quick word on handling failure during lock acquisition. When the node tries to acquire the lock and fails, the whole chef-client run process could have been stopped or waited for the lock to become available. Both of the options are not desirable as one could understand the implications. That being the reason for choosing the approach of waiting for sometime for the lock to become available and if not remember that the restart need to happen in the next chef-client run. This has the advantage of chef-client run further steps and be successful even when the particular service is not restarted and also automatically restart the service in the next chef-client run which seems like a balanced approach. 

Note this solution uses the ``zookeeper`` ruby gem and the complete code can be found in the [chef-bach bcpc-hadoop cookbook](https://github.com/bloomberg/chef-bach/tree/master/cookbooks/bcpc-hadoop). 

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/)
