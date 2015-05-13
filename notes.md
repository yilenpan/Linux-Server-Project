# Udacity Project 5, Linux Server Configuration

-Note: grader's password is 'password' without the quotes. 

- Create new user called grader, give sudo, give ssh access

`adduser grader`

`adduser grader sudo`

`sudo su - grader`

`mkdir .ssh`

`sudo chmod 700 .ssh`

`sudo touch .ssh/authorized_keys`

`sudo chmod 600 .ssh/authorized_keys`


copy over the root's authorized_keys to grader's authorized keys
Log out of the ssh shell
log back in using grader's account. You can use the same rsa keys.


- Remove root from SSH access, change port to 2200

`sudo vim /etc/ssh/sshd_config'

Find line `PermitRootLogin` and set it to `no`
Find line `Port` and set it to 2200


- Update packages, install goaccess and fail2ban

`sudo apt-get update`

`sudo apt-get install cron-apt goaccess fail2ban`

goaccess - allows you to watch incoming requests by binding to apache2 access log

cron-apt - allows you to auto apt-get update

fail2ban - adds ip to ban list if multiple failed authentication attempts > 3

- set firewall by configuring iptables

See current settings by typing

`sudo iptables -L`

Allow established sessions

`sudo iptables -A INPUT -m conntract --ctstate ESTABLISHED,RELATED -j ACCEPT`

Allow certain ports


`sudo iptables -A INPUT -p tcp --dport 2200 -j ACCEPT`

`sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`

`sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT`

`sudo iptables -A INPUT -p tcp --dport 123 -j ACCEPT`


Port 53 is domain name service - need it for apt-get

`sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT`

Drop any connection to any port not listed here


`sudo iptables -A INPUT -j DROP`

- Install and configure apache to run python mod_wsgi app

FIrst we need to install apache, the modwsgi lib for apache and python-dev


`sudo apt-get install apache2 libapache2-mod-wsgi python-dev`

enable mod_wsgi by running


`sudo a2enmod wsgi`

Now we jsut need to test to see if we can get flask up and running. we're going to make a dir for the flask app in the /var/www folder

We are going to make a virutal enviroment where the app lives, from what I understand, when a request is made, apache will connect it to the wsgi app running in a virtualized enviroment, and act as a runner between the client and the app. this ensures that everything is separated, and no nonsense like directory traversals can be made. But iono. I didn't read it in detail.


`cd /var/www`
`sudo mkdir Catalog` -> holds the wsgi file
`cd Catalog`
`sudo mkdir Catalog` -> holds the app and virtual env
`cd Catalog; sudo mkdir static templates`
`sudo touch __init__.py`

In the __init__.py file, create a small flask app the routes '/' to a func that returns 'Hello World'

Now we need to install virtualenv and flask. for that we need pip

`sudo apt-get install python-pip`
`sudo pip install virtualenv`

name your virtual envirmoment, here i've named it venv

`sudo virtualenv venv`

Activate the virtual env

`source venv/bin/activate`

Install flask into your virtual enviroment

`sudo pip install Flask`

check to see if it works by running the __init__.py file

`sudo python __init__.py`

if it works, deactivate the enviroment

`deactivate`

COnfigure configure apache2 so that any request made is directed to the WSGI script and your python app.

You can find instructions for that at www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

see site step 4

finally create a wsgi file that connects the request to the flask app

see site step 5

then restart apache

- Configure PSQL

download and install postgresql

`sudo apt-get postgresql`

change user to postgres then create a new user with password

`sudo -i -u postgres`
`sudo createuser -P -S catalog --interactive`

create database with the name of your choosing. I used sportsitems

`psql`
`create database sportsitems`
`\q`

- Download git, clone your repo

get git

`sudo apt-get install git`

clone your repo

`sudo git clone http://github.com/your/repo.git`

install all the of dependencies for your python app, including the ones needed to run psql.

`sudo pip install sqlalchemy oauth2client requests psycopg2`

populate your db with your data

`python database_setup.py`
`python db_setup.py

restart apache

`sudo service apache2 restart`

run goaccess to check your error logs

`sudo goaccess -f /var/log/apache2/error.log -a`

CHeck your site and debug till it works!

- configuring fail2ban

By default, fail2ban runs on ssh port 22

to activated it we need to tke the default jail.conf file and cp it to a fail.local file

`sudo cp etc/fail2ban/jail.conf /etc/fail2ban/jail.local`

in the file, there are references to fail2ban-ssh, change all of those port values to 2200.

BOOM. You're done.
