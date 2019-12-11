# OpenShift Jenkins Lab

How to use Jenkins CI/CD integrations with OpenShift 

# Using a shared Jenkins installation

Install a jenkins on a separate namespace:

```
oc new-project cicd
oc new-app jenkins-persistent --param ENABLE_OAUTH=true --param MEMORY_LIMIT=2Gi --param VOLUME_CAPACITY=4Gi --param DISABLE_ADMINISTRATIVE_MONITORS=true
```

Alternatively, jenkins without a persistent volume:

```
oc new-app jenkins-ephemeral --param ENABLE_OAUTH=true --param MEMORY_LIMIT=2Gi --param DISABLE_ADMINISTRATIVE_MONITORS=true
```

Add a template to jenkins placeholder services:

```
oc create -f jenkins-services.yaml -n openshift
```

(Optional) Configure `/etc/origin/master/master-config.yaml` on all masters to disable auto-provisioning:

```
jenkinsPipelineConfig:
  autoProvisionEnabled: false
  templateNamespace: openshift 
  templateName: jenkins-ephemeral 
  serviceName: jenkins
```

# Using another project for buildconfigs

If you want your pipelines/buildconfigs to be located in another namespace, do these steps. This is useful when developers have access only to their projects,
not the jenkins project.

Create the project and give permissions to jenkins in another namespace:

```
oc new-project pipelines
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins
oc create -f jenkins-sa.yaml
```

(Optional) If you didn't disable jenkins auto-provisioning, create jenkins services:

```
oc process jenkins-services -n openshift | oc create -f -
```

## Configuring openshift-jenkins-sync plugin

You only need this if you're using a separate namespace. If you're creating
the pipelines inside the `cicd` project, this isn't needed.

1. Login into Jenkins

   ```
   oc get route -n cicd
   ```

2. Go to *Manage Jenkins* -> *Configure System* -> *OpenShift Jenkins Sync*

3. Set field *Namespace* to:

   ```
   cicd pipelines
   ```

   (Namespaces are separated by a single space)

4. Click *Save*.

## Populating the sample pipelines templates

Create the templates:

```
oc create -f pipelines/simple/bc.yaml
oc create -f pipelines/other-namespace/bc.yaml
```

# Creating the buildconfigs

Simple pipeline:

```
oc process simple-pipeline | oc create -f -
```

Simple pipeline with alternate git repository:

```
oc process simple-pipeline BC_NAME=simple-pipeline-altgit GITURL=https://example.com/gitlab/openshift-jenkins-lab.git | oc create -f -
```

Simple pipeline building in another namespace (`pipelines`):

```
oc process other-namespace-pipeline BUILD_NAMESPACE=pipelines | oc create -f -
```

# Running builds

You can run directly inside Jenkins or with OpenShift:

```
oc get bc
oc start-build <bc-name>
```
