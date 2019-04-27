# Spinnaker Reference guide

## Set-up Project  

` gcloud auth list `

` gcloud config list project `

` export ZONE=us-central1-f `

` gcloud config set compute/zone $ZONE 
` 

` export PROJECT=$(gcloud info --format='value(config.project)')  `
` export CICD_DEMO_K8S=spinnaker-k8s  `

## Launch GKE 

` gcloud container clusters create $CICD_DEMO_K8S --machine-type=n1-standard-2 --enable-legacy-authorization ` 

## Config IAM 

### Create SA
` gcloud iam service-accounts create  spinnaker-storage-account --display-name spinnaker-storage-account 
`

### Export SA email

` export SPIN_GCS_SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-storage-account" \
    --format='value(email)') 
 `
 
###  Assign Storage Admin access to SA

` gcloud projects add-iam-policy-binding \
    $PROJECT --role roles/storage.admin --member serviceAccount:$SPIN_SA_EMAIL
`

###  Download SA access key
` gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SPIN_SA_EMAIL `

## Helming Spinnaker

### install helm into GKE

```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.5.0-linux-amd64.tar.gz 

tar zxfv helm-v2.5.0-linux-amd64.tar.gz 

cp linux-amd64/helm . 

./helm init 

./helm repo update 

./helm version 
```

### config GCS for spinnaker

``` 
export PROJECT=$(gcloud info --format='value(config.project)') 

export BUCKET=$PROJECT-spinnaker-config 

gsutil mb -c regional -l us-central1 gs://$BUCKET 
```

### config spinnaker

``` scripts
export SPIN_SA_JSON=$(cat spinnaker-sa.json)
cat > spinnaker-config.yaml <<EOF
storageBucket: $BUCKET
gcs:
  enabled: true
  project: $PROJECT
  jsonKey: '$SPIN_SA_JSON'

# Disable minio the default
minio:
  enabled: false

# Configure your Docker registries here
accounts:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SPIN_SA_JSON'
  email: parthi.gnext18@gmail.com
EOF
```

## Deploy the Spinnaker with above config 

```
./helm install -n cd stable/spinnaker -f spinnaker-config.yaml --timeout 600 \
    --version 0.3.1
```



 
