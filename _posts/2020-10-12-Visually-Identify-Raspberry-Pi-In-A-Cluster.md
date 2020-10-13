---
title: "Visually Identify Raspberry Pi In A Cluster"
date: 2020-10-12
layout: "post"
categories: "led, iot, rpi, raspberry pi, cluster, k8s"
---

# Visually Identify Raspberry Pi In A Cluster

I run a small Kubernetes cluster on about nine Raspberry Pi nodes, and while they take up much less space, finding a specific fail[ed|ing] node can be just as challenging as finding a random server in a datacenter. Good news is, you can make one of the onboard LEDs flash to give you an idea of which node it is.

## Make the node blink

The following instructions assume you have both remote access (SSH) to your Raspberry Pi, as well as `sudo` or root permissions.

1. Verify the current kernel module for `LED0`:

    ```shell
    $ cat /sys/class/leds/led0/trigger
    none rc-feedback kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock
    kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock
    kbd-ctrlrlock timer oneshot heartbeat backlight gpio cpu cpu0 cpu1 cpu2 cpu3 default-on
    input panic actpwr [mmc0] rfkill-any rfkill-none
    ```

2. Enable the `ledtrig_heartbeat` kernel module:
   
```shell
   $ sudo su -
   # modprobe ledtrig_heartbeat
   ```

3. Set `LED0` to blinking:

   ```shell
   # echo "heartbeat" > /sys/class/leds/led0/trigger
   # cat /sys/class/leds/led0/trigger
   none rc-feedback kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock
   kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock
   kbd-ctrlrlock timer oneshot [heartbeat] backlight gpio cpu cpu0 cpu1 cpu2 cpu3 default-on
   input panic actpwr mmc0 rfkill-any rfkill-none
   ```

4. Reset back to SD card access:

   ```shell
   # echo "mmc0" > /sys/class/leds/led0/trigger
   # cat /sys/class/leds/led0/trigger
   none rc-feedback kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock
   kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock
   kbd-ctrlrlock timer oneshot heartbeat backlight gpio cpu cpu0 cpu1 cpu2 cpu3 default-on
   input panic actpwr [mmc0] rfkill-any rfkill-none
   ```

## Explanation

The status LED (the flashy green one, or `LED0`) on a Raspberry Pi will flash on SD card access. To identify a specific node, we take advantage of the fact that LED0 is programmable, and set it to heartbeat, which makes it flash on and off in a steady pattern.

You can also configure the red power LED to heartbeat as well by changing `led0` to `led1` in the preceding commands.

## Bonus - Enable and disable blinking with Ansible

The commands discussed before work for a single node, but, honestly, they can be hard to remember when you are in a pinch. Have no fear, the following is an Ansible playbook that will set them to blinking for you:

```yaml
---
# Sets led0 on an rPI to heartbeat
# To allow for identification of a node in a cluster of rPI.

- name: Host Provisioning
  hosts: swarm
  become: true
  gather_facts: false
  become_method: sudo
  vars_files:
    - vars/provision.yaml
  tasks:
    - name: Modprobe led heartbeat
      modprobe:
        name: ledtrig_heartbeat
        state: present
    - name: Set LED0 to blink
      shell: "echo heartbeat > /sys/class/leds/led0/trigger"
      args:
        executable: /bin/bash
      when:
        - identify_me is defined
        - identify_me
    - name: Set LED0 to normal
      shell: "echo mmc0 > /sys/class/leds/led0/trigger"
      args:
        executable: /bin/bash
      when:
        - identify_me is not defined
```

Then run the playbook as follows:

```shell
# Enable heartbeat LED across the cluster
$ ansible-playbook -i swarm.yml -e "identify_me=true" identify-rpi.yml

# Disable it across the cluster
$ ansible-playbook -i swarm.yml identify-rpi.yml

# Enable heartbeat LED for a specific node
$ ansible-playbook -i swarm.yml -e "identify_me=true" --limit swarm-02 identify-rpi.yml
```

## Resources

* [https://www.raspberrypi.org/forums/viewtopic.php?t=12530](https://www.raspberrypi.org/forums/viewtopic.php?t=12530)