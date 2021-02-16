# Howto

## Deploy Artifactory on Kubernetes with Helm


1. Add an JFrog Artifactory helm chart repo 

```bash
helm repo add https://charts.jfrog.io
```

2. Install artifactory from helm chart

```bash
helm install --name artifactory jfrog/artifactory
```

Helm chart as default will install all necessary services:
 - Postgresql 
 - NginX as LoadBalancer service
 - Artifactory web service

## Use a customized installation

There are many preferences which helm installation can use to deployment customization.
To print the options use following: 

```bash
helm inspect jfrog/artifactory
```

## Versions
You can specify the docker image and release version of the Jfrog Artifactory package, which should be deployed with following arguments:
```bash
--set artifactory.image.repository="<docker image>" --set artifactory.image.version="<version>"
```
[List of Artifactory releases](https://www.jfrog.com/confluence/display/RTF/Release+Notes), and also [Known Issues](https://www.jfrog.com/confluence/display/RTF/Known+Issues) in specific version.

In general there is two main version of JFrog Artifactory. 
[JFrog Artifactory PRO version](https://bintray.com/jfrog/product/artifactory/download) and [JFrog Artifactory OSS version (open-source)](https://jfrog.com/open-source/#artifactory)

Helm chart installation command, with latest open-source version of JFrog Artifactory will be:

```bash
helm install --name="artifactory" --set artifactory.image.repository=”docker.bintray.io/jfrog/artifactory-oss” --set artifactory.image.version="latest" jfrog/artifactory
```

## Upgrade Artifactory

Similar as the installation process, there is an Artifactory upgrade option, to re-deploy artifactory with new chart version. 
The upgrade process will be applied on the existing Artifactory deployment.

```bash
helm upgrade artifactory jfrog/artifactory
```

Read more about Artifactory deployment on [Helm-chart Artifactory repo](https://github.com/helm/charts/tree/master/stable/artifactory)


## Artifactory Maintaining in Kubernetes

## Loggers

Artifactory uses the Logback Framework to manage logging. Activity is logged according to type in four different log files which can be found under the ARTIFACTORY_HOME/logs folder.
The following log files are available: 

 - artifactory.log - The main Artifactory log file containing data on Artifactory server activity.
 - access.log - Security log containing important information about accepted and denied requests, configuration changes and password reset requests. The originating IP address for each event is also recorded.
 - request.log - Generic http traffic information similar to the Apache HTTPd request log.
 - import.export.log - A log used for tracking the process of long-running import and export commands.

Enable the specific log is possible with parameter within executing helm install / upgrade process
```bash
  --set artifactory.loggers[]="<log-name>"
```

To print required log in kubernetes, use following command:
```bash
kubectl logs <artifactory-pod-name> --namespace <namespace-name> <log-name>
```

## Backup Management

Artifactory allows you to make your own backup plan. Backup interval and repositories for backup could be set from Artifactory WebUI under Services -> Backup.
To backup setup, there are following configuration options:

 - Backup Key - A unique logical name for this backup
 - Cron Expression - A valid Cron expression that you can use to control backup frequency. For example, to back up every 12 hours use a value of: 0 0 /12 * * ?
 - Next Time Backup - When the next backup is due to run
 - Server Path for Backup - The directory to which local repository data should be backed up as files. The default is $ARTIFACTORY_HOME/backup/[backup_key]. Each run of this backup will create a new directory under this one with the time stamp as its name.
 - Exclude Builds - Notice, from Artifactory version 6.6, the Exclude builds checkbox is removed. To exclude builds, you'll need to add the artifactory-build-info repository to the Excluded Repositories.
 - Retention Period - The number of hours to keep a backup before Artifactory will clean it up to free up disk space. Applicable only to non-incremental backups.
 - Verify enough disk - If set, Artifactory will verify that the backup target location has enough disk space available to hold the backed up data.
 - Incremental - When set, this backup should be incremental. In this case, only changes from the previous run will be backed up, so the process is very fast.
 - Backup to a ZIP Archive - If set, backups will be created within a Zip archive

More about Backup management on [Artifactory Managing Backups](https://www.jfrog.com/confluence/display/RTF/Managing+Backups). 


## Artifactory data keeping in persistent manner

A Default helm installation process without any storage specifications will create all necessary components included storage volumes for postgresql and Artifactory.
But in case of upgrade or uninstalling Artifactory from kubernetes, all data will be deleted within storage volumes. Because of that, it is appropriate to ensure that storage volumes, what we want to use
are created and available for our Artifactory deployment. To specify helm-chart installation, which volume should be used, add following parameter to the installation process:

```bash
--set artifactory.persistence.existingClaim="kubernetes-persistent-volume-claim-name"  
```

The process of creating persistent volumes in Kubernetes are described on [Kubernetes persistent volume storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).













