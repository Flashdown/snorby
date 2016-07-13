# Snorby

* [github.com/Snorby/snorby](http://github.com/Snorby/snorby/)
* [github.com/Snorby/snorby/issues](http://github.com/Snorby/snorby/issues)
* [github.com/Snorby/snorby/wiki](http://github.com/Snorby/snorby/wiki)
* irc.freenode.net #snorby

## Description

Snorby is a ruby on rails web application for network security monitoring that interfaces with current popular intrusion detection systems (Snort, Suricata and Sagan). The basic fundamental concepts behind Snorby are **simplicity**, organization and power. The project goal is to create a free, open source and highly competitive application for network monitoring for both private and enterprise use.

## Requirements

* Snort
* Ruby >= 1.9.2
* Rails >= 3.0.0

## Install Debian Wheezy using an already SSL configured Apache2 (you can also use another webserver) 
* Installing required software:
`$ apt-get install imagemagick ruby git`
`$ apt-get install rails`

`$ git clone http://github.com/Flashdown/snorby.git`
* Move snorby folder to www or better make its contents to be in the www folder

* Install bundler
 `$ gem install bundler`

* Now switch to a less privileged account (make sure your user has sufficient permissions on the snorby folder I used chown www-data:www-data /var/www/snorby -R ), otherwise the application will be broken for non root users ;)

* Switch into snorby dir
* Add yourself temporary in the sudoers list (needed by bundle and should be removed after the installation is done)
`$ visudo`
Add: `www-data  ALL=(ALL:ALL) ALL`

* Install Gem Dependencies
`$ bundle install --without development`

* Enter your users password for using sudo, when asked
`$ sudo gem update --system`

* Copy Default Configs and edit them
`$ cp ./config/database.yml.example ./config/database.yml`
* Enter the DB credentials.
`$ vim ./config/database.yml`

`$ cp config/snorby_config.yml.example config/snorby_config.yml`
`$vim ./config/snorby_config.yml`

* Adjust the following lines so that they match in snorby_config.yml
```
 domain: 'host.your.domain.com'
 ssl: true
 mailer_sender: 'snorby@vm-bcr-snort'
 wkhtmltopdf: /usr/bin/wkhtmltopdf
```

* Mail configuration can be configured here, can be done later: "config/initializers/mail_config.rb"

`$ sudo gem install rails bundler passenger`
`$ sudo apt-get install libcurl4-openssl-dev apache2-threaded-dev libapr1-dev libaprutil1-dev`

`$ sudo passenger-install-apache2-module`
* Only select ruby
* Add to end of your apache configuration file /etc/apache2/apache2.conf
```
  LoadModule passenger_module /var/lib/gems/2.1.0/gems/passenger-5.0.29/buildout/apache2/mod_passenger.so
   <IfModule mod_passenger.c>
     PassengerRoot /var/lib/gems/2.1.0/gems/passenger-5.0.29
     PassengerDefaultRuby /usr/bin/ruby2.1
   </IfModule>
```
* Restart Apache2
`$ service apache2 restart`

* Now you can remove the user www-data from the sudoers list again (Run as root and remove the previous added lines for user www-data) `$ visudo`

* Run the Snorby Setup by running the following as root (adjust /var/www/snorby to thatever your webroot containing the snorby files is)
`$ su -c "cd /var/www/snorby && RAILS_ENV=production /usr/local/bin/bundle exec rake snorby:setup" -s /bin/bash www-data`

* Create Crontabs that will start Snorby and the Workers automatically each time the system has been rebooted:
`$ @reboot su -c "bundle exec rails server -e production" -s /bin/bash www-data`
`$ @reboot su -c "cd /var/www/snorby && RAILS_ENV=production /usr/local/bin/bundle exec ruby script/delayed_job start" -s /bin/bash www-data`

* Now just reboot once and the Snorby webinterface should be available on the port you have configured in your apache2 SSL site config.

* Default User Credentials

	* E-mail: **snorby@example.com**
	* Password: **snorby** 


## Fixing PDFExport not working:
Error:
/var/lib/gems/2.1.0/gems/actionpack-3.2.22/lib/action_dispatch/http/mime_type.rb:102: warning: already initialized constant Mime::PDF
/var/lib/gems/2.1.0/gems/actionpack-3.2.22/lib/action_dispatch/http/mime_type.rb:102: warning: previous definition of PDF was here

# Fixing the bug in Snorby, since newer Ruby versions already register that Mime Type:
* Change into the home directory of the user you are using for Snorby since there .bundle directory is located
* then run the following:
`$ sed -i 's/\(^.*\)\(Mime::Type.register.*application\/pdf.*$\)/\1if Mime::Type.lookup_by_extension(:pdf) != "application\/pdf"\n\1  \2\n\1end/' ./.bundle/ruby/2.1.0/ezprint-*/lib/ezprint/railtie.rb`

## Create suitable wkhtmltopdf package for Debian since the one from the repo was build using unpatched qt and therefore won't work in the way Snorby needs it to

* Downloaded and created a deb file manually: 
`$ apt-get remove --purge wkhtmltopdf`
`$ wget http://download.gna.org/wkhtmltopdf/0.12/0.12.3/wkhtmltox-0.12.3_linux-generic-amd64.tar.xz`

Then I have extracted the archive which contains the files and directorie structures, I created a new folder in it called usr and moved every folder into it so that the path will be usr/bin instead of bin and so on.
Then I created a folder called DEBIAN in the same folder as the usr folder is, there IÄve created the "control" and "conffiles" in the DEBIAN folder. I leaved the conffiles file empty so I just used touch.
for the control file and for the creation of the deb file I followed this guide: http://www.sj-vs.net/creating-a-simple-debian-deb-package-based-on-a-directory-structure/

But I also used apt-cache show wkhtmltopdf to get more and better stuff for the control file. then I have gone one directory up called wkhtmltopdf which now contain the folder usr and DEBIAN. and executed this dpkg-deb --build wkhtmltopdf
* now install your package
`$dpkg -i wkhtmltopdf.deb`


* How to open the rails console:
`$ su -c "cd /var/www/snorby && RAILS_ENV=production /usr/local/bin/bundle exec rails console" -s /bin/bash www-data`

* When configuring mail reports using the console and the following commands for testing is very useful

`irb(main):001:0> ReportMailer.daily_report.deliver`
		 `ReportMailer.weekly_report.deliver`
		 `ReportMailer.monthly_report.deliver`

## Updating Snorby

* In the root Snorby directory type the following command:
	`git pull origin master`
	
* Once the pull has competed successfully run the Snorby update rake task:
`$ su -c "cd /var/www/snorby && RAILS_ENV=production /usr/local/bin/bundle exec rake snorby:update" -s /bin/bash www-data`
	

## Install OLD (only left what may could be usefull)

* Edit The Snorby Configuration File

	`config/snorby_config.yml`
	
* Edit The Snorby Mail Configurations

	`config/initializers/mail_config.rb`
	
* Once all options have been configured and snorby is up and running

	* Make sure you start the Snorby Worker from the Administration page.
	* Make sure that both the `DailyCache` and `SensorCache` jobs are running.
	
* Default User Credentials

	* E-mail: **snorby@example.com**
	* Password: **snorby**
	
* NOTE - If you do not run Snorby with passenger (http://www.modrails.com) people remember to start rails in production mode.

	`rails -e production`
	
## Updating Snorby

In the root Snorby directory type the following command:

	`git pull origin master`
	
Once the pull has competed successfully run the Snorby update rake task:

	`rake snorby:update`
	
# Helpful Commands

You can open the rails console at anytime and interact with the Snorby environment. Below are a few helpful commands that may be useful:

 * Open the rails console by typing `rails c` in the Snorby root directory
 * You should never really need to run the below commands. They are all available within the
	Snorby interface but documented here just in case.

**Snorby Worker**

	Snorby::Worker.stop      # Stop The Snorby Worker
	Snorby::Worker.start     # Start The Snorby Worker
	Snorby::Worker.restart   # Restart The Snorby Worker

**Snorby Cache Jobs**
	
	# This will manually run the sensor cache job - pass true or false for verbose output
	Snorby::Jobs::SensorCacheJob.new(true).perform`

	# This will manually run the daily cache job - once again passing true or false for verbose output
	Snorby::Jobs::DailyCacheJob.new(true).perform

	# Clear All Snorby Cache - You must pass true to this method call for confirmation.
	Snorby::Jobs.clear_cache

	# If the Snorby worker is running this will start the cache jobs and set the run_at time for the current time.
	Snorby::Jobs.run_now!

## License

Please refer to the LICENSE file found in the root of the snorby project.


