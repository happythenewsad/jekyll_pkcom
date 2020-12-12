---
title: Running a Rails 3.1 app in production on CentOS 6 / MySQL
redirect_from:
  - /posts/running-a-rails-31-app-in-production-on-centos-6-mysql/
---

Here is a blow-by-blow list of commands I issued and errors I encountered/made while deploying a Rails 3.1 app to CentOS 6. I've found CentOS much easier to work with than Ubuntu for Ruby environments. Hope this is helpful for someone.

	$root useradd USER 
	# add USER to sudoer's file. I used this line: USER ALL=(ALL)       ALL
	$root visudo 
	$root su - USER

Install and start ssh server at login for ssh access:

	$ sudo yum install openssh-server
	$ sudo echo "/sbin/service sshd start" >> /etc/rc.d/rc.local 

Important libraries required by RVM, Ruby, and Git:

	$ sudo yum install git gcc-c++ make patch zlib-devel openssl-devel readline-devel

Install RVM as single user:

	$ bash < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer )

Install VMWare Tools through the CentOS GUI (my server happens to be a VMWare VM):

	$ rvm install 1.9.2

Install MySQL. After installation, use the 'mysql' and 'mysqladmin' commands to set up your MySQL users and databases:

	$ sudo yum install mysql-server mysql mysql-devel
	$ cd RAILS_PROJECT_DIR 

	$ rake db:migrate RAILS_ENV=production
	#=> Could not find a JavaScript runtime

Oops, I am used to OS X which comes with a JavaScript runtime. I installed Node.js for the V8 JS runtime. Go here for install instructions: http://nodejs.org. It should be as simple as ./configure ; make ; make install. The libraries below were Node dependencies:

	$ sudo yum install python libssl-dev

Then:

	$ cd RAILS_PROJECT_DIR
	$ rake db:migrate RAILS_ENV=production
	#=> success!

Install and set up Passenger:

	$ sudo yum install curl-devel httpd-devel apr-devel apr-util-devel
	$ gem install passenger
	$ passenger-install-apache2-module

Passenger will ask you to add 8 or so lines to the Apache config file (I found my config file here: /etc/httpd/conf/httpd.conf). I've included lines I used below as an example:

	LoadModule passenger_module /home/USER/.rvm/gems/ruby-1.9.2-p290/gems/passenger-3.0.9/ext/apache2/mod_passenger.so
	PassengerRoot /home/USER/.rvm/gems/ruby-1.9.2-p290/gems/passenger-3.0.9
	PassengerRuby /home/USER/.rvm/wrappers/ruby-1.9.2-p290/ruby

	...

	<VirtualHost *:80>
	   ServerName localhost
	   DocumentRoot /var/www/html/RAILS_PROJECT_DIR/public    
	  <Directory /var/www/html/RAILS_PROJECT_DIR/public>
	     AllowOverride all             
	     Options -MultiViews
	  </Directory>
	</VirtualHost>


	$ sudo apachectl -k start
	#=> httpd: Syntax error on line 202 of /etc/httpd/conf/httpd.conf: Cannot load /home/USER/.rvm/gems/ruby-1.9.2-p290/gems/passenger-3.0.9/ext/apache2/mod_passenger.so into server: /home/USER/.rvm/gems/ruby-1.9.2-p290/gems/passenger-3.0.9/ext/apache2/mod_passenger.so: cannot open shared object file: Permission denied

After googling a bit, I found that many people blamed this issue on Selinux. Selinux is enabled by default in CentOS 6. Switching Selinux off fixed this problem. I should probably learn Selinux and create my own ruleset, but I wanted to get something running today.

	$ sudo emacs /etc/selinux/config 
	# set: SELINUX=disabled
	$ su - USER # create a new login shell to be sure that Selinux options are refreshed
	$ sestatus # confirm that Selinux is disabled
	#=> SELinux status:   disabled

	$ sudo apachectl -k start
	#=> ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

Oops, need to start MySQL:

	$ service mysqld start

	$ rake db:migrate RAILS_ENV=production
	#=> success

At this point, your server should be serving your app. In my particular environment, I had add a firewall rule to iptables:

	 ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:http

You can check your firewall rules with this command:

	$ iptables -L

Depending on how you like to manage your assets in Rails 3.1, you may see this error when hitting your app:


	ActionView::Template::Error (application.css isn't precompiled)


I chose to precompile my assets:

	$ rake assets:precompile
	$ emacs RAILS_PROJECT_DIR/config/environments/production.rb #config.assets.compile = true

Feel free to add suggestions or other problems encountered in the comments! 