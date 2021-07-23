# browser-request-breakdown
**What**:
A document trying to go over many touch points in the context of "what happens when a browsers makes a request to get a webpage"

**who**:
This document is bias towards back end web developer concepts and will benefit developers who are in that role but anyone interested in or working with web development or internet requests could benefit from

**Caveats**:
This is not a complete exhaustive list, as mentioned in the who, this has an audience so we will not be diving into things like [how computers and graphic cards render things to a monitor](https://computer.howstuffworks.com/graphics-card.htm)

There are also a number of assumptions made throughout the document, I have tried to note them where they are

I have included links where I have found deeper information but they should be starting off points for deeper dives and not considered exhaustive

---




# Preamble:
**we assume**:
1. there is a functional computer with internet connection  (note the internet connection is a deep subject with countless rabbit holes and will require its own document to say the least)
2. that computer has a browser open and a url present in the address bar - we will use  https://gitlab.com/gitlab-org/gitlab as an example
3. the breakdown starts when the user hits the enter/return key to execute the request





# Pre-Network:
1. The Keyboard will send a keycode (13 for enter in a standard US keyboard) to the computer; 
  you can explore keycodes [here](https://www.cambiaresearch.com/articles/15/javascript-char-codes-key-codes)
2. The OS goes through an [interrupt](http://faculty.salina.k-state.edu/tim/ossg/Introduction/OSworking.html)
3. The OS will pass the keypress to the application in focus; in this case the browser which will then begin the process of loading the web page

---



# Client Side

## OSI Model:
The [OSI model](https://en.wikipedia.org/wiki/OSI_model) is a model that breaks down network traffic into 7 layers
for the purposes of this conversation we will not go down into the media layers(layer 1-3) but those layers are interesting topics if you are creating or primarily working with the physical network



## DNS:
At this point the browser needs to know what server to make a request to
If we look at our example url: https://gitlab.com/gitlab-org/gitlab
DNS will resolve gitlab.com to an IP address.
1. First it will look up in a local browser DNS cache to see if it already has the value
2. assuming the value is not in local cache it will perform a DNS lookup which will go through tiers of attempts to find the value
 - potential layers:
   - Operating system (eg if content exists in [/etc/hosts](https://docs.rackspace.com/support/how-to/modify-your-hosts-file/)
   - router or internal network DNS resolver
   - ISP cache
   - [Root Name Servers](https://en.wikipedia.org/wiki/Root_name_server)
3. at the end of this process the browser will have an IP address to move forward with, it is most likely that address will be an IPv4 address but might be IPv6 for more information see [here](https://www.thousandeyes.com/learning/techtorials/ipv4-vs-ipv6)



## Server Connection:
with the IP address known the browser can now start to make a connection to the server.
There are 2 primary protocols that that communication may take, [TCP or UDP](https://www.lifesize.com/en/blog/tcp-vs-udp/ ) both of which use [IP](https://en.wikipedia.org/wiki/Internet_Protocol) as the supporting protocol for the network to sum up:
 - TCP focuses on ensuring the data is correct
 - UDP focuses on speed sacrificing correctness
Most web browsing will be using TCP/IP and the initial request to the server will almost certainly be TCP/IP

I will not go into detail on the TCP connection setup, if you wish to read more about it see [here](https://www.oreilly.com/library/view/http-the-definitive/1565925092/ch04s01.html)
the important part here is that the TCP connection to the server sets up a method of communication between the browser and server to facilitate the next steps



## Transport Protocol:
if we look at our example url: https://gitlab.com/gitlab-org/gitlab
we can see we are using [https](https://www.cloudflare.com/en-ca/learning/ssl/what-is-https/) which is an encrypted version of [http](https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/). HTTPS usually uses a protocol called [Transport Layer Security(TLS)](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) to encrypt the data being used in the exchange between the browser and the server



## REST:
or [Representational state transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)

Most modern web frameworks use [restful](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) requests, the first request in the example we are using will either be a HEAD request or a GET request. HEAD is very rarely used but it is worth noting that it exists. Some Other possible request verbs are:
- GET
- POST
- PATCH
- PUT
- DELETE


So if we break down the example URL we have at this point with the initial request it is:

https://gitlab.com/gitlab-org/gitlab
GET the resource `gitlab-org/gitlab` from the server `gitlab.com` using the `https` protocol

It is also worth mentioning [request headers](https://developer.mozilla.org/en-US/docs/Glossary/Request_header) at this point, which will contain information about the requesting system and some instruction to the server about what the system would like back like which language the content should be in

---




# Server Side:
At this point it's worth jumping over the server side of this process where we will touch on some infrastructure and some application process before getting back to the browser and how it interacts with the response



## Server Resolution:
In a very small production environment there might only be 1 web server and the DNS resolution might resolve to that specific server
It is much more likely that there are many webservers to handle hire capacity and either a [load balancer](https://www.nginx.com/resources/glossary/load-balancing/) or [reverse proxy](https://www.cloudflare.com/en-ca/learning/cdn/glossary/reverse-proxy) is acting as a publicly visible DNS resolution that passes that request to 1 of many running web servers. here or in sessions section is a good place to talk about [sticky sessions](https://www.imperva.com/learn/availability/sticky-session-persistence-and-cookies/)


## Network Security:
although not strictly necessary to talk about for the resolution of such a request here is a good point  in the conversation to bring [network security](https://blog.netwrix.com/2019/01/22/network-security-devices-you-need-to-know-about/). From the low level [firewall](https://en.wikipedia.org/wiki/Firewall_(computing)) to the higher level [WAF](https://en.wikipedia.org/wiki/Web_application_firewall) and concepts like IP [whitelisting](https://en.wikipedia.org/wiki/Whitelisting)


## Web server to application:
There are a number of possible web servers and applications that could be used for this process.  For our example we'll be using [nginx](https://www.nginx.com/) and [ruby on rails](https://rubyonrails.org/) for this example

In short nginx handles the up to OSI layer 6 and layer 7 is passed onto the Rails application to handle which is a good separation of responsibility allowing each layer to specialize and create an abstraction allowing for interchangeability or potential scaling



## Application:
As stated we will be using Ruby on Rails for this example which means this section will be very application specific and may be very different in other applications

### Middleware:
Rails can have multiple pieces of middleware present that might interact with the request before the main classic parts of the application get to it, for instance it almost certainly will be using [rack](https://github.com/rack/rack) it might also be using something to send requests to logging or an [Application performance monitoring(APM)](https://www.appdynamics.com/blog/product/newbie-guide-apm/) system

### Authentication:
Authentication will likely be taken care of via middleware but doesn’t have to be and might be custom to the application, there are a number of technologies worth potentially talking about here

### Sessions:
Rails usually uses cookie based sessions which will be included in each request to the server as part of the headers of the request, there are pros and cons to this approach
See https://prosancons.com/computer/pros-and-cons-of-cookies/ for some examples

#### JWT tokens:
Are a technology used to pass proof of login in many modern web transactions see https://en.wikipedia.org/wiki/JSON_Web_Token for more information

#### OAuth:
A standard / protocol to allow authentication, often used to authenticate between systems
See https://en.wikipedia.org/wiki/OAuth for more information

#### Identity Providers:
Using an external system like google or facebook for authentication instead of an internal system see https://en.wikipedia.org/wiki/Identity_provider for more information

### Routes:
Once the request has been processed by all middleware the rails application will use the route of the request for a lookup in the routes definitions, so in our example case
https://gitlab.com/gitlab-org/gitlab we will look up gitlab-org/gitlab in our [routes file](https://guides.rubyonrails.org/routing.html) to see which controller and method to call to process this request

### MVC:
This is a good point to mention that rails is based on an MVC framework if your not familiar with it its worth reading https://www.codecademy.com/articles/mvc

#### Controller:
The controller will process the request, this will be very different depending on what the request is, in our example https://gitlab.com/gitlab-org/gitlab we are likely hitting the show method in a controller used to interact with repositories;
the controller will orchestrate what needs to be done to complete the request, this will likely involve extantiating *models* and calling *services*, and finally returning content 

#### Models:
Rails models are usually an [object relational mapping(ORM)](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) and by default will be [active record](https://guides.rubyonrails.org/active_record_basics.html)

For our example will likely make a request to a database for the ‘gitlab’ repository in order to get the information we need from it to generate our response

#### Databases:
There are a number of possible Databases the system could be connecting to each with pros and cons
Some examples:
- [SQL compliant](https://www.w3schools.com/sql/sql_intro.asp): (This is a good point to talk about [DB indexes](https://en.wikipedia.org/wiki/Database_index))
   - Mysql
   - Postgres

- [NoSql](https://en.wikipedia.org/wiki/NoSQL):
   - mongoDB
   - Graph Database

- [Object store](https://en.wikipedia.org/wiki/Object_storage)(techincally not a DB but good place to talk about it)
   - S3
   - Google storage

#### Services:
Services are a way to push logic and process outside of the controller for better code organization, reuse and testability
There might be services called to process this request, for instance we might want to trigger a repository viewed log 
https://en.wikipedia.org/wiki/Service-oriented_architecture is a good place to start to learn more about why you might want to push things into services

#### Views:
Once the controller has gathered all the data needed to process the request that data will be passed to a view to create the actual content that will be sent back to the requesting browser
This separations allows for better code reuse if there are multiple ways to request the same data as well as a better separation of responsibilities and testability

In a classic monolithic rails application the view will likely compose a quantity of HTML/CSS/JS together, likely by composing together many partials (see https://guides.rubyonrails.org/layouts_and_rendering.html for more information) 

However it is also possible to use rails to power modern single page apps like react by providing JSON views which will only return raw JSON, no HTML/CSS/JS 

##### Content types:
It is worth noting that rails will use the content type header submitted by the request in order to help figure out which view it should use to compose the response



### Caching:
Once the view is complete and returned there will be a number of caches that will keep information to speed up the response of that request should it be made again some examples:
DB query level cache
Rails DB query level cache
Rails page caching
See https://guides.rubyonrails.org/caching_with_rails.html for more information on rails specific cache layers

This is also a good place to talk about [CDNs](https://en.wikipedia.org/wiki/Content_delivery_network) where static content can be stored in geodistributed caches to reduce latency

---




# Back to the Browser:
Once the response it sent back to the browser a number of things happen in the browser to render the response


## Page Rendering:
The actual page rendering has many steps
The first is [DOM processing](https://www.programmersought.com/article/34414461236/) where the HTML, CSS and JS are rendered

This is also where requests to additional CSS files, JS files and media will be started

The result is then rendered on screen


Once the page is fully rendered there might still be dynamic content that needs to be loaded or other actions taken, for example:
- JS might make additional requests to get more data or media to load onto the page
    - [Ajax](https://www.w3schools.com/js/js_ajax_http_send.asp) is the right place to start to get more information on that process.
    - It is also possible to setup a pipe to allow a server to push data to the active webpage via [websockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Cookies](https://en.wikipedia.org/wiki/HTTP_cookie) might get set recording preferences or other information required for the web app
- If we are using a single page JS framework like [react](https://reactjs.org/) the framework start to intercept some aspects of classic navigation
- there might be a number of [tracking](https://en.ryte.com/wiki/Tracking_Pixel) or [Web Analytics](https://en.wikipedia.org/wiki/Web_analytics) systems that would be worth mentioning here














