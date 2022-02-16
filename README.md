# Deploying ROSA with AWS Gateway Load Balancer

This article will outline a solution for deploying ROSA using a frictionless secure approach to gain access to the Internet using AWS Gateway Load Balancer without the complexities of proxies.

ROSA requires access to publically hosted Red Hat image repositories in order to install the core components and operators that form the basis of OpenShift. Thus solving the problem of how to efficiently and securely connect to the Internet in adherence to corporate policies typically embedded inside firewall rules is a key challenge to address at the start of the cloud adoption journey.

There are several well-known approaches such as using NAT Gateways and Transit Gateways with proxies that are documented in the ROSA installation guide as per the following link: https://docs.openshift.com/rosa/welcome/index.html

One method that is not so well-known, and which does not rely on proxies, is to leverage AWS Gateway Load Balancer as a "bump-in-the-wire" in conjunction with firewall appliances to perform packet inspection and payload manipulation in a non-intrusive manner before exiting traffic via the Interent Gateway towards the Red Hat image repositories. Eliminating the dependency on proxies reduces friction, and improves security by leveraging the multiple capabilities of various stateful firewall appliances available from the AWS Marketplace.

From an architectural pattern there are two options for integrating ROSA with AWS Gateway Load Balancer (GWLB) that this article will examine. The first pattern leverages a centralised control plane with a distributed data plane and the second leverages a centralised control and data plane.

The first architectural pattern is shown in the following diagram.

The control plane is composed of the GWLB and a scaleable fleet of inline firewall appliances that implement allowed egress paths. For ROSA these are defined in the following link: https://docs.openshift.com/rosa/rosa_getting_started/rosa-aws-prereqs.html#osd-aws-privatelink-firewall-prerequisites. The main defining feature of this architectural pattern is that all Internet-bound traffic is hairpinned through the control plane and requires an additional public subnet to be created in the VPC hosting the private ROSA cluster. The advantage of this design is that if static public IP addresses are configured for the Internet Gateway then any third-party receiver can define explicit firewall rules targeting traffic originating from the ROSA cluster to support data-sensitive exchanges



a price-point    A critical aspect for any ROSA deployment is connectivity to the Internet for the purpose of downloading image files for OpenShift core and operators required to install the cluster. Without a functioning egress path for routing traffic to public Red Hat image registries this quickly becomes a barrier to any installation, especially as ROSA does not support a disconnected installation method.

Thus the network architecture enabling Internet egress becomes a critical design point and should typically be defined as part of an overall Landing Zone Architecture or similar construct.

ROSA supports various Internet egress options including those based on well-known approaches such as NAT Gateways and Transit Gateways. The one that may not be so well-known is the still relatively new AWS Gateway Load Balancer which operates as a "bump-in-the-wire" in which Internet-bound traffic is transparently proxied across a fleet of firewall appliances for inspection prior to exiting the AWS cloud via an Internet Gateway.

Two example architectures are presented. The first is setup is based on a centralised control plane with a distributed data plane. In this setup egress traffic is "hairpinned" through the firewall appliances which at all times remain hidden from the world inside a private subnet. The downside is that each client VPC leveraging this centralised control plane will need to setup it's own public subnet and Internet Gateway which can become a limiting factor in terms of consumption of IPv4 address space.

The second setup based on a centralised control plane with a centralised data plane reduces the total required IPv4 addresses space but does require more sophisticated firewall devices that have a leg in both private and public subnets and can perform network address translation.

Whichever deployment option is chosen the key advantage that this architectural pattern brings to the table is that because GWLB functions as a transparent proxy no additional configuration is needed as is the case when deploying with expicit proxies. From a ROSA perspective everything looks like it is layer-3 based routing;  the only special configuration required is configure each subnet routing tables to point the default route (0.0.0.0/0) to a corresponding GWLB endpoint.


