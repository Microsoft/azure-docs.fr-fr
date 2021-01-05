---
title: 'Tutoriel : Utiliser une identité managée pour accéder à Azure Cosmos DB - Windows - Azure AD'
description: Un didacticiel qui vous guide tout au long du processus d’utilisation d’une identité managée et attribuée par le système sur une machine virtuelle Windows en vue d’accéder à Azure Cosmos DB.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc3417284137cdbc9f93ac02f825820bfe744843
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107496"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Tutoriel : Utiliser une identité managée de machine virtuelle Windows attribuée par le système pour accéder à Azure Cosmos DB

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce didacticiel vous indique comment utiliser une identité managée attribuée par le système pour une machine virtuelle Windows afin d’accéder à Cosmos DB. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Création d’un compte Cosmos DB
> * Autoriser l’identité managée attribuée par le système de machine virtuelle Windows à obtenir les clés d’accès du compte Cosmos DB
> * Obtenir un jeton d’accès à l’aide de l’identité managée attribuée par le système de machine virtuelle Windows pour appeler Azure Resource Manager
> * Obtenir les clés d’accès à partir d’Azure Resource Manager pour effectuer des appels Cosmos DB

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](overview.md). 
- Si vous n’avez pas encore de compte Azure, [Inscrivez-vous sur un compte gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour effectuer les opérations de création de ressource et de gestion de rôles nécessaires, votre compte doit bénéficier des autorisations « Propriétaire » avec l’étendue appropriée (votre abonnement ou groupe de ressources). Si vous avez besoin d’aide concernant l’attribution de rôle, consultez [Utiliser le contrôle d’accès en fonction du rôle pour gérer l’accès aux ressources d’un abonnement Azure](../../role-based-access-control/role-assignments-portal.md).
- Installer la dernière version [d’Azure PowerShell](/powershell/azure/install-az-ps)
- Vous avez également besoin d’une machine virtuelle Windows sur laquelle les identités managées attribuées par le système sont activées.
  - Si vous devez créer une machine virtuelle pour ce tutoriel, vous pouvez suivre les instructions de l’article intitulé [Créer une machine virtuelle avec une identité affectée par le système activée](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity)

## <a name="create-a-cosmos-db-account"></a>Création d’un compte Cosmos DB 

Si vous n’en avez pas déjà un, créez un compte Cosmos DB. Vous pouvez ignorer cette étape et utiliser un compte Cosmos DB existant. 

1. Cliquez sur le bouton **+ Créer une ressource** dans le coin supérieur gauche du portail Azure.
2. Cliquez sur **Bases de données**, puis sur **Azure Cosmos DB** pour afficher le panneau Nouveau compte.
3. Entrez un **ID** pour le compte Cosmos DB, que vous pourrez utiliser plus tard.  
4. **API** doit être défini sur « SQL ». L’approche décrite dans ce didacticiel peut être utilisée avec les autres types d’API disponibles. Toutefois, les étapes de ce didacticiel correspondent à celles de l’API SQL.
5. Assurez-vous que les champs **Abonnement** et **Groupe de ressources** correspondent à ceux que vous avez spécifiés lorsque vous avez créé votre machine virtuelle à l’étape précédente.  Sélectionnez un **emplacement** où Cosmos DB est disponible.
6. Cliquez sur **Créer**.

### <a name="create-a-collection"></a>Création d'une collection 

Ensuite, ajoutez une collection de données dans le compte Cosmos DB que vous pourrez interroger dans les prochaines étapes.

1. Accédez au compte Cosmos DB que vous venez de créer.
2. Sous l’onglet **Vue d’ensemble**, cliquez sur le bouton **+ Ajouter une collection** pour afficher le panneau Ajouter une collection.
3. Attribuez à la collection un ID de base de données et un ID de collection. Sélectionnez une capacité de stockage, entrez une clé de partition et une valeur de débit, puis cliquez sur **OK**.  Pour ce didacticiel, il suffit d’utiliser « Test » comme ID de base de données et ID de collection, et de sélectionner une capacité de stockage fixe et le débit le plus bas (400 RU/s).  


## <a name="grant-access"></a>Accorder l'accès

Cette section montre comment autoriser l’identité managée attribuée par le système de machine virtuelle Windows à obtenir les clés d’accès du compte Cosmos DB. Cosmos DB ne prend pas en charge l’authentification Azure AD en mode natif. Toutefois, vous pouvez utiliser une identité managée attribuée par le système pour récupérer une clé d’accès Cosmos DB à partir de Resource Manager, puis utiliser cette clé pour accéder à Cosmos DB. Dans cette étape, vous autoriserez votre identité managée attribuée par le système de machine virtuelle Windows à accéder aux clés pour le compte Cosmos DB.

Pour autoriser votre identité managée attribuée par le système de machine virtuelle Windows à accéder au compte Cosmos DB dans Azure Resource Manager à l’aide de PowerShell, mettez à jour les valeurs suivantes :

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>`

Cosmos DB prend en charge deux niveaux de granularité lors de l’utilisation des clés d’accès : accès en lecture/écriture au compte et accès en lecture seule au compte.  Pour obtenir les clés en lecture/écriture du compte, affectez le rôle `DocumentDB Account Contributor`, et pour obtenir les clés en lecture seule du compte, affectez le rôle `Cosmos DB Account Reader Role`.  Pour ce didacticiel, assignez le rôle `Cosmos DB Account Reader Role` :

```azurepowershell
$spID = (Get-AzVM -ResourceGroupName myRG -Name myVM).identity.principalid
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Cosmos DB Account Reader Role" -Scope "/subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>"
```

>[!NOTE]
> Si vous ne parvenez pas à effectuer une opération, il est possible que vous ne disposiez pas des autorisations appropriées. Si vous souhaitez accéder en écriture à des clés, vous devez utiliser un rôle Azure, tel que Contributeur de compte DocumentDB, ou créer un rôle personnalisé. Pour plus d’informations, consultez [Contrôle d’accès en fonction du rôle Azure dans Azure Cosmos DB](../../cosmos-db/role-based-access-control.md).

## <a name="access-data"></a>Accéder aux données

Cette section montre comment appeler Azure Resource Manager à l’aide d’un jeton d’accès pour obtenir l’identité managée attribuée par le système de machine virtuelle Windows. Pour la suite de ce didacticiel, nous allons utiliser la machine virtuelle que nous avons créée précédemment. 

Vous devez installer la dernière version d’[Azure CLI](/cli/azure/install-azure-cli) sur votre machine virtuelle Windows.

### <a name="get-an-access-token"></a>Obtention d’un jeton d’accès

1. Dans le portail Azure, accédez à **Machines virtuelles**, accédez à votre machine virtuelle Windows, puis, en haut de la page **Vue d’ensemble**, cliquez sur **Se connecter**. 
2. Entrez le **Nom d’utilisateur** et le **Mot de passe** que vous avez ajoutés lorsque vous avez créé la machine virtuelle Windows. 
3. Maintenant que vous avez créé une **Connexion au Bureau à distance** avec la machine virtuelle, ouvrez PowerShell dans la session à distance.
4. À l’aide de Invoke-WebRequest de Powershell, envoyez une requête aux identités managées locales pour le point de terminaison de ressources Azure afin d’obtenir un jeton d’accès pour Azure Resource Manager.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
   ```

   > [!NOTE]
   > La valeur du paramètre « ressource » doit correspondre exactement à ce qui est attendu par Azure AD. Lorsque vous utilisez l’ID de ressource Azure Resource Manager, vous devez inclure la barre oblique de fin à l’URI.
    
   Ensuite, extrayez l’élément « Content » qui est stocké sous forme de chaîne au format JSON (JavaScript Objet Notation) dans l’objet $response. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```
   Ensuite, extrayez le jeton d’accès de la réponse.
    
   ```powershell
   $ArmToken = $content.access_token
   ```

### <a name="get-access-keys"></a>Obtenir les clés d’accès 

Cette section montre comment obtenir les clés d’accès à partir d’Azure Resource Manager pour effectuer des appels Cosmos DB. Nous utilisons PowerShell pour appeler Resource Manager à l’aide du jeton d’accès que nous avons récupéré, afin de récupérer la clé d’accès au compte Cosmos DB. Une fois la clé d’accès obtenue, nous pouvons interroger Cosmos DB. Utilisez vos propres valeurs pour remplacer les entrées ci-dessous :

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>` 
- Remplacez la valeur `<ACCESS TOKEN>` par le jeton d’accès que vous avez récupéré précédemment. 

>[!NOTE]
>Si vous souhaitez récupérer les clés en lecture/écriture, utilisez le type d’opération de clé `listKeys`.  Si vous souhaitez récupérer les clés en lecture seule, utilisez le type d’opération de clé `readonlykeys`. Si vous ne pouvez pas utiliser « listkeys », vérifiez que vous avez affecté le [rôle approprié](../../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role) à l’identité managée.

```powershell
Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>/readonlykeys/?api-version=2016-03-31' -Method POST -Headers @{Authorization="Bearer $ARMToken"}
```
La réponse vous donne la liste des clés.  Par exemple, si vous obtenez des clés en lecture seule :

```powershell
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```
Maintenant que vous disposez de la clé d’accès pour le compte Cosmos DB, vous pouvez la passer à un SDK Cosmos DB et effectuer des appels pour accéder au compte.  Pour un exemple rapide, vous pouvez passer la clé d’accès à Azure CLI.  Vous pouvez obtenir l’`<COSMOS DB CONNECTION URL>` sous l’onglet **Vue d’ensemble**, dans le panneau du compte Cosmos DB, dans le portail Azure.  Remplacez `<ACCESS KEY>` par la valeur que vous avez obtenue ci-dessus :

```azurecli
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

Cette commande CLI retourne des informations détaillées sur la collection :

```output
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser une identité managée attribuée par le système de machine virtuelle Windows pour accéder à Cosmos DB.  Pour en savoir plus sur Cosmos DB, consultez :

> [!div class="nextstepaction"]
>[Présentation d’Azure Cosmos DB](../../cosmos-db/introduction.md)