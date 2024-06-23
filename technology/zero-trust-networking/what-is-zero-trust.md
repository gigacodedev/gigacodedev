# Introduction to Zero Trust

Threat actors are getting smarter by the day, and the assumptions around protecting your network and assets are constantly evolving. Zero Trust changes the thinking from premise based "Protect the castle perimeter" thinking to a principal of "Never trust, always verify"

## Why is this important?

With more and more systems becoming separated from a single site and moving to cloud hosting, the "network perimiter" is no longer a viable entity to protect. Additionally, threat actors are targeting the usage of misconfigured corportate networks to move laterally within networks.

Key principals of Zero Trust include:

1. Verify Explicitly: Always authenticate based on *all* available datapoints, including identity, asset, network, and other signals
2. Enforce Least Privilege: Chances are, your call center agent doesn't need direct access to the production software environment. Limit your user and application access to only what is nessesary based on who is accessing it, but also on factors like the device security they are connecting in from
3. Assume Breach: Operate your environments with the assumtion the threat actors are already there. Segment access, apply monitoring, and respond quickly to alerts

## How is Zero Trust different from other networking theories?

### Traditional remote site networking (Site to Site, Point to Site)

- One network is created for resource access, segmented based on the incoming connection parameters
- VLANs can be used to segment networks and stateful firewalls used to control which applications/ports can be used to cross between VLANs
- Typically, only a valid identity is required to access the network
- Prone to misconfigurations allowing for a wider scope of lateral movement between networks with high confidentiality and integrity requirements

In my experience working in the IT and security consulting world, many networks still operate under these practices. Does this mean they are inherently unsafe? Not necessarily. However, they may not be as robust as they can be, and can offer much higher attack surfaces for movement between networks. It's not uncommon for an initial attack vector to be a VPN client with stolen credentials, whether from an internal employee or contractor, with way too much access scopes. When everyone has access to a whole network subnet, it makes it much easier to expand the scope of what you can see

### Zero Trust Networking

- Eliminates the concept of a trusted internal network and untrusted external network
- Continuously verifies each access attempt, whether from inside or outside the network
- Evaluates each resource to access individually, rather than allowing full access to a subnet of resources
- Prioritizes secure access to resources based on modern authentication protocols

By contrast, Zero Trust networking essentially makes a VLAN for each individual resource. It's a bit more than that, but assume that each resource is on its own network essentially for access when configured properly. If someone needs to get into the network to access a file share resource, this does not also automatically give them access to an application server, or to an intranet web resource. Each must have their access evaluated individually, and you may be authenticated to connect to all but not allowed to access some based on other signals

## Scalability Benefits

Modern Zero Trust networking platforms allow you to quickly and security create individual tunnels to resources, create many access rules, and modify/revoke access if any changes need to be made. Where on a traditional network you may need to add a resource to an overscoped subnet or spin up multiple stateful rules, VLANs, and NAT policies, Zero Trust allows you a high level of adaptability and access to resources no matter where they are on the web.

## Chosing your Zero Trust client

There's many platforms to choose from, with some big players like Cisco, Microsoft, Cloudflare, and Zscaler. We decided to go with Cloudflare, for the following reasons:

- Maturity
  - When evaluating, at the time it was a much more mature platform that over offerings providing aribtrary TCP tunneling, browser based isolation and rendering, and the ability to insert short life SSH certificates for SSH connections
- Affordability
  - Working in the SMB (Small/Medium Business) space, afforability is an important consideration. Cloudflare's offerings are relatively easy to include in our existing support plans, including a free plan
  - While a free plan is available, it does have limitations for log retention, network locations, and user counts. Additionally, it does not have an uptime SLA and only community forum support
  - We typically deploy the standard "Pay-As-You-Go" plan, which at the time of writing, is $7/user/mo. This includes 100% uptime SLA guarantees, support with a median response of 4 hours, up to 50 network locations, and 30 days of dashboard/log retention
- Reliability
  - Cloudflare serves 20% of all internet traffic globally, and has a global edge network that is <50ms from 95% of all internet users. While outages happen, they have a track record of reliable services, and include a 100% uptime SLA in their paid plans
- Commonality
  - Already using Cloudflare for our DNS and WAF management, adding their ZTNA/SASE platforms were more appealing than adding yet another vendor to our toolbelt

Does this mean other platforms are bad? No! This platform just works the best for us, but Microsoft Global Secure Access has come a long way since we evaluated it, and other enterprise platforms are spoken of highly by our peers. While this section will be using the setup of Cloudflare ZTNA as our example, the concepts should carry over into most other plaforms. I highly recommend you evaluate all options available to you to find the best solution for your organization, not just ours.
