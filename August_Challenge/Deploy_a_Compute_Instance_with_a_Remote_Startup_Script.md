
```bash

    chmod +x resources-install-web.sh 

     gsutil mb gs://apache-bucket



     gsutil cp resources-install-web.sh gs://mybucket-apache

    gsutil acl  ch -u AllUsers:R gs://mybucket-apache/resources-install-web.sh

    gcloud compute instances add-metadata lab-monitor --zone us-east4-c --metadata startup-script-url=gs://mybucket-apache/resources-install-web.sh

    gcloud compute firewall-rules create allow-apache --network=default --allow tcp:80  

```