# EC2-Setup

### Install necessary packages

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install libcurl4-gnutls-dev make gcc gfortran g++
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
sudo apt-get install r-base
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
sudo apt-get install apache2
sudo apt-get install apache2
sudo apt-get install libapache2-mod-proxy-html
sudo apt-get install libxml2-dev
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


### Install GIT
```
sudo apt-get update
sudo apt-get install git
```

### Gitlab Config
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md#configuring-the-external-url-for-gitlab

### Gitlab Apache Config
http://jasonrichardsmith.org/blog/gitlab-apache-ubuntu
