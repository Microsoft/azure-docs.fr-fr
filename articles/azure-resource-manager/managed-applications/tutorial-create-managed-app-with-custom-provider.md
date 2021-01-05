---
title: 'Tutoriel : Actions et ressources personnalisées'
description: Ce tutoriel explique comment créer une application managée Azure avec un fournisseur personnalisé Azure.
ms.topic: tutorial
ms.author: lazinnat
author: lazinnat
ms.date: 06/20/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 50a65583c41dd94b174a33432afcf42b31e67df0
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96437046"
---
# <a name="tutorial-create-managed-application-with-custom-actions-and-resources"></a>Tutoriel : Créer une application managée avec des actions et des ressources personnalisées

Dans ce tutoriel, vous allez créer votre propre application managée avec des actions et des ressources personnalisées. L’application managée contient une action personnalisée dans la page `Overview`, un type de ressource personnalisé affiché sous la forme d’un élément de menu distinct dans `Table of Content`, et une action contextuelle personnalisée dans la page des ressources personnalisées.

Ce tutoriel comprend les étapes suivantes :

> [!div class="checklist"]
> * Créer un fichier de définition d’interface utilisateur pour la création d’une instance d’application managée
> * Créer un modèle de déploiement avec un [fournisseur personnalisé Azure](../custom-providers/overview.md), un compte Stockage Azure et une fonction Azure
> * Créer un artefact de définition de vue avec des actions et des ressources personnalisées
> * Déployer une définition d’application managée
> * Déployer une instance d’application managée
> * Exécuter des actions personnalisées et créer des ressources personnalisées

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez savoir :

* Comment [Créer et publier une définition d’application managée](publish-service-catalog-app.md).
* Comment [Déployer une application du catalogue de services à l’aide du portail Microsoft Azure](deploy-service-catalog-quickstart.md).
* Comment [Créer une interface utilisateur de portail Azure pour votre application managée](create-uidefinition-overview.md).
* Comment fonctionnent les capacités d’[artefact de définition de vue](concepts-view-definition.md).
* Comment fonctionnent les capacités de [fournisseur personnalisé Azure](../custom-providers/overview.md).

## <a name="user-interface-definition"></a>Définition d’interface utilisateur

Dans ce tutoriel, vous allez créer une application managée et son groupe de ressources managé contenant une instance de fournisseur personnalisé, un compte de stockage et une fonction. La fonction Azure utilisée dans cet exemple implémente une API qui gère des opérations de fournisseur personnalisé pour des actions et des ressources. Le compte Stockage Azure est utilisé comme stockage de base pour vos ressources de fournisseur personnalisé.

La définition de l’interface utilisateur pour la création d’une instance d’application managée comprend des éléments d’entrée `funcname` et `storagename`. Le nom du compte de stockage et le nom de la fonction doivent être globalement uniques. Par défaut, les fichiers de fonction seront déployés à partir d’un [exemple de package de fonction](https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip), mais vous pouvez changer cela en ajoutant un élément d’entrée pour un lien de package dans *createUiDefinition.json* :

```json
{
  "name": "funcname",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the function to be created",
  "toolTip": "Name of the function to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "storagename",
  "type": "Microsoft.Common.TextBox",
  "label": "Name of the storage to be created",
  "toolTip": "Name of the storage to be created",
  "visible": true,
  "constraints": {
    "required": true
  }
},
{
  "name": "zipFileBlobUri",
  "type": "Microsoft.Common.TextBox",
  "defaultValue": "https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip/functionpackage.zip",
  "label": "The Uri to the uploaded function zip file",
  "toolTip": "The Uri to the uploaded function zip file",
  "visible": true
}
```

et une sortie dans *createUiDefinition.json* :

```json
  "funcname": "[steps('applicationSettings').funcname]",
  "storageName": "[steps('applicationSettings').storagename]",
  "zipFileBlobUri": "[steps('applicationSettings').zipFileBlobUri]"
```

L’exemple *createUiDefinition.json* complet se trouve dans [Référence : Artefacts des éléments d’interface utilisateur](reference-createuidefinition-artifact.md).

## <a name="template-with-custom-provider"></a>Modèle avec fournisseur personnalisé

