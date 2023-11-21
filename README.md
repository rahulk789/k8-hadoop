# hadoop-on-k8s
Kubernetes manifest files for building Hadoop clusters

This repository hosts the kubernetes manifest files to deploy multi-node hadoop clusters on Kubernetes. This project uses the docker image files created using eiswar/hadoop-on-docker project.

# Steps to setup the Hadoop cluster on Kubernetes

1. Create the Hadoop docker image using eiswar/hadoop-on-docker project.
2. Checkout this repo or download https://raw.githubusercontent.com/eiswar/hadoop-on-k8s/master/hadoop.yml
3. Update the image name in the manifest file.
4. Setup storage
5. Execute the following command to create the Hadoop cluster.

`kubectl create -f https://raw.githubusercontent.com/eiswar/hadoop-on-k8s/master/hadoop.yml`

or

`kubectl create -f hadoop.yml`

# Steps to setup storage

1. NameNode deployment uses a PersistentVolumeClaim. Make sure that PersistentVolumes are configured with the Kubernetes cluster. The PVC will for namenode data will be created automatically.
2. DataNode deployment uses hostpath. Datanode and node-manager are deployed as Kubernetes daemonset. So, the datanode and node-manager will be running on all the nodes. All the nodes should have a mount point /hdfs. If there are multiple disks, the disks can be added to a logical volume group(LVM) and it can be mounted on /hdfs mount point.

# How it works

1. The single Kubernetes manifest file https://raw.githubusercontent.com/eiswar/hadoop-on-k8s/master/hadoop.yml will do the following things:
- Create a service account in Kubernetes for Hadoop operations.
- Create a namespace in Kubernetes for Hadoop operations.
- Create a role, cluster role, role-bindings, etc to grant privileges to the hadoop service account
- Create a PVC for namenode data and it will bind the PVC with the hadoop-master pod
- Create a service that will point the hostname hadoop-master.${namespace}.svc.cluter.local to the hadoop-master pods.
- Create a deployment for runnng hadoop-master pods.
- Create a daemonset for running hadoop-worker pods on all the nodes in the cluster.
- The namenode data will be saved in the PVC, so that when the hadoop-master pod fails, a new one will be created and the same PVC will be mounted to the new pod.
- The datanode data will be saved in the /hdfs directory on each node.
- The datanode daemonset uses host network, so the pods get the same IP address all the times.
2. Once the hadoop-master pod is created, it will automatically creates a kubernetes secret for storing and retreiving authorized keys to allow master to connect to worker pods using SSH passwordless authentication.
3. The hadoop worker pods will pull the authorized keys that were generated by masters and it will setup SSH passwordless authentication automatically.
4. Since host network is used for Hadoop worker pods, all the worker pods will get static hostnames and IP addresses.

# To do

1. Enable namenode High Availablity
2. Enable HDFS federation