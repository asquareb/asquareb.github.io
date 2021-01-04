---
layout: post
title: "Chef LWRP using HDFS directory as an example"
date: 2015-04-19 22:48:58 -0400
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [chef]
---
``Chef`` provides a large set of ``resources`` to work with. But there are situations where  resources provided by ``Chef`` may not be sufficient. For e.g, distributed file systems can’t be handled by the file system related resources (``file``, ``directory`` etc) which comes out of the box with ``Chef``. Being flexible and customizable, ``Chef`` provides two options (LWRP, HWRP) for users to create their own resources.  
<!-- more -->
Light Weight Resource Providers (LWRP) use DSL to simplify the creation of resources and are used when existing chef ``resources`` can be leveraged with minimal ``Ruby`` code. In contrast, Heavy Weight Resource Providers (HWRP) are used when existing resources can’t be leveraged and ``Ruby`` code need to be used to implement the resource provider. 

Lets quickly look at a LWRP using HDFS (which is a distributed file system) directory resource as an example. A LWRP is created and stored in a cookbook and there are two parts to it. First the resource definition which defines the actions supported and attributes accepted by the LWRP. The resource definition resides in the ``resources`` directory of the cookbook. The second part is the provider or simply the code which implements the actions supported by the LWRP. The provider is stored in the ``provider`` directory of the cookbook in which the LWRP is being created.

For ``chef`` to be able to identify the new resource, the file name of the resource definition and the provider file need to have the same name. For e.g. lets assume that the HDFS directory resource is created in hdfs cookbook and the resource is named hdfsdir, the following will be the cookbook directory structure (showing only the required directories)

{% codeblock %}
hdfs
  |_____ providers
  |             |________ hdfsdir.rb        
  |
  |_____ Resources
                |________ hdfsdir.rb

{% endcodeblock %}

When the resource need to be used in a ``recipe``, the resource need to be prefixed with the cookbook name separated by an “_” (underscore).  For e.g.

{% codeblock %}

hdfs_hdfsdir “/tmp/pass” do
  action :delete
end

{% endcodeblock %}

Lets look at the LWRP ``resource`` definition for HDFS directory resource

{% codeblock %}
actions :create, :delete, :chown, :chmod, :rename, :chgrp
default_action :create
#
# fqdn or ip address of the name node server
#
attribute :namenode, :kind_of => String, :required => true
#
# port number of the namenode
#
attribute :nnport, :kind_of => String, :required => true
#
# Directory path on which actions need to be taken
#
attribute :path, :kind_of => String, :name_attribute => true, :required => true
...
{% endcodeblock %}

``actions`` defines the actions supported by the new resource.

``default_action`` defines the action when the resource is used in an recipe and no action clause is specified in the recipe.

``attribute`` defines each attribute which can be set when using the resource. It also defines whether the attribute is required and the attribute type. 

One ``attribute`` can take the ``name`` value of the resource if it is not set explicitly in the recipe and in this case the ``attribute`` path takes the name value of the resource and it is specified using ``:name_attribute => true``. The complete definition can be found [here](https://github.com/bijugs/simple-scripts/blob/master/hdfsdir_resource.rb).

With the ``resource`` definition out of the way lets look at the ``provider`` for the hdfs directory resource. The following is the code skeleton for the provider and the full code can be found [here](https://github.com/bijugs/simple-scripts/blob/master/hdfsdir_provider.rb).

{% codeblock %}
require 'chef/log'
require 'webhdfs'
#

#
use_inline_resources
#
# To enable -W/--why-run option of chef-client
#
def whyrun_supported?
   true
end
#
# Method automatically called by Chef during the client execution phase
# Can be used to initialize variables and also verify the current state
#
def load_current_resource
   @current_resource = Chef::Resource::HdfsHdfsdir.new(new_resource.name)
   new_resource.user == nil ? @current_resource.user = ENV['USER'] : @current_resource.user = new_resource.user

...
end

action :create do
  if (dir_exists?(@path))
    Chef::Log::info("Directory #{ @path } exits; create action not taken")
  else
    converge_by("Create #{ @new_resource }") do
      @client.mkdir(@path,'permission' => @mode)
    end
    new_resource.updated_by_last_action(true)
  end
end

action :delete do
...
end

...
{% endcodeblock %}

The `` require `` method is used to include external files as in any Ruby code. Note that ``webhdfs`` gem need to installed for this code to work.

 ``use_inline_resources`` is a must for LWRP. The reason is to make sure that any ``notifications`` raised from any of the resources in the LWRP (remember LWRP can leverage other Chef resources to implement its functionality) will be treated as being raised by the LWRP resource collection as a whole and not the individual resource within the LWRP resource which is raising the notification.

``new_resource`` instance object is automatically created when the resource is used and the attribute values in the new object is set to the values passed from the recipe that is using the resource.

When creating resources one of the key requirement is to make sure that it is **idempotent**. This requires the current state of the resource to be known. For this ``chef`` provides an empty method ``load_current_resource`` which can be overwritten by the resource provider. Since this method will be the first to be called when the resource is used in a recipe, the method call can be used to check the current state of the resource. For e.g. if the directory resource is already existing and the resource action is to ``create`` the directory, the resource provider can skip the requested action since the directory is up to date. For anyone interested in more details look into the code for [LWRPBase class](https://github.com/chef/chef/blob/24c3387634f32f27f63d388ddbf64004e17c311b/lib/chef/provider/lwrp_base.rb) which is the parent for the LWRP provider.

The remaining sections in the code skeleton are to implement the actions supported by the resource. They can use the existing ``chef`` resources and/or use Ruby code. If you had a chance to look at the complete code for [hdfsdir provider](https://github.com/bijugs/simple-scripts/blob/master/hdfsdir_provider.rb) contrary to how LWRP is meant to be implemented, the code doesn’t use any existing ``chef`` resource since there is none for a distributed file system like HDFS and all the actions had to be implemented using ``Ruby``. But to understand the various aspects of writing an LWRP it is still helpful.

``whyrun_supported?`` method is used to enable/disable support for the ``--why-run`` option of ``chef-client`` by setting the return value to ``true`` or ``false``.

When ``whyrun_supported?`` is set to ``true`` and if ``chef-client`` run uses ``--why-run`` option the string passed to  ``converge_by`` clause will be logged instead of actual convergence.

When an action is taken on a resource ``new_resource.updated_by_last_action(true)`` is used to notify ``chef`` that the resource was updated by the requested action.

Finally, note that you can use the hdfs LWRP code used in this example if you are dealing with HDFS by renaming the files and copying into the resources and provider directories of your cookbook.     

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/)
