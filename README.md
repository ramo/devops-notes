# DevOps Notes

## Linux Commands

- `ls /etc/*release*` - Check OS version.
- `curl <url> -O` or `wget <url> -O <filename>` - Download files.

## Package Managers

### RPM

- `rpm -i <rpm>` - Install package.
- `rpm -e <rpm>` - Uninstall package.
- `rpm -q <rpm>` - Query package.
- Dependencies are not installed for the package.

### YUM

- Highlevel package manager, takes care of dependencies.
- Underlying uses `rpm`.
- `/etc/yum.repos.d/` - contains information about yum repositories.
- `yum repolist` - List the yum repos.
- `yum list ansible` - List installed/available packages.
- `yum remove ansible` - Removes the package.
- `yum --showduplicates list ansible` - Show all versions.
- `yum install ansible-2.4.2.0` - Install specific version of the package.

## Services

Help softwares to run as background process all the time and after server restart. And also follows right order of startup.

- `systemctl start httpd` - Start httpd service.
- `systemctl stop httpd` - Stop httpd Service.
- `systemctl status httpd` - Check httpd service status.
- `systemctl enable httpd` - Configure httpd to start at startup.
- `systemctl disable httpd` - Confgure httpd to not start at startup.

### Configure a service

- Using systemd unit file in `/etc/systemd/system` directory.

```
# my_app.service
[Unit]
Description=My python web application

[Service]
ExecStart=/usr/bin/python3 /opt/code/my_app
ExecStartPre=pre_script.sh
ExecStartPost=post_script.sh
Restart=always # Restart automatically if app crashes.

[Install]
WantedBy=multi-user.target # required for service to run at startup.
```

- Run `systemctl daemon-reload` to read this new service file.

## Virtual Box

- Virtualization softwares or Hypervisors are used to create virtual machines.
- Type-1 hypervisors are installed directly on the baremetal servers. Eg. Vmware ESXi, Microsoft Hyper-v.
- Type-2 hypervisors are installed on top of OS. Eg. Oracle VirtualBox, Vmware workstation.
- https://www.osboxes.org is a site where we can get virtual box images.

### Virtual Box Connectivity

- Though VMs are running on our machine, we should think that they are separate machines connected to the same network.
- VMs configured with IP addresses and relevant services should be running on the VMs for connectivity (SSH, RDP, etc.,)
- Use console to configure the connectivity and then use remote services to login to the machine.
- `ip addr show` command is used to check the IP address in CentOS.
- `ip addr add 192.168.1.10/24 dev eth0` command to set the IP address.
- `service sshd status` to check the status of the SSH service.
- `service sshd start` to start the SSH service.

### IP Address

- A machine has multiple network intefaces/adapters, they each have different IP addresses.
- Based on the network, the interfaces have internet or not.

### VirtualBox Networking

- There are 4 network adapters are available for Virtual box vms.
- Default network adapter is enabled with **NAT**

|                                                    | NAT                | NAT Network        | Host Network       | Host Network + NAT | Bridge             |
| -------------------------------------------------- | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ |
| VM can reach internet/other systems in the network | :heavy_check_mark: | :heavy_check_mark: | :x:                | :heavy_check_mark: | :heavy_check_mark: |
| VMs can reach each other                           | :x:                | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Host can reach VM (Without Port forwarding)        | :x:                | :x:                | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Other systems in network can reach VM              | :x:                | :x:                | :x:                | :x:                | :heavy_check_mark: |

#### HostOnly Network

![Host only Network](images/host-networks.png?raw=true 'Host only Network')

- Using HostNetworkManager create new hostonly network adapter. This step creates a virtual adapater in the host machine and IP address will be assigned for host.
- In creation of Guest VM machines, this same hostonly network adapter is selected, thus the communication between these VMs are enabled.

#### NAT network

- Similar to host only network, but in addition to the communication between the VMs, they have access to outside network (like internet).
- File > Preferences > Network - Create NAT network.
- In NAT network, no virtual network adapter is created in the host machine unlike host only network.
- Machines from outside network can't establish connection to these VMs.

#### NAT

- VMs can access outside network similar to NAT network. But VMs can't access each other.

#### Bridge

