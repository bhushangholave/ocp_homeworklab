************************************************************************************************************************************************************************************************************
*****************************************************Openshift 3.9 Homework Lab Assignment******************************************************************************************************************
************************************************************************************************************************************************************************************************************

	- Lab created for homework:- OTLC-LAB-bgholave-whitehedge.com-PROD_OCP_HA_HOMEWORK_LAB-e5d7
	- Started assignment:-
	- git repository used for this : https://github.com/bhushangholave/ocp_homeworklab
	- Folders 2.1,2.2 contains inventory hosts file and 2.3 contains snapshots of cicd workflow 
	- Folder 2.4 contains only readme file for 2.4 section of the assignment

********************************************************************************************************************************************************

****************************************************************************************************************************************************
**********************************2.1. Basic and HA Requirements- folder name "2.1"*****************************************************************
*****************************************************************************************************************************************************

	a. clone https://github.com/bhushangholave/ocp_homeworklab.git

	b. Look into 2.1/host.nfs file for first point in assignment- inventory file with default network policy

	c. https://loadbalancer1.e5d7.example.opentlc.com/console/ 
	   Web console is reachable and can login with
	 user payment1 and password: r3dh4t1!
	 
	d. Registry has storage attached -registry volume with 20Gi space and default/registry-claim

	e. router is configured on each infra node

	f. 50 PVs of different types are available for users to consume- 25 are readwriteonce 5GB and 25 are readwritemany 10GB

	g. Deployed simple nodejs-mongodb app under project - "project-testapp"
	  app-name-"nodejs-mongo-persistent"
	  
	h.3 ansible masters are working 
	  ansible masters --list
	  hosts (3):
	    master1.e5d7.internal
	    master2.e5d7.internal
	    master3.e5d7.internal
	    
	i. 3 etcd are working 
	ansible etcd --list
	  hosts (3):
	    master1.e5d7.internal
	    master2.e5d7.internal
	    master3.e5d7.internal
	    
	j. There is loadbalancer to access master called loadbalancer1.e5d7.internal

	k. There are at least two infranodes, labeled env=infra

	infranode1.e5d7.internal openshift_hostname=infranode1.e5d7.internal  openshift_node_labels="{'env':'infra', 'cluster': 'e5d7'}"

	infranode2.e5d7.internal openshift_hostname=infranode2.e5d7.internal  openshift_node_labels="{'env':'infra', 'cluster': 'e5d7'}"

	l. DNS for infranodes:- nodejs-mongo-persistent-project-testapp.apps.e5d7.example.opentlc.com

	oc get route

	NAME                      HOST/PORT                                                               PATH      SERVICES                  PORT      TERMINATION   WILDCARD

	nodejs-mongo-persistent   nodejs-mongo-persistent-project-testapp.apps.e5d7.example.opentlc.com             nodejs-mongo-persistent   <all>                   None

	m. Application running on http://nodejs-mongo-persistent-project-testapp.apps.e5d7.example.opentlc.com/

