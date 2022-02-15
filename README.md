# ROSA with AWS Gateway Load Balancer

A critical aspect for any ROSA deployment is connectivity to the Internet for the purpose of downloading image files for OpenShift core and operators required to install the cluster. Without a functioning egress path for routing traffic to public Red Hat image registries this quickly becomes a barrier to any installation, especially as ROSA does not support a disconnected installation method.

Thus the network architecture enabling Internet egress becomes a critical design point and should typically be defined as part of an overall Landing Zone Architecture or similar construct.

ROSA supports various Internet egress options including those based on well-known approaches such as NAT Gateways and Transit Gateways. The one that may not be so well-known is the still relatively new AWS Gateway Load Balancer which operates as a "bump-in-the-wire" in which Internet-bound traffic is transparently proxied across a fleet of firewall appliances for inspection prior to exiting the AWS cloud via an Internet Gateway.

Two example architectures are presented. The first is setup is based on a centralised control plane with a distributed data plane. In this setup egress traffic is "hairpinned" through the firewall appliances which at all times remain hidden from the world inside a private subnet. The downside is that each client VPC leveraging this centralised control plane will need to setup it's own public subnet and Internet Gateway which can become a limiting factor in terms of consumption of IPv4 address space.

The second setup based on a centralised control plane with a centralised data plane reduces the total required IPv4 addresses space but does require more sophisticated firewall devices that have a leg in both private and public subnets and can perform network address translation.


