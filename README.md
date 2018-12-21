[![FIWARE Banner](https://fiware.github.io/tutorials.XACML-Access-Rules/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/security.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.XACML-Access-Rules.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
<br/>
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial introduces an additional security generic enabler - **Authzforce** and adds fine grained control to the security rules generated by **Keyrock**. Access to the  entities created in the
[previous tutorial](https://github.com/Fiware/tutorials.PEP-Proxy) is now configured and controlled using XACML rules - this creates flexible rulesets which can be uploaded and reinterpreted on the fly so complex business rules can be created and changed according to current circumstances.

The tutorial discusses code showing how to integrate **Authzforce** within a web
application and demonstrates examples of **Authzforce** XACML Server-PDP interactions. [cUrl](https://ec.haxx.se/) commands are used to show the interactions between generic enablers, some **Keyrock** GUI functions are also required.
[Postman documentation](https://fiware.github.io/tutorials.XACML-Access-Rules/)is available.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://www.getpostman.com/collections/66d8ba3abaf7319941b1)

# Contents

TBD

# Ruleset Based Permissions

> "Say: Come, I will rehearse what *Allah* hath prohibited you from:
>
> * Join not anything as equal with *Him*
> * Be good to your parents
> * Kill not your children on a plea of want - *We* provide sustenance for you and for them
> * Come not nigh to shameful deeds. Whether open or secret
> * Take not life, which *Allah* hath made sacred, except by way of justice and law
>
> thus doth *He* command you, that ye may learn wisdom."
>
> — Quran 6.151, Sūrat al-Anʻām


TBD

## What is XACML



# Prerequisites

## Docker

To keep things simple all components will be run using
[Docker](https://www.docker.com). **Docker** is a container technology which
allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions
    [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions
    [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions
    [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker
applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Identity-Management/master/docker-compose.yml)
is used configure the required services for the application. This means all
container services can be brought up in a single command. Docker Compose is
installed by default as part of Docker for Windows and Docker for Mac, however
Linux users will need to follow the instructions found
[here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should
download [cygwin](http://www.cygwin.com/) to provide a command-line
functionality similar to a Linux distribution on Windows.

# Architecture

This application adds OAuth2-driven security into the existing Stock Management
and Sensors-based application created in
[previous tutorials](https://github.com/Fiware/tutorials.IoT-Agent/) by using
the data created in the first
[security tutorial](https://github.com/Fiware/tutorials.Identity-Management/)
and reading it programmatically. It will make use of three FIWARE components -
the [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/),the
[IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/)
and integrates the use of the
[Keyrock](https://fiware-idm.readthedocs.io/en/latest/) Generic enabler. Usage
of the Orion Context Broker is sufficient for an application to qualify as
_“Powered by FIWARE”_.

Both the Orion Context Broker and the IoT Agent rely on open source
[MongoDB](https://www.mongodb.com/) technology to keep persistence of the
information they hold. We will also be using the dummy IoT devices created in
the [previous tutorial](https://github.com/Fiware/tutorials.IoT-Sensors/).
**Keyrock** uses its own [MySQL](https://www.mysql.com/) database.

Therefore the overall architecture will consist of the following elements:

-   The FIWARE
    [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which
    will receive requests using
    [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
-   The FIWARE
    [IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/)
    which will receive southbound requests using
    [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) and convert
    them to
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    commands for the devices
-   FIWARE [Keyrock](https://fiware-idm.readthedocs.io/en/latest/) offer a
    complement Identity Management System including:
    -   An OAuth2 authentication system for Applications and Users
    -   A site graphical frontend for Identity Management Administration
    -   An equivalent REST API for Identity Management via HTTP requests

-   FIWARE [Authzforce](https://fiware-pep-proxy.rtfd.io/) is a XACML Server providing an interpretive Policy Decision Point (PDP)
    access to the **Orion** and/or **IoT Agent** microservices
-   FIWARE [Wilma](https://fiware-pep-proxy.rtfd.io/) is a PEP Proxy securing
    access to the **Orion** microservices, it requests authorisation decisions from **Authzforce**
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the **Orion Context Broker** to hold context data information
        such as data entities, subscriptions and registrations
    -   Used by the **IoT Agent** to hold device information such as device URLs
        and Keys
-   A [MySQL](https://www.mysql.com/) database :
    -   Used to persist user identities, applications, roles and permissions
-   The **Stock Management Frontend** does the following:
    -   Displays store information
    -   Shows which products can be bought at each store
    -   Allows users to "buy" products and reduce the stock count.
    -   Allows authorized users into restricted areas, it requests authoriation decisions from **Authzforce**
-   A webserver acting as set of
    [dummy IoT devices](https://github.com/Fiware/tutorials.IoT-Sensors) using
    the
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    protocol running over HTTP - access to certain resources is restricted.

Since all interactions between the elements are initiated by HTTP requests, the
entities can be containerized and run from exposed ports.

![](https://fiware.github.io/tutorials.XACML-Access-Rules/img/architecture.png)

The specific architecture of each section of the tutorial is discussed below.

## Keyrock Configuration

```yaml
  keyrock:
    image: fiware/idm
    container_name: fiware-keyrock
    hostname: keyrock
    networks:
      default:
        ipv4_address: 172.18.1.5
    depends_on:
      - mysql-db
      - authzforce
    ports:
      - "3005:3005"
    environment:
      - DEBUG=idm:*
      - DATABASE_HOST=mysql-db
      - IDM_DB_PASS_FILE=/run/secrets/my_secret_data
      - IDM_DB_USER=root
      - IDM_HOST=http://localhost:3005
      - IDM_PORT=3005
      - IDM_ADMIN_USER=alice
      - IDM_ADMIN_EMAIL=alice-the-admin@test.com
      - IDM_ADMIN_PASS=test
      - IDM_PDP_LEVEL=advanced
      - IDM_AUTHZFORCE_ENABLED=true
      - IDM_AUTHZFORCE_HOST=authzforce
      - IDM_AUTHZFORCE_PORT=8080
    secrets:
      - my_secret_data
```

The `keyrock` container is a web application server listening on a single port:

-   Port `3005` has been exposed for HTTP traffic so we can display the web page
    and interact with the REST API.


The `keyrock` container is connecting to **Authzforce** and is driven by environment variables as shown:

| Key                   | Value                                  | Description                                                                                    |
| --------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------- |
| IDM_PDP_LEVEL                 | `advanced`                           | Flag indicating that **Keyrock** should delegate PDP decisions to Authzforce |
| PEP_PROXY_AZF_PROTOCOL                 | `http`                           | Transport protocol used by the XACML Server microservice                                                                   |
| PEP_PROXY_AZF_HOST          | `authzforce`                                 | This is URL where the **Authzforce** is found users                                    |
| PEP_PROXY_AZF_PORT           | `8080`                     |   Port that **Authzforce** is listening on |

The other `keyrock` container configuration values described in the YAML file
have been described in previous tutorials


## PEP Proxy Configuration

```yaml
  orion-proxy:
    image: fiware/pep-proxy
    container_name: fiware-orion-proxy
    hostname: orion-proxy
    networks:
      default:
        ipv4_address: 172.18.1.10
    depends_on:
      - keyrock
      - authzforce
    ports:
      - "1027:1027"
    expose:
      - "1027"
    environment:
      - PEP_PROXY_APP_HOST=orion
      - PEP_PROXY_APP_PORT=1026
      - PEP_PROXY_PORT=1027
      - PEP_PROXY_IDM_HOST=keyrock
      - PEP_PROXY_HTTPS_ENABLED=false
      - PEP_PROXY_IDM_SSL_ENABLED=false
      - PEP_PROXY_IDM_PORT=3005
      - PEP_PROXY_APP_ID=tutorial-dckr-site-0000-xpresswebapp
      - PEP_PROXY_USERNAME=pep_proxy_00000000-0000-0000-0000-000000000000
      - PEP_PASSWORD=test
      - PEP_PROXY_PDP=authzforce
      - PEP_PROXY_AUTH_ENABLED=true
      - PEP_PROXY_MAGIC_KEY=1234
      - PEP_PROXY_AZF_PROTOCOL=http
      - PEP_PROXY_AZF_HOST=authzforce
      - PEP_PROXY_AZF_PORT=8080
```

The `orion-proxy` container is an instance of FIWARE **Wilma** listening on port
`1027`, it is configured to forward traffic to `orion` on port `1026`, which is
the default port that the Orion Context Broker is listening to for NGSI
Requests.

The `orion-proxy` container is delegating PDP decisions to **Authzforce** and is driven by environment variables as shown:

| Key                   | Value                                  | Description                                                                                    |
| --------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------- |
| PEP_PROXY_PDP                 | `authzforce`                           | Flag ensuring that the PEP Proxy uses Authzforce as a PDP                                                                   |
| PEP_PROXY_AZF_PROTOCOL                 | `http`                           | Flag to enable use of the XACML PDP                                                                   |
| PEP_PROXY_AZF_HOST          | `authzforce`                                 | This is URL where the **Authzforce** is found users                                    |
| PEP_PROXY_AZF_PORT           | `8080`                     |   Port that **Authzforce** is listening on |


The other `orion-proxy` container configuration values described in the YAML file
have been described in previous tutorials

## Authzforce Configuration

```yaml
  authzforce:
    image: fiware/authzforce-ce-server
    hostname: authzforce
    container_name: fiware-authzforce
    networks:
      default:
        ipv4_address: 172.18.1.12
    ports:
      - "8080:8080"
    volumes:
      - ./authzforce/domains:/opt/authzforce-ce-server/data/domains
```

The `authzforce` container is listening on port `8080`, where it receives requests to make PDP decisions. A volume has been exposed to upload a pre-configured domain so that a set of XACML rules has already been supplied.


## Tutorial Security Configuration

```yaml
tutorial:
    image: fiware/tutorials.context-provider
    hostname: tutorial
    container_name: fiware-tutorial
    networks:
        default:
            ipv4_address: 172.18.1.7
    expose:
        - "3000"
        - "3001"
    ports:
        - "3000:3000"
        - "3001:3001"
    environment:
        - "DEBUG=tutorial:*"
        - "WEB_APP_PORT=3000"
        - "KEYROCK_URL=http://localhost"
        - "KEYROCK_IP_ADDRESS=http://172.18.1.5"
        - "KEYROCK_PORT=3005"
        - "KEYROCK_CLIENT_ID=tutorial-dckr-site-0000-xpresswebapp"
        - "KEYROCK_CLIENT_SECRET=tutorial-dckr-site-0000-clientsecret"
        - "CALLBACK_URL=http://localhost:3000/login"
        - "AUTHZFORCE_ENABLED=true"
        - "AUTHZFORCE_URL=http://authzforce"
        - "AUTHZFORCE_PORT=8080"
```

The `tutorial` container is listening on two ports:

-   Port `3000` is exposed so we can see the web page displaying the Dummy IoT
    devices.
-   Port `3001` is exposed purely for tutorial access - so that cUrl or Postman
    can make UltraLight commands without being part of the same network.

The `tutorial` container is now secured by **Authforce**, and is driven by environment variables as shown:

| Key                   | Value                                  | Description                                                                                    |
| --------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------- |
| AUTHZFORCE_ENABLED                 | `true`                           | Flag to enable use of the XACML PDP                                                                   |
| AUTHZFORCE_URL          | `http://authzforce`                                 | This is URL where the **Authzforce** is found users                                    |
| AUTHZFORCE_PORT           | `8080`                     |   Port that **Authzforce** is listening on |



The other `tutorial` container configuration values described in the YAML file
have been described in previous tutorials



# Start Up

To start the installation, do the following:

```console
git clone git@github.com:Fiware/tutorials.XACML-Access-Rules.git
cd tutorials.XACML-Access-Rules

./services create
```

> **Note** The initial creation of Docker images can take up to three minutes

Thereafter, all services can be initialized from the command-line by running the
[services](https://github.com/Fiware/tutorials.XACML-Access-Rules/blob/master/services)
Bash script provided within the repository:

```console
./services start
```


> :information_source: **Note:** If you want to clean up and start over again
> you can do so with the following command:
>
> ```console
> ./services stop
> ```

### Dramatis Personae

The following people at `test.com` legitimately have accounts within the
Application

-   Alice, she will be the Administrator of the **Keyrock** Application
-   Bob, the Regional Manager of the supermarket chain - he has several store
    managers under him:
    -   Manager1
    -   Manager2
-   Charlie, the Head of Security of the supermarket chain - he has several
    store detectives under him:
    -   Detective1
    -   Detective2

| Name       | eMail                     | Password |
| ---------- | ------------------------- | -------- |
| alice      | alice-the-admin@test.com  | `test`   |
| bob        | bob-the-manager@test.com  | `test`   |
| charlie    | charlie-security@test.com | `test`   |
| manager1   | manager1@test.com         | `test`   |
| manager2   | manager2@test.com         | `test`   |
| detective1 | detective1@test.com       | `test`   |
| detective2 | detective2@test.com       | `test`   |

The following people at `example.com` have signed up for accounts, but have no
reason to be granted access

-   Eve - Eve the Eavesdropper
-   Mallory - Mallory the malicious attacker
-   Rob - Rob the Robber

| Name    | eMail               | Password |
| ------- | ------------------- | -------- |
| eve     | eve@example.com     | `test`   |
| mallory | mallory@example.com | `test`   |
| rob     | rob@example.com     | `test`   |


# Using an XACML Server

## Reading XACML Rulesets

### Authzforce - Obtain Version Information

#### :one: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/version \
  -H 'Accept: application/xml'
```

#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<productMetadata xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6" name="AuthzForce CE Server" version="8.0.1" release_date="2017-12-05" uptime="P0Y0M0DT0H8M47.642S" doc="https://authzforce.github.io/fiware/authorization-pdp-api-spec/5.2/"/>
```

### List all domains


#### :two: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/domains
```




#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<resources xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6">
    <ns2:link rel="item" href="gQqnLOnIEeiBFQJCrBIBDA" title="gQqnLOnIEeiBFQJCrBIBDA"/>
</resources>
```

### Read a single domain

#### :three: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA
```

#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<domain xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6">
    <properties externalId="tutorial-dckr-site-0000-xpresswebapp"/>
    <childResources>
        <ns2:link rel="item" href="/properties" title="Domain properties"/>
        <ns2:link rel="item" href="/pap" title="Policy Administration Point"/>
        <ns2:link rel="http://docs.oasis-open.org/ns/xacml/relation/pdp" href="/pdp" title="Policy Decision Point"/>
    </childResources>
</domain>
```

### List all PolicySets available within a Domain

#### :four: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pap/policies
```

#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<resources xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6">
    <ns2:link rel="item" href="f8194af5-8a07-486a-9581-c1f05d05483c"/>
    <ns2:link rel="item" href="root"/>
</resources>
```

### List the available revisions of a PolicySet

#### :five: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pap/policies/f8194af5-8a07-486a-9581-c1f05d05483c
```



#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<resources xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6">
    <ns2:link rel="item" href="2"/>
    <ns2:link rel="item" href="1"/>
</resources>
```




### Read a single version of a PolicySet

#### :six: Request

```console
curl -X GET \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pap/policies/f8194af5-8a07-486a-9581-c1f05d05483c/2
```

#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ns3:PolicySet xmlns="http://authzforce.github.io/rest-api-model/xmlns/authz/5" xmlns:ns2="http://www.w3.org/2005/Atom" xmlns:ns3="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns4="http://authzforce.github.io/core/xmlns/pdp/6.0" xmlns:ns5="http://authzforce.github.io/pap-dao-flat-file/xmlns/properties/3.6" PolicySetId="f8194af5-8a07-486a-9581-c1f05d05483c" Version="2" PolicyCombiningAlgId="urn:oasis:names:tc:xacml:3.0:policy-combining-algorithm:deny-unless-permit">
    <ns3:Description>Policy Set for application tutorial-dckr-site-0000-xpresswebapp</ns3:Description>
    <ns3:Target/>
    <ns3:Policy PolicyId="security-role-0000-0000-000000000000" Version="1.0" RuleCombiningAlgId="urn:oasis:names:tc:xacml:3.0:rule-combining-algorithm:deny-unless-permit">
        <ns3:Description>Role security-role-0000-0000-000000000000 from application tutorial-dckr-site-0000-xpresswebapp</ns3:Description>
        <ns3:Target>
           ...etc
        </ns3:Target>
        <ns3:Rule RuleId="alrmbell-ring-0000-0000-000000000000" Effect="Permit">
            ...etc
        </ns3:Rule>
        ..etc
    </ns3:Policy>
</ns3:PolicySet>
```


## Requesting Policy Decisions

### Permit Access to a Resource

#### :seven: Request

```console
curl -X POST \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pdp \
  -H 'Content-Type: application/xml' \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<Request xmlns="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" CombinedDecision="false" ReturnPolicyIdList="false">
   <Attributes Category="urn:oasis:names:tc:xacml:1.0:subject-category:access-subject">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:2.0:subject:role" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">managers-role-0000-0000-000000000000</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:resource">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:resource:resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">tutorial-dckr-site-0000-xpresswebapp</AttributeValue>
      </Attribute>
      <Attribute AttributeId="urn:thales:xacml:2.0:resource:sub-resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">/app/price-change</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:action">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:action:action-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">GET</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:environment" />
</Request>'
```

#### Response

### Deny Access to a Resource

#### :eight: Request

```console
curl -X POST \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pdp \
  -H 'Content-Type: application/xml' \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<Request xmlns="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" CombinedDecision="false" ReturnPolicyIdList="false">
   <Attributes Category="urn:oasis:names:tc:xacml:1.0:subject-category:access-subject">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:2.0:subject:role" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">security-role-0000-0000-000000000000</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:resource">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:resource:resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">tutorial-dckr-site-0000-xpresswebapp</AttributeValue>
      </Attribute>
      <Attribute AttributeId="urn:thales:xacml:2.0:resource:sub-resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">/app/price-change</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:action">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:action:action-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">GET</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:environment" />
</Request>'
```

#### Response



# PDP - Advanced Authorization


As a reminder, there are three Levels of PDP Access Control:

* Level 1: Authentication Access - Allow all actions to every signed in user and no actions to an anonymous user.
* Level 2: Basic Authorization - Check which resources and verbs the currently logged in user should have access to
* Level 3: Advanced Authorization - Fine grained control through [XACML](https://en.wikipedia.org/wiki/XACML)

Within FIWARE, level 3 access control can be provided by adding **Authzforce** to the existing security microservices (IDM and PEP Proxy) of the infrastructure. Access control levels 1 and 2 have been covered in [previous tutorials](https://github.com/Fiware/tutorials.Securing-Access) and can be fulfilled using **Keyrock** alone or with or without an associated PEP Proxy.

## Advanced Authorization

Advanced Authorization is able to deal with complex rulesets. Permissions are no longer merely based on a fixed role, resource and an action, but can be extended as necessary.

For example users in role `XXX` can access URL **starting with** `YYY` provided that the HTTP verb **is either**  `GET`, `PUT` or `POST`. Such users may also `DELETE` **provided that** they were the creator in the first place.

Within the tutorial programatic example we are using our own trusted instance of **Keyrock** - once a user has signed in and obtained an `access_token`, the `access_token` can be stored in session and used to retrieve user details on demand. All access to the Orion context broker is hidden behind a PEP Proxy. Whenever a request is made to Orion, the `access_token` is passed in the header of the request, and the PEP proxy handles the decision to whether to execute the request.

### User Obtains an Access Token

#### :nine: Request

```console
curl -X POST \
  http://localhost:3005/oauth2/token \
  -H 'Accept: application/json' \
  -H 'Authorization: Basic dHV0b3JpYWwtZGNrci1zaXRlLTAwMDAteHByZXNzd2ViYXBwOnR1dG9yaWFsLWRja3Itc2l0ZS0wMDAwLWNsaWVudHNlY3JldA==' \
  -H 'Content-Type: application/x-www-form-urlencoded'
```

#### Response

### Obtain Roles and Domain

#### :one::zero: Request

```console
curl -X GET \
  'http://localhost:3005/user?access_token=1b88827586409b3b4dc67378e6b945c99b94c6cf&app_id=tutorial-dckr-site-0000-xpresswebapp&authzforce=true'
```

#### Response

### Apply a Policy to a Request

#### :one::one: Request

```console
curl -X POST \
  http://localhost:8080/authzforce-ce/domains/gQqnLOnIEeiBFQJCrBIBDA/pdp \
  -H 'Content-Type: application/xml' \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<Request xmlns="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" CombinedDecision="false" ReturnPolicyIdList="false">
   <Attributes Category="urn:oasis:names:tc:xacml:1.0:subject-category:access-subject">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:2.0:subject:role" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">managers-role-0000-0000-000000000000</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:resource">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:resource:resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">tutorial-dckr-site-0000-xpresswebapp</AttributeValue>
      </Attribute>
      <Attribute AttributeId="urn:thales:xacml:2.0:resource:sub-resource-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">/v2/entities</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:action">
      <Attribute AttributeId="urn:oasis:names:tc:xacml:1.0:action:action-id" IncludeInResult="false">
         <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">POST</AttributeValue>
      </Attribute>
   </Attributes>
   <Attributes Category="urn:oasis:names:tc:xacml:3.0:attribute-category:environment" />
</Request>'
```

#### Response



### Advanced Authorization - Sample Code

Programmatically, any Policy Execution Point consists of two parts, an oAuth request to Keyrock retrieves information about the user (such as the assigned roles) as well as the policy domain to be queried.

A second request is sent to the relevant domain endpoint within Authzforce, providing all of the information necessary for Authzforce to provide a judgement. Authzforce responds with a **permit** or **deny** response, and the decision whether to continue can be made thereafter.

```javascript
function authorizeAdvancedXACML(req, res, next, resource = req.url) {

  const keyrockUserUrl ='http://keyrock/user?access_token=' +
    req.session.access_token + '&app_id=' + clientId +
    '&authzforce=true';

  return oa
    .get(keyrockUserUrl)
    .then(response => {
      const user = JSON.parse(response);
      return azf.policyDomainRequest(
        user.app_azf_domain,
        user.roles,
        resource,
        req.method
      );
    })
    .then(authzforceResponse => {
      res.locals.authorized = authzforceResponse === 'Permit';
      return next();
    })
    .catch(error => {
      debug(error);
      res.locals.authorized = false;
      return next();
    });
}
```

The full code to supply each request to Authzforce can be found within the tutorials' [Git Repository](https://github.com/Fiware/tutorials.Step-by-Step/blob/master/context-provider/lib/azf.js) - the actual information to supply will depend on business use case - it could be expanded to include temporal information, relationships between records and so on, but in this very simple example only roles are necessary.

```javascript
const xml2js = require('xml2js');
const request = require('request');

function policyDomainRequest (domain, roles, resource, action) {
  let body =
    '<?xml version="1.0" encoding="UTF-8"?>\n' +
    '<Request xmlns="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" CombinedDecision="false" ReturnPolicyIdList="false">\n';
  // Code to create the XML body for the request is omitted
  body = body +  '</Request>';

  const options = {
    method: 'POST',
    url: 'http://authzforceUrl/authzforce-ce/domains/' + domain + '/pdp',
    headers: { 'Content-Type': 'application/xml' },
    body
  };

  return new Promise((resolve, reject) => {
    request(options, function(error, response, body) {
      let decision;
      xml2js.parseString(
        body,
        { tagNameProcessors: [xml2js.processors.stripPrefix] },
        function(err, jsonRes) {
          // The decision is found within the /Response/Result[0]/Decision[0] XPath
          decision = jsonRes.Response.Result[0].Decision[0];
        }
      );
      decision = String(decision);
      return error ? reject(error) : resolve(decision);
    });
  });
};
```

### Advanced Authorization - PEP Proxy

Applying advanced  authorization within a PEP proxy requires
very similar code to the programmatic example described above. The **Wilma** generic enabler extracts a token from the header supplied by the request and makes a request to **Keyrock** to obtain further information about the user. A PDP request is then made to **Authzforce** to decide whether to procede.

Obviously any scalable solution should also cache information about the PDP requests made and the responses to avoid making unnecessary requests.

## PDP - Advanced Authorization - Running the Example

> **Note** Five resources have been secured at level 3:
>
> -   sending the unlock door command
> -   sending the ring bell command
> -   access to the price-change area
> -   access to the order-stock area
> -   access to Orion (behind a PEP Proxy)

#### Eve the Eavesdropper

Eve has an account, but no roles in the application.

> **Note** As Eve has a recognized account, she gains full authentication
> access. This means she is able to *view* the Store page, even though her account has no roles attached.

-   From `http://localhost:3000`, log in as `eve@example.com` with the password
    `test`

##### Level 3 : Advanced Authorization Access

-   Click on any store page - access to view the page is **permitted** for any logged in users, however access to retrieve Orion data is now **denied** since Eve has no role which permits access.

-   Click on the restricted access links at `http://localhost:3000` - access is
    **denied**
-   Open the Device Monitor on `http://localhost:3000/device/monitor`
    -   Unlock a door - access is **denied**
    -   Ring a bell - access is **denied**

#### Bob The Regional Manager

Bob has the **management** role

-   From `http://localhost:3000`, log in as `bob-the-manager@test.com` with the
    password `test`

##### Level 3 : Advanced Authorization Access

-   Click on the restricted access links at `http://localhost:3000` - access is
    **permitted** - This is a management only permission
-   Open the Device Monitor on `http://localhost:3000/device/monitor`
    -   Unlock a door - access is **denied**. - This is a security only
        permission
    -   Ring a bell - access is **permitted** - This is permitted to management
        users

#### Charlie the Security Manager

Charlie has the **security** role

-   From `http://localhost:3000`, log in as `charlie-security@test.com` with the
    password `test`

##### Level 3: Advanced Authorization Access

-   Click on the restricted access links at `http://localhost:3000` - access is
    **denied** - This is a management only permission
-   Open the Device Monitor on `http://localhost:3000/device/monitor`
    -   Unlock a door - access is **permitted** - This is a security only
        permission
    -   Ring a bell - access is **permitted** - This is permitted to security
        users

---

## License

[MIT](LICENSE) © FIWARE Foundation e.V.
