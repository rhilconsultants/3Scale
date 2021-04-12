# Exercise 1 - Getting Familiar with APIs
### V1 Date: 05/2020
### V2 Date: 03/2021



# Setup & Examine the lab environment
## Configure Environment Variables
To make life easier we will configure some environment variables.
The variables' values are dedicated for each participant and are provided by the instructor.

```bash
echo "export OCP_USERNAME=" >> ~/environ.sh
echo "export OCP_PASSWORD=openshift" >> ~/environ.sh
echo "export API_USERNAME=” >> ~/environ.sh
echo "export API_PASSWORD=admin" >> ~/environ.sh
echo "export OCP_CONSOLE_URL=" >> ~/environ.sh
echo "export OCP_API_URL=$(oc status | grep server | cut -d ‘ ‘ -f 6)” >> ~/environ.sh
echo "export API_URL=" >> ~/environ.sh
echo "export API_NAMESPACE=3scale-api0" >> ~/environ.sh
echo "export OCP_WILDCARD_DOMAIN=" >> ~/environ.sh
source ~/environ.sh
done
```

![Screenshot](file:///home/tamber/Pictures/Screenshot%20from%202021-04-12%2013-53-38.png![image](https://user-images.githubusercontent.com/60185557/114383956-cd7f3e00-9b96-11eb-9903-b2b5f30f39b5.png)


