# ☁️ GCP Challenge Solutions
💡 Answers to some of the challenges from **Google Cloud Platform** learning paths and courses.

<img width="1500" height="1000" alt="Untitled design (1)" src="https://github.com/user-attachments/assets/505e3f6d-0cdf-40b9-b243-24eacc6de968" />

<hr style="border:0;height:1px;background:#ddd;" />

## 📌 Paths 

### 🎓 Associate Cloud Engineer Certification Learning Path
- [Implement Load Balancing on Compute Engine: Challenge Lab](#load-balancing)
- [Set Up an App Dev Environment on Google Cloud: Challenge Lab](#app-dev-environment)
- [Develop your Google Cloud Network: Challenge Lab](#develop-network)
- [Build Infrastructure with Terraform on Google Cloud: Challenge Lab](#terraform-infra)

### 🤖 Beginner: Introduction to Generative AI Learning Path
- [Prompt Design in Vertex AI: Challenge Lab](#prompt-design)

### 👨‍💻 Professional Cloud Developer Certification Learning Path
- Coming soon - in progress

## 📚 Courses

### ⚡ Cloud Run Functions: 3 Ways
- [Cloud Run Functions: 3 Ways Skill Badge: Challenge Lab: Introductory](#cloud-run-functions)

<hr style="border:0;height:1px;background:#eee;" />

## 📝 Solutions

<a id="load-balancing"></a>
### ⚖️ Implement Load Balancing on Compute Engine: Challenge Lab
<aside>

### **Task 1. Create a project jumphost instance**

//Declaring all the variables

```jsx
export PROJECT_ID=$(gcloud config get-value project)
export REGION=europe-west1
export ZONE=europe-west1-c
export INSTANCE_NAME=nucleus-jumphost-870
export TEMPLATE_NAME=nucleus-backend-template
export MIG_NAME=nucleus-web-mig
export FIREWALL_RULE_NAME=grant-tcp-rule-737
export HEALTH_CHECK_NAME=nucleus-health-check
export BACKEND_SERVICE_NAME=nucleus-backend-service
export URL_MAP_NAME=nucleus-url-map
export HTTP_PROXY_NAME=nucleus-http-proxy
export FORWARDING_RULE_NAME=nucleus-forwarding-rule
```

//Checking if region and zone are ok

```jsx
echo $REGION
echo $ZONE
```

//Configuring global region and zone

```jsx
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
```

//Creating the instance: Name the instance **`Instance name,`** Create the instance in the `ZONE` zone. Use an *e2-micro* machine type. Use the default image type (Debian Linux)

```jsx
gcloud compute instances create $INSTANCE_NAME \
    --zone=$ZONE \
    --machine-type=e2-micro \
    --image-family=debian-11 \
    --image-project=debian-cloud
```

//I refresh the page to check if the instance was created

```jsx
gcloud compute instances list --zone=$ZONE
```

```jsx
gcloud compute instances describe $INSTANCE_NAME --zone=$ZONE
```

</aside>

<aside>

### **Task 2. Set up an HTTP load balancer**

//Setting the script

```jsx
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

//Creating the instance template -*Here the problems started, it doesn’t accept the flag —global. *I deleted the “global” flag, the reason is by default it is global, there’s no need to specify it* 

```jsx
gcloud compute instance-templates create $TEMPLATE_NAME \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata-from-file=startup-script=startup.sh \
   --tags=allow-health-check
```

//Checking the instance template it is ok

```jsx
gcloud compute instance-templates list
```

```jsx
gcloud compute instance-templates describe $TEMPLATE_NAME
```

//Trying to update the GC version: Google Cloud SDK 513.0.0

```jsx
gcloud version
```

//Create a managed instance group based on the template. MIG

```jsx
gcloud compute instance-groups managed create $MIG_NAME \
  --template=$TEMPLATE_NAME \
  --size=2 \
  --zone=$ZONE
```

//Setting the name ports for the managed instance group

```jsx
gcloud compute instance-groups managed set-named-ports $MIG_NAME \
  --named-ports=http:80 \
  --zone=$ZONE
```

//Create a firewall rule named as `Firewall rule` to allow traffic (80/tcp)

```jsx
gcloud compute firewall-rules create $FIREWALL_RULE_NAME \
  --allow=tcp:80 \
  --target-tags=allow-health-check
```

//The health check

```jsx
gcloud compute health-checks create http $HEALTH_CHECK_NAME \
  --port=80
```

//Create a backend service and add your instance group as the backend to the backend service group with named port (http:80).

```jsx
gcloud compute backend-services create $BACKEND_SERVICE_NAME \
  --protocol=HTTP \
  --health-checks=$HEALTH_CHECK_NAME \
  --global
```

//The instance group as the backend

```jsx
  gcloud compute backend-services add-backend $BACKEND_SERVICE_NAME \
	  --instance-group=$MIG_NAME \
	  --instance-group-zone=$ZONE \
	  --balancing-mode=UTILIZATION \
	  --max-utilization=0.8 \
	  --global
```

//Create a URL map, and target the HTTP proxy to route the incoming requests to the default backend service.

```jsx
gcloud compute url-maps create $URL_MAP_NAME \
  --default-service=$BACKEND_SERVICE_NAME
```

//Create a target HTTP proxy to route requests to your URL map

```jsx
gcloud compute target-http-proxies create $HTTP_PROXY_NAME \
  --url-map=$URL_MAP_NAME
```

//The forwarding rule

```jsx
gcloud compute forwarding-rules create $FORWARDING_RULE_NAME \
  --target-http-proxy=$HTTP_PROXY_NAME \
  --ports=80 \
  --global
```

//Checking if everything goes well

```jsx
gcloud compute instances list
gcloud compute instance-templates list
gcloud compute instance-groups managed list --zones=$ZONE
gcloud compute firewall-rules list
gcloud compute health-checks list --global
gcloud compute backend-services list --global
gcloud compute url-maps list --global
gcloud compute target-http-proxies list --global
gcloud compute forwarding-rules list --global
```

```jsx
curl http://[EXTERNAL IP 34.]
```

```jsx
curl http://[EXTERNAL LOAD BALANCER 35.]
```

</aside>
<hr style="border:0;height:1px;background:#eee;" />

<a id="app-dev-environment"></a>
### 🚧 Set Up an App Dev Environment on Google Cloud: Challenge Lab
# **Set Up an App Dev Environment on Google Cloud: Challenge Lab**

<aside>

### **Task 1. Create a bucket**

//Setting the variable

```jsx
export BUCKET_NAME=qwiklabs-gcp-02-d4de145b95a2-bucket
export REGION=us-west1
export ZONE=us-west1-a
export TOPIC_NAME=topic-memories-549
export CLOUD_FUNCTION_NAME=memories-thumbnail-creator
export ADMIN=student-01-5d5f34d5aa73@qwiklabs.net
export VIEWER=student-01-1dd0105a3546@qwiklabs.net
export PROJECT_ID=$(gcloud config get-value project)
```

```jsx
echo $BUCKET_NAME
echo $REGION
echo $ZONE
echo $TOPIC_NAME
echo $CLOUD_FUNCTION_NAME
echo $ADMIN
echo $VIEWER
```

//Setting global region a zone

```jsx
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
```

//Create a bucket 

```jsx
gcloud storage buckets create gs://$BUCKET_NAME --location=$REGION
```

//Checking if is ok created via CLI

```jsx
gcloud storage buckets list
gcloud storage describre $BUCKET_NAME
```

//Checking if is ok created via CONSOLE

Cloud storage > buckets > it should appear the bucket called with the variable $BUCKET_NAME = ….

</aside>

<aside>

### **Task 2. Create a Pub/Sub topic**

//Creating the topic

```jsx
gcloud pubsub topics create $TOPIC_NAME
```

//Checking if created ok

```jsx
gcloud pubsub topics list 
```

</aside>

<aside>

### **Task 3. Create the thumbnail Cloud Run Function**

//Creating the function

*Make sure you set the **Entry point** (Function to execute) to `Cloud Run Function Name` and **Trigger** to `Cloud Storage`.*

*Add the following code to the `index.js`:*

```jsx
const functions = require('@google-cloud/functions-framework');
const { Storage } = require('@google-cloud/storage');
const { PubSub } = require('@google-cloud/pubsub');
const sharp = require('sharp');

functions.cloudEvent('', async cloudEvent => {
  const event = cloudEvent.data;

  console.log(`Event: ${JSON.stringify(event)}`);
  console.log(`Hello ${event.bucket}`);

  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = new Storage().bucket(bucketName);
  const topicName = "";
  const pubsub = new PubSub();

  if (fileName.search("64x64_thumbnail") === -1) {
    // doesn't have a thumbnail, get the filename extension
    const filename_split = fileName.split('.');
    const filename_ext = filename_split[filename_split.length - 1].toLowerCase();
    const filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length - 1); // fix sub string to remove the dot

    if (filename_ext === 'png' || filename_ext === 'jpg' || filename_ext === 'jpeg') {
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      const newFilename = `${filename_without_ext}_64x64_thumbnail.${filename_ext}`;
      const gcsNewObject = bucket.file(newFilename);

      try {
        const [buffer] = await gcsObject.download();
        const resizedBuffer = await sharp(buffer)
          .resize(64, 64, {
            fit: 'inside',
            withoutEnlargement: true,
          })
          .toFormat(filename_ext)
          .toBuffer();

        await gcsNewObject.save(resizedBuffer, {
          metadata: {
            contentType: `image/${filename_ext}`,
          },
        });

        console.log(`Success: ${fileName} → ${newFilename}`);

        await pubsub
          .topic(topicName)
          .publishMessage({ data: Buffer.from(newFilename) });

        console.log(`Message published to ${topicName}`);
      } catch (err) {
        console.error(`Error: ${err}`);
      }
    } else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```

//Add the following code to the `package.json`:

```jsx
{
 "name": "thumbnails",
 "version": "1.0.0",
 "description": "Create Thumbnail of uploaded image",
 "scripts": {
   "start": "node index.js"
 },
 "dependencies": {
   "@google-cloud/functions-framework": "^3.0.0",
   "@google-cloud/pubsub": "^2.0.0",
   "@google-cloud/storage": "^6.11.0",
   "sharp": "^0.32.1"
 },
 "devDependencies": {},
 "engines": {
   "node": ">=4.3.2"
 }
}

```

//Test the function setting only region  for cloud run, cloud run no need zone by default

```jsx
gcloud config set run/region $REGION
```

//Creating and entering to the directory file

```jsx
mkdir -p ~/memories-function
```

```jsx
cd ~/memories-function
```

//Opening the  file index.js with nano

```jsx
nano index.js
```

//Pasting the code for the action and I fixed the cloud Event

*Add the code provided on the exercise to the `index.js`:*

*Closing the nano with CTRL + X and Y for save the file*

```jsx
const functions = require('@google-cloud/functions-framework');
const { Storage } = require('@google-cloud/storage');
const { PubSub } = require('@google-cloud/pubsub');
const sharp = require('sharp');

functions.cloudEvent('memories-thumbnail-creator', async cloudEvent => {
  const event = cloudEvent.data;

  console.log(`Event: ${JSON.stringify(event)}`);
  console.log(`Hello ${event.bucket}`);

  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = new Storage().bucket(bucketName);
  const topicName = "topic-memories-549";
  const pubsub = new PubSub();

  if (fileName.search("64x64_thumbnail") === -1) {
    // doesn't have a thumbnail, get the filename extension
    const filename_split = fileName.split('.');
    const filename_ext = filename_split[filename_split.length - 1].toLowerCase();
    const filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length - 1); // fix sub string to remove the dot

    if (filename_ext === 'png' || filename_ext === 'jpg' || filename_ext === 'jpeg') {
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      const newFilename = `${filename_without_ext}_64x64_thumbnail.${filename_ext}`;
      const gcsNewObject = bucket.file(newFilename);

      try {
        const [buffer] = await gcsObject.download();
        const resizedBuffer = await sharp(buffer)
          .resize(64, 64, {
            fit: 'inside',
            withoutEnlargement: true,
          })
          .toFormat(filename_ext)
          .toBuffer();

        await gcsNewObject.save(resizedBuffer, {
          metadata: {
            contentType: `image/${filename_ext}`,
          },
        });

        console.log(`Success: ${fileName} → ${newFilename}`);

        await pubsub
          .topic(topicName)
          .publishMessage({ data: Buffer.from(newFilename) });

        console.log(`Message published to ${topicName}`);
      } catch (err) {
        console.error(`Error: ${err}`);
      }
    } else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```

//The same procedure for the package.json file

 *CTRL + X and then Y to save*

```jsx
nano package.json
```

```jsx
{
 "name": "thumbnails",
 "version": "1.0.0",
 "description": "Create Thumbnail of uploaded image",
 "scripts": {
   "start": "node index.js"
 },
 "dependencies": {
   "@google-cloud/functions-framework": "^3.0.0",
   "@google-cloud/pubsub": "^2.0.0",
   "@google-cloud/storage": "^6.11.0",
   "sharp": "^0.32.1"
 },
 "devDependencies": {},
 "engines": {
   "node": ">=4.3.2"
 }
}

```

//Installing the dependecies

```jsx
npm install
```

//Giving IAM permissions

```jsx
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:service-890041366730@gs-project-accounts.iam.gserviceaccount.com" \
    --role="roles/pubsub.publisher"
```

//Creating the function

Create a Cloud Run Function (2nd generation) called `Cloud Run Function Name` using `Node.js 22`.

```jsx
gcloud functions deploy memories-thumbnail-creator \
	--gen2 \
	--runtime=nodejs22 \
	--region=$REGION \
	--entry-point=memories-thumbnail-creator \
	--trigger-resource=$BUCKET_NAME \
  --trigger-event=google.storage.object.finalize \
  --allow-unauthenticated

```

```jsx
gcloud functions list 
```

//Testing: uploading the image

Upload a PNG or JPG image of your choice to the `Bucket Name` bucket

```jsx
gsutil cp https://storage.googleapis.com/cloud-training/gsp315/map.jpg gs://$BUCKET_NAME/map.jpg
```

//Testing the action via CONSOLE

1. Cloud storage > buckets > select the bucket and then > upload and upload an image
2. After a couple of minutes refresh the page and it should appear the new image thumbnail

//Testing the action via CLI

```jsx
gsutil cp map.jpg gs://$BUCKET_NAME
```

```jsx
gcloud pubsub topics publish thumbailFunction --message="thumbailFunction"
```

</aside>

<aside>

### **Task 3. Remove the previous cloud engineer**

My solution with CLI

//Deleting the permissions from viewer (username 2)

```jsx
gcloud projects remove-iam-policy-binding $PROJECT_ID \
	--member=$VIEWER \
	--role="roles/viewer"
```

My solution with the console

1.  I have to verify I’m from username1 account
2. Then, Navigation menu > IAM & Admin > IAM
3. Click on the pencil next to username2
4. Click on trashcan and save
5. Wait a few seconds and click on “check assignment”

</aside>
<hr style="border:0;height:1px;background:#eee;" />

<a id="develop-network"></a>
### 🌐 Develop your Google Cloud Network: Challenge Lab
<aside>

### **Task 1. Create development VPC manually**

//First set the variables 

```jsx
export REGION=us-west1
export ZONE=us-west1-a
export USER_TWO=student-01-85ad1af33bb6@qwiklabs.net
```

```jsx
echo $REGION
echo $ZONE
echo $USER_TWO
```

//Setting the region and zone 

```jsx
gcloud config set compute/region REGION
gcloud config set compute/zone ZONE
```

//Creating net VPC DEV

```jsx
gcloud compute networks create griffin-dev-vpc --subnet-mode=custom
```

//Creating subnet 1

```jsx

gcloud compute networks subnets create griffin-dev-wp \
    --network=griffin-dev-vpc \
    --range=192.168.16.0/20 \
    --region=$REGION
```

//Creating subnet 2

```jsx
gcloud compute networks subnets create griffin-dev-mgmt \
    --network=griffin-dev-vpc \
    --range=192.168.32.0/20 \
    --region=$REGION
```

</aside>

<aside>

### **Task 2. Create production VPC manually**

//Creating  VPC PROD

```jsx
gcloud compute networks create griffin-prod-vpc --subnet-mode=custom
```

//Creating subnet 1

```jsx
gcloud compute networks subnets create griffin-prod-wp \
    --network=griffin-prod-vpc \
    --range=192.168.48.0/20 \
    --region=$REGION
```

//Creating second subnet

```jsx
gcloud compute networks subnets create griffin-prod-mgmt \
    --network=griffin-prod-vpc \
    --range=192.168.64.0/20 \
    --region=$REGION
```

//Checking if they are ok on CS

```jsx
gcloud compute networks list
```

</aside>

<aside>

### **Task 3. Create bastion host**

//Getting the ip public

```jsx
export IP_PUBLIC=$(curl -s ifconfig.me)
```

//Creating the VM instance

```jsx
gcloud compute instances create griffin-bastion \
	--tags=allow-ssh \
  --zone=$ZONE \
  --machine-type=e2-medium \
  --network-interface subnet=griffin-dev-mgmt \
  --network-interface subnet=griffin-prod-mgmt
```

//Creating both firewall rules for every network

*SSH se accede con el puerto 22 **nota*

```jsx
gcloud compute firewall-rules create griffin-prod-allow-ssh \
--network=griffin-prod-vpc \
--allow=tcp:22 \
--source-ranges=0.0.0.0/0 \
--target-tags=allow-ssh \
--description="Allow SSH traffic to griffin-prod-vpc"
```

```jsx
gcloud compute firewall-rules create griffin-dev-allow-ssh \
--network=griffin-dev-vpc \
--allow=tcp:22 \
--source-ranges=0.0.0.0/0 \
--target-tags=allow-ssh \
--description="Allow SSH traffic to griffin-prod-vpc"
```

</aside>

<aside>

### **Task 4. Create and configure Cloud SQL Instance**

*Create a **MySQL Cloud SQL Instance** called `griffin-dev-db` in `REGION`.*

*Connect to the instance and run the following SQL commands to prepare the **WordPress** environment:*

*These SQL statements create the worpdress database and create a user with access to the wordpress database.*

*You will use the username and password in task 6.*

```jsx
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

//Creating the mysql warehouse instance Cloud SQL

```jsx
gcloud sql instances create griffin-dev-db \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-2 \
  --zone=$ZONE \
  --root-password=stormwind_rules \
  --edition=ENTERPRISE \
  
  ?
  --database-flags=character_set_server=utf8mb4,collation_server=utf8mb4_unicode_ci
```

//Creating WordPress data base

```jsx
gcloud sql databases create wordpress --instance=griffin-dev-db
```

//Creating wp_user with password 

```jsx

gcloud sql users create wp_user \
  --instance=griffin-dev-db \
  --password=stormwind_rules \
  --host=%
```

//Connecting the instance - not sure if this step is necesary- !!! creo q hay q borrarlo

*Wait a few minutes and then > Click on SQL instance overview*

```jsx
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

</aside>

<aside>

### **Task 5. Create Kubernetes cluster**

//Create a 2 node cluster (e2-standard-4) called `griffin-dev`, in the `griffin-dev-wp` subnet, and in zone `ZONE`.

```jsx
gcloud container clusters create griffin-dev  \
	--num-nodes 2  \
	--machine-type e2-standard-4  \
	--network griffin-dev-vpc \
	--subnetwork griffin-dev-wp \
	--zone=$ZONE
```

</aside>

<aside>

### **Task 6. Prepare the Kubernetes cluster**

*Use the command below to create the key, and then add the key to the Kubernetes environment:*

```jsx
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

//Getting credential clusters

```jsx
gcloud container clusters get-credentials griffin-dev --zone=$ZONE
```

//Downloading the file and entering 

```jsx
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```

```jsx
cd wp-k8s
```

//Checking

```jsx
echo -n "wp_user" | base64
echo -n "stormwind_rules" | base64
```

//Going inside the .yaml file

```jsx
nano wp-env.yaml
```

//Updating the username and password

```jsx
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  username: wp_user
  password: stormwind_rules
```

```jsx
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  username: d3BfdXNlcg==  # base64 encoded "wp_user"
  password: c3Rvcm13aW5kX3J1bGVz  # base64 encoded "stormwind_rules"
```

//Applying the changes

```jsx
kubectl apply -f wp-env.yaml
```

//Creating the keys and credentials for the username

```jsx
gcloud iam service-accounts keys create key.json \
  --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

```jsx
kubectl create secret generic cloudsql-instance-credentials \
  --from-file key.json
```

</aside>

<aside>

### **Task 7. Create a WordPress deployment**

//Creating the sql instance

```jsx
gcloud sql instances describe griffin-dev-db --format="value(connectionName)”
```

//Taking the name the previous command gave 

*Hay que esperar un par de minutos extra para que ejecutar y que funcione este comando*

```jsx
qwiklabs-gcp-04-befbc2a37d18:us-west1:griffin-dev-db
```

//Entering into the file and then into the deployment.yaml

*Make sure you're in the directory with the files*

```jsx
cd wp-k8s  
```

//Entering into the file and then into the deployment.yaml

*Into the .yaml*

```jsx
nano wp-deployment.yaml
```

//Looking for the  line with YOUR_SQL_INSTANCE  and replacing it with 

```jsx
qwiklabs-gcp-04-befbc2a37d18:us-west1:griffin-dev-db
```

//Applying those changes deployment

```jsx
kubectl apply -f wp-deployment.yaml
```

//Applying those changes service

```jsx
kubectl apply -f wp-service.yaml
```

//Getting the services from wordpress

```jsx
kubectl get service wordpress
```

//Getting the credentials

```jsx
gcloud container clusters get-credentials griffin-dev --zone=$ZONE
```

</aside>

<aside>

### **Task 8.  Enable monitoring C**reate an uptime check for your WordPress development site.

//Obtaining the external IP and assigned into a variable

*Exports the external IP address of your WordPress service ***

```jsx
kubectl get service wordpress
```

```jsx
export GRIFFIN_EXTERNAL_IP=$(kubectl get service wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

```jsx
echo $GRIFFIN-EXTERNAL-IP
```

//Creating the uptime checked 

```jsx
gcloud monitoring uptime create griffin-uptime-check \
  --path=/ \
  --resource-type=uptime-url \
  --regions=usa-oregon,usa-virginia,usa-iowa \
  --port=80 \
  --protocol=http \
  --timeout=30 \
  --resource-labels=host=$GRIFFIN_EXTERNAL_IP
```

</aside>

<aside>

### **Task 9. Provide access for an additional engineer**

//Giving them via cloud shell

```jsx
gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
	--member=user:$USER_TWO \
	--role=roles/editor
```

</aside>
<hr style="border:0;height:1px;background:#eee;" />

<a id="terraform-infra"></a>
### 🏗️ Build Infrastructure with Terraform on Google Cloud: Challenge Lab
<aside>

### **Task 1. Create the configuration files**

*In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:*

```jsx
main.tf
variables.tf
modules/
└── instances
    ├── instances.tf
    ├── outputs.tf
    └── variables.tf
└── storage
    ├── storage.tf
    ├── outputs.tf
    └── variables.tf
```

//Creating the variables

*On the cloud shell execute*

*Instance  = to the third instance the should be created at step 4*

```jsx
export REGION=us-east1
export ZONE=us-east1-d
export PROJECT_ID=qwiklabs-gcp-01-a4a76bf2a9b1
export BUCKET=tf-bucket-863479
export INSTANCE=tf-instance-843913
export VPC=tf-vpc-243023
export INSTANCE_ID_1=tf-instance-1
export INSTANCE_ID_2=tf-instance-2
```

//Creating the folders and files

```jsx
cat > modules/instances/instances.tf <<EOF_END
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"
  zone         =  "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
	  network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
EOF_END
```

```jsx
touch main.tf variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf outputs.tf variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf outputs.tf variables.tf
cd
```

//Filling the variables .tf [](http://variables.tf)ROOT 

```jsx
variable "region" {
	type        = string
	default     = "$REGION"
	description = "challenge region"
}

variable "zone" {
	type        = string
	default     = "$ZONE"
	description = "challenge zone"
}

variable "project_id" {
	type        = string
	default     = "$PROJECT_ID"
	description = "challenge id"
}
```

//Filling the main .tf [](http://variables.tf)ROOT 

```jsx
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  region     = var.region
  zone       = var.zone
  project    = var.project_id 
}

module "instances" {
  source     = "./modules/instances"
}
```

//Init the terraform document

```jsx
terraform init 
```

</aside>

<aside>

### **Task 2. Import infrastructure**

//Looking and coping the instances ID’sm boot disk, machine type

```jsx
tf-instance-1 y 2
instance id -> 4500354172801803476
boot disk image -> e2-micro
machine type -> debian-11-bullseye-v20250311
e2-micro"
```

//Going to the instance -tf file and pasting the resources for Instances  1 and 2

```jsx
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }

  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"
  zone         = "$ZONE"
  
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }

 metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

```

//Importing module instance 1

*Important note: change the INSTANCE ID for the instance id*

```jsx
terraform import module.instances.google_compute_instance.tf-instance-1 $PROJECT_ID/$ZONE/$INSTANCE_ID_1
```

//Importing module instance 2

*Important note: change the INSTANCE ID for the instance id*

```jsx
terraform import module.instances.google_compute_instance.tf-instance-2 $PROJECT_ID/$ZONE/$INSTANCE_ID_2
```

//Applying changes

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

</aside>

<aside>

### **Task 3. Configure a remote backend**

//Going to the storage module then storage .tf then adding the code

*Important note: change the bucket name for the one the lab would give me once the time starts*

  *project = var.project_id*

```jsx
resource "google_storage_bucket" "storage-bucket" {
  name          = "tf-bucket-016463"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```

//Going to main .tf and call the module

```jsx
  module "storage" {
  source     = "./modules/storage"
}
```

//Applying

```jsx
terraform init
```

```jsx
terraform apply -auto-approve
```

//Going to main .tf againg to writte the set de bucket

****THIS CODE MUST BE AT THE TOP OF THE FILE*

```jsx
terraform {
  backend "gcs" {
    bucket = "tf-bucket-016463" 
    prefix = "terraform/state"
  }
  ...
```

//Init

```jsx
terraform init
```

</aside>

<aside>

### **Task 4. Modify and update infrastructure**

//Changing the machine type for the Instance -1  and 2 for:

```jsx
e2-standard-2
```

//Adding a new instance resource within the file instances .tf

*Wait for the name the challenge have to given* 

```jsx
...
  machine_type = "e2-standard-2"
...
```

//Adding a new instance resource within instances .tf

```jsx
resource "google_compute_instance" "$INSTANCE" {
  name         = "$INSTANCE"
  machine_type = "e2-standard-2"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

//Applying those changes

```jsx
terraform init
terraform apply -auto-approve
```

</aside>

<aside>

### **Task 5. Destroy resources**

//Delete the third INSTANCE created on the file instance

//Applying those changes

```jsx
terraform apply -auto-approve
```

</aside>

<aside>

### **Task 6. Use a module from the Registry**

//Goin to main .tf and paste this

*Network name will we the one provided by the challenge:* 

```jsx
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "$PROJECT_ID"
    network_name = "$VPC"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "$REGION"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "$REGION"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Hola"
        }
    ]
```

//Applying changes

```jsx
terraform init
```

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

//Going to instances .tf 

*Changing only the network_interface part*

```jsx
resource "google_compute_instance" "tf-instance-1" {
  ...
    
  network_interface {
    network    = "$VPC"
    subnetwork = "subnet-01"
  }
  
   network_interface {
    network    = "$VPC"
    subnetwork = "subnet-02"
  }
  
 ...
   
}
```

//Applying changes

```jsx
terraform init
```

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

</aside>

<aside>

### **Task 7. Configure a firewall**

//Going to main .tf to create a firewall tule

```jsx
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
  network = "projects/$PROJECT_ID/global/networks/$VPC"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web"]

}
```

//Applying changes

```jsx
terraform init
```

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

</aside>
<hr style="border:0;height:1px;background:#eee;" />

<a id="prompt-design"></a>
### ✨ Prompt Design in Vertex AI: Challenge Lab
Coming soon
<hr style="border:0;height:1px;background:#eee;" />

<a id="cloud-run-functions"></a>
### ⚡ Cloud Run Functions: 3 Ways Skill Badge: Challenge Lab: Introductory

https://www.cloudskillsboost.google/course_templates/696/labs/562153

*All the instructions given are meant to be executed on the Cloud shell.*

*After you enter the platform open the shell and always check you’re under the right project.* 

**Task 1. Create a Cloud Storage bucket**

//Setting global variables 

```jsx
export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-east4
gcloud config set compute/region $REGION
export BUCKET=gs://qwiklabs-gcp-03-8ba5fc35fd0d
export FUNCTION_NAME=cs-monitor
export HTTP_FUNCTION=http-messenger
```

//Enable the Apis

```jsx
gcloud services enable \
  artifactregistry.googleapis.com \
  cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com \
  eventarc.googleapis.com \
  run.googleapis.com \
  logging.googleapis.com \
  pubsub.googleapis.com
```

//Create the bucket

```jsx
gcloud storage buckets create $BUCKET --location=$REGION
```

**Task 2. Create, deploy, and test a Cloud Storage function**
//Create the folder and files on the cloud shell

```jsx
mkdir ~/$FUNCTION_NAME && cd $_
touch index.js && touch package.json
```

//Paste the code provided within the index.js file 

```jsx
cat > index.js << EOF
const functions = require('@google-cloud/functions-framework');

functions.cloudEvent('cs-monitor', (cloudevent) => {
  console.log('A new event in your Cloud Storage bucket has been logged!');
  console.log(cloudevent);
});
EOF
```

//Paste the code provided within the package.json file 

```jsx
cat > package.json << EOF
{
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^2.0.0"
  }
}
EOF
```

//Give the permissions

```jsx
PROJECT_NUMBER=$(gcloud projects list --filter="project_id:$PROJECT_ID" --format='value(project_number)')
SERVICE_ACCOUNT=$(gsutil kms serviceaccount -p $PROJECT_NUMBER)
```

```jsx
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member serviceAccount:$SERVICE_ACCOUNT \
  --role roles/pubsub.publisher
```

//Deploy

```jsx
gcloud functions deploy $FUNCTION_NAME \
  --gen2 \
  --runtime nodejs20 \
  --entry-point $FUNCTION_NAME \
  --source . \
  --region $REGION \
  --trigger-bucket $BUCKET \
  --trigger-location $REGION \
  --max-instances 2 
```

//Test uploading a simple file 

```jsx
echo "test" > testFile.txt
```

```jsx
gcloud storage cp testFile.txt $BUCKET
```

**Task 3. Create and deploy a HTTP function with minimum instances**

//Go back to the root of the route

```jsx
cd ..
```

//Again, create another folder with to the two necessary files: index.js and package.json 

```jsx
mkdir ~/$HTTP_FUNCTION && cd $_
touch index.js && touch package.json
```

//Paste the code provided within the index.js file 

```jsx
 cat > index.js << EOF
  const functions = require('@google-cloud/functions-framework');
	
			functions.http('http-messenger', (req, res) => {
			  res.status(200).send('HTTP function (2nd gen) has been called!');
			});
	 EOF
```

//Paste the code provided within the package.json file 

```jsx
  cat > package.json << EOF
 {
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^2.0.0"
  }
}
EOF
```

//Deploy

```jsx
gcloud functions deploy $HTTP_FUNCTION \
	  --gen2 \
	  --runtime nodejs20 \
	  --entry-point $HTTP_FUNCTION \
	  --source . \
	  --region $REGION \
	  --trigger-http \
	  --timeout 600s \
	  --min-instances 1 \
	  --max-instances 2
```

//Test

```jsx
 gcloud functions call $HTTP_FUNCTION \
  --gen2 --region $REGION
```

</aside>
<hr style="border:0;height:1px;background:#eee;" />
