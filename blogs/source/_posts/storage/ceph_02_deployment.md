---
title: Ceph 02 deployment on Kubernetes
tags: storage
categories:
- storage
---

This document describes the steps to enable Ceph cluster in Kubernetes(k8s)  

## **Prerequisites**
1. A running Kubernetes environmen.  


## **Setup steps**
1. The first step is to deploy the Rook operator. Check that you are using the example yaml files that correspond to your release of Rook.  


	$ git clone --single-branch --branch release-1.3 https://github.com/rook/rook.git
	$ cd cluster/examples/kubernetes/ceph
	$ kubectl create -f common.yaml
	$ kubectl create -f operator.yaml
	
	Verify the rook-ceph-operator is in the Running state before proceeding
	$ kubectl -n rook-ceph get pod
2. Now that the Rook operator is running we can create the Ceph cluster. For the cluster to survive reboots, make sure you set the dataDirHostPath property that is valid for your hosts.  


	$ kubectl create -f cluster.yaml
3. Use kubectl to list pods in the rook-ceph namespace. You should be able to see the following pods once they are all running. The number of osd pods will depend on the number of nodes in the cluster and the number of devices configured. If you did not modify the cluster.yaml above, it is expected that one OSD will be created per node. The CSI, rook-ceph-agent, and rook-discover pods are also optional depending on your settings.  


	$ kubectl -n rook-ceph get pod
	  NAME                                                 READY   STATUS      RESTARTS   AGE
	  csi-cephfsplugin-provisioner-d77bb49c6-n5tgs         5/5     Running     0          140s
	  csi-cephfsplugin-provisioner-d77bb49c6-v9rvn         5/5     Running     0          140s
	  csi-cephfsplugin-rthrp                               3/3     Running     0          140s
	  csi-rbdplugin-hbsm7                                  3/3     Running     0          140s
	  csi-rbdplugin-provisioner-5b5cd64fd-nvk6c            6/6     Running     0          140s
	  csi-rbdplugin-provisioner-5b5cd64fd-q7bxl            6/6     Running     0          140s
	  rook-ceph-agent-4zkg8                                1/1     Running     0          140s
	  rook-ceph-crashcollector-minikube-5b57b7c5d4-hfldl   1/1     Running     0          105s
	  rook-ceph-mgr-a-64cd7cdf54-j8b5p                     1/1     Running     0          77s
	  rook-ceph-mon-a-694bb7987d-fp9w7                     1/1     Running     0          105s
	  rook-ceph-mon-b-856fdd5cb9-5h2qk                     1/1     Running     0          94s
	  rook-ceph-mon-c-57545897fc-j576h                     1/1     Running     0          85s
	  rook-ceph-operator-85f5b946bd-s8grz                  1/1     Running     0          92m
	  rook-ceph-osd-0-6bb747b6c5-lnvb6                     1/1     Running     0          23s
	  rook-ceph-osd-1-7f67f9646d-44p7v                     1/1     Running     0          24s
	  rook-ceph-osd-2-6cd4b776ff-v4d68                     1/1     Running     0          25s
	  rook-ceph-osd-prepare-node1-vx2rz                    0/2     Completed   0          60s
	  rook-ceph-osd-prepare-node2-ab3fd                    0/2     Completed   0          60s
	  rook-ceph-osd-prepare-node3-w4xyz                    0/2     Completed   0          60s
	  rook-discover-dhkb8                                  1/1     Running     0          140s
4. To verify that the cluster is in a healthy state, connect to the Rook toolbox and run the ceph status command.  

* All mons should be in quorum  
* A mgr should be active  
* At least one OSD should be active  
* If the health is not HEALTH_OK, the warnings or errors should be investigated.  

Running the Toolbox in Kubernetes:  

	$ kubectl create -f toolbox.yaml
Wait for the toolbox pod to download its container and get to the running state:

	$ kubectl -n rook-ceph get pod -l "app=rook-ceph-tools"
Once the rook-ceph-tools pod is running, you can connect to it with:

	$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash
Example:

	$ ceph status
	  cluster:
	    id:     a0452c76-30d9-4c1a-a948-5d8405f19a7c
	    health: HEALTH_OK
	
	  services:
	    mon: 3 daemons, quorum a,b,c (age 3m)
	    mgr: a(active, since 2m)
	    osd: 3 osds: 3 up (since 1m), 3 in (since 1m)
	...

