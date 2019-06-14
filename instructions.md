Enable the API's

gcloud services enable compute.googleapis.com sourcerepo.googleapis.com cloudbuild.googleapis.com storage-api.googleapis.com

Create the source repo

gcloud source repos create packer-image-build

Clone the REPO FROM CLOUD SHELL

gcloud source repos clone packer-image-build && cd packer-image-build

Create Dockerfile for packer

cat > Dockerfile <<EOF

FROM alpine:3.9

ARG PACKER_VERSION=1.3.1
ARG PACKER_VERSION_SHA256SUM=254cf648a638f7ebd37dc1b334abe940da30b30ac3465b6e0a9ad59829932fa3

COPY packer_${PACKER_VERSION}_linux_amd64.zip .
RUN echo "${PACKER_VERSION_SHA256SUM}  packer_${PACKER_VERSION}_linux_amd64.zip" > checksum && sha256sum -c checksum

RUN /usr/bin/unzip packer_${PACKER_VERSION}_linux_amd64.zip


FROM ubuntu
RUN apt-get -y update && apt-get -y install ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=0 packer /usr/bin/packer
ENTRYPOINT ["/usr/bin/packer"]

EOF

Build the docker image via cloudbuild

gcloud builds submit --config cloudbuild.yaml .

Create the JSON file for PACKER

cat > packer.json <<EOF

{
  "variables": {
    "source_image_family": "debian-9",
    "machine_type": "f1-micro",
    "region": "europe-west2",
    "zone": "europe-west2-b",
    "project_id": "gra-jupyter-vm" #your project ID
  },
  "builders": [
    {
      "type": "googlecompute",
      "project_id": "{{user  `project_id`}}",
      "machine_type": "{{user  `machine_type`}}",
      "source_image_family": "{{user  `source_image_family`}}",
      "region": "{{user  `region`}}",
      "zone": "{{user  `zone`}}",
      "image_description": "Debian9-{{isotime \"20060102\"}}-{{timestamp}}",
      "image_name": "jupytr-{{isotime \"20060102\"}}-{{timestamp}}",
      "disk_size": 16,
      "disk_type": "pd-ssd",
      "ssh_username": "cloud-user"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "uname -a >> ~/sysinfo.txt"
      ]
    }
  ]
}

Create cloudbuild.yaml file with pipeline steps

cat > cloudbuild.yaml <<EOF

steps:
- name: 'gcr.io/$PROJECT_ID/packer'
  args:
  - build
  - packer.json
 
 EOF
 
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


