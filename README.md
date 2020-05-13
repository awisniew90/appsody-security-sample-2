# appsody-security-sample-2

The Open Liberty Appsody stack provides default security credentials for local development. These default values (in configDropins/default/quick-start-security.xml) are removed from the official deployment image for security purposes with the intent that a user will add their own security mechanism when deploying. This applications shows how that can be accomplished using Service Account tokens. Additionally, a Service Monitor configuration allows for easy connection from a Prometheus instance.

## Prerequisites 

* A Service Account will need to be created in the same namespace where the app will be deployed. When the SA is created, a Secret is created within it containing a token. That token will be used to authenticate to the Open Liberty server.

## Open Liberty configuration

In `server.xml`, there are several changes made to enable authentication via service account tokens:

1. The following features have been added:
```xml
<feature>appSecurity-3.0</feature>
<feature>socialLogin-1.0</feature>
```

1. The `okdServiceLogin` element has been added:
```xml
<okdServiceLogin
    realmName="okd123" >
</okdServiceLogin>
```

1. An administrative-role has been configured where "my-namespace" is the namespace where the app will be deployed:

```xml
<administrator-role>
    <group-access-id>group:okd123/my-namespace</group-access-id>
</administrator-role>
``` 


## Service Monitor

This app defines a monitoring spec that the OL Operator will use to create a Prometheus Service Monitor. This configuration has Prometheus add a Bearer Token to the Authorization header of each scrape request (e.g. "Authorization: Bearer <token>"). The token is taken from the Secret "my-token-secret" which exists within the Service Account created in the prereq above. 

NOTE: This is currently blocked by https://github.com/OpenLiberty/open-liberty-operator/issues/157

```yaml
monitoring:
    endpoints:
    - bearerTokenSecret:
        key: token
        name: my-token-secret
      interval: 5s
      tlsConfig:
        insecureSkipVerify: true
    labels:
      app-monitoring: ""
```

Prometheus may need to be configured to watch the namespace where this app is deployed, and the namespace will need to have the label "app-monitoring."
