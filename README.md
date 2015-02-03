# EC2-Setup

### Install necessary packages

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y libcurl4-gnutls-dev make gcc gfortran g++
```

### Base-R Installation
Add RStudio repo to sources list
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
sudo sh -c 'echo "\n# RStudio Source\ndeb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list'
```

Add Keyserver
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
```

Install R
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y r-base
```

### RRO Installation
Check newest link for the RRO Packes under http://mran.revolutionanalytics.com/download/

Download and unzip
```
cd
wget http://mran.revolutionanalytics.com/install/RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.tar.gz
tar -xzf RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.tar.gz
```

Run the install script:
```
sudo ./install.sh
rm RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.tar.gz install.sh RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.deb
```

Change the default repository for RRO, i.e., remove the automatic "Snapshot" function and use the RStudio mirror
```
cd /usr/lib64/RRO-8.0.1/R-3.1.2/lib/R/etc/
sudo cp Rprofile.site Rprofile.site.backup
sudo wget https://rawgit.com/greenore/AWS-Setup/master/Rprofile.site -O Rprofile.site
```

### RStudio
Check newest link for the RStudio Server under http://www.rstudio.com/products/rstudio/download/preview/

```
cd
wget https://s3.amazonaws.com/rstudio-dailybuilds/rstudio-server-0.99.179-amd64.deb
sudo dpkg -i rstudio-server-0.99.179-amd64.deb
rm rstudio-server-0.99.179-amd64.deb
```

### Install Apache2 Server
To enable an instance of Apache running on the same server to act as a front-end proxy to RStudio you need to use the mod_proxy. The steps for enabling this module vary across operating systems so you should consult your distribution's Apache documentation for details.

```
sudo apt-get install -y apache2 libapache2-mod-proxy-html libxml2-dev
```

Configure the Apache2 server
```
cd /etc/apache2/sites-available/
sudo wget https://rawgit.com/greenore/AWS-Setup/master/rstudio.conf -O rstudio.conf
```

Then, to update the Apache configuration files to activate mod_proxy you execute the following commands:
```
sudo a2enmod proxy
sudo a2enmod proxy_http
```

Enable the host and test Apache configuration.
```
sudo a2ensite rstudio.conf
sudo apachectl configtest
sudo service apache2 reload
```

### Install Nginx Server

On Debian or Ubuntu a version of Nginx that supports reverse-proxying can be installed using the following command:
```
sudo apt-get install -y nginx
```

To enable an instance of Nginx running on the same server to act as a front-end proxy to RStudio you would add commands like the following to your nginx.conf file:

```
http {
  server {
    listen 80;
    
    location / {
      proxy_pass http://localhost:8787;
      proxy_redirect http://localhost:8787/ $scheme://$host/;
    }
  }
}
```

If you want to serve RStudio from a custom path (e.g. /rstudio) you would edit your nginx.conf file as shown below:

```
location /rstudio/ {
  rewrite ^/rstudio/(.*)$ /$1 break;
  proxy_pass http://localhost:8787;
  proxy_redirect http://localhost:8787/ $scheme://$host/rstudio/;
}
```

After adding these entries you'll then need to restart Nginx so that the proxy settings take effect:
```
sudo /etc/init.d/nginx restart
```

### Install GIT
```
sudo apt-get update
sudo apt-get install -y git
```

Configure GIT, i.e., define the author name to be used for all commits in the current repository. Typically, you’ll want to use the --global flag to set configuration options for the current user.

```
git config --global user.name <name>
git config --global user.email <email>
```

### Gitlab Config
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md#configuring-the-external-url-for-gitlab

### Gitlab Apache Config
http://jasonrichardsmith.org/blog/gitlab-apache-ubuntu

### Jekyll
```
sudo apt-get install -y make ruby ruby1.9.1-dev
sudo gem install -y jekyll
```
