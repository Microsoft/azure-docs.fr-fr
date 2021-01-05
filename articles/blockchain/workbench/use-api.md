---
title: Utilisation des API REST Azure Blockchain Workbench
description: Scénarios d’utilisation de l’API REST d’Azure Blockchain Workbench Preview
ms.date: 03/05/2020
ms.topic: how-to
ms.reviewer: brendal
ms.openlocfilehash: 696f1f2f96034f7a044f6a39182774c02804518f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004838"
---
# <a name="using-the-azure-blockchain-workbench-preview-rest-api"></a>Utilisation de l’API REST d’Azure Blockchain Workbench Preview

L’API REST d’Azure Blockchain Workbench Preview permet aux développeurs et aux professionnels de l’information de créer des intégrations riches à des applications blockchain. Cet article présente plusieurs scénarios d’utilisation de l’API REST Workbench. Supposons, par exemple, que vous souhaitez créer un client blockchain personnalisé pour permettre aux utilisateurs connectés d’afficher les applications blockchain qui leur sont attribuées et d’interagir avec elles. Ce client peut utiliser l’API Blockchain Workbench pour afficher les instances du contrat et intervenir sur les contrats intelligents.

## <a name="blockchain-workbench-api-endpoint"></a>Point de terminaison d’API Blockchain Workbench

