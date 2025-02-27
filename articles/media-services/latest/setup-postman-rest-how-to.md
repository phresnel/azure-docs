---
title: Configure Postman for Azure Media Services v3 REST API
description: This article shows you how to configure Postman so it can be used to call Azure Media Services (AMS) REST APIs.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
---
# Configure Postman for Media Services v3 REST API calls

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

This article shows you how to configure **Postman** so it can be used to call Azure Media Services (AMS) REST APIs. This is provided as a learning tool, and not recommended for production applications. Production applications should use the supported client SDKs, which contain Azure Resource Management retry policies built-in.

[!INCLUDE [warning-rest-api-retry-policy.md](./includes/warning-rest-api-retry-policy.md)]

The article shows how to import environment and collection files into **Postman**. The collection contains grouped definitions of HTTP requests that call Azure Media Services (AMS) REST APIs. The environment file contains variables that are used by the collection.

Before you start developing, review [Developing with Media Services v3 APIs](media-services-apis-overview.md).

## Prerequisites

- [Create a Media Services account](./account-create-how-to.md). Make sure to remember the resource group name and the Media Services account name. 
- Get information needed to [access APIs](./access-api-howto.md)
- Install the [Postman](https://www.getpostman.com/) REST client to execute the REST APIs shown in some of the AMS REST tutorials. 

    We are using **Postman** but any REST tool would be suitable. Other alternatives are: **Visual Studio Code** with the REST plugin or **Telerik Fiddler**. 

> [!IMPORTANT]
> Review [naming conventions](media-services-apis-overview.md#naming-conventions).

## Download Postman files

Clone a GitHub repository that contains the  Postman collection and environment files.

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

## Configure Postman

### Configure the environment 

1. Open the **Postman** app.
2. On the right of the screen, select the **Manage environment** option.

    ![Manage env](./media/develop-with-postman/postman-import-env.png)
4. From the **Manage environment** dialog, click **Import**.
2. Browse to the `Azure Media Service v3 Environment.postman_environment.json` file that was downloaded when you cloned `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
6. The **Azure Media Service v3 Environment** environment is added.

    > [!Note]
    > Update access variables with values you got from the **Access the Media Services API** section above.

7. Double-click on the selected file and enter values that you got by following the accessing API steps.
8. Close the dialog.
9. Select the **Azure Media Service v3 Environment** environment from the dropdown.

    ![Choose env](./media/develop-with-postman/choose-env.png)
   
### Configure the collection

1. Click **Import** to import the collection file.
1. Browse to the `Media Services v3.postman_collection.json` file that was downloaded when you cloned `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`
3. Choose the **Media Services v3.postman_collection.json** file.

    ![Import a file](./media/develop-with-postman/postman-import-collection.png)

## Get Azure AD Token 

Before you start manipulating AMS v3 resources you need to get and set Azure AD Token for Service Principal Authentication.

1. In the left window of the Postman app, select "Step 1: Get AAD Auth token".
2. Then, select "Get Azure AD Token for Service Principal Authentication".
3. Press **Send**.

    The following **POST** operation is sent.

    ```
    https://login.microsoftonline.com/:tenantId/oauth2/token
    ```

4. The response comes back with the token and sets the "AccessToken" environment variable to the token value.  

    ![Get AAD token](./media/develop-with-postman/postman-get-aad-auth-token.png)

## Troubleshooting 

* If your application fails with "HTTP 504: Gateway Timeout", make sure that the location variable has not been explicitly set to a value other than the expected location of the Media Services account. 
* If you get an "account not found" error, also check to make sure that the location property in the Body JSON message is set to the location that the Media Services account is in. 

## See also

- [Create filters with Media Services - REST](filters-dynamic-manifest-rest-howto.md)
- [Azure Resource Manager based REST API](https://github.com/Azure-Samples/media-services-v3-arm-templates)

## Next steps

- [Stream files with REST](stream-files-tutorial-with-rest.md).  
- [Tutorial: Encode a remote file based on URL and stream the video - REST](stream-files-tutorial-with-rest.md)