When you are done with the toolbox, you can remove the deployment, but if you want continue to doing other test experiments below, please keep the toolbox:

	$ kubectl -n rook-ceph delete deployment rook-ceph-tools

## **Object Store**
### 1. Create an Object Store

	$ cd cluster/examples/kubernetes/ceph
	// Create the object store
	$ kubectl create -f object.yaml
	// To confirm the object store is configured, wait for the rgw pod to start
	$ kubectl -n rook-ceph get pod -l app=rook-ceph-rgw
### 2. Define a storage class that will allow object clients to create a bucket.

	// Define the storage class that will allow object clients to create a bucket
	$ kubectl create -f storageclass-bucket-delete.yaml
### 3.1 Create a Bucket
An Object Bucket Claim (OBC) is custom resource which requests a bucket (new or existing) and is described by a Custom Resource Definition (CRD).

	// Create an Object Bucket Claim (OBC) 
	$ kubectl create -f object-bucket-claim-delete.yaml
When the OBC is created, the Rook-Ceph bucket provisioner will create a new bucket.A `secret` and `ConfigMap` are created with the same name as the OBC and in the same namespace. The secret contains credentials used by the application pod to access the bucket. The ConfigMap contains bucket endpoint information and is also consumed by the pod. 

	// Check the created bucket name, "ceph-bkt-5124df41-7939-4aa6-b989-2bff0aa38deb" as shown below.
	$ kubectl describe obc ceph-delete-bucket
	  ......
	  Spec:
	    Object Bucket Name:    obc-default-ceph-delete-bucket
	    Bucket Name:           ceph-bkt-5124df41-7939-4aa6-b989-2bff0aa38deb
	    Generate Bucket Name:  ceph-bkt
	    Storage Class Name:    rook-ceph-delete-bucket
	  Status:
	    Phase:  Bound		// [Notice]: The Status must be Bound, if `Pending`, please wait or delete and recreate the obc.
	  Events:   <none>
An Object Bucket (OB) is a custom resource automatically generated when a bucket is provisioned. It is a global resource, typically not visible to non-admin users, and contains information specific to the bucket. It is described by an OB CRD.

	// Check the OB
	$ kubectl get ob
	  NAME                             AGE
	  obc-default-ceph-delete-bucket   5m16s
### 3.2 Create another Bucket
Change the ObjectBucketClaim metadata.name and create another bucket.

	$ vim object-bucket-claim-delete.yaml
	  apiVersion: objectbucket.io/v1alpha1
	  kind: ObjectBucketClaim
	  metadata:
	    name: ceph-bucket-new		# Change bucket name
	  spec:
	    generateBucketName: ceph-bkt
	    storageClassName: rook-ceph-bucket
	$ kubectl create -f object-bucket-claim-delete.yaml
**Client Connections**
The following commands extract key pieces of information from the secret and configmap:

	// config-map, secret, OBC will part of default if no specific name space mentioned
	$ export AWS_HOST=$(kubectl -n default get cm ceph-delete-bucket -o yaml | grep BUCKET_HOST | awk '{print $2}' | awk 'NR==1')
	$ export AWS_ACCESS_KEY_ID=$(kubectl -n default get secret ceph-delete-bucket -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | awk 'NR==1'| base64 --decode)
	$ export AWS_SECRET_ACCESS_KEY=$(kubectl -n default get secret ceph-delete-bucket -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | awk 'NR==1'| base64 --decode)

### 4. Consume the Object Storage
**Enter into the toolbox**

	kubectl exec $(kubectl get po -l app=rook-ceph-tools -n rook-ceph -o jsonpath='{.items[0].metadata.name}') -it -n rook-ceph -- bash
Connected to the Rook toolbox Pod and then set the four environment variables for use by your client(ie. inside the toolbox).

	$ export AWS_HOST=<host>
	$ export AWS_ENDPOINT=<endpoint>
	$ export AWS_ACCESS_KEY_ID=<accessKey>
	$ export AWS_SECRET_ACCESS_KEY=<secretKey>
