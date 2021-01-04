---
layout: post
title: "Download and Install Software Using Chef"
date: 2014-06-10 05:33:55 -0400
author: Biju Nair
comments: true
categories: [chef, infrastructure-mgmt]
---
With open source software it is common to download and install software from the web. And with distributed computing plaftforms like hadoop software need to get installed on multiple servers. This can be automated using Chef. The following is an example of how zookeeper software can be downloaded and installed.

- Create a Chef cookbook. Assumed that Chef is already available for use.
{% codeblock %}
knife create cookbook zookeep
{% endcodeblock %}
<!-- more -->

- Create default.rb under attributes folder with attributes for where the software need to be installed, version of software to be downloaded and the URL to download from
{% codeblock %}
default[:zookeeper][:install_dir]="/opt"
default[:zookeeper][:version]="3.4.6"
default[:zookeeper][:download_url]="http://apache.claz.org/zookeeper/stable"
{% endcodeblock %}

- Create a default recipe "default.rb" under the recipe folder in the cookbook with the following resources
{% codeblock %}
tarball = "zookeeper-#{node[:zookeeper][:version]}.tar.gz"
download_file = "#{node[:zookeeper][:download_url]}/#{tarball}"

remote_file "#{Chef::config[:file_cache_path]}/#{tarball}" do
  source download_file
  action :create_if_missing
  mode 00644
end
{% endcodeblock %}
 The tarball variable will store the name of the zookeeper gzip file available for download from the apache mirror site
 The download_file variable stores the complete URL including the gzip file name for download. Both these variables use values from the attribute values defined in attributes file.
 "remote_file" resource is the chef resource which is used to retrieve remote files. The name passed to the remote_file resource is the destination where the remote file need to be stored and in this case is the temporary cache directory used by Chef. The download is defined to happen only if the tar file is not already available in the cache thus preventing unnecessary download. 

{% codeblock %}
zk_install_dir=node[:zookeeper][:install_dir]

execute "tar" do
  user "root"
  group "root"
  cwd zk_install_dir
  action :run
  command "tar xvzf #{Chef::config[:file_Cache_path]}/#{tarball}"
  not_if{ ::File.directory?("#{zk_install_dir}/zookeeper")
end
{% endcodeblock %}
 "zk_install_dir" variable will store the directory into which the zookeeper software need to be installed into and uses the value set to default[:zookeeper][:install] attribute in the attribute file.
 The execute resource executes the tar command to untar/unzip the gzip file downloaded from the apache site into the folder store in the "zk_install_dir" variable. The tar command gets executed only if a "zookeeper" directory is not in the "/opt" directory which is the value set in "zk_install_dir".

- Once untarred the directory created will include the version number of zookeeper downloaded. Inorder to remove the version number the following execute resource can be used.
{% codeblock %}
execute "move" do
  user "root"
  group "root"
  cwd zk_install_dir
  action :run
  command "mv zookeeper* zookeeper"
  not_if{ ::File.directory?("#{zk_install_dir}/zookeeper")
end
{% endcodeblock %}

This recipe downloaded and installed software at the required location. Using Chef we can run the recipe in multiple nodes and automate the repeatable process. We will improve on this recipe next time. 

Until next time Happy computing!!
 
