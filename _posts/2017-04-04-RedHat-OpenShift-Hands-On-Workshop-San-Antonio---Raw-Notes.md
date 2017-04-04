---
title: "RedHat OpenShift Hands On Workshop, San Antonio - Raw Notes"
date: 2017-04-04
layout: "post"
categories: openshift, kubernetes, redhat, docker, cicd, devops
---

RedHat came to town recently to give a one day, almost entirely lab driven workshop around OpenShift. The workshop was well put together, and the live labs were over-all pretty good.

What follows here, are my raw notes from the lab, sanitized of usernames & passwords, and some light editing for things that were pretty ugly.

---
# Begin notes

The parksmap image: docker.io/openshiftroadshow/parksmap:1.2.0

## Check status

```
$ oc get pods
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-a3ppj   1/1       Running   0          33m

$ oc status
In project explore-xx on server https://127.0.0.1:443

svc/parksmap - 172.30.203.33:8080
  dc/parksmap deploys istag/parksmap:1.2.0
    deployment #1 deployed 32 minutes ago - 1 pod

1 warning identified, use 'oc status -v' to see details.

$ oc status -v
In project explore-xx on server https://127.0.0.1:443

svc/parksmap - 172.30.203.33:8080
  dc/parksmap deploys istag/parksmap:1.2.0
    deployment #1 deployed 32 minutes ago - 1 pod

Warnings:
  * Unable to list statefulsets resources.  Not all status relationships can be established.

Info:
  * dc/parksmap has no readiness probe to verify pods are ready to accept traffic or ensure deployment is successful.
    try: oc set probe dc/parksmap --readiness ...
  * dc/parksmap has no liveness probe to verify pods are still running.
    try: oc set probe dc/parksmap --liveness ...

View details with 'oc describe <resource>/<name>' or list everything with 'oc get all'.
```

## Scaling

Get some info about the pod before
```
$ oc get pods
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-a3ppj   1/1       Running   0          33m

$ oc get dc
NAME       REVISION   DESIRED   CURRENT   TRIGGERED BY
parksmap   1          1         1         config,image(parksmap:1.2.0)

$ oc get rc
NAME         DESIRED   CURRENT   READY     AGE
parksmap-1   1         1         1         33m
```

Scale the deployment config
```
$ oc scale --replicas=2 dc/parksmap
deploymentconfig "parksmap" scaled

$ oc get pods
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-a3ppj   1/1       Running   0          34m
parksmap-1-tuj0b   1/1       Running   0          9s
```

Review the new config
```
$ oc describe svc parksmap
Name:           parksmap
Namespace:      explore-xx
Labels:         app=parksmap
Selector:       deploymentconfig=parksmap
Type:           ClusterIP
IP:         172.30.203.33
Port:           8080-tcp    8080/TCP
Endpoints:      10.1.16.37:8080,10.1.20.20:8080
Session Affinity:   None
No events.

$ oc get endpoints
NAME       ENDPOINTS                         AGE
parksmap   10.1.16.37:8080,10.1.20.20:8080   35m
```

Autohealing:
This deletes one of the pods, then watches the new one create:
```
oc delete pod parksmap-1-a3ppj; watch "oc get pods"
```

Scale down:
This sets us back to one replica and the watches the new one terminate.
```
oc scale --replicas=1 dc/parksmap; watch "oc get pods"
```

## Routes

Get routes:
```
$ oc get routes
```

Get the name of our service:
```
$ oc get services
```

Expose it:
```
$ oc expose service parksmap
```

```
$ oc get routes
NAME       HOST/PORT                                                   PATH      SERVICES   PORT       TERMINATION   WILDCARD
parksmap   parksmap-explore-xx.cloudapps.ksat.openshift3roadshow.com             parksmap   8080-tcp                 None
```

## Logs

Get logs:
```
$ oc logs parksmap-1-a3ppj
14:47:51.350 [main] DEBUG io.fabric8.kubernetes.client.Config - Trying to configure client from Kubernetes config...
14:47:51.373 [main] DEBUG io.fabric8.kubernetes.client.Config - Did not find Kubernetes config at: [/.kube/config]. Ignoring.
14:47:51.373 [main] DEBUG io.fabric8.kubernetes.client.Config - Trying to configure client from service account...
14:47:51.374 [main] DEBUG io.fabric8.kubernetes.client.Config - Found service account ca cert at: [/var/run/secrets/kubernetes.io/serviceaccount/ca.crt].
14:47:51.381 [main] DEBUG io.fabric8.kubernetes.client.Config - Found service account token at: [/var/run/secrets/kubernetes.io/serviceaccount/token].
14:47:51.381 [main] DEBUG io.fabric8.kubernetes.client.Config - Trying to configure client namespace from Kubernetes service account namespace path...
14:47:51.381 [main] DEBUG io.fabric8.kubernetes.client.Config - Found service account namespace at: [/var/run/secrets/kubernetes.io/serviceaccount/namespace].
2017-04-04 14:47:53.101  WARN 1 --- [           main] i.f.s.cloud.kubernetes.StandardPodUtils  : Failed to get pod with name:[parksmap-1-a3ppj]. You should look into this if things aren't working as you expect. Are you missing serviceaccount permissions?
```

