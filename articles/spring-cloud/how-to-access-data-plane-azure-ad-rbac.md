---
title: "Access Config Server and Service Registry"
titleSuffix: Azure Spring Cloud
description: How to access Config Server and Service Registry Endpoints with Azure Active Directory role-based access control.
author: karlerickson
ms.author: karler
ms.service: spring-cloud
ms.topic: how-to
ms.date: 08/25/2021
ms.custom: devx-track-java, subject-rbac-steps
---

# Access Config Server and Service Registry

This article explains how to access the Spring Cloud Config Server and Spring Cloud Service Registry managed by Azure Spring Cloud using Azure Active Directory (Azure AD) role-based access control (RBAC).

## Assign role to Azure AD user/group, MSI, or service principal

Assign the role to the [user | group | service-principal | managed-identity] at [management-group | subscription | resource-group | resource] scope.

| Role name                                       | Description                                                                  |
|-------------------------------------------------|------------------------------------------------------------------------------|
| Azure Spring Cloud Config Server Reader         | Allow read access to Azure Spring Cloud Config Server.                       |
| Azure Spring Cloud Config Server Contributor    | Allow read, write, and delete access to Azure Spring Cloud Config Server.    |
| Azure Spring Cloud Service Registry Reader      | Allow read access to Azure Spring Cloud Service Registry.                    |
| Azure Spring Cloud Service Registry Contributor | Allow read, write, and delete access to Azure Spring Cloud Service Registry. |

For detailed steps, see [Assign Azure roles using the Azure portal](../role-based-access-control/role-assignments-portal.md).

## Access Config Server and Service Registry Endpoints

After the role is assigned, the assignee can access the Spring Cloud Config Server and the Spring Cloud Service Registry endpoints using the following procedures:

1. Get an access token. After an Azure AD user is assigned the role, they can use the following commands to sign in to Azure CLI with user, service principal, or managed identity to get an access token. For details, see [Authenticate Azure CLI](/cli/azure/authenticate-azure-cli).

    ```azurecli
    az login
    az account get-access-token
    ```

1. Compose the endpoint. We support the default endpoints of the Spring Cloud Config Server and Spring Cloud Service Registry managed by Azure Spring Cloud.

    * *'https://SERVICE_NAME.svc.azuremicroservices.io/eureka/{path}'*
    * *'https://SERVICE_NAME.svc.azuremicroservices.io/config/{path}'*

    >[!NOTE]
    > If you're using Azure China, replace `*.azuremicroservices.io` with `*.microservices.azure.cn`. For more information, see the section [Check endpoints in Azure](/azure/china/resources-developer-guide#check-endpoints-in-azure) in the [Azure China developer guide](/azure/china/resources-developer-guide).

1. Access the composed endpoint with the access token. Put the access token in a header to provide authorization: `--header 'Authorization: Bearer {TOKEN_FROM_PREVIOUS_STEP}`.

    For example:

    a. Access an endpoint like *'https://SERVICE_NAME.svc.azuremicroservices.io/config/actuator/health'* to see the health status of Config Server.

    b. Access an endpoint like *'https://SERVICE_NAME.svc.azuremicroservices.io/eureka/eureka/apps'* to see the registered apps in Spring Cloud Service Registry (Eureka here).

    If the response is *401 Unauthorized*, check to see if the role is successfully assigned.  It will take several minutes for the role to take effect or to verify that the access token has not expired.

For more information about actuator endpoint, see [Production ready endpoints](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints).

For Eureka endpoints, see [Eureka-REST-operations](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations)

For config server endpoints and detailed path information, see [ResourceController.java](https://github.com/spring-cloud/spring-cloud-config/blob/main/spring-cloud-config-server/src/main/java/org/springframework/cloud/config/server/resource/ResourceController.java) and [EncryptionController.java](https://github.com/spring-cloud/spring-cloud-config/blob/main/spring-cloud-config-server/src/main/java/org/springframework/cloud/config/server/encryption/EncryptionController.java).

## Register Spring Boot apps to Spring Cloud Config Server and Service Registry managed by Azure Spring Cloud

After the role is assigned, you can register Spring Boot apps to Spring Cloud Config Server and Service Registry managed by Azure Spring Cloud with Azure AD token authentication. Both Config Server and Service Registry support [custom REST template](https://cloud.spring.io/spring-cloud-config/reference/html/#custom-rest-template) to inject the bearer token for authentication.

For more information, see the samples [Access Azure Spring Cloud managed Config Server](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/custom-config-server-client) and [Access Azure Spring Cloud managed Spring Cloud Service Registry](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/custom-eureka-client). The following sections explain some important details in these samples.

**In *AccessTokenManager.java*:**

`AccessTokenManager` is responsible for getting an access token from Azure AD. Configure the service principal's sign-in information in the *application.properties* file and initialize `ApplicationTokenCredentials` to get the token. You can find this file in both samples.

```java
prop.load(in);
tokenClientId = prop.getProperty("access.token.clientId");
String tenantId = prop.getProperty("access.token.tenantId");
String secret = prop.getProperty("access.token.secret");
String clientId = prop.getProperty("access.token.clientId");
credentials = new ApplicationTokenCredentials(
    clientId, tenantId, secret, AzureEnvironment.AZURE);
```

**In *CustomConfigServiceBootstrapConfiguration.java*:**

`CustomConfigServiceBootstrapConfiguration` implements the custom REST template for Config Server and injects the token from Azure AD as `Authorization` headers. You can find this file in the [Config Server sample](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/custom-config-server-client).

```java
public class RequestResponseHandlerInterceptor implements ClientHttpRequestInterceptor {

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        String accessToken = AccessTokenManager.getToken();
        request.getHeaders().remove(AUTHORIZATION);
        request.getHeaders().add(AUTHORIZATION, "Bearer " + accessToken);

        ClientHttpResponse response = execution.execute(request, body);
        return response;
    }

}
```

**In *CustomRestTemplateTransportClientFactories.java*:**

The previous two classes are for the implementation of the custom REST template for Spring Cloud Service Registry. The `intercept` part is the same as in the Config Server above. Be sure to add `factory.mappingJacksonHttpMessageConverter()` to the message converters. You can find this file in the [Spring Cloud Service Registry sample](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/custom-eureka-client).

```java
private RestTemplate customRestTemplate() {
    /*
     * Inject your custom rest template
     */
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.getInterceptors()
        .add(new RequestResponseHandlerInterceptor());
    RestTemplateTransportClientFactory factory = new RestTemplateTransportClientFactory();

    restTemplate.getMessageConverters().add(0, factory.mappingJacksonHttpMessageConverter());

    return restTemplate;
}
```

If you're running applications on a Kubernetes cluster, we recommend that you use an IP address to register Spring Cloud Service Registry for access.

```properties
eureka.instance.prefer-ip-address=true
```

## Next steps

* [Authenticate Azure CLI](/cli/azure/authenticate-azure-cli)
* [Production ready endpoints](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)
* [Create roles and permissions](how-to-permissions.md)
