---
layout: post
title: "Basic Chef Knife Commands"
date: 2014-06-09 07:16:50 -0400
author: Biju Nair
comments: true
categories: [chef, infrastructure-mgmt]
---
While using any tool, 80% of all work is accomplished by very small percentage of commands available. Following are some of the commonly used Chef Knife commands.

- To create a new cookbook
{% codeblock %}
knife cookbook create cookbook_name 
{% endcodeblock %}
  Note: use underscore instead of "-" (hyphen) in cookbook names. Hypen seem to prevent identification of LWRPs included in the cookbook.
<!-- more -->
- To upload a new or modified cookbook
{% codeblock %}
knife cookbook upload cookbook_name
{% endcodeblock %}

- To list all the nodes currently defined in Chef
{% codeblock %}
knife node list 
{% endcodeblock %}

- To show details about a node
{% codeblock %}
knife node show node_name -l
{% endcodeblock %}
 Note: -l option generates a detailed list of properties about the node. Leaving it out will generate a summary about the node.
 Use -a option to query about a particular node attribute

- To edit a new node or modifying an existing node
{% codeblock %}
knife node edit node_name
{% endcodeblock %}
 This will let you edit the node details using the default editor set

- To add a node defined in a file to Chef
{% codeblock %}
knife node from file file_name
{% endcodeblock %}

- To list all the roles defined in Chef
{% codeblock %}
knife role list
{% endcodeblock %}

- To show details about a role
{% codeblock %}
knife role show role_name
{% endcodeblock %}
 Use -a option to query about a particular role attribute value 

- To add a role from a file
{% codeblock %}
knife role from file file_name
{% endcodeblock %}

- To edit a new or existing role
{% codeblock %}
knife role edit role_name
{% endcodeblock %}

- To add a role or recipe to a node
{% codeblock %}
knife node run_list add node_name "recipe[cookbook::recipe]"
knife node run_list add node_name "role[role_name]"
knife node run_list add node_name "role[role_name],recipe[cookbook::recipe]"
{% endcodeblock %}

- To remove role or recipe from a node use "remove" instead of "add"
{% codeblock %}
knife node run_list remove node_name "role[role_name]"
{% endcodeblock %}

- To bootstrap a new node with Chef
{% codeblock %}
knife bootstrap new-host-ip -x root -P password -N node_name
{% endcodeblock %}

Happy Cheffing!!