Pour créer une instance d’application managée avec un fournisseur personnalisé, vous devez définir une ressource de fournisseur personnalisé avec le nom **public** et le type **Microsoft.CustomProviders/resourceProviders** dans votre fichier **mainTemplate.json**. Dans cette ressource, vous définissez les types de ressources et les actions de votre service. Pour déployer des instances de fonction Azure et de compte Stockage Azure, définissez des ressources de type `Microsoft.Web/sites` et `Microsoft.Storage/storageAccounts`, respectivement.

Dans ce tutoriel, vous allez créer un type de ressource `users`, une action personnalisée `ping`, et une action personnalisée `users/contextAction` qui sera exécutée dans le contexte d’une ressource personnalisée `users`. Pour chaque type de ressource et action, fournissez un point de terminaison pointant vers la fonction nommée dans [createUiDefinition.json](#user-interface-definition). Comme **routingType**, affectez la valeur `Proxy,Cache` pour les types de ressources et `Proxy` pour les actions :

```json
{
  "apiVersion": "[variables('customrpApiversion')]",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "name": "[variables('customProviderName')]",
  "location": "[parameters('location')]",
  "properties": {
    "actions": [
      {
        "name": "ping",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      },
      {
        "name": "users/contextAction",
        "routingType": "Proxy",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ],
    "resourceTypes": [
      {
        "name": "users",
        "routingType": "Proxy,Cache",
        "endpoint": "[listSecrets(resourceId('Microsoft.Web/sites/functions', parameters('funcname'), 'HttpTrigger1'), '2018-02-01').trigger_url]"
      }
    ]
  },
  "dependsOn": [
    "[concat('Microsoft.Web/sites/',parameters('funcname'))]"
  ]
}
```

L’exemple *mainTemplate.json* complet se trouve dans [Référence : Artefact de modèle de déploiement](reference-main-template-artifact.md).

## <a name="view-definition-artifact"></a>Afficher un artefact de définition

Pour définir une interface utilisateur qui comprend des actions personnalisées et des ressources personnalisées dans votre application managée, vous devez créer un artefact **viewDefinition.json**. Pour plus d’informations sur les artefacts de définition de vue, consultez [Artefact de définition de vue dans Applications managées Azure](concepts-view-definition.md).

Dans ce tutoriel, vous allez définir :
* Une page *Overview* avec un bouton de barre d’outils qui représente une action personnalisée `TestAction` avec une entrée de texte de base.
* Une page *Users* qui représente un type de ressource personnalisé `users`.
* Une action de ressource personnalisée `users/contextAction` dans la page *Users* qui sera exécutée dans un contexte de ressource personnalisée de type `users`.

L’exemple suivant illustre la configuration de vue pour une page « Overview » :

```json
{
    "kind": "Overview",
    "properties": {
      "header": "Welcome to your Demo Azure Managed Application",
      "description": "This Managed application with Custom Provider is for demo purposes only.",
      "commands": [{
          "displayName": "Ping Action",
          "path": "/customping",
          "icon": "LaunchCurrent"
      }]
    }
  }
```

L’exemple ci-dessous comprend la configuration de la page de ressources « Users » avec l’action de ressource personnalisée :

```json
{
    "kind": "CustomResources",
    "properties": {
      "displayName": "Users",
      "version": "1.0.0.0",
      "resourceType": "users",
      "createUIDefinition": {
      },
      "commands": [{
        "displayName": "Custom Context Action",
        "path": "users/contextAction",
        "icon": "Start"
      }],
      "columns": [
        { "key": "properties.FullName", "displayName": "Full Name" },
        { "key": "properties.Location", "displayName": "Location", "optional": true }
      ]
    }
  }
```

L’exemple *viewDefinition.json* complet se trouve dans [Référence : Artefact de définition de vue](reference-view-definition-artifact.md).

## <a name="managed-application-definition"></a>Définition d’application managée

Empaquetez les artefacts d’application managée suivants dans une archive zip et chargez-les dans le stockage :

* createUiDefinition.json
* mainTemplate.json
* viewDefinition.json

Tous les fichiers doivent se trouver au niveau racine. Le package avec les artefacts peut être stocké dans n’importe quel stockage, par exemple un objet blob GitHub ou un objet blob de compte Stockage Azure. Voici un script permettant de charger le package d’application dans un compte de stockage : 

```powershell
$resourceGroup="appResourcesGroup"
$storageName="mystorageaccount$RANDOM"

# Sign in to your Azure subscription
Connect-AzAccount
# Create resource group for managed application definition and application package
New-AzResourceGroup -Name $resourceGroup -Location eastus

# Create storage account for a package with application artifacts
$storageAccount=New-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -SkuName Standard_LRS `
  -Location eastus `
$ctx=$storageAccount.Context

# Create storage container and upload zip to blob
New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob
Set-AzStorageBlobContent `
  -File "path_to_your_zip_package" `
  -Container appcontainer `
  -Blob app.zip `
  -Context $ctx 

# Get blob absolute uri
$blobUri=(Get-AzureStorageBlob -Container appcontainer -Blob app.zip -Context $ctx).ICloudBlob.uri.AbsoluteUri
```

Exécutez le script Azure CLI ci-dessous ou effectuez les étapes dans le portail Azure pour déployer une définition d’application managée du catalogue de services :

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli-interactive)

