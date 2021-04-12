# Exercise 1
## Getting Familiar with APIs
### V1 Date: 05/2020
### V2 Date: 03/2021

# Setup & Examine the lab environment
## Configure Environment Variables
To make life easier we will configure some environment variables that correspond to the lab.
The variables' values are dedicated for each participant and are provided by the instructor.

```
echo "export OCP_USERNAME=" >> ~/environ.sh
echo "export OCP_PASSWORD=openshift" >> ~/environ.sh
echo "export API_USERNAME=” >> ~/environ.sh
echo "export API_PASSWORD=admin" >> ~/environ.sh
echo "export OCP_CONSOLE_URL=" >> ~/environ.sh
echo "export OCP_API_URL=$(oc status | grep server | cut -d ‘ ‘ -f 6)” >> ~/environ.sh
echo "export API_URL=" >> ~/environ.sh
echo "export API_NAMESPACE=3scale-api0" >> ~/environ.sh
echo "export OCP_WILDCARD_DOMAIN=" >> ~/environ.sh
**source ~/environ.sh**
```


