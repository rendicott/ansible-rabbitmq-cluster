rabbitmq
======

This role configures RabbitMQ cluster. This playbook is derived from alexeymedvedchikov.rabbitmq but adds the ability to use FQDN.
All props to https://github.com/alexey-medvedchikov/ansible-rabbitmq 

Requirements
------------

Clustering requires proper short hostname resolution on every node in cluster.
Use some /etc/hosts setup role for this. All other requirements are listed in
metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows.

	# Set /var/lib/rabbitmq/.erlang.cookie
	rabbitmq_erlang_cookie: 1qa2ws3ed

	# Set max open files for rabbit process 
	rabbitmq_ulimit_open_files: 1024

	# Use cluster or no
	rabbitmq_create_cluster: no

	# Short hostname of cluster master, on master node it must be specified too
	rabbitmq_cluster_master: localhost

	# Vhosts to create
	rabbitmq_vhosts: []

	# Enabled plugins
	rabbitmq_plugins:
	  - rabbitmq_management

	# Users to create, self explained parameters
	rabbitmq_users:
	  - user: admin
	    password: admin
	    vhost: /
	    configure_priv: .*
	    read_priv: .*
	    write_priv: .*
	    tags: administrator

	# Users to remove
	rabbitmq_users_removed:
	  - guest

       # Use FQDN
       rabbitmq_use_longname: 'true'

Examples
--------

First of all you need configure /etc/hosts:

	[all]
	hosts:
	  - { ip: '10.0.0.1', domain: 'mq1' }
	  - { ip: '10.0.0.2', domain: 'mq2' }
	  - { ip: '10.0.0.3', domain: 'mq3' }
	  - { ip: '10.0.0.4', domain: 'mq4' }

	[rabbidz]
	mq1.local rabbitmq_cluster_master=mq2
	mq2.local rabbitmq_cluster_master=mq2
	mq3.local rabbitmq_cluster_master=mq3
	mq4.local rabbitmq_cluster_master=mq3

	[rabbidz:vars]
	rabbitmq_create_cluster=yes

or you can use group variables (preferably)

	[rabbidz1]
	mq1.local
	mq2.local

	[rabbidz1:vars]
	rabbitmq_create_cluster=yes
	rabbitmq_cluster_master=mq3

	[rabbidz2]
	mq3.local
	mq4.local

	[rabbidz2:vars]
	rabbitmq_create_cluster=yes
	rabbitmq_cluster_master=mq3


Rendicott notes
-------
hosts file looked like this:

        [umber-rab]
        rabr01.docl.nic  rabbitmq_cluster_loadbalancer=no
        rabr02.docl.nic  rabbitmq_cluster_loadbalancer=no
        rabr03.docl.nic  rabbitmq_cluster_loadbalancer=no
        rabr04.docl.nic  rabbitmq_cluster_loadbalancer=no
        rabh01.docl.nic  rabbitmq_cluster_loadbalancer=yes
        
        [umber-rab:vars]
        rabbitmq_create_cluster=yes
        rabbitmq_cluster_master=rabr01
        rabbitmq_high_availability=yes
        rabbitmq_ha_enabled=yes

Note: in the umber-rab:vars section the cluster master name has to be the short name (matches hostname) and it has to be resolvable by all the nodes

Then run playbook like this: 

        ansible-playbook -e machineName=umber-rab linuxRabbitCluster.yml -vv --inventory-file=/root/ucluster

Then go to HA admin page like this:

        http://rabh01.docl.nic:15672
        admin /admin

License
-------

LGPL

Author Information
------------------

- Alexey Medvedchikov, 2GIS, LLC

