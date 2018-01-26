---
title: "OpenFaaS on Kubernetes on Raspberry-Pi"
date: 2018-01-05
layout: "post"
categories: faas, function as a service, kubernetes, openfaas, devops, cloud
---

In this post, we're going to setup a cluster of Raspberry Pi 2 Model B's with Kubernetes. We are then going to install OpenFaaS on top of it so we can build _serverless_ apps. (Not that there aren't servers, but you're not supposed to care, or some such).

__Update!__

The prior release of this post ended up duplicating a lot of the work that [Alex Ellis](https://blog.alexellis.io/) published [here.](https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975).

In fact, once I'd hit publish, my post was already out of date. So, to get Kubernetes and OpenFaaS going on the PI, start there. What follows here, then, are the changes I made to the process to fit my environment.

# Requirements

For my lab cluster, I wanted the environment to be functional, portable, isolated from the rest of my network, and accessible over wifi. The network layout looks a bit like this:

![Network layout](https://i.imgur.com/xhMT7Hz.png)

Networks:

* Green - Lab-Net - 10.127.8.x/24
* Blue - K8s Net - 172.12.0.0/12

# Build

The build has two parts, hardware and software. Rather than provide you with a complete manifest of the hardware, we'll summarize it so we can spend more time on the software parts.

## Hardware

* 7x Raspberry Pi 2 Model B
* 1x 8 port 100Mbit switch (It's what I had around)
* 1x Anker 10 port usb charger
* 7x Ethernet cables
* 1x usb wifi adapter
* 2x GeauxRobot 4-layer Dog Bone Stack Clear Case Box Enclosure
* Some zip ties

## Software

Here's where the rubber meets the road. First, familiarize yourself with Alex's build instructions, [here.](https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975).

To make my process a little smoother, I used [Packer](https://www.packer.io/) to embed Docker, Kubeadm, avahi, and cloud-init into the image. Then used Hypriot's [flash](https://github.com/hypriot/flash) utility to burn all 7 images with the right host names.

> __Note:__ Avahi allows me to connect to the nodes from node-01 over the private network by name, without having to setup DNS.

### Building the new Raspbian image

> __Assumption:__ Here we make the assumption that you have Vagrant installed and operational.

Packer does not ship with an arm image builder. Thankfully, the community has supplied one: [https://github.com/solo-io/packer-builder-arm-image](https://github.com/solo-io/packer-builder-arm-image)

First, clone the repo, and start the [Vagrant Test Environment](https://github.com/solo-io/packer-builder-arm-image#vagrant-test-environment)

```
git clone https://github.com/solo-io/packer-builder-arm-image
cd packer-builder-arm-image
vagrant up && vagrant ssh
```

Next, you will need to create some supporting files for packer. Specifically, we need to create a json file that tells Packer what to build, a customization script to install our additional packages, and a user-data.yml for the ```flash``` process.

The files I used can be found [here](https://gist.github.com/bunchc/99a04536be770d0c9a64efca0b0fbe00), and places into the ```/vagrant/``` folder of the VM.

> __Note:__ If you are using Raspberry PI 3's with built in Wifi, the user-data.yml I supplied will have them all connect to your network, rather than forcing them to communicate through the master node.

Finally, we're ready to build the image, to do that, from inside the Test VM, run the following commands:

```
sudo packer build /vagrant/kubernetes_base.json
rsync --progress --archive /home/vagrant/output-arm-image/image /vagrant/raspbian-stretch-modified.img
```

You can now shutdown the Test VM.

### Burning the image to the cards

Now that you have a custom image, it's time to put it on the card. To do that, with the ```flash``` command installed, the following command will burn each SD card and set it's hostname:

```
for i in {01..07}; do ~/Downloads/flash --hostname node-$i --userdata ./user-data.yml ./raspbian-stretch-modified.img; done
```

Place the SD cards into your RaspberryPIs and boot! This will take a few minutes.

## Setting up networking

From the network diagram earlier, you'll have seen that we're using node-01 as both the Kubernetes master, as well as the network gateway for the rest of the nodes. To supply the nodes with connectivity, we need to configure NAT and DHCP. The following commands will do this for you:

1. Set eth0 to static

Change the address and netmask to fit your environment.

```
# Set eth0 to static
echo "allow-hotplug eth0
auto eth0
iface eth0 inet static
    address 172.12.0.1
    netmask 255.240.0.0" | sudo tee /etc/network/interfaces.d/eth0

# Restart networking
sudo systemctl restart networking
```

2. Configure NAT

```
# Configure NAT
echo -e '\n#Enable IP Routing\nnet.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf \
  && sudo sysctl -p \
  && sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE \
  && sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT \
  && sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT \
  && sudo apt-get install -y iptables-persistent

# Enable the service that retains our iptables through reboots
sudo systemctl enable netfilter-persistent
```

3. Configure DHCP

Change ```dhcp-range=172.12.0.200,172.15.255.254,12h``` in the command below to fit your private network.

```
# Configure DHCP
sudo apt-get install -y dnsmasq \
  && sudo sed -i "s/#dhcp-range=192.168.0.50,192.168.0.150,12h/dhcp-range=172.12.0.200,172.15.255.254,12h/" /etc/dnsmasq.conf \
  && sudo sed -i "s/#interface=/interface=eth0/" /etc/dnsmasq.conf
sudo systemctl daemon-reload && sudo systemctl restart dnsmasq
```

4. Check that your nodes are getting addresses

```
sudo cat /var/lib/misc/dnsmasq.leases
1516260768 b8:27:eb:65:ae:6c 172.13.235.110 node-06 01:b8:27:eb:65:ae:6c
1516259308 b8:27:eb:48:29:01 172.14.32.87 node-02 01:b8:27:eb:48:29:01
1516260040 b8:27:eb:c4:8b:6b 172.15.204.242 node-04 01:b8:27:eb:c4:8b:6b
1516261126 b8:27:eb:e2:1f:27 172.12.85.144 node-07 01:b8:27:eb:e2:1f:27
1516260385 b8:27:eb:69:fa:ff 172.14.174.146 node-05 01:b8:27:eb:69:fa:ff
1516259665 b8:27:eb:66:2d:2d 172.14.218.40 node-03 01:b8:27:eb:66:2d:2d
```

## Installing Kubernetes

Next up, is the actual Kubernetes install process. My process differed slightly from Alex's, in a few ways.

### Master node

On the Master node I first pulled all the images. This speeds up the ```kubeadm init``` process significantly.

```
docker pull gcr.io/google_containers/kube-scheduler-arm:v1.8.6
docker pull gcr.io/google_containers/kube-controller-manager-arm:v1.8.6
docker pull gcr.io/google_containers/kube-apiserver-arm:v1.8.6
docker pull gcr.io/google_containers/pause-arm:3.0
docker pull gcr.io/google_containers/etcd-arm:3.0.17
```

After ```kubeadm init``` finishes, I installed weave using the method suggested in their [documentation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/):

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### Adding the remaining nodes

Once all the services showed healthy, rather than ssh to each remaining node, I used the following bash one-liner:

```
for i in {02..07}; do ssh -t pi@node-01.local ssh -t pi@node-$i.local kubeadm join --token redacted 172.12.0.1:6443 --discovery-token-ca-cert-hash sha256:redacted; done
```

## Install OpenFaaS

Once all the nodes show in ```kubectl get nodes```, you can preform the OpenFaaS install as documented in [Alex's blog post](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/).

# Resources

There were a lot of guides, blog posts, and more that went into making this post:

* [https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975](https://gist.github.com/alexellis/fdbc90de7691a1b9edb545c17da2d975)
* [https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/)
* [https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx](https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx)
* [https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md](https://github.com/openfaas/faas/blob/master/guide/deployment_k8s.md)