## IAM Custom Roles

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region <REGION>
```

## Task 1. View the available permissions for a resource
#### Run the following to get the list of permissions available for your project:
```bash
gcloud iam list-testable-permissions 
//cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
```

## Task 2. Get the role metadata
#### To view the role metadata, use command below, replacing <ROLE_NAME> with the role. 
For example: <b>roles/viewer</b> or <b>roles/editor</b>:
```bash
gcloud iam roles describe <ROLE_NAME>
```

## Task 3. View the grantable roles on resources
#### Execute the following <b>gcloud</b> command to list grantable roles from your project:
```bash
gcloud iam list-grantable-roles 
//cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
```

## Task 4. Create a custom role

### 4.1 Create a custom role using a YAML file

#### Create a YAML file that contains the definition for your custom role. The file must be structured in the following way:
``` yaml
title: <ROLE_TITLE>
description: <ROLE_DESCRIPTION>
stage: <LAUNCH_STAGE>
includedPermissions:
- <PERMISSION_1>
- <PERMISSION_2>
```

#### Each of the placeholder values is described below:
- <ROLE_TITLE> is a friendly title for the role, such as Role Viewer.
- <ROLE_DESCRIPTION> is a short description about the role, such as My custom role description.
- <LAUNCH_STAGE> indicates the stage of a role in the launch lifecycle, such as ALPHA, BETA, or GA.
- includedPermissions specifies the list of one or more permissions to include in the custom role, such as iam.roles.get.

#### Time to get started! Create your role definition YAML file by running:
```bash
nano role-definition.yaml
```

#### Add this custom role definition to the YAML file:
``` yaml
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
```

#### Then save and close the file by pressing CTRL+X, Y and then ENTER.
#### Execute the following <b>gcloud</b> command:
```bash
gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID --file role-definition.yaml
```

### 4.2 Create a custom role using flags
#### Execute the following <b>gcloud</b> command to create a new role using flags:
```bash
gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \
--title "Role Viewer" --description "Custom role description." \
--permissions compute.instances.get,compute.instances.list --stage ALPHA
```

## Task 5. List the custom roles
#### Execute the following <b>gcloud</b> command to list custom roles, specifying either project-level or organization-level custom roles:
```bash
gcloud iam roles list --project $DEVSHELL_PROJECT_ID
```

#### Execute the following <b>gcloud</b> command to list predefined roles:
```bash
gcloud iam roles list
```

## Task 6. Update an existing custom role

#### 6.1 Update a custom role using a YAML file

#### Get the current definition for the role by executing the following gcloud command, replacing <ROLE_ID> with editor.
```bash
gcloud iam roles describe <ROLE_ID> --project $DEVSHELL_PROJECT_ID
```
#### The <b>describe</b> command returns the following output:
```yaml 
description: <ROLE_DESCRIPTION>
etag: <ETAG_VALUE>
includedPermissions:
- <PERMISSION_1>
- <PERMISSION_2>
name: <ROLE_ID>
stage: <LAUNCH_STAGE>
title: <ROLE_TITLE>
```
#### Copy the output to use to create a new YAML file in the next steps.
#### Create a <b>new-role-definition.yaml</b> file with your editor:
```bash
nano new-role-definition.yaml
```
#### Paste in the output from the last command and add these two permissions under <b>includedPermissions</b>:
```yaml 
- storage.buckets.get
- storage.buckets.list
```
#### Save and close the file CTRL+X, Y and then ENTER.
#### Now you’ll use the update command to update the role. Execute the following <b>gcloud</b> command, replacing <ROLE_ID> with <b>editor</b>:
```bash
gcloud iam roles update <ROLE_ID> --project $DEVSHELL_PROJECT_ID \
--file new-role-definition.yaml
```

#### 6.2 Update a custom role using flags
#### Each part of a role definition can be updated using a corresponding flag. For a list of all possible flags from the SDK reference documentation, see the topic gcloud iam roles update.

#### Use the following flags to add or remove permissions:

- <b>--add-permissions</b>: Adds one or more comma-separated permissions to the role.
- <b>--remove-permissions</b>: Removes one or more comma-separated permissions from the role.

#### Alternatively, you can simply specify the new permissions using the <b>--permissions</b> <PERMISSIONS> flag and providing a comma-separated list of permissions to replace the existing permissions list.
#### Execute the following <b>gcloud</b> command to add permissions to the <b>viewer</b> role using flags:
```bash
gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--add-permissions storage.buckets.get,storage.buckets.list
```

## Task 7. Disable a custom role
#### The easiest way to disable an existing custom role is to use the <b>--stage</b> flag and set it to DISABLED.
#### Execute the following <b>gcloud</b> command to disable the <b>viewer</b> role:
```bash
gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--stage DISABLED
```

## Task 8. Delete a custom role

#### Use the <b>gcloud iam roles delete</b> command to delete a custom role. Once deleted the role is inactive and cannot be used to create new IAM policy bindings:
```bash
gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID
```
#### After the role has been deleted, existing bindings remain, but are inactive. The role can be undeleted within 7 days. After 7 days, the role enters a permanent deletion process that lasts 30 days. After 37 days, the Role ID is available to be used again.

## Task 9. Restore a custom role

#### Within the 7 days window you can restore a role. Deleted roles are in a DISABLED state. To make it available again, update the <b>--stage</b> flag:
```bash
gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
```

## Congratulations!