Enable the API's

gcloud services enable compute.googleapis.com sourcerepo.googleapis.com cloudbuild.googleapis.com storage-api.googleapis.com

Create the source repo

gcloud source repos create packer-image-build

Clone the REPO FROM CLOUD SHELL

gcloud source repos clone packer-image-build && cd packer-image-build

Clone this repo

git clone https://github.com/davidmitchell2019/packer-pipeline-cloud-build-source-repos.git

Edit the packer.json file and add your project id

Build the docker image via cloudbuild

gcloud builds submit --config cloudbuild.yaml .

Give cloud build editor access
 
CLOUD_BUILD_SA=$(gcloud projects get-iam-policy $GOOGLE_CLOUD_PROJECT --flatten=bindings --filter="bindings.role:roles/cloudbuild.builds.builder" --format="value(bindings.members[])")
 
Grant access to your project
 
 gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member $CLOUD_BUILD_SA \
    --role roles/editor
    
 
 Push the two files to the REPO
 
 git config --global user.email "you@example.com"
 git config --global user.name "Your Name"
 
 git add -A
 git commit -a -m "Initial commit"
 git push
 
GO IN TO CLOUD BUILD AND CHECK YOUR IMAGE BUILDING
USE YOUR IMAGE ID OUTPUT TO CREATE VM's


