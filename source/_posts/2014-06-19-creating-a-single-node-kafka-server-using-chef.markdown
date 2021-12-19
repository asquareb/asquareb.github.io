---
layout: post
title: "Creating a single node Kafka server using Chef"
date: 2014-06-19 18:13:42 -0400
author: Biju Nair
comments: true
categories: [chef, infrastructure-mgmt]
---
If you need to create a single node Kafka server quickly for development or testing and need to be repeatable, the following are the steps. This uses Chef-Solo and community cookbooks.
<!-- more -->
<h2>Pre-req:</h2>
- A linux server machine or VM with just the OS installed

- Connection to internet

- git, curl and wget tools available for the user

- If the tools are not available, as root or using sudo install these tools using apt-get or yum

<h2>Installing Chef:</h2>
- As root: 
{% codeblock %}
curl -L https://www.opscode.com/chef/install.sh | bash
{% endcodeblock %}
- cwd to a directory where you want to run chef-solo from 

- Run: 
{% codeblock %}
wget http://github.com/opscode/chef-repo/tarball/master
{% endcodeblock %}

- Do:
{% codeblock %} 
tar -zxf master
{% endcodeblock %}

- Do:
{% codeblock %}
mv opscode-chef-repo* chef-repo
{% endcodeblock %}

- Do: 
{% codeblock %}
rm master
{% endcodeblock %}

- cd to chef-repo; if you do a ls there will be a list of directories
{% codeblock %}
mkdir .chef
{% endcodeblock %}

- Do: 
{% codeblock %}
echo "cookbook_path [ '/path-to/chef-repo/cookbooks' ]" > .chef/knife.rb
{% endcodeblock %}

- This completes the installation and configuration of Chef

- cd into the chef-repo/cookbooks directory
{% codeblock %}
git clone https://github.com/socrata-cookbooks/java.git
git clone https://github.com/bijugs/kafka-cookbook.git -b kafka_enhancement kafka
{% endcodeblock %}

- cd into kafka directory
{% codeblock %}
vi attributes/zookeeper.rb 
{% endcodeblock %}
and change the default[:zookeeper][:data_dir] value if required

- Under chef-solo directory: vi solo.rb and add the following details
{% codeblock %}
file_cache_path "/where-ever/chef-solo"
cookbook_path "/where-ever/chef-repo/cookbooks"
{% endcodeblock %}
	
- Under chef-solo directory: vi kafka.json and include the following
{% codeblock %}
{
  "java": {
  	"jdk_version" : "7",
  	"accept_license_agreement": true,
  	"oracle": {
  	  "accept_oracle_download_terms": true
  	}   
   },
   "run_list": [
     "recipe[java::oracle]",
     "recipe[kafka::zookeeper]",
     "recipe[kafka]"
   ]
}
{% endcodeblock %}

- Staying under the chef-solo directory run
{% codeblock %}
sudo chef-solo -c solo.rb -j kafka.json
{% endcodeblock %}

Happy cheffing!!
