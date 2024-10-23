## Getting Started with Google Cloud Learning Path
 - [2. Getting Started with Google Cloud Learning Path](#google-cloud-computing-foundations-certificate)
   - [2.1 A Tour of Google Cloud Hands-on Labs (45min)](#a-tour-of-google-cloud-hands-on-lab)
   - [2.2 Implement Load Balancing on Compute Engine (3h:30min)](#implement-load-balancing-on-compute-engine))
     - [2.2.1 Creating a Virtual Machine]()
     - [2.2.2 Compute Engine: Qwik Start Windows]()
     - [2.2.3 Getting Started with Cloud Shell and gcloud]()
     - [2.2.4 Set Up Network and HTTP Load Balancers](#224-set-up-network-and-http-load-balancers)
     - [2.2.5 Implement Load Balancing on Compute Engine: Challenge Lab]() 
   - [2.3 Google Cloud Fundamentals: Core Infrastructure (5h)](#google-cloud-fundamentals-core-infrastructure)
   - [2.4 Set Up an App Dev Environment on Google Cloud (4h)](#set-up-an-app-dev-environment-on-google-cloud))
   - [2.5 Introduction to AI and Machine Learning on Google Cloud (8h)](#introduction-to-ai-and-machine-learning-on-google-cloud)
   - [2.6 Prepare Data for ML APIs on Google Cloud (6h:30min)](#prepare-data-for-ml-apis-on-google-cloud)
   - [2.7 Google Cloud Fundamentals for AWS Professionals (8h)](#google-cloud-fundamentals-for-aws-professionals)





### A Tour of Google Cloud Hands-on Lab
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### 2.2 Implement Load Balancing on Compute Engine

#### 2.2.1 Creating a Virtual Machine [2]
#### 2.2.2 Compute Engine: Qwik Start - Windows [2]
####	2.2.3 Getting Started with Cloud Shell and gcloud [2]
#### 2.2.4 Set Up Network and HTTP Load Balancers

<br>  

##### Set Up Network and HTTP Load Balancers

**Overview**  

In this hands-on lab you learn the differences between a network load balancer and an HTTP load balancer, and how to set them up for your applications running on Compute Engine virtual machines (VMs).

There are several ways you can load balance on Google Cloud. This lab takes you through the set up of the following load balancers:

[Network Load Balancer](https://cloud.google.com/load-balancing/docs/network/networklb-backend-service?hl=pt-br)  
[HTTP(s) Load Balancer](https://cloud.google.com/load-balancing/docs/https?hl=pt-br)  

You are encouraged to type the commands yourself, which can help you learn the core concepts. Many labs include a code block that contains the required commands. You can easily copy and paste the commands from the code block into the appropriate places during the lab.

**What you'll learn**  

- Set up a network load balancer.  
- Set up an HTTP load balancer.  
- Get hands-on experience learning the differences between network load balancers and HTTP load balancers.  

<br>  

**Task 1. Set the default region and zone for all resources**  


1. Set the default region:
```
gcloud config set compute/region us-east4
```
2. In Cloud Shell, set the default zone:
```
gcloud config set compute/zone us-east4-a
```
Learn more about choosing zones and regions in Compute Engine's documentation Regions and Zones Guide.
<br>  

**Task 2. Create multiple web server instances**  

For this load balancing scenario, create three Compute Engine VM instances and install Apache on them, then add a firewall rule that allows HTTP traffic to reach the instances.

The code provided sets the zone to us-east4-a. Setting the tags field lets you reference these instances all at once, such as with a firewall rule. These commands also install Apache on each instance and give each instance a unique home page.

1. Create a virtual machine www1 in your default zone using the following code:
```
  gcloud compute instances create www1 \
    --zone=us-east4-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

2. Create a virtual machine www2 in your default zone using the following code:
```
gcloud compute instances create www2 \
    --zone=us-east4-a \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www2</h3>" | tee /var/www/html/index.html'
```

3. Create a virtual machine www3 in your default zone.
```
  gcloud compute instances create www3 \
    --zone=us-east4-a  \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www3</h3>" | tee /var/www/html/index.html'
```

4. Create a firewall rule to allow external traffic to the VM instances:
```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

Now you need to get the external IP addresses of your instances and verify that they are running.


5. Run the following to list your instances. You'll see their IP addresses in the EXTERNAL_IP column:    
```
gcloud compute instances list
```

6. Verify that each instance is running with curl, replacing [IP_ADDRESS] with the IP address for each of your VMs:
```
curl http://[IP_ADDRESS]
```

<br>  

**Task 3. Configure the load balancing service**  

When you configure the load balancing service, your virtual machine instances receives packets that are destined for the static external IP address you configure. Instances made with a Compute Engine image are automatically configured to handle this IP address.

1. Create a static external IP address for your load balancer:
```
gcloud compute addresses create network-lb-ip-1 \
  --region us-east4
```

2. Add a legacy HTTP health check resource:
```
gcloud compute http-health-checks create basic-check
```

3. Add a target pool in the same region as your instances. Run the following to create the target pool and use the health check, which is required for the service to function:
gcloud compute target-pools create www-pool \
```
   --region us-east4 --http-health-check basic-check
```

5. Add the instances to the pool:
gcloud compute target-pools add-instances www-pool \
```
    --instances www1,www2,www3
``` 

6. Add a forwarding rule:
```
gcloud compute forwarding-rules create www-rule \
    --region  us-east4 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

<br>  
    
**Task 4. Sending traffic to your instances**  

Now that the load balancing service is configured, you can start sending traffic to the forwarding rule and watch the traffic be dispersed to different instances.

1. Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:
```
gcloud compute forwarding-rules describe www-rule --region us-east4
```
2. Access the external IP address
```
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-east4 --format="json" | jq -r .IPAddress)
```
3. how the external IP address
```
echo $IPADDRESS
```
4.  Use curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command:
```
while true; do curl -m1 $IPADDRESS; done
```
The response from the curl command alternates randomly among the three instances. If your response is initially unsuccessful, wait approximately 30 seconds for the configuration to be fully loaded and for your instances to be marked healthy before trying again.


5. Use Ctrl + C to stop running the command.

<br>  

**Task 5. Create an HTTP load balancer**  

HTTP(S) Load Balancing is implemented on Google Front End (GFE). GFEs are distributed globally and operate together using Google's global network and control plane. You can configure URL rules to route some URLs to one set of instances and route other URLs to other instances.

Requests are always routed to the instance group that is closest to the user, if that group has enough capacity and is appropriate for the request. If the closest group does not have enough capacity, the request is sent to the closest group that does have capacity.

To set up a load balancer with a Compute Engine backend, your VMs need to be in an instance group. The managed instance group provides VMs running the backend servers of an external HTTP load balancer. For this lab, backends serve their own hostnames.


1. First, create the load balancer template:
```
gcloud compute instance-templates create lb-backend-template \
   --region=us-east4 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```
(MIGs) let you operate apps on multiple identical VMs. You can make your workloads scalable and highly available by taking advantage of automated MIG services, including: autoscaling, autohealing, regional (multiple zone) deployment, and automatic updating.

2. Create a managed instance group based on the template:
gcloud compute instance-groups managed create lb-backend-group \
```
   --template=lb-backend-template --size=2 --zone=us-east4-a
```

4. Create the fw-allow-health-check firewall rule.
```
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

Note: The ingress rule allows traffic from the Google Cloud health checking systems (130.211.0.0/22 and 35.191.0.0/16). This lab uses the target tag allow-health-check to identify the VMs

4. Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer:
gcloud compute addresses create lb-ipv4-1 \
```
  --ip-version=IPV4 \
  --global
```

Note the IPv4 address that was reserved:

```
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
```

5. Create a health check for the load balancer:
gcloud compute health-checks create http http-basic-check \
```
   --port 80
```

Note: Google Cloud provides health checking mechanisms that determine whether backend instances respond properly to traffic. For more information, please refer to the Creating health checks document.

6. Create a backend service:
```
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```

7. Add your instance group as the backend to the backend service:
```
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=us-east4-a \
  --global
```
  
8. Create a URL map to route the incoming requests to the default backend service:
gcloud compute url-maps create web-map-http \
```
    --default-service web-backend-service
``` 

Note: URL map is a Google Cloud configuration resource used to route requests to backend services or backend buckets. For example, with an external HTTP(S) load balancer, you can use a single URL map to route requests to different destinations based on the rules configured in the URL map:
Requests for https://example.com/video go to one backend service.
Requests for https://example.com/audio go to a different backend service.
Requests for https://example.com/images go to a Cloud Storage backend bucket.
Requests for any other host and path combination go to a default backend service.


9. Create a target HTTP proxy to route requests to your URL map:
gcloud compute target-http-proxies create http-lb-proxy \
```
    --url-map web-map-http
``` 

10. Create a global forwarding rule to route incoming requests to the proxy:
```
gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80
```

<br>  

**Task 6. Testing traffic sent to your instances**  

1. In the Google Cloud console, from the Navigation menu, go to Network services > Load balancing.  

2. Click on the load balancer that you just created (web-map-http).  

3. In the Backend section, click on the name of the backend and confirm that the VMs are Healthy. If they are not healthy, wait a few moments and try reloading the page.  

4. When the VMs are healthy, test the load balancer using a web browser, going to http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's IP address.  

This may take three to five minutes. If you do not connect, wait a minute, and then reload the browser.  

Your browser should render a page with content showing the name of the instance that served the page, along with its zone (for example, Page served from: lb-backend-group-xxxx).  

<br>  <br>  

####	2.2.5 Implement Load Balancing on Compute Engine: Challenge Lab [1]
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Google Cloud Fundamentals: Core Infrastructure

### Set Up an App Dev Environment on Google Cloud

### Introduction to AI and Machine Learning on Google Cloud

### Prepare Data for ML APIs on Google Cloud

### Google Cloud Fundamentals for AWS Professionals
