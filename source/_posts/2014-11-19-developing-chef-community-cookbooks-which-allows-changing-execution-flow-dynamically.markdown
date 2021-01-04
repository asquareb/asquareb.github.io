---
layout: post
title: "Developing Chef community cookbook which allows changing execution flow dynamically"
date: 2014-11-19 07:37:24 -0500
author: Biju Nair
comments: true
categories: [chef, chef-patterns, infrastructure-mgmt]
---
If you have been working on Chef you may already know that attributes and resources in recipes defined in a community cookbook can be modified. But there can be a need to allow users of community cookbook to be able to change the execution flow of recipes in the community cookbook. We will go over an approach using an example. 

<!-- more -->

Consider a generic use case of a community cookbook which installs a service along with its configuration which can be modified by the user of the cookbook using a wrapper cookbook. When there is a change in the configuration the service need to be restarted so that the configuration can take in effect. 
In a distributed compluting platform, multiple instances of a service is deployed in an environment/cluster for scalability and availability. When chef-client is run as daemon regularly which is typically the case in large clusters for cluster management, there is a possibility that auto run of chef-client on the nodes of the cluster can bring down and restart all the service instances at the same time so that the new configuration can take in effect. This can result in service outage which is not preferred. So there need to be some coordination which need to be implemented so that all instances of the service is not brought down at the same time and this will vary with each user using the community cookbook. If you are wondering about cases which matches this generic use cases, some of the distributed computing platform components like Apache HDFS, HBase, YARN, Kakfa, ZooKeeper are good examples.

Here is an approach to provide the flexibility for users to be able to incorporate the logic to coordinate the restart of the service so that all instances doesn't come down at the same time. We will use Kafka as an example. Kafka server gets deployed at multiple nodes to form a cluster so that it is highly available and scalable.

- Add a new attribute to the cookbook which defines the name of the recipe which should be used to coordinate start/restart of the service. For e.g. assuming Kafka as the cookbook name the attribute can be

{% codeblock %}
  default[:kafka][:start_coordination][:recipe] = `kafka::_coordinate`
{% endcodeblock %}
- Instead of defining a service resource in a recipe in the cookbook to start/restart the Kafka service, create a new recipe called `` _coordinate `` similar to the following
{% codeblock %}
  ruby_block "coordinate_kafka_start" do
    block do
      Chef::Log.debug ("Default recipe to coordinate Kafka start is used")
    end
    action :create
    notifies :restart, 'service[kafka]', :delayed
  end
  service 'kafka' do
    provider kafka_init_opts[:provider]
    supports start: true, stop: true, restart: true, status: true
    action [:enable, :start]
  end
{% endcodeblock %}
- Include this recipe at the location where you would have defined the service resource to start/restart Kafka. For e.g.
{% codeblock %}
  include_recipe node.kafka.start_coordination.recipe
{% endcodeblock %} 
- Send all notifications normally targeted to the service resource like notifications from template resources for configuration changes, redirect it to the ruby_block resource in the `` _coordinate `` recipe with `` :immediate `` option.
{% codeblock %}
  notifies :create, 'ruby_block[coordinate_kafka_start]', :immediately
{% endcodeblock %}
- This will make the community cookbook flexible enough for users to be able to include their logic to control the service restart process.

If you are an user who would want to use a community cookbook with similar option to overwrite the restart logic, the following is an approach on how to go about overwriting the default logic without making any modifications to the community cookbook.

- Assuming that you will be creating a cookbook (e.g. ``use_community``) to use the community cookbook, add a new recipe to the cookbook similar to the following
{% codeblock %}
  ruby_block "coordinate_kafka_start" do
    block do
      Chef::Log.info (" Custom recipe to coordinate Kafka start is used")
    end
    action :create
    notifies :create, 'ruby_block[restart_coordination]', :delayed
  end
  ruby_block "restart_coordination" do
    block do
      Chef::Log.info ("Need to implement the process to coordinate the restart process like using ZK")
    end
    action :nothing
    notifies :restart, 'service[kafka]', :delayed
  end
  service 'kafka' do
    provider kafka_init_opts[:provider]
    supports start: true, stop: true, restart: true, status: true
    action [:enable, :start]
  end
  ruby_block "restart_coordination_cleanup" do
     block do
       Chef::Log.info ("Implement any cleanup logic required after restart like releasing locks")
     end
     action :nothing
  end
{% endcodeblock %}
Note that the first resource is the same as the default `` _coordinate `` recipe in the community cookbook and it is requirement so that notifcations are intercepted appropriately. Also the new cookbook should retain the service resource defined in the community `` _coordinate `` recipe.

- Since all the restart notifications will be received by the `` ruby_block[coordinate-kafka-start] `` resource logic can be included in the new recipe as in the sample above.

- Now we need to overwrite the community cookbook attribute value which stored the name of the recipe to be used for the start/restart process. It is easy as setting the attribute value in wrapper cookbook. In order to have the overwrite to take in effect, atleast one recipe from the wrapper cookbook should be included in the run list. This an be accomplished by creating a "NOP" recipe or a recipe which does the overwrite of the attribute and include it in the run list.

- The following is an example of the content in the  recipe which over writes the attribute in the community cookbook
{% codeblock %}
node.default.kafka.start_coordination.recipe = 'kafka-bcpc::coordinate'
{% endcodeblock %} 
- Once this is done, update the role which installs the service with the recipe of the wrapper cookbook

{% codeblock %}
{
  "name": "Kafka-Server",
  "json_class": "Chef::Role",
  "run_list": [
    "role[Basic]",
    ...
    "recipe[kafka-use-community::default]",
    "recipe[kafka-community::setattr]",
    ...
  ],
  "description": "Role to setup Kafka Server",
  ...
{% endcodeblock %}

Even though the write-up is long, as you may have noted the idea is to simply use a notification interceptor `` ruby_block `` to pass control to the users of the community cookbook. In this case `` ruby_block "coordinate_kafka_start" ``. 

The same technique can be used to change the behavior of other community cookbooks by the users of the cookbook. As with any framework the users of community cookbook need to know the notification interceptor ``ruby_block`` and the creator of the community cookbook should adhere to the framework strictly. 

 If you are looking for real world example,[Kafka community cookbook](https://github.com/mthssdrbrg/kafka-cookbook/pull/63) is an example and the [readme](https://github.com/mthssdrbrg/kafka-cookbook/blob/master/README.md#controlling-restart-of-kafka-brokers-in-a-cluster) provides the details about restart coordination.

Also if you are looking for other similar chef patterns and solutions using chef, come join us at [ChefConf 2015](http://sched.co/2HtF).

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/).
