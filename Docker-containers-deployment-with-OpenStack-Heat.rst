####
Docker containers deployment with OpenStack Heat
####


:Version: 1.0
:Authors: Marouen Mechtri and Chaima Ghribi 
:License: Apache License Version 2.0
:Keywords: Docker, Heat, OpenStack, Icehouse, Ubuntu 14.04


===============================

**Authors:**

Copyright (C) `Marouen Mechtri <https://www.linkedin.com/in/mechtri>`_


Copyright (C) `Chaima Ghribi <https://www.linkedin.com/profile/view?id=53659267&trk=nav_responsive_tab_profile>`_


================================

.. contents::


1. Overview
============

We’ve all heard about Docker and the immense amount a buzz around it from last year !

What’s Docker ? why is it useful ? how can we use it and integrate it into Openstack ? 
and how to deploy our applications with it ? 

All these questions came to our minds when we started discovering Docker.

So, we wrote this manual to share with you our experience with Docker and OpenStack.
We will first introduce Docker and show you how it can be integrated into Openstack. Then,
we will explain how to successfully deploy your docker containers with Openstack Heat. 

All the steps of installation and deployment are described in details.
Hope our step by step instructions are easy to follow, even for beginners.


2. What Is Docker?
==================

Docker is an open source project to automatically deploy applications into containers. 
It commoditizes the well known LXC (Linux Container) solution that provides operating system
level virtualization and allows to run multiple containers on the same server. 

To make a simple analogy, a Docker is like an hypervisor.  But unlike traditional Vms,
docker containers are lightweight as they  don’t run OSes but share the host’s operating system (see the figure below).

A docker container is also portable, it hosts the application and its dependencies and it is able
to be deployed or relocated on any Linux server.

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-vs-hypervisor.jpg

The Docker element that manages containers and deploys applications on them is called Docker Engine.

The second component of Docker is Docker Hub. We found it really Awesome ! 
It's the Docker's repository of application. You can share your application with your team
members and you can ship and run it anywhere... It's really amazing :)

It was a quick introduction to Docker ! Now, let's see how to use it with OpenStack ;) 

3. OpenStack & Docker
======================

Openstack can be easily enhanced by docker plugins. 
Docker can be integrated into OpenStack Nova as a form of hypervisor (Containers used as VMs).
But there is a better way to use Docker with OpenStack.
It to orchestrate containers with OpenStack Heat !

You have just to install the docker plugin on Heat and you will be able to create
containers and deploy your applications on the top of them.
You need to identify the required resources, edit your template and deploy it on Heat. Your stack will be created!
(For detailed information on Heat and template creation, you can see our `Heat usage guide <https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/Create-your-first-stack-with-Heat.rst>`_). 


If you want to test Docker with Heat, we recommand you to deploy OpenStack using a flat networking model (see our `guide <https://github.com/ChaimaGhribi/Icehouse-Installation-Flat-Networking>`_).
One issue we encountered when we considered a multi-node architecture with isolated neutron networks is that 
instances were not able to signal to Heat. So, stack creation fails because it dependens on 
connectivity from VMs to the Heat engine. 

The figure below shows the communication between Heat, Nova, Docker and the instance when creating a stack. 
In this example we have deployed apache and mysql on two Docker containers. Stack deployment fails
if the signal (3) can not reach Heat API.

I think this limitation will be overcome in the next versions to allow
users using isolated neutron networks to deploy and test Docker!

But now you can enjoy docker on OpenStack with Flat-networking ;)
  

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-plugin.jpg

4. Deploy Docker containers with OpenStack Heat
===============================================

Now that we have discussed about Docker and Heat, let's move to practice !
We will show you how to install the Docker plugin, how to write your template and how to deploy it with Heat ;)


4.1. Install the Docker Plugin 
--------------------------------

* To get the Docker plugin, download the Heat folder available on GitHub::

    download heat (the ZIP folder) from here
    https://github.com/openstack/heat/tree/stable/icehouse

* Unzip it::

    unzip heat-stable-icehouse.zip


