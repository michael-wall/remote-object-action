## Introduction ##
- A simple Object Action Client Extension with OAuth.
- The Client Extension and the Spring Boot App are in separate repositories.
    - The Client Extension is in this repository.
    - The Spring Boot App is in a separate Liferay Workspace repository: https://github.com/michael-wall/remote-object-action-new-app
- The expectation is that the Spring Boot App is deployed outside of Liferay PaaS and Docker.
- For test purposes the setup assumes a local Liferay and local Spring Boot App, but the Spring Boot App can be remote as long as the hostnames are resolvable in both directions etc.
- **Customer is responsible for provisioning the Spring Boot App e.g. with a custom CI/CD pipeline and managing availability and scale etc.**
- **Customer can update the code within the Object Action class and redeploy the Spring Boot App without needing to do a Liferay PaaS build.**
- **Customer is responsible for Blue / Green deployment or similar to avoid downtime while the new Spring Boot App is being deployed.**

## Self Hosted / Local Setup Steps ##
- Update the following configuration (as needed) for the target environment:
  - mw-object-action CX client-extension.yaml:
    - **.serviceAddress: mw.com:58081** and **.serviceScheme: http** are the hostname, port and protocol of the Spring Boot App. Liferay users these to connect to the Object Action endpoint.
  - mw-object-action Spring Boot App module application-default.properties:
    - **server.port=58081** is the Spring Boot App port and it MUST match the port from **.serviceAddress: mw.com:58081** above.
    - **com.liferay.lxc.dxp.domains=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.mainDomain=localhost:8080** is the Liferay environment hostname and port.
    - **com.liferay.lxc.dxp.server.protocol=http** is the Liferay environment protocol.
        - These com.liferay.lxc.dxp... properties are required even if running everything outside of Liferay PaaS or Liferay SaaS.
- Build the Client Extension with a regular gradle build command.
- CX LUFFA ZIP (mw-object-action.zip): Copy the CX LUFFA ZIP from remote-object-action\client-extensions\mw-object-action\dist to the local Liferay DXP osgi/extensions folder of the running Liferay
- This should generate logging like this:
  - OAuth 2 application with external reference code mw-spring-boot-oauth-app-user-agent and company ID 40473633903803 has client ID ..............................
-Build the Spring Boot App Jar with a regular gradle build command.
- Spring Boot Jar (com.mw.object.action-1.0.0.jar): Copy the Spring Boot Jar from remote-object-action-new-app\remote-object-action-new-app\modules\mw-object-action\build\libs out of the workspace
- Note the Liferay server must be running BEFORE the Spring Boot App is started, otherwise it won't start.
- Run java -jar com.mw.object.action-1.0.0.jar to Start the Spring Boot App and confirm it starts as expected.
  - java -jar build\libs\mw-object-action.jar
  - Note that a pre-existing environment specific application.properties file the Spring Boot App artifact can be injected in during startup to manage the environment specific com.liferay.lxc.dxp... properties with the following syntax:
      - java -jar build\libs\mw-object-action.jar **--spring.config.name=application-dev**
