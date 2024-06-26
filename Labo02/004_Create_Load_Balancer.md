### Deploy the elastic load balancer

In this task you will create a load balancer in AWS that will receive
the HTTP requests from clients and forward them to the Drupal
instances.

![Schema](./img/CLD_AWS_INFA.PNG)

## Task 01 Prerequisites for the ELB

* Create a dedicated security group

|Key|Value|
|:--|:--|
|Name|SG-DEVOPSTEAM[XX]-LB|
|Inbound Rules|Application Load Balancer|
|Outbound Rules|Refer to the infra schema|

```bash
aws ec2 create-security-group --group-name SG-DEVOPSTEAM14-LB --description "Security group for DevOps Team 14 Load Balancer" --vpc-id vpc-03d46c285a2af77ba
{
    "GroupId": "sg-0360061815f731650"
}
```

```bash
aws ec2 authorize-security-group-ingress --protocol tcp --port 8080 --cidr 10.0.0.0/28 --group-id sg-0360061815f731650

{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-07c63206eb2033a59",
            "GroupId": "sg-0360061815f731650",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.0.0/28"
        }
    ]
}
```

* Create the Target Group

|Key|Value|
|:--|:--|
|Target type|Instances|
|Name|TG-DEVOPSTEAM[XX]|
|Protocol and port|Refer to the infra schema|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Protocol version|HTTP1|
|Health check protocol|HTTP|
|Health check path|/|
|Port|Traffic port|
|Healthy threshold|2 consecutive health check successes|
|Unhealthy threshold|2 consecutive health check failures|
|Timeout|5 seconds|
|Interval|10 seconds|
|Success codes|200|

```bash
aws elbv2 create-target-group --name TG-DEVOPSTEAM14 --protocol HTTP --port 8080 --vpc-id vpc-03d46c285a2af77ba --target-type instance --protocol-version HTTP1 --health-check-protocol HTTP --health-check-path / --health-check-port 8080 --health-check-interval-seconds 10 --health-check-timeout-seconds 5 --healthy-threshold-count 2 --unhealthy-threshold-count 2 --matcher "HttpCode=200"



{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM14/51f38f269fe06e8a",
            "TargetGroupName": "TG-DEVOPSTEAM14",
            "Protocol": "HTTP",
            "Port": 8080,
            "VpcId": "vpc-03d46c285a2af77ba",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "8080",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 10,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 2,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/",
            "Matcher": {
                "HttpCode": "200"
            },
            "TargetType": "instance",
            "ProtocolVersion": "HTTP1",
            "IpAddressType": "ipv4"
        }
    ]
}

```
```bash
aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM14/51f38f269fe06e8a --targets Id=i-0a71f8b0c14a977f3 Id=i-0aab65b64442b4464

[no output]
```

## Task 02 Deploy the Load Balancer

