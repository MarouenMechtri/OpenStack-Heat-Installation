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
