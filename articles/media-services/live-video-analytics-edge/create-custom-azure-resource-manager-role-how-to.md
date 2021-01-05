---
title: Créer un rôle Azure Resource Manager personnalisé et l’affecter à un principal du service – Azure
description: Cet article explique comment créer un rôle Azure Resource Manager personnalisé et l’affecter à un principal de service pour Live Video Analytics sur IoT Edge, à l’aide de Microsoft Azure CLI.
ms.topic: how-to
ms.date: 05/27/2020
ms.openlocfilehash: 40bf0f60a718d512e02481d977b8208112ed1a55
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92425728"
---
# <a name="create-custom-azure-resource-manager-role-and-assign-to-service-principal"></a>Créer un rôle Azure Resource Manager personnalisé et l’affecter à un principal du service

L’instance de module Live Video Analytics sur IoT Edge nécessite un compte Azure Media Services actif pour fonctionner correctement. La relation entre le module Live Video Analytics sur IoT Edge et le compte Azure Media Services est établie via un ensemble de propriétés de jumeau de module. L’une de ces propriétés est un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) qui permet à l’instance de module de communiquer avec le compte Media Services et de déclencher les opérations nécessaires via ce dernier. Pour réduire le risque d’utilisation malveillante et/ou d’exposition accidentelle des données à partir du périphérique, ce principal de service doit avoir le moins de privilèges possible.

Cet article décrit les étapes de création d’un rôle Azure Resource Manager personnalisé avec Azure Cloud Shell, qui est ensuite utilisé pour créer un principal de service.

## <a name="prerequisites"></a>Prérequis  

Pour les besoins de cet article, il vous faut les éléments suivants :

* Abonnement Azure avec abonnement propriétaire.
* Instance Azure Active Directory dotée de privilèges permettant de créer une application et d’affecter un principal de service à un rôle.