****************************************************************************************************************************************************
**********************************2.2. Environment Configuration************************************************************************************
****************************************************************************************************************************************************

	a. chaged os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant' in inventory file
	   look into 2.2/host.nfs file for change in inventory file for multitenancy

	b. Run separate ansible commands to change the plug-in and restart the services
	 	following are the commands executed
		# reconfigure all the config files
		ansible masters -m shell -a "sed -i -e 's/openshift-ovs-subnet/openshift-ovs-multitenant/g' /etc/origin/master/master-config.yaml"
		ansible nodes -m shell -a "sed -i -e 's/openshift-ovs-subnet/openshift-ovs-multitenant/g' /etc/origin/node/node-config.yaml"

		# Stop all the services
		ansible masters -m shell -a'systemctl stop atomic-openshift-master-api'
		ansible masters -m shell -a'systemctl stop atomic-openshift-master-controllers'
		ansible nodes -m shell -a'systemctl stop atomic-openshift-node'

		# restart openvswitch
		ansible nodes -m shell -a'systemctl restart openvswitch'

		# start all the services
		ansible masters -m shell -a'systemctl start atomic-openshift-master-api'
		ansible masters -m shell -a'systemctl start atomic-openshift-master-controllers'
		ansible nodes -m shell -a'systemctl start atomic-openshift-node'
		
	c. Aggregate logging is configured and working

		## Enable cluster logging
		openshift_logging_install_logging=True
		#
		openshift_logging_storage_kind=nfs
		openshift_logging_storage_access_modes=['ReadWriteOnce']
		openshift_logging_storage_nfs_directory=/srv/nfs
		openshift_logging_storage_nfs_options='*(rw,root_squash)'
		openshift_logging_storage_volume_name=logging
		openshift_logging_storage_volume_size=10Gi
		openshift_logging_storage_labels={'storage': 'logging'}
		
	d. Metrics collection is configured and working

		## Enable cluster metrics
		openshift_metrics_install_metrics=True
		#
		openshift_metrics_storage_kind=nfs
		openshift_metrics_storage_access_modes=['ReadWriteOnce']
		openshift_metrics_storage_nfs_directory=/srv/nfs
		openshift_metrics_storage_nfs_options='*(rw,root_squash)'
		openshift_metrics_storage_volume_name=metrics
		openshift_metrics_storage_volume_size=10Gi
		openshift_metrics_storage_labels={'storage': 'metrics'}
		
	e. Router and Registry Pods run on Infranodes

		NAMESPACE                           NAME                                      READY     STATUS      RESTARTS   AGE       IP              NODE
		default                             docker-registry-1-nrzpx                   1/1       Running     0          3h        10.129.0.3      infranode1.e5d7.internal

		default                             registry-console-1-mxmdh                  1/1       Running     0          3h        10.129.0.4      infranode1.e5d7.internal

		default                             router-1-5lp24                            1/1       Running     0          3h        192.199.0.204   infranode2.e5d7.internal

		default                             router-1-dw54n                            1/1       Running     0          3h        192.199.0.114   infranode1.e5d7.internal

	f. Metrics and Logging components run on Infranodes

		NAMESPACE                           NAME                                      READY     STATUS      RESTARTS   AGE       IP              NODE

		logging                             logging-curator-1-pr58d                   1/1       Running     0          3h        10.130.0.9      infranode2.e5d7.internal

		logging                             logging-es-data-master-8ts36w2o-1-wsmvd   2/2       Running     0          3h        10.130.0.12     infranode2.e5d7.internal

		logging                             logging-fluentd-2knbn                     1/1       Running     0          3h        10.129.0.7      infranode1.e5d7.internal

		logging                             logging-fluentd-btwlv                     1/1       Running     0          3h        10.130.0.10     infranode2.e5d7.internal

		logging                             logging-kibana-1-jfhlg                    2/2       Running     0          3h        10.130.0.7      infranode2.e5d7.internal

		openshift-metrics                   prometheus-0                              6/6       Running     0          3h        10.129.0.8      infranode1.e5d7.internal

		openshift-metrics                   prometheus-node-exporter-95tvf            1/1       Running     0          3h        192.199.0.114   infranode1.e5d7.internal

		openshift-metrics                   prometheus-node-exporter-z4v9g            1/1       Running     0          3h        192.199.0.204   infranode2.e5d7.internal

	g. Service Catalog, Template Service Broker, and Ansible Service Broker are all working

	Service Catalog

		NAMESPACE                           NAME                                      READY     STATUS      RESTARTS   AGE       IP              NODE

		kube-service-catalog                apiserver-dgb2s                           1/1       Running     0          2h        10.130.2.31     master3.e5d7.internal

		kube-service-catalog                apiserver-fz9bg                           1/1       Running     0          2h        10.131.0.4      master2.e5d7.internal

		kube-service-catalog                apiserver-wrbbs                           1/1       Running     0          2h        10.128.0.7      master1.e5d7.internal

		kube-service-catalog                controller-manager-bmvwt                  1/1       Running     0          2h        10.130.2.32     master3.e5d7.internal

		kube-service-catalog                controller-manager-bpcpm                  1/1       Running     3          2h        10.128.0.8      master1.e5d7.internal

		kube-service-catalog                controller-manager-d5558                  1/1       Running     0          2h        10.131.0.5      master2.e5d7.internal

	Template Service Broker and Ansible Service Broker

		openshift-ansible-service-broker    asb-1-qdts2                               1/1       Running     0          2h        10.128.2.3      node1.e5d7.internal

		openshift-ansible-service-broker    asb-etcd-1-bvkw6                          1/1       Running     0          2h        10.128.4.3      node4.e5d7.internal

		openshift-template-service-broker   apiserver-64xdz                           1/1       Running     0          2h        10.130.0.13     infranode2.e5d7.internal

		openshift-template-service-broker   apiserver-rk6pv                           1/1       Running     0          2h        10.129.0.9      infranode1.e5d7.internal
	
	

