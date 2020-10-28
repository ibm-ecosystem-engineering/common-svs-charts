# FSS Cloud Helm charts

A collection of helm charts used by and/or provided for the FSS Cloud - Falcon Banc development

This GitHub repository also serves as a Helm repository, hosting the helm charts via GitHub pages. The url for the Helm repository is https://github.com/ibm-gsi-ecosystem/fss-charts

# Helm

Helm is a package manager. Package managers automate the process of installing, configuring, upgrading, and removing computer programs. Examples include the Red Hat® Package Manager (RPM), Homebrew, and Windows PackageManagement.

An application in OpenShift typically consists of at least two resource types: a deployment resource, which describes a set of pods to be deployed together, and a services resource, which defines endpoints for accessing the APIs in those pods. The application can also include ConfigMaps, Secrets, and Ingress.

# Helm Install 

You can install Helm Charts

```
helm install <NAME> <CHART FOLDER>/ --values <CHART FOLDER>/values.yaml
```

## Helm Install - OpenLDAP

helm install openldap openldap/ --values openldap/values.yaml


## Helm Install - LDAPConsole

helm install ldapconsole ldapconsole/  --set LDAP.host=<Host IPAddress> --values ldapconsole/values.yaml

# Helm Dependency Update

You can run the helm dependency update within any chart folder

```
helm dep update
```

# Helm Tutorials
- https://helm.sh/
- https://www.ibm.com/cloud/architecture/content/course/helm-fundamentals
