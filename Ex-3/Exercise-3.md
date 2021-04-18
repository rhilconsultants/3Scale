# Exercise 3 - 3Scale Security Capabilities
### V1 Date: 05/2020
### V2 Date: 03/2021

# 3Scale Application Plan
## Intro
3Scale has several main security features that greatly enhance the security of any API that 3Scale protects. 

One of the core features with security functionality is application plans, which also contains other capabilities such as monitoring based on metrics and rate limiting.

An application plan can be looked at as a methods access policy; A 3scale user that registers to access an API gets an "application plan" which defines  what methods (Verb+ Path) can be accessed, how many requests can be sent in a certain period of time (Rate Limiting), either to a specific method or all of them. 

Application plans are configured on the product level - because it “sees” several backends simultaneously and it’s configuration needs to be enforced in the APIcast.

## Deploy Application Plans
### Basic Application Plan
The first plan that we are going to deploy is a Basic application plan. 

This application plan is meant to grant limited access to “low-level” users - GET methods only, to non-confidential api methods - it corresponds with read-only permissions.

Let’s configure it; Navigate to:

**Coolstore API** ⇒ **Applications** ⇒ **Application Plans** ⇒ **Create Application Plan**

Name: **Read Only Application Plan**

System Name: **read_only_application_plan**

Application Require Approval: **do not mark** 
* If this option is checked, when the user registers to the API - the 3Scale admin needs to approve them before they can get access to the API.

Press **Ok**, and then press **Publish**. Hidden plans are not available for users.

Choose the new application plan; Notice the separation to *Product Level* and to *Backend Level* - in the upper you can configure limitations that will enforce them on **all backends** (mostly useful for metrics that the APIcast counts for all the backends behind it), the latter enables limitations on a specific backend method access.

Let’s limit the Read-only plan to 4 requests a minute - to prevent D/DOS attacks on the API.

**Product level** ⇒ **get all products metric** ⇒ **Limits** ⇒ **New usage limit** ⇒ **Period: minutes** & **Max Value:3**

Also lets prevent access to the POST method;

**Backend level** ⇒ **Catalog Backend** ⇒ **add new product** ⇒ Press the **enabled & visible checks**

**Roll the page up and press "Update Application plan"**

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114394522-f8bc5a00-9ba3-11eb-870a-761c4fdc5f9a.png">
</p>

We need to change the user that we configured in the previous exercise to use the new application plan

**Audience** ⇒ **Applications** ⇒ **Coolstore App** ⇒ **Change application plan** ⇒ **Choose newly configured application plan**

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114394648-1b4e7300-9ba4-11eb-9c08-b3edf2bdd1ae.png">
</p>

Updated:

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114394760-39b46e80-9ba4-11eb-8554-0eb878b9e812.png">
</p>

At the bottom of the page:

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114394848-5badf100-9ba4-11eb-954b-8bcfd5ddcc44.png">
</p>

### Lets test

To make life easier - let's create an environment variable for the user key for further usage 

```bash
# option 1 - bashrc file:
echo export USER_KEY=<paste the key here> >> ~/environ.sh
source ~/environ.sh

# option 2 - variable only:
export USER_KEY=<paste the key here>
```

First of all - verify that you can’t access some method for more then 10 times

```bash
END=6
for i in $(seq 1 $END); do echo iteration $i: ; curl -iks "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=${USER_KEY}" | grep HTTP; done
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114396837-b21c2f00-9ba6-11eb-84d8-3cb29a6ca5e6.png">
</p>

Run one last time with “-v” switch and find the 429 error response
```bash
curl -k -v "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=${USER_KEY}"
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114397161-09ba9a80-9ba7-11eb-9d35-064478c722a0.png">
</p>

Secondly - verify that you can’t access the POST method;
```bash
curl -k -X POST "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=${USER_KEY}" -H "accept: application/json" -H "Content-Type: application/json" -d "{ \"itemId\": \"itemIdTest\", \"name\": \"lol\", \"desc\": \"lol\", \"price\": 555}" ; echo
```
<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114397833-c876ba80-9ba7-11eb-910b-f0e5b8b3fbb9.png">
</p>

The error we got - “Authentication Failed” is vague on purpose. The 3Scale developers team decided that it’ll be safer to respond with not-so-clear, default error messages in order to information gathering by a potential attacker. 

These error messages are editable here:

**Coolstore API** ⇒ **Integration** ⇒ **Settings** 

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114398171-27d4ca80-9ba8-11eb-9e7a-78e05d6ff8d7.png">
</p>

### Premium Application Plan
This application plan is basically an unlimited access plan. 

Let’s configure it; Navigate to:

**Coolstore API** ⇒ **Applications** ⇒ **Application Plans** ⇒ **Create Application Plan**

Name: **Premium**

System Name: **premium_application_plan**

Application Require Approval: **Mark** 
*  If this option is checked, when the user registers to the API - the 3Scale admin needs to approve them before they can get access to the API. 

Press **Ok**, and then press **Publish**. Hidden plans are not available for users. 

We need to change the user so it can use the new application plan

**Audience** ⇒ **Applications** ⇒ **Mark specific app** ⇒ **Change application plan** ⇒ **Choose newly configured application plan**

Run both tests for the previous section and see that they both work with no errors received.