- Unlike host only network or NAT network, bridge network doesn't need to be created. It is always there, VMs just need to connect to it.
- Connecting to this adapter, VMs get IP address in the same network of the host machine.

### Port Forwarding

- Forwarding port from host machine to guest machine.
- Configurations are done in VM's network settings.

## Vagrant

- Vagrant is a tool for building and managing virtual machine development environments in a single workflow.
- Boxes are the package format for Vagrant environments. They are available in [public Vagrant box catalog](https://app.vagrantup.com/boxes/search).
- `vagrant init centos/7` - Initialize a Vagrantfile in the current directory.
- `vagrant up` - Creates the environment.
- `vagrant reload` - After changing the configuration in Vagrantfile, reload is used.
- `vagrant halt` - For stopping the VM.
- `vagrant ssh` - To SSH to the VM. Uses VM portforwarding and SSH based keys for auth.

### Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network "forwarded_port", guest: 80, host:8080
  config.vm.synced_folder "../data", "vagrant_data"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
  config.vm.provision "shell", inline: <<-SHELL
    yum update
    yum install -y httpd
  SHELL
end
```

## Vagrant Providers

- Virtualbox
- VMware
- Hyper-V
- Docker
- Custom

## Networking Basics

### Switching

- A switch connects machines to same network and enable communication between the machines.
- `ip link` command used to identify the network interface in the machine.

### Routing

- Router helps to connect two network together.

### Gateway

- `route` or `ip route` command is used to check the IP routing table in the machine.
- `ip route add 192.168.2.0/24 via 192.168.1.1` is used to add a Gateway entry in routing table.
- `ip route add default via 192.168.2.1` - Adding default gateway route entry. Helps to reach any IP out of current network.
- `0.0.0.0` can be used instead of `default`.

| Note                                                                                                           |
| :------------------------------------------------------------------------------------------------------------- |
| When facing internet related issues, it is best to start checking the route table and Default gateway entries. |

![Network routing](images/network-routing.png?raw=true 'Network Routing')

```
# To ping from machine A to C vice-versa.

# In machine A
ip route add 192.168.3.0/24 via 192.168.2.4

# In machine C
ip route add 192.168.2.0/24 via 192.168.3.3

# In machine B
echo 1 > /proc/sys/net/ipv4/ip_forward
# To make the changes persist.
# /etc/sysctl.conf
# net.ipv4.ip_forward = 1
```

## DNS

- `/etc/hosts` file is used for name resolution (Name to IP).
- DNS server is a centralized way of name resolution instead of adding entries to /etc/hosts file of all machines on the network.
- `/etc/resolv.conf` file has DNS `nameserver` entry.
- By default `/etc/hosts` file takes priority in name resolution. This is decided based on `/etc/nsswitch.conf` file. (`hosts: files dns`)
- Unknown DNS requests can be forwarded from corporate DNS server to global DNS servers like `8.8.8.8`.
- `nslookup` and `dig` commands are used to test DNS name resolution. These directly query the nameserver and doesn't care about the entries in the `/etc/hosts` file.

### Domain Names

- `.` - Root
- `.com` - Top Level Domain Name
- `google` - Domain name
- `mail`, `maps` - Subdomain.

### Search Domain

- `search` directive in `/etc/resolv.conf` file is uesd for providing search domains.
- Eg. `search mycompany.com` entry would help resolve `ping web` query to `ping web.mycompany.com`

### Record Types

| Type  |                 |                                        |
| ----- | --------------- | -------------------------------------- |
| A     | web-server      | 192.168.1.1                            |
| AAAA  | web-server      | 2001:db8:3333:4444:5555:6666:7777:8888 |
| CNAME | food.web-server | eat.web-server                         |

## Application Basics
### Java
- Free, Open-Source, Huge community
- Java versions <=8 uses version name convention like 1.8, 1.7, etc. From 9 onwards this got changed. 
- `JDK` - Java Development Kit. `JRE` - Java Runtime Environment.
- Before version 9, the JDK and JRE can be installed separately. Now JRE is part of the JDK.
- Develop - `jdb` `javadoc`
- Build - `javac` `jar`
- Run - `java`
- JDK under GPL license available from Oracle https://jdk.java.net/archive/

#### Java - Build & Packaging
- Develop Source Code > Compile `javac MyClass.java` > Run `java MyClass`
- Java source code is compiled by `javac` to produce byte code which runs on JVM (Java Virtual Machine).
- Develop once and run anywhere. 
- Package - `JAR` (Java Archive), `WAR` (Web Archive).
- In a Jar file, `META-INF/MANIFEST.MF` file has information about the entrypoint (Main-Class) of the application. 
- `jar cf MyApp.jar MyClass.class Service1.class Service2.class ...`
- `java -jar MyApp.jar`
- Documentaion - `javadoc -d doc MyClass.java`
- Build Process - Develop, Compile, Package, Document.
- Build Tools - `Maven`, `Gradle`, `Ant` - Helps manage the complexities in the build process.

### NodeJS
- Server side Javascript framework.
- Supports many concurrent connections using non blocking I/O model.
- Free, OpenSource and Cross Platform Compatible.
- `node -v` to see the version of the nodeJS.
- `node add.js` to run the javascript program.

#### NPM (Node Package Manager)
- Reusable node packages available in the NPM repository.
- `npm -v`
- `npm search file` - Search for a NPM package.
- `npm install file` - Install a NPM package in current directory.
- `package.json` - Every node module contains this file to show the dependencies of that module.
- `node -e "console.log(module.paths)"` - List the modules path in which node looks for NPM packages.
- `npm install file -g` - Install the package globally.
- Built-In Modules `/usr/lib/node_modules/npm/node_modules`
  - `fs` - To handle filesystem
  - `http` - To host an HTTP server
  - `os` - To work with the Operating System.
  - `events` - To handle events.
  - `tls` - To implement TLS and SSL.
  - `url` - To parse URL strings.
- External Modules `/usr/lib/node_modules`
  - `express` - Fast, Unopinionated, minimalist web framework.
  - `react` - To create user interafaces.
  - `debug` - To debug applications.
  - `async` - To work with asynchronous JS.
  - `lodash` - To work with arrays, objects, strings etc.


### Python
- Free, Open source and Cross platform compatible.
- Python2 - (2000 - 2010), Python3 - (2008 to present).
- `python2` invokes the python2 interpretor.
- `python3` invokes the python3 interpretor.

#### pip (Python Package Manager)
- `pip2` and `pip3` corresponding to python2 and python3.
- `pip install flask` - Install a package.
- `/usr/lib/python{x.y}/site-packages` & `/usr/lib64/python{x.y}/site-packages`
- `pip show flask` - shows the package location.
- `python -c "import sys; print(sys.path)"` - Path where python looks for the packages.
- `pip install -r requirements.txt` 
- `pip install flask --upgrade` - upgrade a package.
- `pip uninstall flask` - Uninstall a package.

#### Other Package Managers
- easy_install
  - `app.py` ->(setuptools) -> `app.egg` - `easy_install install app`
- wheel
  - `app.py` ->(setuptools) -> `app.whl` - `pip install app.whl`


## Source Control Management (SCM)
### GIT
- What changes were made by whom and when?
- version control system.
- `git init` - Initialize a GIT repository. `.git` directory is created.
- `git status` - Checks the status of GIT repository.
- `git add <List of files>` - configure files to track, Stage changes.
- `git commit -m "Commit message"` - Commits the changes to Local GIT repository.

### GIT - Remote Repositories
- A central place to push/pull changes from local GIT repositories.
- `GitHub` `GitLab` are example remote GIT repositories. It is possible to run our own GIT remote server as well.
- Before pushing the changes to remote repositories, it needs to be created. GitHub provides web interface to do that.
- `git remote add <name> <url>` - Adding remote repository to local. Name could be anything. 
- `git push -u <name> <branch>` - Pushing the local changes to remote repository. `-u` is required when pushing for the first time after adding the remote using `git remote add`
- `git clone <repo url>` - First time cloning the remote repository to local. For this local repo the remote will be added automatically. 
- `git remote -v` - Command to check the remote of the repository.
- By default, `origin` is the remote name of the remote repository. It represents `original`.
- `git pull` - Command to pull the latest changes from remote repository to local.

| Note |
|:-----|
| Bare repository is used for creating remote repositories locally. `--bare` flag is used with either `init` or `clone` command to create the bare repository.|

## Web Servers
- Backend web applications - Java, NodeJS, Python.
- FrondEnd application - HTML, Javascript, CSS.
- Web servers can host multiple applications at same time.
- Static websites - Only uses static content.
- Dynamic websites - Connects with DB, etc and provide dynamic content to users. 
- Static web servers refer as web servers - Nginx, Apache.
- Dynamic web servers refer as application servers - Tomcat, Gunicorn, uWSGI.

### Apache Web server
- OpenSource web server.
- `yum install httpd` - Install httpd
- `service httpd start` - Start the httpd service.
- `service httpd status` - Check the status of the httpd service.
- `firewall-cmd --permanent --add-service=http`
- Log files - `/var/log/httpd/access_log` and `/var/log/httpd/error_log` 
- Config file - `/etc/httpd/conf/httpd.conf`. `Include` directive is used to include external configuration files to the main httpd.conf file.
- `VirtualHost` - For hosting different websites on the same machine. 

### Apache Tomcat
- Java is a Prerequisite.
- In production installation `tomcat` user is created and Apache tomcat is installed as a service.
- `server.xml` in the conf directory has entry for `Connector`, which defines the tomcat port,etc.
- `logs` directory will have logs.
- `war` files are deployed in the `webapps` directory of the tomcat server.
-  `catalina.out` logs will show the status of the app deployment.

### Python - Deploy Flask App
- `python main.py` will start the development flask server. 
- Production deployments
  - Gunicorn - `gunicorn main:app -w 2` - Default 1 worker. 
  - uWSGI
  - Gevent
  - Twisted Web

### NodeJS - Deploy Express App
- `npm install` and then `node app.js` to start the application.
- `package.json` has a scripts section these script can be invoked by `npm run <script>`
- `npm run start` - used for starting the application instead of `node app.js`
- `supervisord`, `forever` and `pm2` can be used to ensure the node servers are running continously. Otherwise if the application crashes for some reason.
- `pm2 start app.js -i 4` - runs 4 instance of the application server load balanced.
- `pm2 delete app.js` - Removes the application from pm2.

### IPs and Ports
- Application can be made to listen to a port of specific network interface or all network intefaces. (All IPs or specific IPs).
- In Flask app, `app.run(port=5000, host="0.0.0.0")` wil make the app to listen to all IPs. If `host` is not mentioned, only listen to loopback interface (localhost).
- In tomcat `address` attribute of `Connector` configuration is used for similar settings as above.

## Database Basics
- `SQL` - Tabular/Relational 
  - Row, Table
  - `select * from persons where AGE > 10`
  - MySQL, PostgreSQL, Microsoft SQL Server
- `NoSQL` 
  - Document, Collection
  - `db.persons.find( { age: { $gt: 10 } } )`
  - mongoDB, DynamoDB, cassandra

### MySQL
- Open Source, Fast, Reliable, SQL
- Both community(free) and Commercial edition available.
- `3306` - default port
- `/var/log/mysqld.log` - log file

#### Privileges
```
mysql> GRANT <PERMISSION> ON <DB.TABLE> TO 'john'@'%';
```

| Privileges ||
|----|-----|
|ALL PRIVILEGES|Grant all access|
|CREATE|Create databases|
|DROP|Delete databases|
|DELETE|Delete rows from table|
|INSERT|Insert rows into table|
|SELECT|Read/Query table|
|UPDATE|Update rows in table|

Give Read/Query table permission of `persons` table in `school` database to user `john`.
```
mysql > GRANT SELECT ON school.persons TO 'john'@'%';
```

```
mysql > GRANT SELECT, UPDATE ON school.persons TO 'john'@'%';
```

```
mysql > GRANT SELECT, UPDATE ON school.* TO 'john'@'%';
```

```
mysql > GRANT ALL PRIVILEGES ON *.* TO 'john'@'%';
```

```
mysql > SHOW GRANTS FOR 'john'@'localhost';
```


## Security
## General Pre-Requisites
## 2 Tier Applications