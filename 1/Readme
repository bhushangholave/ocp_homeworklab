
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