Le moyen le plus simple pour vérifier que votre compte dispose des autorisations adéquates est d’utiliser le portail. Consultez [Vérifier l’autorisation requise](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="overview"></a>Vue d’ensemble  

Nous allons passer en revue les étapes à suivre pour créer un rôle personnalisé et le lier à un principal de service, dans l’ordre suivant :

1. Créez un compte Media Services si vous n’en avez pas.
1. Créer un principal de service.
1. Créez un rôle Azure Resource Manager personnalisé avec des privilèges limités.
1. « Limitez » les privilèges du principal du service à l’aide du rôle personnalisé créé.
1. Exécutez un test simple pour voir s’il est possible de limiter le principal de service.
1. Capturez les paramètres qui seront utilisés dans les manifestes de déploiement IoT Edge.

### <a name="create-a-media-services-account"></a>Créer un compte Media Services  

Si vous n’avez pas de compte Media Services, procédez comme suit pour en créer un.

1. Accédez à [Cloud Shell](https://shell.azure.com/).
1. Sélectionnez « Bash » en tant qu’environnement dans la liste déroulante figurant sur le côté gauche de la fenêtre Cloud Shell.

    ![La capture d’écran montre Bash sélectionné dans la fenêtre de l’interpréteur de commandes.](./media/create-custom-azure-resource-manager-role-how-to/bash.png)
1. Définissez votre abonnement Azure en tant que compte par défaut à l’aide du modèle de commande suivant :
    
    ```
    az account set --subscription " <yourSubscriptionName or yourSubscriptionId>"
    ```
1. Créez un [groupe de ressources](/cli/azure/group?view=azure-cli-latest#az-group-create) et un [compte de stockage](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create).
1. À présent, créez un compte Azure Media Services à l’aide du modèle de commande suivant dans Cloud Shell :

    ```
    az ams account create --name <yourAMSAccountName>  --resource-group <yourResouceGroup>  --storage-account <yourStorageAccountName>
    ```

### <a name="create-service-principal"></a>Créer un principal du service  

Nous allons maintenant créer un principal de service et le lier à votre compte Media Services.

Sans paramètre d’authentification, l’authentification basée sur un mot de passe est utilisée avec un mot de passe aléatoire créé pour votre principal de service. Dans Cloud Shell, utilisez le modèle de commande suivant :

```
az ams account sp create --account-name < yourAMSAccountName > --resource-group < yourResouceGroup >
```

Cette commande génère une réponse semblable à celle-ci :

```
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.microsoftonline.com",
  "AadSecret": "<yourServicePrincipalPassword>",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": " <yourAMSAccountName >",
  "ArmAadAudience": "https://management.core.windows.net/",
  "ArmEndpoint": "https://management.azure.com/",
  "Region": "Central US",
  "ResourceGroup": " <yourResouceGroup >",
  "SubscriptionId": "<yourSubscriptionId>"
}

```
1. La sortie d’un principal de service avec authentification par mot de passe comprend la clé de mot de passe qui, dans ce cas, est le paramètre « AadSecret ». 

    N’oubliez pas de copier cette valeur : elle ne pourra être récupérée. Si vous oubliez le mot de passe, effectuez une [réinitialisation des informations d’identification du principal du service](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#reset-credentials).
1. L’appId et la clé de locataire s’affichent dans la sortie en tant que « AadClientId » et « AadTenantId », respectivement. Ces éléments sont utilisés pour l’authentification du principal de service. Enregistrez leurs valeurs, mais elles peuvent être récupérées à tout moment avec la [liste az ad sp](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-list).

### <a name="create-a-custom-role-definition"></a>Créer une définition de rôle personnalisé  

Pour créer un rôle personnalisé, voici comment procéder :

1. Créez un fichier JSON de définition de rôle sur votre système local et enregistrez le texte suivant dans ce fichier. 
    1. Remplacez < yourSubscriptionId> par votre ID d’abonnement Azure.
    1. Les seules actions autorisées pour ce rôle sont les suivantes :
        * listContainerSas : aide le module à répertorier les URL de conteneur de stockage avec des signatures d’accès partagé (SAP) pour le chargement et le téléchargement du contenu des ressources.
        * Write Assets : aide le module à créer ou à mettre à jour n’importe quelle ressource.
        * listEdgePolicies : répertorie les stratégies qui sont appliquées au périphérique.  
        
        ```
        {
          "Name": "LVAEdge User",
          "IsCustom": true,
          "Description": "Can create assets, view list of containers and view Edge policies",
          "Actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action"
          ],
          "NotActions": [],
          "DataActions": [],
          "NotDataActions": [],
          "AssignableScopes": [
            "/subscriptions/<yourSubscriptionId>"
          ]
        }
        ```  
          
1. Une fois qu’il créé, exécutez le modèle de commande suivant pour générer la nouvelle définition de rôle dans l’abonnement :
    
    ```
    az role definition create --role-definition "<location of the Role Definition JSON file >"
    ```

    Lorsque l’exécution de la commande est terminée, la sortie suivante s’affiche :
    
    ```
    {
      "assignableScopes": [
      "/subscriptions/<yourSubscriptionId>"
      ],
      "description": "Can create assets, view list of containers and view Edge policies",
      "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<unique name>",
      "name": "<unique name>",
      "permissions": [
        {
          "actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action",
          ],
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": " LVAEdge User ",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

### <a name="create-role-assignment"></a>Créer une attribution de rôle  

Pour ajouter une attribution de rôle, vous aurez besoin de la valeur objectId du principal de service auquel vous souhaitez affecter le rôle personnalisé que vous venez de créer.

Pour obtenir cette valeur objectId, exécutez la commande suivante dans Cloud Shell :

```
az ad sp show --id "<appId>" | Select-String "objectId"
```

> [!NOTE]
> L’élément `<appId>` peut être récupéré à partir de la sortie de l’étape [Créer un principal du service](#create-service-principal).

La commande ci-dessus affiche la valeur objectId du principal de service. 

```
“objectId” : “<yourObjectId>”,
```

Utilisez le modèle de [commande az role assignment create](/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) pour lier le rôle personnalisé au principal de service :

```
az role assignment create --role “LVAEdge User” --assignee-object-id < objectId>    
```

Paramètres :

|Paramètres|Description| 
|---|---|
|--role |Nom ou ID du rôle personnalisé. Ici : « LVAEdge User ».|
|--assignee-object-id|ID d’objet du principal de service que vous allez utiliser.|

Le résultat aura l’aspect suivant :

```
{
  "canDelegate": null,
  "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleAssignments/<yourCustomRoleId>",
  "name": "<yourCustomRoleId>",
  "principalId": "<yourServicePrincipalId>",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<yourRoleDefinitionId>",
  "scope": "/subscriptions/<yourSubscriptionId>",
  "type": "Microsoft.Authorization/roleAssignments"
} 
```

### <a name="confirm-that-role-assignment-happened"></a>Confirmer la réussite de l’attribution de rôle

Pour confirmer que le principal du service est maintenant lié au rôle personnalisé que nous venons de créer, exécutez la commande suivante :

```
az role assignment list  --assignee < objectId>
```

Le résultat doit avoir l’aspect suivant :

```
[
  {
    "canDelegate": null,
    "id": "/subscriptions/xxx/providers/Microsoft.Authorization/roleAssignments/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "principalId": "<yourServicePrincipalID>",
    "principalName": "<yourServicePrincipalName>",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/xxx/providers/Microsoft.Authorization/roleDefinitions/zzz",
    "roleDefinitionName": "LVAEdge User",
    "scope": "/subscriptions/<yourSubscription ID>",
    "type": "Microsoft.Authorization/roleAssignments
  }
]  
```
 
Recherchez « roleDefinitionName » et vérifiez que sa valeur est définie sur « LVAEdge User ». 

Cela confirme que nous avons bien lié le rôle d’utilisateur personnalisé au principal de service utilisé pour notre application.

### <a name="test-the-service-principal-access-control"></a>Tester le contrôle d’accès du principal du service

1. Connectez-vous avec le principal de service. Pour ce faire, nous avons besoin de trois informations permettant à Azure Active Directory de nous accorder le jeton d’accès approprié, que nous pouvons obtenir à partir de la sortie de l’étape [Créer un principal du service](#create-service-principal) :
    1. AadClientID 
    1. AadSecret
    1. AadTenantId
1. À présent, essayons de nous connecter à l’aide du modèle de commande ci-dessous :
    
    ```
    az login --service-principal --username "< AadClientID>" --password " <AadSecret>" --tenant "<AadTenantId>"
    ```
3.  Voyons si la connexion est limitée au principal du service avec le rôle « Utilisateur LVAEdge », en tentant de créer un groupe de ressources pour s’assurer que cela échoue. Exécutez la commande suivante dans Cloud Shell :

    ```
    az group create --location "central us" --name "testresourcegroup"
    ```

    Cette commande doit échouer et se présenter comme suit :
    
    ```
    The client '<AadClientId>' with object id '<AadClientId>' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/write' over scope '/subscriptions/<yourSubscriptionId>/resourcegroups/testresourcegroup' or the scope is invalid. If access was recently granted, please refresh your credentials.
    ```

## <a name="next-steps"></a>Étapes suivantes  

Notez les valeurs suivantes. Ces valeurs sont requises pour vous aider à configurer les propriétés du jumeau de module associé à Live Video Analytics sur IoT Edge. Consultez la section relative au [schéma JSON du jumeau de module](module-twin-configuration-schema.md).

| Variable de cet article|Nom de la propriété du jumeau pour Live Video Analytics sur IoT Edge|
|---|---|
|AadSecret |    aadServicePrincipalPassword|
|AadTenantId |  aadTenantId|
|AadClientId    |aadServicePrincipalAppId|
