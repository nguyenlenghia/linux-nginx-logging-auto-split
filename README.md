# linux-nginx-logging-auto-split
Split logs (nginx or all other log) automatically every day in Linux environment

# Using logrotate lib of linux. It will automatically split logs each days, months, years of split by size,...

Related resource(s):

"linuxcommand: logrotate":http://linuxcommand.org/man_pages/logrotate8.htmlX42X
logrotate is designed to ease administration of systems that generate large numbers of log files. Normally, logrotate is run as a daily cron job.

Some important knowledges:

Any number of config files may be given. Later config files may override the options given in earlier files, so the order in which the logrotate config files are listed in is important. Normally, a single config file which includes any other config files which are needed should be used. If a directory is given, every file in that directory is used as a config file.
Default config file: /etc/logrotate.conf. You can include other config files within it using include directive.
Assumption
Your site is example.com
The site is located in /var/www/example/
Your site is deployed by Capistrano, so you can find your logs in /var/www/example/shared/log/
Your static contents server is Nginx, and its logs are located in /var/www/example/shared/log/ and their names start with nginx_
How to do
1. Login your server
2. Make sure that "include /etc/logrotate.d" is existed in default config file and not commented:
```linux
$ cat /etc/logrotate.conf
```
You should be able to find the directive shown below, if not, append it manully.
```linux
include /etc/logrotate.d
```
3.Create new logrotate config files for your site's logs
A. Create new rotate config file for application log:
```linux
$ sudo vim /etc/logrotate.d/example_production_log
```
Type following contents, and save.
```logrotate config
/var/www/example/shared/log/production.log {

  daily

  missingok

  rotate 30

  notifempty

  create 664 deploy deploy

  copytruncate

}
```
B. Create new rotate config file for server log:

$ sudo vim /etc/logrotate.d/nginx-log-for-example
Type following contents, and save.

/var/www/example/shared/log/nginx_*.log {

  daily

  missingok

  rotate 30

  notifempty

  create 664 root root

  sharedscripts

  postrotate

  [ ! -f /var/run/nginx.pid ] || kill -USR1 `cat /var/run/nginx.pid`

  endscript
}
Attention:

/var/run/nginx.pid is your nginx pid file path, but someone may use default nginx pid path(/opt/nginx/logs/nginx.pid). You have two solutions to solve this conflict:

A. Change /var/run/nginx.pid to /opt/nginx/logs/nginx.pid or other path you have defined in your nginx config file.
B. Set your "pid" directive to expected path:

pid /var/run/nginx.pid;
in your nginx config file(such as, /opt/nginx/conf/nginx.conf), and then restart your server.

If the above work are all finished, everything done! You should remember to check if everything runs normally at other days.


References:
https://linux.die.net/man/8/logrotate
https://www.digitalocean.com/community/tutorials/how-to-configure-logging-and-log-rotation-in-nginx-on-an-ubuntu-vps
https://www.cambus.net/log-rotation-directly-within-nginx-configuration-file/
https://segmentfault.com/a/1190000000758485
