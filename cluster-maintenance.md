#drain node
kubectl drain <nodename> --ignore-daemonsets //first try without ignore daemonsets~
of there this you have to uncordon kubectl uncordon <nodename>
#every drained node is also cordoned by default (unschedulable)
#Mark node03 as unschedulable but do not remove any apps currently running on it .
kubectl cordon <nodename> #this doesnt remove any running pods
#if the node has any pods that are not part of replicaset you have to force it
kubectl drain <nodename> --ignore-daemonsets --force (the pods will not be rescheduled though)


#cluster version and upgrade
## check current version
kubectl get nodes #check version sof every node
kubeadm upgrade plan #shows current version and upgrade options
## upgrade master
apt install kubeadm=1.17.4-00
apt-get upgrade kubeadm=1.17.4-00
kubeadm upgrade apply v1.17.0 #you shouldnt stop kubelet while doing this, it has to be running
kubeadm upgrade apply v1.17.0 --ignore-preflight-errors ControlPlaneNodesReady -may be potentially required to ignore errors for drained node
apt install kubelet=1.17.0-00
service restart kubelet

##upgrade worker nodes
apt install kubeadm=1.17.0-00 and then 
apt install kubelet=1.17.0-00 and then 
kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
service restart kubelet

#etcd cluster
kubectl describe pod etcd-master -n kube-system and look for --listen-client-urls for cluster url
#backup cluster
##resource configuration
- kubectl get all --all-namespaces -o yaml > back.yaml 
- Heptio

##etcd cluster
take snapshot: 
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /tmp/snapshot-pre-boot.db
status of snapshot: ECTDCTL_API=3 etcdtl snapshot status <snapshotname.db>
to restore:
1. first stop kube-apiserver - service kube-apiserver stop
2. restore snapshot (creates a new etcd clsuter) - etcdctl snapshot restore <snapshot.db> --data-dir /var/lib/etcd-from-backup --initial-cluster <clusterurl> --initial-cluster-token etcd-cluster-1
3. systemctl daemon-reload
4. service etcd start
5. service kube-apiserver start