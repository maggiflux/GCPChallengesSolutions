# ☁️ GCP Challenges Solutions
💡 My approaches to some of the challenges from **Google Cloud Platform** paths and courses.

<img width="800" height="400" alt="google-cloud-banner-2019" src="https://github.com/user-attachments/assets/42a4a53c-3c08-4d80-a034-823dd60b2d1c" />

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
- 🚧 Coming soon - in progress

## 📚 Courses

### ⚡ Cloud Run Functions: 3 Ways
- [Cloud Run Functions: 3 Ways: Challenge Lab](#cloud-run-functions)

<hr style="border:0;height:1px;background:#eee;" />

## 📝 Solutions

<!-- Anchor explícita: garantiza que el #link funcione -->
<a id="load-balancing"></a>
### ⚖️ Implement Load Balancing on Compute Engine: Challenge Lab

**Challenge scenario**

You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:

1. Create all resources in the default region or zone, unless otherwise directed. The default region is `us-west4`, and the default zone is `us-west4-c`.
2. Naming normally uses the format *team-resource*; for example, an instance could be named **nucleus-webserver1**.
3. Make sure to create an instance template in `global` location.
4. Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. This is the guidance the monitoring team is willing to share: unless directed, use **e2-micro** for small Linux VMs, and use **e2-medium** for Windows or other applications, such as Kubernetes nodes.

Your challenge

As soon as you sit down at your desk and open your new laptop, you receive several requests from the Nucleus team. Read through each description, and then create the resources.

</aside>

<aside>

**Task 1. Create a project jumphost instance**

You will use this instance to perform maintenance for the project.

**Requirements:**

- Name the instance **`Instance name`**.
- Create the instance in the `ZONE` zone.
- Use an *e2-micro* machine type.
- Use the default image type (Debian Linux).

## My solution

### 1.- Declaring all the variables

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

### 2.- Checking if region and zone are ok

```jsx
echo $REGION
echo $ZONE
```

### 3.- Configuring global region and zone

```jsx
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
```

### 4.- Creating the instance

Name the instance **`Instance name,`** Create the instance in the `ZONE` zone. Use an *e2-micro* machine type. Use the default image type (Debian Linux)

```jsx
gcloud compute instances create $INSTANCE_NAME \
    --zone=$ZONE \
    --machine-type=e2-micro \
    --image-family=debian-11 \
    --image-project=debian-cloud
```

### 5.- I refresh the page to check if the instance was created

```jsx
gcloud compute instances list --zone=$ZONE
```

```jsx
gcloud compute instances describe $INSTANCE_NAME --zone=$ZONE
```

</aside>

<aside>

**Task 2. Set up an HTTP load balancer**

You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of **2 nginx web servers**. Use the following code to configure the web servers; the team will replace this with their own configuration later.

**Note:** There is a limit to the resources you are allowed to create in your project, so do not create more than 2 instances in your managed instance group. If you do, the lab might end and you might be banned.

```jsx
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

You need to:

- Create an instance template. Don't use the default machine type. Make sure you specify **e2-medium** as the machine type and create the **Global** template.
- Create a managed instance group based on the template.
- Create a firewall rule named as ____ to allow traffic (80/tcp).
- Create a health check.
- Create a backend service and add your instance group as the backend to the backend service group with named port (http:80).
- Create a URL map, and target the HTTP proxy to route the incoming requests to the default backend service.
- Create a target HTTP proxy to route requests to your URL map
- Create a forwarding rule.

## My solution

### 1.-  Setting the script

```jsx
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

### 2.-  Creating the instance template -Here the problems started, it doesn’t accept the flag —global

Create an instance template. Don't use the default machine type. Make sure you specify **e2-medium** as the machine type and create the **Global** template. Make sure to create an instance template in `global` location.

*I deleted the “global” flag, the reason is by default it is global, no need to specify 

```jsx
gcloud compute instance-templates create $TEMPLATE_NAME \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata-from-file=startup-script=startup.sh \
   --tags=allow-health-check
```

Checking the instance template it is ok

```jsx
gcloud compute instance-templates list
```

```jsx
gcloud compute instance-templates describe $TEMPLATE_NAME
```

### 3.- Trying to update the GC version: Google Cloud SDK 513.0.0

```jsx
gcloud version
```

### 4.- Creating the managed instance group

Create a managed instance group based on the template. MIG

```jsx
gcloud compute instance-groups managed create $MIG_NAME \
  --template=$TEMPLATE_NAME \
  --size=2 \
  --zone=$ZONE
```

Set the named ports for the managed instance group

```jsx
gcloud compute instance-groups managed set-named-ports $MIG_NAME \
  --named-ports=http:80 \
  --zone=$ZONE
```

### 5.- Creating the firewall rule

Create a firewall rule named as `Firewall rule` to allow traffic (80/tcp)

```jsx
gcloud compute firewall-rules create $FIREWALL_RULE_NAME \
  --allow=tcp:80 \
  --target-tags=allow-health-check
```

### 6.- The health check

```jsx
gcloud compute health-checks create http $HEALTH_CHECK_NAME \
  --port=80
```

### 7.- The backend service

Create a backend service and add your instance group as the backend to the backend service group with named port (http:80).

```jsx
gcloud compute backend-services create $BACKEND_SERVICE_NAME \
  --protocol=HTTP \
  --health-checks=$HEALTH_CHECK_NAME \
  --global
```

### 8.- The instance group as the backend

```jsx
  gcloud compute backend-services add-backend $BACKEND_SERVICE_NAME \
	  --instance-group=$MIG_NAME \
	  --instance-group-zone=$ZONE \
	  --balancing-mode=UTILIZATION \
	  --max-utilization=0.8 \
	  --global
```

### 9.- The url map

Create a URL map, and target the HTTP proxy to route the incoming requests to the default backend service.

```jsx
gcloud compute url-maps create $URL_MAP_NAME \
  --default-service=$BACKEND_SERVICE_NAME
```

### 10.- The HTTP proxy

Create a target HTTP proxy to route requests to your URL map

```jsx
gcloud compute target-http-proxies create $HTTP_PROXY_NAME \
  --url-map=$URL_MAP_NAME
```

### 11.- The forwarding rule

```jsx
gcloud compute forwarding-rules create $FORWARDING_RULE_NAME \
  --target-http-proxy=$HTTP_PROXY_NAME \
  --ports=80 \
  --global
```

### 12.- Checking if everything goes well

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
### 🛠️ Set Up an App Dev Environment on Google Cloud: Challenge Lab
**Set Up an App Dev Environment on Google Cloud: Challenge Lab**

<aside>

**Challenge scenario**

You are just starting your junior cloud engineer role with Jooli inc. So far you have been helping teams create and manage Google Cloud resources.

You are expected to have the skills and knowledge for these tasks, so don’t expect step-by-step guides.

Your challenge

You are asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called *Memories*. You have been asked to assist the Memories team with initial configuration for their application development environment.

You receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Run Function you create.
- Create a Cloud Run Function.
- Remove the previous cloud engineer’s access from the memories project.

Some Jooli Inc. standards you should follow:

- Create all resources in the `REGION` region and `ZONE` zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally *team-resource*, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **e2-micro** for small Linux VMs and **e2-medium** for Windows or other applications such as Kubernetes nodes.

Each task is described in detail below, good luck!

</aside>

<aside>

**Task 1. Create a bucket**

You need to create a bucket called `Bucket Name` for the storage of the photographs. Ensure the resource is created in the `REGION` region and `ZONE` zone.

## My solution

### 1.-Setting the variable

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

### 2.-Setting global region a zone

```jsx
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
```

### 3.-Create a bucket

```jsx
gcloud storage buckets create gs://$BUCKET_NAME --location=$REGION
```

### 4.-Checking if it was created ok via CLI

```jsx
gcloud storage buckets list
gcloud storage describre $BUCKET_NAME
```

### 4.-Checking if it was created ok via CONSOLE

Cloud storage > buckets > it should appear the bucket called with the variable $BUCKET_NAME = ….

</aside>

<aside>

**Task 2. Create a Pub/Sub topic**

Create a Pub/Sub topic called `Topic Name` for the Cloud Run Function to send messages.

## My solution

### 1.- Creating the topic

```jsx
gcloud pubsub topics create $TOPIC_NAME
```

### 2.- Checking if created ok

```jsx
gcloud pubsub topics list 
```

</aside>

<aside>

**Task 3. Create the thumbnail Cloud Run Function**

Create the function

Create a Cloud Run Function `Cloud Run Function Name` that will to create a thumbnail from an image added to the `Bucket Name` bucket.

Ensure the Cloud Run Function is using the **Cloud Run function** environment (which is 2nd generation). Ensure the resource is created in the `REGION` region and `ZONE` zone.

1. Create a Cloud Run Function (2nd generation) called `Cloud Run Function Name` using `Node.js 22`.

**Note:** The Cloud Run Function is required to execute every time an object is created in the bucket created in Task 1. During the process, Cloud Run Function may request permission to enable APIs or request permission to grant roles to service accounts. Please enable each of the required APIs and grant roles as requested.

1. Make sure you set the **Entry point** (Function to execute) to `Cloud Run Function Name` and **Trigger** to `Cloud Storage`.
2. Add the following code to the `index.js`:

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

4. Add the following code to the `package.json`:

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

**Note:** If you get a permission denied error stating it may take a few minutes before all necessary permissions are propagated to the Service Agent, wait a few minutes and try again. Ensure you have the appropriate roles (Eventarc Service Agent, Eventarc Event Receiver, Service Account Token Creator, and Pub/Sub Publisher) assigned to the correct service accounts.

# Test the function

- Upload a PNG or JPG image of your choice to the `Bucket Name` bucket.

**Note:** Alternatively, download this image `https://storage.googleapis.com/cloud-training/gsp315/map.jpg` to your machine. Then, upload it to the bucket.

You will see a thumbnail image appear shortly afterwards (use **REFRESH** on the bucket details page).

After you upload the image file, you can click to check your progress below. You do not need to wait for the thumbnail image to be created.

**Optional:** If the function deployed successfully and you do not see the thumbnail image in the bucket, you can check that the **Triggers** tab displays the trigger information that you previously provided for the function, which may not have saved correctly if you previously encountered errors. If you do not see the Cloud Storage trigger in the **Triggers** tab of the function, you can recreate the trigger (see the documentation page titled [Create a trigger for services](https://cloud.google.com/run/docs/triggering/trigger-with-events#trigger-services)), and then upload a new file again to test again (refresh the page after adding a new file).

## My solution

### 1.- Setting only region  for cloud run, cloud run no need zone by default

```jsx
gcloud config set run/region $REGION
```

### 2.- Creating and entering to the directory file

```jsx
mkdir -p ~/memories-function
```

```jsx
cd ~/memories-function
```

### 3.-Opening the  file index.js with nano

```jsx
nano index.js
```

### 4.-Pasting the code for the action and I fixed the cloud Event

Add the following code to the `index.js`:

Closing the nano with CTRL + X and Y for save the file

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

### 5.- The same procedure for the package.json file, CTRL + X and then Y to save

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

### 6.- Installing the dependecies

```jsx
npm install
```

### 7.-Giving IAM permissions

```jsx
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:service-890041366730@gs-project-accounts.iam.gserviceaccount.com" \
    --role="roles/pubsub.publisher"
```

### 8.-Creating the function

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

### 9.- Testing: uploading the image

Upload a PNG or JPG image of your choice to the `Bucket Name` bucket

```jsx
gsutil cp https://storage.googleapis.com/cloud-training/gsp315/map.jpg gs://$BUCKET_NAME/map.jpg
```

### 10.- Testing the action via CONSOLE

Cloud storage > buckets > select the bucket and then > upload and upload an image

After a couple of minutes refresh the page and it should appear the new image thumbnail

### 11.- Testing the action via CLI

```jsx
gsutil cp map.jpg gs://$BUCKET_NAME
```

```jsx
gcloud pubsub topics publish thumbailFunction --message="thumbailFunction"
```

</aside>

<aside>

# **Task 3. Remove the previous cloud engineer**

You will see that there are two users defined in the project.

- One is your account (`Username 1` with the role of Owner).
- The other is the previous cloud engineer (`Username 2` with the role of Viewer).
1. Remove the previous cloud engineer’s access from the project.

## My solution with CLI

### 1.- Deleting the permissions from viewer (username 2)

```jsx
gcloud projects remove-iam-policy-binding $PROJECT_ID \
	--member=$VIEWER \
	--role="roles/viewer"
```

## My solution with the console

### 1.- I have to verify I’m from username1 account

### 2.- Then, Navigation menu > IAM & Admin > IAM

### 3.- Click on the pencil next to username2

### 4.- Click on trashcan and save

### 5.- Wait for a few seconds and click on check assignment

</aside>

<hr style="border:0;height:1px;background:#eee;" />

<a id="develop-network"></a>
### 🌐 Develop your Google Cloud Network: Challenge Lab

# **Develop your Google Cloud Network: Challenge Lab**

<aside>

# **Challenge scenario**

As a cloud engineer at Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

You need to complete the following tasks:

- Create a development VPC with three subnets manually
- Create a production VPC with three subnets manually
- Create a bastion that is connected to both VPCs
- Create a development Cloud SQL Instance and connect and prepare the WordPress environment
- Create a Kubernetes cluster in the development VPC for WordPress
- Prepare the Kubernetes cluster for the WordPress environment
- Create a WordPress deployment using the supplied configuration
- Enable monitoring of the cluster
- Provide access for an additional engineer

Some Jooli Inc. standards you should follow:

- Create all resources in the `REGION` region and `ZONE` zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally *team-resource*, e.g. an instance could be named **kraken-webserver1**.
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use `e2-medium`.

# Your challenge

You need to help the team with some of their initial work on a new project. They plan to use WordPress and need you to set up a development environment. Some of the work was already done for you, but other parts require your expert skills.

As soon as you sit down at your desk and open your new laptop you receive the following request to complete these tasks. Good luck!

**Environment**

![UE5MydlafU0QvN7zdaOLo+VxvETvmuPJh+9kZxQnOzE=.png](attachment:49d6d61b-ea18-46ec-91fc-fecce5f0f927:UE5MydlafU0QvN7zdaOLoVxvETvmuPJh9kZxQnOzE.png)

</aside>

<aside>

# **Task 1. Create development VPC manually**

- Create a VPC called `griffin-dev-vpc` with the following subnets only:
    - `griffin-dev-wp`
        - IP address block: `192.168.16.0/20`
    - `griffin-dev-mgmt`
        - IP address block: `192.168.32.0/20`

## My solution

### 1.- First set the variables

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

### 2.-Setting the region and zone

```jsx
gcloud config set compute/region REGION
gcloud config set compute/zone ZONE
```

### 3.- Creating net VPC DEV

```jsx
gcloud compute networks create griffin-dev-vpc --subnet-mode=custom
```

### 4.- Creating subnet 1

```jsx

gcloud compute networks subnets create griffin-dev-wp \
    --network=griffin-dev-vpc \
    --range=192.168.16.0/20 \
    --region=$REGION
```

### 3.- Creating subnet 2

```jsx
gcloud compute networks subnets create griffin-dev-mgmt \
    --network=griffin-dev-vpc \
    --range=192.168.32.0/20 \
    --region=$REGION
```

</aside>

<aside>

# **Task 2. Create production VPC manually**

- Create a VPC called `griffin-prod-vpc` with the following subnets only:
    - `griffin-prod-wp`
        - IP address block: `192.168.48.0/20`
    - `griffin-prod-mgmt`
        - IP address block: `192.168.64.0/20`

## My solution

### 1.- Creating  VPC PROD

```jsx
gcloud compute networks create griffin-prod-vpc --subnet-mode=custom
```

### 3.- Creating subnet 1

```jsx
gcloud compute networks subnets create griffin-prod-wp \
    --network=griffin-prod-vpc \
    --range=192.168.48.0/20 \
    --region=$REGION
```

### 3.- Creating second subnet

```jsx
gcloud compute networks subnets create griffin-prod-mgmt \
    --network=griffin-prod-vpc \
    --range=192.168.64.0/20 \
    --region=$REGION
```

### 2.- Checking if they are ok on CS

```jsx
gcloud compute networks list
```

</aside>

<aside>

# **Task 3. Create bastion host**

- Create a bastion host with two network interfaces, one connected to `griffin-dev-mgmt` and the other connected to `griffin-prod-mgmt`. Make sure you can SSH to the host.

## My solution

### 1.- Getting the ip public

```jsx
export IP_PUBLIC=$(curl -s ifconfig.me)
```

### 2.- Creating the VM instance

```jsx
gcloud compute instances create griffin-bastion \
	--tags=allow-ssh \
  --zone=$ZONE \
  --machine-type=e2-medium \
  --network-interface subnet=griffin-dev-mgmt \
  --network-interface subnet=griffin-prod-mgmt
```

### 2.- Creating both firewall rules for every network

SSH se accede con el puerto 22 **nota

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

# **Task 4. Create and configure Cloud SQL Instance**

1. Create a **MySQL Cloud SQL Instance** called `griffin-dev-db` in `REGION`.
2. Connect to the instance and run the following SQL commands to prepare the **WordPress** environment:

```jsx
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

These SQL statements create the worpdress database and create a user with access to the wordpress database.

You will use the username and password in task 6.

## My solution

### 1.- Creating the mysql warehouse instance Cloud SQL

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

### 2.- Creating WordPress data base

```jsx
gcloud sql databases create wordpress --instance=griffin-dev-db
```

### 3.- Creating wp_user with password

```jsx

gcloud sql users create wp_user \
  --instance=griffin-dev-db \
  --password=stormwind_rules \
  --host=%
```

### 4.- Connecting the instance - not sure if this step is necesary- !!! creo q hay q borrarlo

Wait a few minutes and then > Click on SQL instance overview

```jsx
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

W

</aside>

<aside>

# **Task 5. Create Kubernetes cluster**

- Create a 2 node cluster (e2-standard-4) called `griffin-dev`, in the `griffin-dev-wp` subnet, and in zone `ZONE`.

Click *Check my progress* to verify the objective.

## My solution

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

# **Task 6. Prepare the Kubernetes cluster**

1. From Cloud Shell copy all files from `gs://cloud-training/gsp321/wp-k8s`.

The **WordPress** server needs to access the MySQL database using the *username* and *password* you created in task 4.

1. You do this by setting the values as secrets. **WordPress** also needs to store its working files outside the container, so you need to create a volume.
2. Add the following secrets and volume to the cluster using `wp-env.yaml`.
3. Make sure you configure the *username* to `wp_user` and *password* to `stormwind_rules` before creating the configuration.

You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container.

1. Use the command below to create the key, and then add the key to the Kubernetes environment:

```jsx
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

## My solution

### 1.-Getting credential clusters

```jsx
gcloud container clusters get-credentials griffin-dev --zone=$ZONE
```

### 2.-Downloading the file and entering

```jsx
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```

```jsx
cd wp-k8s
```

### 3.-Checking - not sure

# This will output the base64 encoded username, copy it

# This will output the base64 encoded password, copy it

d3BfdXNlcg==
c3Rvcm13aW5kX3J1bGVz

```jsx
echo -n "wp_user" | base64
echo -n "stormwind_rules" | base64
```

### 4.-Going inside the .yaml file

```jsx
nano wp-env.yaml
```

### 5.-Updating the username and password

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

### 5.-Applying the changes

```jsx
kubectl apply -f wp-env.yaml
```

### 5.-Creating the keys and credentials for the username

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

# **Task 7. Create a WordPress deployment**

Now that you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using `wp-deployment.yaml`.

1. Before you create the deployment you need to edit `wp-deployment.yaml`.
2. Replace **YOUR_SQL_INSTANCE** with griffin-dev-db's **Instance connection name**.
3. Get the **Instance connection name** from your Cloud SQL instance.
4. After you create your WordPress deployment, create the service with `wp-service.yaml`.
5. Once the Load Balancer is created, you can visit the site and ensure you see the **WordPress** site installer.
    
    At this point the dev team will take over and complete the install and you move on to the next task.
    

![u9QONUelkiErVu8MR+euhxVDI0QdUxWqK+mlEdVgpao=.png](attachment:2b121430-7996-4e0e-841e-b677f987fd99:u9QONUelkiErVu8MReuhxVDI0QdUxWqKmlEdVgpao.png)

## My solution

### 1.-Creating the sql instance

```jsx
gcloud sql instances describe griffin-dev-db --format="value(connectionName)”
```

### 2.-Taking the name the previous command gave

Hay que esperar un par de minutos extra para que ejecutar y que funcione este comando

```jsx
qwiklabs-gcp-04-befbc2a37d18:us-west1:griffin-dev-db
```

### 3.-Entering into the file and then into the deployment.yaml

Make sure you're in the directory with the files

```jsx
cd wp-k8s  
```

### 4.-Entering into the file and then into the deployment.yaml

Into the .yaml

```jsx
nano wp-deployment.yaml
```

### 5.-Looking for the  line with YOUR_SQL_INSTANCE  and replacing it with

```jsx
qwiklabs-gcp-04-befbc2a37d18:us-west1:griffin-dev-db
```

### 6.-Applying those changes deployment

```jsx
kubectl apply -f wp-deployment.yaml
```

### 6.-Applying those changes service

```jsx
kubectl apply -f wp-service.yaml
```

### 7.- Getting the services from wordpress

```jsx
kubectl get service wordpress
```

### 8.- Getting the credentials

```jsx
gcloud container clusters get-credentials griffin-dev --zone=$ZONE
```

</aside>

<aside>

# **Task 8.  Enable monitoring**

Create an uptime check for your WordPress development site.

## My solution

### 1.-Obtaining the external IP and assigned into a variable

Exports the external IP address of your WordPress service **ojito

```jsx
kubectl get service wordpress
```

```jsx
export GRIFFIN_EXTERNAL_IP=$(kubectl get service wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

```jsx
echo $GRIFFIN-EXTERNAL-IP
```

### 2.-Creating the uptime checked

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

# **Task 9. Provide access for an additional engineer**

- You have an additional engineer starting and you want to ensure they have access to the project. Grant them the editor role to the project.

The second user account for the lab represents the additional engineer.

## My solution

### 1.-Giving them via cloud shell

```jsx
gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
	--member=user:$USER_TWO \
	--role=roles/editor
```

</aside>

<hr style="border:0;height:1px;background:#eee;" />

<a id="terraform-infra"></a>
### 🏗️ Build Infrastructure with Terraform on Google Cloud: Challenge Lab

contexto: challenge google cloud, respuestas en español- si mis soluciones están bien

[Build Infrastructure with Terraform on Google Cloud: Challenge Lab | GCP lab with Explanation](https://www.youtube.com/watch?v=X71RzZBgW3c&t=554s)

[qwiklabs/gcp-challenge-labs/Build Infrastructure with Terraform on Google Cloud Challenge Lab.md at main · techgalary/qwiklabs](https://github.com/techgalary/qwiklabs/blob/main/gcp-challenge-labs/Build%20Infrastructure%20with%20Terraform%20on%20Google%20Cloud%20Challenge%20Lab.md)

# **Build Infrastructure with Terraform on Google Cloud: Challenge Lab**

<aside>

# **Challenge scenario**

You are a cloud engineer intern for a new startup. For your first project, your new boss has tasked you with creating infrastructure in a quick and efficient manner and generating a mechanism to keep track of it for future reference and changes. You have been directed to use [Terraform](https://www.terraform.io/) to complete the project.

For this project, you will use Terraform to create, deploy, and keep track of infrastructure on the startup's preferred provider, Google Cloud. You will also need to import some mismanaged instances into your configuration and fix them.

In this lab, you will use Terraform to import and create multiple VM instances, a VPC network with two subnetworks, and a firewall rule for the VPC to allow connections between the two instances. You will also create a Cloud Storage bucket to host your remote backend.

**Note:** At the end of every section, `plan` and `apply` your changes to allow your work to be successfully verified. Since we will be updating many terraform files in this lab make sure to use the correct file path and maintain the correct indentation.

# Topics tested:

- Import existing infrastructure into your Terraform configuration.
- Build and reference your own Terraform modules.
- Add a remote backend to your configuration.
- Use and implement a module from the Terraform Registry.
- Re-provision, destroy, and update infrastructure.
- Test connectivity between the resources you've created.

</aside>

<aside>

# **Task 1. Create the configuration files**

1. In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:

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

1. Fill out the `variables.tf` files in the root directory and within the modules. Add three variables to each file: `region`, `zone`, and `project_id`. For their default values, use , `<filled in at lab start>`, and your Google Cloud Project ID.

**Note:** You should use these variables anywhere applicable in your resource configurations.

1. Add the Terraform block and the [Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs) to the `main.tf` file. Verify the **zone** argument is added along with the **project** and **region** arguments in the Google Provider block.
2. Initialize Terraform.

## My solution

### 1.-Creating the variables

On the cloud shell execute

Instance  = to the third instance the should be created at step 4

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

### 2.-Creating the folders and files

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

### 6.-Filling the variables .tf [](http://variables.tf)ROOT

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

### 6.-Filling the main .tf [](http://variables.tf)ROOT

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

### 7.- Init the terraform document

```jsx
terraform init 
```

</aside>

<aside>

# **Task 2. Import infrastructure**

1. In the Google Cloud Console, on the **Navigation menu**, click **Compute Engine > VM Instances**. Two instances named **tf-instance-1** and **tf-instance-2** have already been created for you.

**Note:** by clicking on one of the instances, you can find its **Instance ID**, **boot disk image**, and **machine type**. These are all necessary for writing the configurations correctly and importing them into Terraform.

1. [Import](https://www.terraform.io/docs/cli/commands/import.html#example-import-into-module) the existing instances into the **instances** module. To do this, you will need to follow these steps:
- First, add the module reference into the `main.tf` file then re-initialize Terraform.
- Next, write the [resource configurations](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance) in the `instances.tf` file to match the pre-existing instances.Copied!
    - Name your instances `tf-instance-1` and `tf-instance-2`.
    - For the purposes of this lab, the resource configuration should be as minimal as possible. To accomplish this, you will only need to include the following additional arguments in your configuration: `machine_type`, `boot_disk`, `network_interface`, `metadata_startup_script`, and `allow_stopping_for_update`. For the last two arguments, use the following configuration as this will ensure you won't need to recreate it:
    
    `metadata_startup_script = <<-EOT
            #!/bin/bash
        EOT
    allow_stopping_for_update = true`
    
    content_copy
    
- Once you have written the resource configurations within the module, use the `terraform import` command to import them into your **instances** module.
1. Apply your changes. Note that since you did not fill out all of the arguments in the entire configuration, the `apply` will **update the instances in-place**. This is fine for lab purposes, but in a production environment, you should make sure to fill out all of the arguments correctly before importing.

## My solution

### 1.- Looking and coping the instances ID’sm boot disk, machine type

In the Google Cloud Console, on the **Navigation menu**, click **Compute Engine > VM Instances**. Two instances named **tf-instance-1** and **tf-instance-2** have already been created for you.

**Note:** by clicking on one of the instances, you can find its **Instance ID**, **boot disk image**, and **machine type**. These are all necessary for writing the configurations correctly and importing them into Terraform

```jsx
tf-instance-1 y 2
instance id -> 4500354172801803476
boot disk image -> e2-micro
machine type -> debian-11-bullseye-v20250311
e2-micro"
```

### 2.- Going to the instance -tf file and pasting the resources for Instances  1 and 2

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

### 3.-Importing module instance 1

Important note: change the INSTANCE ID for the instance id

```jsx
terraform import module.instances.google_compute_instance.tf-instance-1 $PROJECT_ID/$ZONE/$INSTANCE_ID_1
```

### 4.-Importing module instance 2

Important note: change the INSTANCE ID for the instance id

```jsx
terraform import module.instances.google_compute_instance.tf-instance-2 $PROJECT_ID/$ZONE/$INSTANCE_ID_2
```

### 7.-Applying changes

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

</aside>

<aside>

# **Task 3. Configure a remote backend**

1. Create a [Cloud Storage bucket resource](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket) inside the `storage` module. For the bucket **name**, use **`Bucket Name`**. For the rest of the arguments, you can simply use:
    - `location = "US"`
    - `force_destroy = true`
    - `uniform_bucket_level_access = true`

**Note:** You can optionally add output values inside of the `outputs.tf` file.

1. Add the module reference to the `main.tf` file. Initialize the module and `apply` the changes to create the bucket using Terraform.
2. Configure this storage bucket as the [remote backend](https://www.terraform.io/docs/language/settings/backends/gcs.html) inside the `main.tf` file. Be sure to use the **prefix** `terraform/state` so it can be graded successfully.
3. If you've written the configuration correctly, upon `init`, Terraform will ask whether you want to copy the existing state data to the new backend. Type `yes` at the prompt.

## My solution

### 1.- Going to the storage module then storage .tf then adding the code

Important note: change the bucket name for the one the lab would give me once the time starts

  project = var.project_id

```jsx
resource "google_storage_bucket" "storage-bucket" {
  name          = "tf-bucket-016463"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```

### 3.-Going to main .tf and call the module

```jsx
  module "storage" {
  source     = "./modules/storage"
}
```

### 2.-Applying

```jsx
terraform init
```

```jsx
terraform apply -auto-approve
```

### 3.-Going to main .tf againg to writte the set de bucket

***THIS CODE MUST BE AT THE TOP OF THE FILE

```jsx
terraform {
  backend "gcs" {
    bucket = "tf-bucket-016463" 
    prefix = "terraform/state"
  }
  ...
```

### 4.-Init

```jsx
terraform init
```

</aside>

<aside>

# **Task 4. Modify and update infrastructure**

1. Navigate to the **instances** module and modify the **tf-instance-1** resource to use an `e2-standard-2` machine type.
2. Modify the **tf-instance-2** resource to use an `e2-standard-2` machine type.
3. Add a third instance resource and name it **`Instance Name`**. For this third resource, use an `e2-standard-2` machine type. Make sure to change the machine type to `e2-standard-2` **to all the three instances**.
4. Initialize Terraform and `apply` your changes.

**Note:** Optionally, you can add output values from these resources in the `outputs.tf` file within the module.

## My solution

### 1.-Changing the machine type for the Instance -1  and 2 for:

```jsx
e2-standard-2
```

### 2.- Adding a new instance resource within the file instances .tf

Wait for the name the challenge have to given 

```jsx
...
  machine_type = "e2-standard-2"
...
```

### 3.- Adding a new instance resource within instances .tf

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

### 4.- Applying those changes

```jsx
terraform init
terraform apply -auto-approve
```

</aside>

<aside>

# **Task 5. Destroy resources**

1. Destroy the third instance **`Instance Name`** by removing the resource from the configuration file. After removing it, initialize terraform and `apply` the changes.

## My solution

### 1.- Delete the third INSTANCE created on the file instance

### 2.- Applying those changes

```jsx
terraform apply -auto-approve
```

</aside>

<aside>

# **Task 6. Use a module from the Registry**

1. In the Terraform Registry, browse to the [Network Module](https://registry.terraform.io/modules/terraform-google-modules/network/google/6.0.0).
2. Add this module to your `main.tf` file. Use the following configurations:
- Use version `6.0.0` (different versions might cause compatibility errors).
- Name the VPC **`VPC Name`**, and use a **global** routing mode.
- Specify **2** subnets in the region, and name them `subnet-01` and `subnet-02`. For the subnets arguments, you just need the **Name**, **IP**, and **Region**.
- Use the IP `10.10.10.0/24` for `subnet-01`, and `10.10.20.0/24` for `subnet-02`.
- You do **not** need any secondary ranges or routes associated with this VPC, so you can omit them from the configuration.
1. Once you've written the module configuration, initialize Terraform and run an `apply` to create the networks.
2. Next, navigate to the `instances.tf` file and update the configuration resources to connect **tf-instance-1** to `subnet-01` and **tf-instance-2** to `subnet-02`.

**Note:** Within the instance configuration, you will need to update the **network** argument to `VPC Name`, and then add the **subnetwork** argument with the correct subnet for each instance.

## My solution

### 1.-Goin to main .tf and paste this

network name will we the one provided by the challenge: 

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

### 2.-Applying changes

```jsx
terraform init
```

```jsx
terraform plan
```

```jsx
terraform apply -auto-approve
```

### 3.-Going to instances .tf

Changing only the network_interface part

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

### 4.-Applying changes

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

# **Task 7. Configure a firewall**

- Create a [firewall rule](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_firewall) resource in the `main.tf` file, and name it **tf-firewall**.
    - This firewall rule should permit the **`VPC Name`** network to allow ingress connections on *all* IP ranges (`0.0.0.0/0`) on **TCP port 80**.
    - Make sure you add the `source_ranges` argument with the correct IP range (`0.0.0.0/0`).
    - Initialize Terraform and `apply` your changes.

**Note:** To retrieve the required `network` argument, you can inspect the state and find the **ID** or **self_link** of the `google_compute_network` resource you created. It will be in the form `projects/PROJECT_ID/global/networks/VPC Name`.

## My solution

### 1.-Going to main .tf to create a firewall tule

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

### 2.-Applying changes

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
explicación
<hr style="border:0;height:1px;background:#eee;" />
<a id="cloud-run-functions"></a>
### ⚡ Cloud Run Functions: 3 Ways: Challenge Lab

<aside>
✨

// TAREA 1 //

//Setear las variables globales

```jsx
export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-east4
gcloud config set compute/region $REGION
export BUCKET=gs://qwiklabs-gcp-03-8ba5fc35fd0d
export FUNCTION_NAME=cs-monitor
export HTTP_FUNCTION=http-messenger
```

//Habilitar las apis

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

//Crear el bucket

```jsx
gcloud storage buckets create $BUCKET --location=$REGION
```

// TAREA 2 //
//Crear la carpeta y archivos

```jsx
mkdir ~/$FUNCTION_NAME && cd $_
touch index.js && touch package.json
```

//Cambiar index.js 

```jsx
cat > index.js << EOF
const functions = require('@google-cloud/functions-framework');

functions.cloudEvent('cs-monitor', (cloudevent) => {
  console.log('A new event in your Cloud Storage bucket has been logged!');
  console.log(cloudevent);
});
EOF
```

//Cambiar package.json

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

//Permisos 

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

//Testear

```jsx
echo "prueba2" > test2-file.txt
```

```jsx
gcloud storage cp test2-file.txt $BUCKET
```

// TAREA 3 //

//Cambiar a ruta raíz

```jsx
cd ..
```

//Crear carpeta y archivos

```jsx
mkdir ~/$HTTP_FUNCTION && cd $_
touch index.js && touch package.json
```

//Cambiar el index.js

```jsx
 cat > index.js << EOF
  const functions = require('@google-cloud/functions-framework');
	
			functions.http('http-messenger', (req, res) => {
			  res.status(200).send('HTTP function (2nd gen) has been called!');
			});
	 EOF
```

//Cambiar el package.json

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

//Deployar

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

//Testear

```jsx
 gcloud functions call $HTTP_FUNCTION \
  --gen2 --region $REGION
```

</aside>

<hr style="border:0;height:1px;background:#eee;" />
