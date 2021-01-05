---
title: Publier une application managée du catalogue de services
description: Montre comment créer une application managée Azure destinée aux membres de votre organisation.
author: tfitzmac
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.date: 04/14/2020
ms.author: tomfitz
ms.openlocfilehash: 342fa722d704933f22cec00a46d11ccc38fc6e4d
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91650644"
---
# <a name="quickstart-create-and-publish-a-managed-application-definition"></a>Démarrage rapide : Créer et publier une définition d’application gérée

Ce guide de démarrage rapide offre une présentation de l’utilisation des [applications managées Azure](overview.md). Vous pouvez créer et publier une application managée destinée aux membres de votre organisation.

Pour publier une application managée sur le catalogue de services, vous devez :

* Créer un modèle qui définit les ressources à déployer avec l’application managée.
* Définir les éléments d’interface utilisateur du portail lors du déploiement de l’application managée.
* Créer un package _.zip_ qui contient les fichiers de modèles nécessaires.
* Désigner l’utilisateur, le groupe ou l’application qui doit avoir accès au groupe de ressources dans l’abonnement de l’utilisateur.
* Créer la définition de l’application managée qui pointe vers le package _.zip_ et qui demande l’accès pour l’identité.

## <a name="create-the-arm-template"></a>Créer le modèle ARM

Chaque définition d’application managée contient un fichier nommé _mainTemplate.json_. Vous y définissez les ressources Azure à déployer. Le modèle n’est en rien différent d’un modèle ARM normal.

Créez un fichier nommé _mainTemplate.json_. Cette valeur respecte la casse.

Ajoutez le code JSON suivant à votre fichier. Il définit les paramètres pour la création d’un compte de stockage et spécifie les propriétés du compte de stockage.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountNamePrefix": {
      "type": "string"
    },
    "storageAccountType": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "storageAccountName": "[concat(parameters('storageAccountNamePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
    }
  }
}
```

Enregistrez le fichier _mainTemplate.json_.

## <a name="define-your-create-experience"></a>Définir votre expérience de création

En tant qu’éditeur, vous définissez l’expérience du portail pour la création de l’application managée. Le fichier _createUiDefinition.json_ génère l’interface du portail. Vous définissez la manière dont les utilisateurs fournissent une entrée pour chaque paramètre à l’aide d’[éléments de contrôle](create-uidefinition-elements.md), notamment les listes déroulantes, les zones de texte et les zones de mot de passe.

Créez un fichier _createUiDefinition.json_ (ce nom doit respecter la casse).

Ajoutez le code de démarrage JSON suivant au fichier et enregistrez-le.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "storageConfig",
        "label": "Storage settings",
        "subLabel": {
          "preValidation": "Configure the infrastructure settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Storage settings",
        "elements": [
          {
            "name": "storageAccounts",
            "type": "Microsoft.Storage.MultiStorageAccountCombo",
            "label": {
              "prefix": "Storage account name prefix",
              "type": "Storage account type"
            },
            "defaultValue": {
              "type": "Standard_LRS"
            },
            "constraints": {
              "allowedTypes": [
                "Premium_LRS",
                "Standard_LRS",
                "Standard_GRS"
              ]
            }
          }
        ]
      }
    ],
    "outputs": {
      "storageAccountNamePrefix": "[steps('storageConfig').storageAccounts.prefix]",
      "storageAccountType": "[steps('storageConfig').storageAccounts.type]",
      "location": "[location()]"
    }
  }
}
```

Pour en savoir plus, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).

## <a name="package-the-files"></a>Empaqueter les fichiers

Ajoutez les deux fichiers à un fichier _.zip_ nommé _app.zip_. Les deux fichiers doivent se trouver au niveau racine du fichier _.zip_. Si vous les placez dans un dossier, vous obtenez une erreur au moment de créer la définition de l’application managée qui indique que les fichiers nécessaires ne sont pas présents.

Chargez le package à un emplacement accessible à partir de là où il peut être utilisé. Vous devez fournir un nom unique pour le compte de stockage.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name storageGroup -Location eastus

$storageAccount = New-AzStorageAccount `
  -ResourceGroupName storageGroup `
  -Name "mystorageaccount" `
  -Location eastus `
  -SkuName Standard_LRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob

Set-AzStorageBlobContent `
  -File "D:\myapplications\app.zip" `
  -Container appcontainer `
  -Blob "app.zip" `
  -Context $ctx
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name storageGroup --location eastus

az storage account create \
    --name mystorageaccount \
    --resource-group storageGroup \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2

az storage container create \
    --account-name mystorageaccount \
    --name appcontainer \
    --public-access blob

az storage blob upload \
    --account-name mystorageaccount \
    --container-name appcontainer \
    --name "app.zip" \
    --file "D:\myapplications\app.zip"