- Verify connectivity to the public /ready GET endpoint from the Liferay server e.g. using curl.
- Add an 'On After Add' Object Action to An Object and select the following from the Action > Then dropdown and Save the Object Action:
  - object-action-executor[function#mw-object-action]
- Create an Object Record and check the logs of the Spring Boot application, you should see a blob of JSON as well as the JWT Claims, JWT ID and JWT Subject in the Spring Boot App log output.

## Liferay PaaS Setup Steps ##
- **These steps assume the BETA feature flag to enable the Jenkins CX CI/CD pipeline is not enabled for the Liferay PaaS Project.**
- For each Liferay PaaS environment, generate the environment specific LUFFA file based on the environment specific client-extension.yaml and application.properties details from above and copy the resulting LUFFA to the appropriate liferay\configs\[ENV]\osgi\client-extensions folder.
- Deploy the resulting Liferay PaaS build to a the appropriate environment.
    - You can create a single build with the appropriate copy of the CX LUFFA file in liferay\configs\dev\osgi\client-extensions, liferay\configs\uat\osgi\client-extensions and liferay\configs\prod\osgi\client-extensions etc.
- Note the Liferay server must be running BEFORE the Spring Boot App is started, otherwise it won't start.
- Provision / start the Spring Boot App with the environment specific configuration. Confirm it starts as expected.
- Verify connectivity to the public /ready GET endpoint from the Liferay server e.g. using curl from the Liferay service shell.
- Add an 'On After Add' Object Action to An Object and select the following from the Action > Then dropdown and Save the Object Action:
  - object-action-executor[function#mw-object-action]
- Create an Object Record and check the logs of the Spring Boot application, you should see a blob of JSON as well as the JWT Claims, JWT ID and JWT Subject in the Spring Boot App log output.

## Setup Notes ##
- The Dockerfile and LCP.json files are mandatory for the Client Extension to compile but the included files are empty.
- The ReadyRestController.java class is not needed if not running as a Liferay PaaS Custom Service, but I left it there for testing connectivity, as it is a public GET available at /ready
- If the OAuth CX name (i.e. mw-spring-boot-oauth-app-user-agent) is changed after the CX LUFFA zip has been deployed then you will need to also change the Object Action CX name (i.e. mw-object-action) and do build and deployment of the CX LUFFA, restart the new Spring Boot App and remap the action...
   - This is required because the mapping to the old OAuth CX name doesn't change even if you delete the Object Action and recreate it.
- The Spring Boot App has some Liferay dependencies in particular BaseRestController.java from com.liferay.client.extension.util.spring.boot3. See the specific method signature of the POST method:
```
public ResponseEntity<String> post(@AuthenticationPrincipal Jwt jwt, @RequestBody String json)
```
- As a result the Spring Boot App requires the Liferay DXP (defined in the **com.liferay.lxc.dxp...** properties) to be running when it is started, otherwise the following errors will occur:
```
- 2025-07-04T15:59:36.736+01:00  WARN 11592 --- [           main] c.l.c.e.u.s.b.client.LiferayOAuth2Util   : Unable to get client ID: Connection refused: getsockopt: localhost/127.0.0.1:8080
2025-07-04T15:59:37.391+01:00  WARN 11592 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration': Unsatisfied dependency expressed through method 'setFilterChains' parameter 0: Error creating bean with name 'securityFilterChain' defined in class path resource [com/liferay/client/extension/util/spring/boot3/LiferayOAuth2ResourceServerEnableWebSecurity.class]: Failed to instantiate [org.springframework.security.web.SecurityFilterChain]: Factory method 'securityFilterChain' threw exception with message: Error creating bean with name 'jwtDecoder' defined in class path resource [com/liferay/client/extension/util/spring/boot3/LiferayOAuth2ResourceServerEnableWebSecurity.class]: Failed to instantiate [org.springframework.security.oauth2.jwt.JwtDecoder]: Factory method 'jwtDecoder' threw exception with message: Couldn't retrieve remote JWK set: Connection refused: getsockopt
```

## OAuth2 Administration > MW Spring Boot OAuth App User Agent ##
- The **.serviceAddress: mw.com:58081** and **.serviceScheme: http** values can be manually updated through the Liferay GUI > Control Panel > OAuth2 Administration > MW Spring Boot OAuth App User Agent.
- The values are concatenated into the Website URL field.

## client-extension.yaml ##
- The Client Extension project (and the Spring Boot App module) can contain multiple Mircoservice Client Extensions of different types.
- These Client Extensions can all share the same OAuth Client Extension but ideally you should consider having one per Client Extension with the specific scopes limited only to what is needed by that Client Extension.
- Additional Object Action client-extension.yaml properties:
 - Use allowedObjectDefinitionNames to restrict the Object Action Client Extensions to be visible on one or more specific Objects rather than all Objects. If the Client Extension has business logic tied to a specific Object then it can be restricted to be available for that Object only. See the Liferay Learn article below for sample usage.
- Use dxp.lxc.liferay.com.virtualInstanceId to restrict the Object Action Client Extension to a specific Virtual Instance. See the Liferay Learn article below for sample usage.

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

## Sample Object Action that uses the Liferay headless REST APIs ##
- Get the latest Client Extensions sample Workspace here:
```
curl -o com.liferay.sample.workspace-latest.zip https://repository.liferay.com/nexus/service/local/artifact/maven/content\?r\=liferay-public-releases\&g\=com.liferay.workspace\&a\=com.liferay.sample.workspace\&\v\=LATEST\&p\=zip
```
- See the examples here: client-extensions\liferay-sample-etc-spring-boot\src\main\java\com\liferay\sample
    - ObjectAction2RestController.java

## Notes ##
- This is a ‘proof of concept’ that is being provided ‘as is’ without any support coverage or warranty.

## Reference ##
- https://learn.liferay.com/w/dxp/liferay-development/integrating-microservices
- https://learn.liferay.com/w/dxp/liferay-development/integrating-microservices/object-action-yaml-configuration-reference