Endpoint: The endpoint where the rgw service is listening. Run ```kubectl -n rook-ceph get svc rook-ceph-rgw-my-store```, then combine the clusterIP and the port.

**Install s3cmd**

	$ echo proxy=http://Proxy-Name:913 >> /etc/yum.conf
	$ yum --assumeyes install s3cmd
**PUT or GET an object**
Upload a file to the newly created bucket

	$ echo "Hello Rook" > /tmp/rookObj
	$ s3cmd put /tmp/rookObj --no-ssl --host=${AWS_HOST} --host-bucket=  s3://<-Your-Bucket-Name-> // As follow.
	// $ s3cmd put /tmp/rookObj --no-ssl --host=${AWS_HOST} --host-bucket=  s3://ceph-bkt-5124df41-7939-4aa6-b989-2bff0aa38deb
	  upload: '/tmp/rookObj' -> 's3://ceph-bkt-5124df41-7939-4aa6-b989-2bff0aa38deb/rookObj'  [1 of 1]
	   11 of 11   100% in    0s   190.11 B/s  done
Download and verify the file from the bucket

	$ s3cmd get s3://<-Your-Bucket-Name->/rookObj /tmp/rookObj-download --no-ssl --host=${AWS_HOST} --host-bucket=
	  download: 's3://ceph-bkt-5124df41-7939-4aa6-b989-2bff0aa38deb/rookObj' -> '/tmp/rookObj-download'  [1 of 1]
	   11 of 11   100% in    0s   254.34 B/s  done
	$ cat /tmp/rookObj-download
List the buckets

	$ s3cmd ls --no-ssl --host=${AWS_HOST} --host-bucket=

## Teardown
1. First you will need to clean up the resources created on top of the Rook cluster.  


	$ kubectl delete -n rook-ceph cephblockpool replicapool
	$ kubectl delete storageclass rook-ceph-block
	$ kubectl delete -f csi/cephfs/kube-registry.yaml
	$ kubectl delete storageclass csi-cephfs
2. Delete the CephCluster CRD


	$ kubectl -n rook-ceph delete cephcluster rook-ceph
3. Delete the Operator and related Resources


	$ kubectl delete -f operator.yaml
	$ kubectl delete -f common.yaml
4. /var/lib/rook: Path on each host in the cluster where configuration is cached by the ceph mons and osds. so need to clean the files in the path.


	$ rm -rf /var/lib/rook/
Additional: If there are ceph related files in the "/var/lib/kubelet/plugins/" and "/var/lib/kubelet/plugins_registry/" path, delete them.

	$ rm -rf /var/lib/kubelet/plugins/*
	$ rm -rf /var/lib/kubelet/plugins_registry/*
5. Delete the data on hosts


	#!/usr/bin/env bash
	DISK="/dev/sdb"
	# Zap the disk to a fresh, usable state (zap-all is important, b/c MBR has to be clean)
	# You will have to run this step for all disks.
	sgdisk --zap-all $DISK
	dd if=/dev/zero of="$DISK" bs=1M count=100 oflag=direct,dsync
	
	# These steps only have to be run once on each node
	# If rook sets up osds using ceph-volume, teardown leaves some devices mapped that lock the disks.
	ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %
	# ceph-volume setup can leave ceph-<UUID> directories in /dev (unnecessary clutter)
	rm -rf /dev/ceph-*
## FAQ
If the cluster resource still exists even though you have executed the delete command earlier, see the command below remove the finalizer.

	$ kubectl -n rook-ceph patch crd cephclusters.ceph.rook.io --type merge -p '{"metadata":{"finalizers": [null]}}'

## 遇到的问题
删除其它机器pvc绑定的/var/lib/kubelet/plugins/*出错

	$ rm -rf /var/lib/kubelet/plugins/*
	rm: cannot remove ‘/var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-d6ea6990-2a0b-4f4b-9838-3a971424732d/globalmount’: Device or resource busy
	rm: cannot remove ‘/var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-a08fed0b-b16d-4a74-a052-7936d6fb8340/globalmount’: Device or resource busy
解决方法:

	$ umount /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-d6ea6990-2a0b-4f4b-9838-3a971424732d/globalmount
	$ umount /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-a08fed0b-b16d-4a74-a052-7936d6fb8340/globalmount
再进行删除即可

