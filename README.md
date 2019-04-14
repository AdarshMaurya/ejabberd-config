# Ejabberd on Linux https://docs.ejabberd.im/admin/installation/#install-on-linux :

Ejabberd is a powerful and popular open source XMPP server. Ejabberd has been designed from the ground-up with fault-tolerance, easy configurations, and scalability. It is able to utilize resources of multiple clustered machines and can be easily scaled when more capacity is required – By adding more VMs.

Ejabberd enables authenticating users using external or internal databases (Mnesia, SQL), LDAP or external scripts

For storing persistent data, ejabberd uses Mnesia (the distributed internal Erlang database), but you can opt for other storage:

SQL databases like MySQL or PostgreSQL
NoSQL databases like Riak (also written in Erlang)
Features of Ejabberd XMPP Server
Ejabberd have a modular architecture which allows for high customisability and easy access to the required features, which includes:

Store-and-forward (offline messages)
Contact list (roster) and presence
One-to-one messaging
User presence extension: Personal Event Protocol (PEP) and typing indicator
User profile with vCards
Group chat: MUC (Multi-User Chat)
Messaging archiving with Message Archive Management (MAM)
Message Delivery Receipts (aka XEP-184)
Privacy settings, through privacy list and simple blocking extensions
Last activity
Metrics and full command-line administration
Full feature web support, with BOSH and web sockets
Stream management for message reliability on mobile (aka XEP-0198)
and many many more.

## Install Ejabberd in Linux/Ubuntu 

How to install Ejabberd XMPP Server on Ubuntu or Linux
We’re going to install ejabberd from a binary Installer which provides a full-featured ejabberd server without a need for any extra dependencies.

## Step 1: Download ejabberd installer.
- Ports which need to be open ```5280,4560,5299```
- SSH into your machine and install the required dependencies:
  ```terminal
  apt-get update
  ```
  ```terminal
  apt-get install make gcc libexpat1-dev libyaml-dev  automake libssl-dev erlang  build-essential libncurses5-dev openssl zlib1g-dev libgd-dev libwebp-dev fop xsltproc unixodbc-dev -y
  ```
- Go to the ejabberd official download page. Note the latest version of the software.
  Export latest version to a variable
``` terminal 
cd /opt
wget -O ejab.tgz https://www.process-one.net/downloads/downloads-action.php?file=/ejabberd/19.02/ejabberd-19.02.tgz
tar -xvzf ejab.tgz
cd ejabberd-19.02

## Step 2: Configure & install ejabberd

```terminal
./configure --enable-mysql
make
make install
```

- A system user called ‘ejabberd‘ is created
- ejabberd application directory is /opt/ejabberd. This is a home for the ejabberd user.
- When ejabberd is started, the processes that are started in the system are `beam` or `beam.smp`, and also `epmd`.

## Step 3: Prepare the database for ejabberd
- login to the database server and create a new user with all access to database:
```sql
GRANT ALL ON ejabberd.* TO 'ejabberd'@'%' IDENTIFIED BY 'password';
CREATE DATABASE ejabberd;
flush privileges;
```
- Download the schema
```terminal
wget    https://raw.githubusercontent.com/processone/ejabberd/master/sql/mysql.sql
```
- Install the schema in DB
```sql
mysql -h localhost/IP_Address -D ejabberd -u ejabberd -p < mysql.sql
```
- Edit the config file /usr/local/etc/ejabberd/ejabberd.yml to make necessary changes:
Change the authentication method:
```yml
auth_method: sql
Use mysql for all the modules:

default_db: sql

sql_type: mysql
sql_server: "IP address or RDS endpoint or localhost"
sql_database: "ejabberd"
sql_username: "ejabberd"
sql_password: "password"
sql_port: 3306
```
## Step 4: Starting ejabberd service
- Check by running `ejabberdctl live`
- Since Linux use systemd init system, we need to copy ejabberd.service  to /etc/systemd/system directory.

```terminal 
sudo updatedb
sudo cp $(locate ejabberd.service) /etc/systemd/system
Reload systemd
```
```terminal
sudo systemctl daemon-reload
Start the service and enable it to start on boot
```
```terminal
sudo systemctl enable --now ejabberd
```

- Check status by running:
```terminal
systemctl status ejabberd.service
```
- Stop ejabberd
```
sudo systemctl stop epmd
```

## Step 5: Create ejabberd XMPP admin account.
- You need an XMPP account and grant him administrative privileges to enter the ejabberd Web Admin. Register an XMPP account on your ejabberd server.
- Add ejabberdctl command location to your PATH
- Locate ejabberdctl.
```terminal
$ locate ejabberdctl | grep bin
/opt/ejabberd-19.02/bin/ejabberdctl
```

- Add the path of to your .bashrc file.
```terminal
$ vim ~/.bashrc
```
- Set like below – but replace /opt/ejabberd-19.02/bin/ with your version path as found by locate
```terminal
$ PATH=$PATH:/opt/ejabberd-19.02/bin/
```
- Source the file for the new path to be reflected
```terminal
$ source ~/.bashrc
```
- Then add user
```terminal
$ ejabberdctl register admin localhost password
User admin@localhostsuccessfully registered
```
localhost should be replaced with correct server hostname.

- Edit the ejabberd configuration file to give administration rights to the XMPP account you created.
```terminal
$ sudo vim /opt/ejabberd/conf/ejabberd.yml
```
```yml
acl:
  admin:
    user:
      - "admin": "example.com"
access:
  configure:
    admin: allow
 ```
You can grant administrative privileges to many XMPP accounts, and also to accounts in other XMPP servers.

## Step 5: Access ejabberd Web Admin
The Web Admin should be accessible on  http://ip-address:5280/admin/. Open the URL usng your favorite browser.
