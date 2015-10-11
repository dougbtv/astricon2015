name: inverse
layout: true
class: center, middle, inverse
---
#Asterisk using Docker and CoreOS
.footnote[By Douglas K. Smith, 800response]
---
layout: false
.left-column[
  ## Who am I?
]
.right-column[

### I'm Doug Smith

- Hailing from Vermont

- Member of Burlington, Vermont's hackerspace [LaboratoryB.org](http://laboratoryb.org)

- Into systems & development

- Develop Asterisk solutions every day.

]
---
layout: false
.left-column[
  ## 800response
]
.right-column[

- CallFinder

- Speech Analytics

- Strong telephony background

- We implement lots of FOSS

]
???

---
name: inverse
layout: true
class: center, middle, inverse
---
#10,000' view
---
layout: false
.left-column[
  ## This is documented online
]
.right-column[
  # You can find this online @ [astricon.dougbtv.com](astricon.dougbtv.com)

  - DIY Workshop on Github

  - Whole presentation is in markdown.

  - Twitter [@dougbtv](https://twitter.com/dougbtv)

]
---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker?
]
.right-column[
## Docker
* Component re-use
* Dependency management
* Congruency between development & production

### Another choice?
* [RKT](https://coreos.com/rkt/docs/) ("Rocket") for containers


]
???

  Docker & CoreOS are great because they give a few advantages that keep us clean, give us consistent environments, and tools that help with scale. 

  * Docker is great for component re-use, sharing & team development, rapid deployment, and simplified maintenance by reducing the risk of problems with application dependencies.

  * CoreOS is a *very light* Linux that allows us to run our containers, and comes chock full of simple tools to help us maintain a cluster of machines, using tools like `etcd` and `fleet` 

  * ...You might not have to buy a hypervisor anymore. (But you can still use one.)

  * They're not the only options.

  * For containers: [RKT](https://coreos.com/rkt/docs/) (said, "rocket")

  * For OS: [Project Atomic](http://www.projectatomic.io/download/), a Fedora for running containers, and uses Kubernetes for management.

---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker?
  ## Why CoreOS?
]
.right-column[
## CoreOS
* Security, consistency & reliability
* Cluster tools
  * Fleet
  * etcd

### Other choice?
* [Project Atomic](http://www.projectatomic.io/download/) for OS


]
???

...notes

---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker?
  ## Why CoreOS?
  ## Considerations
]
.right-column[

* High Availability

* Scalability

* Visibility

]
---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker?
  ## Why CoreOS?
  ## Considerations
  ## Overview
]
.right-column[

* Technology intro / review

* Getting CoreOS running

* Sample system architecture

* `kamailio-etcd-dispatcher`

* Continuous Deployment Opportunities

]
---
name: inverse
layout: true
class: center, middle, inverse
---
# Intro to the technologies
---
layout: false
.left-column[
  ## Docker
]
.right-column[

  *Docker allows you to package an application with all of its dependencies into a standardized unit for software development.*

  - Portability & congruency

  - A great way to manage running processes

  - Rather convenient for developers

```bash
FROM fedora:latest
MAINTAINER Doug Smith <info@laboratoryb.org>
RUN yum install -y cowsay
```

So you could serve a file:

```bash
docker run -it dougbtv/cowsay \
/usr/bin/cowsay -s "Vermont Is Awesome"
# ____________________ 
#< Vermont is awesome >
# -------------------- 
#        \   ^__^
#         \  (**)\_______
#            (__)\       )\/\
#             U  ||----w |
#                ||     ||
```
]
???

Docker is a way to really use LXC (Linux Containers).
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
]
.right-column[
  CoreOS is a light-weight OS to run your containers on.

  - Just a couple hundred megs.

  - Fork of ChromeOS

  - Bootstrapped with simple tools to manage and maintain a cluster.

  - Your containers run the Linux flavor that you're used to.

  - Run it where you want: In the closet, in a public/private cloud.
]
???

# Oh no another O/S? 
Don't worry.
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
]
.right-column[
## etcd is a discovery service.

  - Distributed key-value pair database 

  - Accessed via REST API

  - Used by CoreOS for `fleet`

  - A way to dynamically store configurations, like the simplest one: "Hey I'm a service and I live here!"
]
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
  ## Fleet
]

