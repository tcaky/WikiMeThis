
List all projects created before 6 months ago.

`gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime.date('%Y-%m-%d', Z)<='$(date -d '6 months ago' '+%Y-%m-%dT%H:%M:%S.%3NZ')'"`