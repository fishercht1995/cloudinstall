# Build a cluster for own-designed k8s scheduler

## set up kubernetes clusters

git clone https://github.com/fishercht1995/cloudinstall.git

### in master node

deploy k8s system
```
./cloudinstall/kubernetes/deploy.sh
```
init k8s in the master node, and get `token`
```
./cloudinstall/kubernetes/init.sh
```

### in worker node
deploy k8s system
```
./cloudinstall/kubernetes/deploy.sh
```
Using Token get in the first step, so the worker will join into k8s cluster

## set up prometheus and grafana monitor system

### in worker
```
./cloudinstall/pgmonitor/slave.sh
```
### in master
```
./cloudinstall/pgmonitor/master.sh
```

## set up nfs system

### in both master and worker nodes
get externel ip address
```
dig +short myip.opendns.com @resolver1.opendns.com
```

### on master
```
sudo apt-get update
sudo apt install nfs-kernel-server
sudo mkdir -p /mnt/linuxidc
sudo chown nobody:nogroup /mnt/linuxidc
sudo chmod 777 /mnt/linuxidc

sudo nano /etc/exports

/mnt/linuxidc client1IP(rw,sync,no_subtree_check)
/mnt/linuxidc client2IP(rw,sync,no_subtree_check)

sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### on worker
```
sudo apt-get update
sudo apt-get install nfs-common
sudo mkdir -p /mnt/linuxidc_client

sudo mount master_ip:/mnt/linuxidc /mnt/linuxidc_client

```

## set up configuration in kubernetes

### step 1

```
kubectl edit clusterrole system:kube-scheduler
```

add `my-scheduler` in `resourceNames:`


add after config file 
```
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - watch
  - list
  - get
```

### step 2

change `pv` config for its service ip address

### step 3

change `assistance pod` config for their node name

adjust the number of assistance pod

### step 4

change `scheduler pod` config for its node name