[Source](https://aws.amazon.com/elasticloadbalancing/)

* Create the Load Balancer

|Key|Value|
|:--|:--|
|Type|Application Load Balancer|
|Name|ELB-DEVOPSTEAM99|
|Scheme|Internal|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Security group|Refer to the infra schema|
|Listeners Protocol and port|Refer to the infra schema|
|Target group|Your own target group created in task 01|

Provide the following answers (leave any
field not mentioned at its default value):

```bash
aws elbv2 create-load-balancer --name ELB-DEVOPSTEAM14 --type application --scheme internal --ip-address-type ipv4 --subnets subnet-08532e833f35bd94d subnet-03f814992c543a1f8 --security-groups sg-0360061815f731650 --tags Key=Name,Value=ELB-DEVOPSTEAM14


[OUTPUT]
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM14/3bf0a2ed52a20016",
            "DNSName": "internal-ELB-DEVOPSTEAM14-609778290.eu-west-3.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z3Q77PNBQS71R4",
            "CreatedTime": "2024-03-21T15:06:11.070000+00:00",
            "LoadBalancerName": "ELB-DEVOPSTEAM14",
            "Scheme": "internal",
            "VpcId": "vpc-03d46c285a2af77ba",
            "State": {
                "Code": "provisioning"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "eu-west-3a",
                    "SubnetId": "subnet-03f814992c543a1f8",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "eu-west-3b",
                    "SubnetId": "subnet-08532e833f35bd94d",
                    "LoadBalancerAddresses": []
                }
            ],
            "SecurityGroups": [
                "sg-0360061815f731650"
            ],
            "IpAddressType": "ipv4"
        }
    ]
}
```
```bash
aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM14/3bf0a2ed52a20016 --protocol HTTP --port 8080 --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM14/51f38f269fe06e8a

{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:listener/app/ELB-DEVOPSTEAM14/3bf0a2ed52a20016/ee8f3d684ba563f4",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM14/3bf0a2ed52a20016",
            "Port": 8080,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM14/51f38f269fe06e8a",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM14/51f38f269fe06e8a",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
            ]
        }
    ]
}
```

* Get the ELB FQDN (DNS NAME - A Record)

```bash
[INPUT]
aws elbv2 describe-load-balancers --names ELB-DEVOPSTEAM14 --query "LoadBalancers[0].DNSName" --output text
[OUTPUT]
internal-ELB-DEVOPSTEAM14-609778290.eu-west-3.elb.amazonaws.com
```

* Get the ELB deployment status

Note : In the EC2 console select the Target Group. In the
       lower half of the panel, click on the **Targets** tab. Watch the
       status of the instance go from **unused** to **initial**.

* Ask the DMZ administrator to register your ELB with the reverse proxy via the private teams channel

* Update your string connection to test your ELB and test it

```bash
ssh devopsteam14@15.188.43.46 -i CLD_KEY_DMZ_DEVOPSTEAM14.pem -L 80:internal-ELB-DEVOPSTEAM14-609778290.eu-west-3.elb.amazonaws.com:8080
```

* Test your application through your ssh tunneling

```bash
[INPUT]
curl localhost

[OUTPUT]
StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    <html lang="en" dir="ltr" style="--color--primary-hue:202;--color--primary-saturation:79%;--color--primary-lightness:50">
                      <head>
                        <meta charset="utf-8" />
                    <meta name="Generator" c...
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    X-Drupal-Dynamic-Cache: MISS
                    Content-language: en
                    X-Content-Type-Options: nosniff
                    X-Frame-Options: SAMEORIGIN
                    X-Generator: Drupal 10 (https://www.drupal.or...
Forms             : {search-block-form, search-block-form--2}
Headers           : {[Connection, keep-alive], [X-Drupal-Dynamic-Cache, MISS], [Content-language, en], [X-Content-Type-Options, nosniff]...}
Images            : {}
InputFields       : {@{innerHTML=; innerText=; outerHTML=<INPUT id=edit-keys title="Enter the terms you wish to search for." class="form-search form-element form-element--type-search
                    form-element--api-search" maxLength=128 size=15 name=keys data-drupal-selector="edit-keys" placeholder="Search by keyword or phrase.">; outerText=; tagName=INPUT; id=edit-keys;
                    title=Enter the terms you wish to search for.; class=form-search form-element form-element--type-search form-element--api-search; maxLength=128; size=15; name=keys;
                    data-drupal-selector=edit-keys; placeholder=Search by keyword or phrase.}, @{innerHTML=; innerText=; outerHTML=<INPUT id=edit-keys--2 title="Enter the terms you wish to search for."
                    class="form-search form-element form-element--type-search form-element--api-search" maxLength=128 size=15 name=keys data-drupal-selector="edit-keys" placeholder="Search by keyword or
                    phrase.">; outerText=; tagName=INPUT; id=edit-keys--2; title=Enter the terms you wish to search for.; class=form-search form-element form-element--type-search form-element--api-search;
                    maxLength=128; size=15; name=keys; data-drupal-selector=edit-keys; placeholder=Search by keyword or phrase.}}
Links             : {@{innerHTML=Skip to main content ; innerText=Skip to main content ; outerHTML=<A class="visually-hidden focusable skip-link" href="#main-content">Skip to main content </A>;
                    outerText=Skip to main content ; tagName=A; class=visually-hidden focusable skip-link; href=#main-content}, @{innerHTML=My blog; innerText=My blog; outerHTML=<A title=Home href="/"
                    rel=home>My blog</A>; outerText=My blog; tagName=A; title=Home; href=/; rel=home}, @{innerHTML=<SPAN class="primary-nav__menu-link-inner primary-nav__menu-link-inner--level-1">Home</SPAN>
                    ; innerText=Home ; outerHTML=<A class="primary-nav__menu-link primary-nav__menu-link--link primary-nav__menu-link--level-1 is-active" href="/"
                    data-drupal-selector="primary-nav-menu-link-has-children" data-drupal-link-system-path="<front>"><SPAN class="primary-nav__menu-link-inner
                    primary-nav__menu-link-inner--level-1">Home</SPAN> </A>; outerText=Home ; tagName=A; class=primary-nav__menu-link primary-nav__menu-link--link primary-nav__menu-link--level-1 is-active;
                    href=/; data-drupal-selector=primary-nav-menu-link-has-children; data-drupal-link-system-path=<front>}, @{innerHTML=Log in; innerText=Log in; outerHTML=<A class="secondary-nav__menu-link
                    secondary-nav__menu-link--link secondary-nav__menu-link--level-1" href="/user/login" data-drupal-link-system-path="user/login">Log in</A>; outerText=Log in; tagName=A;
                    class=secondary-nav__menu-link secondary-nav__menu-link--link secondary-nav__menu-link--level-1; href=/user/login; data-drupal-link-system-path=user/login}...}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 16461

```

#### Questions - Analysis

* On your local machine resolve the DNS name of the load balancer into
  an IP address using the `nslookup` command (works on Linux, macOS and Windows). Write
  the DNS name and the resolved IP Address(es) into the report.

```
$ nslookup internal-ELB-DEVOPSTEAM14-609778290.eu-west-3.elb.amazonaws.com

Non-authoritative answer:
Name:    internal-ELB-DEVOPSTEAM14-609778290.eu-west-3.elb.amazonaws.com
Addresses:  10.0.14.4
          10.0.14.134
```

* From your Drupal instance, identify the ip from which requests are sent by the Load Balancer.

Help : execute `tcpdump port 8080`

```
18:22:20.689843 IP 10.0.14.4.24254 > 10.0.14.10.http-alt: Flags [P.], seq 1:131, ack 1, win 106, options [nop,nop,TS val 514770564 ecr 4148482609], length 130: HTTP: GET / HTTP/1.1
```

* In the Apache access log identify the health check accesses from the
  load balancer and copy some samples into the report.

```.log
10.0.14.137 - - [28/Mar/2024:15:16:07 +0000] "GET / HTTP/1.1" 500 67
10.0.14.13 - - [28/Mar/2024:15:16:16 +0000] "GET / HTTP/1.1" 500 67
10.0.14.137 - - [28/Mar/2024:15:16:17 +0000] "GET / HTTP/1.1" 500 67
10.0.14.13 - - [28/Mar/2024:15:16:26 +0000] "GET / HTTP/1.1" 500 67
10.0.14.137 - - [28/Mar/2024:15:16:27 +0000] "GET / HTTP/1.1" 500 67
10.0.14.13 - - [28/Mar/2024:15:16:36 +0000] "GET / HTTP/1.1" 500 67
10.0.14.137 - - [28/Mar/2024:15:16:37 +0000] "GET / HTTP/1.1" 500 67
```
