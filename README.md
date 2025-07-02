## Introduction ##
- A simple Object Action Client Extension with OAuth running in Spring Boot, outside of Liferay PaaS or Docker.

## Setup to run locally ##
- Update the following configuration:
  - client-extension.yaml:
    - **.serviceAddress: mw.com:58081** and **.serviceScheme: http** are are the hostname, port and protocol of the Spring Boot App.
  - application-default.properties:
    - **server.port=58081** is the Spring Boot App port, must match **.serviceScheme: http** from above.
    - **com.liferay.lxc.dxp.domains=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.mainDomain=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.server.protocol=http** is the Liferay environment protocol.
- Build the Client Extension with a regular gradle build command.
- CX LUFFA ZIP (mw-object-action.zip): Copy the CX LUFFA ZIP from remote-object-action\client-extensions\mw-object-action\dist to the local Liferay DXP osgi/extensions folder of the running Liferay
- This should generate logging like this:
  - OAuth 2 application with external reference code mw-spring-boot-oauth-app-user-agent and company ID 40473633903803 has client ID ..............................
- Spring Boot Jar (mw-object-action.jar): Copy the Spring Boot Jar from remote-object-action\client-extensions\mw-object-action\build\libs out of the workspace
- Run java -jar mw-object-action.jar to Start the Spring Boot application
- Add an 'On After Add' Object Action to An Object and select the following from the Action > Then dropdown: object-action-executor[function#mw-object-action] and Save the Object Action.
- Create an Object Record and check the logs of the Spring Boot application, you should see a blob of JSON as well as the JWT Claims, JWT ID and JWT Subject.

## Setup Notes ##
- The Dockerfile is mandatory for the build to compile but the included file is empty.
- The LCP.json is mandatory for the build to compile but the included file contains only {}.
- The /ReadyRestController.java class is not needed if not running as a Liferay PaaS Custom Service, but I left it there to test. The endpoint is /ready
- If the OAuth CX name (i.e. mw-spring-boot-oauth-app-user-agent) is changed after for CX LUFFA zip has been deployed then you will need to also change the Object Action CX name (i.e. mw-object-action).
   - This is required because the mapping to the old OAuth CX name doesn't change even if you delete the Object Action and recreate it.


## Environment ##
- The module was built and tested with 2025.Q1.0 (Liferay Workspace gradle.properties > liferay.workspace.product = dxp-2025.q1.0-lts)
- JDK 21 is expected for compile time and runtime.

## Notes ##
- This is a ‘proof of concept’ that is being provided ‘as is’ without any support coverage or warranty.
