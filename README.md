# Single Sign-on for Camunda BPM Webapp on Wildfly/JBoss AS7 (Container-based Authentication)

This project adds Single Sign On (SSO) support to the [Camunda BPM Webapp](https://docs.camunda.org/manual/latest/webapps/), which contains Tasklist, Cockpit and Admin.
Fortunately, application servers can do the actual authentication of a user before a request is forwarded to the application.
The only thing that needs to be done inside the Camunda REST API, is to take the user id and optionally also the group ids provided by the container through the Servlet API and put them into the Servlet session of the REST API.
Thats why we also call this Container-based Authentication.

As a particular example, this project shows how to do SSO with keycloak and Wildfly.
However, the [Container-based Authentication Filter](src/main/java/de/novatec/bpm/webapp/impl/security/auth/ContainerBasedUserAuthenticationFilter.java)
is only using the standard Servlet and Java Security APIs.
Therefore it works exactly the same on all Servlet containers and with any authentication mechanism supported by the container.
For example the [fork for Single Sign-on on Weblogic](https://github.com/camunda-consulting/camunda-sso-weblogic/) uses the same Java code.

There are two variations of the Authentication Filter:
One [takes the user's groups from the Camunda IdentityService](src/main/java/de/novatec/bpm/webapp/impl/security/auth/ContainerBasedUserAuthenticationFilter.java)
and requires the keycloak adapter or another identity provider.
The other one [takes the groups from the container](src/main/java/de/novatec/bpm/webapp/impl/security/auth/ContainerBasedUserAndGroupsAuthenticationFilter.java)
and leverages the user roles provided by the identity provider.

The project also shows how to configure the Camund Webapp in a way that allows for smooth updates to future Camunda BPM versions.
The [config-processor-maven-plugin](https://github.com/lehphyro/maven-config-processor-plugin)
helps to gently modify the original deployment decriptors
[web.xml](src/assembly/web.updates.xml),
[jboss-web.xml](src/assembly/jboss-web.updates.xml)
and [jboss-deployment-structure.xml](src/assembly/jboss-deployment-structure.updates.xml)
provided inside Camunda binary packages.


## Documentation

### Problem

The Camunda BPM Webapp has to be secured.

### Business Requirements

* Login once via a SSO mechanism
* Security has to be implemented by standard JBoss configuration

### Technical Requirements

* Keycloak, or use the Dockerfile provided in keycloak-demo-server/Dockerfile
* JBoss AS7 / Widlfly / EAP

### Solution

* Add Keycloak openId connect adapter to JBoss
* Configure Camunda-Webapp

### Acceptance tests

* Keycloak user can login into CamundaBPM
* All GUIs and REST APIs are only accessible via Keycloak Login

## Get started

### Configuration Keycloak

We need a keycloak with configured domain (for example 'demo') and users/roles for camunda.

You can use the Dockerfile located in keycloak-demo-server/, this starts a keycloak server with preconfigured domain 'demo' and a user 'demo' with password 'notdemo' who is in roles 'camunda-admin' and 'management', start it for example with
```{r, engine='bash', count_lines}
cd keycloak-demo-server
docker build -t keycloak-demo-server .
docker run --rm -d -p 8081:8080 keycloak-demo-server
```

### JBoss Configuration

Follow the instructions in [Keycloak Manual](http://www.keycloak.org/docs/3.0/securing_apps/topics/oidc/java/jboss-adapter.html).
It suffices to download and extract the adapter and call the appropriate jboss-cli script.

### Camunda Webapp Configuration
For getting the authentication and authorization from Keycloak, it is needed to modify the camunda webapp.

We need to:

- modify web.xml to point to our custom authorization filter class and add security constraints
- modify web.xml to configure login by KEYCLOAK
- modify jboss-web.xml to activate keycloak for security domain
- adding keycloak.json to make the adapter connect to the right server/domain

All that can be found in [Assembly scripts](src/assembly/)

## Testing

Try to log in into Camunda Webapp.
You will be redirected to the keycloak login page, login with demo/notdemo.
Now you are logged in as user demo with the Camunda groups corresponding to the keycloak roles.

## Maintainer

- Eberhard Heber (eberhardheber@novatec-gmbh.de)
- Falko Menge (falko.menge@camunda.com)

## License

Apache License, Version 2.0
