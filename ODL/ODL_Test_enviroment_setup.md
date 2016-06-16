# OpenDaylight Environment Setup

## Overview

This is an instruction base on **OpenDaylight Performance White Paper**

### Hardware Environment ###

All component list below is running in one node, communicating by **loopback** network.

#### Dependencies send up ####
	sudo apt-get update
	sudo apt-get install -y curl wget unzip git
	
	# change sudoer file to allow the currnet use (sdn e.g) to use sudo command without password
	$ visudo
	# insert the following setting:
	# sdn     ALL=NOPASSWD: ALL
	
##### 1. Java #####
	sudo apt-get install software-properties-common -y
	sudo add-apt-repository ppa:webupd8team/java -y
	sudo apt-get update
	sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
	export JAVA_HOME=/usr/lib/jvm/java-8-oracle
	
##### 2. Mininet #####
	git clone http://github.com/mininet/mininet
	# you can select a version
	mininet/util/install.sh -nvfw
	
##### 3. pypy #####
	curl -O https://bootstrap.pypa.io/get-pip.py
	sudo pypy get-pip.py
	
##### 4. pypy python packages #####
	sudo /usr/local/bin/pip install cffi==1.3.1
	sudo /usr/local/bin/pip install greenlet==0.4.9
	sudo /usr/local/bin/pip install netaddr==0.7.18
	sudo /usr/local/bin/pip install readline==6.2.4.1
	sudo /usr/local/bin/pip install requests==2.9.1
	sudo /usr/local/bin/pip install wsgiref==0.1.2
	
##### 5.1 Option 1: OpenDaylight package #####
	wget https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.4.0-Beryllium/distribution-karaf-0.4.0-Beryllium.tar.gz
	tar xf distribution-karaf-0.4.0-Beryllium.tar.gz
	cd distribution-karaf-0.4.0-Beryllium/
	export JAVA_MAX_MEM=8g
	./bin/start
	
	# install plugin
	./bin/client -u karaf
	# old(He)
	odl> feature:install odl-openflowplugin-flow-services-ui
	# new(Li)
	odl> feature:install odl-openflowplugin-flow-services-ui-li
	./bin/stop
	
	# disable flow persistance
	vi etc/org.opendaylight.controller.cluster.datastore.cfg
	# Enable or disable data persistence.
	# persistent=false
	
	# start controller
	./bin/start
	export JAVA_MAX_MEM=8g; ./apache-karaf-3.0.5/bin/start
	
##### 5.2 Option 2: ONOS package #####	
	wget https://downloads.onosproject.org/release/onos-1.5.1.zip
	unzip -q onos-1.5.1.zip
	cd onos-1.5.1/
	export JAVA_MAX_MEM=8g; ./apache-karaf-3.0.5/bin/start
	./apache-karaf-3.0.5/bin/client
	# 
	onos> apps -s
	onos> app activate org.onosproject.openflow

##### 6. performance test packages #####
	cd
	git clone https://git.opendaylight.org/gerrit/p/integration/test.git
	cd test/tools/odl-mdsal-clustering-tests/clustering-performance-test/
	
#### TestCase: OpenFlow Northbound performance ####
You will need to open multiple ssh session for operating the test

##### session 1: start mininet topology #####
	# 15 switches
	sudo mn --controller=remote,ip=127.0.0.1,port=6653 --topo tree,4
	# 31 switches
	sudo mn --controller=remote,ip=127.0.0.1,port=6653 --topo tree,5
	# 63 switches
	sudo mn --controller=remote,ip=127.0.0.1,port=6653 --topo tree,6

##### session 2: monitoring flows #####
	cd test/tools/odl-mdsal-clustering-tests/clustering-performance-test/
	./ovs-scripts/get-total-found.sh
	
##### session 3.1: execute ODL test script #####
	# 1 flow / REST call
	cd test/tools/odl-mdsal-clustering-tests/clustering-performance-test/
	pypy ./flow_add_delete_test.py --threads 5 --flows 20000 --auth --timeout 10
	
	# 200 flows / REST call
	pypy ./flow_add_delete_test.py --threads 5 --flows 20000 --auth --timeout 10 --fpr 200 --bulk-delete
	
##### session 3.2: execute ONOS test script #####
	cd test/tools/odl-mdsal-clustering-tests/clustering-performance-test/
	# 1 flow / REST call
	pypy ./onos_tester.py --flows 100000 --timeout 10
	
	# 200 flows / REST call	
	pypy ./onos_tester.py --flows 100000 --timeout 10 --fpr 200 --bulk-delete