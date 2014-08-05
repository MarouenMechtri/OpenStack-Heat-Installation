OpenStack-Heat-Installation
===========================

In this guide, we will go over the installation of an awesome OpenStack service !

OpenStack Heat !

Heat is an openstack service that handles the orchestration of complex deployments on top of OpenStack clouds. Orchestration basically manages the infrastructure but it supports also the software configuration management.
Heat provides users the ability to define their applications in terms of templates.

Just write a Heat template that describes your infrastructure resources (instances, networks, database, images …) and send it to Heat! It will talk to all the other OpenStack APIs to deploy your stack!
If you want to extend or redesign your infrastructure, modify the template and update your stack. Heat will do everything for you ;)

In our previous [OpenStack Icehouse installation guide](https://github.com/ChaimaGhribi/OpenStack-Icehouse-Installation/blob/master/OpenStack-Icehouse-Installation.rst), we 've installed the basic services on the controller node.

Now we will add the Heat orchestration service ;)

The Installation guide is available here [OpenStack-Heat-Installation](https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/OpenStack-Heat-Installation.rst) 

![Heat image](https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/controller+heat.jpg)

If you want to create your first template with Heat, follow the instructions in our stack creation guide available here [Create-First-Stack-with-Heat](https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/Create-your-first-stack-with-Heat.rst). 
