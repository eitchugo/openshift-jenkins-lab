# OpenShift Jenkins Lab

How to use Jenkins CI/CD integrations with OpenShift 

# Using a shared Jenkins installation

Install a jenkins on a separate namespace:

```
oc new-project cicd
oc new-app jenkins-persistent --param ENABLE_OAUTH=true --param MEMORY_LIMIT=2Gi --param VOLUME_CAPACITY=4Gi --param DISABLE_ADMINISTRATIVE_MONITORS=true
```

Global template for deploying services:

```
oc create -f jenkins-services.yaml -n openshift
```
