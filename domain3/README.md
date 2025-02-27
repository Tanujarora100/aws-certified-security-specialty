# Domain 3 - Infrastructure Security
### Bastion Server
- Is a server which runs at a network edge
- It is hardened to withstand attacks from public networks
- Acts as a gatekeeper between two different zones (public => private)
- A jumpserver is a generic term used for a server between two or more networking zones. It allows to jump between networks using this server
- A bastion host is a subset o jumpserver, it runs between public and private networks and it is hardened A bastion host is essentially used an an ingress control point, a single hole at the perimeter of a private network. It is extensively logged, it has some form of authentication (SSH, ID federation) and has tightly controlled network security
- VPN
### VPC Peering
- VPC peering lets us create a private and encrypted network link between two VPCs
- One peering connections link two and only two VPCs
- VPC peering can be created in the same region or cross-region. Peered VPCs can be in the same account or cross-account
- When a VPC peer is created, we can enable the option to have public host names being resolved to private IPs inside the VPCs. `Same DNS can be used to locate public services and resolve their hostname to their private IP`.
- If the VPCs are in the same region, they can reference each other's security group IDs. In different regions we have to reference IP addresses and IP ranges
- VPC peering does not support transitive peering: if VPC_A and VPC_B is peering, but also VPC_B and VPC_C are peering, this does not mean that VPC_A is peered with VPC_C
- When a peering is created, a logical gateway is created in both VPC. This means we have to configure routing
- The IP ranges of the VPC CIDRs can not overlap!
- Any data between VPC peers is encrypted. If the peered VPCs are in different regions, the data between peers will go over `AWS global transit network`

#### VPC Peering Considerations
- Security Groups are tied to a VPC in a region. They are attached to ENIs and the can reference each other and themselves
- When we create a peer between VPCs in the same region, SGs from one VPC can reference SGs from other the VPC. If the VPCs are in different account, but same region, we can reference a SG by `accountID/SG ID`
- In case of cross-region peering we can not reference SGs from the other peered VPC
- DNS in peered VPCs:
    - Reminder, in case of a VPC (no VPC peering!):
        - If we are resolving public IPv4 DNS from inside a VPC, it will resolve to the corresponding private IPv4 address of that DNS
        - Outside of the VPC, the public EIP v4 address will resolve to a public or Elastic IPv4 address
    - In case of VPC peering, we can have both options for DNS resolution depending on what setting we choose:
        - **Requester DNS resolution**
        - **Accepter DNS resolution**
    - These settings are independent and they define if one VPC can resolve public DNS to private IP addresses for the other VPC
    - Both VPCs must enable **DNS hostnames** and **DNS resolution** for these settings to work
#### VPC Peering Unsupported Configurations
- The following configurations are not supported by VPC peering:
    - Overlapping CIDR blocks
    - Transitive peering: if A is peered with B and B is peered with C, this does not mean that A and C are also peered
- Handle overlapping CIDR blocks:
    - Split routing in VPC per subnets with different route tables
    - Use more specific routes `(longest prefix wins)` in case of overlaps
- DNS
- EndPoints
- PrivateLink
- NACL & Security Groups
### Transit Gateway
- Transit gateway used to connect multiple vpc together.
- We can connect VPN connection to the transit gateway and give access to all the VPC.
- We can share the tg using Resource Access Manager.
- We can peer transit gateways across regions.
- Traffic Control is done by Route table, these are tgw route tables and not subnet route tables.
- Support `IP MULTICAST` only service.
- Only service to support Equal Cost Multi Path Routing(ECMP).
    - Sending packet to multiple best paths.
    - Can create more tunnels for networks leading to higher throughput of data.
    ![alt text](image-1.png)
- We can connect direct connect and share transit gateway to multiple vpc using Resource Access Manager.
![alt text](image.png)
### CloudFront
- 216 Points of Presense with integration of Shield.
![alt text](image-2.png)
- **Origin**: the source location of the content, can be `S3` or `custom origin` (publicly routable `IPv4` address)
- Custom origin(`HTTP`)
- Origin access control replaced Origin access identity in case of s3 buckets.
- **Distribution**: unit of configuration within CloudFront, which gets deployed out to the CloudFront network. Almost everything is configured within the distribution directly or indirectly.
- **Edge Location**: pieces of global infrastructure where the content is cached. They are smaller than AWS regions, but they are way bigger in number and more widely distributed. Can be used to distribute static data only
- **Regional Edge Cache**: larger version of an edge location, but there are fewer of them. Provides another layer of caching. `If we are using S3 origins, the region edge location is not used in case there is a cache miss for the edge location`. 
    - **Only custom origin can use the regional edge cache**
- **Origin fetch**: the content is fetched from the origin in case of a cache miss on the edge location
- **Behavior**: it is configuration within a distribution. Origins are directly linked to behaviors, behaviors are linked to distributions
- **CloudFront Geo Restriction**: Gives a way to restrict content to a particular location
    - They are 2 types of restriction: Whitelist or Blacklist countries
    - Only works with countries!
    - Uses a `GeoIP` database with 99.8% accuracy
    - Applies to the entire distribution
    - Use Case: Copyright Laws
    - 3rd Party Geolocation: Completely customizable, can be used to filter on lots of other attributes, example: username, user attributes, etc. Requires an application server in front of CloudFront, which controls `weather the customer has access to the content` or not The application generates a `signed url/cookie` which is returned to the browser. This can be sent to CloudFront for authorization
![alt text](image-3.png)
### WAF
- AWS WAF is a web application firewall that lets you monitor the HTTP and HTTPS requests that are forwarded to an Amazon CloudFront distribution, and Amazon API Gateway REST API, an Application Load Balancer, or an AWS AppSync GraphQL API. 
- AWS WAF also lets you control access to your content. Based on conditions that you specify, such as the IP addresses that requests originate from or the values of query strings, Amazon CloudFront, Amazon API Gateway, Application Load Balancer, or AWS AppSync responds to requests either with the requested content or with an HTTP 403 status code (Forbidden).
- A web access control list (web ACL) gives you fine-grained control over all of the HTTP(S) web requests that your protected resource responds to. You can protect Amazon CloudFront, Amazon API Gateway, Application Load Balancer, and AWS AppSync resources. You can use criteria like the following to allow or block requests: 1. IP address origin of the request 2. Country of origin of the request 3. String match or regular expression (regex) match in a part of the request 4. Size of a particular part of the request 5. Detection of malicious SQL code or scripting
- AWS WAF supports full logging of all web requests inspected by the service. Customers can store these logs in Amazon S3 for compliance and auditing needs as well as use them for debugging and additional forensics.AWS WAF logs now provide raw HTTP/S headers along with information on which AWS WAF rules are triggered. This is useful for troubleshooting custom WAF rules and Managed Rules for AWS WAF. These logs will be made available via Amazon Kinesis Data Firehose in `JSON format`.
- Enabling AWS WAF full logs is done in two steps. First, on the Amazon Kinesis console, create an instance of the Amazon Kinesis Data Firehose in the relevant account(s). As part of this configuration, customers can choose a destination for the data from Amazon S3, Amazon ElasticSearch, or Amazon RedShift. Customers can also leverage third-party tool(s) from Splunk or Sumo Logic to enable advanced SIEM solutions, giving them a platform for advanced monitoring. Second, on the AWS WAF console, enable the logs and select the Firehose instance. When configuring, customers also have the option of redacting fields from web requests that they do not want to be logged.
- Firewall Manager
- Shield
- API Gateway
- Network Firewall
- Route53
- SES