* Remove the tests folder to avoid conflicts::

    cd heat-stable-icehouse/contrib/
    rm -rf docker/docker/tests

* create a new directory under /usr/lib/heat/:: 

    mkdir /usr/lib/heat 
    mkdir /usr/lib/heat/docker-plugin

* Copy the docker plugin under your new directory::

    cp -r docker/* /usr/lib/heat/docker-plugin
  
* Now, install the docker plugin::

    cd /usr/lib/heat/docker-plugin
    apt-get install python-pip
    pip install -r requirements.txt  
    
    
* Edit /etc/heat/heat.conf file::

    vi /etc/heat/heat.conf
    (add)
    plugin_dirs=/usr/lib/heat/docker-plugin/docker
 
 
* Restart services::

    service heat-api restart
    service heat-api-cfn restart
    service heat-engine restart    
    

* Check that the DockerInc\::Docker\::Container resource was successfully added and appears in your resource list::

    heat resource-type-list | grep Docker 
    

4.2. Create your Heat template
-------------------------------

Before editing the template, let's discuss a bit about the content and the resources we will define ;)

In this example, we want to dockerize and deploy a lamp application. So, we will create a docker 
container running apache with php and another one running mysql database. 

We define an OS::Heat::SoftwareConfig resource that describes the configuration and an OS::Heat::SoftwareDeployment resource to
deploy configs on OS::Nova::Server (the Docker server). We associate 
a floating IP to the Docker server to be able to connect to Internet ( using OS::Nova::FloatingIP and OS::Nova::FloatingIPAssociation resources). 
Then, we create two docker containers of type DockerInc::Docker::Container on the Docker host. 

Note: here we provide a simple template, many other interseting parameters ( port_bindings, name, links...) can enhance 
the template and enable more sophisticated use of Docker. These parameters are not supported by the current Docker plugin.  
We will provide more complex templates with the next plugin version. 

Now let's edit our template! 

* Create template in the docker-stack.yml file with the following content::

    vi docker-stack.yml

    heat_template_version: 2013-05-23

    description: >
      Dockerize a multi-node application with OpenStack Heat.
      This template defines two docker containers running
      apache with php and mysql database. 
      
    parameters:
      key:
        type: string
        description: >
          Name of a KeyPair to enable SSH access to the instance. Note that the
          default user is ec2-user. 
        default: key1

      flavor:
	type: string
	description: Instance type for the docker server.
	default: m1.medium
	
      image:
	type: string
	description: >
	  Name or ID of the image to use for the Docker server.  This needs to be
	  built with os-collect-config tools from a fedora base image.
	default: fedora-software-config
	  
      public_net:
	type: string
	description: name of public network for which floating IP addresses will be allocated.
	default: nova 

    resources:
      configuration:
	type: OS::Heat::SoftwareConfig
	properties:
	  group: script
	  config: |
	    #!/bin/bash -v
	    setenforce 0
	    yum -y install docker-io
	    cp /usr/lib/systemd/system/docker.service /etc/systemd/system/
	    sed -i -e '/ExecStart/ { s,fd://,tcp://0.0.0.0:2375, }' /etc/systemd/system/docker.service
	    systemctl start docker.service
	    docker -H :2375 pull marouen/mysql
	    docker -H :2375 pull marouen/apache
	  
      deployment:
        type: OS::Heat::SoftwareDeployment
	properties:
	  config: {get_resource: configuration}
	  server: {get_resource: docker_server}
	  
      docker_server:
	type: OS::Nova::Server
	properties:
	  key_name: {get_param: key}
	  image: { get_param: image }
	  flavor: { get_param: flavor}
	  user_data_format: SOFTWARE_CONFIG
	  
      server_floating_ip:
	type: OS::Nova::FloatingIP
	properties:
	  pool: { get_param: public_net}

      associate_floating_ip:
	type: OS::Nova::FloatingIPAssociation
	properties:
	  floating_ip: { get_resource: server_floating_ip}
	  server_id: { get_resource: docker_server}
	  
      mysql:
	type: DockerInc::Docker::Container
	depends_on: [deployment]
	properties:
	  image: marouen/mysql
	  port_specs:
	    - 3306
	  docker_endpoint:
	    str_replace:
	      template: http://host:2375
	      params:
	        host: {get_attr: [docker_server, networks, private, 0]}

      apache:
	type: DockerInc::Docker::Container
	depends_on: [mysql]
	properties:
	  image: marouen/apache
	  port_specs:
	    - 80
	  docker_endpoint:
	    str_replace:
	      template: http://host:2375
	      params:
		host: {get_attr: [docker_server, networks, private, 0]}

    outputs:
      url:
	description: Public address of apache
	value:
	  str_replace:
	    template: http://host
	    params:
	      host: {get_attr: [docker_server, networks, private, 0]}


4.3. Deploy your stack
-----------------------

4.3.1. Pre-deployment
^^^^^^^^^^^^^^^^^^^^^^

* Create a simple credential file::

    vi creds
    #Paste the following:
    export OS_TENANT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=admin_pass
    export OS_AUTH_URL="http://controller:5000/v2.0/"
    
* To create a fedora based image, we followed the steps bellow `(source via this link) <https://github.com/openstack/heat-templates/tree/master/hot/software-config/elements>`_::    
    

    git clone https://git.openstack.org/openstack/diskimage-builder.git
    git clone https://git.openstack.org/openstack/tripleo-image-elements.git
    git clone https://git.openstack.org/openstack/heat-templates.git
    export ELEMENTS_PATH=tripleo-image-elements/elements:heat-templates/hot/software-config/elements
    diskimage-builder/bin/disk-image-create vm \
    fedora selinux-permissive \
    heat-config \
    os-collect-config \
    os-refresh-config \
    os-apply-config \
    heat-config-cfn-init \
    heat-config-puppet \
    heat-config-salt \
    heat-config-script \
    -o fedora-software-config.qcow2
    glance image-create --disk-format qcow2 --container-format bare --name fedora-software-config < \
    fedora-software-config.qcow2
    
* If you didn't created a key, use these commands::

   ssh-keygen
   nova keypair-add --pub-key ~/.ssh/id_rsa.pub key1    
    
* Add rules to the default security group to enable the access to the docker server::

   # Permit ICMP (ping):
   nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0

   # Permit secure shell (SSH) access:
   nova secgroup-add-rule default tcp 22 22 0.0.0.0/0

   # Permit 2375 port access (Docker endpoint):
   nova secgroup-add-rule default tcp 2375 2375 0.0.0.0/0  
   

* If you need to create a new private network, use these commands::

   source creds

   #Create a private network:
   nova network-create private --bridge br100 --multi-host T  --dns1 8.8.8.8  \
   --gateway 172.16.0.1 --fixed-range-v4 172.16.0.0/24
   
* Create a floating IP pool to connect instances to Internet::

   nova-manage floating create --pool=nova --ip_range=192.168.100.100/28
   

4.3.2. Create your stack
^^^^^^^^^^^^^^^^^^^^^^^^^

* Create a stack from the template (file available `here <https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/heat%20templates/docker-stack.yml>`_)::

    source creds

    heat stack-create -f docker-stack.yml docker-stack


* Verify that the stack was created::

    heat stack-list


It could take some minutes, so just wait ... 

Here is a snapshot of the Horizon dashboard interface after stack launching: 

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-stack.png
  
  
* To check that your containers are created::
  
    ssh ec2-user@192.168.100.97
  
    sudo docker -H :2375 ps 
  
  
.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-containers.png

That's it! you can now play with your Docker containers ;)
Please get back to us if you have any question. 


5. License
=========
Institut Mines Télécom - Télécom SudParis  

Copyright (C) 2014  Authors

Original Authors -  Marouen Mechtri and  Chaima Ghribi 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except 

in compliance with the License. You may obtain a copy of the License at::

    http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


6. Contacts
===========

Marouen Mechtri : marouen.mechtri@it-sudparis.eu

Chaima Ghribi: chaima.ghribi@it-sudparis.eu
