# k8s and related commands


```bash
kpt fn eval --image='gcr.io/kpt-fn/set-labels:v0.2.0' --match-kind='Project' --truncate-output=false -- ssc_cbrid='0000'
kpt fn eval --image='gcr.io/kpt-fn/set-annotations:v0.1.3' --truncate-output=false -- 'cnrm.cloud.google.com/deletion-policy'='abandon'
```



