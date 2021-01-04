---
layout: post
title: "Chef HWRP using an example"
date: 2015-04-23 23:34:17 -0400
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [chef]
---
Heavy Weight Resource Provider (HWRP) is one of the options Chef offers to create custom resources and the other being [LWRP](http://blog.asquareb.com/blog/2015/04/19/chef-lwrp-using-hdfs-directory-as-an-example/). It would be good to read the notes on [LWRP](http://blog.asquareb.com/blog/2015/04/19/chef-lwrp-using-hdfs-directory-as-an-example/) to understand the context and the difference between ``LWRP`` and ``HWRP``.

Similar to LWRP, HWRP requires a resource definition and the corresponding provider. The key difference is that there are no DSL in the HWRP as in LWRP and everything is coded in ``Ruby`` code. So taking the same example of HDFS directory resource used in the notes on LWRP, the following is the skeleton of the resource definition. 
<!-- more -->
{% codeblock %}
class Chef
  class Resource
    class HdfsDir < Chef::Resource

      #
      # What provider this resource provides
      #
      provides :hdfsdir

      def initialize(name, run_context=nil)
        super

        #
        # Set the resource name
        #
        @resource_name = :hdfsdir

        #
        # Allowed actions in this resource
        #
        @allowed_actions = [:create, :delete, :chown, :chmod, :rename, :chgrp, :nothing]

        #
        # Default action if none specified when using the resource
        #
        @action = :create

        #
        # Set default values for resource attributes
        #
        @path = name
        @namenode = nil
        ...
      end
      
      #
      # Methods to get/set attributes and define additional characteristics
      #
      def path(arg=nil)
        set_or_return(:path, arg, :kind_of => String, :required => true)
      end

      def namenode(arg=nil)
        set_or_return(:namenode, arg, :kind_of => String, :required => true)
      end

      ...

    end
  end
end

{% endcodeblock %}

The HWRP is a ``Ruby`` class in this case ``HdfsDir`` which is a subclass of ``Chef::Resource`` class. The ``provides`` method specifies the resource provider for this resource and in this case it is ``hdfsdir``.

As in any ``Ruby`` class, the ``initialize`` method is used perform initializations like setting initial values of variables. In this case the pre-defined instance variable ``resource_name`` is set to a name which can be used to create a resource block in recipes using this HWRP. An array of symbols specifying the supported ``actions`` supported by this HWRP is assigned to the instance variable ``allowed_actions``. A default action which will be taken if an ``action`` is not set for while creating a resource using this HWRP (in this case ``create``) is set to the instance variable ``action``.

The remaining section in the skeleton is to define the characteristics of all the attributes of this resource which is similar to the attribute definition in LWRP. The key difference is that they are all defined as ``Ruby`` methods and the ``set_or_return`` is similar to ``Ruby`` ``attr_accessor`` method which creates the getters and setters for the attributes.

Unlike LWRP, the HWRP resource and provider code is stored in files under the ``libraries`` directory of the cookbook. Also there is no strict rules about the file naming conventions since these are `` Ruby `` classes and they get loaded first during the `` Chef client `` run.

Now lets turn to the corresponding provider definition and the following is the skeleton. It is more or less similar to the LWRP provider code we had seen earlier with some differences.

{% codeblock %}
require 'chef/log'

class Chef
  class Provider
    class HdfsDir < Chef::Provider

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
        require 'webhdfs'

        new_resource.user == nil ? @user = ENV['USER'] : @user = new_resource.user
        nnaddress = new_resource.namenode
        nnport = new_resource.nnport
        @client = WebHDFS::Client.new(nnaddress,nnport,@user)
        if (!validnn?())
          raise RuntimeError, "Invalid namenode provided or HDFS not available"
        end
        @omode = new_resource.mode
        new_resource.mode == nil ? @mode = "0750" : @mode = new_resource.mode
        @path = new_resource.path
        @tpath = new_resource.tpath
        @tgroup = new_resource.tgroup
        @tuser = new_resource.tuser
      end

      ...
      #
      # Action to create a directory in HDFS
      #
      def action_create
        if (dir_exists?(@path))
          Chef::Log::info("Directory #{ @path } exits; create action not taken")
        else
          converge_by("Create #{ @new_resource }") do
            @client.mkdir(@path,'permission' => @mode)
          end
          new_resource.updated_by_last_action(true)
        end
      end
    …
      #
      # Method to check whether the namenode provided is valid
      #
      def validnn?()
        return dir_exists?("/") ? true : false
      end
      ...
    end
  end
end

{% endcodeblock %}

As with the resource definition, the provider is also a ``Ruby`` class which is a subclass of ``Chef::Provider`` class. The method ``whyrun_supported`` is to specify whether the resource supports the ``chef client`` run with ``why-run`` option. If this method is set to return ``true``, then the strings provided in the ``converge_by`` statement of the ``action`` requested in the recipe will be logged instead of performing the actual convergence of the resource.

``load_current_resource`` method need to be overwritten in an HWRP which is optional in as LWRP. As discussed in the LWRP note, this method can be used to check the current state of the resource. 

The methods for the ``actions`` supported are defined using the naming convention ``action_name``. For e.g. for the ``create`` action the method name is ``action_create``. Supporting methods can be defined as in any ``Ruby`` class for e.g. in this case `validnn??`` method. 

The method ``new_resource.updated_by_last_action`` is called with a value of ``true`` so that ``Chef`` is notified that the resource got updated by that particular ``action``.

More notes in this category can be found [here](http://blog.asquareb.com/blog/categories/chef/).
