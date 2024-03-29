# Oracle WebLogic Operator Tutorial #

### Application Lifecycle Management ###

This tutorial implements the Docker image with the WebLogic domain inside the image deployment. This means all the artefacts including the deployed applications, domain related files are stored within the image. This results new WebLogic Docker image every time when the application modified. In this - widely adopted - approach the image is the packaging unit instead of the Web/Enterprise Application Archive (*war*, *ear*).

For the purpose of this lab we created another image that contains domain and updated version of the application (green title on the main page). This image is available at fra.ocir.io/XXXXXXXXXX (TO BE FIXED)

#### Modify the domain.yaml ####

Go and edit  your domain resource definition (*domain.yaml*) file and add the following entry to the `spec:` part. For example after the `serverStartPolicy: "IF_NEEDED"` line:
```
TO BE DONE - RESTART IS DIFFERENT NOW

spec:
  [ ... ]
  serverStartPolicy: "IF_NEEDED"
  restartVersion: "applicationV2"
  [ ... ]
```

Don't forget the leading spaces to keep the proper indentation.

The `restartVersion` property lets you force the operator to restart servers. It's basically a user-specified string that gets added to new server pods (as a label) so that the operator can tell which servers need to be restarted. If the value is different, then the server pod is old and needs to be restarted. If the value matches, then the server pod has already been restarted.

Each time you want to restart some servers, you need to set `restartVersion` to a different string (the particular value doesn't matter). The operator will notice the new value and restart the affected servers.

Apply the domain resource changes:
```
kubectl apply -f /u01/domain.yaml
```
You can immediately check the status of your servers/pods:
```
$ kubectl get po -n sample-domain1-ns
NAME                             READY     STATUS        RESTARTS   AGE
sample-domain1-admin-server      1/1       Terminating   0          22m
sample-domain1-managed-server1   1/1       Running       0          20m
sample-domain1-managed-server2   1/1       Running       0          21m
sample-domain1-managed-server3   1/1       Running       0          21m
```
The operator now performs a rolling server restart one by one. The first one is the *Admin* server than the *Managed* servers. There is a property called `maxUnavailable` on the domain resource which determines how many of the cluster's servers may be taken out of service at a time when doing a rolling restart. It can be specified at the domain and cluster levels and defaults to 1 (that is, by default, clustered servers are restarted one at a time).

During the rolling restart check your web application periodically. If the responding server already restarted then you have to see the change (green fonts) you made on the application. If the server is not yet restarted then it still serves the old version of the application.

`http://EXTERNAL-IP/opdemo/?dsname=testDatasource`

![](images/update.application/004.check.changes.png)
