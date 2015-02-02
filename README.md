# AWS-Setup

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

Check packages
```
dpkg -l make gcc gfortran g++
```

Download and unzip the installer
```
wget http://mran.revolutionanalytics.com/install/RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.tar.gz
tar -xzf RRO-8.0.1-Beta3-Ubuntu-14.04.x86_64.tar.gz
```

Run the install script:
```
./install.sh
```

### RStudio
Check newest link for the RStudio Server under http://www.rstudio.com/products/rstudio/download/preview/

```
wget https://s3.amazonaws.com/rstudio-dailybuilds/rstudio-server-0.99.179-amd64.deb
```

### Gitlab Config
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md#configuring-the-external-url-for-gitlab

### Gitlab Apache Config
http://jasonrichardsmith.org/blog/gitlab-apache-ubuntu

