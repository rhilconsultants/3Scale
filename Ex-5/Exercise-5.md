# Exercise 4 - 3Scale Automation Using API-Calls
### V1 Date: 05/2020
### V2 Date: 03/2021

# Using 3Scale Management API
## Generate new Access Token
Before we do anything with the API of the 3Scale we are going to generate an access token to grant ourselves the capability to send api calls to 3scale from an external source, with the specific permissions so that even-if that access-token get into the wrong hands - it can only do limited access before it’ll be deprecated by the 3Scale manager.
Navigate to:

**Gears icon** ⇒ **Personal** ⇒ **Tokens** ⇒ **add access token** ⇒ **mark all & change permission to read & write** ⇒ **Create** ⇒ **copy the access token**


<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114512395-c06b5900-9c41-11eb-8a29-1b8e5ec46f84.png">
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114512900-425b8200-9c42-11eb-9b69-35a252e94bc0.png">
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114513047-70d95d00-9c42-11eb-9415-1db85e615e95.png">
</p>

```bash
ACCESS_TOKEN=<Copy it here>
```

## Examine 3Scale API Docs

Navigate to:

**Gears icon** ⇒ **Integrate** ⇒ **3Scale API Docs**

As you can see there are many APIs that 3Scale exposes; It actually enables you to do everything that you can do from the UI but from another source - a management server with automatic scripts, Tekton/Jenkins pipelines are just some of the options that 3Scale management teams use.

Every method can be used from that page as well for testing, and it also generates a cURL command that is re-usable with the parameters that you put into it so you don't need to generate it yourself. 

Here you can see an example of the usage of that capability to list the existing backends in 3Scale (don’t worry about the plain-text exposed token in the picture - it doesn’t exist anymore);

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114514923-7c2d8800-9c44-11eb-8a2b-6b4162b2b4ec.png">
</p>

Full disclosure: most of the commands in that lab were copy-pasted & edited to enable more flexibility & visibility using jq to parse the JSON responses.

## Generate new Backend & Method
Let’s send a request to get a list of the already existing backends;

```bash
curl -k -s -X GET "https://${API_URL}/admin/api/backend_apis.json?access_token=${ACCESS_TOKEN}&per_page=500" | jq
```

Add the SOAP API application as a new backend;

```bash
curl -k  -X POST "https://${API_URL}/admin/api/backend_apis.json" -d "access_token=${ACCESS_TOKEN}&name=SOAP+app+backend&system_name=soap_app_backend&private_endpoint=http://stores-soap.${OCP_USERNAME}-soap-api.svc.cluster.local:8080"
```

Navigate to the dashboard (in the UI) and see that you can see the new backend in the nav-bar options. Another way to verify that it worked will be to send the previous command again to list the existing backends - like we would do using an automated process;

```bash
curl -k -s -X GET "https://${API_URL}/admin/api/backend_apis.json?access_token=${ACCESS_TOKEN}&per_page=500" | jq
```

Notice the id of the new backend - every backend gets a unique id when it gets created and is been used by the system to tag everything related to that backend using that id;

Extract it for later commands.

```bash
SOAP_BACKEND_ID=`curl -k -s "https://${API_URL}/admin/api/backend_apis.json?access_token=${ACCESS_TOKEN}&per_page=500" | jq -r '.[]' | grep -B3 $OCP_USERNAME-soap-api  | grep id | awk '{print $2}' | sed 's/,//'`

echo ${SOAP_BACKEND_ID}
```

Another important thing to mention is that every backend created get’s a default metric id (Hits) for it’s backends - it can be overridden by a custom metric but this is the default one and it’s getting attached to every new-method if not mentioned otherwise; Let’s extract it as well.

First see what I'm talking about:

```bash
curl -s -k  -X GET "https://${API_URL}/admin/api/backend_apis/${SOAP_BACKEND_ID}/metrics.json?access_token=${ACCESS_TOKEN}&page=1&per_page=500" | jq
```

Now let’s extract it to the environment variable as well.

```bash
SOAP_METRIC_ID=`curl -s -k  -X GET "https://${API_URL}/admin/api/backend_apis/${SOAP_BACKEND_ID}/metrics.json?access_token=${ACCESS_TOKEN}&page=1&per_page=500" | jq -r .metrics[0].metric.id`

echo ${SOAP_METRIC_ID}
```

For the final stage of that exercises - create a new method to the new backend;

```bash
curl -s -k -X POST "https://${API_URL}/admin/api/backend_apis/${SOAP_BACKEND_ID}/metrics/${SOAP_METRIC_ID}/methods.json" -d "access_token=${ACCESS_TOKEN}&friendly_name=get+all+stores&system_name=get_all_stores&unit=hit"
```

List the existing methods of that backend to verify it worked:

```bash
curl -k -s -X GET "https://${API_URL}/admin/api/backend_apis/${SOAP_BACKEND_ID}/metrics/${SOAP_METRIC_ID}/methods.json?access_token=${ACCESS_TOKEN}&page=1&per_page=500" | jq
```

Navigate to the new backend in the UI and then to the methods and see the one we just created.

# Conclusion
It's just an example but as you can see that you can do basically every operation of 3Scale via the management API while maintaining control over which user can access which method/backend/product etc (security-wise). 

Common use case that we see a lot with our customers is integration of 3Scale management API in the CI/CD pipelines in such a manner that for every application that exposes api - a 3Scale configuration creates everything regarding 3Scale automatically - and the developers can consume it like every other cloud self-service products.

