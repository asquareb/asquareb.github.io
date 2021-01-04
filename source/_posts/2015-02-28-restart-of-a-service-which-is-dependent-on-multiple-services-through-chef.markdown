---
layout: post
title: "Restart of a service which is dependent on multiple services through Chef"
date: 2015-02-28 23:07:47 -0400
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [chef, chef-patterns, infrastructure-mgmt]
---
In a node there can be services that are dependent on other services. For e.g. a monitoring service is dependent on services which it monitors and collects data. So when a service being monitored is restarted, the service monitoring may have to be restarted to establish back the connections. This was the case with older version of JMXTrans which required a restart when any of the services it is monitoring got restarted. Assuming that the older version of JMXTrans is used to monitor various services running on a node how do we restart JMXTrans when any of the process is restarted while using `` Chef `` to manage the environment.
<!--more-->
First, the processes which need to be monitored can vary in each node and there can be one or many processes. Since Chef role determines the recipes which are run on a node, a default attribute can be added to define the services JMXTrans is dependent on. The following is an example 

{% codeblock %}
...
"default_attributes" : {
   "jmxtrans":{
      "servers":[
                  {
                    "type": "datanode",
                    "service": "hadoop-hdfs-datanode",
                    "service_cmd": "org.apache.hadoop.hdfs.server.datanode.DataNode"
                  }, 
                  {
                    "type": "hbase_rs",
                    "service": "hbase-regionserver",
                    "service_cmd": “org.apache.hadoop.hbase.regionserver.HRegionServer"
                  }
                ]
              } ...
{% endcodeblock %}

Here the `` service `` key defines the name of the service on which JMXTrans is dependent on. The `` service_cmd `` key stores a string which can uniquely identify the running process of the service. The `` type `` key is not relevant for this discussion. These attributes will change for each role based on which service is installed as part of the role.

Since JMXTRans need to be installed, configured and started on all the nodes to collect statistics from various services a recipe need to be created for the same. But the recipe can now take into account the new node attribute to build the JMXTrans restart logic. The following is a sample code snippet which shows how the logic can be built

{% codeblock %}
...
jmx_services = Array.new

node['jmxtrans']['servers'].each do |server|
  jmx_services.push(server['service'])
  jmx_srvc_cmds[server['service']] = server['service_cmd']
end

service "restart jmxtrans on dependent service" do
  service_name "jmxtrans"
  supports :restart => true, :status => true, :reload => true
  action   :restart
  #
  # Create subcribes entries for each of the services on which JMXTrans is dependent on
  #
  jmx_services.each do |jmx_dep_service| 
    subscribes :restart, "service[#{jmx_dep_service}]", :delayed
  end
end
...
{% endcodeblock %}

With the ``subscribes`` clause added to the ``service`` resource for every service JMXTrans is monitoring on the node, restarts of any of these services during ``chef-client`` run will make sure that JMXTrans service is restarted. 

But what will happen if any of the dependent service is restarted externally for e.g. manually or upstart bounces a service. Then Chef will not restart JMXTrans during ``chef-client`` run since all the services are running. This will result in services restarted manually not being monitored by JMXTrans. The following is an updated code snippet which will handle this scenario. The complete recipe is available [here](https://github.com/bloomberg/chef-bach/blob/master/cookbooks/bcpc_jmxtrans/recipes/default.rb).

{% codeblock %}
...
jmx_services = Array.new
jmx_srvc_cmds = Hash.new
node['jmxtrans']['servers'].each do |server|
  jmx_services.push(server['service'])
  jmx_srvc_cmds[server['service']] = server['service_cmd']
end

service "restart jmxtrans on dependent service" do
  service_name "jmxtrans"
  supports :restart => true, :status => true, :reload => true
  action   :restart
  jmx_services.each do |jmx_dep_service| 
    subscribes :restart, "service[#{jmx_dep_service}]", :delayed
  end
  #
  # To determine any of the service JMXTrans depends on is started after JMXTrans service
  #
  only_if {process_require_restart?("jmxtrans","jmxtrans-all.jar", jmx_srvc_cmds)}
end
...
{% endcodeblock %}

As you may have noticed there is a `` only_if `` condition calls a library function which takes in a hash of all the services and the string which identifies the service processes as the third paramter. The first two paramters passed are the JMXTrans service name and a string which can uniquely identify a running JMXTrans process.

The following is the code snippet of the library function and the complete code can be found [here](https://github.com/bloomberg/chef-bach/blob/master/cookbooks/bcpc_jmxtrans/libraries/utils.rb).

{% codeblock %}
def process_require_restart?(process_name, process_cmd, dep_cmds)
  tgt_proces_pid = `pgrep -f #{process_cmd}`
  ...
  #
  # Find the start time of the JMXTrans process
  #
  tgt_proces_stime = `ps --no-header -o start_time #{tgt_process_pid}`
  ...
  ret = false
  restarted_processes = Array.new
  #
  # Loop through the processess of all the dependent services
  #
  dep_cmds.each do |dep_process, dep_cmd|
    dep_pids = `pgrep -f #{dep_cmd}` 
    if dep_pids != ""
      dep_pids_arr = dep_pids.split("\n")
      dep_pids_arr.each do |dep_pid| 
        #
        # Find the start time of a dependent process
        #
        dep_process_stime = `ps --no-header -o start_time #{dep_pid}`
        #
        # If the dependent process start time is greater than JMXTrans start time 
        # set the return value to true
        #
        if DateTime.parse(tgt_proces_stime) < DateTime.parse(dep_process_stime)
          restarted_processes.push(dep_process)
          ret = true
        end 
  ...
{% endcodeblock %}      

The logic behind the library function is to compare the start time of the JMXTrans process against the start time of processes of all the services it is dependent on. If any of the dependent processes start time is later than the start time of JMXTrans process then the JMXTrans service will get restarted.  

It would be good to have a Chef primitive that can accomplish what the library function does so that similar situations can use it. Hope this provides a solution option to restart a service dependent on multiple services during a ``chef-client`` run. 

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/).