Les API Blockchain Workbench sont accessibles via un point de terminaison pour votre déploiement. Pour obtenir l’URL du point de terminaison de l’API pour votre déploiement :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.
1. Choisissez le nom du groupe de ressources de votre Blockchain Workbench déployé.
1. Sélectionnez l’en-tête de colonne **TYPE** pour trier la liste par ordre alphabétique et par type.
1. Deux ressources présentent le type **App Service**. Sélectionnez la ressource de type **App Service** *avec* le suffixe « -api ».
1. Dans la section **Vue d’ensemble** App Service, copiez la valeur **URL** qui représente l’URL du point de terminaison de l’API sur votre Blockchain Workbench déployé.

    ![URL du point de terminaison de l'API App Service](media/use-api/app-service-api.png)

## <a name="authentication"></a>Authentification

Les requêtes envoyées à l’API REST Blockchain Workbench sont protégées par Azure Active Directory (Azure AD).

Pour effectuer une requête authentifiée auprès des API REST, le code client requiert une authentification à l’aide d’informations d’identification valides avant de pouvoir appeler l’API. L’authentification est coordonnée entre les différents intervenants par Azure AD et fournit à votre client un [jeton d’accès](../../active-directory/develop/developer-glossary.md#access-token) comme preuve d’authentification. Le jeton est alors envoyé dans l’en-tête d’autorisation HTTP des requêtes de l’API REST. Pour en savoir plus sur l’authentification Azure AD, consultez [Azure Active Directory pour les développeurs](../../active-directory/develop/index.yml).

Pour obtenir des exemples d’authentification, consultez [REST API samples](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples).

## <a name="using-postman"></a>Utilisation de Postman

Si vous souhaitez tester ou expérimenter les API Workbench, vous pouvez utiliser [Postman](https://www.postman.com) pour envoyer des appels d’API à votre déploiement. [Téléchargez un exemple de la collection Postman composée de requêtes d’API Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/rest-api-samples/postman) à partir de GitHub. Pour plus d’informations sur l’authentification et l’utilisation des exemples de requêtes d’API, consultez le fichier LISEZ-MOI.

## <a name="create-an-application"></a>Créer une application

Vous utilisez deux appels d’API pour créer une application Blockchain Workbench. Cette méthode ne peut être effectuée que par les utilisateurs qui sont administrateurs Workbench.

Utilisez l’[API POST Applications](/rest/api/azure-blockchain-workbench/applications/applicationspost) pour charger le fichier JSON de l’application et récupérer un ID d’application.

### <a name="applications-post-request"></a>Requête POST des applications

Utilisez le paramètre **appFile** pour envoyer le fichier config en tant que partie du corps de la demande.

``` http
POST /api/v1/applications
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="appFile"; filename="/C:/smart-contract-samples/HelloWorld.json"
Content-Type: application/json
```

### <a name="applications-post-response"></a>Réponse POST des applications

L’ID d’application créé est retourné dans la réponse. Vous avez besoin de l’ID d’application pour associer le fichier config au fichier de code lorsque vous appelez l’API suivante.

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
1
```

### <a name="contract-code-post-request"></a>Requête POST du code de contrat

Utilisez l’[API POST de code de contrat d’applications](/rest/api/azure-blockchain-workbench/applications/contractcodepost) en transmettant l’ID d’application pour charger le fichier de code Solidity de l’application. La charge utile peut être un fichier Solidity unique ou un fichier compressé contenant des fichiers Solidity.

Remplacez les valeurs suivantes :

| Paramètre | Valeur |
|-----------|-------|
| {applicationId} | Valeur renvoyée de l’API POST Applications. |
| {ledgerId} | Index du registre. La valeur est généralement 1. Vous pouvez également vérifier la valeur dans la [table Ledger](data-sql-management-studio.md). |

``` http
POST /api/v1/applications/{applicationId}/contractCode?ledgerId={ledgerId}
Content-Type: multipart/form-data;
Authorization : Bearer {access token}
Content-Disposition: form-data; name="contractFile"; filename="/C:/smart-contract-samples/HelloWorld.sol"
```

### <a name="contract-code-post-response"></a>Réponse POST de code de contrat

En cas de réussite, la réponse comprend l’ID de code de contrat créé de la [table ContractCode](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
Content-Type: "application/json"
2
```

## <a name="assign-roles-to-users"></a>Attribuer des rôles aux utilisateurs

Utilisez l’[API POST des attributions de rôles d’applications](/rest/api/azure-blockchain-workbench/applications/roleassignmentspost) en transmettant l’ID d’application, l’identifiant utilisateur et l’ID de rôle d’application pour créer un mappage utilisateur-rôle dans l’application blockchain spécifiée. Cette méthode ne peut être effectuée que par les utilisateurs qui sont administrateurs Workbench.

### <a name="role-assignments-post-request"></a>Requête POST des attributions de rôles

Remplacez les valeurs suivantes :

| Paramètre | Valeur |
|-----------|-------|
| {applicationId} | Valeur renvoyée de l’API POST Applications. |
| {userId} | Valeur de l’ID utilisateur dans la [table User](data-sql-management-studio.md). |
| {applicationRoleId} | Valeur d’ID de rôle d’application associée à l’ID d’application dans la [table ApplicationRole](data-sql-management-studio.md). |

``` http
POST /api/v1/applications/{applicationId}/roleAssignments
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "userId": {userId},
  "applicationRoleId": {applicationRoleId}
}
```

### <a name="role-assignments-post-response"></a>Réponse POST des attributions de rôles

En cas de réussite, la réponse comprend l’ID d’attribution de rôle créé à partir de la [table RoleAssignment](data-sql-management-studio.md).

``` http
HTTP/1.1 200
1
```

## <a name="list-applications"></a>Liste des applications

Utilisez l’[API GET Applications](/rest/api/azure-blockchain-workbench/applications/applicationsget) pour récupérer toutes les applications Blockchain Workbench pour l’utilisateur. Dans cet exemple, l’utilisateur connecté a accès à deux applications :

- [Asset transfer](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer/readme.md) (Transfert d’actifs)
- [Refrigerated transportation](https://github.com/Azure-Samples/blockchain/blob/master/blockchain-workbench/application-and-smart-contract-samples/refrigerated-transportation/readme.md) (Transport frigorifique)

### <a name="applications-get-request"></a>Requête GET des applications

``` http
GET /api/v1/applications
Authorization : Bearer {access token}
```

### <a name="applications-get-response"></a>Réponse GET des applications

La réponse recense toutes les applications blockchain auxquelles un utilisateur a accès dans Blockchain Workbench. Les administrateurs Blockchain Workbench obtiennent toutes les applications blockchain. Les administrateurs non Workbench obtiennent toutes les applications blockchain pour lesquelles ils ont au moins un rôle d’application associé ou un rôle d’instance de contrat intelligent associé.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications?skip=2",
    "applications": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
            "displayName": "Asset Transfer",
            "createdByUserId": 1,
            "createdDtTm": "2018-04-28T05:59:14.4733333",
            "enabled": true,
            "applicationRoles": null
        },
        {
            "id": 2,
            "name": "RefrigeratedTransportation",
            "description": "Application to track end-to-end transportation of perishable goods.",
            "displayName": "Refrigerated Transportation",
            "createdByUserId": 7,
            "createdDtTm": "2018-04-28T18:25:38.71",
            "enabled": true,
            "applicationRoles": null
        }
    ]
}
```

## <a name="list-workflows-for-an-application"></a>Répertorier les flux de travail d’une application

Utilisez l’[API GET Applications Workflows](/rest/api/azure-blockchain-workbench/applications/workflowsget) pour répertorier tous les workflows d’une application blockchain spécifiée à laquelle un utilisateur a accès dans Blockchain Workbench. Chaque application blockchain possède un ou plusieurs flux de travail, et chaque flux de travail possède zéro instance de contrat ou des instances de contrat intelligent. Pour une application cliente blockchain comportant un seul workflow, nous recommandons d’éviter le flux d’expérience utilisateur qui permet aux utilisateurs de sélectionner le workflow approprié.

### <a name="application-workflows-request"></a>Requête de workflows d’application

``` http
GET /api/v1/applications/{applicationId}/workflows
Authorization: Bearer {access token}
```

### <a name="application-workflows-response"></a>Réponse de workflows d’application

Les administrateurs Blockchain Workbench obtiennent tous les workflows blockchain. Les administrateurs non Workbench obtiennent tous les workflows pour lesquels ils ont au moins un rôle d’application associé ou un rôle d’instance de contrat intelligent associé.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/applications/1/workflows?skip=1",
    "workflows": [
        {
            "id": 1,
            "name": "AssetTransfer",
            "description": "Handles the business logic for the asset transfer scenario",
            "displayName": "Asset Transfer",
            "applicationId": 1,
            "constructorId": 1,
            "startStateId": 1
        }
    ]
}
```

## <a name="create-a-contract-instance"></a>Créer une instance de contrat

Utilisez l’[API POST Contracts V2](/rest/api/azure-blockchain-workbench/contractsv2/contractpost) pour créer une nouvelle instance de contrat intelligent pour un workflow. Les utilisateurs peuvent uniquement créer une nouvelle instance de contrat intelligent si l’utilisateur est associé à un rôle d’application qui peut lancer une instance de contrat intelligent pour le workflow.

> [!NOTE]
> Dans cet exemple, la version 2 de l’API est utilisée. Les API de contrat version 2 offrent une plus grande granularité pour les champs ProvisioningStatus associés.

### <a name="contracts-post-request"></a>Requête POST des contrats

Remplacez les valeurs suivantes :

| Paramètre | Valeur |
|-----------|-------|
| {workflowId} | La valeur de l’ID de workflow est le ConstructorID du contrat dans la [table Workflow](data-sql-management-studio.md). |
| {contractCodeId} | Valeur d’ID du code de contrat de la [table ContractCode](data-sql-management-studio.md). Mettez en corrélation l’ID de l’application et l’ID du registre pour l’instance de contrat que vous souhaitez créer. |
| {connectionId} | Valeur de l’ID de connexion de la [table Connection](data-sql-management-studio.md). |

Pour le corps de la demande, définissez des valeurs à l’aide des informations suivantes :

| Paramètre | Valeur |
|-----------|-------|
| workflowFunctionID | ID de la [table WorkflowFunction](data-sql-management-studio.md). |
| workflowActionParameters | Paires nom/valeur des paramètres transmis au constructeur. Pour chaque paramètre, utilisez la valeur workflowFunctionParameterID de la table [WorkflowFunctionParameter](data-sql-management-studio.md). |

``` http
POST /api/v2/contracts?workflowId={workflowId}&contractCodeId={contractCodeId}&connectionId={connectionId}
Content-Type: application/json;
Authorization : Bearer {access token}

{
  "workflowFunctionID": 2,
  "workflowActionParameters": [
    {
      "name": "message",
      "value": "Hello, world!",
      "workflowFunctionParameterId": 3
    }
  ]
}
```

### <a name="contracts-post-response"></a>Réponse POST des contrats

En cas de réussite, l’API des attributions de rôles retourne ContractActionID à partir de la [table ContractActionParameter](data-sql-management-studio.md).

``` http
HTTP/1.1 200 OK
4
```

## <a name="list-smart-contract-instances-for-a-workflow"></a>Répertorier les instances de contrat intelligent d’un flux de travail

Utilisez l’[API GET Contracts](/rest/api/azure-blockchain-workbench/contractsv2/contractsget) pour afficher toutes les instances de contrat intelligent pour un workflow. Vous pouvez également permettre aux utilisateurs d’explorer en profondeur l’une des instances de contrat intelligent présentées.

### <a name="contracts-request"></a>Requête de contrats

Dans cet exemple, imaginons qu’un utilisateur souhaite intervenir et ainsi interagir avec l’une des instances de contrat intelligent.

``` http
GET api/v1/contracts?workflowId={workflowId}
Authorization: Bearer {access token}
```

### <a name="contracts-response"></a>Réponse de contrats

La réponse recense toutes les instances de contrat intelligent du workflow spécifié. Les administrateurs Workbench obtiennent toutes les instances de contrat intelligent. Les administrateurs non Workbench obtiennent toutes les instances de contrat intelligent pour lesquelles ils ont au moins un rôle d’application associé ou un rôle d’instance de contrat intelligent associé.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts?skip=3&workflowId=1",
    "contracts": [
        {
            "id": 1,
            "provisioningStatus": 2,
            "connectionID": 1,
            "ledgerIdentifier": "0xbcb6127be062acd37818af290c0e43479a153a1c",
            "deployedByUserId": 1,
            "workflowId": 1,
            "contractCodeId": 1,
            "contractProperties": [
                {
                    "workflowPropertyId": 1,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 2,
                    "value": "My first car"
                },
                {
                    "workflowPropertyId": 3,
                    "value": "54321"
                },
                {
                    "workflowPropertyId": 4,
                    "value": "0"
                },
                {
                    "workflowPropertyId": 5,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 6,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 7,
                    "value": "0x0000000000000000000000000000000000000000"
                },
                {
                    "workflowPropertyId": 8,
                    "value": "0xd882530eb3d6395e697508287900c7679dbe02d7"
                }
            ],
            "transactions": [
                {
                    "id": 1,
                    "connectionId": 1,
                    "transactionHash": "0xf3abb829884dc396e03ae9e115a770b230fcf41bb03d39457201449e077080f4",
                    "blockID": 241,
                    "from": "0xd882530eb3d6395e697508287900c7679dbe02d7",
                    "to": null,
                    "value": 0,
                    "isAppBuilderTx": true
                }
            ],
            "contractActions": [
                {
                    "id": 1,
                    "userId": 1,
                    "provisioningStatus": 2,
                    "timestamp": "2018-04-29T23:41:14.9333333",
                    "parameters": [
                        {
                            "name": "Description",
                            "value": "My first car"
                        },
                        {
                            "name": "Price",
                            "value": "54321"
                        }
                    ],
                    "workflowFunctionId": 1,
                    "transactionId": 1,
                    "workflowStateId": 1
                }
            ]
        }
    ]
}
```

## <a name="list-available-actions-for-a-contract"></a>Répertorier les actions disponibles d’un contrat

Utilisez l’[API GET Contract Action](/rest/api/azure-blockchain-workbench/contractsv2/contractactionget) pour afficher les actions utilisateur disponibles en fonction de l’état du contrat. 

### <a name="contract-action-request"></a>Requête d’action de contrat

Dans cet exemple, l’utilisateur recherche toutes les actions disponibles pour un nouveau contrat intelligent qu’il a créé.

``` http
GET /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
```

### <a name="contract-action-response"></a>Réponse d’action de contrat

La réponse recense toutes les actions qu’un utilisateur peut entreprendre étant donné l’état actuel de l’instance de contrat intelligent spécifiée.

* Modifier : permet à l’utilisateur de modifier la description et le prix d’une ressource.
* Terminer : permet à l’utilisateur de mettre fin au contrat de la ressource.

Les utilisateurs obtiennent toutes les actions applicables si l’utilisateur possède un rôle d’application associé ou est associé à un rôle d’instance de contrat intelligent pour l’état actuel de l’instance de contrat intelligent spécifiée.

``` http
HTTP/1.1 200 OK
Content-type: application/json
{
    "nextLink": "/api/v1/contracts/1/actions?skip=2",
    "workflowFunctions": [
        {
            "id": 2,
            "name": "Modify",
            "description": "Modify the description/price attributes of this asset transfer instance",
            "displayName": "Modify",
            "parameters": [
                {
                    "id": 1,
                    "name": "description",
                    "description": "The new description of the asset",
                    "displayName": "Description",
                    "type": {
                        "id": 2,
                        "name": "string",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                },
                {
                    "id": 2,
                    "name": "price",
                    "description": "The new price of the asset",
                    "displayName": "Price",
                    "type": {
                        "id": 3,
                        "name": "money",
                        "elementType": null,
                        "elementTypeId": 0
                    }
                }
            ],
            "workflowId": 1
        },
        {
            "id": 3,
            "name": "Terminate",
            "description": "Used to cancel this particular instance of asset transfer",
            "displayName": "Terminate",
            "parameters": [],
            "workflowId": 1
        }
    ]
}
```

## <a name="execute-an-action-for-a-contract"></a>Exécuter une action pour un contrat

Utilisez l’[API POST Contract Action](/rest/api/azure-blockchain-workbench/contractsv2/contractactionpost) pour effectuer une action pour l’instance de contrat intelligent spécifiée.

### <a name="contract-action-post-request"></a>Requête POST d’action de contrat

Dans ce cas, imaginons qu’un utilisateur souhaite modifier la description et le prix d’une ressource.

``` http
POST /api/v1/contracts/{contractId}/actions
Authorization: Bearer {access token}
actionInformation: {
    "workflowFunctionId": 2,
    "workflowActionParameters": [
        {
            "name": "description",
            "value": "My updated car"
        },
        {
            "name": "price",
            "value": "54321"
        }
    ]
}
```

L’utilisateur est uniquement en mesure d’intervenir en fonction de l’état actuel de l’instance de contrat intelligent spécifiée et de son rôle d’application ou de son rôle d’instance de contrat intelligent associé.

### <a name="contract-action-post-response"></a>Réponse POST d’action de contrat

Si la publication aboutit, une réponse HTTP 200 OK est renvoyée sans corps de réponse.

``` http
HTTP/1.1 200 OK
Content-type: application/json
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des informations de référence sur les API Blockchain Workbench, consultez la [documentation de référence sur l’API REST Azure Blockchain Workbench](/rest/api/azure-blockchain-workbench).
