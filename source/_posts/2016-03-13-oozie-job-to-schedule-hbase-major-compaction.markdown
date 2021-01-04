---
layout: post
title: "Oozie job to schedule HBase major compaction"
date: 2015-12-20 12:10:23 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: [oozie, hbase]
---
HBase performs time based major compaction and in-order to prevent this resource intensive process interfere performance sensitive applications, this can be disabled. Once disabled, in order to keep the HBase store files in optimal condition, application team need to schedule regular compaction. The following details the steps which need to be followed to schedule daily compaction through Oozie. 
<!--more-->

**Non Kerberized Cluster**
Create the following files with the content shown in a local directory. In this example the files are created in hbasecompact under the users local home directory. Please note that the files are under different directories.
Shell script to start HBase compaction on a table and copy the output of the command to a HDFS directory. The command output can be checked for any issues.
{% codeblock %}
~$ cat hbasecompact/scripts/hbaseCompact.sh
#!/bin/ksh
echo "Starting HBase major compaction on table $1 - $3" > majorcompact.log
echo "major_compact \"$1\"" | /usr/bin/hbase shell 2>&1 >> majorcompact.log
echo "HBase major compaction request complete on table $1" >> majorcompact.log
hdfs dfs -moveFromLocal -f majorcompact.log $2
{% endcodeblock %}
Shell script to copy the Oozie job id into a HDFS directory. The job id can be used to check any issues.
{% codeblock %}
~$ cat hbasecompact/scripts/logOozieId.sh
echo $1 > oozieId.log
hdfs dfs -moveFromLocal -f oozieId.log $2
{% endcodeblock %}
Oozie workflow definition to perform HBase compaction. The first step  major_compact runs the script hbaseCompact.sh. The next step logOozieId runs the script logOozieId.sh to copy the Oozie workflow id onto HDFS.
{% codeblock %}
~$ cat hbasecompact/workflow/workflow.xml
<workflow-app xmlns='uri:oozie:workflow:0.5' name="major_compact_wf">
   <global>
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>
      <configuration>
         <property>
            <name>mapred.job.queue.name</name>
            <value>${queueName}</value>
         </property>
      </configuration>
   </global>
   <start to="major_compact"/>
   <action name="major_compact">
      <shell xmlns="uri:oozie:shell-action:0.3">
         <configuration>
            <property>
              <name>oozie.launcher.mapreduce.map.memory.mb</name>
              <value>${mapMemoryMB}</value>
            </property>
         </configuration>
         <exec>${majorCompactScriptPath}/${majorCompactScriptName}</exec>
         <argument>${tableName}</argument>
         <argument>${hdfsLogDir}</argument>
         <argument>${timestamp()}</argument>
         <file>${majorCompactScriptPath}/${majorCompactScriptName}#${majorCompactScriptName}</file>
         <capture-output/>
      </shell>
      <ok to="logOozieId"/>
      <error to="logOozieId"/>
   </action>
   <action name="logOozieId">
      <shell xmlns="uri:oozie:shell-action:0.3">
         <exec>${majorCompactScriptPath}/${logOozieIdScriptName}</exec>
         <argument>${wf:id()}</argument>
         <argument>${hdfsLogDir}</argument>
         <file>${majorCompactScriptPath}/${logOozieIdScriptName}#${logOozieIdScriptName}</file>
      </shell>
      <ok to="end"/>
      <error to="fail"/>
   </action>
   <kill name="fail">
      <message>Major compaction failed [${wf:errorMessage(wf:lastErrorNode())}]</message>
   </kill>
   <end name="end"/>
</workflow-app>
{% endcodeblock %}
Oozie coordinator.xml definition to run the major_compact_wf workflow defined in the previous step. 
{% codeblock %}
~$ cat hbasecompact/coordinator/coordinator.xml
<coordinator-app name="major_compact"
   frequency="${coord:days(1)}"
   start="${starttime}" end="${endtime}" timezone="${timezone}"
   xmlns="uri:oozie:coordinator:0.1">
    <controls>
      <concurrency>1</concurrency>
      <execution>FIFO</execution>
    </controls>
    <action>
      <workflow>
         <app-path>${nameNode}${rootDir}/workflow</app-path>
         <configuration>
            <property>
               <name>HBaseMajorCompact</name>
               <value>'${coord:user()}: Kicking off HBase Major Compact WF'</value>
            </property>
         </configuration>
      </workflow>
   </action>
