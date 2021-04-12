# Exercise 2 - Getting Started with 3scale
### V1 Date: 05/2020
### V2 Date: 03/2021



# 3scale Login & Dashboard
## Login Credentials
The 3Scale administrator usually using the following commands to extract the admin username & password after the product installation;
```bash
oc get secret system-seed -o json -n ${API_NAMESPACE} | jq -r .data.ADMIN_USER | base64 -d ; echo
oc get secret system-seed -o json -n ${API_NAMESPACE} | jq -r .data.ADMIN_PASSWORD | base64 -d ; echo
```

**You are not going to use these commands because your ocp user (user1…..etc) can only login to its own 3Scale tenant - using the credentials you got from your instructor;**
```bash
echo $API_USERNAME
echo $API_PASSWORD

```

Use browser go to the following url and enter the credentials
```bash
echo -en "https://${API_URL}:443" ; echo
```

## Examin 3scale Dashboard
In the 3scale dashboard we can see several key components of it. 

First of all notice the separation between the tabs for "PRODUCTS" and "BACKENDS".

Each backend represents an internal service that we would like to expose via 3scale, and each product represents an APIcast that is responsible for the exposure of the backends.

It’s important to mention that a single “product” can have several “backends” that it can expose simultaneously, behind a single public URL. We will do it later on in an exercise.

In the upper part of the screen we can see a nav-bar which will help us navigate between the different components that we would like to manage by 3scale. 

Also notice that in the navigation bar drop-down menu (labeled "Dashboard"). 

From it we can choose the page “Audience” - the page we will use to add new users that will be able to access our apps.

Last but not least take a look at the gear icon on the upper right; That will take us to the page from which we can add more administrative users, grant administration access to developers to manage their APIs and get Tokens to manage 3scale from its API; we will try it out later in that workshop.

# Manage our first API
## Add Backends & Methods
### Introduction
It’s important to know the convention of internal urls of OpenShift resources;

An OpenShift resource has an internal URL for inter-cluster access between different services; It’s not uncommon for services/pods to communicate with each other not through exposed routes - either because it’s not a public service or to reduce network latency.

The convention for an internal services route is: service_name.namespace.svc.cluster.local

* service_name == oc get svc -n $OCP_USERNAME-coolstore ⇒ inventory-service/catalog-service
* namespace == echo $OCP_USERNAME-coolstore

### Add new backend #1
Choose the BACKENDS tab in the dashboard page and press “NEW BACKEND”:

	Name: **Coolstore Catalog Backend**

	System Name: **coolstore_catalog_backend**

	Description: **The backend Service for the coolstore catalog**

	Private Base URL: *the output of the following command:*
    
    ```bash
    echo http://catalog-service.$OCP_USERNAME-coolstore.svc.cluster.local:8080
    ```