****************************************************************************************************************************************************
**********************************2.3. CICD Workflow************************************************************************************************
****************************************************************************************************************************************************
	a. Referred github repo for CICD demo

	https://github.com/OpenShiftDemos/openshift-cd-demo

	b. provisioned jenkins,gogs,nexus and sonarqube using the script provision.sh

	c. Pods deployed after successful completion

		NAME        HOST/PORT                                             PATH      SERVICES    PORT       TERMINATION     WILDCARD
		che         che-cicd-admin1.apps.e5d7.example.opentlc.com                   che-host    <all>                      None
		gogs        gogs-cicd-admin1.apps.e5d7.example.opentlc.com                  gogs        <all>                      None
		jenkins     jenkins-cicd-admin1.apps.e5d7.example.opentlc.com               jenkins     <all>      edge/Redirect   None
		nexus       nexus-cicd-admin1.apps.e5d7.example.opentlc.com                 nexus       8081-tcp                   None
		sonarqube   sonarqube-cicd-admin1.apps.e5d7.example.opentlc.com             sonarqube   <all>                      None

	d. 


	    A Jenkins pipeline is pre-configured which clones Tasks application source code from Gogs (running on OpenShift), builds, deploys and promotes the result through the deployment pipeline. In 		    the CI/CD project, click on Builds and then Pipelines to see the list of defined pipelines.

	    Click on tasks-pipeline and Configuration and explore the pipeline definition.

	    You can also explore the pipeline job in Jenkins by clicking on the Jenkins route url, logging in with the OpenShift credentials and clicking on tasks-pipeline and Configure.

	    Run an instance of the pipeline by starting the tasks-pipeline in OpenShift or Jenkins.

	    During pipeline execution, verify a new Jenkins slave pod is created within CI/CD project to execute the pipeline.

	    Pipelines pauses at Deploy STAGE for approval in order to promote the build to the STAGE environment. Click on this step on the pipeline and then Promote.

	    After pipeline completion, demonstrate the following:

	    Explore the snapshots repository in Nexus and verify openshift-tasks is pushed to the repository
	    Explore SonarQube and show the metrics, stats, code coverage, etc
	    Explore Tasks - Dev project in OpenShift console and verify the application is deployed in the DEV environment
	    Explore Tasks - Stage project in OpenShift console and verify the application is deployed in the STAGE environment

	e. Dev environment is up and running 

		[root@bastion ~]# oc get route -n dev-admin1 

		NAME      HOST/PORT                                        PATH      SERVICES   PORT       TERMINATION   WILDCARD

		tasks     tasks-dev-admin1.apps.e5d7.example.opentlc.com             tasks      8080-tcp                 None

	f. Stage environment is up and running

		[root@bastion ~]# oc get route -n stage-admin1 

		NAME      HOST/PORT                                          PATH      SERVICES   PORT       TERMINATION   WILDCARD

		tasks     tasks-stage-admin1.apps.e5d7.example.opentlc.com             tasks      8080-tcp                 None

	g. Git clone eap-7 branch of OpenShiftDemos/openshift-tasks.git by forking it in my account.

	Commit my change and pushed it into forked repo and sent pull request with my repo https://github.com/bhushangholave/openshift-tasks.git

	h. Observed change in jenkins pipeline with error for unit test then made changes again and unit test passed.
              
              Jenkins pod is running with a persistent 
		
              Jenkins deploys openshift-tasks app

	      Jenkins OpenShift plugin is used to create a CICD workflow

	      HPA is configured and working on production deployment of openshift-tasks
	
	i. git repository folder 2.3/ contains screenshots of the applications running for this assignment