```azurecli-interactive
resourceGroup="appResourcesGroup"
# Select subscription and create resource group (if you have not created yet)
az account set --subscription <subscriptionID>
az group create --name $resourceGroup --location eastus

# Get object ID of your identity
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
# Get role definition ID for the Owner role
roleid=$(az role definition list --name Owner --query [].name --output tsv)

# Create managed application definition resource
az managedapp definition create \
  --name "ManagedUsersAppDefinition" \
  --location "eastus" \
  --resource-group $resourceGroup \
  --lock-level ReadOnly \
  --display-name "Managed users app definition" \
  --description "Managed application with Azure Custom Provider" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "path to your app.zip package"
```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste des ressources, tapez et sélectionnez **Centre d’applications managées**.
2. Dans le **Centre d’applications managées**, choisissez **Définition d’application du catalogue de services**, puis cliquez sur **Ajouter**. 
    
    ![Ajouter un catalogue de services](./media/tutorial-create-managed-app-with-custom-provider/service-catalog-managed-application.png)

3. Fournissez des valeurs pour créer une définition de catalogue de services :

    * Fournissez un **Nom** unique pour la définition du catalogue de services, un **Nom complet** et une *Description* (facultative).
    * Sélectionnez l’**Abonnement**, le **Groupe de ressources** et l’**Emplacement** où la définition d’application sera créée. Vous pouvez utiliser le même groupe de ressources que celui utilisé pour le package zip, ou en créer un.
    * Pour **URI du fichier de package** , indiquez le chemin du fichier zip que vous avez créé à l’étape précédente.

    ![Fournir des valeurs](./media/tutorial-create-managed-app-with-custom-provider/add-service-catalog-managed-application.png)

4. Lorsque vous accédez à la section Authentification et niveau de verrouillage, sélectionnez **Ajouter une autorisation**.

    ![Ajouter une autorisation](./media/tutorial-create-managed-app-with-custom-provider/add-authorization.png)

5. Choisissez un groupe Azure Active Directory pour gérer les ressources, puis sélectionnez **OK**.

   ![Ajouter un groupe d’autorisations](./media/tutorial-create-managed-app-with-custom-provider/add-auth-group.png)

6. Après avoir fourni toutes les valeurs, sélectionnez **Créer**.

   ![Créer la définition d’application gérée](./media/tutorial-create-managed-app-with-custom-provider/create-service-catalog-definition.png)

---

## <a name="managed-application-instance"></a>Instance d’application managée

Quand la définition d’application managée est déployée, exécutez le script ci-dessous ou effectuez les étapes dans le portail Azure pour déployer votre instance d’application managée avec un fournisseur personnalisé :

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli-interactive)

