---
title: Greenplum 01 deployment on Kubernetes
tags: storage
categories:
- storage
---

## **Prerequisites**
1. [Kubernetes Node Configuration](http://greenplum-kubernetes.docs.pivotal.io/2-0/node-requirements.html) describes the Linux kernel configuration requirements for each Kubernetes node that is used in a Pivotal Greenplum cluster. These requirements are common to all Pivotal Greenplum deployments, regardless of which Kubernetes environment you use.
2. Ensure that any previous Pivotal Greenplum installation has been uninstalled as described in Uninstalling Pivotal Greenplum.
3. kubectl configured to refer to a Kubernetes cluster.

## **Install Greenplum Operator for Kubernetes**
1. Download the Pivotal Greenplum software from [VMware Tanzu Network](https://network.pivotal.io/products/greenplum-for-kubernetes) or skip to step 3 if have greenplum related files. The download file has the name: ```greenplum-for-kubernetes-<version>.tar.gz.```


2. Go to the directory where you downloaded Greenplum for Kubernetes, and unpack the downloaded software. For example:


	$ cd ~/Downloads
	$ tar xzf greenplum-for-kubernetes-*.tar.gz
The above command unpacks the distribution into a new directory named ```greenplum-for-kubernetes-<version>```.
3. Go into the new greenplum-for-kubernetes-<version> directory:


	$ cd ./greenplum-for-kubernetes-*
4. Load the Greenplum for Kubernetes Docker image to the local Docker registry:


	$ docker load -i ./images/greenplum-for-kubernetes
5. Load the Greenplum Operator Docker image to the Docker registry:


	$ docker load -i ./images/greenplum-operator
6. Push the Greenplum docker images to the local container registry. For example:


	$ IMAGE_REPO="hci-node01:5000"
	$ GREENPLUM_IMAGE_NAME="${IMAGE_REPO}/greenplum-for-kubernetes:$(cat ./images/greenplum-for-kubernetes-tag)"
	$ docker tag $(cat ./images/greenplum-for-kubernetes-id) ${GREENPLUM_IMAGE_NAME}
	$ docker push ${GREENPLUM_IMAGE_NAME}
	
	$ OPERATOR_IMAGE_NAME="${IMAGE_REPO}/greenplum-operator:$(cat ./images/greenplum-operator-tag)"
	$ docker tag $(cat ./images/greenplum-operator-id) ${OPERATOR_IMAGE_NAME}
	$ docker push ${OPERATOR_IMAGE_NAME}
7. Create a new YAML file in the workspace subdirectory with two lines to indicate the registry where you pushed the images


	cat <<EOF >workspace/operator-values-overrides.yaml
	operatorImageRepository: ${IMAGE_REPO}/greenplum-operator
	greenplumImageRepository: ${IMAGE_REPO}/greenplum-for-kubernetes
	operatorWorkerSelector: {
	greenplum-operator: "default"
	}
	EOF
8. Use helm to create a new Greenplum Operator release.


	// 先给某台机器添加标签来部署greenplum-operator
	$ kubectl label node hci-node02  greenplum-operator=default
	
	$ kubectl create namespace greenplum
	$ helm install -n greenplum-operator -f workspace/operator-values-overrides.yaml operator/ --namespace greenplum
	$ helm install greenplum-operator operator/

## **Create Local Persistent Volumes for Greenplum**
1. Create the directory, partition, or logical volume that you want to use as a Kubernetes local volume.


2. Create the StorageClass definition, specifying no-provisioner in order to manually provision local persistent volumes. Using volumeBindingMode: WaitForFirstConsumer is also recommended to delay binding the local PersistenVolume until a pod requires.


	$ vim gpdb-storage-class.yaml
	kind: StorageClass
	apiVersion: storage.k8s.io/v1
	metadata:
	  name: gpdb-storage
	provisioner: kubernetes.io/no-provisioner
	volumeBindingMode: WaitForFirstConsumer
3. Create a PersistentVolume definition, specifying the local volume and the required NodeAffinity field. For example:


	$ vim pv-master.yaml
	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: greenplum-master-node02
	spec:
	  capacity:
	    storage: 1Gi
	  accessModes:
	  - ReadWriteOnce
	  persistentVolumeReclaimPolicy: Retain
	  storageClassName: gpdb-storage
	  local:
	    path: /mnt/disks/greenplum-master-vol0	// 需要提前在相应机器上创建此目录
	  nodeAffinity:
	    required:
	      nodeSelectorTerms:
	      - matchExpressions:
	        - key: kubernetes.io/hostname
	          operator: In
	          values:
	          - hci-node02
	---
	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: greenplum-master-node02
	......重复上面的内容, 改变下local.path, nodeAffinity等在不同node机器上创建多个pv

4. Repeat the previous step for each PersistentVolume required for your cluster. Remember that each Greenplum segment host requires a dedicated storage volume.

5. Use kubectl to apply the StorageClass and PersistentVolume configurations that you created.

6. Specify the local storage StorageClass name when you deploy a new Greenplum cluster as below.

## **Deploy a greenplum cluster**
1. Go to the workspace subdirectory where you unpacked the Pivotal Greenplum distribution for Kubernetes:


	$ cd ./greenplum-for-kubernetes-*/workspace
2. If necessary, create a Kubernetes manifest file to specify the configuration of your Greenplum cluster. A sample file is provided in workspace/my-gp-instance.yaml. my-gp-instance.yaml contains the minimal set of instructions necessary to create a demonstration cluster named “my-greenplum” with a single segment and default storage, memory, and CPU settings:


	apiVersion: "greenplum.pivotal.io/v1"
	kind: "GreenplumCluster"
	metadata:
	  name: my-greenplum
	spec:
	  masterAndStandby:
	    hostBasedAuthentication: |
	      # host   all   gpadmin   1.2.3.4/32   trust
	      # host   all   gpuser    0.0.0.0/0   md5
	    memory: "800Mi"
	    cpu: "0.5"
	    storageClassName: gpdb-storage
	    storage: 1G
	    antiAffinity: "yes"
	    workerSelector: {}
	  segments:
	    primarySegmentCount: 2		# Expand the segment to 2
	    memory: "800Mi"
	    cpu: "0.5"
	    storageClassName: gpdb-storage		# Use the specify storageclass
	    storage: 10G				# Expand the storage to 10G
	    antiAffinity: "yes"
	    workerSelector: {}
	    mirrors: "yes"
3. Use kubectl apply command and specify your manifest file to send the deployment request to the Greenplum Operator. For example, to use the sample my-gp-instance.yaml file:


	$ kubectl apply -f ./my-gp-instance.yaml 
	  greenplumcluster.greenplum.pivotal.io/my-greenplum created
## **Deploy multiple greenplum cluster**
1. Create namespaces for greenplum cluster to deploy. Deploy two greenplum cluster instances:


	$ kubectl create namespace gpinstance-1
	$ kubectl create namespace gpinstance-2
2. Deploy Greenplum cluster into the correspond namespace.


	$ cd workspace
	$ kubectl apply -f ./my-gp-instance.yaml -n gpinstance-1
	$ kubectl apply -f ./my-gp-instance.yaml -n gpinstance-2
## **Test whether the Greenplum Cluster deployment is successful**


	$ kubectl exec -it master-0 -n greenplum -- bash -c "source /opt/gpdb/greenplum_path.sh; psql"
	 psql (8.3.23)
	 Type "help" for help.
	
	 gpadmin=# select * from gp_segment_configuration;
	如果报一些错误无法执行可以:
	1. 先进入master-0: kubectl exec -it master-0 -n greenplum -- bash, 再查找greenplum_path.sh
	2. 执行`$ source /opt/gpdb/greenplum_path.sh`, 再 `$ exit`退出, 然后就可以用以上命令了
(Enter `\q` to exit the psql utility.)

## **Delete a greenplum cluster and uninstall pivotal greenplum for kubernetes**

### **Delete a greenplum cluster**
1. Navigate to the workspace directory of the Pivotal Greenplum distribution (or to the location of the Kubernetes manifest that you used to deploy the cluster). For example:


	$ cd ./greenplum-for-kubernetes-*/workspace
2. Execute the kubectl delete command, specifying the manifest that you used to deploy the cluster. For example:


	$ kubectl delete -f ./my-gp-instance.yaml --wait=false
**Note:** Use the optional --wait=false flag to return immediately without waiting for the deletion to complete.
3. Use kubectl to describe the Greenplum cluster to verify Status.Phase and Events:


	$ kubectl describe greenplumcluster my-greenplum
	 [...]
	 Status:
	   Instance Image:    greenplum-for-kubernetes:latest
	   Operator Version:  greenplum-operator:latest
	   Phase:             Deleting
	 Events:
	   Type    Reason                    Age   From               Message
	   ----    ------                    ----  ----               -------
	   Normal  CreatingGreenplumCluster  3m    greenplumOperator  Creating Greenplum cluster my-greenplum in default
	   Normal  CreatedGreenplumCluster   1m    greenplumOperator  Successfully created Greenplum cluster my-greenplum in default
	   Normal  DeletingGreenplumCluster  6s    greenplumOperator  Deleting Greenplum cluster my-greenplum in default
If for any reason stopping the Greenplum instance fails, you should see a warning message in the greenplum-operator logs as shown below:


	$ kubectl logs -l app=greenplum-operator
	[...]
	{"level":"INFO","ts":"2020-01-24T19:03:22.874Z","logger":"controllers.GreenplumCluster","msg":"DeletingGreenplumCluster","name":"my-greenplum","namespace":"default"}
	{"level":"INFO","ts":"2020-01-24T19:03:23.068Z","logger":"controllers.GreenplumCluster","msg":"initiating shutdown of the greenplum cluster"}
	{"level":"INFO","ts":"2020-01-24T19:03:31.971Z","logger":"controllers.GreenplumCluster","msg":"gpstop did not stop cleanly. Please check gpAdminLogs for more info."}
	[...]
	{"level":"INFO","ts":"2020-01-24T19:03:32.252Z","logger":"controllers.GreenplumCluster","msg":"DeletedGreenplumCluster","name":"my-greenplum","namespace":"default"}
4. Use kubectl to monitor the progress of terminating Greenplum resources in your cluster. For example, if your cluster deployment was named my-greenplum:


	$ kubectl get all -l greenplum-cluster=my-greenplum
	$ kubectl label nodes hci-node01 greenplum-affinity-greenplum-master-	// 去掉node上关于greenplum的label
	$ kubectl label nodes hci-node01 greenplum-affinity-greenplum-segment-
### **Delete greenplum persistent volume claims**
**Caution:** If the Persistent Volumes were created using dynamic provisioning, then deleting the PVCs will also delete the associated PVs. In this case, do not delete the PVCs unless you are certain that you no longer need the data.
1. Verify that the PVCs are present for your cluster. For example, to show the Persistent Volume Claims created for a cluster named my-greenplum:


	$ kubectl get pvc -l greenplum-cluster=my-greenplum
2. Use kubectl to delete the PVCs associated with the cluster. For example, to delete all PersistentVolume Claims created for a cluster named my-greenplum:


	$ kubectl delete pvc -l greenplum-cluster=my-greenplum
3. If the Persistent Volumes were provisioned manually, then deleting the PVCs does not delete the associated PVs. (You can check for the PVs using kubectl get pv.) To delete any remaining Persistent Volumes, execute the command:


	$ kubectl delete pv -l greenplum-cluster=my-greenplum
### **Uninstall pivotal greenplum for kubernetes**
1. Use the helm delete command to delete the greenplum-operator release:


	$ helm delete greenplum-operator
	$ helm del --purge greenplum-operator;
2. Delete the node label of greenplum operator when deployed on kubernetes


	$ kubectl label nodes hci-node02 greenplum-operator-
3. Use docker rmi to delete images


	$ docker rmi <ImageName or ImgaeID>
## **FAQ**
When uninstall greenplum, encountered this problem: Object is being deleted: customresourcedefinitions.apiextensions.k8s.io "greenplumclusters.greenplum.pivotal.io" already exists.

solution refer to this:[delete crd](https://github.com/kubernetes/kubernetes/issues/60538)