```

---

## <a name="create-the-managed-application-definition"></a>Créer la définition d’application gérée

### <a name="create-an-azure-active-directory-user-group-or-application"></a>Créer un groupe d’utilisateurs ou une application Azure Active Directory

L’étape suivante consiste à sélectionner un groupe d’utilisateurs, un utilisateur ou une application afin de gérer les ressources pour le client. Cette identité dispose d’autorisations sur le groupe de ressources managé en fonction du rôle attribué. Le rôle peut être n’importe quel rôle intégré Azure comme Propriétaire ou Contributeur. Pour créer un groupe d’utilisateurs Active Directory, consultez [Créer un groupe et ajouter des membres dans Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Vous avez besoin de l’ID d’objet du groupe d’utilisateurs à utiliser pour gérer les ressources.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$groupID=(Get-AzADGroup -DisplayName mygroup).Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
groupid=$(az ad group show --group mygroup --query objectId --output tsv)
```

---

### <a name="get-the-role-definition-id"></a>Obtenir l’ID de définition de rôle

Ensuite, vous avez besoin de l’ID de définition de rôle du rôle Azure intégré auquel vous souhaitez accorder l’accès à l’utilisateur, au groupe d’utilisateurs ou à l’application. En règle générale, vous utilisez le rôle Propriétaire, Collaborateur ou Lecteur. La commande suivante montre comment obtenir l’ID de définition de rôle pour le rôle Propriétaire :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$ownerID=(Get-AzRoleDefinition -Name Owner).Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
ownerid=$(az role definition list --name Owner --query [].name --output tsv)
```

---

### <a name="create-the-managed-application-definition"></a>Créer la définition d’application gérée

Si vous ne disposez pas déjà d’un groupe de ressources pour stocker la définition de votre application managée, créez-en un maintenant :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name appDefinitionGroup -Location westcentralus
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name appDefinitionGroup --location westcentralus
```

---

À présent, créez la ressource de définition de l’application managée.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$blob = Get-AzStorageBlob -Container appcontainer -Blob app.zip -Context $ctx

New-AzManagedApplicationDefinition `
  -Name "ManagedStorage" `
  -Location "westcentralus" `
  -ResourceGroupName appDefinitionGroup `
  -LockLevel ReadOnly `
  -DisplayName "Managed Storage Account" `
  -Description "Managed Azure Storage Account" `
  -Authorization "${groupID}:$ownerID" `
  -PackageFileUri $blob.ICloudBlob.StorageUri.PrimaryUri.AbsoluteUri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
blob=$(az storage blob url --account-name mystorageaccount --container-name appcontainer --name app.zip --output tsv)

az managedapp definition create \
  --name "ManagedStorage" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Storage Account" \
  --description "Managed Azure Storage Account" \
  --authorizations "$groupid:$ownerid" \
  --package-file-uri "$blob"
