# Installing ROSA with AWS Gateway Load Balancer

This article will outline a solution for installing Red Hat OpenShift Service on AWS (ROSA) using AWS Gateway Load Balancer for frictionless access to the public Internet minus the complexities associated with traditional proxy-based solutions.

ROSA requires access to publically hosted Red Hat registries in order to download images for core platform components and operators that form the basis of any OpenShift installation as well as post-installation telemetry for cluster health monitoring. Thus solving the problem of how to efficiently and securely route egress traffic to the Internet in adherence to corporate compliance policies typically involves the introduction of a firewall device into the network communication chain.

There are several well-known approaches to solving this problem such as using NAT or Transit Gateways in conjunction with proxies that are documented in the official ROSA documentation which can be found here: https://docs.openshift.com/rosa/welcome/index.html

One method that is not so well-known, and which does not rely on proxies, is to leverage AWS Gateway Load Balancer as a "bump-in-the-wire" in conjunction with firewall appliances which perform packet inspection and payload manipulation in a non-intrusive manner before exiting traffic via the Interent Gateway. Eliminating the dependency on proxies reduces friction and improves security by leveraging the capabilities of stateful firewall appliances available from the AWS Marketplace.

From an architectural pattern there are two options for integrating ROSA with AWS Gateway Load Balancer (GWLB) that this article will examine. The first pattern leverages a centralised control plane with a distributed data plane and the second leverages a centralised control and data plane.

The first architectural pattern is shown in the following diagram. To aid visual simplicity not all components deployed by ROSA are shown. It is assumed that ROSA will be deployed in a customer-managed VPC with private subnets; this is a common approach for customers adopting a Landing Zone construct to organize multiple VPCs for the purpose of distributing systems.

<img src=https://github.com/redhat-apac-stp/rosa-with-aws-gateway-load-balancer/blob/main/ROSA%20-%20AWS%20Gateway%20Load%20Balancer%20-%20A.png>

The control plane is composed of a central GWLB instance and a scaleable fleet of inline firewall appliances that allow egress traffic originating from a ROSA cluster to the following list of destinations: https://docs.openshift.com/rosa/rosa_getting_started/rosa-aws-prereqs.html#osd-aws-privatelink-firewall-prerequisites. 

The main defining feature of this architectural pattern is that all Internet-bound traffic is hairpinned through the control plane and requires an additional public subnet in the VPC hosting the ROSA cluster prior to exiting traffic via the Internet Gateway. The advantage of this design is that supports use-cases involving bring-your-own static public IP addresses that third-party systems can allowlist for the purpose of communicating with private ROSA clusters without the need for building a site-to-site VPN.

If this is not a requirement and third-party systems support open communication then a more flexible second architectural pattern can be considered as shown in the following diagram. In this scenario traffic from ROSA and all other VPCs egress to the Internet via a shared Internet Gateway enabled by dynamic IP addresses. The advantage is that no additional public subnet or Internet Gateway needs to be configured in the VPC hosting a private ROSA cluster.

<img src=https://github.com/redhat-apac-stp/rosa-with-aws-gateway-load-balancer/blob/main/ROSA%20-%20AWS%20Gateway%20Load%20Balancer%20-%20B.png>

For both architectural patterns what makes the solution non-intrusive and work seamlessly is that it only requires for the default route (0.0.0.0/0) of the subnets hosting the ROSA cluster to point to the corresponding Gateway Load Balancer endpoint of the availability zone. Assuming that the adjoining Security VPC has been correctly setup (see link to firewall rules above) no additional configuration is required on the ROSA cluster (e.g., a cluster-wide proxy) to make this solution work.

## Conclusion

Deploying ROSA in conjunction with GWLB enables frictionless secure access to the Internet at a compelling price-point when compared to other AWS offerings such as NAT Gateway and Transit Gateway, resulting in a lower overall TCO.