.right-column[
## Fleet is your scheduler

- An init system at a cluster level
  - Like `systemctl` across boxes

- Deploy containers on arbitrary hosts

- Maintain *N* instances of a service, and re-schedule 

- Rather low level

- Runs Docker containers in systemd-style units

- Allows you to create dependencies

### Other choices in this realm 
- [Kubernetes](http://kubernetes.io/) by Google
- [Mesos](http://mesos.apache.org/) by Apache
]
???

Kubernetes uses an API, and has a number of features such as being an advanced scheduler. It also implements "quotas"
which allow you to specify CPU usage and other metrics at a cluster level and allocate resources more intelligently.

Fleet is a little more manageable for a smaller cluster.
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
  ## Fleet
  ## Kamailio
]
.right-column[
 ## You probably know Kamailio!

  - Used as a load balancer

  - Add a VIP using `keepalived`

  - Use `kamailio-etcd-dispatcher` for service discovery

  - CI/CD opportunities

]
???

  - As we know, Asterisk is a B2BUA not a proxy.

  - We'll use it for load balancing our cluster of Asterisk machines.

  - Use it in concert with `keepalived` which will provide us with a VIP for an HA load balancer
    - If you use AWS you probably want to use [Elastic IPs and an API](https://aws.amazon.com/articles/2127188135977316)

  - We use a custom application `kamailio-etcd-dispatcher` which dynamically builds instructions for Kamailio's Dispatcher module

  - We can make a "canary release" easily by automatically rebalancing our cluster using `kamailio-etcd-dispatcher`


At 11:40 you can hear Kyle Marks talk about "Fronting your Asterisk cluster with, Asterisk"

I'll tell you if I like his idea, I will implement it. The bottom line is that I feel like I have such higher confidence in diagnosing issues on my Asterisk boxen, over Kamailio.

By the same token, Kamailio hasn't failed me.

More is still needed here, especially in terms of replicated dialogue state. Which with Kamailio, will require some database redundancy which is a big chunk to chew off in conjunction with what we're covering right here.

Don't use keepalived in AWS. Since they don't like multicast traffic, it's not worth it. It's honestly cleaner to use an API and fail over on your own monitoring conditions.

---
layout: false
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
  ## Fleet
  ## Kamailio
  ## Homer
]
.right-column[
 ## Homer is a SIP capture server

  - Get some visibility of all the signalling whipping over your network.

  - Runs on a (likely familiar) LAMP stack.

  - More info in the DIY workshop

]
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
  ## Fleet
  ## Kamailio
  ## Homer
  ## Supporting tools
]
.right-column[

- Flannel 
  - Packaged with CoreOS for network overlay
  
- Configuration Management, and you need it.
  - Ansible, Salt, Puppet, Terraform, etc.

- Logging (you'll want a centralized log server)

]
???

Config management choices:
- salt
- ansible
- puppet
- terraform (multiple clouds)
---
name: inverse
layout: true
class: center, middle, inverse
---
# . . . Oh yeah! 
# Asterisk, too.
---
name: inverse
layout: true
class: center, middle, inverse
---
# Getting CoreOS & Fleet running
---
layout: false
.left-column[
  ## Cloud-init
]
.right-column[

- The *defacto* package to handle early initialization of cloud instances
  - Not just specific to CoreOS

- We use the `#cloud-config` flavor

- It's just YAML

- Set your SSH keys

- Define some networking properties.

- Define properties for services.

]
???

This might be familiar if you're already using OpenStack or AWS or Digital Ocean, etc.

Start up things like etcd, flanneld, docker.

---
layout: false
.left-column[
  ## Cloud-init
  ## etcd
]
.right-column[

- Your discovery service uses a discovery service, `discovery.etcd.io`

- This will probably be the hardest part.

- If you use ASTDB, you already know how to use a key-value store.

- Give it a whirl.

```bash
$ etcdctl set /foo/bar "Quux"

$ etcdctl set /foo/bar "Hello world" --ttl 60

$ etcdctl get /foo/bar
Hello world
```

]
???

# The discovery service.

  - Generate a key, every box uses the same key
  - The key is set in the `cloud-config`
  - ...You can use a private discovery service, too.


Public discovery service isn't so bad, all it's going to store are a few private IPs.

---
layout: false
.left-column[
  ## Cloud-init
  ## etcd
  ## Fleet units
]
.right-column[

- [Get familiar with systemd](https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/)

- They're really just systemd units with an `[X-Fleet]` section.

- Define service to run & it's dependancies and conflicts.

- Can use meta-data from `cloud-config`

```bash
[Unit]
Description=Cowsay
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill cowsay
ExecStartPre=-/usr/bin/docker rm cowsay
ExecStartPre=/usr/bin/docker pull dougbtv/cowsay:latest
ExecStart=/usr/bin/docker run \
    --name cowsay \
    -t dougbtv/cowsay \
    cowsay 'Buy Maple Syrup'
ExecStop=/usr/bin/docker stop cowsay

[X-Fleet]
Conflicts=cowsay@*.service
MachineMetadata=boxrole=cowsay
```

]
???
## Philosophy

The idea is to run a docker container in the foreground, and let's output go to stdout / stderr, which great for...

## Dependencies & Conflicts

You might not want to run a database container in the same place as a backup container. 

But you will want to run your kamailio on the same box as a keepalived.

## Journald

Journal d is great
 
## Meta data

You might specify different metadata here to specify hardware configurations, like a database might have different hardware requirements than X.

In this example we use it to logically group together services, so for example we say "these boxes are for kamailio" and "these other boxes are for"

---
layout: false
.left-column[
  ## Cloud-init
  ## etcd
  ## Fleet units
  ## Fleet instances
]
.right-column[

- Starting an instance of Asterisk
```bash
$ fleetctl start asterisk@1
$ fleetctl start asterisk@2
[...]
$ fleetctl start asterisk@100
```

- Showing our running instances
```bash
$ fleetctl list-units
```

]
???
 
# notes

---
layout: false
.left-column[
  ## Cloud-init
  ## etcd
  ## Fleet units
  ## Fleet instances
  ## Fleet features
]
.right-column[

- Getting logs for anything in the cluster
```bash
$ fleetctl journal asterisk@15
```

- SSH to any box
```bash
$ fleetctl ssh asterisk@15
```
]
???
 
# notes

---
layout: false
.left-column[
  ## Cloud-init
  ## etcd
  ## Fleet units
  ## Fleet instances
  ## Fleet features
  ## Tips
]
.right-column[

* When in doubt... check the etcd logs

* Think *n+1*

  * An extra machine for utility

  * An extra machine for each class

]
???
 
# Everything hinges off etcd, luckily it's distributed.

---
name: inverse
layout: true
class: center, middle, inverse
---
# System Architecture
---
layout: false
## Ultra-simple business case

* Market research has discovered a pent-up customer demand to hear the sound of screaming monkeys.

* User story: Customer dials a phone and hears the sound of monkeys.

* We connect to a managed gateway / ITSP, conviently on the same subnet.

## In reality...

* You'll probably have layers of this topology

* Your networking will probably be more complex

???

## conveniences to avoid NAT and other network wigginess
---
layout:false
background-image: url(/images/platform_stack.png)
---
layout:false
background-image: url(/images/network.png)
---
name: inverse
layout: true
class: center, middle, inverse
---
# Asterisk + Kamailio + Discovery Service
---
layout: false
# Kamailio etcd dispatcher

* Adds service discovery for Asterisk to Kamailio

* Written in Node.js

* On [GitHub](https://github.com/dougbtv/kamailio-etcd-dispatcher) and [NPM as kamailio-etcd-dispatcher](https://www.npmjs.com/package/kamailio-etcd-dispatcher)

* Leverages etcd for redundancy

* Runs in two modes
  * Announcer -- To "announce" where Asterisk boxen are
  * Dispatcher -- alongside Kamailio to update `dispatcher.list` and run `kamctl`

???

## You don't have to care that it's Node.js

You don't have to care what it's written in, since it's intended to be packaged as a docker image & container, you don't need to worry about babying the stack of technologies. So even if you're not a node.js shop -- it's OK. 

This is one of the powers of containerization, you can let the maintainers of software define how to get it up and running and keep it that way.


---
layout:false
background-image: url(/images/hkam_01.png)
---
layout:false
background-image: url(/images/hkam_02.png)
---
layout:false
background-image: url(/images/hkam_03.png)
---
layout:false
background-image: url(/images/hkam_04.png)
---
layout: false
## Taking an Asterisk box out of rotation

* Asterisk failure, doesn't respond to SIP `OPTIONS` method

* `core shutdown gracefully` will report congestion to Kamailio

* Stopping the announcer

## Putting 'em back in rotation

* Just start the announcer again

---
name: inverse
layout: true
class: center, middle, inverse
---
# Continuous Deployment Opportunities
---
layout:false
background-image: url(/images/hkam_05.png)
---
layout:false
background-image: url(/images/deploy_nominal.png)
---
layout:false
background-image: url(/images/deploy_canary.png)
---
layout:false
background-image: url(/images/deploy_bluegreen.png)
---
name: inverse
layout: true
class: center, middle, inverse
---
# Thank you!
.footnote[Available online @ [astricon.dougbtv.com](http://astricon.dougbtv.com)]
