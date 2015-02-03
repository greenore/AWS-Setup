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
```
sudo apt-get install apache2
sudo a2enmod proxy_http
```

Configure the Apache2 server
```
cd /etc/apache2/sites-available/
sudo wget https://rawgit.com/greenore/AWS-Setup/master/rstudio.conf -O rstudio.conf
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

### Connecting with SSH without a PEM key

Navigate to the .ssh folder. If this folder doesn’t exist use mkdir to make it.
```
cd
cd .ssh
```

Once in your ssh folder on your local machine which should be in /Users/yourusername/.ssh generate your key by executing the following.

```
ssh-keygen -t rsa -b 1024
```
When prompted enter the file name to save the key enter *id_rsa_aws*, when prompted to enter a password leave blank.
In your .ssh directory execute the following command and copy the output to paste later.

In your .ssh directory execute the following command and copy the output to paste later.
```
cat id_rsa_aws.pub
```

Now connect to you AWS instance using you PEM key

```
ssh -i path/to/yourkeyname.pem ubuntu@54.213.166.232
```

Once in

echo 'the key you copied from id_rsa_aws.pub' >> .ssh/authorized_keys
chmod 640 .ssh/authorized_keys
chmod 750 .ssh

You should now be able to connect to your webserver using ssh and the path to your id_rsa_aws in the same way as you were using your pem key.

However to make it even simpler to connect to your web server we will take an extra step. Go back to your local ssh folder and type the following.

vi config

Once in vi (a file editor) type the following. To start press ‘o’ then to write and exit the file type ‘wq’.

Host webserver
Hostname 54.213.166.232
User ubuntu
IdentityFile ~/.ssh/id_rsa_aws

You should be able to to connect to your server using ssh webserver or another name you chose for your Host if it suits better.

If you get permissions errors set your .ssh folder to 750 and your id_rsa_aws.pub to 600

Setup GIT for web deployment and version control

Once in your server install git with the following command.

sudo apt-get install git

This part of the tutorial has been adapted from http://danbarber.me/using-git-for-deployment/ if you want more in depth instructions check it out.

Firstly we will initialize the server with two git repositories the first will act as a centralized hub with a bare repository. The other will reside in the code base which will contain the live code.

We will store the live git repo in awsproject, you can change the name if that suits your needs better.

First off ssh to your server and follow the commands wehn you create index .html

ssh webserver
cd /var/www/
sudo mkdir awsproject
cd awsproject/
sudo vi index.html

In the index.html file start typing with ‘o’ and ‘wq’ to write and quit vi. Type a message to indicate the site is up and running “Site up and running powered by amazon web services and git!”

After you have save your index.html in awsproject initialize a git repository add the files within and commit it using the commands below.

sudo git init
sudo git add .
sudo git commit -m "initial live site commit"

Next we need to create a bare repository to act as a mediator between the live code and the local code. We will place this in /var/git/

sudo mkdir -p /var/git/awsproject.git
cd /var/git/awsproject.git
sudo git init --bare
cd /var/www/awsproject
sudo git push /var/git/awsproject.git master

Update the config of the live repo to be a remote of the bare repo by editing the file at /var/www/awsproject/.git/config

[remote "hub"]
url = /var/git/awsproject.git
fetch = +refs/heads/*:refs/remotes/hub/*

After that we need to set up some hooks to push updates from the bare repo to the live repo

Create/edit the file at /var/git/awsproject.git/hooks/post-update with the following

#!/bin/sh
echo
echo "**** Pulling changes into Live [Hub's post-update hook]"
echo
cd /var/www/awsproject || exit
unset GIT_DIR
git pull hub master
exec git-update-server-info

Additionally if we decide to make changes on the live server for some reason we need to be able to push these changes to the bare repo. Create/edit the file at /var/www/awsproject/.git/hooks/post-commit with the following

#!/bin/sh
echo
echo "**** pushing changes to Hub [Live's post-commit hook]"
echo
git push hub

Both of these files need to be executable so execute the following commands

sudo chmod +x /var/git/awsproject.git/hooks/post-update
sudo chmod +x /var/www/awsproject/.git/hooks/post-commit

In order for the ubuntu user to have permissions to push the changes to the bare repo and live repo we need to change the owner on the following folders with the following commands.

sudo chown -R ubuntu /var/git/
sudo chown -R ubuntu /var/www/

Once that is done navigate to a directory on your local computer to clone your local repo too.

cd ~/Dropbox/Websites/

For small sites I store mine in my Dropbox folder so the projects can sync between different work machines, then I can update bits and pieces without any hassle and then just push the changes when on a machine with git installed.

git clone webserver:/var/git/awsproject.git awsproject

Once You have successfully cloned your repo to your computer you should be able to update your index.html file e.g. append it with ‘updated locally’.

Now in your local repository on your computer you should now be able to execute the following and see your updates on the server.

git add .
git commit -m "initial commit on local machine"
git push origin master

In order to prevent people from being able to access the contents of your git repositories we will need to update the file at

/etc/apache2/sites-available/default

The following AllowOverride FileInfo in the first part allows for .htaccess override which your site will probably need. The second section forbids access to directories with .git in their name.

<Directory /var/www/>
Options Indexes FollowSymLinks MultiViews
AllowOverride FileInfo
Order allow,deny
allow from all
</Directory>
<Directorymatch "^/.*/\.git/">
Order deny,allow
Deny from all
</Directorymatch>

If you do plan on using .htaccess files on your server you will need to enable mod_rewrite with the following commands on your server.

sudo a2enmod rewrite
sudo service apache2 restart









Once in your ssh folder on your local machine which should be in /Users/yourusername/.ssh generate your key by executing the following.


Git can be server-less you init your repository and then you access it from remote via SSH. So instructions like this on the Ubuntu Server should do it:

Setting Up the Server
Let’s walk through setting up SSH access on the server side. In this example, you’ll use the authorized_keys method for authenticating your users. We also assume you’re running a standard Linux distribution like Ubuntu. First, you create a git user and a .ssh directory for that user.

```
sudo adduser git
su git
cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```



### Gitlab Config
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md#configuring-the-external-url-for-gitlab

### Gitlab Apache Config
http://jasonrichardsmith.org/blog/gitlab-apache-ubuntu