Also pod, then archive, loads EFK

## RBAC

Fix service account
```
$ oc policy add-role-to-user view -z default
role "view" added: "default"
```

Grant other users access:
```
$ oc policy add-role-to-user view userxx
role "view" added: "userxx"
```

View acceesses:
```
$ oc describe policyBindings :default -n explore-xx
Name:                   :default
Namespace:              explore-xx
Created:                21 hours ago
Labels:                 <none>
Annotations:                <none>
Last Modified:              2017-04-04 10:50:03 -0500 CDT
Policy:                 <none>
RoleBinding[admin]:
                    Role:           admin
                    Users:          userxx
                    Groups:         <none>
                    ServiceAccounts:    <none>
                    Subjects:       <none>
RoleBinding[system:deployers]:
                    Role:           system:deployer
                    Users:          <none>
                    Groups:         <none>
                    ServiceAccounts:    deployer
                    Subjects:       <none>
RoleBinding[system:image-builders]:
                    Role:           system:image-builder
                    Users:          <none>
                    Groups:         <none>
                    ServiceAccounts:    builder
                    Subjects:       <none>
RoleBinding[system:image-pullers]:
                    Role:           system:image-puller
                    Users:          <none>
                    Groups:         system:serviceaccounts:explore-xx
                    ServiceAccounts:    <none>
                    Subjects:       <none>
RoleBinding[view]:
                    Role:           view
                    Users:          userxx
                    Groups:         <none>
                    ServiceAccounts:    default
                    Subjects:       <none>
```

Show service accounts:
```
$ oc describe serviceaccounts -n explore-xx
Name:       builder
Namespace:  explore-xx
Labels:     <none>

Mountable secrets:  builder-dockercfg-z921w
                    builder-token-22bfm

Tokens:             builder-token-0imdk
                    builder-token-22bfm

Image pull secrets: builder-dockercfg-z921w


Name:       default
Namespace:  explore-xx
Labels:     <none>

Mountable secrets:  default-token-yhj99
                    default-dockercfg-q4i5u

Tokens:             default-token-f9zyz
                    default-token-yhj99


Image pull secrets: default-dockercfg-q4i5u

Name:       deployer
Namespace:  explore-xx
Labels:     <none>


Image pull secrets: deployer-dockercfg-bwpor

Mountable secrets:  deployer-token-ajlo3
                    deployer-dockercfg-bwpor

Tokens:             deployer-token-ajlo3
                    deployer-token-aqcyk

Name:       jenkins
Namespace:  explore-xx
Labels:     app=jenkins-ephemeral
        template=jenkins-ephemeral-template

Mountable secrets:  jenkins-token-16g9q
                    jenkins-dockercfg-x2ftc

Tokens:             jenkins-token-16g9q
                    jenkins-token-l24vf

Image pull secrets: jenkins-dockercfg-x2ftc
```

Redeploy app:
```
$ oc deploy parksmap --latest --follow
Flag --latest has been deprecated, use 'oc rollout latest' instead
Started deployment #2
--> Scaling up parksmap-2 from 0 to 1, scaling down parksmap-1 from 1 to 0 (keep 1 pods available, don't exceed 2 pods)
    Scaling parksmap-2 up to 1
    Scaling parksmap-1 down to 0
--> Success
```

Check on that:
```
$ oc get dc/parksmap
NAME       REVISION   DESIRED   CURRENT   TRIGGERED BY
parksmap   2          1         1         config,image(parksmap:1.2.0)
```

## Remote shell

Get pods, then login:
```
$ oc get pods
NAME               READY     STATUS    RESTARTS   AGE
parksmap-2-k7o7m   1/1       Running   0          2m

$ oc rsh parksmap-2-k7o7m
sh-4.2$
```

One-off commands:
```
$ oc exec parksmap-2-k7o7m -- ls -l /parksmap.jar
-rw-r--r--. 1 root root 21753930 Feb 20 11:14 /parksmap.jar

$ oc rsh parksmap-2-k7o7m whoami
whoami: cannot find name for user ID 1001050000 
```

## S2I deploys

