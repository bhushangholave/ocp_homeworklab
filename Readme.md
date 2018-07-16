2.1. Basic and HA Requirements- folder name "2.1"

a. clone https://github.com/bhushangholave/ocp_homeworklab.git
b. Look into host.nfs file for first point in assignment- inventory file with default network policy-subnet
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

2.2. Environment Configuration

a. chnaged os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant' in inventory file
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





