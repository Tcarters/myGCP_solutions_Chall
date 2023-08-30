
## Task 1. Creating and managing service accounts

- Create a service account

```bash
    gcloud iam service-accounts create my-sa-123 --display-name "my service account"

```

- Granting roles to service accounts

```bash
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/editor

```


## Task 2. Understanding roles

- Types of roles

> Primitive roles, which include the Owner, Editor, and Viewer roles that existed prior to the introduction of Cloud IAM.
> Predefined roles, which provide granular access for a specific service and are managed by Google Cloud.
> Custom roles, which provide granular access according to a user-specified list of permissions.


## Get the role Metadata


```bash
    gcloud iam roles describe [ROLE_NAME]
```

## View the grantable roles on resources

```bash
    
    gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID

```

## Create a custom role using yaml 

```text
title: [ROLE_TITLE]
description: [ROLE_DESCRIPTION]
stage: [LAUNCH_STAGE]
includedPermissions:
- [PERMISSION_1]
- [PERMISSION_2]

```

- `nano role-definition.yaml`

```yaml
    title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete

```

- Execute

```bash
    gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml

```


## Task 5. Create a custom role using flags

```bash
    gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \
--title "Role Viewer" --description "Custom role description." \
--permissions compute.instances.get,compute.instances.list --stage ALPHA
```

## Task 6. List the custom roles

```bash
    gcloud iam roles list --project $DEVSHELL_PROJECT_ID

    gcloud iam roles list

```

## Task 7. Update a custom role using a YAML file

```bash
    gcloud iam roles describe [ROLE_ID] --project $DEVSHELL_PROJECT_ID > new-role-definition.yaml

```
- cat new-role-definition.yaml by adding two permissions for storage

```yaml
    description: Edit access for App Versions
etag: BwYCSWaWtYE=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
name: projects/qwiklabs-gcp-00-5086d5eaae9e/roles/editor
stage: ALPHA
title: Role Editor

```

- Use command to update

```bash
    gcloud iam roles update editor --project $DEVSHELL_PROJECT_ID \
--file new-role-definition.yaml
```

## Task 8. Update a custom role using flags

```bash
    gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--add-permissions storage.buckets.get,storage.buckets.list

```

## Task 9. Disable a custom role

```bash
    gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--stage DISABLED


```

## Task 10. Delete a custom role

```bash
    gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID
```

## Task 11. Restore a custom role

```bash
    gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
    
```