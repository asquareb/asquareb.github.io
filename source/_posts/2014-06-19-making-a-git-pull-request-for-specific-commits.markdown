---
layout: post
title: "Making a git pull request for specific commits"
date: 2014-06-19 18:12:46 -0400
author: Biju Nair
comments: true
categories: [git, scm]
---
Time to time when working on a project using a fork from a git repository, situations arise that a pull request need to be made to the master 
repository for a sub set of commits you made to the fork. The following are a set of steps which can help with accomplishing these scenarios.
<!-- more -->
- On the git repository in your local machine created from your fork, create a new branch

{% codeblock %}
git branch cherry-branch origin/master
{% endcodeblock %}

- The assumption is that there is only one branch in your fork and the master repository and origin is pointing to your github fork.

- Switch to the new branch created
{% codeblock %}
git checkout cherry-branch 
{% endcodeblock %}

- You can verify that you are on the correct branch by running the command
{% codeblock %}
git branch
{% endcodeblock %}

- Identify ids of the commits you want to include into the pull request from your github fork.

- Issue a git cherry-pick command to include the commits into the new branch
{% codeblock %}
git cherry-pick COMMIT-ID
{% endcodeblock %}

- Push the new branch to your fork
{% codeblock %}
git push origin cherry-branch
{% endcodeblock %}

- On github you can now see the new branch in your fork with only the commits selected. Now a pull request to the master repository.

What if there are multiple branches on the master repository and your fork and you want to send a pull request from a particular branch of your fork 
to a branch in the master repository.

- Create a clone of the master repository branch you are interested in
{% codeblock %}
git clone https://github.com/master-owner/project-repo -b interested-branch
{% endcodeblock %}

- Create a remote reference to the local repository of the your fork where all the commits are
{% codeblock %}
git remote add -t local-branch-name mylocalrepo /location/of/local/gitrepo
{% endcodeblock %}

- Do a fetch of all the changes available in the local repository from your fork
{% codeblock %}
git fetch mylocalrepo
{% endcodeblock %}

- Identify ids of all the commits you want to include into the pull request from your github fork.

- Issue a git cherry-pick command to include the commits you want to get pulled
{% codeblock %}
git cherry-pick COMMIT-ID
{% endcodeblock %}

- Create a branch
{% codeblock %}
git branch cherry-branch
{% endcodeblock %}

- Switch to the new branch
{% codeblock %}
git checkout cherry-branch
{% endcodeblock %}

- Push the changes into your fork in github
{% codeblock %}
git push https://github.com/yourid/project-repo cherry-branch
{% endcodeblock %}

- Now the branch "cherry-branch" will be show up in github UI with the changes selected. A new pull request can be created for the changes to be committed into the master repository.

- Once the changes are merged, the branch can be deleted and the local repository created can be removed.

{% codeblock %}
git push origin :cherry-branch
{% endcodeblock %} 
