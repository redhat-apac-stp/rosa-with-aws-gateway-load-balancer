# Deploying ROSA with AWS Gateway Load Balancer

This article will outline a solution for deploying a managed Red Hat OpenShift Service on AWS (ROSA) by applying a secure approach to gain frictionless access to the public Internet using AWS Gateway Load Balancer without the complexities associated with traditional proxy-based solutions.

ROSA requires access to publically hosted Red Hat image repositories in order to download core components and operators that form the basis of the OpenShift platform. Thus solving the problem of how to efficiently and securely route egress traffic to the Internet in adherence with corporate compliance policies typically involves the introduction of a firewall in the service chain.

There are several well-known approaches to solving this problem such as using NAT or Transit Gateways in conjunction with proxies that are documented in the official ROSA documentation which can be found here: https://docs.openshift.com/rosa/welcome/index.html

One method that is not so well-known, and which does not rely on proxies, is to leverage AWS Gateway Load Balancer as a "bump-in-the-wire" in conjunction with firewall appliances to perform packet inspection and payload manipulation in a non-intrusive manner before exiting traffic via the Interent Gateway. Eliminating the dependency on proxies reduces friction and improves security by leveraging the capabilities of stateful firewall appliances available from the AWS Marketplace.

From an architectural pattern there are two options for integrating ROSA with AWS Gateway Load Balancer (GWLB) that this article will examine. The first pattern leverages a centralised control plane with a distributed data plane and the second leverages a centralised control with data plane.

The first architectural pattern is shown in the following diagram. To aid visual simplicity not all components deployed by ROSA are shown. It is assumed that ROSA will be deployed in a customer-managed VPC in private subnets as this is a common approach for customers that already have a Landing Zone defined.

https://github.com/redhat-apac-stp/rosa-with-aws-gateway-load-balancer/blob/main/ROSA%20-%20AWS%20Gateway%20Load%20Balancer%20-%20A.png

The control plane is composed of a central GWLB and a scaleable fleet of inline firewall appliances that allow egress traffic originating from ROSA as defined by the following link: https://docs.openshift.com/rosa/rosa_getting_started/rosa-aws-prereqs.html#osd-aws-privatelink-firewall-prerequisites. 

The main defining feature of this architectural pattern is that all Internet-bound traffic is hairpinned through the control plane and requires an additional public subnet in the VPC hosting the private ROSA cluster for routing to the Internet Gateway. The advantage of this design is that allows for bring-your-own static public IP addresses to be configured on the Internet Gateway that third-party systems to which ROSA needs to communicate may rely upon.

If this is not a requirement or the third-party system is willing to allow a more dynamic address space then a more flexible second architectural pattern can be considered as shown in the following diagram. In this scenario traffic from ROSA and all other VPCs egress to the Internet via a shared Internet Gateway. The advantage is that no additional public subnet and Internet Gateway needs to be created.

https://github.com/redhat-apac-stp/rosa-with-aws-gateway-load-balancer/blob/main/ROSA%20-%20AWS%20Gateway%20Load%20Balancer%20-%20B.png

For both architectural patterns what makes the solution non-intrusive and work seamlessly is that it only requires for the default route (0.0.0.0/0) in the subnet hosting the ROSA cluster to point to a Gateway Load Balancer endpoint in the same availability zone. Assuming that the adjoining Security VPC has been correctly setup no further configuration is required on the ROSA cluster to make this work.

## Conclusion

Deploying ROSA in conjunction with GWLB enables frictionless secure access to the Internet with a compelling price-point (when compared to other AWS offerings such as NAT Gateway and Transit Gateway) for a lower overall TCO.

