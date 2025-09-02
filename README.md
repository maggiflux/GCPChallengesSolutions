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
RESPUESTA
<hr style="border:0;height:1px;background:#eee;" />

<a id="terraform-infra"></a>
### 🏗️ Build Infrastructure with Terraform on Google Cloud: Challenge Lab
RESPUESTA
<hr style="border:0;height:1px;background:#eee;" />

<a id="prompt-design"></a>
### ✨ Prompt Design in Vertex AI: Challenge Lab
RESPUESTA
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
