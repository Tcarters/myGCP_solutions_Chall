
## Task 1. Create a new repository

```bash
    gcloud source repos create REPO_DEMO

```

## Task 2. Clone the new repository into your Cloud Shell session

```bash
    gcloud source repos clone REPO_DEMO
```

## Push to the Cloud Source Repository

```bash
    cd REPO_DEMO
    git config --global user.name "Your Name"
    git config --global user.email "you@example.com"
    git commit -m "First file using Cloud Source Repositories" myfile.txt
    git push origin master

```

## Task 4. Browse files in the Google Cloud Source Repository

```bash
    gcloud source repos list
```