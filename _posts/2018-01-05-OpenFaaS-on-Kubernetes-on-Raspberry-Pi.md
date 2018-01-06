---
title: "OpenFaaS on Kubernetes on Raspberry-Pi"
date: 2018-01-05
layout: "post"
categories: faas, function as a service, kubernetes, openfaas, devops, cloud
---

In this post, we're going to setup a cluster of Raspberry Pi 2 Model B's with Kubernetes. We are then going to install OpenFaaS on top of it so we can build _serverless_ apps. (Not that there aren't servers, but you're not supposed to care, or some such).

As I found when building this cluster out, the Kubernetes space is moving _fast_. Extremely so. That means, this post, like the ones I followed to build the cluster, will be outdated by the time you get here. With luck however, you'll find a clue or two here that will help you get going.

# Install the head node

First things first, you need to install and prep the first node. My head node is Raspbian Jesse ([https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/)), written with Etcher to an SD card. Once booted, you have some work to do.

## Setup the node

1. Update the OS and install some required packages:

```
sudo apt-get update \
    && sudo apt-get dist-upgrade -y --force-yes
sudo apt-get install -y \
    vim \
    git \
    wget \
    curl \
    unzip \
    build-essential \
    raspi-config \
    mosh \
    ntpdate \
    glances \
    avahi-daemon \
    netatalk
```

2. Next enable ssh:

```
sudo systemctl enable ssh
sudo systemctl start ssh
```

3. Disable swap:

```
sudo dphys-swapfile swapoff && \
  sudo dphys-swapfile uninstall && \
  sudo update-rc.d dphys-swapfile remove
```

3a. Enable cgroups:

Add the following to the end of ```/boot/cmdline.txt```:

```
cgroup_enable=cpuset cgroup_enable=memory
```

4. Copy over ssh keys

```
ssh-copy-id pi@node-01.local
```

5. (Optional) Setup the screen

My head node has a screen that I'll be using to show cluster status. It was installed and setup thusly:

```
curl -SLs https://apt.adafruit.com/add-pin | sudo bash
sudo apt-get install -y --force-yes raspberrypi-bootloader adafruit-pitft-helper raspberrypi-kernel

sudo adafruit-pitft-helper -t 35r
```

6. Make eth0 static

```
echo "allow-hotplug eth0
auto eth0
iface eth0 inet static
    address 172.16.1.1
    netmask 255.255.255.0" | sudo tee /etc/network/interfaces.d/eth0
```

7. Install Docker

```
curl -sSL get.docker.com | sh && \
sudo usermod pi -aG docker
```

(Optional) My other cluster nodes run Hypriot, which meant I needed to do some extra leg work to get the right docker onboard:

```
sudo apt-get autoremove -y docker-engine \
    && sudo apt-get purge docker-engine -y \
    && sudo rm -rf /etc/docker/ \
    && sudo rm -f /etc/systemd/system/multi-user.target.wants/docker.service \
    && sudo rm -rf /var/lib/docker \
    && sudo systemctl daemon-reload \
    && sudo apt-get install -y docker-engine=17.03.1~ce-0~raspbian-jessie
```

8. (Optional) Setup NAT (from ethernet to wifi)

I wanted my cluster to be isolated from the rest of my network. To this end, all the cluster nodes communicate over ethernet, while the head node functions as a NAT device, allowing them to connect over it's wifi interface to the rest of my network.

```
echo -e '\n#Enable IP Routing\nnet.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf \
    && sudo sysctl -p \
    && sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE \
    && sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT \
    && sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT \
    && sudo apt-get install -y iptables-persistent

sudo systemctl enable netfilter-persistent
```

8a. (Optional) Setup DHCP

As my cluster nodes are isolated, the head node runs DHCP to provide addressing.

```
sudo apt-get install -y dnsmasq \
    && sudo sed -i "s/#dhcp-range=192.168.0.50,192.168.0.150,12h/dhcp-range=172.16.1.2,172.16.1.254,12h/" /etc/dnsmasq.conf \
    && sudo sed -i "s/#interface=/interface=eth0/" /etc/dnsmasq.conf
```

9. Reboot

At this point we've made extensive modifications to the head node. To ensure swap stays disabled, cgroups are enabled, and the screen does as it should, a reboot is needed here.

```
sudo reboot
```

10. Install kubeadm

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -q && sudo apt-get install -qy kubeadm=1.8.5-00
```

11. Initialize the head node

As we want our cluster to communicate over the private network, we need to specify address for the API server to listen on. This is done with ```--apiserver-advertise-address=``` and allows the other nodes in our cluster to join properly.

We also specify ```--pod-network-cidr``` as a pool of addresses that our pods will be assigned.

__Note:__ This will take a while.

```
sudo kubeadm init \
    --apiserver-advertise-address=172.16.1.1 \
    --pod-network-cidr 172.16.1.0/24
```

Then, run the bit it gives you at the end:

```
mkdir -p $HOME/.kube \
    && sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config \
    && sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

__Note:__ Save the join token. You can generate another one if you forgot to save this, but you will need the token to join additional nodes.

12. Check that it worked

```
kubectl get pods --namespace=kube-system
```

13. Generate a new machine ID

As a side effect of the imaging process, you may end up with nodes that do not have a unique machine-id. Generate a new one:

```
sudo rm /var/lib/dbus/machine-id
sudo rm /etc/machine-id
sudo systemd-machine-id-setup
sudo reboot
```

14. Networking

On the master node:

```
kubectl apply -f https://git.io/weave-kube-1.6
```

15. Check our work:

```
kubectl exec -n kube-system weave-net-2zl6f -c weave -- /home/weave/weave --local status connections
```

16. (Optional) Create a user & role

If you will be deploying the dashboard, you will need an admin user, role, and login token.

Creating the user & role:

```
echo "apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system" | tee role.yml

echo "apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system"| tee user.yml

kubectl create -f user.yml
kubectl create -f role.yml
```

Getting a login token:

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

## Additional nodes

1. Updates

```
sudo apt-get update && sudo apt-get dist-upgrade -y --force-yes
```

2. Install kubeadm

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubeadm
```

3. Join the cluster

This is where the token from before comes in handy:

```
sudo kubeadm join --token  172.16.1.1:6443 --discovery-token-unsafe-skip-ca-verification
```

4. Check for the new nodes

On the head node, you can use the following command to check if your new nodes have joined properly:

```
kubectl get nodes
```

## Install OpenFaaS

At this point you can follow from step 2.0b on the official OpenFaaS guide: [https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md](https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md)

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

git clone https://github.com/openfaas/faas-netes
cd faas-netes
kubectl apply -f faas.armhf.yml,rbac.yml,monitoring.armhf.yml
```

# Resources

There were a lot of guides, blog posts, and more that went into making this post:

* [https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975](https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975)
* [https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/)
* [https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx](https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx)
* [https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md](https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md)