```
$ oc new-app --image="simple-java-s2i:latest" --name="nationalparks" http://gitlab-127.0.0.1/userxx/nationalparks.git
Flag --image has been deprecated, use --image-stream instead
--> Found image e2182f7 (6 months old) in image stream "openshift/simple-java-s2i" under tag "latest" for "simple-java-s2i:latest"

    Java S2I builder 1.0
    --------------------
    Platform for building Java (fatjar) applications with maven or gradle

    Tags: builder, maven-3, gradle-2.6, java, microservices, fatjar

    * The source repository appears to match: jee
    * A source build using source code from http://gitlab-127.0.0.1/userxx/nationalparks.git will be created
      * The resulting image will be pushed to image stream "nationalparks:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "nationalparks"
    * Port 8080/tcp will be load balanced by service "nationalparks"
      * Other containers can access this service through the hostname "nationalparks"

--> Creating resources ...
    imagestream "nationalparks" created
    buildconfig "nationalparks" created
    deploymentconfig "nationalparks" created
    service "nationalparks" created
--> Success
    Build scheduled, use 'oc logs -f bc/nationalparks' to track its progress.
    Run 'oc status' to view your app.
```

Check status:
```
$ oc get builds
NAME              TYPE      FROM          STATUS     STARTED          DURATION
nationalparks-1   Source    Git@240e177   Complete   57 seconds ago   52s
```

Build logs:
```
$ oc logs -f builds/nationalparks-1
Pushing image 172.30.17.230:5000/explore-xx/nationalparks:latest ...
Pushed 0/12 layers, 0% complete
Pushed 1/12 layers, 15% complete
Pushed 2/12 layers, 22% complete
Pushed 3/12 layers, 29% complete
Pushed 4/12 layers, 41% complete
Pushed 5/12 layers, 52% complete
Pushed 6/12 layers, 59% complete
Pushed 7/12 layers, 65% complete
Pushed 8/12 layers, 73% complete
Pushed 9/12 layers, 83% complete
Pushed 10/12 layers, 96% complete
Pushed 11/12 layers, 100% complete
Pushed 12/12 layers, 100% complete
Push successful
```

## Add a DB

```
$ oc new-app --template="mongodb-ephemeral" \
    -p MONGODB_USER=mongodb \
    -p MONGODB_PASSWORD=mongodb \
    -p MONGODB_DATABASE=mongodb \
    -p MONGODB_ADMIN_PASSWORD=mongodb
```

Wire the DB to the rest
```
$ oc env dc nationalparks \
    -e MONGODB_USER=mongodb \
    -e MONGODB_PASSWORD=mongodb \
    -e MONGODB_DATABASE=mongodb \
    -e MONGODB_SERVER_HOST=mongodb

deploymentconfig "nationalparks" updated

$ oc get dc nationalparks -o yaml

      - env:
        - name: MONGODB_USER
          value: mongodb
        - name: MONGODB_PASSWORD
          value: mongodb
        - name: MONGODB_DATABASE
          value: mongodb
        - name: MONGODB_SERVER_HOST
          value: mongodb

$ oc env dc/nationalparks --list
# deploymentconfigs nationalparks, container nationalparks
MONGODB_USER=mongodb
MONGODB_PASSWORD=mongodb
MONGODB_DATABASE=mongodb
MONGODB_SERVER_HOST=mongodb
```

Set some labels:
```
oc label route nationalparks type=parksmap-backend
```

Redeploy the front-end:
```
oc rollout latest parksmap
```

## Config Maps

```
$ wget http://gitlab-127.0.0.1/user98/nationalparks/raw/1.2.1/ose3/application-dev.properties

$ oc create configmap nationalparks --from-file=application.properties=./application-dev.properties
```

Describe it:
```
$ oc describe configmap nationalparks
Name:       nationalparks
Namespace:  explore-xx
Labels:     <none>
Annotations:    <none>

Data
====
application.properties: 123 bytes

$ oc get configmap nationalparks -o yaml
apiVersion: v1
data:
  application.properties: |
    # NationalParks MongoDB
    mongodb.server.host=mongodb
    mongodb.user=mongodb
    mongodb.password=mongodb
    mongodb.database=mongodb
kind: ConfigMap
metadata:
  creationTimestamp: 2017-04-04T16:54:38Z
  name: nationalparks
  namespace: explore-xx
  resourceVersion: "298191"
  selfLink: /api/v1/namespaces/explore-xx/configmaps/nationalparks
  uid: 638b0913-1957-11e7-9e39-02ef4875286e
```

Wire up the configmap:
```
$ oc set volumes dc/nationalparks --add -m /opt/openshift/config --configmap-name=nationalparks
```

Now remove the env variables:
```
$ oc env dc/nationalparks MONGODB_USER- MONGODB_PASSWORD- MONGODB_DATABASE- MONGODB_SERVER_HOST-
```

## Set up some probes:

