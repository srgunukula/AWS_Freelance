
The Guide to Load Balancers
=====================================


| Specific AWS Services                 | Basics                         | Tips                          | Gotchas                                        |
|---------------------------------------|--------------------------------|-------------------------------|------------------------------------------------|
| [Load Balancers](#load-balancers) | [📗](#load-balancer-basics) | [📘](#load-balancer-tips) | [📙](#load-balancer-gotchas-and-limitations) |
| [CLB (ELB)](#clb) | [📗](#clb-basics) | [📘](#clb-tips) | [📙](#clb-gotchas-and-limitations) |
| [ALB](#alb) | [📗](#alb-basics) | [📘](#alb-tips) | [📙](#alb-gotchas-and-limitations) |


Introduction
------------------

A lot of information on AWS is already written. Most people learn AWS by reading a blog or a “[getting started guide](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)” and referring to the [standard AWS references](https://aws.amazon.com/documentation/). Nonetheless, trustworthy and practical information and recommendations aren’t easy to come by. AWS’s own documentation is a great but sprawling resource few have time to read fully, and it doesn’t include anything but official facts, so omits experiences of engineers. The information in blogs or [Stack Overflow](http://stackoverflow.com/questions/tagged/amazon-web-services) is also not consistently up to date.


Scope
-----

-	Currently, this guide covers Load Balancers. We expect it to expand.
-	It is not a tutorial, but rather a collection of information you can read and return to. It is for both beginners and the experienced.
-	The goal of this guide is to be:
	-	**Brief:** Keep it dense and use links
	-	**Practical:** Basic facts, concrete details, advice, gotchas, and other “folk knowledge”
	-	**Current:** We can keep updating it, and anyone can contribute improvements
	-	**Thoughtful:** The goal is to be helpful rather than present dry facts. Thoughtful opinion with rationale is welcome. Suggestions, notes, and opinions based on real experience can be extremely valuable.
-	This guide is not sponsored by AWS or AWS-affiliated vendors. It is written by and for engineers who use AWS.

Legend
------

-	📒 Marks standard/official AWS pages and docs
-	🔹 Important or often overlooked tip
-	❗ Gotcha or warning (where risks or time or resource costs are significant)
-	🔸 Limitation or quirk (where it’s not quite so bad)
-	📜 Undocumented feature (folklore)
-	🐥 Relatively new (and perhaps immature) services or features
-	⏱ Performance discussions
-	⛓ Lock-in: Products or decisions that are likely to tie you to AWS in a new or significant way — that is, later moving to a non-AWS alternative would be costly in terms of engineering effort
-	🚪 Alternative non-AWS options
-	💸 Cost issues, discussion, and gotchas
-	📗📘📙 Colors indicate basics, tips, and gotchas, respectively.
-	🚧 Areas where correction or improvement are needed (possibly with link to an issue — do help!)

General Information
-------------------

### When to Use AWS

	-	Generally, the more AWS services you use, the more lock-in you have to AWS — that is, the more engineering resources (time and money) it will take to change to other providers in the future.
	-	Basic services like virtual servers and standard databases are usually easy to migrate to other providers or on premises. Others like load balancers and IAM are specific to AWS but have close equivalents from other providers. The key thing to consider is whether engineers are architecting systems around specific AWS services that are not open source or relatively interchangeable. For example, Lambda, API Gateway, Kinesis, Redshift, and DynamoDB do not have substantially equivalent open source or commercial service equivalents, while EC2, RDS (MySQL or Postgres), EMR, and ElastiCache more or less do. (See more [below](#which-services-to-use), where these are noted with ⛓.)

### Which Services to Use

	-	[EC2](#ec2): Virtual servers and associated components, including:
		-	[AMIs](#amis): Machine Images
		-	[Load Balancers](#load-balancers): CLBs and ALBs
		-	[Autoscaling](#auto-scaling): Capacity scaling (adding and removing servers based on load)
		-	[EBS](#ebs): Network-attached disks
		-	[Elastic IPs](#elastic-ips): Assigned IP addresses

### Service Matrix

Many services within AWS can at least be compared with Google Cloud offerings or with internal Google services. And often times you could assemble the same thing yourself with open source software. This table is an effort at listing these rough correspondences. (Remember that this table is imperfect as in almost every case there are subtle differences of features!)

| Service                       | AWS                                                                          | Google Cloud                 | Google Internal | Microsoft Azure                    | Other providers                   | Open source “build your own”                               |
|-------------------------------|------------------------------------------------------------------------------|------------------------------|-----------------|------------------------------------|-----------------------------------|------------------------------------------------------------|
| Load balancer                 | CLB/ALB                                                                      | Load Balancing               | GFE             | Load Balancer, 




Load Balancers
--------------

Elastic Load Balancing supports three types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers. This guide discusses Application Load Balancers. For more information about Network Load Balancers, see the User Guide for Network Load Balancers. For more information about Classic Load Balancers, see the User Guide for Classic Load Balancers. 

### Components

A load balancer serves as the single point of contact for clients. The load balancer distributes incoming application traffic across multiple targets, such as EC2 instances, in multiple Availability Zones. This increases the availability of your application. You add one or more listeners to your load balancer.

A listener checks for connection requests from clients, using the protocol and port that you configure, and forwards requests to one or more target groups, based on the rules that you define. Each rule specifies a target group, condition, and priority. When the condition is met, the traffic is forwarded to the target group. You must define a default rule for each listener, and you can add rules that specify different target groups based on the content of the request (also known as content-based routing).

Each target group routes requests to one or more registered targets, such as EC2 instances, using the protocol and port number that you specify. You can register a target with multiple target groups. You can configure health checks on a per target group basis. Health checks are performed on all targets registered to a target group that is specified in a listener rule for your load balancer.

The following diagram illustrates the basic components. Notice that each listener contains a default rule, and one listener contains another rule that routes requests to a different target group. One target is registered with two target groups.




### Load Balancer Basics

-	AWS has 2 load balancing products - “Classic Load Balancers” (CLBs) and “Application Load Balancers” (ALBs).
-	Before the introduction of ALBs, “Classic Load Balancers” were known as “Elastic Load Balancers” (ELBs), so older documentation, tooling, and blog posts may still reference “ELBs”.
-	CLBs have been around since 2009 while ALBs are a recent (2016) addition to AWS.
-	CLBs support TCP and HTTP load balancing while ALBs support HTTP load balancing only.
-	Both can optionally handle termination for a single SSL certificate.
-	Both can optionally perform active health checks of instances and remove them from the destination pool if they become unhealthy.
-	CLBs don't support complex / rule-based routing, while ALBs support a (currently small) set of rule-based routing features.
-	CLBs can only forward traffic to a single globally configured port on destination instances, while ALBs can forward to ports that are configured on a per-instance basis, better supporting routing to services on shared clusters with dynamic port assignment (like ECS or Mesos).
-	CLBs are supported in EC2 Classic as well as in VPCs while ALBs are supported in VPCs only.

### Load Balancer Tips

-	If you don’t have opinions on your load balancing up front, and don’t have complex load balancing needs like application-specific routing of requests, it’s reasonable just to use an CLB or ALB for load balancing instead.
-	Even if you don’t want to think about load balancing at all, because your architecture is so simple (say, just one server), put a load balancer in front of it anyway. This gives you more flexibility when upgrading, since you won’t have to change any DNS settings that will be slow to propagate, and also it lets you do a few things like terminate SSL more easily.
-	**CLBs and ALBs have many IPs:** Internally, an AWS load balancer is simply a collection of individual software load balancers hosted within EC2, with DNS load balancing traffic among them. The pool can contain many IPs, at least one per availability zone, and depending on traffic levels. They also support SSL termination, which is very convenient.
-	**Scaling:** CLBs and ALBs can scale to very high throughput, but scaling up is not instantaneous. If you’re expecting to be hit with a lot of traffic suddenly, it can make sense to load test them so they scale up in advance. You can also [contact Amazon](http://aws.amazon.com/articles/1636185810492479) and have them “pre-warm” the load balancer.
-	**Client IPs:** In general, if servers want to know true client IP addresses, load balancers must forward this information somehow. CLBs add the standard [X-Forwarded-For](https://en.wikipedia.org/wiki/X-Forwarded-For) header. When using an CLB as an HTTP load balancer, it’s possible to get the client’s IP address from this.
-	**Using load balancers when deploying:** One common pattern is to swap instances in the load balancer after spinning up a new stack with your latest version, keep old stack running for one or two hours, and either flip back to old stack in case of problems or tear it down.

### Load Balancer Gotchas and Limitations

-	❗CLBs and ALBs have **no fixed external IP** that all clients see. For most consumer apps this doesn’t matter, but enterprise customers of yours may want this. IPs will be different for each user, and will vary unpredictably for a single client over time (within the standard [EC2 IP ranges](http://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html)). And similarly, never resolve an CLB name to an IP and put it as the value of an A record — it will work for a while, then break!
-	❗Some web clients or reverse proxies cache DNS lookups for a long time, which is problematic for CLBs and ALBs, since they change their IPs. This means after a few minutes, hours, or days, your client will stop working, unless you disable DNS caching. Watch out for [Java’s settings](http://docs.oracle.com/javase/8/docs/api/java/net/InetAddress.html) and be sure to [adjust them properly](http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/java-dg-jvm-ttl.html). Another example is nginx as a reverse proxy, which [resolves backends only at start-up](https://www.jethrocarr.com/2013/11/02/nginx-reverse-proxies-and-dns-resolution/).
-	❗It’s not unheard of for IPs to be recycled between customers without a long cool-off period. So as a client, if you cache an IP and are not using SSL (to verify the server), you might get not just errors, but responses from completely different services or companies!
-	🔸As an operator of a service behind an CLB or ALB, the latter phenomenon means you can also see puzzling or erroneous requests by clients of other companies. This is most common with clients using back-end APIs (since web browsers typically cache for a limited period).
-	❗CLBs and ALBs take time to scale up, it does not handle sudden spikes in traffic well. Therefore, if you anticipate a spike, you need to “pre-warm” the load balancer by gradually sending an increasing amount of traffic.
-	❗Tune your healthchecks carefully — if you are too aggressive about deciding when to remove an instance and conservative about adding it back into the pool, the service that your load balancer is fronting may become inaccessible for seconds or minutes at a time. Be extra careful about this when an autoscaler is configured to terminate instances that are marked as being unhealthy by a managed load balancer.
-	❗CLB HTTPS listeners don't support Server Name Indication (SNI). If you need SNI, you can work around this limitation by either providing a certificate with Subject Alternative Names (SANs) or by using TCP listeners and terminating SSL at your backend.




CLB
---

### CLB Basics

-	📒 [Homepage](https://aws.amazon.com/elasticloadbalancing/classicloadbalancer/) ∙ [User guide](https://aws.amazon.com/elasticloadbalancing/classicloadbalancer/developer-resources/) ∙ [FAQ](https://aws.amazon.com/elasticloadbalancing/classicloadbalancer/faqs/) ∙ [Pricing](https://aws.amazon.com/elasticloadbalancing/classicloadbalancer/pricing/)
- Classic Load Balancers, formerly known as Elastic Load Balancers, are HTTP and TCP load balancers that are managed and scaled for you by Amazon.

### CLB Tips

-	**Best practices:** [This article](http://aws.amazon.com/articles/1636185810492479) is a must-read if you use CLBs heavily, and has a lot more detail.

### CLB Gotchas and Limitations

-	In general, CLBs are not as “smart” as some load balancers, and don’t have fancy features or fine-grained control a traditional hardware load balancer would offer. For most common cases involving sessionless apps or cookie-based sessions over HTTP, or SSL termination, they work well.
-       🔸By default, CLBs will refuse to route traffic from a load balancer in one Availability Zone (AZ) to a backend instance in another. This [will cause 503s](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/ts-elb-error-message.html#ts-elb-errorcodes-http503) if the last instance in an AZ becomes unavailable, even if there are healthy instances in other zones. If you’re running fewer than two backend instances per AZ, you almost certainly want to [enable cross-zone load balancing](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/enable-disable-crosszone-lb.html#enable-cross-zone).
-	Complex rules for directing traffic are not supported. For example, you can’t direct traffic based on a regular expression in the URL, like [HAProxy](http://www.haproxy.org/) offers.
-	**Apex DNS names:** Once upon a time, you couldn’t assign an CLB to an apex DNS record (i.e. example.com instead of foo.example.com) because it needed to be an A record instead of a CNAME. This is now possible with a Route 53 alias record directly pointing to the load balancer.
-	🔸CLBs use [HTTP keep-alives](https://en.wikipedia.org/wiki/HTTP_persistent_connection) on the internal side. This can cause an unexpected side effect: Requests from different clients, each in their own TCP connection on the external side, can end up on the same TCP connection on the internal side. Never assume that multiple requests on the same TCP connection are from the same client!
-	🔸 Traffic between CLBs and back-end instances in the same subnet **will** have [Network ACL](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) rules evaluated (EC2 to EC2 traffic in the same subnet would not have Network ACL rules evaluated). If the default '0.0.0.0/0 ALLOW' rule is removed from the Network ACL applied to the subnet, a rule that allows traffic on both the health check port and any listener port must be added.

ALB
---


### ALB Basics

-	📒 [Homepage](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/) ∙ [User guide](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/developer-resources/) ∙ [FAQ](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/faqs/) ∙ [Pricing](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/pricing/)
-	🐥**Websockets and HTTP/2** are [now supported](https://aws.amazon.com/blogs/aws/new-aws-application-load-balancer/).
-	Prior to the Application Load Balancer, you were advised to use TCP instead of HTTP as the protocol to make it work (as described [here](http://www.quora.com/When-will-Amazon-ELB-offer-SPDY-support)) and use [the obscure but useful Proxy Protocol](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-proxy-protocol.html) ([more on this](https://chrislea.com/2014/03/20/using-proxy-protocol-nginx/)) to pass client IPs over a TCP load balancer.

### ALB Tips

-	Use ALBs to route to services that are hosted on shared clusters with dynamic port assignment (like ECS or Mesos).
-	ALBs support HTTP path-based routing (send HTTP requests for “/api/*” -> {target-group-1}, “/blog/*” -> {target group 2}).

### ALB Gotchas and Limitations

-	ALBs support HTTP routing but not port-based TCP routing.
-	ALBs do not (yet) support routing based on HTTP “Host” header or HTTP verb.
-	Instances in the ALB's target groups have to either have a single, fixed healthcheck port (“EC2 instance”-level healthcheck) or the healthcheck port for a target has to be the same as its application port (“Application instance”-level healthcheck) - you can't configure a per-target healthcheck port that is different than the application port.
-	ALBs are VPC-only (they are not available in EC2 Classic)
-	In a target group, if there is no healthy target, all requests are routed to all targets. For example, if you point a listener at a target group containing a single service that has a long initialization phase (during which the health checks would fail), requests will reach the service while it is still starting up.

	-	For single-instance deployments, you could just assign elastic IP to an instance, give that IP a DNS name, and consider that your deployment. Most of the time, you should provision a [load balancer](#load-balancers) instead:
	-	It’s easy to add and remove instances from load balancers. It's also quicker to add or remove instances from a load balancer than to reassign an elastic IP.
	-	It’s more convenient to point DNS records to load balancers, instead of pointing them to specific IPs you manage manually. They can also be Route 53 aliases, which are easier to change and manage.

	-	Be sure to [use and understand CLBs/ALBs](#load-balancers) appropriately. Many outages are due to not using load balancers, or misunderstanding or misconfiguring them.


	-	🔸Periodically you may find that your server or load balancer is receiving traffic for (presumably) a previous EC2 server that was running at the same IP address that you are handed out now (this may not matter, or it can be fixed by migrating to another new instance).

