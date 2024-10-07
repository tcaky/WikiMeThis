# Useful Gcloud Stuff

## Getting Assigned Ip Addresses For Given Folder (and subfolders)
```bash
FOLDER_ID=1234567890ab

# output to file
gcloud asset search-all-resources --scope=folders/${$FOLDER_ID} --asset-types='compute.googleapis.com/Address' --read-mask='versionedResources,project,parentFullResourceName' --format="csv(versionedResources.resource.addressType,versionedResources.resource.address,parentFullResourceName:label='project_id',versionedResources.resource.selfLink:label='resource_name')" | awk -F, '{gsub(/.*\//, "", $3);gsub(/.*\//, "", $4); print}' OFS=, > ips.csv

# search for a particular IP address (or part of one)
gcloud asset search-all-resources --scope=folders/${$FOLDER_ID} --asset-types='compute.googleapis.com/Address' --read-mask='versionedResources,project,parentFullResourceName' --format="csv(versionedResources.resource.addressType,versionedResources.resource.address,parentFullResourceName:label='project_id',versionedResources.resource.selfLink:label='resource_name')" | awk -F, '{gsub(/.*\//, "", $3);gsub(/.*\//, "", $4); print}' OFS=, | grep '35.203.1'
```
## Getting Assigned Ip Addresses For Entire Organization
```bash
ORG_ID=1234567890ab
gcloud asset search-all-resources --scope=organizations/${$ORG_ID} --asset-types='computer.googleapis.com/Address' --read-mask='*' --format=json
```


# Working with dates in filters
```bash
# https://en.wikipedia.org/wiki/ISO_8601#Durations
# find projects created less than 6 months ago
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime<=-P6M"
# or
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime.date('%Y-%m-%d', Z)<='$(date -d '6 months ago' '+%Y-%m-%dT%H:%M:%S.%3NZ')'"

# find projects from more than 6 months ago 
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime>-P6M"
# or
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime.date('%Y-%m-%d', Z)>'$(date -d '6 months ago' '+%Y-%m-%dT%H:%M:%S.%3NZ')'"
```

# Delete Anthos Config Controller (can only be done from CLI)
```bash
gcloud anthos config controller delete CLUSTER_NAME --location=northamerica-northeast1
```