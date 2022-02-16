# Deploying ROSA with AWS Gateway Load Balancer

This article will outline a solution for deploying ROSA using a frictionless secure approach to gain access to the Internet using AWS Gateway Load Balancer without the complexities of proxies.

ROSA requires access to publically hosted Red Hat image repositories in order to install the core components and operators that form the basis of OpenShift. Thus solving the problem of how to efficiently and securely connect to the Internet in adherence to corporate policies typically embedded inside firewall rules is a key challenge to address at the start of the cloud adoption journey.

There are several well-known approaches such as using NAT Gateways and Transit Gateways with proxies that are documented in the ROSA installation guide as per the following link: https://docs.openshift.com/rosa/welcome/index.html

One method that is not so well-known, and which does not rely on proxies, is to leverage AWS Gateway Load Balancer as a "bump-in-the-wire" in conjunction with firewall appliances to perform packet inspection and payload manipulation in a non-intrusive manner before exiting traffic via the Interent Gateway towards the Red Hat image repositories. Eliminating the dependency on proxies reduces friction, and improves security by leveraging the multiple capabilities of various stateful firewall appliances available from the AWS Marketplace.

From an architectural pattern there are two options for integrating ROSA with AWS Gateway Load Balancer (GWLB) that this article will examine. The first pattern leverages a centralised control plane with a distributed data plane and the second leverages a centralised control and data plane.

The first architectural pattern is shown in the following diagram.

The control plane is composed of a central GWLB and a scaleable fleet of inline firewall appliances that implement allowed egress paths. For ROSA these are defined in the following link: https://docs.openshift.com/rosa/rosa_getting_started/rosa-aws-prereqs.html#osd-aws-privatelink-firewall-prerequisites. The main defining feature of this architectural pattern is that all Internet-bound traffic is hairpinned through the control plane and requires an additional public subnet to be created in the VPC hosting the private ROSA cluster. The advantage of this design is that if static public IP addresses are configured for the Internet Gateway then any third-party system can define explicit firewall rules for allowing traffic originating from the ROSA cluster to support data-sensitive exchanges.

If this is not a requirement or the third-party system is willing to allow a more dynamic address space then the second architectural pattern can be considered. In this scenario traffic from ROSA and all other VPCs egress to the Internet via a shared Internet Gateway. The advantage is that no additional public subnet needs to be created in the VPC hosting ROSA.

For both architectural patterns what makes the solution non-intrusive and work seamlessly is that it only requires for the default route (0.0.0.0/0) in the subnet hosting the ROSA cluster to point to Gateway Load Balancer endpoint in the corresponding availability zone. Assuming that the adjoining Security VPC has been correctly setup no further configuration is required on the ROSA cluster to make this work. Note that configuring subnet route tables is only supported when ROSA is deployed in a customer-managed VPC.





