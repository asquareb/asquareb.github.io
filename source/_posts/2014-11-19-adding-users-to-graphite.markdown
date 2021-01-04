---
layout: post
title: "Adding users to Graphite"
date: 2014-11-19 14:42:56 -0500
author: Biju Nair
comments: true
categories: [monitoring, graphite, infrastructure-mgmt]
---
Once you start populating data to Graphite database, users can see data on Graphite UI and create custom graphs to see the data in realtime. When graphs are created users want to save the graphs for future use and probably want to create a dashboard with all the graphs created. One simple (but non user friendly) way to accomplish this is to save the URLs of the graphs created and users can use it to bring up the graphs when they need them. 
<!-- more -->
Another option is to use the graph save option in Graphite web app UI which also provides an option to create and save dashboards. Both these options require logging onto web app which requires an user-id/password to log on to the web app. Here is the way to go about creating one.

- Logon to the server running Graphite web app
- `` cd `` to `` /opt/graphite/webapp/graphite/ `` (or the install directory)
- Execute the python script `` manage.py `` with the option `` createsuperuser ``

```
  python manage.py createsuperuser
  Username (Leave blank to use 'root'): test
  E-mail address: test@happyday.com
  Password:
  Password (again):
  Superuser created successfully.
```
  
- `` admin `` user is defined during installation and if you don't know the password (for admin or any user) run

```
  python manage.py changepassword
```
  
- Once you have the user-id/password set the user can log on to the web app and create graphs/dahsboards and save them for future use.
