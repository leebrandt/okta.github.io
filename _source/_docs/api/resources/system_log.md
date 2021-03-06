---
layout: docs_page
title: System Log
category: management
---

# System Log API

{% api_lifecycle ea %}

The Okta System Log records system events related to your organization in order to provide an audit trail that can be used to understand platform activity and to diagnose problems.

The Okta System Log API provides near real-time read-only access to your organization's system log and is the programmatic counterpart of the [System Log UI](https://help.okta.com/en/prod/Content/Topics/Reports/Reports_SysLog.htm).

Often the terms "event" and "log event" are used interchangeably. In the context of this API, an "event" is an occurrence of interest within the system and "log" or "log event" is the recorded fact.

Notes on the System Log API:

* It contains much more [structured data](#logevent-object) than the [Events API](/docs/api/resources/events#event-model).
* It supports additional [SCIM filters](#request-parameters) and the `q` query parameter, because of the presence of more structured data than the [Events API](/docs/api/resources/events#request-parameters).
* Its primary supported use cases are:
  * Event data export into a security information and event management system (SIEM).
  * System monitoring.
  * Development debugging.
  * Event introspection and audit.
* It is not intended to be used as a Database as a Service (DBaaS), or otherwise directly serve data to downstream consumers without an intermediate data store.

## Getting Started

The System Log API has one endpoint:

{% api_operation get /api/v1/logs %}

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/9cfb0dd661a5432a77c6){:target="_blank"}

This collection resource is backed by a [LogEvent object](#logevent-object) model and associated [event types](#event-types).

See [Examples](#examples) for ways you can use the System Log API. For common use cases see [Useful System Log Queries](https://support.okta.com/help/Documentation/Knowledge_Article/Useful-System-Log-Queries).

## LogEvent Object

Each LogEvent object describes a single logged action or "event" performed by a set of actors for a set of targets.

### Example LogEvent Object
```json
{
  "version": "0",
  "severity": "INFO",
  "client": {
    "zone": "OFF_NETWORK",
    "device": "Unknown",
    "userAgent": {
      "os": "Unknown",
      "browser": "UNKNOWN",
      "rawUserAgent": "UNKNOWN-DOWNLOAD"
    },
    "ipAddress": "12.97.85.90"
  },
  "actor": {
    "id": "00u1qw1mqitPHM8AJ0g7",
    "type": "User",
    "alternateId": "admin@tc1-trexcloud.com",
    "displayName": "John Fung"
  },
  "outcome": {
    "result": "SUCCESS"
  },
  "uuid": "f790999f-fe87-467a-9880-6982a583986c",
  "published": "2017-09-31T22:23:07.777Z",
  "eventType": "user.session.start",
  "displayMessage": "User login to Okta",
  "transaction": {
    "type": "WEB",
    "id": "V04Oy4ubUOc5UuG6s9DyNQAABtc"
  },
  "debugContext": {
    "debugData": {
      "requestUri": "/login/do-login"
    }
  },
  "legacyEventType": "core.user_auth.login_success",
  "authenticationContext": {
    "authenticationStep": 0,
    "externalSessionId": "1013FfF-DKQSvCI4RVXChzX-w"
  }
}
```

### LogEvent Object Annotated Example

```html
{
"uuid": Randomly generated String, Required
"published": ISO8601 string for timestamp, Required
"eventType": String, Required
"version": String, Required
"severity": String, one of DEBUG, INFO, WARN, ERROR, Required
"legacyEventType": String, Optional
"displayMessage": String, Optional
"actor": { Object, Required
     "id": String, Required
     "type": String, Required
     "alternateId": String, Optional
     "displayName": String, Optional
     "detailEntry" = {
     String -> String/Resource Map
     }
},
"client": { Object, Optional
     "userAgent": { Object, Optional
          "rawUserAgent": String, Optional
          "os": String, Optional
          "browser": String, Optional
     },
     "geographicalContext": { Object, Optional
          "geolocation": { Object, Optional
               "lat":Double, Optional
               "lon": Double, Optional
          }
          "city": String, Optional
          "state": String, Optional
          "country": String, Optional
          "postalCode": String, Optional
     },
     "zone": String, Optional
     "ipAddress": String, Optional
     "device": String, Optional
     "id": String, Optional
},
"outcome": { Object, Optional
     "result": String, one of: SUCCESS, FAILURE, SKIPPED, UNKNOWN, Required
     "reason": String, Optional
},
"target": [ List of Objects of the form:
          {
               "id": String, Required
               "type": String, Required
               "alternateId": String, Optional
               "displayName": String, Optional
               "detailEntry" = {
                    String -> String/Resource Map
               }
     }
],
"transaction": { Object, Optional
     "id": String, Optional
     "type": String one of "WEB", "JOB", Optional
     "detail" = {
          String -> String/Resource Map
     }
},
"debugContext": { Object, Optional
     "debugData": {
          String -> String/Resource Map
          "requestUri": "/api/1/devtools/global/test/orgs/specific"
          "originalPrincipal": {
               "id": "00ujchcbjpltartYI0g3",
               "type": "User",
               "alternateId": "admin@saasure.com",
               "displayName": "Piras Add-min"
          },
     }
},
"authenticationContext": { Object, Optional
     "authenticationProvider": String one of OKTA_AUTHENTICATION_PROVIDER, ACTIVE_DIRECTORY, LDAP, FEDERATION,
            SOCIAL, FACTOR_PROVIDER, Optional
          "credentialProvider": String one of OKTA_CREDENTIAL_PROVIDER, RSA, SYMANTEC, GOOGLE, DUO, YUBIKEY, Optional
          "credentialType": String one of OTP, SMS, PASSWORD, ASSERTION, IWA, EMAIL, OAUTH2, JWT, Optional
          "issuer": Object, Optional {
               "id": String, Optional
               "type": String Optional
          }
          "externalSessionId": String, Optional
          "interface": String, Optional i.e. Outlook, Office365, wsTrust
},
"securityContext": { Object, Optional
          "asNumber": Integer, Optional
          "asOrg": String, Optional
          "isp": String, Optional
          "domain": String, Optional
          "isProxy": Boolean, Optional
},
"request": { Object, Optional
          "ipChain": List of objects of the form [
              "ip": String, Optional
              "geographicalContext": { Object, Optional
                        "geolocation": { Object, Optional
                             "lat":Double, Optional
                             "lon": Double, Optional
                        }
                        "city": String, Optional
                        "state": String, Optional
                        "country": String, Optional
                        "postalCode": String, Optional
                   },
              "version": String, one of V4, V6 Optional
              "source": String, Optional
          ], Optional
}
```

### Attributes

LogEvent objects are read-only. The following properties are available:

|-----------+-----------------------------------------------------------------------+----------------------------------------------------------------+----------+--------+----------+-----------+-----------|
| Property  | Description                                                           | DataType                                                       | Nullable | Unique | Readonly | MinLength | MaxLength |
| -------   | --------------------------------------------------------------------- | ---------------------------------------------------------------| -------- | ------ | -------- | --------- | --------- |
| uuid      | Unique identifier for an individual event                             | String                                                         | FALSE    | TRUE   | TRUE     |           |           |
| published | Timestamp when event was published                                    | Date                                                           | FALSE    | FALSE  | TRUE     | 1         | 255       |
| eventType | Type of event that was published                                      | String                                                         | FALSE    | FALSE  | TRUE     | 1         | 255       |
| version   | Versioning indicator                                                  | String                                                         | FALSE    | FALSE  | TRUE     | 1         | 255       |
| severity  | Indicates how severe the event is: `DEBUG`, `INFO`, `WARN`, `ERROR`   | String                                                         | FALSE    | FALSE  | TRUE     | 1         | 255       |
| legacyEventType | Associated Events API [Action `objectType`](events#action-objecttypes) attribute value| String                                                         | TRUE     | FALSE  | TRUE     | 1         | 255       |
| displayMessage | The display message for an event                                 | String                                                         | TRUE     | FALSE  | TRUE     | 1         | 255       |
| actor     | Describes the entity that performed an action                         | Array of [Actor Object](#actor-object)                         | TRUE     | FALSE  | TRUE     |           |           |
| client    | The client that requested an action                                   | [Client Object](#client-object)                                | TRUE     | FALSE  | TRUE     |           |           |
| outcome   | The outcome of an action                                              | [Outcome Object](#outcome-object)                              | TRUE     | FALSE  | TRUE     |           |           |
| target    | Zero or more targets of an action                                     | [Target Object](#target-object)                                | TRUE     | FALSE  | TRUE     |           |           |
| transaction   |  The transaction details of an action                             | [Transaction Object](#transaction-object)                      | TRUE     | FALSE  | TRUE     |           |           |
| debugContext   | The debug request data of an action                              | [DebugContext Object](#debugcontext-object)                    | TRUE     | FALSE  | TRUE     |           |           |
| authenticationContext | The authentication data of an action                      | [AuthenticationContext Object](#authenticationcontext-object)  | TRUE     | FALSE  | TRUE     |           |           |
| securityContext | The security data of an action                                  | [SecurityContext Object](#securitycontext-object)              | TRUE     | FALSE  | TRUE     |           |           |
|-----------+-----------------------------------------------------------------------+----------------------------------------------------------------+----------+--------+----------+-----------+-----------|

> The actor and/or target of an event is dependent on the action performed. All events have actors but not all have targets.

> The `authenticationContext.externalSessionId` identifies events that occurred in the same session. A single `transaction.id` identifies events that occurred together as part of an operation (e.g. a request to Okta's servers). Use `authenticationContext.externalSessionId` to link events that occurred in the same session, and the `transaction.id` to link events that occurred as part of the same operation.

### Actor Object

Describes the user, app, client, or other entity (actor) who performed an action on a target

|-------------+-----------------------------------------------+-------------------+----------|
| Property    | Description                                   | DataType          | Nullable |
| ----------- | ----------------------------------------------| ----------------- | -------- |
| id          | ID of actor                                   | String            | FALSE    |
| type        | Type of actor                                 | String            | FALSE    |
| alternateId | Alternative ID of actor                       | String            | TRUE     |
| displayName | Display name of actor                         | String            | TRUE     |
| detail      | Details about actor                           | Map[String->Object]| TRUE    |
|-------------+-----------------------------------------------+-------------------+----------|

### Target Object

The entity upon which an actor performs an action. Targets may be anything: an app user, a login token or anything else.

|-------------+--------------------------------------------------------------+-----------------+----------|
| Property    | Description                                                  | DataType        | Nullable |
| ----------- | ------------------------------------------------------------ | --------------- | -------- |
| id          | ID of a target                                               | String          | FALSE    |
| type        | Type of a target                                             | String          | FALSE    |
| alternateId | Alternative id of a target                                   | String          | TRUE     |
| displayName | Display name of a target                                     | String          | TRUE     |
| detail      | Details about target                                         | Map[String->Object] | TRUE |
|-------------+--------------------------------------------------------------+-----------------+----------|

```json
{
    "id": "00u3gjksoiRGRAZHLSYV",
    "displayName": "Jon Stewart",
    "alternateId": "00uKrs9rsRSAXN",
    "login": "jon@example.com",
    "type": "User"
}
```

### Client Object

When an event is triggered by an HTTP request, the `client` object describes the [client](https://en.wikipedia.org/wiki/Category:Hypertext_Transfer_Protocol_clients) that issues that HTTP request. For instance, the web browser is the client when a user accesses Okta. When this request is received and processed, a login event is fired. When the event is not sourced to an HTTP request, such as in the case of an automatic update, the Client Object field is blank.

|------------+--------------------------------------------------------------------------------------------------------------------+-----------------+----------|
| Property   | Description                                                                                                        | DataType        | Nullable |
| ---------- | ------------------------------------------------------------------------------------------------------------------ | --------------- | -------- |
| id  | For OAuth requests this is the id of the OAuth [client](https://tools.ietf.org/html/rfc6749#section-1.1) making the request. For SSWS token requests, this is the id of the agent making the request. | [UserAgent Object](#useragent-object) | TRUE |
| userAgent  | The [user agent](https://en.wikipedia.org/wiki/User_agent) used by an actor to perform an action | [UserAgent Object](#useragent-object) | TRUE |
| geographicalContext | The physical location where the client made its request from | [GeographicalContext Object](#geographicalcontext-object)    | TRUE     |
| zone       | The `name` of the [Zone](/docs/api/resources/zones#ZoneModel) that the client's location is mapped to       | String          | TRUE     |
| ipAddress  | Ip address that the client made its request from                                                                   | String          | TRUE     |
| device     | Type of device that the client operated from (e.g. Computer)                                                       | String          | TRUE     |
|------------+--------------------------------------------------------------------------------------------------------------------+-----------------+----------|

### UserAgent Object

"A user agent is software (a software agent) that is acting on behalf of a user." ([Wikipedia](https://en.wikipedia.org/wiki/User_agent))

In the Okta event data model, the UserAgent object provides specifications about the client software that makes event-triggering HTTP requests. User agent identification is often useful for identifying interoperability problems between servers and clients, and also for browser and operating system usage analytics.

|--------------+---------------------------------------------------------------------------------------------------+----------------+----------|
| Property     | Description                                                                                       | DataType       | Nullable |
| ------------ | ------------------------------------------------------------------------------------------------- | -------------- | -------- |
| Browser      | If the client is a web browser, this field identifies the type of web browser (e.g. CHROME, FIREFOX) | String      | TRUE     |
| OS           | The [Operating System](https://en.wikipedia.org/wiki/Operating_system) the client runs on (e.g. Windows 10) | String | TRUE   |
| RawUserAgent | A raw string representation of the user agent, formatted according to [section 5.5.3 of HTTP/1.1 Semantics and Content](https://tools.ietf.org/html/rfc7231#section-5.5.3). Both the `browser` and the `OS` fields can be derived from this field.                                                                                                             | String         | TRUE     |
|--------------+---------------------------------------------------------------------------------------------------+----------------+----------|

### Request Object

The request object describes details related to the HTTP request that triggers this event, if available. When the event is not sourced to an http request, such as in the case of an automatic update on the Okta servers, the Request object will still exist, but the `ipChain` field will be empty.

|--------------|---------------------------------------------------------------------------------------------------|----------------|----------|
| Property     | Description                                                                                       | DataType       | Nullable |
| ------------ | ------------------------------------------------------------------------------------------------- | -------------- | -------- |
| ipChain      | If the incoming request passes through any proxies, the IP addresses of those proxies will be stored here in the format (clientIp, proxy1, proxy2, ...). This field is useful when working with trusted proxies.                             | Array of [IpAddress](#ipaddress-object) | TRUE     |
|--------------+---------------------------------------------------------------------------------------------------+----------------+----------|

### GeographicalContext Object

Geographical Context describes a set of geographic coordinates. In addition to containing latitude and longitude data, this object also contains address data of postal code-level granularity. Within the [Client](#client-object) object, the geographical context refers to the physical location of the client when it sends the request that triggers this event.

|-------------+------------------------------------------------------------------------------------+---------------------------------------+----------|
| Property    | Description                                                                        | DataType                              | Nullable |
| ----------- | ---------------------------------------------------------------------------------- | ------------------------------------- | -------- |
| geolocation | Contains the geolocation coordinates (latitude, longitude)                         | [Geolocation Object](#geolocation-object) | TRUE |
| city        | The city encompassing the area containing the geolocation coordinates, if available (e.g. Seattle, San Francisco) | String | TRUE     |
| state       | Full name of the state/province encompassing the area containing the geolocation coordinates (e.g. Montana, Incheon) | String | TRUE  |
| country     | Full name of the country encompassing the area containing the geolocation coordinates (e.g. France, Uganda) | String       | TRUE     |
| postalCode  | Postal code of the area encompassing the geolocation coordinates                   | String                                | TRUE     |
|-------------+------------------------------------------------------------------------------------+---------------------------------------+----------|

### Geolocation Object

Latitude and longitude of the geolocation where an action was performed, formatted according to the [ISO-6709](https://en.wikipedia.org/wiki/ISO_6709) standard.

|------------+---------------------------------------------------------------------------------------------------+-----------------+----------|
| Property   | Description                                                                                       | DataType        | Nullable |
| ---------- | ------------------------------------------------------------------------------------------------- | --------------- | -------- |
| lat        | Latitude. Uses 2 digits for the [integer part](https://en.wikipedia.org/wiki/ISO_6709#Latitude)   | Double          | FALSE    |
| lon        | Longitude. Uses 3 digits for the [integer part](https://en.wikipedia.org/wiki/ISO_6709#Longitude) | Double          | FALSE    |
|------------+---------------------------------------------------------------------------------------------------+-----------------+----------|

### Outcome Object

Describes the result of an action and the reason for that result.

|------------+------------------------------------------------------------------------+-----------------+----------+---------+-----------+-----------|
| Property   | Description                                                            | DataType        | Nullable | Default | MinLength | MaxLength |
| ---------- | ---------------------------------------------------------------------- | --------------- | -------- | ------- | --------- | --------- |
| result     | Result of the action: `SUCCESS`, `FAILURE`, `SKIPPED`, `UNKNOWN`       | String          | FALSE    |         |           |           |
| reason     | Reason for the result, for example `INVALID_CREDENTIALS`               | String          | TRUE     |         | 1         | 255       |
|------------+------------------------------------------------------------------------+-----------------+----------+---------+-----------+-----------|

### Transaction Object

The `transaction` object contains metadata associated with the event. This is useful for sourcing and identifying events. For example, a transaction object such as

```json
{
    "id": "Wn4f-0RQ8D8lTSLkAmkKdQAADqo",
    "type": "WEB",
    "detail": null
}
```

indicates that a single web request with requestId `Wn4f-0RQ8D8lTSLkAmkKdQAADqo` was responsible for creating this event.

|------------+----------------------------------------------------------------+-----------------+----------|
| Property   | Description                                                    | DataType        | Nullable |
| ---------- | -------------------------------------------------------------- | --------------- | -------- |
| id         | Id of the transaction Object. When the `type` is `WEB`, this field will contain the requestId of the web request. | String | TRUE |
| type       | Type of transaction. When the transaction is initiated from a single web request, this value is `WEB`. For jobs, this value is `JOB` | String | TRUE |
| detail     | Details about the transaction                                  | Map[String → Object] | TRUE |
|------------+----------------------------------------------------------------+-----------------+----------|

### DebugContext Object

For some kinds of events (e.g. OMM provisioning, login, second factor SMS, etc.), the fields provided in other response objects will not be sufficient to adequately describe the operations the event has performed. In such cases, the `debugContext` object provides a way to store additional information.

For example, an event where a second factor SMS token is sent to a user might have a `debugContext` that looks like:

```json
{
    "debugData": {
        "requestUri": "/api/v1/users/00u3gjksoiRGRAZHLSYV/factors/smsf8luacpZJAva10x45/verify",
        "smsProvider": "TELESIGN",
        "transactionId": "268632458E3C100F5F5F594C6DC689D4"
    }
}

```

By inspection of the `debugData` field, one can find the URI used to trigger the second factor SMS (`/api/v1/users/00u3gjksoiRGRAZHLSYV/factors/smsf8luacpZJAva10x45/verify`), the SMS provider (`TELESIGN`) and the ID used by Telesign to identify this transaction (`268632458E3C100F5F5F594C6DC689D4`).

If for some reason the information needed to implement a feature is not provided in other response objects, it is advised to scan the `debugContext.debugData` field for potentially useful fields.

|------------+---------------------------------------------------------------------------------+-----------------+----------|
| Property   | Description                                                                     | DataType        | Nullable |
| ---------- | ------------------------------------------------------------------------------- | --------------- | -------- |
| debugData  | Dynamic field containing miscellaneous information dependent on the event type. | Map[String->Object] | TRUE |
|------------+---------------------------------------------------------------------------------+-----------------+----------|

### AuthenticationContext Object

All authentication relies on validating one or more credentials that proves the authenticity of the actor's identity. Credentials are sometimes provided by the actor, as is the case with passwords, and at other times provided by a third party, and validated by the authentication provider.

The `authenticationContext` contains metadata about how the actor is authenticated. For example, an `authenticationContext` for an event in which a user authenticates with IWA will look something like:

```json
{
    "AuthenticationProvider": "ACTIVE_DIRECTORY",
    "AuthenticationStep": 0,
    "CredentialProvider": null,
    "CredentialType": "IWA",
    "ExternalSessionId": "102N1EKyPFERROGvK9wizMAPQ",
    "Interface": null,
    "Issuer": null
}
```
In such a case, one can recognize that the user used an IWA credential to authenticate against an Active Directory instance. All of the user's future generated events in this login session will share the same `externalSessionId`.

Among other operations, this response object can be used to scan for suspicious login activity or perform analytics on user authentication habits (e.g. how often authentication scheme X is used versus authentication scheme Y).

|------------+----------------------------------------------------------------+-----------------+----------+-----------+-----------|
| Property   | Description                                                    | DataType        | Nullable | MinLength | MaxLength |
| ---------- | -------------------------------------------------------------- | --------------- | -------- | --------- | --------- |
| authenticationProvider | The system that proves the identity of an actor using the credentials provided to it | String | TRUE  | | |
| credentialProvider | A credential provider is a software service that manages identities and their associated credentials. When authentication occurs via credentials provided by a credential provider, that credential provider will be recorded here. | String | TRUE | | |
| credentialType | The underlying technology/scheme used in the credential    | String          | TRUE     |           |           |
| issuer     | The specific software entity that created and issued the credential. | [Issuer Object](#issuer-object) | TRUE | |   |
| externalSessionId | A proxy for the actor's [session ID](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet) | String | TRUE | 1 | 255 |
| interface  | The third party user interface that the actor authenticates through, if any. | String | TRUE | 1        | 255       |
|------------+----------------------------------------------------------------+-----------------+----------+-----------+-----------|

###### Possible Values

Some of the fields listed above have a finite set of possible values.

|------------------------+-------------------------------------------------------------------------------------------------------|
| Property               | Possible Values                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------- |
| authenticationProvider | `OKTA_AUTHENTICATION_PROVIDER`, `ACTIVE_DIRECTORY`, `LDAP`, `FEDERATION`, `SOCIAL`, `FACTOR_PROVIDER` |
| credentialProvider     | `OKTA_CREDENTIAL_PROVIDER`, `RSA`, `SYMANTEC`, `GOOGLE`, `DUO`, `YUBIKEY`                             |
| credentialType         | `OTP`, `SMS`, `PASSWORD`, `ASSERTION`, `IWA`, `EMAIL`, `OAUTH2`, `JWT`                                |
|------------------------+-------------------------------------------------------------------------------------------------------|

### Issuer Object

Describes the `issuer` of the authorization server when the authentication is performed via OAuth. This is the location where well-known resources regarding details of the authorization servers are published.

|------------+----------------------------------------------------------------+-----------------+----------|
| Property   | Description                                                    | DataType        | Nullable |
| ---------- | -------------------------------------------------------------- | --------------- | -------- |
| id         | Varies depending on the type of authentication. If authentication is SAML 2.0, `id` is the issuer in the SAML assertion. For social login, `id` is the issuer of the token.                                         | String          | TRUE     |
| type       | Information regarding `issuer` and source of the SAML assertion or token.                                         | String          | TRUE     |
|------------+----------------------------------------------------------------+-----------------+----------|

### SecurityContext Object

The `securityContext` object provides security information directly related to the evaluation of the event's IP reputation. IP reputation is a trustworthiness rating that evaluates how likely a sender is to be malicious based on the sender's IP address. As the name implies, the `securityContext` object is useful for security applications-flagging and inspecting suspicious events.

|----------+-------------------------------------------------------------------------------------------------+----------+----------|
| Property | Description                                                                                     | DataType | Nullable |
| -------- | ----------------------------------------------------------------------------------------------- | -------- | -------- |
| asNumber | [Autonomous system](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) number associated with the autonomous system that the event request was sourced to | Integer | TRUE |
| asOrg    | Organization associated with the autonomous system that the event request was sourced to        | String   | TRUE     |
| isp      | [Internet service provider](https://en.wikipedia.org/wiki/Internet_service_provider) used to sent the event's request | String | TRUE |
| domain   | The [domain name](https://en.wikipedia.org/wiki/Domain_name) associated with the IP address of the inbound event request | String | TRUE |
| isProxy  | Specifies whether an event's request is from a known proxy                                      | Bool     | TRUE     |
|----------+-------------------------------------------------------------------------------------------------+----------+----------|

### IpAddress Object

Describes an IP address used in a request.

| ------------+----------------------------------------------------------------+-----------------+---------- |
| Property                                                                                                   | Description                            | DataType                                                    | Nullable |
|:-----------------------------------------------------------------------------------------------------------|:---------------------------------------|:------------------------------------------------------------|:---------|
| ip                                                                                                         | IP address                             | String                                                      | TRUE     |
| geographicalContext                                                                                        | Geographical context of the IP address | [GeographicalContext Object](#geographicalcontext-object) | TRUE     |
| version                                                                                                    | IP address version                     | V4 or V6                                                    | TRUE     |
| source                                                                                                     | Details regarding the source           | String                                                      | TRUE     |
| ------------+----------------------------------------------------------------+-----------------+---------- |

## Event Types

Event types categorize event instances by action and recorded in a LogEvent's [`eventType`](#attributes) attribute. They are key to navigating the system log via [Expression Filters](#expression-filter).

The following sections outline the major event types captured by the system log.

> Note that the listing is not complete and you may see additional event types returned from the API.

### Application Event

| Event                                        | Description                                                |
|:---------------------------------------------|:-----------------------------------------------------------|
| application.lifecycle.activate               | An application was activated.                              |
| application.lifecycle.create                 | An application was created.                                |
| application.lifecycle.deactivate             | An application was deactivated.                            |
| application.lifecycle.delete                 | An application was deleted.                                |
| application.lifecycle.update                 | An application was updated.                                |
| application.user_membership.add              | A user was assigned to an application.                     |
| application.user_membership.change_username  | The name of a user assigned to an application was changed. |
| application.user_membership.remove           | A user was removed from an application.                    |

### Group Event

| Event                        | Description                      |
|:-----------------------------|:---------------------------------|
| group.user_membership.add    | A user was added to a group.     |
| group.user_membership.remove | A user was removed from a group. |

### Policy Events

| Event                       | Description                         |
|:----------------------------|:------------------------------------|
| policy.lifecycle.activate   | A rule in a policy was activated.   |
| policy.lifecycle.create     | A rule in a policy was created.     |
| policy.lifecycle.deactivate | A rule in a policy was deactivated. |
| policy.lifecycle.delete     | A rule in a policy was deleted.     |
| policy.lifecycle.update     | A rule in a policy was updated.     |
| policy.rule.activate        | A rule in a policy was activated.   |
| policy.rule.add             | A rule was added to a policy.       |
| policy.rule.deactivate      | A rule in a policy was deactivated. |
| policy.rule.delete          | A rule was deleted from a policy.   |
| policy.rule.update          | A rule in a policy was updated.     |

#### Policy Event Details

* `policy.evaluate_sign_on` provides context on the values used and evaluated in the context of the Okta sign on policy. For example, you can determine which network zones were matched for this event.
* For `policy.lifecycle` and `policy.rule` events, the corresponding policy is listed in the target object.

### System Events

| Event              | Description                         |
|:-------------------|:------------------------------------|
| system.org.rate_limit.warning | An endpoint is near its [rate limit](/docs/api/getting_started/rate-limits).  |
| system.org.rate_limit.violation | An endpoint has exceeded its [rate limit](/docs/api/getting_started/rate-limits). |

Rate limit warnings are sent at different times, depending on the org type. For One App and Enterprise orgs, the warning is sent when the org is at 60% of its limit.

> Note: For orgs created before 2018-05-17, the warning is sent at 90%.

Rate limit violations are sent when a rate limit is exceeded.

### User Events

| Event                     | Description                                               |
|:--------------------------|:----------------------------------------------------------|
| user.authentication.sso   | A user attempted to SSO to an application managed in Okta |
| user.lifecycle.activate   | A user account was activated.                             |
| user.lifecycle.create     | A user account was created.                               |
| user.lifecycle.deactivate | A user account was deactivated.                           |
| user.lifecycle.suspend    | A user account was suspended.                             |
| user.lifecycle.unsuspend  | A user account was moved from suspended status.           |
| user.session.start        | Okta issued a session to a user who is authenticating     |

#### User Event Details

* `user.authentication.sso` doesn't capture whether the SSO attempt was successful or failed because Okta can't collect the subsequent authentication attempt status from the third-party service.


## Event ID Correlation

Throughout a user's session many requests (transactions) can occur, such as logging into Okta or opening an application. Within a given request many events may be logged. To illustrate this principle, the table below shows 18 events produced from 13 transactions over 6 different sessions, all performed by one user.

| External Session ID       | Transaction ID              | Event ID                             | Event Type                                  | Display Message                   |
|:--------------------------|:----------------------------|:-------------------------------------|:--------------------------------------------|:----------------------------------|
| trs5JnlvlaIQTOqOj9imLy7lA | WcKPxq1f8QLfFvv3UPHhhgAACGM | f24790d0-d324-47f8-aac5-c27a31ab928d | user.session.access_admin_app               | User accessing Okta administrator app     |
|                           | WcKPxq1f8QLfFvv3UPHhhgAACGM | ed317758-8776-4240-a540-277c44dcb408 | application.lifecycle.update                | Update application                |
|                           |                             | 421c1551-71b2-4ebe-a70d-b5f7d3698429 | application.lifecycle.update                | Update application                |
|                           |                             | 06a50bbe-44fd-40de-83e6-2e4cc2a17d16 | application.lifecycle.update                | Update application                |
| trsUz2TG3wKS6ar1lvWzHo71w | Wij-6q4YuniRd9yTmWHpfwAAADc | 3e240ff4-6af7-47f2-b107-a2ef661ffc01 | application.user_membership.change_username | Change users application username |
|                           |                             | 572b05e9-b6be-4dfe-8bc3-01bb3a5a1af5 | application.user_membership.add            | Add user to application membership |
|                           |                             | 30f29bbf-3218-429b-827a-0a93809591db | application.user_membership.remove        | Remove users application membership |
|                           | Wij-964YuniRd9yTmWHu1AAAAEA | 5f98d062-05a9-4ede-89a0-8a2ce27efdd4 | user.session.access_admin_app               | User accessing Okta administrator app     |
|                           | Wij-95eCbHF7In2MKNavlgAAD9I | 45f71ac2-e8b2-4c19-b4cc-d2560108c889 | application.lifecycle.update                | Update application                |
|                           |                             | 46b85d65-01c6-44d2-86d2-25704804b1c5 | application.lifecycle.update                | Update application                |
| 102GALFw8CzRT2KXoqnca8Jdg | Wij-AJeCbHF7In2MKNaOpAAAEC4 | b9ab9263-a4ae-4780-9981-377ec8f2da86 | user.session.start                          | User login to Okta                |
|                           | Wij-7q4YuniRd9yTmWHrBQAAAKQ | ff325685-0220-484c-82cf-5f8dc596acbe | user.authentication.sso                     | User single sign on to app        |
| trsf8nlpDJZTZeFlcc8nszbjw | Wij-7a4YuniRd9yTmWHqqAAAAKY | 5526a4c4-7f68-4b2a-bab7-2d10ebaeeb1c | mim.checkOSXAccessEligibility.true          | *blank*                           |
|                           | Wij-764YuniRd9yTmWHrkAAAAGw | 232774ba-8feb-4b00-a732-e0ec99a24434 | user.session.start                          | User login to Okta                |
| trswPONv4wIRaKDNWVVcmtceg | Wij-6K4YuniRd9yTmWHo9wAAAAY | d31d819a-1427-45b0-a8b4-8a8fb40c72f1 | user.session.start                          | User login to Okta                |
|                           | Wij-564YuniRd9yTmWHoaQAAAII | 0cc6f4c8-9b91-4a70-b5c4-09d6ad159d32 | mim.checkOSXAccessEligibility.true          | *blank*                           |
|                           | Wij-2q4YuniRd9yTmWHjRAAAADA | 92606da8-7eeb-4ad7-8ffb-502dd0ec64cc | user.authentication.sso                     | User single sign on to app        |
| *null*                    | Wm@-R2s5lEMbNIB03krtvAAACyo | 566671be-ec0b-400d-ad2e-6fc73ed12fb1 | user.session.start                          | User login to Okta                |
{:.table .table-word-break}

Note that, as evidenced by the `null` `External Session ID` field in the last row, neither `Transaction ID` nor `Event ID` maintain a many-to-one relationship with `External Session ID`. In this particular case, the `null` `External Session ID` field can be explained by a failed user login. Since the login failed, no session was granted back to the user's client.


## Operations

### List Events
{:.api .api-operation}

{% api_operation get /api/v1/logs %}

Fetch a list of ordered log events from your Okta organization's system log.

#### Request Parameters
{:.api .api-request .api-request-params}

The table below summarizes the supported query parameters:

|------------ + ------------------------------------------------------------------------------------------------------+----------------------------------------------------------+-------------------------|
| Parameter   | Description                                                                                           | Format                                                   | Default                 |
| ----------- | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ----------------------- |
| `since`     | Filters the lower time bound of the log events `published` property | The [Internet Date/Time Format profile of ISO 8601](https://tools.ietf.org/html/rfc3339#page-8). An example: `2017-05-03T16:22:18Z` | 7 days prior to `until` |
| `until`     | Filters the upper time bound of the log events `published` property | The [Internet Date/Time Format profile of ISO 8601](https://tools.ietf.org/html/rfc3339#page-8). An example: `2017-05-03T16:22:18Z` | Current time |
| `after`     | Used to retrieve the next page of results. Okta returns a link in the HTTP Header (`rel=next`) that includes the after query parameter. | Opaque token           |                         |
| `filter`    | [Filter Expression](#expression-filter) that filters the results                                      | [SCIM Filter expression](/docs/api/getting_started/design_principles#filtering) |  |
| `q`         | Filters the log events results by one or more exact [keywords](#keyword-filter)                       | URL encoded string                                       |                         |
| `sortOrder` | The order of the returned events sorted by `published`                                                | `ASCENDING` or `DESCENDING`                              | `ASCENDING`             |
| `limit`     | Sets the number of results returned in the response                                                   | Integer between 0 and 1000                                | 100                     |
|-------------+-------------------------------------------------------------------------------------------------------+----------------------------------------------------------+-------------------------|

##### Filtering Results

###### Expression Filter

An expression filter is useful for performing structured queries where constraints on LogEvent attribute values can be explicitly targeted.

The following expressions are supported for events with the `filter` query parameter:

Filter                                       | Description
-------------------------------------------- | ------------------------------------------------------------------------------
`eventType eq ":eventType"`                  | Events that have a specific action [eventType](#attributes)
`target.id eq ":id"`                         | Events published with a specific target id
`actor.id eq ":id"`                          | Events published with a specific actor id

See [Filtering](/docs/api/getting_started/design_principles#filtering) for more information about expressions.

The following are some examples of common filter expressions.

Events published for a target user:
```javascript
filter=target.id eq "00uxc78lMKUMVIHLTAXY"
```

Failed login events:
```javascript
filter=eventType eq "user.session.start" and outcome.result eq "FAILURE"
```

Events published for a target user and application:
```javascript
filter=target.id eq "00uxc78lMKUMVIHLTAXY" and target.id eq "0oabe82gnXOFVCDUMVAK"
```

App SSO events for a target user and application:
```javascript
filter=eventType eq "app.auth.sso" and target.id eq "00uxc78lMKUMVIHLTAXY" and target.id eq "0oabe82gnXOFVCDUMVAK"
```

Events published for a given ip address:
```javascript
filter=client.ipAddress eq "184.73.186.14"
```

###### Keyword Filter

The query parameter `q` can be used to perform keyword matching against a LogEvents object's attribute values. In order to satisfy the constraint, all supplied keywords must be matched exactly.
Note that matching is case-insensitive.

The following are some examples of common keyword filtering:

* Events that mention a specific city: `q=San Francisco`
* Events that mention a specific url: `q=interestingURI.com`
* Events that mention a specific person: `q=firstName lastName`

###### Datetime Filter

LogEvent objects can be filtered by [`published`](#attributes) attribute value with the following combination of parameters:

* `since`
* `until`
* `since` and `until`
* `after`

Note that `since` and `after` are mutually exclusive and cannot be specified simultaneously.

The `after` parameter is system generated for use in ["next" links](#next-link-response-header). Users should not attemp to craft requests using this value and rely on the system generated links instead.

##### Response
{:.api .api-response .api-response-params}

The response contains a JSON array of [LogEvent objects](#logevent-object).

###### Self Link Response Header
The response always includes a `self` `Link` header, which is a link to the current query that was executed.

This header is of the form:
```
Link: <url>; rel="self"
```

For example:
```
Link: <https://{yourOktaDomain}.com/api/v1/logs?q=&sortOrder=DESCENDING&limit=20&until=2017-09-17T23%3A59%3A59%2B00%3A00&since=2017-06-10T00%3A00%3A00%2B00%3A00>; rel="self"
```

###### Next Link Response Header
The response may include a `next` `Link` header, which is a link to the next page of results, if there is one. Note that while the `self` `Link` will always exist, the `next` `Link` may not exist.

This header is of the form:
```
Link: <url>; rel="next"
```

For example:
```
Link: <https://{yourOktaDomain}.com/api/v1/logs?q=&sortOrder=DESCENDING&limit=20&until=2017-09-17T15%3A41%3A12.994Z&after=349996bd-5091-45dc-a39f-d357867a30d7&since=2017-06-10T00%3A00%3A00%2B00%3A00>; rel="next"
```

#### Timeouts
Individual queries have a timeout of 30 seconds.

### Errors
```json
{
  "errorCode": "E0000001",
  "errorSummary": "Api validation failed: 'until': The date format in your query is not recognized. Please enter dates using ISO8601 string format.. 'until': must be a valid date-time or empty.",
  "errorId": "dd4998a1-2267-499b-9e4d-ec821fcc5ca9",
  "errorCauses": [
    {
      "errorSummary": "until: The date format in your query is not recognized. Please enter dates using ISO8601 string format."
    },
    {
      "errorSummary": "until: must be a valid date-time or empty."
    }
  ]
}
```

An invalid SCIM filter returns a 400 with a description of the issue with the SCIM filter. For example:
```json
{
  "errorCode": "E0000053",
  "errorSummary": "Invalid filter 'display_message eqq \"Create okta user\"': Unrecognized attribute operator 'eqq' at position 16. Expected: eq,co,sw,pr,gt,ge,lt,le",
  "errorId": "eb83dfe1-6d76-458c-8c0c-f8df8fb7a24b"
}
```

An Invalid field returns a 400 with a message indicating which field is invalid. For example:
```json
{
  "errorCode": "E0000053",
  "errorSummary": "field is not valid: some_invalid_field",
  "errorId": "ec93dhe2-6d76-458c-8c0c-f8df8fb7a24b"
}
```

Another example, where the parameters are invalid:
```json
{
  "errorCode": "E0000053",
  "errorSummary": "Invalid parameter: The since parameter is over 180 days prior to the current day.",
  "errorId": "55166534-b7d8-45a5-a4f6-3b38a5507046"
}
```

An internal service error returns a 500 with the message:
```json
{
  "errorCode": "E0000053",
  "errorSummary": "Sorry, there's been an error. We aren't sure what caused it, but we've logged this and will work to address it. Please try your request again.",
  "errorId": "55166534-b7d8-45a5-a4f6-3b38a5507046"
}
```

A timeout returns a 500 with the message:
```json
{
  "errorCode": "E0000009",
  "errorSummary": "Your last request took too long to complete. This is likely due to a load issue on our side. We've logged this and will work to address it. Please either simplify your query or wait a few minutes and try again."
}
```

A free form query that is too long:
```json
{
  "errorCode": "E0000001",
  "errorSummary": "Api validation failed: 'q': Freeform search cannot contain items longer than 40 characters. Please shorten the items in your search or use an advanced filter to query by specific fields."
}
```

Exceeding the rate limit results in:
```json
{
  "errorCode": "E0000047",
  "errorSummary": "API call exceeded rate limit due to too many requests."
}
```

### Rate Limits
Callers are limited to 60 queries max per minute.

## Data Retention

Log data older than 90 days is not returned, in accordance with Okta's [Data Retention Policy](https://support.okta.com/help/Documentation/Knowledge_Article/Okta-Data-Retention-Policy). Queries that exceed the retention period will succeed but only results that have a `published` timestamp within the window will be returned.

## Examples

### Debugging
The System Log API can be used to troubleshoot user problems. For example, you
can use the following `curl` command to see events from user "Jane Doe":

```sh
curl -v -X GET \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token}" \
"https://{yourOktaDomain}.com/api/v1/logs?q=Jane+Doe"
```

You can also use this API to search for particular types of events:

```sh
curl -v -X GET \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token}" \
"https://{yourOktaDomain}.com/api/v1/logs?filter=event_type+eq+%22user.session.start%22"
```

### Transferring Data to a Separate System
You can export your log events to a separate system for analysis or compliance. To obtain the entire dataset, query from the appropriate point of time in the past.

```sh
curl -v -X GET \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token}" \
"https://{yourOktaDomain}.com/api/v1/logs?since=2017-10-01T00:00:00.000Z"
```

and retrieve the next page of events through the [`Link` response header](/docs/api/getting_started/design_principles#link-header) value with the `next` link relation. Continue this process until no events are returned.

> Do not attempt to transfer data by manually paginating using `since` and `until` as this may lead to skipped or duplicated events. Instead, always follow the `next` links.
