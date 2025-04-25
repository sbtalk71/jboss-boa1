# Download JBoss EAP Container Image for Building your containerized app

1. go to https://catalogue.redhat.com and login with your credentials
2. Search JBoss Application Platform 7
3. When the page loads, click on the certification tab
4. select the image you want to use (e.g. JBoss EAP 7.4 with OpenJDK 17) and click on it
5. click on 'get this image' tab
6. use 'Using Red Hat Login' tab
7. follow the given steps to download the image:

```sh
docker login registry.redhat.io
Username: {REGISTRY-SERVICE-ACCOUNT-USERNAME}
Password: {REGISTRY-SERVICE-ACCOUNT-PASSWORD}
Login Succeeded!

docker pull registry.redhat.io/jboss-eap-7/eap74-openjdk17-runtime-openshift-rhel8:7.4.21-5
```

### A Sample Dockerfile to build an application image 

1. **Folder Structure**
	
	```sh
	|
	 App-Root
		|
		Dockerfile
		Test.war
	```
2. **Dockerfile**
	
	```sh
	FROM registry.redhat.io/jboss-eap-7/eap74-openjdk17-openshift-rhel8:7.4.21-6

	# Copy war to deployments folder
	COPY Test.war $JBOSS_HOME/standalone/deployments/

	# User root to modify war owners
	USER root

	# Modify owners war
	RUN chown jboss:jboss $JBOSS_HOME/standalone/deployments/Test.war

	# Important, use jboss user to run image
	USER jboss
	
	```
	
3. **Docker build Command**
	
	```sh
	docker build -t <your-repo-name>/JbossEAP74-App:1.0 .
	
	```

4. **Run the Image**

	```sh
	docker run --rm --name jbosseap7 -p8080:8080 -it sbtalk71/JbossEAP74-App:1.0
	
	```
5. **Test the Application**
 Open the URL in browser: http://localhost:8080/Test/Hello.jsp