```
$ oc set probe dc/nationalparks \
    --readiness \
    --get-url=http://:8080/ws/healthz/ \
    --initial-delay-seconds=20 \
    --timeout-seconds=1
$ oc set probe dc/nationalparks \
    --liveness \
    --get-url=http://:8080/ws/healthz/ \
    --initial-delay-seconds=20 \
    --timeout-seconds=1
```

## CICD Lab

Deploy Jenkins:
```
$ oc new-app --template="jenkins-ephemeral"
```

Add permission:
```
$ oc policy add-role-to-user edit -z jenkins
role "edit" added: "jenkins"
```

Remove the route label:
```
$ oc label route nationalparks type-
```

Create mongo-live
```
$ oc new-app --template="mongodb-ephemeral" \
    -p MONGODB_USER=mongodb \
    -p MONGODB_PASSWORD=mongodb \
    -p MONGODB_DATABASE=mongodb \
    -p MONGODB_ADMIN_PASSWORD=mongodb \
    -p DATABASE_SERVICE_NAME=mongodb-live
```

Pull down new configmap:
```
$ wget http://gitlab-ce-workshop-infra.cloudapps.ksat.openshift3roadshow.com/user98/nationalparks/raw/1.2.1/ose3/application-live.properties

$ oc create configmap nationalparks-live --from-file=application.properties=./application-live.properties
```

Tag our live build:
```
$ oc tag nationalparks:latest nationalparks:live
```

Use our new build:
```
$ oc new-app --image="nationalparks:live" --name="nationalparks-live"
```

Set env variables (because configmap is broken in this lab):
```
$ oc env dc/nationalparks-live \
    -e MONGODB_USER=mongodb \
    -e MONGODB_PASSWORD=mongodb \
    -e MONGODB_DATABASE=mongodb \
    -e MONGODB_SERVER_HOST=mongodb-live
```

Add a route, load the data:
```
$ oc expose service nationalparks-live
curl http://nationalparks-live-explore-xx.127.0.0.1/ws/data/load
```

Add a label:
```
oc label route nationalparks-live type=parksmap-backend
```

Disable auto builds for latest:
```
oc set triggers dc/nationalparks --from-image=nationalparks:latest --remove
```

Create pipeline:
```
$ oc new-app dev-live-pipeline \
→     -p PROJECT_NAME=explore-xx
--> Deploying template "openshift/dev-live-pipeline" to project explore-xx

     dev-live-pipeline
     ---------
     CI/CD Pipeline for Dev and Live environments


     * With parameters:
        * Pipeline name=nationalparks-pipeline
        * Project name=explore-xx
        * Dev resource name=nationalparks
        * Live resource name=nationalparks-live
        * ImageStream name=nationalparks
        * GitHub Trigger=a5iYjDTN # generated
        * Generic Trigger=FY3tGSrP # generated

--> Creating resources ...
    buildconfig "nationalparks-pipeline" created
--> Success
    Use 'oc start-build nationalparks-pipeline' to start a build.
    Run 'oc status' to view your app.
```

Start the pipeline:
```
$ oc start-build nationalparks-pipeline
build "nationalparks-pipeline-1" started
```

Check our data. This spits out a boat load of text/json data:
```
curl http://nationalparks-live-explore-xx.127.0.0.1/ws/data/all
```

Promote the pipeline via the gui.

![Promote from the gui](https://i.imgur.com/3daM4sM.png)

Rollback:
```
$ oc rollback nationalparks-live
#5 rolled back to nationalparks-live-3
Warning: the following images triggers were disabled: nationalparks:live
  You can re-enable them with: oc set triggers dc/nationalparks-live --auto
```

Check on that:
```
curl http://nationalparks-live-explore-xx.127.0.0.1/ws/info/
```

Re-enable the new images trigger:
```
$ oc deploy nationalparks-live --enable-triggers
Flag --enable-triggers has been deprecated, use 'oc set triggers' instead
Enabled image triggers: nationalparks:live
```

Roll forward:
```
$ oc rollback nationalparks-live-4
#6 rolled back to nationalparks-live-4
Warning: the following images triggers were disabled: nationalparks:live
  You can re-enable them with: oc set triggers dc/nationalparks-live --auto
```


# Links

* https://docs.openshift.com/enterprise/3.0/architecture/core_concepts/routes.html
* https://docs.openshift.com/enterprise/3.0/admin_guide/manage_authorization_policy.html
* https://docs.openshift.com/enterprise/3.1/dev_guide/deployments.html
* https://docs.openshift.com/enterprise/3.0/dev_guide/new_app.html
* https://docs.openshift.com/enterprise/3.0/dev_guide/service_accounts.html
* https://docs.openshift.com/enterprise/3.0/dev_guide/volumes.html
* https://blog.openshift.com/openshift-3-3-pipelines-deep-dive/