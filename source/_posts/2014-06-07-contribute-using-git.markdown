---
layout: post
title: "Setting up to contribute using Git"
date: 2014-06-07 10:49:22 -0400
author: Biju Nair
comments: true
categories: [git, scm, software-development]
---
Basic steps if you are planning to contribute to an existing project currently on GitHub.

- Create an account on GitHub.
- Create a fork of the project repository on GitHub.
- Create a Git repository on your local machine. This requires Git to be available on the machine
{% codeblock %}
  git clone https://github.com/your-user-name/repository-name -b branch-name
{% endcodeblock %}
<!-- more -->
- Note the -b option is used to specify a particular branch you want to work on in the project. You can verify the branch the local repository is pointing to by
{% codeblock %}
  git branch
{% endcodeblock %}
- The repository created locally by the clone points to the fork which was created by you on GitHub
- You would also want to pull in all the changes happening in the project repository like changes from other contributers
- Inorder to be able to pull the changes, set "upstream" to point to the project repository. CD to your local repository and issue the following command.
{% codeblock %}
  git remote add upstream https://github.com/project-or-user-name/repository-name
{% endcodeblock %}
- You can verify the "origin" and the "upstream" repositories your local repository is pointing to by issuing the following command
{% codeblock %}
  git remote -v
{% endcodeblock %}
- Inorder to update your local repository with any changes in the project repository, issue the following command
{% codeblock %}
  git pull upstream
{% endcodeblock %}
- Git wll perform automerge of changes where possible and if there are any conflicts they need to be resolved manually
- Make the changes you wish to make to the project repository on your local machine and once satisfied follow these steps
- Perform a pull upstream to make sure that your repository is upto date with project repository  
{% codeblock %}
  git pull upstream
{% endcodeblock %}
- Commit all your changes to the local respository. This step will ask to enter comments about the change etc.
{% codeblock %}
  git commit -a
{% endcodeblock %}
- Push the changes to GitHub which will update the fork you had created initially. This step will prompt for your GitHub user-id and password
{% codeblock %}
  git push -u branch-name
{% endcodeblock %}
- You can verify that the changes are on your fork by browsing the changed components through GitHub web interface
- Once satisfied with your changes, you can make a pull request using the pull request icon on the GitHub web interface
- If the project owner is fine with the changes, the changes will get merged into the project repository. Then the changes will be available for everyone to use.
- Few other useful commands
{% codeblock %}
  git stash
{% endcodeblock %}
  This will revert back all the changes made in the local repository
{% codeblock %}
  git rebase -i HEAD~n
{% endcodeblock %}
  Where n is a number and this will help in merging multiple commits into fewer so that the change history is much cleaner.
- This is a very quick summary of steps. Git documents provide details, options and other useful commands which you may want to look at 
