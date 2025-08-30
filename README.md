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
explicación
<hr style="border:0;height:1px;background:#eee;" />
<a id="terraform-infra"></a>
### 🏗️ Build Infrastructure with Terraform on Google Cloud: Challenge Lab
explicación
<hr style="border:0;height:1px;background:#eee;" />
<a id="prompt-design"></a>
### ✨ Prompt Design in Vertex AI: Challenge Lab
explicación
<hr style="border:0;height:1px;background:#eee;" />
<a id="cloud-run-functions"></a>
### ⚡ Cloud Run Functions: 3 Ways: Challenge Lab
explicación
<hr style="border:0;height:1px;background:#eee;" />
