# Google Cloud Storage - Bucket Lock



## Task 1. Create a new bucket

```bash
    gcloud config set compute/region us-west4
    
    export export BUCKET=$(gcloud config get-value project) 

    ## Create a new bucket
    gsutil mb "gs://$BUCKET"

```

## Task 2. Define a Retention Policy

```bash
    ##  create a Retention Policy for 10 seconds:

    gsutil retention set 10s "gs://$BUCKET"

    ## Verify the Retention Policy for a bucket:
    gsutil retention get "gs://$BUCKET"
 
    ##Now that the bucket has a Retention Policy, add a transaction record object to test it:

    gsutil cp gs://spls/gsp297/dummy_transactions "gs://$BUCKET/"

    ## Review the retention expiration:

    gsutil ls -L "gs://$BUCKET/dummy_transactions"
```

## Task 3. Lock the Retention Policy

```bash
    ## Lock the Retention Policy:
    gsutil retention lock "gs://$BUCKET/"

    ## To view the Retention Policy for a bucket recall the following command:

    gsutil retention get "gs://$Cloud Storage_BUCKET/"

```

## Task 4. Temporary hold

```bash
   ## Set a temporary hold on the transactions object:
    gsutil retention temp set "gs://$BUCKET/dummy_transactions"

    ## Once regulators conclude their audit, the Branch IT Administrator removes the temporary hold. Use the following command to release the hold:
    gsutil retention temp release "gs://$BUCKET/dummy_transactions"

 
```

## Task 5. Event-based holds

```bash
    ## Look at enabling event-based holds for a loan.
    #Enable the default event-based hold for your bucket using the following command:
    gsutil retention event-default set "gs://$BUCKET/"

    ## Add an example loan into the bucket using the following command:

    gsutil cp gs://spls/gsp297/dummy_loan "gs://$BUCKET/"

    ## Verify that the event-based hold is enabled for your newly added loan using the following command:
    gsutil ls -L "gs://$BUCKET/dummy_loan"

    ## When the loan is paid off, the Branch IT Administrator then releases the event-based hold using the following command:
    gsutil retention event release "gs://$BUCKET/dummy_loan"


```

## Task 6. How to remove a Retention Policy

- Delete an empty bucket : `gsutil rb "gs://$BUCKET/"`


-----------------------------------------------------------------------------------END_GAME------------------------------------------------------------------------------------------------------