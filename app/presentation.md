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
  I'm Doug and this is a stub. 

- Develop Asterisk solutions every day.

- A founding member of Burlington, Vermont's hackerspace [LaboratoryB.org](http://laboratoryb.org)

- Love full-stack Javascript

- Open soooource.

- Live in Vermont.

- Into trout fishing, backcountry skiing, ... 

]
---
layout: false
.left-column[
  ## 800response
]
.right-column[
  800response is a company, stub.

- Telephony, cool stuff, technology.

- We like customers.

- VoIP
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
  #You can find this online @ [url.stub.com](url.stub.com)

  - Whole presentation is in markdown.

  - Github

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
  Docker is cool, containers.

  - advantages

  - portability

  - koolaid

```bash
FROM fedora:latest
MAINTAINER Doug Smith <info@laboratoryb.org>
RUN yum install -y cowsay
```

So you could serve a file:

```bash
docker run -it dougbtv/cowsay /usr/bin/cowsay "Vermont Is Awesome"
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
---
layout: false
.left-column[
  ## Docker
  ## CoreOS
]
.right-column[
  CoreOS is awesome, run your containers.

  - Fork of ChromeOS

  - Just a couple hundred megs.

  - Your containers run your Linux flavor that you're used to.

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
  etcd is a discovery service.

  - A distributed key-value pair database with a REST API

  - Used by CoreOS itself for `fleet` a way to manage your "fleet" of CoreOS machines and the services that run on them.

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
- Like `systemctl` at a cluster level
- Controlled at the command line
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
  - We use a custom application `kamailio-etcd-dispatcher` which rebuilds nodes
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
.left-column[
  ## Docker
  ## CoreOS
  ## etcd
  ## Fleet
  ## Kamailio
  ## Homer
  ## Not covered
]
.right-column[
  Somethings are just always too painful for a presentation.

  - Dun dun dunnn! NAT.

  - Configuration Management, and you need it.

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
# System Architecture
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