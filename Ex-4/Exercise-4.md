# Exercise 4 - External, Self Managed APICast
### V1 Date: 05/2020
### V2 Date: 03/2021

# APICast - Self Managed, External 
## Intro
As we already see in the presentation, the APICasts can be deployed on top of Openshift, in a centralized deployment (i.e. APICast & AMP are all deployed together on the same Namespace & serve all of our products) or either in a distributed, self-managed deployment option (i.e. the APICasts are deployed separately, dedicated to each project, manageable by the developers of the API, in their own namespace).

We actually already saw the distributed option; Our entire lab environment is deployed in that exact manner - each participant has its own ${user}-gw project that contains dedicated-APICasts for that specific participant.   

About the APICast Self managed in comparison to APICast managed, and how to deploy it on Openshift, please read the following article that I made specifically about that topic - [link](https://medium.com/@tamber/3scale-mini-guide-apicast-self-managed-for-on-premise-deployments-e2ef53313c8).

In the following exercise we are going to do something a bit more advanced - we will deploy APICast on a containerized environment (using podman) which is external to the Openshift platform.   

## Generate Access Token for our new APICast
The APICast requires an access token in order to communicate with the AMP - they are communicating using 3Scale API calls, and the AMP needs to recognize whoever reaches to it.

The token should have Read-Only permissions — Only the AMP should make changes to the APICast and not vice-versa.

Use the gear icon on your 3Scale page 

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114509599-5ef5bb00-9c3e-11eb-975d-70a143d47a0f.png">
</p>

**Gears icon** ⇒ **Personal** ⇒ **Tokens** ⇒ **add access token** ⇒ **Name it “Read Only Access Token for APICast”** ⇒  **mark only “Account Management API” & Read Only** ⇒ **Create** ⇒ **copy the access token**


<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114509773-995f5800-9c3e-11eb-920e-25e858eb102b.png">
</p>

Copy the access token to a variable;

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114509995-d4618b80-9c3e-11eb-8549-53f204b95af5.png">
</p>

```bash
APICast_ACCESS_TOKEN=<Copy it here>
```

## Expose & Edit Backend
In order to make sure our external APICast can reach the relevant backend API application, which runs on OCP, we are required to expose it and update our backend on the 3Scale dashboard. 

Obviously exposing the API application de-facto means that it is reachable even without the APICast - but it can be further restricted by implementation of Kubernetes network policies that verifies the source IP + other parameters, or either using Service Mesh mtls feature, etc.  This is out of scope of that course, I just want to show you practically how we even deploy & integrate our external APICast with the AMP in such deployment.

First, using the oc binary, run the following command;
```bash
oc expose svc/catalog-service -n  ${OCP_USERNAME}-coolstore
oc get route -n  ${OCP_USERNAME}-coolstore | grep catalog-service | awk '{print $2}'

# Copy the route's hostname
```

In 3Scale, navigate to the coolstore backend’s dashboard, and press the “edit” button, there - paste the route’s hostname from the previous command; 

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114510284-27d3d980-9c3f-11eb-8e1f-5853e43fa920.png">
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114510401-505bd380-9c3f-11eb-9092-119b18358f1c.png">
</p>


# Deploy APICast on your machine
[APICast Github Project](https://github.com/3scale/APIcast)

```bash
sudo docker run --name apicast -d -p 8080:8080 --network host -e THREESCALE_PORTAL_ENDPOINT=https://${APICast_ACCESS_TOKEN}@${API_URL} quay.io/3scale/apicast:master
sudo docker ps -a
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114510962-fb6c8d00-9c3f-11eb-873c-05c867d4e079.png">
</p>

### Switches Explainer
* **--network host**: Runs the container and attaches it’s network interface to the host’s public interface for exposure.
* **-d or --detach**: Runs the container in the background and prints the container ID. When it is not specified, the container runs in the foreground mode and you can stop it using CTRL + c. When started in the detached mode, you can reattach to the container with the docker attach command, for example, docker attach apicast.
 
* **-p or --publish**: Publishes a container’s port to the host. The value should have the format <host port="">:<container port="">, so -p 80:8080 will bind port 8080 of the container to port 80 of the host machine. For example, the Management API uses port 8090, so you may want to publish this port by adding -p 8090:8090 to the docker run command.
 
* **-e or --env**: Sets environment variables.

```bash
sudo docker exec -it apicast hostname
# Copy it 
```

## Update Product
In 3Scale, navigate to the Coolstore product’s page;

**Coolstore API** ⇒ **Integration** ⇒ **Settings**

Make sure the *Self Managed* option is marked and update the staging url to the hostname of your station as we extracted in the previous command (in which we made sure that the APICast container is listening on).

Notice that we exposed it as http on port 8080, so make sure to change the *“https”* to *“http”* in the staging public base url and include the port number as well; 

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114511579-c3b21500-9c40-11eb-8f3c-257ff810bb55.png">
</p>

Press **“Update product”** by the end of the page, go to the **configuration** page and promote the staging APICast for the test. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114511807-0ecc2800-9c41-11eb-8207-9cf0da2dc30b.png">
</p>

## Test

```bash
curl `hostname`:8080/catalog/products?user_key=${USER_KEY}
```
<p align="center">
  <img src="https://user-images.githubusercontent.com/60185557/114511954-3a4f1280-9c41-11eb-84ab-a9b292abee28.png">
</p>


