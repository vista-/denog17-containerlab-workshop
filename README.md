# DENOG17 Containerlab Workshop

## Section 0: Admin

You should have received a note with your instance ID on it.

The VM can be reached on `<ID>.ws.ip.horse`.  
The credentials will be shared during the workshop, in person.

SSH is running on port 2222.

There will be a Visual Studio Code web interface running on HTTPS port 8443. You can optionally use your own VS Code installation on your laptop, if one is available, to connect remotely to the instance -- this might give you a better user experience.

At the end of each section, you will find a solution proposal. This allows you to skip entire sections, while still being able to normally proceed with the following task.  

## Section 1: Intro to Containerlab

### Task 1.1: Prerequisites

To install Containerlab, you will need a Linux system that's recently new (running kernel 5.4 or newer).

If you would like to use Containerlab on a non-Linux host, for Windows, WSL2 is the recommended way of running it, while for Mac, any Linux VM solution (`lima`, OrbStack, UTM, etc) works well.

_For the purposes of this workshop, please use the VMs we have provided and hosted for you._

### Task 1.2: Installing Containerlab

Let's go through the quick install procedure!

The [quick-start script](https://containerlab.dev/quickstart/) on the Containerlab website works with most DEB (Debian, Ubuntu) and RPM-based (Red Hat, CentOS, Fedora, etc) distributions.

After logging in, run the following command:

```
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

This should install Containerlab for you!

> [!TIP]
> If you would like to use Containerlab without sudo, like the output of the installer suggests, you should add your user to this group using the following command:  
> 
> `gpasswd -a <your username here> clab_admins`
> 
> and apply the new group membership to your terminal session using the command
> `newgrp clab_admins` (or log in and out from the SSH session)

The Containerlab installation can be tested by running `containerlab version`:
```
$ containerlab version
  ____ ___  _   _ _____  _    ___ _   _ _____ ____  _       _
 / ___/ _ \| \ | |_   _|/ \  |_ _| \ | | ____|  _ \| | __ _| |__
| |  | | | |  \| | | | / _ \  | ||  \| |  _| | |_) | |/ _` | '_ \
| |__| |_| | |\  | | |/ ___ \ | || |\  | |___|  _ <| | (_| | |_) |
 \____\___/|_| \_| |_/_/   \_\___|_| \_|_____|_| \_\_|\__,_|_.__/

    version: 0.71.0
     commit: 7ef796f07
       date: 2025-10-10T17:36:13Z
     source: https://github.com/srl-labs/containerlab
 rel. notes: https://containerlab.dev/rn/0.71/
```

### Task 1.2: Testing Containerlab

In order to test and validate your Containerlab installation, you can deploy a "hello world" topology example and test if traffic passes between two nodes.

The following command clones the provided Git repository, and deploys the Containerlab topology contained within.

```
clab deploy –t https://github.com/vista-/clab-helloworld
```

Once deployed, you should be able to ping from `host1` to `host2`.

```
$ docker exec clab-helloworld-host1 ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=1.037 ms
```

Finally, we can clean up the topology we just created by doing

```
clab destroy -t https://github.com/vista-/clab-helloworld
```

**What did we learn from this?**
- Containerlab has a command alias, `clab`, which is quicker to type.
- Containerlab topologies can be deployed from remote URLs!
GitHub repositories, direct URLs to Containerlab topologies, even S3 links work fine.
- `docker exec` can be used to run commands on containers deployed by Containerlab.
This is one of the two main methods for interacting with a node in a running Containerlab topology.


### Task 1.3: Creating your first Containerlab topology

#### Terminology

The most important objects within a Containerlab topology definition are:

- Nodes
Within the `nodes` section of a Containerlab topology definition the actual containers that should be deployed are described. In most cases, each node represents a single container. This could be a network element (router, switch, firewall, etc), but can also be any other Linux-based container.
- Links
In order to create an actual topology, the defined nodes need to be interconnected. In the `links` section, we can define links by providing pairs of `endpoints` for Containerlab to connect. An endpoint consist of a `node` and an interface of the node.

