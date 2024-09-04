
List all projects created before 6 months ago.

```bash
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime.date('%Y-%m-%d', Z)<='$(date -d '6 months ago' '+%Y-%m-%dT%H:%M:%S.%3NZ')'"

# Alternative using ISO 8601 format.
# See: https://en.wikipedia.org/wiki/ISO_8601#Durations
gcloud projects list --format="table(projectNumber,projectId,createTime)" --filter="createTime<=-P6M"

```