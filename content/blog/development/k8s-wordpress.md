---
title: k8s wordpress
date: 2020-01-21 10:01:81
category: development
draft: false
---

### k8s wordpress

- os: mac os
- kubernetes version: 1.17

##### 0. kubectl, minikube 설치

```bash
# kubectl 설치
$ brew install kubectl

# minikube 설치
$ brew cask install minikube

# minikube ip 확인
$ minikube ip
```

##### 1. secret 생성

```bash
# minikube 시작
$ minikube start
🎉  minikube 1.6.2 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.6.2
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🙄  minikube v1.6.1 on Darwin 10.15.2
✨  Automatically selected the 'hyperkit' driver (alternates: [virtualbox])
💾  Downloading driver docker-machine-driver-hyperkit:
    > docker-machine-driver-hyperkit.sha256: 65 B / 65 B [---] 100.00% ? p/s 0s
    > docker-machine-driver-hyperkit: 10.81 MiB / 10.81 MiB  100.00% 1.04 MiB p
🔑  The 'hyperkit' driver requires elevated permissions. The following commands will be executed:

    $ sudo chown root:wheel /Users/evanjin/.minikube/bin/docker-machine-driver-hyperkit
    $ sudo chmod u+s /Users/evanjin/.minikube/bin/docker-machine-driver-hyperkit


Password:
🔥  Creating hyperkit VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.17.0 on Docker '19.03.5' ...
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for cluster to come online ...
🏄  Done! kubectl is now configured to use "minikube"
⚠️  /usr/local/bin/kubectl is version 1.14.8, and is incompatible with Kubernetes 1.17.0. You will need to update /usr/local/bin/kubectl or use 'minikube kubectl' to connect with this cluster

$ minikube kubectl
💾  Downloading kubectl v1.17.0
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create         Create a resource from a file or from stdin.
  expose         Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run            Run a particular image on the cluster
  set            Set specific features on objects

Basic Commands (Intermediate):
  explain        Documentation of resources
  get            Display one or many resources
  edit           Edit a resource on the server
  delete         Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout        Manage the rollout of a resource
  scale          Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale      Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate    Modify certificate resources.
  cluster-info   Display cluster info
  top            Display Resource (CPU/Memory/Storage) usage.
  cordon         Mark node as unschedulable
  uncordon       Mark node as schedulable
  drain          Drain node in preparation for maintenance
  taint          Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe       Show details of a specific resource or group of resources
  logs           Print the logs for a container in a pod
  attach         Attach to a running container
  exec           Execute a command in a container
  port-forward   Forward one or more local ports to a pod
  proxy          Run a proxy to the Kubernetes API server
  cp             Copy files and directories to and from containers.
  auth           Inspect authorization

Advanced Commands:
  diff           Diff live version against would-be applied version
  apply          Apply a configuration to a resource by filename or stdin
  patch          Update field(s) of a resource using strategic merge patch
  replace        Replace a resource by filename or stdin
  wait           Experimental: Wait for a specific condition on one or many resources.
  convert        Convert config files between different API versions
  kustomize      Build a kustomization target from a directory or a remote url.

Settings Commands:
  label          Update the labels on a resource
  annotate       Update the annotations on a resource
  completion     Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-resources  Print the supported API resources on the server
  api-versions   Print the supported API versions on the server, in the form of "group/version"
  config         Modify kubeconfig files
  plugin         Provides utilities for interacting with plugins.
  version        Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

$ minikube addons enable ingress
✅  ingress was successfully enabled

$ echo -n test-password > ./password.txt

$ kubectl create secret generic mysql-pass --from-file=./password.txt
secret/mysql-pass created

$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-nvpw8   kubernetes.io/service-account-token   3      95s
mysql-pass            Opaque                                1      20s

$ kubectl describe secret mysql-pass
Name:         mysql-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  13 bytes
```

##### 2. volume 만들기

`local-volume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-volume
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/test-lv
  persistentVolumeReclaimPolicy: Recycle
```

```bash
$ kubectl create -f local-volume.yaml
persistentvolume/local-volume created

$ kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
local-volume   2Gi        RWO            Recycle          Available                                   8s
```

##### 3. mysql pod 실행

`mysql.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-lv-claim
  labels:
    app: mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password.txt
          ports:
            - containerPort: 3306
              name: mysql
          resources:
            requests:
              cpu: 50m
          volumeMounts:
            - name: mysql-local-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-local-storage
          persistentVolumeClaim:
            claimName: mysql-lv-claim
```

```bash
$ kubectl create -f mysql.yaml
service/mysql created
persistentvolumeclaim/mysql-lv-claim created
deployment.apps/mysql created

$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
mysql-78b898485-qmg74   1/1     Running   0          59s
```

```bash
$ kubectl exec -it mysql-78b898485-qmg74 -- bash
root@mysql-78b898485-qmg74:/# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.47 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

mysql> exit
Bye
root@mysql-78b898485-qmg74:/# exit
exit
command terminated with exit code 127
```

#### 4. 워드프레스 앱 실행

`wordpress.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30180
  selector:
    app: wordpress
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
        - image: wordpress:latest
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: mysql
              # - name: WORDPRESS_DB_USER
              #   value: admin
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password.txt
          ports:
            - containerPort: 80
              name: wordpress
          resources:
            requests:
              cpu: 50m
```

```bash
$ kubectl apply -f wordpress.yaml
service/wordpress created
deployment.apps/wordpress created

$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mysql-78b898485-qmg74        1/1     Running   0          28m
wordpress-6f6f54bc64-hjxlv   1/1     Running   0          15m

$ minikube service wordpress --url
http://192.168.64.2:30180
```

위에 나오는 url로 접속하여 워드프레스 설치화면이 나온다면 정상적으로 동작한 것입니다.
