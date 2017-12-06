
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






EC2
---

### EC2 Basics

-	📒 [Homepage](https://aws.amazon.com/ec2/) ∙ [Documentation](https://aws.amazon.com/documentation/ec2/) ∙ [FAQ](https://aws.amazon.com/ec2/faqs/) ∙ [Pricing](https://aws.amazon.com/ec2/pricing/) (see also [ec2instances.info](http://www.ec2instances.info/)\)
-	**EC2** (Elastic Compute Cloud) is AWS’ offering of the most fundamental piece of cloud computing: A [virtual private server](https://en.wikipedia.org/wiki/Virtual_private_server). These “instances” can run [most Linux, BSD, and Windows operating systems](https://aws.amazon.com/ec2/faqs/#What_operating_system_environments_are_supported). Internally, they use [Xen](https://en.wikipedia.org/wiki/Xen) virtualization.
-	The term “EC2” is sometimes used to refer to the servers themselves, but technically refers more broadly to a whole collection of supporting services, too, like load balancing (CLBs/ALBs), IP addresses (EIPs), bootable images (AMIs), security groups, and network drives (EBS) (which we discuss individually in this guide).
-	💸**[EC2 pricing](https://aws.amazon.com/ec2/pricing/)** and **[cost management](#ec2-cost-management)** is a complicated topic. It can range from free (on the [AWS free tier](https://aws.amazon.com/free/)) to a lot, depending on your usage. Pricing is by instance type, by hour and changes depending on AWS region and whether you are purchasing your instances [On-Demand](https://aws.amazon.com/ec2/pricing/on-demand/), on the [Spot market](https://aws.amazon.com/ec2/spot/) or pre-purchasing ([Reserved Instances](https://aws.amazon.com/ec2/pricing/reserved-instances/)).


### EC2 Gotchas and Limitations

-	❗Never use ssh passwords. Just don’t do it; they are too insecure, and consequences of compromise too severe. Use keys instead. [Read up on this](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) and fully disable ssh password access to your ssh server by making sure 'PasswordAuthentication no' is in your /etc/ssh/sshd_config file. If you’re careful about managing ssh private keys everywhere they are stored, it is a major improvement on security over password-based authentication.
-	🔸For all [newer instance types](https://aws.amazon.com/amazon-linux-ami/instance-type-matrix/), when selecting the AMI to use, be sure you select the HVM AMI, or it just won’t work.
-	❗When creating an instance and using a new ssh key pair, [make sure the ssh key permissions are correct](http://stackoverflow.com/questions/1454629/aws-ssh-access-permission-denied-publickey-issue).
-	🔸Sometimes certain EC2 instances can get scheduled for retirement by AWS due to “detected degradation of the underlying hardware,” in which case you are given a couple of weeks to migrate to a new instance
 	-	If your instance root device is an EBS volume, you can typically stop and then start the instance which moves it to healthy host hardware, giving you control over timing of this event. Note however that you will lose any instance store volume data ([ephemeral drives](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)) if your instance type has instance store volumes.
 	-   The instance public IP (if it has one) will likely change unless you're using Elastic IPs. This could be a problem if other systems depend on the IP address.
-	🔸Periodically you may find that your server or load balancer is receiving traffic for (presumably) a previous EC2 server that was running at the same IP address that you are handed out now (this may not matter, or it can be fixed by migrating to another new instance).


Load Balancers
--------------

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

Elastic IPs
-----------

### Elastic IP Basics

-	📒 [Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) ∙ [FAQ](https://aws.amazon.com/ec2/faqs/#Elastic_IP) ∙ [Pricing](https://aws.amazon.com/ec2/pricing/#Elastic_IP_Addresses)
-	**Elastic IPs** are static IP addresses you can rent from AWS to assign to EC2 instances.

### Elastic IP Tips

-	🔹**Prefer load balancers to elastic IPs:** For single-instance deployments, you could just assign elastic IP to an instance, give that IP a DNS name, and consider that your deployment. Most of the time, you should provision a [load balancer](#load-balancers) instead:
	-	It’s easy to add and remove instances from load balancers. It's also quicker to add or remove instances from a load balancer than to reassign an elastic IP.
	-	It’s more convenient to point DNS records to load balancers, instead of pointing them to specific IPs you manage manually. They can also be Route 53 aliases, which are easier to change and manage.
	-	But in some situations, you do need to manage and fix IP addresses of EC2 instances, for example if a customer needs a fixed IP. These situations require elastic IPs.
-	Elastic IPs are limited to 5 per account. It’s possible to [request more](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-elastic-ips-ec2-classic).
-	If an Elastic IP is not attached to an active resource there is a small [hourly fee](https://aws.amazon.com/ec2/pricing/#Elastic_IP_Addresses).
-	Elastic IPs are [no extra charge](https://aws.amazon.com/ec2/pricing/#Elastic_IP_Addresses) as long as you’re using them. They have a (small) cost when not in use, which is a mechanism to prevent people from squatting on excessive numbers of IP addresses.

### Elastic IP Gotchas and Limitations

-	🔸There is [officially no way](https://forums.aws.amazon.com/thread.jspa?threadID=171550) to allocate a contiguous block of IP addresses, something you may desire when giving IPs to external users. Though when allocating at once, you may get lucky and have some be part of the same CIDR block.



Route 53
--------

### Route 53 Basics

-	📒 [Homepage](https://aws.amazon.com/route53/) ∙ [Developer guide](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/) ∙ [FAQ](https://aws.amazon.com/route53/faqs/) ∙ [Pricing](https://aws.amazon.com/route53/pricing/)
-	**Route 53** is AWS’ DNS service.

### Route 53 Alternatives and Lock-In

-	Historically, AWS was slow to penetrate the DNS market (as it is often driven by perceived reliability and long-term vendor relationships) but Route 53 has matured and [is becoming the standard option](https://www.datanyze.com/market-share/dns/) for many companies. Route 53 is cheap by historic DNS standards, as it has a fairly large global network with geographic DNS and other formerly “premium” features. It’s convenient if you are already using AWS.
-	⛓Generally you don’t get locked into a DNS provider for simple use cases, but increasingly become tied in once you use specific features like geographic routing or Route 53’s alias records.
-	🚪Many alternative DNS providers exist, ranging from long-standing premium brands like [UltraDNS](https://www.neustar.biz/services/dns-services) and [Dyn](http://dyn.com/managed-dns/) to less well known, more modestly priced brands like [DNSMadeEasy](http://www.dnsmadeeasy.com/). Most DNS experts will tell you that the market is opaque enough that reliability and performance don’t really correlate well with price.
-	⏱Route 53 is usually somewhere in the middle of the pack on performance tests, e.g. the [SolveDNS reports](http://www.solvedns.com/dns-comparison/).


### VPC and Network Security Tips

-	❗**Security groups** are your first line of defense for your servers. Be extremely restrictive of what ports are open to all incoming connections. In general, if you use CLBs, ALBs or other load balancing, the only ports that need to be open to incoming traffic would be port 22 and whatever port your application uses.
-	**Port hygiene:** A good habit is to pick unique ports within an unusual range for each different kind of production service. For example, your web frontend might use 3010, your backend services 3020 and 3021, and your Postgres instances the usual 5432. Then make sure you have fine-grained security groups for each set of servers. This makes you disciplined about listing out your services, but also is more error-proof. For example, should you accidentally have an extra Apache server running on the default port 80 on a backend server, it will not be exposed.
-	**Migrating from Classic**: For migrating from older EC2-Classic deployments to modern EC2-VPC setup, [this article](http://blog.kiip.me/engineering/ec2-to-vpc-executing-a-zero-downtime-migration/) may be of help.
-	For basic AWS use, one default VPC may be sufficient. But as you scale up, you should consider mapping out network topology more thoroughly. A good overview of best practices is [here](http://blog.flux7.com/blogs/aws/vpc-best-configuration-practices).
-	Consider controlling access to your private AWS resources through a [VPN](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpn-connections.html).
	-	You get better visibility into and control of connection and connection attempts.
	-	You expose a smaller surface area for attack compared to exposing separate (potentially authenticated) services over the public internet.
		-	e.g. A bug in the YAML parser used by the Ruby on Rails admin site is much less serious when the admin site is only visible to the private network and accessed through VPN.
	-	Another common pattern (especially as deployments get larger, security or regulatory requirements get more stringent, or team sizes increase) is to provide a [bastion host](https://www.pandastrike.com/posts/20141113-bastion-hosts) behind a VPN through which all SSH connections need to transit.
-	🔹Consider using other security groups as sources for security group rules instead of using CIDRs — that way, all hosts in the source security group and only hosts in that security group are allowed access. This is a much more dynamic and secure way of managing security group rules.
-   **VPC Flow Logs** allow you to monitor the network traffic to, from, and within your VPC. Logs are stored in CloudWatch Logs groups, and can be used for security monitoring (with third party tools), performance evaluation, and forensic investigation.
	-	See the [VPC Flow Logs User Guide](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/flow-logs.html) for basic information.
	-	See the [flowlogs-reader](https://github.com/obsrvbl/flowlogs-reader) CLI tool and Python library to retrieve and work with VPC Flow Logs.



High Availability
-----------------

This section covers tips and information on achieving [high availability](https://en.wikipedia.org/wiki/High_availability).

### High Availability Tips

-	AWS offers two levels of redundancy, [regions and availability zones (AZs)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-regions-availability-zones).
-	When used correctly, regions and zones do allow for high availability. You may want to use non-AWS providers for larger business risk mitigation (i.e. not tying your company to one vendor), but reliability of AWS across regions is very high.
-	**Multiple regions:** Using multiple regions is complex, since it’s essentially like managing completely separate infrastructures. It is necessary for business-critical services with the highest levels of redundancy. However, for many applications (like your average consumer startup), deploying extensive redundancy across regions may be overkill.
-	The [High Scalability Blog](http://highscalability.com/blog/2016/1/11/a-beginners-guide-to-scaling-to-11-million-users-on-amazons.html) has a good guide to help you understand when you need to scale an application to multiple regions.
-	🔹**Multiple AZs:** Using AZs wisely is the primary tool for high availability!
	-	A typical single-region high availability architecture would be to deploy in two or more availability zones, with load balancing in front, as in [this AWS diagram](http://media.amazonwebservices.com/architecturecenter/AWS_ac_ra_ftha_04.pdf).
	-	The bulk of outages in AWS services affect one zone only. There have been rare outages affecting multiple zones simultaneously (for example, the [great EBS failure of 2011](http://aws.amazon.com/message/65648/)) but in general most customers’ outages are due to using only a single AZ for some infrastructure.
	-	Consequently, design your architecture to minimize the impact of AZ outages, especially single-zone outages.
	-	Deploy key infrastructure across at least two or three AZs. Replicating a single resource across more than three zones often won’t make sense if you have other backup mechanisms in place, like S3 snapshots.
	-	A second or third AZ should significantly improve availability, but additional reliability of 4 or more AZs may not justify the costs or complexity (unless you have other reasons like capacity or Spot market prices).
	-	💸Watch out for **cross-AZ traffic costs**. This can be an unpleasant surprise in architectures with large volume of traffic crossing AZ boundaries.
	-	Deploy instances evenly across all available AZs, so that only a minimal fraction of your capacity is lost in case of an AZ outage.
	-	If your architecture has single points of failure, put all of them into a single AZ. This may seem counter-intuitive, but it minimizes the likelihood of any one SPOF to go down on an outage of a single AZ.
-	**EBS vs instance storage:** For a number of years, EBSs had a poorer track record for availability than instance storage. For systems where individual instances can be killed and restarted easily, instance storage with sufficient redundancy could give higher availability overall. EBS has improved, and modern instance types (since 2015) are now EBS-only, so this approach, while helpful at one time, may be increasingly archaic.
-	Be sure to [use and understand CLBs/ALBs](#load-balancers) appropriately. Many outages are due to not using load balancers, or misunderstanding or misconfiguring them.


