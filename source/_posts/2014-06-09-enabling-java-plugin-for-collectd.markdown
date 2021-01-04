---
layout: post
title: "Enabling JAVA plugin for collectd"
date: 2014-06-09 21:33:50 -0400
author: Biju Nair
comments: true
categories: [monitoring, infrastructure-mgmt]
---
If you are trying to enable JAVA plugin on Ubuntu platform and facing issues the following is something you can try to fix it.

- Since JAVA plug-in is not enabled by default, collectd need to be compiled from the code.

- Download the source code and configure collectd with the following options
<!-- more -->
{% codeblock %}
./configure --with-java=$JAVA_HOME JAVA_CPPFLAGS="-I$JAVA_HOME/include -I$JAVA_HOME/include/linux" JAVA_CFLAGS="-I$JAVA_HOME/include -I$JAVA_HOME/include/linux" JAVA_LDFLAGS="-I$JAVA_HOME/include -I$JAVA_HOME/include/linux" JAVA_LIBS="-I$JAVA_HOME/include" JAR="/path/to/jar" JAVAC="/path/to/javac" --enable-write_graphite --enable-java=force 
{% endcodeblock %}

 Note: JAVA_HOME is something like /usr/lib/jvm/java-1.7.0-openjdk-amd64. Also without the option "force" for enable-java, configure will fail

- Once configure is complete, compile and install collectd

{% codeblock %}
make all install
{% endcodeblock %}

collectd gets installed by default under /opt/collectd in Ubuntu

- Start collectd using the following command assuming that the PWD is /opt/collectd/sbin

{% codeblock %}
LD_PRELOAD=/usr/lib/jvm/java-1.7.0-openjdk-amd64/jre/lib/amd64/server/libjvm.so ./collectd
{% endcodeblock %}

- Verify that collectd is running using "ps" command. If it is not running check the syslog by default located at /var/log/syslog.

- If there are no entries in syslog try starting the collectd process in the foreground which will write any errors on the console

{% codeblock %}
./collectd -f
{% endcodeblock %}

Have fun with collectd and JMX stats collection!! 
