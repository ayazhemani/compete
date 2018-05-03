# compete
Tutorial and code to create a mooshak programming competition server

Run this block all together
sudo su
add-apt-repository ppa:webupd8team/sublime-text-3
apt-get update
apt-get upgrade
apt-get -y install xfce4 xrdp gnome-icon-theme-full xfce4-terminal sublime-text-installer rsync gcc make tcl apache2 firefox
apt-get -y install apache2-suexec-custom cups-bsd cups-pdf libxml2-utils xsltproc default-jre default-jdk g++ fpc 
echo xfce4-session >~/.xsession
sudo service xrdp restart
reboot now


RDP to your server and do this
cd /etc/apache2/mods-enabled
sudo ln -s ../mods-available/userdir.conf
sudo ln -s ../mods-available/userdir.load
sudo ln -s ../mods-available/suexec.load
sudo subl userdir.conf
INSERT THIS INSIDE <IfModule mod_userdir.c>
-------
   <Directory /home/*/public_html/cgi-bin>
        Options +ExecCGI -Includes -Indexes
        SetHandler cgi-script
	Order allow,deny
	Allow from all
    </Directory>
-------

May or may not be needed
sudo a2dismod mpm_event
sudo a2enmod mpm_prefork
sudo service apache2 restart
sudo a2enmod cgi
sudo service apache2 restart

Modify MOOSHAK tarball
Edit the install script, look for the line with
    exec useradd -c "Mooshak user" $User
and surround it with a catch
    catch { exec useradd -c "Mooshak user" $User }
Repeat for all useradd and userdel lines…

Modify source tgz by adding #include <string.h> to the top of pass.c file.


Run MOOSHAK
tar xzf mooshak-version.tgz
cd mooshak-version
sudo ./install

Enable Automatic redirects to Login Page “/cgi-bin/execute?login”
a2enmod rewrite
subl /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf

        <Directory /var/www/html>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Require all granted
        </Directory>
</VirtualHost>
subl /var/www/html/.htaccess
	RewriteEngine On
	RewriteRule ^/?(.*)$ /~mooshak/$1 [L,R=301]

Add Contest description to landing page
subl /home/mooshak/public_html/index.html
	content=”2; URL=cgi-bin/execute?login”>
	Also edit any Welcome Text here

Add Contest languages (already installed in initialization)
sudo apt-get update
sudo apt-get upgrade
sudo apt-get -y install default-jre default-jdk g++ fpc 


Language
Compiler
Version
Command line
Execute
Extension
C
gcc
5.4.0
/usr/bin/gcc -w -lm $file
a.out
.c
C++
gcc
5.4.0
/usr/bin/g++ -w $file
a.out
.cpp
Java
jdk
javac 1.8.0
/usr/bin/javac $file
/usr/bin/java $name
.java
Python
python
2.7.12
$home/contrib/python/compiler $file
/usr/bin/python $file
.py


Setup SendGrid
Open TCP port 587
sudo apt-get install mailutils postfix
subl /etc/postfix/main.cf
Change hostname as needed
Add the following:
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_tls_security_level = encrypt
header_size_limit = 4096000
relayhost = [smtp.sendgrid.net]:587
subl /etc/postfix/sasl_passwd
	Add the following:
[smtp.sendgrid.net]:587 yourSendGridUsername:yourSendGridPassword
Use apikey:<your sendgrid api key> for easy configuration
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
sudo systemctl restart postfix
/etc/init.d/postfix reload

Test with:
echo “message” | mail -s “title” your@email.com

Other docs
https://askubuntu.com/questions/325807/arrow-keys-tab-complete-not-working 
https://askubuntu.com/questions/456190/cant-find-config-folder-in-ubuntu-14-04 


