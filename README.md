# vagrant-deploy-ansible-using-shell

Vagrant Ansible LAB Guide – Bridged network
===========================================

![](https://www.multicastbits.com/wp-content/uploads/2020/05/ANSVagrant-1-1024x595.png)


Here’s a is a quick guide to get you started with a “Ansible core lab” using Vagrant.

Alright lets get started

TLDR Version
------------

-   Install Vagrant
-   Install Virtual-box
-   Create project folder and CD in to it

```  
Vagrant init
```

-   Vagrantfile –     [link](https://github.com/malindarathnayake/Ansible_Vagrant_LAB/blob/master/Vagrantfile)
-   Vagrant Provisioning Shell Script to Deploy Ansible     –[link](https://github.com/malindarathnayake/Ansible_Vagrant_LAB/blob/master/Ansible_LAB_setup.sh)
-   Install the vagrant-vbguest plugin to deploy missing

``` 
vagrant plugin install vagrant-vbguest
```

-   Bring up the Vagrant environment

``` 
Vagrant up
```

Install Vagrant and Virtual box
-------------------------------

*For this demo we are using windows 10 1909 but you can use the same guide for MAC OSX*

Windows

Download Vagrant and virtual box and install it the good ol way – 
``` 
https://www.vagrantup.com/downloads.html
https://www.virtualbox.org/wiki/Downloads
https://www.vagrantmanager.com/downloads/
```

Install the vagrant-vbguest plugin (We need this with newer versions of Ubuntu)

```  
vagrant plugin install vagrant-vbguest
```

Or Using chocolatey

``` 
choco install vagrant
```

``` 
choco install virtualbox
```

```  
choco install vagrant-manager
```

Install the vagrant-vbguest plugin (We need this with newer versions of Ubuntu)

```  
vagrant plugin install vagrant-vbguest
```

MAC OSX – using Brewcask

Install virtual box

```  
$ brew cask install virtualbox
```

Now install Vagrant either [from the website](https://www.vagrantup.com/downloads.html) or use homebrew for installing it.

```  
$ brew cask install vagrant
```

[Vagrant-Manager](http://vagrantmanager.com/) is a nice way to manage all your virtual machines in one place directly from the menu bar.

```  
$ brew cask install vagrant-manager
```

Install the vagrant-vbguest plugin (We need this with newer versions of Ubuntu)

```  
vagrant plugin install vagrant-vbguest
```

### Setup the Vagrant Environment

Open Powershell

to get started lets check our environment

```  
vagrant version
```

![](http://45.79.141.228/wp-content/uploads/2020/05/image-3.png)

Create a project directory and Initialize the environment for the project directory im using D:\\vagrant

Open powershell and run

```  
mkdir D:\vagrant
cd D:\vagrant
```

Initialize the environment under the project folder

```  
vagrant init
```

![](http://45.79.141.228/wp-content/uploads/2020/05/image-6.png)

this will create Two Items

![](http://45.79.141.228/wp-content/uploads/2020/05/image-7.png)

.vagrant – Hidden folder holding Base Machines and meta data

Vagrantfile – Vagrant config file

Lets Create the Vagrantfile to deploy the VMs

[https://www.vagrantup.com/docs/vagrantfile/](https://www.vagrantup.com/docs/vagrantfile/)

The syntax of *Vagrantfiles* is *Ruby* this gives us a lot of flexibility to program in logic when building your files

Im using Atom to edit the vagrantfile

```  
Vagrant.configure("2") do |config|
     config.vm.define "controller" do |controller|
                  controller.vm.box = "ubuntu/trusty64"
                  controller.vm.hostname = "LAB-Controller"
                  controller.vm.network "public_network", bridge: "Intel(R) I211 Gigabit Network Connection", ip: "172.17.10.120"
                    controller.vm.provider "virtualbox" do |vb|
                                 vb.memory = "2048"
                  end
                  controller.vm.provision :shell, path: 'Ansible_LAB_setup.sh'
   end
   (1..3).each do |i|
         config.vm.define "vls-node#{i}" do |node|
                       node.vm.box = "ubuntu/trusty64"
                       node.vm.hostname = "vls-node#{i}"
                       node.vm.network "public_network", bridge: "Intel(R) I211 Gigabit Network Connection" ip: "172.17.10.12#{i}"
                      node.vm.provider "virtualbox" do |vb|
                                                  vb.memory = "1024"
                     end
              end
        end
end
```

You can grab the code from my Repo

[https://github.com/malindarathnayake/Ansible\_Vagrant\_LAB/blob/master/Vagrantfile](https://github.com/malindarathnayake/Ansible_Vagrant_LAB/blob/master/Vagrantfile)

Let’s talk a little bit about this code and unpack this

**Vagrant API version**

![](http://45.79.141.228/wp-content/uploads/2020/05/image-9.png)

Vagrant uses API versions for its configuration file, this is how it can stay backward compatible. So in every *Vagrantfile* we need to specify which version to use. The current one is version 2 which works with Vagrant 1.1 and up.

**Provisioning the Ansible VM**

![](http://45.79.141.228/wp-content/uploads/2020/05/image-31-1024x223.png)

This will

-   Provision the controller Ubuntu VM
-   Create a bridged network adapter
-   Set the host-name – LAB-Controller
-   Set the static IP – 172.17.10.120/24
-   Run the Shell script that installs Ansible using apt-get install (We will get to this below)

Lets start digging in…

Specifying the Controller VM Name, base box and hostname

![](http://45.79.141.228/wp-content/uploads/2020/05/image-34-1024x80.png)

Vagrant uses a base image to clone a virtual machine quickly. These base images are known as “boxes” in Vagrant, and specifying the box to use for your Vagrant environment is always the first step after creating a new Vagrantfile.

You can find different base boxes from [app.vagrantup.com](https://app.vagrantup.com/boxes/search)

Or you can create custom base boxes for pretty much anything including “CiscoVIRL(CML)” images – *keep an eye out for the next article on this*

Network configurations

![](http://45.79.141.228/wp-content/uploads/2020/05/image-36-1024x32.png)

```  
controller.vm.network "public_network", bridge: "Intel(R) I211 Gigabit Network Connection", ip: "your IP"
```

in this case, we are asking it to create a bridged adapter using the *Intel(R) I211 NIC* and set the IP address you defined on under IP attribute

You can the relavant interface name using

```  
get-netadapter
```

![](http://45.79.141.228/wp-content/uploads/2020/05/image-30-1024x176.png)

You can also create a host-only private network

```  
controller.vm.network :private_network, ip: "10.0.0.10"
```

for more info checkout the network section in the KB

[https://www.vagrantup.com/docs/networking/](https://www.vagrantup.com/docs/networking/)

Define the provider and VM resources

![](http://45.79.141.228/wp-content/uploads/2020/05/image-13.png)

We declaring virtualbox(we installed this earlier) as the provider and setting VM memory to 2048

You can get more granular with this, refer to the below KB

[https://www.vagrantup.com/docs/virtualbox/configuration.html](https://www.vagrantup.com/docs/virtualbox/configuration.html)

Define the shell script to customize the VM config and install the Ansible Package

![](http://45.79.141.228/wp-content/uploads/2020/05/image-14.png)

Now this is where we define the provisioning shell script

this script installs *Ansible*and set the host file entries to make your life easier

In case you are wondering VLS stands for V=virtual,L – linux S – server.

I use this naming scheme for my VMs. Feel free to use anything you want; make sure it matches what you defined on the Vagrantfile under *node.vm.hostname*

```  
!/bin/bash
sudo apt-get update
sudo apt-get install software-propetise-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible -y
echo "
172.17.10.120 LAB-controller
172.17.10.121 vls-node1
172.17.10.122 vls-node2
172.17.10.123 vls-node3" >> /etc/hosts
```

create this file and save it as *Ansible\_LAB\_setup.sh* in the Project folder

in this case I’m going to save it under D:\\vagrant

You can also do this inline with a script block instead of using a separate file

[https://www.vagrantup.com/docs/provisioning/basic\_usage.html](https://www.vagrantup.com/docs/provisioning/basic_usage.html)

**Provisioning the Member servers for the lab**

![](http://45.79.141.228/wp-content/uploads/2020/05/image-37-1024x239.png)

We covered most of the code used above, the only difference here is we are using **[each method](https://mixandgo.com/learn/how-to-use-ruby-each)** to create 3 VMs with the same template (I’m lazy and it’s more convenient)

This will create three Ubuntu VMs with the following Host-names and IP addresses, you should update these values to match you LAN, or use a private Adapter

vls-node1 – 172.17.10.121
vls-node2 – 172.17.10.122
vls-node1 – 172.17.10.123

So now that we are done with explaining the code, let’s run this

### Building the Lab environment using Vagrant

Issue the following command to check your syntax

```  
Vagrant status
```

Issue the following command to bring up the environment

```  
Vagrant up
```

![](http://45.79.141.228/wp-content/uploads/2020/05/image-18.png)

If you get this message Reboot in to UEFI and make sure virtualization is enabled

Intel – VT-D
AMD Ryzen – SVM

If everything is kumbaya you will see *vagrant* firing up the deployment

![](http://45.79.141.228/wp-content/uploads/2020/05/image-19-1024x264.png)

It will provision 4 VMs as we specified

Notice since we have the “*vagrant-vbguest*” plugin installed, it will reinstall the relevant guest tools along with the dependencies for the OS\

```  
==> vls-node3: Machine booted and ready!
[vls-node3] No Virtualbox Guest Additions installation found.
rmmod: ERROR: Module vboxsf is not currently loaded
rmmod: ERROR: Module vboxguest is not currently loaded
Reading package lists...
Building dependency tree...
Reading state information...
Package 'virtualbox-guest-x11' is not installed, so not removed
The following packages will be REMOVED:
  virtualbox-guest-utils*
0 upgraded, 0 newly installed, 1 to remove and 0 not upgraded.
After this operation, 5799 kB disk space will be freed.
(Reading database ... 61617 files and directories currently installed.)
Removing virtualbox-guest-utils (6.0.14-dfsg-1) ...
Processing triggers for man-db (2.8.7-3) ...
(Reading database ... 61604 files and directories currently installed.)
Purging configuration files for virtualbox-guest-utils (6.0.14-dfsg-1) ...
Processing triggers for systemd (242-7ubuntu3.7) ...
Reading package lists...
Building dependency tree...
Reading state information...
linux-headers-5.3.0-51-generic is already the newest version (5.3.0-51.44).
linux-headers-5.3.0-51-generic set to manually installed.
```

**Check the status**

```  
Vagrant status
```

![](http://45.79.141.228/wp-content/uploads/2020/05/image-21.png)

![](http://45.79.141.228/wp-content/uploads/2020/05/image-20-1024x484.png)

### **Testing**

**Connecting via SSH to your VMs**

```  
vagrant ssh controller
```

*“Controller” is the VMname we defined before not the hostname, You can find this by running **Vagrant status*** on posh or your terminal

We are going to connect to our controller and check everything

![](http://45.79.141.228/wp-content/uploads/2020/05/image-43.png)

![](http://45.79.141.228/wp-content/uploads/2020/05/image-42.png)

**Little bit more information on the networking side**

Vagrant Adds two interfaces, for each VM

NIC 1 – Nat’d in to the host (control plane for Vagrant to manage the VMs)

![](http://45.79.141.228/wp-content/uploads/2020/05/image-22.png)

NIC 2 – Bridged adapter we provisioned in the script with the IP Address

![](http://45.79.141.228/wp-content/uploads/2020/05/image-23.png)

Default route is set via the Private(NAT’d) interface (you cant change it)

![](http://45.79.141.228/wp-content/uploads/2020/05/image-38.png)

Netplan configs

Vagrant creates a custom **netplan**yaml for interface configs

![](http://45.79.141.228/wp-content/uploads/2020/05/image-39.png)

![](http://45.79.141.228/wp-content/uploads/2020/05/image-40.png)

* * * * *

### Destroy/Tear-down the environment

```  
vagrant destroy -f
```

[https://www.vagrantup.com/intro/getting-started/teardown.html](https://www.vagrantup.com/intro/getting-started/teardown.html)

I hope this helped someone. when I started with Vagrant a few years back it took me a few tries to figure out the system and the logic behind it, this will give you a basic understanding on how things are plugged together.

let me know in the comments if you see any issues or mistakes.

Until Next time…..

