---
title: Configuring a user for restored/imported database in MS SQL Server
date: 2015-03-26T14:09:30+00:00
author: "Taras Kushnir"
permalink: /configuring-a-user-for-restoredimported-database-in-ms-sql-server/
categories:
  - Programming
tags:
  - login
  - management studio
  - security
  - sql server
  - windows
---
When you import or restore a database from bak file, you can have same problems as I had. Problems with user access and restoration of login schemes.

First of all, make sure SQL server is in Mixed Authentication mode (right click on SQL server connection and Security tab). Then create a new Login in top Security object, and memorize username and password for your connection string.

Then you can add a User in appropriate Security object of your database, also setting a Login to one you created just before. Now it's valid!