References:
- [Containerlab nodes](https://containerlab.dev/manual/nodes/)
- [Containerlab links](https://containerlab.dev/manual/topo-def-file/#links)

#### The first lab topology

Containerlab supports [over 35 different types of network OSes](https://containerlab.dev/manual/kinds/) that can be ran inside a topology. However, most commercial NOSes can only be downloaded with an active account for a given vendor's website or download portal.

For this workshop, we will use two freely downloadable NOSes that do not require a registration:

- [FRR](https://docs.frrouting.org/projects/dev-guide/en/latest/index.html#)
- [Nokia SR Linux](https://learn.srlinux.dev/)

To get started, we will create a simple topology consisting of 2 nodes with a single link in between them.

`leaf1` will be an SR Linux node, running SR Linux 24.10, and `leaf2` will be FRR, running the latest stable version, 10.4.1, using the container image `quay.io/frrouting/frr`!

To make things simple, we will use the first available data-plane interface for both nodes:  
in case of SR Linux, this is `ethernet-1/1`, and `eth1` for FRR.

```
┌─────────┐                 ┌─────────┐
│         │ethernet-1/1     │         │
│  leaf1  │─────────────────│  leaf2  │
│         │             eth1│         │
└─────────┘                 └─────────┘
  SR Linux                      FRR 
```

Armed with this topology diagram and the [Containerlab topology format definition](https://containerlab.dev/manual/topo-def-file/#topology-definition-components), you should be able to write your first Containerlab topology in the file `workshop.clab.yaml`!

If you are already familiar with the Containerlab basics and want to skip over this exercise, you'll find the solution right here:

<details>
<summary>✅ Task 1.3 solution</summary>

```yaml
name: denog-workshop

topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10
    leaf2:
      kind: linux
      image: quay.io/frrouting/frr:10.4.1

  links:
    - endpoints: ["leaf1:ethernet-1/1", "leaf2:eth1"]
```

</details>

### Task 1.4: Running and Configuring the Lab

Let's get a simple connectivity test going on in this lab!

To do that, we will first deploy the lab using the `containerlab deploy` command.  
You might notice during the deploy process, that Containerlab will automatically download the necessary container images if they are not already present on the VM.

Once the nodes have started, Containerlab will give us an overview like this:
```
╭───────────────────────────┬──────────────────────────────┬─────────┬───────────────────╮
│            Name           │          Kind/Image          │  State  │   IPv4/6 Address  │
├───────────────────────────┼──────────────────────────────┼─────────┼───────────────────┤
│ clab-denog-workshop-leaf1 │ nokia_srlinux                │ running │ 172.20.20.3       │
│                           │ ghcr.io/nokia/srlinux:24.10  │         │ 3fff:172:20:20::3 │
├───────────────────────────┼──────────────────────────────┼─────────┼───────────────────┤
│ clab-denog-workshop-leaf2 │ linux                        │ running │ 172.20.20.2       │
│                           │ quay.io/frrouting/frr:10.4.1 │         │ 3fff:172:20:20::2 │
╰───────────────────────────┴──────────────────────────────┴─────────┴───────────────────╯
```

The nodes are automatically assigned a management IPv4 and IPv6 address by Containerlab, but these can also be set statically in the topology file, including what subnet the management IPs should be in. 

>[!NOTE]
> Given that most network OSes require additional configuration to be remotely manageable, Containerlab loads them with an initial configuration, just enough to give them a hostname, management connectivity and fixed credentials to SSH in with.
>
> Some node kinds go a bit further than that - Containerlab will automatically load SSH public keys, pre-configure node hardware configuration, as a convenience feature.


For most nodes, connecting to a node is just a matter of `ssh`-ing to the node's name:

```
$ ssh clab-denog-workshop-leaf1
Warning: Permanently added 'clab-denog-workshop-leaf1' (ED25519) to the list of known hosts.
................................................................
:                  Welcome to Nokia SR Linux!                  :
:              Open Network OS for the NetOps era.             :
:                                                              :
:    This is a freely distributed official container image.    :
:                      Use it - Share it                       :
:                                                              :
: Get started: https://learn.srlinux.dev                       :
: Container:   https://go.srlinux.dev/container-image          :
: Docs:        https://doc.srlinux.dev/24-10                   :
: Rel. notes:  https://doc.srlinux.dev/rn24-10-5               :
: YANG:        https://yang.srlinux.dev/v24.10.5               :
: Discord:     https://go.srlinux.dev/discord                  :
: Contact:     https://go.srlinux.dev/contact-sales            :
................................................................

Using configuration file(s): ['/home/admin/.srlinuxrc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.

--{ running }--[  ]--
A:leaf1#
```

However, this won't work with FRR, as there is no SSH server running inside that container!

Instead, we will be using the _other_ way of connecting to a container: `docker exec`, which allows you to run arbitrary commands inside an running container.

>[!NOTE]
> The command will be executed _inside_ the container, meaning that all files and binaries referenced in the command must also be present _inside_ the container.

FRR's CLI can be started by running the `vtysh` command. To run a command and attach it in the current terminal, we will use the `-it` flags of `docker exec`. Try it out!

```
$ docker exec -it clab-denog-workshop-leaf2 vtysh
% Can't open configuration file /etc/frr/vtysh.conf due to 'No such file or directory'.
Configuration file[/etc/frr/frr.conf] processing failure: 11

Hello, this is FRRouting (version 10.4.1_git).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

leaf2#
```

Your task now is to repeat the ping test we did on the "hello world" lab between leaf1 and leaf2! Configure 10.0.0.1/24 and 10.0.0.2/24 on **leaf1** and **leaf2** respectively, and perform a ping test!

Links to relevant documentations:
- [FRR](https://docs.frrouting.org/en/latest/zebra.html#standard-commands)
- [SR Linux](https://documentation.nokia.com/srlinux/24-10/title/interfaces.html)

> [!TIP]
> SR Linux hints:
> <details>
> <summary>Navigating the CLI</summary>
>
>    - After logging in, you are in the **running** mode. This is the equivalent of operational mode on other network OSes, and you cannot make changes to the configuration in this mode
>    - You can switch between modes using the `enter <mode>` command. The configuration mode is called the **candidate** mode
>    - The `info` command shows you the configuration in the current mode. `info <path>` shows you a specific path of configuration. To view the running configuration while in candidate mode, use `info from running <path>` (and vice-versa)
>    - `show` commands can be used to view certain reports about the switch. For example, `show interface brief` gives you an overview of the state of interfaces, while `show interface mgmt0` (or any specific interface name) gives you details about an interface
> </details>
> <details>
> <summary>Configuring an interface</summary>
>
>    - SR Linux uses a hierarchical, model-based configuration, based on the YANG model of SR Linux. The interface related settings can be found in the `interface` section
>    - IP configuration cannot be directly applied to a physical interface, it must be done on a logical subinterface instead
>    - Parts of the configuration hierarchy must be explicitly enabled with the `admin-state enabled` setting. In this case, `interface X`, `subinterface Y` and `ipv4` are three sections that can have their admin-state toggled, and the first two are implicit enabled, the last section, however, is not
> </details>
> <details>
> <summary>Assigning an interface to a network-instance</summary>
>
>    - In SR Linux, all (logical) interfaces must be associated with a network-instance in order to pass traffic, and they are not associated to any by default
>    - A default network-instance can be used to provide base connectivity and control-plane protocols for other network-instances to be carried over. This is called the 'default' network-instance and has the type 'default'
> </details>
> <details>
> <summary>Managing the configuration</summary>
>
>    - The SR Linux CLI will tell you if there's a change in the candidate configuration compared to the running configuration - look for the **asterisk \*** in the prompt!
>    - SR Linux is transaction-based, and changes are applied atomically by performing a commit
>    - You can use the `diff` command to see the difference between the candidate and running configuration
>    - You can validate your changes before applying the candidate configuration by doing a `commit validate`
>    - Commit can be performed using `commit now` or `commit stay`. The former returns you to the **running** mode, the latter leaves you in **candidate** mode.
>    - Commits can be performed in a safe manner by using `commit confirmed`. This starts a confirm timer, and if it expires, the commit is reverted, unless the commit is confirmed using the `tools system configuration confirmed-accept` command.
>    - When the running configuration is changed, it is not saved to the disk yet. A **plus +** in the prompt marks an unsaved change in the running configuration compared to the startup configuration.
>    - To save a running configuration in SR Linux, run the `save startup` command.
> </details>


> [!TIP]
> FRR hints:
> <details>
> <summary>FRR hints</summary>
>
>    - The FRR CLI is similar to IOS and EOS. You can enter the configuration mode with `conf t`
>    - Changes are immediately applied with the FRR CLI. You can exit from the configuration mode with `end`
>    - The interface names used by FRR match Linux interface naming
>    - There is no need to assign the interface to a network-instance (or VRF, in FRR parlance), every interface implicitly belongs to the default VRF unless otherwise configured
>    - `write` writes the running configuration to disk
> </details>


> ✅ **Task 1.4 Solution**
> 
> <details>
> <summary>leaf1 SR Linux configuration</summary>
> 
> ```
> enter candidate
> # We have entered candidate mode, we can start configuring now
> set / interface ethernet-1/1 admin-state enable
> set / interface ethernet-1/1 subinterface 0 admin-state enable
> set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
> set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/24
> # Assign the (logical) interface to the default network-instance we just created
> set / network-instance default admin-state enable 
> set / network-instance default type default
> set / network-instance default interface ethernet-1/1.0
> # Commit changes and return to running mode
> commit now
> # Save configuration
> save startup
> ```
> 
> </details>
> <details>
> <summary>leaf2 FRR configuration</summary>
> 
> ```
> conf t
>     interface eth1 
>         ip address 10.0.0.2/24
>     !
> end
> ```
> 
> </details>
> <details>
> <summary>Validation</summary>
> 
> ```shell
> --{ running }--[  ]--
> A:leaf1# ping 10.0.0.2 network-instance default
> Using network instance default
> PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
> 64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=51.4 ms
> 64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=2.17 ms
> 64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=2.15 ms
> ^CCommand execution aborted : 'ping 10.0.0.2 network-instance default '
> 
> --{ running }--[  ]--
> ```
> 
> </details>
    
### Task 1.5: Adding startup configs to your lab topology

When working with network labs, you might not want to start from scratch every time you deploy a lab. Even if you have the configurations saved, you probably don't want to apply them by hand node by node...
    
Instead of supplying configuration by hand, we can also let Containerlab load the startup configurations for us!
    
Depending on the `kind` of the node, there might be several different ways on how to add a startup config to the simulated network element. These are all documented under the [Kinds section](https://containerlab.dev/manual/kinds/) of the Containerlab documentation.
    
>[!WARNING]
> As a reminder: Containerlab applies some initial default configurations to some NOS's already. This is important to know when working with startup-configs. 
> 
> A general rule of thumb is leave out management plane configuration, so management connectivity, SSH, authentication, etc, to avoid situations where Containerlab and your own startup configuration might want to modify the same configuration, and leave you with a node that you cannot connect to.
    
In SR Linux, there are two ways of adding a startup config to a node:
- as a partial config added on top of the default Containerlab-added management configuration, either in the form of CLI commands, or in a hierarchical configuration format:

```yaml
name: partial-srl-startup
topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10
      # a path to the partial config in CLI format relative to the current working directory
      startup-config: ./configs/myconfig.cli
```
</details>

- or as a full SR Linux configuration file in, JSON format (note: the hierarchical config format is not a JSON!). This is the format you can find on SR Linux nodes' file systems:
    
```yaml
name: full-srl-startup
topology:
  nodes:
     srl1:
       kind: nokia_srlinux
       image: ghcr.io/nokia/srlinux:24.10
       # a path to the full config in JSON format relative to the current working directory
       startup-config: ./configs/myconfig.json
```

More information about SR Linux startup configs can be found in the [SR Linux Containerlab documentation](https://containerlab.dev/manual/kinds/srl/).

With FRR, we use the generic Linux kind to represent the node with an FRR container image. Therefore, there is no specific feature to load a startup config for FRR nodes. Nonetheless, we can bind a config file into the FRR container, letting us define a startup configuration.

```yaml
name: frr-startup
topology:
  nodes:
    leaf2:
      kind: linux
      image: quay.io/frrouting/frr:10.4.1
      binds:
        - ./config/frr-daemons.conf:/etc/frr/daemons
        - ./config/leaf2.conf:/etc/frr/frr.conf:rw
```

In addition to file mounts, it is also possible to automatically execute CLI commands after container startup. This is useful for Linux client devices, to set up Linux interfaces' IP addressing, or create a bond interface, for example.

```yaml
name: linux-cmd-exec
topology:
  nodes:
    client1:
      kind: linux
      image: ghcr.io/srl-labs/network-multitool
      exec:
        - ip addr add 192.168.10.2/24 dev eth1
        - ip route add 192.168.11.0/24 via 192.168.10.1
        - ip link set eth1 promisc on
        - ip link set eth1 up 
```

**Your next task is to re-deploy the topology with the configuration used in the previous step now in the startup configuration files!**  
The files should be stored in the `./config` directory.

> [!IMPORTANT]
> Before you deploy the topology again, you should **save your work**!

Let's take a quick look our working directory first!   
You can notice that a new directory prefixed with `clab-` has been created.  
    This is called the _lab directory_, and is created on the first deployment of a topology by Containerlab. For node kinds that support it (such as SR Linux), this is where the state of the network OS is persisted between deployments.
    
For example, take a look at the `clab-.../leaff1/config/config.json` file:
    
```json
{
  "_preamble": {
    "header": {
      "generated-by": "SRLINUX",
      "name": "",
      "comment": "",
      "created": "2025-11-05T12:21:01.080Z",
      "release": "v24.10.5",
  ...
}
```
    
This is `leaf1`'s JSON configuration file that we saved onto earlier!
    
Many other NOSes also support saving the node configuration with the `containerlab save` command. However, FRR is not one of these NOSes at this point in time (as it is ran as a generic Linux container).
    
Using the `-c` command-line flag in deploy and destroy commands removes the lab directory created by Containerlab on lab deployment, so let's use the `containerlab destroy -c` command to destroy the topology and remove the lab directory!
    
Once you are done with adding the startup configuration files to the topology, it's time to redeploy again! Pinging should still work :)

<details>
<summary>✅ Task 1.5 solution</summary>

```yaml
name: denog-workshop

topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10
      startup-config: ./config/leaf1.cli
    leaf2:
      kind: linux
      image: quay.io/frrouting/frr:10.4.1
      binds:
        - ./config/leaf2.conf:/etc/frr/frr.conf:rw

  links:
    - endpoints: ["leaf1:ethernet-1/1", "leaf2:eth1"]
```
</details>
    
## Section 1 and a Bit More: Getting a bit more comfy in our VM
    
Now, we have been working a lot in the CLI!
    
We're getting to the point where we have to edit multiple files, and while we know that the audience will likely be filled with `vim` virtuosos and `emacs` fanatics, sometimes we should also look on the GUI side of life :)
    
VS Code is a great little tool that will get us quite far in our Containerlab journey, and you should consider giving it a spin, if only for the Containerlab GUI extension!
    
... [Containerlab GUI extension](https://containerlab.dev/manual/vsc-extension/)? Why yes, we have one those!
    
![VS Code Containerlab extension](./images/vscode-extension.png)

If you have the ability and affinity to install VS Code, now is the time to try it out!
    
For those with a draconic IT (or lack of disk space), we have also prepared a hosted version of VS Code for you on your assigned VM, available on **port 8443**.
The password is the same as the one we gave you for logging in!

The VS Code extension available to download from the VS Code Extension Store, free of charge, of course!


## Section 2: Traffic Generation and Essential Labbing Tools

To validate connectivity, but also to make our topology do a bit more interesting, let's add some _users_ to our network! Don't worry, they won't create any IT tickets 😅

```
   ┌─────────┐                 ┌─────────┐       
   │         │ethernet-1/1     │         │       
   │  leaf1  │─────────────────│  leaf2  │       
   │         │             eth1│         │       
   └─────────┘                 └─────────┘       
        │ethernet-1/2               │eth2
        │192.168.10.1/24            │192.168.11.1/24            
        │                           │            
        │                           │            
        │                           │            
        │                           │            
    eth1│192.168.10.2/24        eth1│192.168.11.2/24        
   ┌─────────┐                 ┌─────────┐       
   │         │                 │         │       
   │ client1 │                 │ client2 │       
   │         │                 │         │       
   └─────────┘                 └─────────┘       
network-multitool           network-multitool
```

### Task 2.1: Adding clients to the Containerlab topology

There are several options available on how to simulate client devices within a Containerlab topology -- there are full-blown traffic generators out there (e.g. BNG Blaster, Ixia-c), but in most cases, a regular Linux OS client works well. Thankfully, with containers, it's really easy (and very resource friendly) to simulate even hundreds of Linux clients!
    
We already use one type of `linux` kind in the container topology, the FRR node.  
Let's use a different image and add two clients to the topology, `client1` and `client2` to `leaf1` and `leaf2` respectively!
    
The clients should be connected to the `ethernet-1/2` interface of `leaf1`, and `eth2` interface of `leaf2`.
    
For this workshop the recommended container image to be used as clients is `ghcr.io/srl-labs/network-multitool`. This image is based on a lightweight Alpine Linux image, and comes with some helpful network-related tools pre-installed, and for your convenience, comes with an SSH server running.
    
> [!NOTE]
> We did not specify a container tag (or version) for the `network-multitool` image here. When you don't specify a specific version of an image, Containerlab, much like Docker or Podman, will automatically grab the container image with the `latest` tag. If this tag does not exist for the container image, Containerlab will return an error.
    
These clients will have L3 connectivity for now to our switches, so we should make sure they get IP addresses on their data plane interfaces assigned during the topology deployment.
    
The leaf switches should also have the IPs configured on their client-facing interfaces, again, all done in the startup config!

> ✅ **Task 2.1 Solution**
>     
> <details>
> <summary>Topology solution</summary>
> 
> ```yaml
> name: denog-workshop
> 
> topology:
>   nodes:
>     leaf1:
>       kind: nokia_srlinux
>       image: ghcr.io/nokia/srlinux:24.10
>       startup-config: ./config/leaf1.cli
>     leaf2:
>       kind: linux
>       image: quay.io/frrouting/frr:10.4.1
>       binds:
>         - ./config/leaf2.conf:/etc/frr/frr.conf:rw
>     client1:
>       kind: linux
>       image: ghcr.io/srl-labs/network-multitool
>       exec:
>         - ip addr add 192.168.10.2/24 dev eth1
>         - ip route add 192.168.11.0/24 via 192.168.10.1
>         - ip link set eth1 up 
> 
>     client2:
>       kind: linux
>       image: ghcr.io/srl-labs/network-multitool
>       exec:
>         - ip addr add 192.168.11.2/24 dev eth1
>         - ip route add 192.168.10.0/24 via 192.168.11.1
>         - ip link set eth1 up
> 
>   links:
>     - endpoints: ["leaf1:ethernet-1/1", "leaf2:eth1"]
>     - endpoints: ["leaf1:ethernet-1/2", "client1:eth1"]
>     - endpoints: ["leaf2:eth2", "client2:eth1"]
> ```
> 
> </details>
> <details>
> <summary>SRL config solution | leaf1.cli</summary>
> 
> ```
> set / interface ethernet-1/1 admin-state enable
> set / interface ethernet-1/1 subinterface 0 admin-state enable
> set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/24
> set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
>
> set / interface ethernet-1/2 admin-state enable
> set / interface ethernet-1/2 subinterface 0 admin-state enable
> set / interface ethernet-1/2 subinterface 0 ipv4 address 192.168.10.1/24
> set / interface ethernet-1/2 subinterface 0 ipv4 admin-state enable
>
> set / network-instance default interface ethernet-1/1.0
> set / network-instance default interface ethernet-1/2.0
> ```
> 
> </details>
>
> <details>
> <summary>FRR config solution | leaf2.conf</summary>
> 
> ```
>     interface eth1 
>         ip address 10.0.0.2/24
>     !
>     interface eth2 
>         ip address 192.168.11.1/24
>     !
> 
> ```
> 
> </details>
    
### Task 2.2 Making friends with BGP (it wouldn't be a DENOG workshop without BGP...!)

We are going to make these two leafs talk to each other, with BGP, to tell each other about their favorite client subnets.
    
Let's configure eBGP! To get you started, here's the topology, but now with AS numbers:


```
   ┌───────────┐10.0.0.1         ┌───────────┐       
   │  AS65001  │ethernet-1/1     │  AS65002  │       
   │   leaf1   │─────────────────│   leaf2   │       
   │ 10.10.0.1 │             eth1│ 10.10.0.2 │       
   └───────────┘         10.0.0.2└───────────┘       
         │ethernet-1/2                 │eth2
         │                             │            
         │                             │            
         │                             │            
         │                             │            
         │                             │            
     eth1│                         eth1│        
   ┌───────────┐                 ┌───────────┐       
   │           │                 │           │       
   │  client1  │                 │  client2  │       
   │           │                 │           │       
   └───────────┘                 └───────────┘
```
    
(these are hand-made ASCII diagrams, by the by ❤️)
    
Since we're going to be delving into a new topic with BGP, here are some helpful pointers to get you started!


> [!TIP]
> Tips for both nodes:
> - You will need to configure eBGP on the link between leaf1 and leaf2
> - To establish an eBGP session, you will need to configure: the router-id, the local AS number, the neighbor peer IP address, and the neighbor peer AS
> - Generally, it's a good idea to use your loopback address as your router-id


> [!TIP]
> SR Linux tips:
> - The BGP protocol is configured under the network-instance
> - The address families on BGP are not implied - they must be configured under `afi-safi` in `protocols bgp`, by `admin-enable`-ing them  
> - BGP peers and BGP peer groups inherit the AFI/SAFI settings of the overall BGP protocol
> - It is mandatory to configure a BGP peer group for a BGP neighbor
> - SR Linux follows best practices and does not let you import routes via eBGP without a policy by default. There is knob to disable this behaviour for import and export as well.
> - To redistribute a route into BGP in SR Linux, this must be done via a routing policy. Connected routes are called 'local' routes
    
> [!TIP]
> FRR tips:
> - You will have to enable the BGP daemon in a separate configuration file, that you will have to bind mount as well
> - By default, eBGP sessions require some form of policy in FRR. You can write a policy, or find the knob that lets you bypass this
> - You should _redistribute_ routes into BGP to make them available for export
> - Similarly to SR Linux, you should enable some address family inside BGP

<details>
<summary>Verification</summary>

Pinging from a client works
```shell
[*]─[client1]─[/]
└──> ping 192.168.11.2
PING 192.168.11.2 (192.168.11.2) 56(84) bytes of data.
64 bytes from 192.168.11.2: icmp_seq=1 ttl=62 time=0.782 ms
```

Routes visible on both sides
```
A:leaf1# show network-instance default protocols bgp neighbor 10.0.0.2 received-routes ipv4
---------------------------------------------------------------------------------------------------------------------------------------------------------------
Peer        : 10.0.0.2, remote AS: 65002, local AS: 65001
Type        : static
Description : None
Group       : ebgp
---------------------------------------------------------------------------------------------------------------------------------------------------------------
Status codes: u=used, *=valid, >=best, x=stale, b=backup
Origin codes: i=IGP, e=EGP, ?=incomplete
+-------------------------------------------------------------------------------------------------------------------------------------------------------+
|      Status            Network            Path-id            Next Hop             MED              LocPref             AsPath             Origin      |
+=======================================================================================================================================================+
|        *           10.0.0.0/24        0                  10.0.0.2                  -                              [65002]                    ?        |
|       u*>          10.10.10.2/32      0                  10.0.0.2                  -                              [65002]                    ?        |
|       u*>          172.20.20.0/24     0                  10.0.0.2                  -                              [65002]                    ?        |
|       u*>          192.168.11.0/24    0                  10.0.0.2                  -                              [65002]                    ?        |
+-------------------------------------------------------------------------------------------------------------------------------------------------------+


leaf2# show bgp ipv4 192.168.10.0/24
BGP routing table entry for 192.168.10.0/24, version 7
Paths: (1 available, best #1, table default)
  Advertised to peers:
  10.0.0.1
  65001
     10.0.0.1 from 10.0.0.1 (10.10.10.1)
      Origin IGP, valid, external, best (First path received)
      Last update: Fri Nov  7 00:51:40 2025
```
</details>



> ✅ **Task 2.2 Solution**
> 
> <details>
> <summary>SRL config solution | leaf1.cli</summary>
> 
> ```
> set / interface ethernet-1/1 admin-state enable
> set / interface ethernet-1/1 subinterface 0 admin-state enable
> set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/24
> set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
>
> set / interface ethernet-1/2 admin-state enable
> set / interface ethernet-1/2 subinterface 0 admin-state enable
> set / interface ethernet-1/2 subinterface 0 ipv4 address 192.168.10.1/24
> set / interface ethernet-1/2 subinterface 0 ipv4 admin-state enable
>
> set / network-instance default interface ethernet-1/1.0
> set / network-instance default interface ethernet-1/2.0
> 
> set / interface lo0 subinterface 0 admin-state enable
> set / interface lo0 subinterface 0 ipv4 address 10.10.10.1/32
> set / network-instance default interface lo0.0
> 
> set / routing-policy policy export-local statement local-only match protocol local
> set / routing-policy policy export-local statement local-only action policy-result accept
>
> set / network-instance default protocols bgp admin-state enable
> set / network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
> set / network-instance default protocols bgp router-id 10.10.10.1
> set / network-instance default protocols bgp autonomous-system 65001
> set / network-instance default protocols bgp ebgp-default-policy import-reject-all false
> set / network-instance default protocols bgp export-policy [ export-local ]
> set / network-instance default protocols bgp group ebgp
> set / network-instance default protocols bgp neighbor 10.0.0.2 admin-state enable
> set / network-instance default protocols bgp neighbor 10.0.0.2 peer-group ebgp
> set / network-instance default protocols bgp neighbor 10.0.0.2 peer-as 65002
>
> ```
> 
> </details>
>
> <details>
> <summary>FRR config solution | leaf2.conf</summary>
> 
> ```
>     interface eth1 
>         ip address 10.0.0.2/24
>     !
>     interface lo
>      ip address 10.10.10.2/32
>     !
>     interface eth2 
>         ip address 192.168.11.1/24
>     !
>     router bgp 65002
>      no bgp ebgp-requires-policy 
>      bgp router-id 10.10.10.2
>      neighbor 10.0.0.1 remote-as 65001
>      !
>      address-family ipv4 unicast
>       redistribute connected 
>      exit-address-family
> 
> ```
> 
> </details>

### Task 2.3: Sending traffic

To get some traffic on our network, we can now send actual data from client to client. It comes in handy that the `network-multitool` container image used for the client devices has the `iperf` speed testing tool pre-installed.

The iperf user docs can be found [here.](https://iperf.fr/iperf-doc.php)
    
So, let's get those bits flowing, and an `iperf` working! 

> [!NOTE]
> The SRL container image comes with some PPS (Packet per second) limitations. You'll probably notice that when trying to push traffic through.

> [!WARNING]
> Network OSes can sometimes come with different MTU settings. Set the MSS to 1400 manually on `iperf`.
    
`iperf` example:
> <details>
> <summary>Server</summary>
> 
> ```shell
> [*]─[client2]─[/]
> └──> iperf3 -s
> -----------------------------------------------------------
> Server listening on 5201 (test #3)
> -----------------------------------------------------------
> Accepted connection from 192.168.10.2, port 49294
> [  5] local 192.168.11.2 port 5201 connected to 192.168.10.2 port 49304
> [ ID] Interval           Transfer     Bitrate
> [  5]   0.00-1.00   sec  1.25 MBytes  10.5 Mbits/sec                   
> [  5]   1.00-2.00   sec  1.25 MBytes  10.5 Mbits/sec                   
> [  5]   2.00-3.00   sec  1.12 MBytes  9.44 Mbits/sec                  
> [  5]   3.00-4.00   sec  1.25 MBytes  10.5 Mbits/sec                  
> [  5]   4.00-5.00   sec  1.12 MBytes  9.44 Mbits/sec                  
> [  5]   5.00-6.00   sec  1.25 MBytes  10.5 Mbits/sec                  
> [  5]   6.00-7.00   sec  1.12 MBytes  9.44 Mbits/sec                  
> [  5]   7.00-8.00   sec  1.25 MBytes  10.5 Mbits/sec                  
> [  5]   8.00-9.00   sec  1.12 MBytes  9.44 Mbits/sec                  
> [  5]   9.00-10.00  sec  1.25 MBytes  10.5 Mbits/sec                  
> - - - - - - - - - - - - - - - - - - - - - - - - -
> [ ID] Interval           Transfer     Bitrate
> [  5]   0.00-10.00  sec  12.0 MBytes  10.1 Mbits/sec                  receiver
> -----------------------------------------------------------
> Server listening on 5201 (test #4)
> -----------------------------------------------------------
> ```
> </details>
> <details>
> <summary>Client</summary>
> 
> ```shell
> [*]─[client1]─[/]
> └──> -c 192.168.11.2 -b 10M -M 1400
> Connecting to host 192.168.11.2, port 5201
> [  5] local 192.168.10.2 port 49304 connected to 192.168.11.2 port 5201
> [ ID] Interval           Transfer     Bitrate         Retr  Cwnd
> [  5]   0.00-1.00   sec  1.25 MBytes  10.5 Mbits/sec    0    132 KBytes       
> [  5]   1.00-2.00   sec  1.25 MBytes  10.5 Mbits/sec    0    132 KBytes       
> [  5]   2.00-3.00   sec  1.12 MBytes  9.44 Mbits/sec    0    132 KBytes       
> [  5]   3.00-4.00   sec  1.25 MBytes  10.5 Mbits/sec    0    132 KBytes       
> [  5]   4.00-5.00   sec  1.12 MBytes  9.44 Mbits/sec    0    132 KBytes       
> [  5]   5.00-6.00   sec  1.25 MBytes  10.5 Mbits/sec    0    132 KBytes       
> [  5]   6.00-7.00   sec  1.12 MBytes  9.44 Mbits/sec    0    132 KBytes       
> [  5]   7.00-8.00   sec  1.25 MBytes  10.5 Mbits/sec    0    132 KBytes       
> [  5]   8.00-9.00   sec  1.12 MBytes  9.44 Mbits/sec    0    132 KBytes       
> [  5]   9.00-10.00  sec  1.25 MBytes  10.5 Mbits/sec    1   94.7 KBytes       
> - - - - - - - - - - - - - - - - - - - - - - - - -
> [ ID] Interval           Transfer     Bitrate         Retr
> [  5]   0.00-10.00  sec  12.0 MBytes  10.1 Mbits/sec    1             sender
> [  5]   0.00-10.00  sec  12.0 MBytes  10.1 Mbits/sec                  receiver
> 
> iperf Done.
> ```
> </details>
    
While the `iperf` is running, let's take a look at the statistics on the switches!
    
#### FRR counters

To view interface counters on Linux, you can use the `ip link show` command with the `-s -s` additional statistics flags:

```shell
[*]─[client2]─[/]
└──> ip -s -s link show dev eth1
133: eth1@if134: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP mode DEFAULT group default
    link/ether aa:c1:ab:a1:a8:07 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    RX:  bytes packets errors dropped  missed   mcast
        210729    3186      0       0       0       0
    RX errors:  length    crc   frame    fifo overrun
                     0      0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
      21497637   10099      0       0       0       0
    TX errors: aborted   fifo  window heartbt transns
                     0      0       0       0       2
```
    
#### SR Linux counters

In most network OSes, `show` commands give you an idea, as a human operator, what is going on inside a switch.
    
```
A:leaf1# show interface ethernet-1/1.0 detail
---------------------------------------------------------------------------------------------------------------------------------------------------------------
===============================================================================================================================================================
  Subinterface: ethernet-1/1.0
---------------------------------------------------------------------------------------------------------------------------------------------------------------
  Description     : <None>
  Network-instance: default
  Type            : routed
  Oper state      : up
  Down reason     : N/A
  Last change     : 15h36m55s ago
  Encapsulation   : null
  IP MTU          : 1500
  Last stats clear: never
  IPv4 addr    : 10.0.0.1/24 (static, preferred, primary)
---------------------------------------------------------------------------------------------------------------------------------------------------------------
ARP/ND summary for ethernet-1/1.0
---------------------------------------------------------------------------------------------------------------------------------------------------------------
  IPv4 ARP entries : 0 static, 1 dynamic
  IPv6 ND  entries : 0 static, 0 dynamic
---------------------------------------------------------------------------------------------------------------------------------------------------------------
Traffic statistics for ethernet-1/1.0
---------------------------------------------------------------------------------------------------------------------------------------------------------------
     Statistics          Rx        Tx
  Packets             12940      6877
  Octets              13474575   490216
  Discarded packets   3745       0
  Forwarded packets   9195       3158
  Forwarded octets    13191209   208943
  CPM packets         -          3719
  CPM octets          -          281273
---------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv4 Traffic statistics for ethernet-1/1.0
---------------------------------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------------------------------
IPv6 Traffic statistics for ethernet-1/1.0
---------------------------------------------------------------------------------------------------------------------------------------------------------------
```   
    
But what we are really interested in right now is the pure _state_ of it. With SR Linux, it's easy to view the data model of the switch with `info`.
    
We already talked about `info` earlier, when we used to look at the configuration of the switch. The `running` and `candidate` configuration is also part of the data model of the switch, but it's specifically part of the data model that we can modify. What about the parts of the data model that are read-only?
    
There's a third "mode", or more properly, datastore, in SR Linux called `state`, which exposes those read-only parameters to us!

```
--{ + running }--[  ]--
A:leaf1# enter state

--{ + state }--[  ]--
A:leaf1# info interface ethernet-1/1 statistics
    interface ethernet-1/1 {
        statistics {
            in-packets 11718
            in-octets 13366838
            in-unicast-packets 11702
            in-broadcast-packets 0
            in-multicast-packets 0
            in-discarded-packets 16
            in-error-packets 0
            in-fcs-error-packets 0
            out-packets 6715
            out-octets 573096
            out-mirror-octets 0
            out-unicast-packets 5678
            out-broadcast-packets 6
            out-multicast-packets 1031
            out-discarded-packets 0
            out-error-packets 0
            out-mirror-packets 0
            carrier-transitions 0
        }
    }
```
    
Instead of doing `enter state`, we can ask our `info` command to fetch information from a specific datastore with the `from <datastore>` keyword. Let's try it out after jumping back into `running` mode!

```
--{ running }--[  ]--
A:leaf1# info from state interface ethernet-1/2 statistics                         
    interface ethernet-1/2 {
        statistics {
...
        }
    }
```

Here are all the counters we were looking for, presented in a structured manner.

The SR Linux CLI also has more tricks up its sleeve!
    
Try to get the following outputs:
- The interface counters of `ethernet-1/1` as YAML
- Only the in and out bytes counters of `ethernet-1/1`, as a table 
- Only the non-zero in and out bytes counters of _all interfaces_, as a table
- An automatically refreshing YAML of error statistics on _all interfaces_, as YAML

>[!TIP]
> You can find an excellent write-up of the [SR Linux CLI capabilities here.](https://learn.srlinux.dev/get-started/cli/)


> ✅ **Task 2.5 Solutions**
> <details>
> <summary>YAML interface counters</summary>
> 
> ```
> --{ + running }--[  ]--
> A:leaf1# info from state interface ethernet-1/1 statistics | as yaml
> ---
> interface:
>   - name: ethernet-1/1
>     statistics:
>       in-packets: 13823
>       in-octets: 13513838
>       in-unicast-packets: 13800
> ...
> ```
> </details>
>     
> <details>
> <summary>Interface in-out octet counter table</summary>
> 
> ```
> --{ + running }--[  ]--
> A:leaf1# info from state interface ethernet-1/1 statistics | filter fields in-octets out-octets | as table
> +---------------------+----------------------+----------------------+
> |      Interface      |      In-octets       |      Out-octets      |
> +=====================+======================+======================+
> | ethernet-1/1        |             13514758 |               881480 |
> +---------------------+----------------------+----------------------+
> ```
> </details>
>     
> <details>
> <summary>All interface non-zero in-out octet counter table</summary>
> 
> ```
> --{ + running }--[  ]--
> A:leaf1# info from state interface * statistics | filter non-zero fields in-octets out-octets | as table
> +---------------------+----------------------+----------------------+
> |      Interface      |      In-octets       |      Out-octets      |
> +=====================+======================+======================+
> | ethernet-1/1        |             13515145 |               882283 |
> | ethernet-1/2        |               210689 |             13527746 |
> | mgmt0               |              6314008 |             58086298 |
> +---------------------+----------------------+----------------------+
> ```
>     
> </details>
>     
> <details>
> <summary>Refreshing YAML all interface error counters</summary>
> 
> ```
> --{ + running }--[  ]--
> A:leaf1# watch info from state interface * statistics | filter fields in-error-packets out-error-packets | as yaml
> 
> Every 2.0s: info from state interface * statistics | filter fields in-error-packets out-error-packets | as yaml                  (Executions 3, Fri 04:36:22PM)
> 
> ---
> interface:
>   - name: ethernet-1/1
>     statistics:
>       in-error-packets: 0
>       out-error-packets: 0
>   - name: ethernet-1/2
>     statistics:
>       in-error-packets: 0
>       out-error-packets: 0
> ...
> ```
> </details>

### Task 2.4: Installing Edgeshark

Edgeshark is a tool that focuses on discovery and visualization of container networking. It is also capable of working with multiple namespaces, like we do to create our Containerlab wiring!
    
Edgeshark makes it easy to run packet captures on discovered interfaces and to stream it directly into a remote Wireshark session. Let's install it on our VM, and use it to capture a few packets!

Edgeshark can be deployed with a single line command using Docker Compose:
    
```shell
curl -sL \
  https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml \
  | DOCKER_DEFAULT_PLATFORM= docker compose -f - up -d
```

Edgeshark runs on port 5001, you can access it on <ID>ws.ip.horse:5001
    
More information can be found on [Edgeshark's homepage.](https://edgeshark.siemens.io/#/)
    
### Task 2.5: Containerlab VS Code extension

Containerlab was primarily a CLI tool, until recently. Last year, the first prototype of the VS Code extension dropped around this time of the year, and since then, it developed into a fully-featured GUI of Containerlab. It would be a crime to miss out on trying out this extension!
    
If you haven't done so in "Section 1 and a Bit More" yet, this is the time to install it!
The Containerlab VS Code extension can be installed from the official VS Code marketplace.
    
![Installing Containerlab VS Code extension](./images/vscode-extension-install.png)

Key feature of the containerlab VS Code extension are:
- Manage the lab lifecycle (deploy, destroy, redeploy, inspect)
- Show and visualize running labs, and undeployed labs in the current project directory
- Create and edit topologies with a visual topology editor
- Capture link traffic and manage link impairments

>[!TIP]
> For those who are using VS Code on their local machines: you can connect your VS Code instance using the Remote-SSH extension.  
> To simplify connecting to the VM, paste the following entry to your ~/.ssh/config file:
> ```
> Host denog-workshop
>    Hostname <ID>.ws.ip.horse
>    Port 2222
>    User workshop
>    ForwardAgent yes
> ```
> You can use the Remote-SSH extension (after it's installed) by clicking on the blue button on the bottom left corner of your VS Code window.  
> In the popup menu, just select `denog-workshop` afterwards.
    
   
Let's give it a spin! The Containerlab extension should be able to discover your running lab topology already. Next to your lab is a status indicator - red means a container is unhealthy in the topology, yellow means deployment in progress, and green is for successfully deployed lab topologies.  

- Clicking on a lab topology's name opens the topology viewer on the lab.
- Clicking on the arrow next to the lab topology's name reveals the nodes running in the topology. By right clicking these nodes, you can do various actions, like connecting to the node via SSH in the built-in VS Code terminal, or running a command inside them.  
- Next to the node name are three buttons - view logs, open a shell inside the container (via Docker exec), and connect via SSH.
- Clicking on a node inside a topology shows the node interfaces, on which you can perform packet captures and set link impairments.

Pretty cool, eh?  
Below the running labs section is the undeployed labs one.  
    
Here, use "Create new topology with Topology Editor" button to open the topology editor, and put together the same topology that we created Task 1.1, just to get some practice in!

![Topology Editor](./images/vscode-extension-topoeditor.png)

    
### Task 2.6: Traffic monitoring & manipulation 
#### Traffic monitoring

You might have noticed the shark fin logo either in the Topology Viewer, or in the list of interfaces in your running lab. **Click it!**
    
![Built-in Wireshark in VS Code](./images/vscode-extension-wireshark.png)

Now you are capturing traffic from a lab topology running a few hundred-thousand kilometers away.  
Way cool. 😎
    
This packet capture is powered by Edgeshark and a containerized, VNC-enabled Wireshark instance that starts and stops on-demand, for your packet capture needs.
    
If you would like to capture traffic from the CLI though, there's a way to do that as well.  
First of all, make sure `tcpdump` installed on the host.  
Then run the following command, substituting the node name and interface name, like so:

``` shell
$ ip netns list
clab-denog-workshop-client2
clab-denog-workshop-client1
clab-denog-workshop-leaf1
clab-denog-workshop-leaf2
# Choose a node name (which is the network namespace too)
$ ip netns exec clab-denog-workshop-leaf2 tcpdump -ni eth1
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
20:52:16.558970 IP 10.0.0.2.48652 > 10.0.0.1.179: Flags [P.], seq 847948919:847948938, ack 2262477350, win 444, options [nop,nop,TS val 908563372 ecr 2917566949], length 19: BGP
20:52:16.601343 IP 10.0.0.1.179 > 10.0.0.2.48652: Flags [.], ack 19, win 31306, options [nop,nop,TS val 2917596984 ecr 908563372], length 0
20:52:16.660497 IP 10.0.0.1.179 > 10.0.0.2.48652: Flags [P.], seq 1:20, ack 19, win 31306, options [nop,nop,TS val 2917597043 ecr 908563372], length 19: BGP
```
    
#### Traffic manipulation
    
We won't stop at traffic monitoring though, we can also test different link impairments' effects on the network topology, such as:
- Delay
- Jitter
- Loss
- Rate-limiting
- Packet corruption

The easiest way to manage these link impairments is the VS Code extension. There's also a way to apply and remove them via the CLI:
    
```shell
$ containerlab tools netem set -n clab-denog-workshop-leaf2 -i eth1 --delay 100ms --jitter 5ms
╭───────────┬───────┬────────┬─────────────┬─────────────┬────────────╮
│ Interface │ Delay │ Jitter │ Packet Loss │ Rate (Kbit) │ Corruption │
├───────────┼───────┼────────┼─────────────┼─────────────┼────────────┤
│ eth1      │ 100ms │ 5ms    │ 0.00%       │ 0           │ 0.00%      │
╰───────────┴───────┴────────┴─────────────┴─────────────┴────────────╯

$ docker exec -it clab-denog-workshop-leaf2 ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=104 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=105 ms

$ containerlab tools netem set -n clab-denog-workshop-leaf2 -i eth1 --loss 50
╭───────────┬───────┬────────┬─────────────┬─────────────┬────────────╮
│ Interface │ Delay │ Jitter │ Packet Loss │ Rate (Kbit) │ Corruption │
├───────────┼───────┼────────┼─────────────┼─────────────┼────────────┤
│ eth1      │ 0s    │ 0s     │ 50.00%      │ 0           │ 0.00%      │
╰───────────┴───────┴────────┴─────────────┴─────────────┴────────────╯

$ sudo ip netns exec clab-denog-workshop-leaf2 ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=1.89 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=2.68 ms
64 bytes from 10.0.0.1: icmp_seq=5 ttl=64 time=2.07 ms
64 bytes from 10.0.0.1: icmp_seq=6 ttl=64 time=2.39 ms

$ containerlab tools netem reset -n clab-denog-workshop-leaf2 -i eth1
Reset impairments on node "clab-denog-workshop-leaf2", interface "eth1"

$ docker exec -it clab-denog-workshop-leaf2 ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=2.14 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=2.33 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=2.23 ms
```

More information on configuring link impairments in containerlab topologies can be found [here.](https://containerlab.dev/cmd/tools/netem/set/)
    
This is the time to break things, so go ahead and play with these settings!

## Section 3: Telemetry for our Virtual Data Center

Let's build a streaming telemetry-based monitoring system for our virtual data center!
    
We're going to be using the following building blocks:
- **gNMIc**
gNMIc is an open-source gNMI client, which allows you to collect information via streaming telemetry from a gRPC-enabled network node. For example, in this topology, SR Linux works with gNMI out of the box.
- **Prometheus**
Prometheus is a very popular time-series database (TSDB). We will use it to query and store the data collected by gnmic.
- **Grafana**
Grafana is the one of the most popular tools for data visualization. It is used across various industries, and allows us to visualize the data that is stored in the Prometheus TSDB.  

### Task 3.1: gNMIc setup

[gNMIc](https://gnmic.openconfig.net/) can be used as a CLI tool to create telemetry subscriptions. Let's install it!
    
```shell
$ bash -c "$(curl -sL https://get-gnmic.openconfig.net)"
Downloading https://github.com/openconfig/gnmic/releases/download/v0.42.1/gnmic_0.42.1_linux_aarch64.tar.gz
Preparing to install gnmic 0.42.1 into /usr/local/bin
gnmic installed into /usr/local/bin/gnmic
version : 0.42.1
 commit : 6b35566f
   date : 2025-10-20T17:04:55Z
 gitURL : https://github.com/openconfig/gnmic
   docs : https://gnmic.openconfig.net
```
    
Now to try it out!
    
```json
$ gnmic -a clab-denog-workshop-leaf1 -u admin -p NokiaSrl1! --skip-verify -e json_ietf get --path "/interface[name=ethernet-1/1]/statistics"
[
  {
    "source": "clab-denog-workshop-leaf1",
    "timestamp": 1762551545696935387,
    "time": "2025-11-07T22:39:05.696935387+01:00",
    "updates": [
      {
        "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]/statistics",
        "values": {
          "srl_nokia-interfaces:interface/statistics": {
            "carrier-transitions": "0",
            "in-broadcast-packets": "15",
            "in-discarded-packets": "28",
            "in-error-packets": "0",
            "in-fcs-error-packets": "0",
            "in-multicast-packets": "0",
            "in-octets": "13736993",
            "in-packets": "16497",
            "in-unicast-packets": "16454",
            "out-broadcast-packets": "6",
            "out-discarded-packets": "0",
            "out-error-packets": "0",
            "out-mirror-octets": "0",
            "out-mirror-packets": "0",
            "out-multicast-packets": "2530",
            "out-octets": "1226245",
            "out-packets": "13117",
            "out-unicast-packets": "10581"
          }
        }
      }
    ]
  }
]
```
    
Now, the CLI command is really, really long... So let's create a [client configuration](https://gnmic.openconfig.net/user_guide/configuration_intro/) in `~/.gnmic.yaml` with the username, password, encoding and skip-verify set!

> ✅ **Task 3.1 Solution**
> 
> <details>
> <summary>gNMIC config solution | ~/.gnmic.yaml</summary>
> 
> ```
> username: admin
> password: NokiaSrl1!
> port: 57400
> timeout: 5s
> skip-verify: true
> encoding: json_ietf
> ```
> </details>

Now we can use gNMIc with much less configuration:

```json
$ gnmic -a clab-denog-workshop-leaf1 get --path "/interface[name=ethernet-1/1]/statistics/in-octets"
[
  {
    "source": "clab-denog-workshop-leaf1",
    "timestamp": 1762551944417989285,
    "time": "2025-11-07T22:45:44.417989285+01:00",
    "updates": [
      {
        "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]/statistics/in-octets",
        "values": {
          "srl_nokia-interfaces:interface/statistics/in-octets": "13738800"
        }
      }
    ]
  }
]
```
    
If we switch from `get` to `subscribe`, we will see how streaming telemetry works!

There are three main modes of streaming telemetry with gNMI: **ON_CHANGE, SAMPLE** and **TARGET_DEFINED**.
- **ON_CHANGE** gives you the changed leaf instantly as it changes value. Perfect for leafs that seldom change between fixed states, like interface operational state. Often changing leafs that don't have fixed values, like a packet counter, will generate a lot of streaming telemetry traffic.
- **SAMPLE**, however, is great for leafs that often change, and need to be sent to the telemetry collector frequently. However, changes happening inside the sample period will be hidden by the sampling frequency, so you might miss an interface flap!
- **TARGET_DEFINED** is a convenience mode that lets the streaming telemetry target decide whether the leaf should be sampled or every change of it should trigger a new streaming telemetry event. It is up to the vendor to define what mode a leaf should be telemetry streamed with.

Let's subscribe to the interface operational states of `ethernet-1/1` and `ethernet-1/2` on the following path: `/interface[name=ethernet-1/{1,2}]/oper-state`. First, let's use the **SAMPLE** stream mode:

```json
$ gnmic -a clab-denog-workshop-leaf1 subscribe --stream-mode SAMPLE --path "/interface[name=ethernet-1/{1,2}]/oper-state"
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553165",
  "timestamp": 1762553165999441826,
  "time": "2025-11-07T23:06:05.999441826+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "up"
        }
      }
    }
  ]
}
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553165",
  "timestamp": 1762553166000053201,
  "time": "2025-11-07T23:06:06.000053201+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/2]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "up"
        }
      }
    }
  ]
}
...
```
    
We'll receive a new update from the switch every second for both interfaces. If we change the interface `admin-state` to disabled on `ethernet-1/2` and commit, we'll see the change at the end of the next sampling period:
    
```json
...
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553165",
  "timestamp": 1762553181002857125,
  "time": "2025-11-07T23:06:21.002857125+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "up"
        }
      }
    }
  ]
}
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553165",
  "timestamp": 1762553181003893417,
  "time": "2025-11-07T23:06:21.003893417+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/2]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "down"
        }
      }
    }
  ]
}
```

However, this generated a lot of telemetry traffic that effectively gave us no new information. Let's try the more appropriate **ON_CHANGE** mode after we turn `ethernet-1/2` back on:

```json
$ gnmic -a clab-denog-workshop-leaf1 subscribe --stream-mode ON_CHANGE --path "/interface[name=ethernet-1/{1,2}]/oper-state"
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553197",
  "timestamp": 1762553197997888508,
  "time": "2025-11-07T23:06:37.997888508+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "up"
        }
      }
    }
  ]
}
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553197",
  "timestamp": 1762553197998437550,
  "time": "2025-11-07T23:06:37.99843755+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/2]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "up"
        }
      }
    }
  ]
}
{
  "sync-response": true
}
```

At the beginning, the state is synchronized: both interfaces' current state is sent. However, after this, complete silence. The moment we disable the interface though...

```json
{
  "source": "clab-denog-workshop-leaf1",
  "subscription-name": "default-1762553197",
  "timestamp": 1762553217550254934,
  "time": "2025-11-07T23:06:57.550254934+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/2]",
      "values": {
        "srl_nokia-interfaces:interface": {
          "oper-state": "down"
        }
      }
    }
  ]
}
```

Note how only an update for `ethernet-1/2` was sent! Since `ethernet-1/1` is unaffected, no new telemetry update was sent about it.
    
Now, let's try to run gNMIc in a bit different way!
We ran gNMIc as an interactive CLI tool. However, we can also start gNMIc as a server, exposing the data it collects to other services!

Using the config below, we can run gNMIc as as Prometheus _scrape endpoint_.
    
```yaml
username: admin
password: NokiaSrl1!
port: 57400
timeout: 5s
skip-verify: true
encoding: json_ietf
targets:
  leaf1:
    address: clab-denog-workshop-leaf1
    subscriptions:
      - srl-system-performance
      - srl-if-stats

subscriptions:
  srl-system-performance:
    mode: stream
    stream-mode: sample
    sample-interval: 5s
    paths:
      - /platform/control[slot=*]/cpu[index=all]/total
      - /platform/control[slot=*]/memory
  srl-if-stats:
    mode: stream
    stream-mode: sample
    sample-interval: 5s
    paths:
      - /interface[name=ethernet-1/*]/oper-state
      - /interface[name=ethernet-1/*]/statistics
      - /interface[name=ethernet-1/*]/traffic-rate
 
processors:
  trim-prefixes:
    event-strings:
      value-names:
        - "^/state/.*"
      transforms:
        - trim-prefix:
            apply-on: "name"
            prefix: "/state/"

  oper-state-to-int:
    event-strings:
      value-names:
        - ".*"
      transforms:
        - replace:
            apply-on: "value"
            old: "up"
            new: "1"
        - replace:
            apply-on: "value"
            old: "down"
            new: "0"

outputs:
  prom-output:
    type: prometheus
    listen: :9273
    event-processors:
      - trim-prefixes
      - oper-state-to-int
```

The configuration is made up of four different parts:
- Generic settings
- Telemetry targets and assigning subscriptions to them
- Subscriptions path groups (and subscription settings)
- Processor rules, trimming the path prefixes and turning up/down to 1/0.
- Outputs, in this case, a single Prometheus scrape endpoint, with associated processor rules

After applying this config, let's start gNMIc with `gnmic subscribe`.
You won't see any output, but the server is running!
    
Let's run `curl http://localhost:9273/metrics` on a different terminal, while keeping the previous terminal open:
```
$ curl http://localhost:9273/metrics
# HELP srl_nokia_interfaces_interface_oper_state gNMIc generated metric
# TYPE srl_nokia_interfaces_interface_oper_state untyped
srl_nokia_interfaces_interface_oper_state{interface_name="ethernet-1/1",source="leaf1",subscription_name="srl-if-stats"} 1
srl_nokia_interfaces_interface_oper_state{interface_name="ethernet-1/10",source="leaf1",subscription_name="srl-if-stats"} 0
...
```
    
Excellent! That's the first building block.

To find out what paths we need to subscribe to get specific information, there are two ways of getting the path:
- Using the SR Linux CLI's `pwc xpath` command after entering a specific part of the state data store:
```
--{ + state }--[  ]--
A:leaf1# network-instance default protocols bgp neighbor 10.0.0.2

--{ + state }--[ network-instance default protocols bgp neighbor 10.0.0.2 ]--
A:leaf1# pwc xpath
/network-instance[name=default]/protocols/bgp/neighbor[peer-address=10.0.0.2]
```
- Using the [SR Linux YANG Explorer.](https://yang.srlinux.dev/) This is also a really useful tool for finding the correct configuration commands!

>[!NOTE] 
> This works, because in SR Linux, the CLI is just yet another consumer of the YANG models that the streaming telemetry API (along with all other interfaces) uses.
>
> ![Our memes are also model driven](./images/modeldrivenmeme.png)
    
### Task 3.2: Prometheus setup
    
**Prometheus** is one of the most popular time series databases (TSDB). It is an open-source solution and comes with additional monitoring capabilities. 
As it is widely being used, lots of integration work has already been done.
    
In Prometheus terminology, the data source from where data is being fetched is called a `scraping endpoint`. This is usually a simple HTTP endpoint from where Prometheus reads the data in a promlog format. 
    
The Prometheus configuration is defined in a well structured YAML file. 
    
The Prometheus docs can be found [here](https://prometheus.io/docs/introduction/overview/).

Now, to write a Prometheus configuration that works with the gNMIc endpoint we just set up, we'll need to configure the following:
- Matching scrape interval of 5s
- A single static scrape target, `<ID>.ws.ip.horse:9273` 

> ✅ **Task 3.2 Solution**
> 
> <details>
> <summary>Prometheus config solution | prometheus.yaml</summary>
> 
> ```yaml
> global:
>   scrape_interval: 5s
> 
> # metrics_path defaults to '/metrics'
> # scheme defaults to 'http'.
> scrape_configs:
>   - job_name: "gnmic"
>     static_configs:
>       - targets: ["<ID>.ws.ip.horse:9273"]
> ```
    
To start Prometheus, instead of installing it as any other application, we're going to be using Docker yet again!
    
```
docker run --name prometheus -v ./prometheus.yaml:/etc/prometheus/prometheus.yml -d -p 9090:9090 prom/prometheus
```

Once it starts, you should be able to connect to the Prometheus web UI at port 9090.
    
![Prometheus metrics](./images/prom-metrics.png)

You can use the query builder to create various types of queries, such as filtering on interface name based on the labels of the metrics.
    
![Prometheus query label filtering](./images/prom-label-filter.png)

Try to capture only a specific interface's operation state!


You can also view a graph of the values, sort of like Grafana, but not with as many bells and whistles! This shows that Prometheus stores the data, unlike gNMIc.

![Prometheus time series graph](./images/prom-graph.png)

>[!IMPORTANT]
> You should stop the Prometheus container by running `docker rm -f prometheus`.
> Since we're about to deploy it again in the next step, please do so, otherwise the port will be already in use.

#### Task 3.3 Grafana

**Grafana** is a very popular tool for data visualization and alarming. It has a many types of charts, graphs and other visualization methods like heat maps, and supports many data sources. One of the data sources Grafana can utilize is Prometheus.
    
The Grafana docs can be found [here](https://grafana.com/docs/).

Now, instead of adding yet another container running on our network, we're going to put all pieces of our telemetry stack into our Containerlab topology!
    
#### Putting the telemetry into the topology
    
First of all, this means you should destroy the existing, running topology. Make sure to save all your work, into the startup configuration files of `leaf1` and `leaf2`!
    
Now, to add the telemetry containers, we'll have to add the following nodes to the topology:
- **gNMIc**: Let's use the latest stable image, `ghcr.io/openconfig/gnmic:0.42.1`.  
    We will have to bind mount the gNMIc configuration file in our home directory to `/gnmic.yaml`. Finally, append the `--config /gnmic.yaml subscribe` to the command of the container so that it picks our configuration file up, and starts the subscription. We don't need to expose port 9273 here, since that will only be used inside the Docker network.
- **Prometheus**: We can use the image we used earlier! Again, we will need to bind mount `prometheus.yaml` inside the container. Also, we will need to expose port 9090 of the container to the outside, this can be done easily in the Containerlab topology as well.
- **Grafana**: the last piece of the puzzle! `grafana/grafana:11.2.0` is the image we are going to use. For now, no bind mounts, and we will just have to expose port 3000 of the container.

Make sure to call the nodes `gnmic`, `prometheus` and `grafana`, so we can refer to them inside configurations in an easy manner!  
Since we only connect these services through the management network, no additional links are required in the topology.
    
Once you are done with putting the topology together, try to deploy it and connect to port 9090 of the VM once again with your web browser. You should see some SR Linux metrics already flowing in from `leaf1`.
    
![Prometheus metrics in the deployed topology](./images/prom-deployed.png)

> ✅ **Task 3.3 Solution**
> 
> <details>
> <summary>Container topology w/ telemetry solution | workshop.clab.yaml</summary>
> 
> ```yaml
> name: denog-workshop
> 
> topology:
>   nodes:
>     leaf1:
>       kind: nokia_srlinux
>       image: ghcr.io/nokia/srlinux:24.10
>       startup-config: ./config/leaf1.cli
>     leaf2:
>       kind: linux
>       image: quay.io/frrouting/frr:10.4.1
>       binds:
>         - ./config/frr-daemons.conf:/etc/frr/daemons
>         - ./config/leaf2.conf:/etc/frr/frr.conf:rw
>     client1:
>       kind: linux
>       image: ghcr.io/srl-labs/network-multitool
>       exec:
>         - ip addr add 192.168.10.2/24 dev eth1
>         - ip route add 192.168.11.0/24 via 192.168.10.1
>         - ip link set eth1 up 
> 
>     client2:
>       kind: linux
>       image: ghcr.io/srl-labs/network-multitool
>       exec:
>         - ip addr add 192.168.11.2/24 dev eth1
>         - ip route add 192.168.10.0/24 via 192.168.11.1
>         - ip link set eth1 up
>     gnmic:
>       kind: linux
>       image: ghcr.io/openconfig/gnmic:0.42.1
>       binds:
>         - ~/.gnmic.yaml:/gnmic.yaml
>       cmd: --config /gnmic.yaml subscribe
>     prometheus:
>       kind: linux
>       image: prom/prometheus
>       binds:
>         - ./prometheus.yaml:/etc/prometheus/prometheus.yml
>       ports:
>         - 9090:9090
>     grafana:
>       kind: linux
>       image: grafana/grafana:11.2.0
>       ports:
>         - 3000:3000
> 
>   links:
>     - endpoints: ["leaf1:ethernet-1/1", "leaf2:eth1"]
>     - endpoints: ["leaf1:ethernet-1/2", "client1:eth1"]
>     - endpoints: ["leaf2:eth2", "client2:eth1"]
> ```
    
#### Configuring data sources in Grafana
Our Grafana deployment is brand new, and has the default credentials admin/admin. Let's change that to "Containerlab!" on the first login.
    
![Welcome to Grafana](./images/grafana-welcome.png)

As the quick start steps suggest, we should add our first data source! Click through the interactive wizard to add the Prometheus data source. Because Docker sets up the DNS entries within the Docker network automagically, we can just use the container's name, that is, `prometheus`, in the URL!
    
![Adding the Prometheus data source](./images/grafana-datasource.png)

#### Setting up your first dashboard
    
Now that we have access to the time series data from Prometheus, let's create our first dashboard. To do so, go back to the Grafana homepage and click the button to create a dashboard.

The first graph we are going to create is a classic graph of interface speed. Our data source is going to be Prometheus, of course. In the query panel, a blank query called "A" is already pre-populated. Let's rename this to "in-bps".
    
The metric field is where you can interactively select the metric from Prometheus to display. Let's pick `srl_nokia_interfaces_interface_traffic_rate_in_bps` here!
    
![Selecting the metric](./images/granafa-selectmetric.png)

By default, Prometheus serves us integers for this time series data, but we know that this is bits/second, so make sure to also tell Grafana that in the right side pane.
    
![Setting the unit on a visualization](./images/grafana-unit.png)

Click Apply to save your changes to your very first visualization in this topology!
    
The next one is going to be somewhat trickier, we're going to visualize the state of the ports across time. How, you might ask? We're going to use a different kind of visualization.
    
![Adding a second visualization from the dashboard](./images/grafana-addvis.png)
![Selecting the visualization type](./images/grafana-vistype.png)

When you first select the metric for the interface operational state, it will look all kinds of messed up...
So, let's fix that!
    
First of all, the legend shown on the visualization is very verbose. Instead, we are going to use a Custom legend, and make it display the value of the `interface_name` label.
    
![Setting a custom legend on a metric](./images/grafana-customlegend.png)

Second, it's all green! This is because by default on this visualization, any value under 80 is shown as green, and over 80 are red, as the panel legend suggests. We can fix this on the right pane.
Delete the 80 threshold, set a value mapping of 0 to red, 1 to green, and set the color scheme to a single color.

![Value mappings and color schemes](./images/grafana-valuemapping.png)
    
You should have a working visualization!

![The finished visualization for interface operational states](./images/grafana-operstate.png)

This is how our final dashboard looks like after resizing the panes:

![It's the final dashboard (toot toot tooooooot toot)]()

**Don't forget to save your work!**
You can do so with the little save icon up top. By clicking on the settings icon and selecting the JSON Model tab, you can even find a JSON-serialized version of the dashboard.

To give you an idea of how complex you can get with Grafana, here you can find a complete Grafana telemetry lab in the [SR Linux Telemetry Lab](https://github.com/srl-labs/srl-telemetry-lab/).

## Section 4: Kubernetes based Network Automation & Monitoring

This section is going to be a guided walkthrough; however, if you have already explored the existing topology, and checked out the SR Linux Telemetry Lab, feel free to follow these short instructions to get started early!
    
### Step 1: Install EDA Playground

Following the official docs to install the EDA playground. Docs are available [here](https://github.com/eda-labs/eda-telemetry-lab). We are going to be running in Simulation mode. _Remember to set the EDA external URL variable correctly._
    
### Step 2: Verify EDA Installation

Verification steps include UI access on port 9443, as well as running some kubectl commands to make sure all pods came up correctly.

The verification docs are [here](https://docs.eda.dev/25.8/getting-started/verification/).
    
### Step 3: Deploy EDA Telemetry Lab

The EDA Telemetry Lab is a bundle of configs and tools to end up with leaf/spine topology, and a fully set up telemetry stack on the side, all running inside Kubernetes. The leaf/spine topology is going to be simulated by EDA's Digital Twin simulation engine.

You can find the EDA Telemetry Lab repo is located [here](https://github.com/eda-labs/eda-telemetry-lab).
    
Besides the telemetry stack, this EDA setup allows you to explore how a K8s based automation tool behaves and what is possible when relying on open, standard, high performance interfaces for the communication between the network elements and the automation tool. 
    
### What can you do with it now?
- check out how EDA streams data from the simulated SR Linux network topology 
- explore the EDA Query language to fetch data from your network
- explore existing configurations (Intents) in EDA and create Deviations (and observe what happens)
- change intents and discover how transactional behaviour works in EDA

## Section 5: Ideas to explore

In case you get bored, feel free to 
    
- create more Grafana dashboards (potentially even adding more/other data in gNMIc)
- additional gNMIc features, such as metric transformations
- explore Edgeshark, capture more traffic!!
- explore other existing containerlab topologies (e.g [here on GitHub](https://github.com/topics/clab-topo)) 
- break stuff :)
