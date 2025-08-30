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
RESPUESTA
<hr style="border:0;height:1px;background:#eee;" />

<a id="load-balancing"></a>
### 🚧 Set Up an App Dev Environment on Google Cloud: Challenge Lab
RESPUESTA
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

//Paste the code provided on the index.js file 

```jsx
cat > index.js << EOF
const functions = require('@google-cloud/functions-framework');

functions.cloudEvent('cs-monitor', (cloudevent) => {
  console.log('A new event in your Cloud Storage bucket has been logged!');
  console.log(cloudevent);
});
EOF
```

//Paste the code provided on the package.json file 

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

//Paste the code provided on the index.js file 

```jsx
 cat > index.js << EOF
  const functions = require('@google-cloud/functions-framework');
	
			functions.http('http-messenger', (req, res) => {
			  res.status(200).send('HTTP function (2nd gen) has been called!');
			});
	 EOF
```

//Paste the code provided on the package.json file 

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
