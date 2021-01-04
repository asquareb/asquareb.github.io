---
layout: post
title: "Dynamically requesting a common service in a Chef managed environment"
date: 2015-02-21 22:39:48 -0400
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [chef, chef-patterns,infrastructure-mgmt]
---
In a cluster environment there can be a common service running on all the nodes in the cluster which may be requested by an application using the cluster. For e.g. in a distributed computing environment logs can be created on each node in the cluster and a common service on each cluster node can be a service which regularly copies log entries into a common location. Copying of logs can be a requirement for some of the applications/services running on the cluster nodes so that users can go to one location to view the log data from all the nodes instead of looking at each node. This can also help with managing the security of the cluster by providing access to this common location instead of providing access to all the nodes in the cluster. 
<!-- more -->
But copying of the logs may not be a requirement for some other service or application and the requirement may change in the future. It is better to build a solution so that the changes in the requirements to copy logs can be accommodated dynamically.

To explain the pattern on how we can accomplish the requirement, we will assume `` Apache Flume `` as the component which will be installed on all the nodes in the cluster and will be used to copy the log file entries to a common location in HDFS. The following is one solution to satisfy the requirement. 

Create a new cookbook or reuse an existing cookbook which will be used on all the nodes in the cluster. Create a node attribute which will store a hash of hashes. For e.g.

{% codeblock %}
default['bcpc']['hadoop']['copylog'] = {}
{% endcodeblock %}

The following is the datastructure which will be used to populate the hash values to be stored in the attributes 

{% codeblock %}
{
  'app_id' =>  { 'logfile' => "/path/file_name_of_log_file",
                 'docopy' => true (or false)
                },...
}
{% endcodeblock %}

where the `` app_id `` is an unique id of an application which requests the service to copy its logs, `` logfile `` is the path of the logfile and `` docopy `` is a flag to enable and disable the request to copy the log files based on change in need. If an application requests more than one logfile to be copied the `` appli_id `` need to be made unique with in the application by adding additonal details like log name or a sequence number.

When an application need to make a request to copy its log file, in the application's recipe, request can be added to the node attribute. For e.g. if logs from the nodes running `` hbase-region-server `` need to copy its logs to the common location, code similar to the following can be included in the recipe installing and starting the `` hbase-region-server `` service.

{% codeblock %}
node.default['bcpc']['hadoop']['copylog']['hbase_regionserver_log'] = {
    'logfile' => "/var/log/hbase/hbase-regionserver-#{node.hostname}.log",
    'docopy' => true
}

node.default['bcpc']['hadoop']['copylog']['hbase_regionserver_out'] = {
    'logfile' => "/var/log/hbase/hbase-regionserver-#{node.hostname}.out",
    'docopy' => true
}
{% endcodeblock %}

Note here how an unique identifier is used as the ``app_id`` for the two log files from `` hbase_regionserver `` to be copied to the common location in HDFS.

For someone not familiar with `` Flume ``, flume agents can be used to move data and they need a config file which defines things like the source, sink, the type of sink,the target location etc. Since `` Flume `` need to be installed in all nodes in the cluster to be a common service, a recipe need to be created to install the software. But when it comes to start `` Flume agents ``, the recipe need to loop through the node attribute to create required Flume configuration files for each logfile copy requests made by the applications running on the node. 

For e.g if the node is running only `` hbase_regionserver `` there will be two copy requests in the attribute for which the recipe need to create Flume configuration files and start the corresponding `` Flume agents ``. If there are other applications/services running on the node for e.g. `` HDFS Datanode `` and if the recipes make copy requests then the number of flume configs and `` flume agents `` running on the node will be more. The following is the code snippet on how the `` Flume `` recipe can accomplish this

{% codeblock %}
...
node['bcpc']['hadoop']['copylog'].each do |id,f|
   if f['docopy'] 
     template "/etc/flume/conf/flume-#{id}.conf" do
       source "flume_flume-conf.erb”
       action :create ...
       variables(:agent_name => "#{id}",
                 :log_location => "#{f['logfile']}" )
       notifies :restart,"service[flume-agent-multi-#{id}]",:delayed
     end
     service "flume-agent-multi-#{id}" do
       supports :status => true, :restart => true, :reload => false
       service_name "flume-agent-multi"
       action :start
       start_command "service flume-agent-multi start #{id}"
       restart_command "service flume-agent-multi restart #{id}"
       status_command "service flume-agent-multi status #{id}"
     end
...
{% endcodeblock %}

A complete `` Flume `` recipe which includes this feature is available [here](https://github.com/bloomberg/chef-bach/blob/master/cookbooks/bcpc-hadoop/recipes/copylog.rb). 

By default the init.d script for Flume starts one `` Flume agent ``. But BIGTOP jira [BIGTOP-1581](https://issues.apache.org/jira/browse/BIGTOP-1581) provides an option to start/stop/restart flume agents by passing agent names as in the `` service `` resource in the previous code snippet. The name passed need to have a corresponding Flume config file under the configuration folder. Since `` Flume `` recipe need to know the copy request from all the applications running on the node, it should be the last recipe to be run on the node. This can be accomplished by creating a role to include the `` Flume `` recipe and add it to the end of chef-client runlist of all the nodes.

Hope this provides a solution option in a similar situation where applications can dynamically request for a common service in an environment. Also note that the data structure can be expanded so that applications can make richer requests for e.g. in this case request to change the location of the common location where logs are copied into etc. 

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/).
