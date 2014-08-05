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