# 3Scale Policies
## Intro

Another key feature of 3Scale security is the policies. The 3Scale policies grants a more modular approach to secure the APIs; You as a 3Scale admin/API developer working with 3Scale can choose which policies to use and configure them independent of each other. The policies are built in a chain succession; each of them runs one after the other and if one fails - the whole chain is considered as “unauthorized”.

There are two types of policies - built-in and custom; The built-in policies come out-of-the-box with 3Scale, and can be configured using the 3Scale admin UI. The custom policies are policies that are written in Lua scripting language, per-use case, by the customer, and can be deployed into 3Scale APIcast (using it’s Lua engine and Openresty) - they are not supported by 3Scale because the customers write them by themselves. 

The custom policies are out-of-scope of that workshop.

## Add a built-in policy

Navigate to: **Coolstore API** ⇒ **Integration** ⇒ **Policies**

Notice the Default policy which is the 3scale APIcast policy - it is the one that triggers the application plans checking.

Press **Add policy** and examine the different options, you can see that there’s a lot of them - each one for a different use. 

Lets test an easy one - choose the Logging policy, and after you added it - drag it to be before the 3Scale APIcast policy;

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114399005-1f30c400-9ba9-11eb-93c5-0fb2ab259e31.png">
</p>

Press the logging policy, verify it’s **enabled** and also mark the **enable_access_logs** option.

Notice that you configure a lot more (using the “+” sign) like send a specific log message for a specific value in the request - we are not going to use it meant to show you that you can use the built-in policies in a wide variety of scenarios without the need to write a custom policy of your own. 

click **update policy** and then click **update policy chain**, and promote the APIcast to stating and to production you saw earlier.
Lets see how it works;

```bash
curl -k "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=${USER_KEY}" 

APICAST_PROD_POD=$(oc get pods -n ${OCP_USERNAME}-gw | grep prod  | grep -i running | awk '{print $1}')

oc logs -f pod/${APICAST_PROD_POD} -n ${OCP_USERNAME}-gw | grep ${USER_KEY}
```

# 3Scale Authentication options

In the settings page of a specific product we will take a look at the Authentication configuration options for the product;  **Coolstore API** ⇒ **Integration** ⇒ **Settings**

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114399389-8b132c80-9ba9-11eb-833e-acbf16625147.png">
</p>

We already made use of the first one - API User Key, we attached it to every cURL command that we used when sending a request to the catalog API; without it - the APIcast will drop the request and send an error message - you can see it in the settings page as well. 
The second one is not much of a difference of the first one - it’s just a 2 parameter authentication - one is the App_ID, which is a key for 3Scale to verify the request source validity and the second one - APP_Key, is the exact same as API User Key in the first option.

They are both not very reliable as a security measure to access an API but there are organizations that use them to simplify the management (although it's a lame excuse at best). We can tell the difference between them based on the source of the API requests; The first one will be used mostly for a C2B (Client to Business) connection - because a regular user doesn't (usually) use a sophisticated client and attach more than a single verifier key to a request.  

The second authentication option is mostly used for B2B (Business to Business)connections - a secondary app (external/internal) that requires data from the API, attaches the APP_ID key to every request to verify the source validity and prevent MITM (Man in the Middle) attacks in the B2B perimeter. 

**The last option is the most secured option - OIDC (OpenID Connect) Authentication**
It works with an external IDP (identity provider) or an equivalent - good examples are RH-IDM or even better - RH-SSO which can integrate several IDPs into one platform and grants authorization mechanism using SAML or JWT tokens.

Syncing between 3Scale and RHSSO for OID auth is out of the scope of that course, but you can read [my medium article](https://medium.com/@tamber/api-management-security-series-3scale-oidc-using-rh-sso-demo-643feb1e1c0d) that covers the topic from top to bottom, including demo & screenshots that you can run be yourself in the lab environment of that course for personal enrichment.

# 3Scale Alerts

**Product Coolstore API**  ⇒ **Applications** ⇒ **Settings** ⇒ **Usage rules** ⇒ **mark the “50%, 80%, 100%”** for the **“Show Web Alerts to Admins of the Developer Account”**

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114400226-6c616580-9baa-11eb-9940-a7c47532272c.png">
</p>

Remember that we configured two application plans, the read only has limit on the access to the product (all backends) so we can create an alert for the admin to see if the user is about to reach the limit of its access in the near future;

We need to change the user so it can use the new application plan

**Audience** ⇒ **Applications** ⇒ **Mark specific app** ⇒ **Change application plan** ⇒ **Choose the read only application plan**

Send the maximum amount of requests permitted:

```bash
END=6
for i in $(seq 1 $END); do echo iteration $i: ; curl -iks "https://${OCP_USERNAME}-coolstore-api-3scale-apicast-production.apps.${OCP_WILDCARD_DOMAIN}:443/catalog/products?user_key=${USER_KEY}" | grep HTTP; done
```

Now lets see what the admin will see in the alerts page; Navigate to:

**Product** ⇒ **Analytics** ⇒ **Alerts**

**Note!** It may take a few minutes to see the alerts on the 3Scale UI due to delay of the messages from the APICasts

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114400285-7a16eb00-9baa-11eb-944b-28e66e7e7c68.png">
</p>