****************************************************************************************************************************************************
**********************************2.4 Multitenancy**************************************************************************************************
****************************************************************************************************************************************************

	a. Create 3 new projects with names alphacorp,betacorp,common
		oc get projects
		NAME                                DISPLAY NAME          STATUS

		alphacorp                           client- alpha         Active

		betacorp                            client- beta          Active

		cicd-admin1                         CI/CD                 Active

		common                              client- unspecified   Active

		default                                                   Active

	b. [root@bastion ~]# oc adm groups new alpha amy andrew

	c. [root@bastion ~]# oc adm groups new beta brian betty
	
	d. [root@bastion ~]# oc label group/alpha client=alpha

 	   group "alpha" labeled

	e. [root@bastion ~]# oc label group/beta client=beta
	
	  group "beta" labeled
	
	f. adding content to project 
		oc new-app openshift/hello-openshift:v1.1.1.1 -n alphacorp
	
	g. adding content to project 
		oc new-app openshift/hello-openshift:v1.1.1.1 -n betacorp 
	
	h. Pods running for both apps
		[root@bastion ~]# oc get pod --all-namespaces | grep hello-openshift
		alphacorp                           hello-openshift-1-s46r4                   1/1       Running     0          4m

		betacorp                            hello-openshift-1-pcm9z                   1/1       Running     0          3m
	
	i. each client pods are running on dedicated nodes

		[root@bastion ~]# oc describe pod  -n alphacorp hello-openshift-1-s46r4 | egrep 'IP|Node:'

		Node:           node3.e5d7.internal/192.199.0.237

		IP:             10.129.2.29
	
		[root@bastion ~]# oc describe pod  -n betacorp hello-openshift-1-pcm9z | egrep 'IP|Node:'

		Node:           node4.e5d7.internal/192.199.0.58

		IP:             10.128.4.24
	
	j. each node labeled for specific client
		
		[root@bastion ~]# oc label node node4.e5d7.internal client=beta
	
		node "node4.e5d7.internal" labeled

		[root@bastion ~]# oc label node node3.e5d7.internal client=alpha

		node "node3.e5d7.internal" labeled

	k. change project request limit per label
		ssh master1.$GUID.internal
		sudo -i
		
		vim /etc/origin/master/master-config.yaml

	   insert following lines

		admissionConfig:

		  pluginConfig:

		    ProjectRequestLimit:

		      configuration:

			apiVersion: v1

			kind: ProjectRequestLimitConfig

			limits:

			- selector:

				client: alpha

			  maxProjects: 5

			- selector:
				client: beta

			  maxProjects: 5
	   
	    systemctl restart atomic-openshift-master-api atomic-openshift-master-controllers

	l. setting project wide node selector for alphacorp and betacorp using patch command

		[root@bastion ~]# oc patch namespace alphacorp -p '{"metadata":{"annotations":{"openshift.io/node-selector":"client=alpha"}}}'

		[root@bastion ~]# oc patch namespace betacorp -p '{"metadata":{"annotations":{"openshift.io/node-selector":"client=beta"}}}'
	M. limit are set using template 

		echo '{
	
		    "kind": "LimitRange",

		    "apiVersion": "v1",

		    "metadata": {

			"name": "limits",

			"creationTimestamp": null

		    },

		    "spec": {

			"limits": [

			    {

				"type": "Pod",

				"max": {

				    "cpu": "100m",

				    "memory": "750Mi"

				},

				"min": {

				    "cpu": "10m",

				    "memory": "5Mi"

				}

			    },

			    {
				
				"type": "Container",

				"max": {

				    "cpu": "100m",

				    "memory": "750Mi"

				},
				"min": {

				    "cpu": "10m",

				    "memory": "5Mi"

				},

				"default": {

				    "cpu": "50m",

				    "memory": "100Mi"

				}
			    }
			]
		    }

		}' | oc create -f -
	modify using template
		The new user template is used to create a user object with the specific label value
	
	Modify the master-config.yaml file to reference the loaded template:...

		projectConfig:

		 projectRequestTemplate: "default/project-request"

		 ...
	
	[root@bastion ~]# oc get limits
	NAME      AGE
	limits    19s

	N. New Client documentation
		i. new project created for new client say myproject then while creating project we will slect a node
			
		[root@bastion ~]# oc adm new-project myproject \
					--node-selector='client=myproject'
		

		ii. Add new group with users for client 

		 oc adm groups new myproject xyz 
	
		iii. Add label fro group with name

		oc label group/myproject client=myproject
	
		iv. Add lable to node for new client

		[root@bastion ~]# oc label node node1.e5d7.internal client=myproject

		v. Change project request limit per client

		vim /etc/origin/master/master-config.yaml

		systemctl restart atomic-openshift-master-api atomic-openshift-master-controllers

		vi. Join the project myproject with other project for multitenency
	
		vii. can create user with label 

			For example, if the user name is theuser and the label is level=gold:$ oc label myproject/theuser client=myproject
	o. oc get pod -n alphacorp $(oc get pod -n alphacorp | grep hello-openshift | awk '{print $1}') -o template --template '{{.status.podIP}}{{"\n"}}'
	
		10.129.2.29
	
	p. oc get pod -n betacorp $(oc get pod -n betacorp | grep hello-openshift | awk '{print $1}') -o template --template '{{.status.podIP}}{{"\n"}}'
		
		10.128.4.24

	q.  oc rsh -n betacorp $(oc get pod -n betacorp | grep shelly | awk '{print $1}')
		
		sh-4.2$ curl 10.129.2.29:8080 -m 1

		curl: (28) Connection timed out after 1001 milliseconds

		sh-4.2$ 

		sh-4.2$ curl 10.128.4.24:8080 -m 1

		Hello OpenShift!

	    oc rsh -n alphacorp $(oc get pod -n alphacorp | grep shelly | awk '{print $1}')
	
		sh-4.2$ curl 10.129.2.29:8080 -m 1

		Hello OpenShift!

		sh-4.2$ curl 10.128.4.24:8080 -m 1

		curl: (28) Connection timed out after 1000 milliseconds
		sh-4.2$ 

	r. Join projects

		[root@bastion ~]# oc adm pod-network join-projects --to=betacorp alphacorp
		
		[root@bastion ~]# oc get netnamespaces | grep corp

			alphacorp                           1877315    []

			betacorp                            1877315    []
		
		oc rsh -n betacorp $(oc get pod -n betacorp | grep shelly | awk '{print $1}')
		
		[root@bastion ~]# oc rsh -n betacorp $(oc get pod -n betacorp | grep shelly | awk '{print $1}')

		sh-4.2$ curl 10.129.2.29:8080 -m 1

		Hello OpenShift!

		sh-4.2$ curl 10.128.4.24:8080 -m 1

		Hello OpenShift!

		sh-4.2$ 
	
	[root@bastion ~]# oc describe service hello-openshift

			Name:              hello-openshift

			Namespace:         alphacorp

			Labels:            app=hello-openshift

			Annotations:       openshift.io/generated-by=OpenShiftNewApp

			Selector:          app=hello-openshift,deploymentconfig=hello-openshift

			Type:              ClusterIP

			IP:                172.30.75.251

			Port:              8080-tcp  8080/TCP

			TargetPort:        8080/TCP

			Endpoints:         10.129.2.29:8080

			Port:              8888-tcp  8888/TCP

			TargetPort:        8888/TCP

			Endpoints:         10.129.2.29:8888

			Session Affinity:  None

			Events:            <none>
	
	session Affinity change to client ip
			Name:              hello-openshift
			Namespace:         servicelayer

			Labels:            app=hello-openshift

			Annotations:       openshift.io/generated-by=OpenShiftNewApp

			Selector:          app=hello-openshift,deploymentconfig=hello-openshift

			Type:              ClusterIP

			IP:                172.30.162.40

			Port:              8080-tcp  8080/TCP

			TargetPort:        8080/TCP

			Endpoints:         10.1.10.10:8080,10.1.4.11:8080,10.1.8.14:8080 + 1 more...

			Port:              8888-tcp  8888/TCP

			TargetPort:        8888/TCP

			Endpoints:         10.1.10.10:8888,10.1.4.11:8888,10.1.8.14:8888 + 1 more...

			Session Affinity:  ClientIP

			Events:            <none>

******************************************************************************************************************************************************************************************************************************************************************************************************************	
		


	
	







