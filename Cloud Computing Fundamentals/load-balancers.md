# Set Up Network and HTTP Load Balancers

## Task 1. Set the default region and zone for all resources

### Set default region

    gcloud config set compute/region us-west1

### Set default zone

    gcloud config set compute/zone us-west1-c

## Task 2. Create multiple web server instances

### Create 3 VMs www1, www2 and www3

      gcloud compute instances create www1 \
    --zone=us-west1-c \
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

### Create a firewall rule to allow external traffic to the VM instances:

    gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80

## Task 3. Configure the load balancing service

### Create a static external IP address for your load balancer:

    gcloud compute addresses create network-lb-ip-1 \
  --region us-west1

### Add a legacy HTTP health check resource:

    gcloud compute http-health-checks create basic-check

### Add a target pool in the same region as your instances.

    gcloud compute target-pools create www-pool \
  --region us-west1 --http-health-check basic-check

### Add the instances to the pool:

    gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3

### Add a forwarding rule:

    gcloud compute forwarding-rules create www-rule \
    --region  us-west1 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

## Task 4. Sending traffic to your instances

### View the external IP address of the www-rule forwarding rule used by the load balancer:

    gcloud compute forwarding-rules describe www-rule --region us-west1

### Access the external IP address

    IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-west1 --format="json" | jq -r .IPAddress)

### Show the external IP address

    echo $IPADDRESS

### Test

    while true; do curl -m1 $IPADDRESS; done

## Task 5. Create an HTTP load balancer

###  create the load balancer template

    gcloud compute instance-templates create lb-backend-template \
   --region=us-west1 \
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

### Create a managed instance group based on the template:

    gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-west1-c

### Create the fw-allow-health-check firewall rule.

    gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80

### set up a global static external IP address that your customers use to reach your load balancer:

    gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global

### Note the IPV4

    gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global

### Create a health check for the load balancer:

    gcloud compute health-checks create http http-basic-check \
  --port 80

### Create a backend service:

    gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

### Add your instance group as the backend to the backend service:

    gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=us-west1-c \
  --global

### Create a URL map to route the incoming requests to the default backend service:

    gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

### Create a target HTTP proxy to route requests to your URL map:

    gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

### Create a global forwarding rule to route incoming requests to the proxy:

    gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80