</coordinator-app>
{% endcodeblock %}
**Note:** Do not use 1440 minutes as frequency in workflow.xml if the expectation is to run compaction everyday at a certain time since this will cause change in job run time when system time gets changed for day light savings. The starttime and endtime should be specified in UTC/GMT. The timezone is required for Oozie to invoke the logic to handle the time changes due to day light savings.
Properties which need to be substituted in the place of the parameters defined in the workflow and coordinator xml files. If this example is used, this is the only file which need to be changed. Inline comments will help in making the changes.
{% codeblock %}
~$ cat hbasecompact/coordinator/coordinator.properties
//
//HDFS Namenode URL: You can find it in hdfs-site.xml
//If HDFS HA is enabled use the value of dfs.nameservices
//
nameNode=hdfs://NN-HA
//
//URL of jobTracker for MR1
//If MR2/YARN is used, use the YARN RM URL : YARN-RM:8032
//If YARN HA is enabled use the YARN cluster-id which is specified in 
//yarn.resourcemanager.cluster-id property of yarn-site.xml
//
jobTracker=YARN-RM-CLUSTER-ID
//
// YARN queue to which the workflow MR jobs need to be submitted
//
queueName=default
//
// HDFS directory where the oozie application is stored
//
rootDir=/user/userid/hbasecompact
//
// HDFS directory where workflow.xml is stored
//
wf.application.path=${nameNode}${rootDir}/workflow
oozie.wf.rerun.failnodes=true
//
// HDFS directory where scripts in the workflow are located
//
majorCompactScriptPath=${nameNode}${rootDir}/scripts
majorCompactScriptName=hbaseCompact.sh
logOozieIdScriptName=logOozieId.sh
//
// HDFS directory where the script output need to be stored
//
hdfsLogDir=/user/userid/compact
//
// Table name which need to be compacted
//
tableName=t
mapMemoryMB=8192
//
// Date time to start and stop the workflow 
//
starttime=2015-10-22T10:24Z
endtime=2020-10-22T10:26Z

timezone=America/New_York
//
// HDFS directory where the coordinator.xml is stored
//
oozie.coord.application.path=${nameNode}${rootDir}/coordinator
oozie.use.system.libpath=true
oozie.libpath=${nameNode}/user/oozie/share/lib
{% endcodeblock %}
If you list the local directory where these file are stored it will look like this
{% codeblock %}
~$ ls -ls -R hbasecompact/
hbasecompact/:
total 12
4 drwxr-xr-x 2 userid users 4096 Oct 22 14:46 coordinator
4 drwxr-xr-x 2 userid users 4096 Oct 22 14:15 scripts
4 drwxr-xr-x 2 userid users 4096 Oct 22 14:05 workflow

hbasecompact/coordinator:
total 8
4 -rw-r--r-- 1 userid users 1197 Oct 22 14:46 coordinator.properties
4 -rw-r--r-- 1 userid users  640 Oct 22 12:35 coordinator.xml

hbasecompact/scripts:
total 8
4 -rwxr-xr-x 1 userid users 243 Oct 22 12:02 hbaseCompact.sh
4 -rwxr-xr-x 1 userid users  65 Oct 22 12:23 logOozieId.sh

hbasecompact/workflow:
total 4
4 -rw-r--r-- 1 userid users 1853 Oct 22 14:05 workflow.xml
{% endcodeblock %}
copy them into to a HDFS directory
{% codeblock %}
hdfs dfs -copyFromLocal ./hbasecompact /user/userid
{% endcodeblock %}
If you list the HDFS directory which was used as the target location in the previous step, the output should be similar to the following.
{% codeblock %}
~$ hdfs dfs -ls -R /user/userid/hbasecompact
drwxr-xr-x   - userid supergroup          0 2015-10-22 12:58 /user/userid/hbasecompact/coordinator
-rw-r--r--   3 userid supergroup        513 2015-10-22 12:58 /user/userid/hbasecompact/coordinator/coordinator.properties
-rw-r--r--   3 userid supergroup        640 2015-10-22 12:58 /user/userid/hbasecompact/coordinator/coordinator.xml
drwxr-xr-x   - userid supergroup          0 2015-10-22 12:58 /user/userid/hbasecompact/scripts
-rw-r--r--   3 userid supergroup        243 2015-10-22 12:58 /user/userid/hbasecompact/scripts/hbaseCompact.sh
-rw-r--r--   3 userid supergroup         65 2015-10-22 12:58 /user/userid/hbasecompact/scripts/logOozieId.sh
drwxr-xr-x   - userid supergroup          0 2015-10-22 12:58 /user/userid/hbasecompact/workflow
-rw-r--r--   3 userid supergroup       1751 2015-10-22 12:58 /user/userid/hbasecompact/workflow/workflow.xml
{% endcodeblock %}
If the execute permissions in the two shell scripts are not set, do a chmod to set the executable permission.
{% codeblock %}
hdfs dfs -chmod 755 /user/userid/hbasecompact/scripts/hbaseCompact.sh
hdfs dfs -chmod 755 /user/userid/hbasecompact/scripts/logOozieId.sh
{% endcodeblock %}
Schedule the Oozie job using the following command. Note that the properties file in this example /home/userid/hbasecompact/coordinator/coordinator.properties should be on the local disk from where this command is executed.
{% codeblock %}
~$ oozie job -oozie http://oozie-host:11000/oozie -config /home/userid/hbasecompact/coordinator/coordinator.properties -run
job: 0000000-150821155141136-oozie-oozi-C
{% endcodeblock %}
The status of the job submitted can be verified through the Oozie UI which should be normally accessible through *http://oozie-host:11000/oozie/*. Also status of MapRed jobs can be viewed through the YARN RM/NM URLS.
Also the id of the last Oozie workflow which was executed and the output from the HBase major_compact command which was executed in the last run can be found from the HDFS files which got created by the job.
{% codeblock %}
~$ hdfs dfs -cat /user/userid/compact/majorcompact.log
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.98.4.2.2.8.0-2928-hadoop2, r87e9f77a121be2dae41c9ef8964d254fdc4c23a3, Fri Aug 21 13:29:40 PDT 2015

major_compact "t"
0 row(s) in 5.0410 seconds


~$ hdfs dfs -cat /user/userid/compact/oozieId.log
0000009-150821155141136-oozie-oozi-W
{% endcodeblock %}