```

---

Une fois la commande terminée, vous avez une définition de l’application managée dans votre groupe de ressources.

Certains des paramètres utilisés dans cet exemple sont les suivants :

* **Groupe de ressources** : nom du groupe de ressources dans lequel la définition de l’application managée est créée.
* **niveau de verrou** : type de verrou placé sur le groupe de ressources managées. Il empêche le client d’effectuer des opérations indésirables sur ce groupe de ressources. Actuellement, ReadOnly est le seul niveau de verrou pris en charge. Lorsque ReadOnly est spécifié, le client peut lire uniquement les ressources présentes dans le groupe de ressources gérées. Les identités de l’éditeur qui ont accès au groupe de ressources managé sont exemptées du verrou.
* **authorizations** : décrit l’ID principal et l’ID de définition de rôle utilisés pour accorder des autorisations au groupe de ressources managées. Il est spécifié sous la forme `<principalId>:<roleDefinitionId>`. Si plusieurs valeurs sont nécessaires, spécifiez-les sous la forme `<principalId1>:<roleDefinitionId1>,<principalId2>:<roleDefinitionId2>`. Les valeurs sont séparées par une virgule.
* **URI du fichier de package** : emplacement d’un package _.zip_ contenant les fichiers nécessaires.

## <a name="bring-your-own-storage-for-the-managed-application-definition"></a>Apporter votre propre stockage pour la définition de l’application managée

Vous pouvez choisir de stocker la définition de votre application managée dans un compte de stockage que vous avez indiqué lors de la création pour pouvoir gérer entièrement son emplacement et son accès conformément à vos obligations réglementaires.

> [!NOTE]
> L’apport de votre propre stockage est uniquement pris en charge avec les déploiements de modèles ARM ou d’API REST de la définition de l’application managée.

### <a name="select-your-storage-account"></a>Sélectionner votre compte de stockage

Vous devez [créer un compte de stockage](../../storage/common/storage-account-create.md) pour contenir votre définition de l’application managée à utiliser avec le catalogue de services.

Copiez l’ID de ressource du compte de stockage. Il sera utilisé plus tard lors du déploiement de la définition.

### <a name="set-the-role-assignment-for-appliance-resource-provider-in-your-storage-account"></a>Définir l’attribution de rôle pour « Fournisseur de ressources d’appliance » dans votre compte de stockage

Pour que votre définition de l’application managée puisse être déployée sur votre compte de stockage, vous devez accorder des autorisations de contributeur au rôle **Fournisseur de ressources d’appliance** afin qu’il puisse écrire les fichiers de définition dans le conteneur de votre compte de stockage.

1. Dans le [Portail Azure](https://portal.azure.com), accédez à votre compte de stockage.
1. Sélectionnez **Contrôle d’accès (IAM)** pour afficher les paramètres de contrôle d’accès du compte de stockage. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles.
1. Dans la fenêtre **Ajouter une attribution de rôle**, sélectionnez le rôle **Contributeur**.
1. Dans le champ **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou principal du service Azure AD**.
1. Sous **Sélectionner**, recherchez le rôle **Fournisseur de ressources d’appliance**, puis sélectionnez-le.
1. Enregistrez l’attribution de rôle.

### <a name="deploy-the-managed-application-definition-with-an-arm-template"></a>Déployer la définition de l’application managée avec un modèle ARM

Utilisez le modèle ARM suivant pour déployer votre application managée packagée en tant que nouvelle définition de l’application managée dans le catalogue de services dont les fichiers de définition sont stockés et gérés dans votre propre compte de stockage :

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "applicationName": {
      "type": "string",
      "metadata": {
        "description": "Managed Application name"
      }
    },
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "definitionStorageResourceID": {
      "type": "string",
      "metadata": {
        "description": "Storage account resource ID for where you're storing your definition"
      }
    },
    "_artifactsLocation": {
      "type": "string",
      "metadata": {
        "description": "The base URI where artifacts required by this template are located."
      }
    }
  },
  "variables": {
    "lockLevel": "None",
    "description": "Sample Managed application definition",
    "displayName": "Sample Managed application definition",
    "managedApplicationDefinitionName": "[parameters('applicationName')]",
    "packageFileUri": "[parameters('_artifactsLocation')]",
    "defLocation": "[parameters('definitionStorageResourceID')]",
    "managedResourceGroupId": "[concat(subscription().id,'/resourceGroups/', concat(parameters('applicationName'),'_managed'))]",
    "applicationDefinitionResourceId": "[resourceId('Microsoft.Solutions/applicationDefinitions',variables('managedApplicationDefinitionName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Solutions/applicationDefinitions",
      "apiVersion": "2020-08-21-preview",
      "name": "[variables('managedApplicationDefinitionName')]",
      "location": "[parameters('location')]",
      "properties": {
        "lockLevel": "[variables('lockLevel')]",
        "description": "[variables('description')]",
        "displayName": "[variables('displayName')]",
        "packageFileUri": "[variables('packageFileUri')]",
        "storageAccountId": "[variables('defLocation')]"
      }
    }
  ],
  "outputs": {}
}
```

Nous avons ajouté une nouvelle propriété nommée `storageAccountId` aux propriétés de votre `applicationDefinitions` et fourni l’ID du compte de stockage dans lequel vous voulez stocker votre définition comme valeur :

Vous pouvez vérifier que les fichiers de définition d’application sont enregistrés dans votre compte de stockage fourni dans un conteneur nommé `applicationDefinitions`.

> [!NOTE]
> Pour renforcer la sécurité, vous pouvez créer une définition de l’application managée pour la stocker dans un [objet blob de compte de stockage Azure où le chiffrement est activé](../../storage/common/storage-service-encryption.md). Le contenu de la définition est chiffré selon les options de chiffrement du compte de stockage. Seuls les utilisateurs disposant d’autorisations d’accès au fichier peuvent voir la définition dans le catalogue de services.

## <a name="make-sure-users-can-see-your-definition"></a>Assurez-vous que les utilisateurs peuvent voir votre définition.

Vous avez accès à la définition de l’application managée, mais vous souhaitez vous assurer que d’autres utilisateurs de votre organisation peuvent y accéder. Accordez-leur au moins le rôle Lecteur dans la définition. Ils ont peut-être hérité de ce niveau d’accès suite à l’abonnement ou au groupe de ressources. Pour vérifier qui a accès à la définition et ajouter des utilisateurs ou des groupes, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide du Portail Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Vous avez publié la définition d’application managée. Découvrez maintenant comment déployer une instance de cette définition.

> [!div class="nextstepaction"]
> [Démarrage rapide : Déployer une application de catalogue de services](deploy-service-catalog-quickstart.md)
