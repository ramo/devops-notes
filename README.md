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
