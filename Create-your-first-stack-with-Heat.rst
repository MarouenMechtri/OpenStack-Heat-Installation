####
Create your First Stack with Heat
####

===============================

**Authors:**

Copyright (C) Marouen Mechtri

Copyright (C) Chaima Ghribi

================================


In this guide, we will detail the steps of creating a stack via Heat.

We will write a template that describes our infrastructure and just deploy it with Heat! 

    
It's quick and easy ;)


.. contents::

1. Overview
============

The most important step in this guide is the creation of the Heat Orchestration Template or HOT !

A HOT template is defined in YAML or JSON. In this guide, we use YAML. we think it's clearer to read ;)

HOT is also composed of three main sections (see the figure below):

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/heat-template.jpg

Detailed information on parameters and resource types are available on these links: `HOT Specification <http://docs.openstack.org/developer/heat/template_guide/hot_spec.html>`_ and  `OpenStack Resource Types <http://docs.openstack.org/developer/heat/template_guide/openstack.html>`_


Using the above information, let's create a simple Hot to deploy one server ;)

* The template looks like this::

	heat_template_version: 2013-05-23
      
	description: Hot Template to deploy a single server
      
	parameters:
		ImageID:
			type: string
			description: Image ID
		NetID:
			type: string
			description: External Network ID 
          
	resources:
		server_0
			type: OS::Nova::Server
			properties:
				name: "server0"
				image: { get_param: ImageID }
				flavor: "m1.small"
				networks:
				- network: { get_param: NetID }
      
	outputs:
		server0_ip:
			description: IP of the server 
			value: { get_attr: [ server_0, first_address ] }

Now, let's create a more complex template!

Here, we consider an infrastructure composed of two interconnected VMs which have
floating ips and are reachable from external networks (see the figure below).

----

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/infrastructure-example.jpg

----

To deploy this stack, we need to specify different resources in the HOT template.
The figure below describes clearly resources and their dependencies:


* The first resource we define is a private network (of type OS\::Neutron\::Net) to which we associate a resource of type subnet (OS\::Neutron\::Subnet).


* Second, we define a router (of type OS\::Neutron\::Router). We connect it to the pre-existing public network (see properties section) and we connect it also to the private subnet by defining a router interface (of type OS\::Neutron\::RouterInterface). 


* Then, for each defined server (OS\::Nova\::Server), we link it to a neutron port resource (of type OS\::Neutron\::Port) and to a floating ip resource (of type OS\::Neutron\::FloatingIP).


* Each port represents a logical switch port and is linked to the default security group to insure secure access to VMs.


* The floating ip resources provide external access to instances.


----

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/stack-resources.jpg


----


After identifying the needed resources, let's edit the template ;)


* Create template in the first-stack.yml file with the following content::

	vi first-stack.yml
         
	heat_template_version: 2013-05-23

	description: HOT template for two interconnected VMs with floating ips.

	parameters:
	  image_id:
		type: string
		description: Image Name
	 
	  secgroup_id:
		type: string
		description : Id of the security groupe

	  public_net:
		type: string
		description: public network id

	resources:
	  private_net:
		type: OS::Neutron::Net
		properties:
		  name: private-net
		 
	  private_subnet:
		type: OS::Neutron::Subnet
		properties:
		  network_id: { get_resource: private_net }
		  cidr: 172.16.2.0/24
		  gateway_ip: 172.16.2.1
		 
	  router1:
		type: OS::Neutron::Router
		properties:
		  external_gateway_info:
			network: { get_param: public_net }
		 
	  router1_interface:
		type: OS::Neutron::RouterInterface
		properties:
		  router_id: { get_resource: router1 }
		  subnet_id: { get_resource: private_subnet }

	  server1_port:
		type: OS::Neutron::Port
		properties:
		  network_id: { get_resource: private_net }
		  security_groups: [ get_param: secgroup_id ]
		  fixed_ips:
			- subnet_id: { get_resource: private_subnet }
	 
	  server1_floating_ip:
		type: OS::Neutron::FloatingIP
		properties:
		  floating_network_id: { get_param: public_net }
		  port_id: { get_resource: server1_port }

	  server1:
		type: OS::Nova::Server
		properties:
		  name: Server1
		  image: { get_param: image_id }
		  flavor: m1.tiny
		  networks:
			- port: { get_resource: server1_port }
		
	  server2_port:
		type: OS::Neutron::Port
		properties:
		  network_id: { get_resource: private_net }
		  security_groups: [ get_param: secgroup_id ]
		  fixed_ips:
			- subnet_id: { get_resource: private_subnet }
		 
	  server2_floating_ip:
		type: OS::Neutron::FloatingIP
		properties:
		  floating_network_id: { get_param: public_net }
		  port_id: { get_resource: server2_port }
		 
	  server2:
		type: OS::Nova::Server
		properties:
		  name: Server2
		  image: { get_param: image_id }
		  flavor: m1.tiny
		  networks:
			- port: { get_resource: server2_port }
		 
	outputs:
	  server1_private_ip:
		description: Private IP address of server1
		value: { get_attr: [ server1, first_address ] }
	  server1_public_ip:
		description: Floating IP address of server1
		value: { get_attr: [ server1_floating_ip, floating_ip_address ] }
	  server2_private_ip:
		description: Private IP address of server2
		value: { get_attr: [ server2, first_address ] }
	  server2_public_ip:
		description: Floating IP address of server2
		value: { get_attr: [ server2_floating_ip, floating_ip_address ] }

3. Create your stack
=====================

Now the template is ready! let's create the stack ;)

* Create a simple credential file::

    vi creds
    #Paste the following:
    export OS_TENANT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=admin_pass
    export OS_AUTH_URL="http://192.168.100.11:5000/v2.0/"
    
* Create a stack from the template (file available `here <https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/heat%20templates/first-stack.yml>`_)::

    Source creds

    NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')

    SEC_ID=$(nova secgroup-list | awk '/ default / { print $2 }')

    heat stack-create -f first-stack.yml \
    -P image_id=cirros-0.3.2-x86_64 \
    -P public_net=$NET_ID \
    -P secgroup_id=$SEC_ID First_Stack

    
4. Verify Stack creation
=========================

* Verify that the stack was created successfully::

    heat stack-list


Here is a snapshot of the Horizon dashboard interface after stack launching, you can see all the created resources ;)


.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/heat-GUI.png

* If you want to update a parameter of your stack (secgroup_id, public_net ...), run a command like this::

    heat stack-update First_Stack -f first-stack.yaml -P PARAMETER_NAME=PARAMETER_NEW_VALUE
 

* If you want to update your stack from a modified template file (available `here <https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/heat%20templates/modified-first-stack.yml>`_), run a command like this::

	NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')

	SEC_ID=$(nova secgroup-list | awk '/ default / { print $2 }')

	heat stack-update First_Stack -f modified-first-stack.yml \
	-P image_id=cirros-0.3.2-x86_64 \
	-P public_net=$NET_ID \
	-P secgroup_id=$SEC_ID
    
Now you are finally done! You can enjoy your first stack ;)

Please contact us for any question or suggestion :)


5. License
============

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