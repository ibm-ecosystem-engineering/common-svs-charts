# Common Services Cloud Helm charts

A collection of helm charts used by and/or provided for the FSS Cloud - Falcon Banc development

This GitHub repository also serves as a Helm repository, hosting the helm charts via GitHub pages. The url for the Helm repository is https://github.com/ibm-gsi-ecosystem/fss-charts

# Helm

Helm is a package manager. Package managers automate the process of installing, configuring, upgrading, and removing computer programs. Examples include the Red Hat Package Manager (RPM), Homebrew, and Windows PackageManagement.

An application in OpenShift typically consists of at least two resource types: a deployment resource, which describes a set of pods to be deployed together, and a services resource, which defines endpoints for accessing the APIs in those pods. The application can also include ConfigMaps, Secrets, Routes and Ingress.

![Helm Components](Helm.png)


# Helm Install 

## Helm Install on Local

[Installing Helm] (https://helm.sh/docs/intro/install/#helm)

## Helm Install - Charts

You can install Helm Charts

```
helm install <NAME> <CHART FOLDER>/ --values <CHART FOLDER>/values.yaml
```

### Helm Install - OpenLDAP

LDAP was originally a protocol, Lightweight Directory Access Protocol, and is now a directory service specification in its own right, including all kinds of schemas and extras.

OpenLDAP is an open-source implementation of LDAP, both server and client.

Here is the script to install the OpenLDAP Server via Helm Chart

```
helm install openldap openldap/ --values openldap/values.yaml
```

**Note:** Restart the pod if it crashloopback

### Helm Install - LDAPConsole (phpLDAPadmin)

phpLDAPadmin (also known as PLA) is a web-based LDAP client. It provides easy, anywhere-accessible, multi-language administration for your LDAP server.

Its hierarchical tree-viewer and advanced search functionality make it intuitive to browse and administer your LDAP directory. Since it is a web application, this LDAP browser works on many platforms, making your LDAP server easily manageable from any location.

**Note:** rename folder "console" to "ldapconsole" as some reason github repo is not showing contents if i keep "ldapconsole"

#### Certificates - Config Map

There are several options [Secret, IBM Storage , Cloud Object Storage, Certificate Manager etc] to load the certificate which is available in OpenShift. 

For the ldap, create 3 Config maps and load the certificates on each correspondingly as shown below:-

![Config Maps - Certificate](ConfigMaps.png)

- **falconbanc-cert**
    Create the child named "falcon_banc.pem" under the data section and load the FalconBanc.pem contents

- **falconbanc-keys**
    Create the child named "falcon_banc.key" under the data section and load the FalconBanc.key content

- **falconbanc-root-cert**
    Create the child named "letsencryptauthorityx3-root.pem" under the data section and load the letsencryptauthorityx3-root.pem contents

Here is the script to install the LDAPConsole (phpLDAPadmin) via Helm Chart

```
helm install ldapconsole ldapconsole/  --set LDAP.host=<Host IPAddress> --values ldapconsole/values.yaml
```

**Note:** Restart the pod if it crashloopback

# OAuth Proxy

### OpenShift oauth-proxy
A reverse proxy and static file server that provides authentication and authorization to an OpenShift OAuth server or Kubernetes master supporting the 1.6+ remote authorization endpoints to validate access to content. It is intended for use within OpenShift clusters to make it easy to run both end-user and infrastructure services that don't provide their own authentication.

**Features:**

- Performs zero-configuration OAuth when run as a pod in OpenShift
- Able to perform simple authorization checks against the OpenShift and Kubernetes RBAC policy engine to grant access
- May also be configured to check bearer tokens or Kubernetes client certificates and verify access
- On OpenShift 3.6+ clusters, supports zero-configuration end-to-end TLS via the out of the box router
- This is a fork of the https://github.com/bitly/oauth2_proxy project with other providers removed (for now). It's focused on providing the simplest possible secure proxy on OpenShift

### Using this proxy with OpenShift
This proxy is best used as a sidecar container in a Kubernetes pod, protecting another server that listens only on localhost. On an OpenShift cluster, it can use the service account token as an OAuth client secret to identify the current user and perform access control checks. 

### Install the OAuth Proxy on ROKS 4.5

Navigate to the OAuthProxy and open the oauthproxy.yaml to update the following args. Refer the OAuth Proxy Parameters for explanation for each parameters - https://github.com/openshift/oauth-proxy#command-line-options

- --https-address=:7443
- --provider=openshift
- --openshift-service-account=oauthproxy
- --upstream=https://<IP Address>:10111
- --request-logging=true
- --tls-cert=/etc/tls/private/tls.crt
- --tls-key=/etc/tls/private/tls.key
- --cookie-secret=SECRET
- --ssl-insecure-skip-verify=true
- --upstream-ca=/etc/tls/private/ca/openpages.crt

Once you modify the oauthproxy.yaml, apply changes to the ROKS Cluster

1. Login to OCP Cluster and copy the CLI Login commands
    oc login --token=**TokenID** --server=**ServerHost**

2. Create the ConfigMAP as "openpages" using the external host CA certificate. 

**Retrieve External Host CA CRT**
  If you don't have CA.crt from the external host machine, then run the following command which retrieves and store the certificate in the file name "openpages.crt"

        
        openssl s_client -showcerts -connect <ExternalHost>:<Port>  </dev/null 2>/dev/null|openssl x509 -outform PEM > <File Name.crt>
       

  After you secured the openpages.crt file, you need to create the ConfigMAP and link the ExternalHost CA certificate
    - Refer the screenshot how the config MAP yaml 

![ConfigMAP OpenPages](/OAuthProxy/images/configmap.png)

**Note:** As reference configmap file, you can refer the contents of openpages.yaml file in the folder [OAuthProxy] 

3. Switch to the namespace which you want proxy to be running
    oc project <Namespace name>

4. Add 3 certificate contents to the oauthproxy.yaml which you received from the domain 

    - Falcon_Banc.pem
    - Falcon_Banc.key
    - letsencryptauthorityx3-root.pem

**oauthproxy.yaml**
```
    certificate: |
      -----BEGIN CERTIFICATE-----
        << Insert Falconbanc.pem PEM Contents >>
      -----END CERTIFICATE-----
    key: "-----BEGIN PRIVATE KEY-----
      << Insert Falconbanc.key Contents>>
      -----END PRIVATE KEY-----\r\n"
    caCertificate: |
      -----BEGIN CERTIFICATE-----
        << Insert Lets Encrypt CA Contents >>
      -----END CERTIFICATE-----
```

5. Apply the YAML File oauthproxy.yaml
    ```
    oc create -f oauthproxy.yaml
    ```

    Here is the CLI Output

    ```
    serviceaccount/oauthproxy created
    service/oauthproxy created
    deployment.apps/oauthproxy created
    route.route.openshift.io/oauthproxy created
    ```

6. Goto OCP Console and verify the POD, Deployment, Services , Route are healthy state

7. Click the Route Location URL that will render the upstream host contents 

![OpenPages ROUTE](/OAuthProxy/images/openpages-routes.png)

**TIP:**

#### Configuring the proxy's service account in OpenShift
In order for service accounts to be used as OAuth clients, they must have the proper OAuth annotations set. to point to a valid external URL. In most cases, this can be a route exposing the service fronting your proxy. We recommend using a Reencrypt type route and service serving certs to maximize end to end security.

```
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: proxy
    annotations:
      serviceaccounts.openshift.io/oauth-redirectreference.primary: '{"kind":"OAuthRedirectReference","apiVersion":"v1","reference":{"kind":"Route","name":"proxy"}}'
```

#### References

OAuth Proxy : https://github.com/openshift/oauth-proxy

