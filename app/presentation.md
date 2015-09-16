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

### I'm Doug and this is a STUB. 

- Develop Asterisk solutions every day.

- Member of Burlington, Vermont's hackerspace [LaboratoryB.org](http://laboratoryb.org)

- Dig devops, fullstack JavaScript, and lots of open source

- Live in Vermont, yadda yadda yadda

]
---
layout: false
.left-column[
  ## 800response
]
.right-column[
  800response is a company, stub.

- Speech Analytics

- VoIP / Telephony

- Open source technologies

- This is a stub...

]
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
  # You can find this online @ [url.stub.com](url.stub.com)

  - DIY Workshop on Github

  - Whole presentation is in markdown.

  - Twitter [@dougbtv](https://twitter.com/dougbtv)

]
---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker & CoreOS
]
.right-column[
  Docker & CoreOS are great because they give a few advantages that keep us clean, give us consistent environments, and tools that help with scale. 

  * Docker is great for component re-use, sharing & team development, rapid deployment, and simplified maintenance by reducing the risk of problems with application dependencies.

  * CoreOS is a *very light* Linux that allows us to run our containers, and comes chock full of simple tools to help us maintain a cluster of machines, using tools like `etcd` and `fleet` 

  * ...You might not have to buy a hypervisor anymore. (But you can still use one.)

  * They're not the only options.

  * For containers: [RKT](https://coreos.com/rkt/docs/) (said, "rocket")

  * For OS: [Project Atomic](http://www.projectatomic.io/download/), a Fedora for running containers, and uses Kubernetes for management.
]
---
layout: false
.left-column[
  ## This is documented online
  ## Why Docker & CoreOS
  ## First class considerations
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
  ## Why Docker & CoreOS
  ## First class considerations
  ## Overview
]
.right-column[

* Technology intro / review

* Getting CoreOS running

* Sample system architecture

* Kamailio etcd dispatcher

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
  You probably know Kamailio!

  - As we know, Asterisk is a B2BUA not a proxy.

  - We'll use it for load balancing our cluster of Asterisk machines.

  - Use it in concert with `keepalived` which will provide us with a VIP for an HA load balancer
    - If you use AWS you probably want to use [Elastic IPs and an API](https://aws.amazon.com/articles/2127188135977316)

  - We use a custom application `kamailio-etcd-dispatcher` which dynamically builds instructions for Kamailio's Dispatcher module

  - We can make a "canary release" easily by automatically rebalancing our cluster using `kamailio-etcd-dispatcher`

]
???

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
  Homer is a SIP capture server

  - Get some visibility of all the signalling whipping over your network.

  - Runs on a (likely familiar) LAMP stack.

  - We won't go deep into configuration, but, it's in github.

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
  - Generate a key, every box uses the same key
  - The key is set in the `cloud-config`
  - ...You can use a private discovery service, too.

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

The discovery service.

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
layout:false
background-image: url(/images/deploy_scheme.png)
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
# Yo dawg, I heard you like sidekicks...
---
layout:false
background-image: url(/images/hkam_05.png)
---
name: inverse
layout: true
class: center, middle, inverse
---
# Thank you!
.footnote[Available online @ [stub.dougbtv.com](http://dougbtv.com)]
---
name: inverse
layout: true
class: center, middle, inverse
---
#remark
[ri-mahrk]
.footnote[Go directly to [project site](https://github.com/gnab/remark)]
---
## What is it and why should I be using it?
---
layout: false
.left-column[
  ## What is it?
]
.right-column[
  A simple, in-browser, Markdown-driven slideshow tool targeted at people who know their way around HTML and CSS, featuring:

- Markdown formatting, with smart extensions

- Presenter mode, with cloned slideshow view

- Syntax highlighting, supporting a range of languages

- Slide scaling, thus similar appearance on all devices / resolutions .red[*]

- Touch support for smart phones and pads, i.e. swipe to navigate slides

.footnote[.red[*] At least browsers try their best]
]
---
.left-column[
  ## What is it?
  ## Why use it?
]
.right-column[
If your ideal slideshow creation workflow contains any of the following steps:

- Just write what's on your mind

- Do some basic styling

- Easily collaborate with others

- Share with and show to everyone

Then remark might be perfect for your next.red[*] slideshow!

.footnote[.red[*] You probably want to convert existing slideshows as well]
]
---
.left-column[
  ## What is it?
  ## Why use it?
]
.right-column[
As the slideshow is expressed using Markdown, you may:

- Focus on the content, expressing yourself in next to plain text not worrying what flashy graphics and disturbing effects to put where

As the slideshow is actually an HTML document, you may:

- Display it in any decent browser

- Style it using regular CSS, just like any other HTML content

- Use it offline!

As the slideshow is contained in a plain file, you may:

- Store it wherever you like; on your computer, hosted from your Dropbox, hosted on Github Pages alongside the stuff you're presenting...

- Easily collaborate with others, keeping track of changes using your favourite SCM tool, like Git or Mercurial
]
---
template: inverse

## How does it work, then?
---
name: how

.left-column[
  ## How does it work?
### - Markdown
]
.right-column[
A Markdown-formatted chunk of text is transformed into individual slides by JavaScript running in the browser:

```remark
# Slide 1
This is slide 1

---

# Slide 2
This is slide 2
```

.slides[
  .first[
  ### Slide 1
  This is slide 1
  ]
  .second[
  ### Slide 2
  This is slide 2
  ]
]

Regular Markdown rules apply with only a single exception:

  - A line containing three dashes constitutes a new slide
  (not a horizontal rule, `&lt;hr /&gt;`)

Have a look at the [Markdown website](http://daringfireball.net/projects/markdown/) if you're not familiar with Markdown formatting.
]
---
.left-column[
  ## How does it work?
  ### - Markdown
  ### - Inside HTML
]
.right-column[
A simple HTML document is needed for hosting the styles, Markdown and the generated slides themselves:

```xml
<!DOCTYPE html>
<html>
  <head>
    <style type="text/css">
      /* Slideshow styles */
    </style>
  </head>
  <body>
*    <textarea id="source">
      <!-- Slideshow Markdown -->
    &lt;/textarea&gt;
*    <script src="remark.js">
    </script>
    <script>
*      var slideshow = remark.create();
    </script>
  </body>
</html>
```

You may download remark to have your slideshow not depend on any online resources, or reference the [latest version](http://remarkjs.com/downloads/remark-latest.min.js) online directly.
]
---
template: inverse

## Of course, Markdown can only go so far.
---
.left-column[
  ## Markdown extensions
]
.right-column[
To help out with slide layout and formatting, a few Markdown extensions have been included:

- Slide properties, for naming, styling and templating slides

- Content classes, for styling specific content

- Syntax highlighting, supporting a range of languages
]

---
.left-column[
  ## Markdown extensions
  ### - Slide properties
]
.right-column[
Initial lines containing key-value pairs are extracted as slide properties:

```remark
name: agenda
class: middle, center

# Agenda

The name of this slide is {{ name }}.
```

Slide properties serve multiple purposes:

* Naming and styling slides using properties `name` and `class`

* Using slides as templates using properties `template` and `layout`

* Expansion of `{{ property }}` expressions to property values

See the [complete list](https://github.com/gnab/remark/wiki/Markdown#slide-properties) of slide properties.
]
---
.left-column[
  ## Markdown extensions
  ### - Slide properties
  ### - Content classes
]
.right-column[
Any occurences of one or more dotted CSS class names followed by square brackets are replaced with the contents of the brackets with the specified classes applied:

```remark
.footnote[.red.bold[*] Important footnote]
```

Resulting HTML extract:

```xml
<span class="footnote">
  <span class="red bold">*</span> Important footnote
</span>
```
]
---
.left-column[
  ## Markdown extensions
  ### - Slide properties
  ### - Content classes
  ### - Syntax Highlighting
]
.right-column[
Code blocks can be syntax highlighted by specifying a language from the set of [supported languages](https://github.com/gnab/remark/wiki/Configuration#highlighting).

Using [GFM](http://github.github.com/github-flavored-markdown/) fenced code blocks you can easily specify highlighting language:

.pull-left[

<pre><code>```javascript
function add(a, b)
  return a + b
end
```</code></pre>
]
.pull-right[

<pre><code>```ruby
def add(a, b)
  a + b
end
```</code></pre>
]

A number of highlighting [styles](https://github.com/gnab/remark/wiki/Configuration#highlighting) are available, including several well-known themes from different editors and IDEs.

]
---
.left-column[
  ## Presenter mode
]
.right-column[
To help out with giving presentations, a presenter mode comprising the
following features is provided:

- Display of slide notes for the current slide, to help you remember
  key points

- Display of upcoming slide, to let you know what's coming

- Cloning of slideshow for viewing on extended display
]
---
.left-column[
  ## Presenter mode
  ### - Inline notes
]
.right-column[
Just like three dashes separate slides,
three question marks separate slide content from slide notes:

```
Slide 1 content

*???

Slide 1 notes

---

Slide 2 content

*???

Slide 2 notes
```

Slide notes are also treated as Markdown, and will be converted in the
same manner slide content is.

Pressing __P__ will toggle presenter mode.
]
???
Congratulations, you just toggled presenter mode!

Now press __P__ to toggle it back off.
---
.left-column[
  ## Presenter mode
  ### - Inline notes
  ### - Cloned view
]
.right-column[
Presenter mode of course makes no sense to the audience.

Creating a cloned view of your slideshow lets you:

- Move the cloned view to the extended display visible to the audience

- Put the original slideshow in presenter mode

- Navigate as usual, and the cloned view will automatically keep up with the original

Pressing __C__ will open a cloned view of the current slideshow in a new
browser window.
]
---
template: inverse

## It's time to get started!
---
.left-column[
  ## Getting started
]
.right-column[
Getting up and running is done in only a few steps:

1. Visit the [project site](http://github.com/gnab/remark)

2. Follow the steps in the Getting Started section

For more information on using remark, please check out the [wiki](https://github.com/gnab/remark/wiki) pages.
]
---
name: last-page
template: inverse

## That's all folks (for now)!

Slideshow created using [remark](http://github.com/gnab/remark).