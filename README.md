## Introduction ##
- A simple Object Action Client Extension with OAuth running in a Spring Boot App, outside of Liferay PaaS or Docker.
- For test purposes the setup assumes a local Liferay and local Spring Boot App, but the Spring Boot App can be remote as long as the hostnames are resolvable in both directions etc.
- Customer is responsible for provisioning the Spring Boot App in all scenarios - local or Liferay PaaS.

## Self Hosted / Local Setup Steps ##
- Update the following configuration:
  - client-extension.yaml:
    - **.serviceAddress: mw.com:58081** and **.serviceScheme: http** are are the hostname, port and protocol of the Spring Boot App.
  - application-default.properties:
    - **server.port=58081** is the Spring Boot App port, must match the port from **.serviceAddress: mw.com:58081** above.
    - **com.liferay.lxc.dxp.domains=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.mainDomain=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.server.protocol=http** is the Liferay environment protocol.
        - The com.liferay.lxc.dxp. properties are required even if running everything outside of Liferay PaaS or Liferay SaaS.
- Build the Client Extension with a regular gradle build command.
- CX LUFFA ZIP (mw-object-action.zip): Copy the CX LUFFA ZIP from remote-object-action\client-extensions\mw-object-action\dist to the local Liferay DXP osgi/extensions folder of the running Liferay
- This should generate logging like this:
  - OAuth 2 application with external reference code mw-spring-boot-oauth-app-user-agent and company ID 40473633903803 has client ID ..............................
- Spring Boot Jar (mw-object-action.jar): Copy the Spring Boot Jar from remote-object-action\client-extensions\mw-object-action\build\libs out of the workspace
- Run java -jar mw-object-action.jar to Start the Spring Boot application and confirm it starts as expected.
- Environment specific application.properties files can be used e.g. application-dev.properties for example to to manage the environment specific com.liferay.lxc.dxp. properties, with the following syntax:
  - java -jar build\libs\mw-object-action.jar --spring.config.name=application-dev
- Verify connectivity to the public /ready GET endpoint from the Liferay server e.g. using curl.
- Add an 'On After Add' Object Action to An Object and select the following from the Action > Then dropdown and Save the Object Action:
  - object-action-executor[function#mw-object-action]
- Create an Object Record and check the logs of the Spring Boot application, you should see a blob of JSON as well as the JWT Claims, JWT ID and JWT Subject in the Spring Boot App log output.


## Liferay PaaS Setup Steps ##
- For each Liferay PaaS environment, generate the environment specific LUFFA file and copy to liferay\configs\[ENV]\osgi\client-extensions folder.
- Deploy the resulting Liferay PaaS build to an environment.
- Provision / start the Spring Boot App for the environment, running outside of Liferay PaaS and confirm it starts as expected.
- Verify connectivity to the public /ready GET endpoint from the Liferay server e.g. using curl from the Liferay service shell.
- Add an 'On After Add' Object Action to An Object and select the following from the Action > Then dropdown and Save the Object Action:
  - object-action-executor[function#mw-object-action]
- Create an Object Record and check the logs of the Spring Boot application, you should see a blob of JSON as well as the JWT Claims, JWT ID and JWT Subject in the Spring Boot App log output.


## Setup Notes ##
- The Dockerfile is mandatory for the build to compile but the included file is empty.
- The LCP.json is mandatory for the build to compile but the included file contains only {}.
- The ReadyRestController.java class is not needed if not running as a Liferay PaaS Custom Service, but I left it there to test as it is a public GET available at /ready
- If the OAuth CX name (i.e. mw-spring-boot-oauth-app-user-agent) is changed after for CX LUFFA zip has been deployed then you will need to also change the Object Action CX name (i.e. mw-object-action) and do build and deployment of the CX LUFFA and restart the new Spring Boot App.
   - This is required because the mapping to the old OAuth CX name doesn't change even if you delete the Object Action and recreate it.
- The Spring Boot App has some Liferay dependencies e.g. BaseRestController.java from com.liferay.client.extension.util.spring.boot3 for the method signature of the POST method:
```
public ResponseEntity<String> post(@AuthenticationPrincipal Jwt jwt, @RequestBody String json)
``` 

## Object Action Code ##
- The JWT available in the Object Action can be used to make requests to the the Liferay headless REST APIs (subject to the assigned scope).
- The Object Action should be a POST with the following method signature:
```
public ResponseEntity<String> post(@AuthenticationPrincipal Jwt jwt, @RequestBody String json)
```
- The response should be a HttpStatus.OK on success etc.

## Environment ##
- The module was built and tested with 2025.Q1.0 (Liferay Workspace gradle.properties > liferay.workspace.product = dxp-2025.q1.0-lts)
- JDK 21 is expected for compile time and runtime.

## Notes ##
- This is a ‘proof of concept’ that is being provided ‘as is’ without any support coverage or warranty.
