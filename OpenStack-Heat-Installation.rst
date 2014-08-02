####
OpenStack Heat Installation Guide
####

Welcome to OpenStack Heat installation manual !

This document is based on `the OpenStack Official Documentation <http://docs.openstack.org/icehouse/install-guide/install/apt/content/index.html>`_ for Icehouse. 

:Version: 1.0
:Authors: Marouen Mechtri and Chaima Ghribi
:License: Apache License Version 2.0
:Keywords: Heat, OpenStack, Icehouse, Orchestration, Installation, Ubuntu

===============================

**Authors:**

Copyright (C) Marouen Mechtri

Copyright (C) Chaima Ghribi

================================

.. contents::

1. Heat Overview
================

In this guide, we will go over the installation of an awesome OpenStack service !  

OpenStack Heat !  

Heat is an openstack service that handles the orchestration of complex deployments on top of OpenStack clouds. Orchestration basically 
manages the infrastructure but it supports also the software configuration management.  

Heat provides users the ability to define their applications in terms of templates.

Just write a Heat template that describes your infrastructure resources (instances, networks, database, images …) and send it to Heat! It will talk to all the other OpenStack APIs to deploy your stack! 

If you want to extend or redesign your infrastructure, modify the template and update your stack. Heat will do everything for you ;)

Let's Install it ;)

2. Heat Install
===============

In our previous `OpenStack Icehouse installation guide <https://github.com/ChaimaGhribi/OpenStack-Icehouse-Installation/blob/master/OpenStack-Icehouse-Installation.rst>`_, we 've installed the basic services on the controller node.

Now we will add the Heat orchestration service ;)


.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/controller+heat.jpg


* Change to super user mode::

    sudo su

* Install heat packages::

    apt-get install -y heat-api heat-api-cfn heat-engine


* Create a MySql database for heat::

    mysql -u root -p

    CREATE DATABASE heat;
    GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' IDENTIFIED BY 'HEAT_DBPASS';
    GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' IDENTIFIED BY 'HEAT_DBPASS';
    exit;


* Configure service user and role::

    keystone user-create --name=heat --pass=service_pass --email=heat@domain.com
    keystone user-role-add --user=heat --tenant=service --role=admin



* Register the service and create the endpoint::
    
    keystone service-create --name=heat --type=orchestration --description="Orchestration"
    keystone endpoint-create \
    --service-id=$(keystone service-list | awk '/ orchestration / {print $2}') \
    --publicurl=http://192.168.100.11:8004/v1/%\(tenant_id\)s \
    --internalurl=http://controller:8004/v1/%\(tenant_id\)s \
    --adminurl=http://controller:8004/v1/%\(tenant_id\)s
    
    keystone service-create --name=heat-cfn --type=cloudformation --description="Orchestration CloudFormation"
    keystone endpoint-create \
    --service-id=$(keystone service-list | awk '/ cloudformation / {print $2}') \
    --publicurl=http://192.168.100.11:8000/v1 \
    --internalurl=http://controller:8000/v1 \
    --adminurl=http://controller:8000/v1


* Create the heat_stack_user role::

    keystone role-create --name heat_stack_user


* Edit the /etc/heat/heat.conf file::

    vi /etc/heat/heat.conf
   
    [database]
    replace connection=sqlite:////var/lib/heat/$sqlite_db ith:
    connection = mysql://heat:HEAT_DBPASS@controller/heat
  
    [DEFAULT]  
    verbose = True
    log_dir=/var/log/heat
    rabbit_host = controller
    heat_metadata_server_url = http://10.0.0.11:8000
    heat_waitcondition_server_url = http://10.0.0.11:8000/v1/waitcondition
    
    [keystone_authtoken]
    auth_host = controller
    auth_port = 35357
    auth_protocol = http
    auth_uri = http://controller:5000/v2.0
    admin_tenant_name = service
    admin_user = heat
    admin_password = service_pass
    
    [ec2authtoken]
    auth_uri = http://controller:5000/v2.0



* Remove heat SQLite database::

    rm /var/lib/heat/heat.sqlite


* Synchronize your database::
  
    heat-manage db_sync

* Restart the Orchestration services::

    service heat-api restart
    service heat-api-cfn restart
    service heat-engine restart

* Verify configuration, list stacks::
  
    source creds
    heat stack-list


That's it ;) 

Installation is too easy and quick but results are really great!

A detailed heat usage guide will be provided soon ;)

You could create and test your own stacks :)

3. License
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


4. Contacts
===========

Marouen Mechtri : marouen.mechtri@it-sudparis.eu

Chaima Ghribi: chaima.ghribi@it-sudparis.eu

