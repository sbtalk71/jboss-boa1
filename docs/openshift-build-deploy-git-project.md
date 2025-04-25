# Building and deploying an application to Openshift Sandbox from GIT Project

1. login to Openshift Sandbox at  https://console.redhat.com/openshift/sandbox
2. Create a Project by some name
3. click add
4. select template `Git Repository import from git`
5. provide the git project url e.g. https://github.com/sbtalk71/openshift-app
6. Customize build properties if required
7. My project is spring boot project version **3.0.10**. Openshift as on today **18-04-2025** uses maven 3.6.2. Spring boot higher versions use maven 3.6.3 onwards. If you use higher version of Spring Boot, you may get build failure due to maven-complier-plug or other maven plugins.
8. Given below is a sample `Dockerfile`

```sh
	FROM maven:3.9.9-amazoncorretto-17
	WORKDIR /app
	COPY pom.xml .
	RUN mvn dependency:go-offline
	COPY src ./src
	RUN mvn clean package
	CMD java -jar target/greet-service.jar

```
9. After the project is built successfully, click on Topology on left panel. Now you can see the details of you deployment and pod etc.
10. Use the route to access the application after you verify the pod is running