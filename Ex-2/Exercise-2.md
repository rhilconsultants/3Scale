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
Choose the *BACKENDS* tab in the dashboard page and press *NEW BACKEND*:

Name: **Coolstore Catalog Backend**

System Name: **coolstore_catalog_backend**

Description: **The backend Service for the coolstore catalog**

Private Base URL: *the output of the following command:*
    
```bash
echo http://catalog-service.$OCP_USERNAME-coolstore.svc.cluster.local:8080
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114390470-ff949e00-9b9e-11eb-8f02-749100626166.png">
</p>

Now you have been redirected to the backend page (notice that the navigation bar changed).

In the side panel, press “Methods & Metrics” and add the methods based on the Swagger editor  from the previous exercise.

In the side panel, press "Mapping Rules" and create rules that correspond to the methods you created in the previous step. Pay close attention to the “Verb” for each rule.

* {itemId} is a wildcard that can accept multiple inputs 
    * The validity check for the wildcard can be implemented in the application or by implementing a built-in policy (header parameter check) in 3scale.

**Note!** If you need help, please go to the last page in that document to see the screenshots of the final configuration.

### Add new backend #2
Add another backend for the inventory:

Choose the *BACKENDS* tab in the dashboard page and press *NEW BACKEND*:

Name: **Coolstore Inventory Backend**

System Name: **coolstore_inventory_backend**

Description: : **The Backend of coolstore inventory**

Private Base URL: *the output of the following command:*

```bash
echo http://inventory-service.$OCP_USERNAME-coolstore.svc.cluster.local:8080
```

Now you have been redirected to the backend page (notice that the navigation bar changed).

In the side panel, press “Methods & Metrics” and add the methods based on the Swagger editor from the previous exercise.

In the side panel, press "Mapping Rules" and create rules that correspond to the methods you created in the previous step. Pay close attention to the “Verb” for each rule.


## Add Product & Metrics
### Introduction
In comparison to methods, metrics are a unit of measure that 3scale uses to calculate the number of “hits” for a specific request type. 

A metric can be calculated for all of the hits by using the default “hits” metric or a custom one.

The component that actually calculates the hits is the API gateway so the metrics are mostly configured in the "product" page and not in the "backend" page. In that regard the metrics are different from methods - but it is possible to add metrics in the backend level in some implementations.

## Add New Product #1

Choose the *PRODUCTS* tab in the dashboard page and press *NEW PRODUCT*:

**Define Manually**

Name: **Coolstore API**

System Name: **coolstore_api**

Now you have been redirected to the product page of Coolstore API, from which we will manage the APIcast for both services we just deployed.

Now navigate to **Integration** ⇒ **Backends** ⇒ **Add backend** and add both backends, for paths choose “*/inventory*” for the inventory backend and “*/catalog*” for the catalog backend.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114391211-e17b6d80-9b9f-11eb-9720-e4a24bfba320.png">
</p>

Lets add the metrics for our methods; Navigate to **Integration** ⇒ **Methods & Metrics** ⇒ **New Metric** and *add one metric for each of the previous methods we configured for both services* (Inventory & Catalog).

you can see an example here:

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114391383-1ab3dd80-9ba0-11eb-8646-89b6e25057d6.png">
</p>

Now when you finished with that you need to map each metric to an API path so it can be monitored separately, as follows:

configure it in: **Integration** ⇒ **Mapping rules**

* **Notice we are mapping the rules in the product based on the absolute path - which means that if we have more than one backend (like in this exercise) the pattern should be based on the “/catalog” prefix that we included for that specific backend**

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114391548-4df66c80-9ba0-11eb-8377-488a7d7edfd4.png">
</p>

## Edit APIcast Public URL
Navigate to **Integration** ⇒ **Settings** 

Now first and foremost change the type of the APIcast deployment to APIcast self-managed.

The difference between “APIcast self-managed” and “APIcast 3scale managed” is that in the former (self-managed) the APIcast is in our control - we control the scalability, the quotas, the high availability, the URL for both staging & production etc.  

Configure in “Staging Public Base URL” to be the output of the following command:

```bash
echo https://$OCP_USERNAME-coolstore-api-3scale-apicast-staging.apps.${OCP_WILDCARD_DOMAIN}:443
```

and for “Production Public Base URL”: 

```bash
echo https://$OCP_USERNAME-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443
```

**Press the “Update Product” button  at the bottom of the page.**

**Note:** because we are using APIcast self-managed we need to expose the new routes by ourselves using the APIcasts services that are already deployed for each of you under project name $OCP_USERNAME-gw; They are dedicated to each of your tenants and you have permissions to create new routes to your products.

```bash
oc get svc -n $OCP_USERNAME-gw

oc create route edge coolstore-staging-route --service=stage-apicast --hostname `echo ${OCP_USERNAME}-coolstore-api-3scale-apicast-staging.apps.${OCP_WILDCARD_DOMAIN}` -n $OCP_USERNAME-gw

oc create route edge coolstore-production-route --service=prod-apicast --hostname `echo ${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}` -n $OCP_USERNAME-gw
```

Verify that both routes were added:

```bash
oc get route -n $OCP_USERNAME-gw | grep coolstore
```

Let's go back to 3scale UI and navigate to **Integration** ⇒ **Configuration** there you can promote our Product to staging and to production.
We will test that everything is working after adding a user that will have permission to send requests through 3scale.

## Add user & application
In order to link an application to a user we need first to add an “application plan” to the application. We will see the application plan more in-depth in the next exercises but for now we’ll just create a dummy one.

In our Product page navigate to **Applications** ⇒ **Application Plans** ⇒ **Create Application Plan** 

Name: **Default Plan**

System Name: **default_plan**

Press *Create Application Plan* and then *Publish*

Using the navigation bar go to *Audience* as we saw earlier and navigate to **Accounts** ⇒ **Listing** ⇒ **Create**

UserName: **Test**

Email: **Test@gmail.com**

Password: **test123**

Organization: **Test**

Now press **Create**

Above the title press the **Applications** option and add the new Product we just created.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114392354-513e2800-9ba1-11eb-8ee2-24c5deb3cca3.png">
</p>

Press **Create Application**, and then choose the application plan associated with the Coolstore API product.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114392481-8480b700-9ba1-11eb-98d8-20459bb6fe7d.png">
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114392560-a24e1c00-9ba1-11eb-903a-af465838ad0e.png">
</p>

You have received a API User-Key for your new user, copy it and let's test our product.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114392670-c7428f00-9ba1-11eb-93fd-b80bd08a1b2b.png">
</p>

```bash
USERKEY=<Copy the User Key Here>
curl -k "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-staging.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=$USERKEY"  
```

If everything works fine you should be able to receive a valid response with data about the products.

Lets see what the metrics we defined earlier means;

Using the navigation bar go to the Catalog API product then to **Analytics** ⇒ **Usage** you should be able to see the “Hits” metric and to switch it to other metrics (switch to the metric that corresponds with the “/products” API call) and you can see that we have a hit on that metric only - which means we can (theoretically) add monitoring rules for each and every method we define using 3scale.

**Note!!** If you don’t see hits on your metric, navigate to: **Integration** ⇒ **Mapping rules** and delete the default “GET /” default statement and test again.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114392940-1d173700-9ba2-11eb-82ff-a3a392a07e73.png">
</p>

# Appendix
## Methods - configure in the backend page

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114393062-4637c780-9ba2-11eb-832c-ee0a4114c4b9.png">
</p>

## Mapping rules - configure in the backend page


<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114393100-53ed4d00-9ba2-11eb-82e2-01b463bdbbdd.png">
</p>
