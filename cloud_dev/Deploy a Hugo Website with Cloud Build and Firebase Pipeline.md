# Deploy a Hugo Website with Cloud Build and Firebase Pipeline

## Create a VM

Install the CLI and run `gcloud auth login`.

### VM in Terraform code
```
# This code is compatible with Terraform 4.25.0 and versions that are backwards compatible to 4.25.0.
# For information about validating this Terraform code, see https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/google-cloud-platform-build#format-and-validate-the-configuration

resource "google_compute_instance" "hugo-dev-vm-20241022-090138" {
  boot_disk {
    auto_delete = true
    device_name = "hugo-dev-vm"

    initialize_params {
      image = "projects/debian-cloud/global/images/debian-11-bullseye-v20241009"
      size  = 100
      type  = "pd-standard"
    }

    mode = "READ_WRITE"
  }

  can_ip_forward      = false
  deletion_protection = false
  enable_display      = false

  labels = {
    goog-ec-src                = "vm_add-tf"
    goog-terraform-provisioned = "true"
  }

  machine_type = "e2-micro"

  metadata = {
    foo            = "bar"
    startup-script = "#!/bin/sh\n# Mandatory Prefix\necho \"STARTUP-SCRIPT START\"\nsudo apt-get install -y git\n#\n# Install firebase binary into /tmp\n#\nwget --quiet -O /tmp/firebase https://firebase.tools/bin/linux/latest\nchmod 755 /tmp/firebase\n#\n# Create hugo installation script in /tmp/installhugo.sh\n#\nPROJECT_ID=$(curl -s -H \"Metadata-Flavor: Google\" http://metadata.google.internal/computeMetadata/v1/project/project-id)\n\ngsutil cp gs://$${PROJECT_ID}-bucket/installhugo.sh /tmp/installhugo.sh\ngsutil cp gs://$${PROJECT_ID}-bucket/cloudbuild.yaml /tmp/cloudbuild.yaml\ngsutil cp gs://$${PROJECT_ID}-bucket/trigger.yaml /tmp/trigger.yaml\nchmod 755 /tmp/installhugo.sh\n# \n# Replace the string DOLLAR with a dollar sign (Hex 24).\n# This is done to avoid substitution from Cloud Deployment\n# Manager.\n#\nsed -i -e \"s/DOLLAR/\\\\x24/g\" /tmp/installhugo.sh\nsed -i -e \"s/DOLLAR/\\\\x24/g\" /tmp/cloudbuild.yaml\necho \"STARTUP-SCRIPT END\"\n"
  }

  name = "hugo-dev-vm-20241022-090138"

  network_interface {
    access_config {
      network_tier = "PREMIUM"
    }

    queue_count = 0
    stack_type  = "IPV4_ONLY"
    subnetwork  = "projects/qwiklabs-gcp-03-43a766dea013/regions/us-east4/subnetworks/default"
  }

  scheduling {
    automatic_restart   = true
    on_host_maintenance = "MIGRATE"
    preemptible         = false
    provisioning_model  = "STANDARD"
  }

  service_account {
    email  = "623788499693-compute@developer.gserviceaccount.com"
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
  }

  shielded_instance_config {
    enable_integrity_monitoring = true
    enable_secure_boot          = false
    enable_vtpm                 = true
  }

  tags = ["lab-vm"]
  zone = "us-east4-a"
}
```

### VM in command Line
```
gcloud compute instances create hugo-dev-vm-20241022-090138 \
    --project=qwiklabs-gcp-03-43a766dea013 \
    --zone=us-east4-a \
    --machine-type=e2-micro \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default \
    --metadata=foo=bar,startup-script=\#\!/bin/sh$'\n'\#\ \
Mandatory\ Prefix$'\n'echo\ \"STARTUP-SCRIPT\ START\"$'\n'sudo\ apt-get\ install\ -y\ git$'\n'\#$'\n'\#\ Install\ firebase\ binary\ into\ /tmp$'\n'\#$'\n'wget\ \
    --quiet\ \
-O\ /tmp/firebase\ https://firebase.tools/bin/linux/latest$'\n'chmod\ 755\ /tmp/firebase$'\n'\#$'\n'\#\ Create\ hugo\ installation\ script\ in\ /tmp/installhugo.sh$'\n'\#$'\n'PROJECT_ID=\$\(curl\ -s\ -H\ \"Metadata-Flavor:\ Google\"\ http://metadata.google.internal/computeMetadata/v1/project/project-id\)$'\n'$'\n'gsutil\ cp\ gs://\$\{PROJECT_ID\}-bucket/installhugo.sh\ /tmp/installhugo.sh$'\n'gsutil\ cp\ gs://\$\{PROJECT_ID\}-bucket/cloudbuild.yaml\ /tmp/cloudbuild.yaml$'\n'gsutil\ cp\ gs://\$\{PROJECT_ID\}-bucket/trigger.yaml\ /tmp/trigger.yaml$'\n'chmod\ 755\ /tmp/installhugo.sh$'\n'\#\ $'\n'\#\ Replace\ the\ string\ DOLLAR\ with\ a\ dollar\ sign\ \(Hex\ 24\).$'\n'\#\ This\ is\ done\ to\ avoid\ substitution\ from\ Cloud\ Deployment$'\n'\#\ Manager.$'\n'\#$'\n'sed\ -i\ -e\ \"s/DOLLAR/\\\\x24/g\"\ /tmp/installhugo.sh$'\n'sed\ -i\ -e\ \"s/DOLLAR/\\\\x24/g\"\ /tmp/cloudbuild.yaml$'\n'echo\ \"STARTUP-SCRIPT\ END\"$'\n' \
    --maintenance-policy=MIGRATE \
    --provisioning-model=STANDARD \
    --service-account=623788499693-compute@developer.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --tags=lab-vm \
    --create-disk=auto-delete=yes,boot=yes,device-name=hugo-dev-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20241009,mode=rw,size=100,type=pd-standard \
    --no-shielded-secure-boot \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --labels=goog-terraform-provisioned=true,goog-ec-src=vm_add-gcloud \
    --reservation-affinity=any
```

### installhugo.sh
```
_HUGO_VERSION=0.96.0
curl -L https://github.com/gohugoio/hugo/releases/download/v${_HUGO_VERSION}/hugo_extended_${_HUGO_VERSION}_Linux-64bit.tar.gz | tar -xz -C /tmp/
# echo The Hugo binary is now at /tmp/hugo.
```