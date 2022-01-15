Secutiry Role :orca_storage_creator_573
Service Account: create orca-private-cluster-634-sa
Cluster name: orca-cluster-931
Project ID: qwiklabs-gcp-02-506354f1217b


gcloud config set compute/zone us-east1-b

TASK 1
nano role-definition.yaml

title: "orca_storage_creator_573"
description: "Add and update objects in Google Cloud Storage buckets"
includedPermissions:
- storage.buckets.get
- storage.objects.get
- storage.objects.list
- storage.objects.update
- storage.objects.create

gcloud iam roles create orca_storage_creator_573 \
   --project $DEVSHELL_PROJECT_ID \
   --file role-definition.yaml

TASK 2
gcloud iam service-accounts create orca-private-cluster-634-sa\
   --display-name "orca private cluster"

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
   --member serviceAccount:orca-private-cluster-634-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.viewer

TASK 3
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
   --member serviceAccount:orca-private-cluster-634-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.metricWriter

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
   --member serviceAccount:orca-private-cluster-634-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/logging.logWriter

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
   --member serviceAccount:orca-private-cluster-634-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role projects/$DEVSHELL_PROJECT_ID/roles/orca_storage_creator_573

TASK 4
gcloud container clusters create orca-cluster-931 --network orca-build-vpc --subnetwork orca-build-subnet --service-account orca-private-cluster-634-sa@qwiklabs-gcp-02-506354f1217b.iam.gserviceaccount.com --enable-master-authorized-networks --master-authorized-networks 192.168.10.2/32 --enable-ip-alias --enable-private-nodes --master-ipv4-cidr 10.142.0.0/28 --enable-private-endpoint

TASK 5
Open VM Instance, scroll down (orca-jumphost), ssh, connect

gcloud config set compute/zone us-east1-b

gcloud container clusters get-credentials orca-cluster-931 --internal-ip --zone=us-east1-b

kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0

kubectl expose deployment hello-server --name orca-hello-service \
   --type LoadBalancer --port 80 --target-port 8080