```azurecli-interactive
appResourcesGroup="appResourcesGroup"
applicationGroup="usersApplicationGroup"

# Create resource group for managed application instance
az group create --name $applicationGroup --location eastus

# Get ID of managed application definition
appid=$(az managedapp definition show --name ManagedUsersAppDefinition --resource-group $appResourcesGroup --query id --output tsv)

# Create the managed application
az managedapp create \
  --name ManagedUsersApp \
  --location "eastus" \
  --kind "Servicecatalog" \
  --resource-group $applicationGroup \
  --managedapp-definition-id $appid \
  --managed-rg-id "managedResourcesGroup" \
  --parameters "{\"funcname\": {\"value\": \"managedusersappfunction\"}, \"storageName\": {\"value\": \"managedusersappstorage\"}}"
```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste des ressources, tapez et sélectionnez **Centre d’applications managées**.
2. Dans le **Centre d’applications managées**, choisissez **Applications du catalogue de services**, puis cliquez sur **Ajouter**. 

    ![Ajouter l’application managée](./media/tutorial-create-managed-app-with-custom-provider/add-managed-application.png)

3. Dans la page **Applications du catalogue de services**, tapez le nom complet de la définition du catalogue de services dans la zone de recherche. Sélectionnez la définition créée à l’étape précédente, puis cliquez sur **Créer**.

    ![Sélectionner Catalogue de services](./media/tutorial-create-managed-app-with-custom-provider/select-service-catalog-definition.png)

4. Fournissez des valeurs pour créer une instance d’application managée à partir de la définition du catalogue de services :

    * Sélectionnez l’**Abonnement**, le **Groupe de ressources** et l’**Emplacement** où l’instance d’application sera créée.
    * Spécifiez un nom de fonction Azure unique et un nom de compte Stockage Azure.

    ![Paramètres de l’application](./media/tutorial-create-managed-app-with-custom-provider/application-settings.png)

5. Quand la validation a réussi, cliquez sur **OK** pour déployer une instance d’une application managée. 
    
    ![Déployer une application managée](./media/tutorial-create-managed-app-with-custom-provider/deploy-managed-application.png)

---

## <a name="custom-actions-and-resources"></a>Actions et ressources personnalisées

Une fois l’instance d’application de catalogue de services déployée, vous avez deux nouveaux groupes de ressources. Le premier groupe de ressources `applicationGroup` contient une instance de l’application managée, tandis que le deuxième groupe de ressources `managedResourceGroup` contient les ressources pour l’application managée, notamment le **fournisseur personnalisé**.

![Groupes de ressources d’application](./media/tutorial-create-managed-app-with-custom-provider/application-resource-groups.png)

Vous pouvez accéder à l’instance d’application managée et effectuer une **action personnalisée** dans la page « Overview », créer une ressource personnalisée **users** dans la page « Users » et exécuter une **action de contexte personnalisée** sur une ressource personnalisée.

* Accédez à la page « Overview » et cliquez sur le bouton « Ping Action » :

![Effectuer une action personnalisée](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-action.png)

* Accédez à la page « Users » et cliquez sur le bouton « Add ». Fournissez des entrées pour la création d’une ressource et envoyez le formulaire :

![Capture d’écran montrant le bouton Add sélectionné dans Users.](./media/tutorial-create-managed-app-with-custom-provider/create-custom-resource.png)

* Accédez à la page « Users », sélectionnez une ressource « users » et cliquez sur « Custom Context Action » :

![Capture d’écran montrant Custom Context Action sélectionné.](./media/tutorial-create-managed-app-with-custom-provider/perform-custom-resource-action.png)

[!INCLUDE [clean-up-section-portal](../../../includes/clean-up-section-portal.md)]

## <a name="looking-for-help"></a>Besoin d’aide

Si vous avez des questions sur Applications managées Azure, n’hésitez pas à vous rendre sur le site [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-managedapps). Avant de la publier, vérifiez si votre réponse a déjà été posée et a déjà reçu une réponse. Ajoutez le mot clé `azure-managedapps` pour obtenir une réponse rapide !

## <a name="next-steps"></a>Étapes suivantes

Pour publier votre application gérée sur la Place de marché Microsoft Azure, consultez [Applications gérées Azure sur la Place de marché](../../marketplace/create-new-azure-apps-offer.md).

Apprenez-en davantage sur les [fournisseurs personnalisés Azure](../custom-providers/overview.md).
