# ansible-kubernetes-cluster

https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04
https://medium.com/@salqadri/build-your-own-multi-node-kubernetes-cluster-with-monitoring-346a7e2ef6e2


vagrant up --provision-with file

ansible-playbook -i hosts -e master_ip=10.0.10.10 kubernetes-cluster.yml
