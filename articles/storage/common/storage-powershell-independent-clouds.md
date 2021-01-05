---
title: Utiliser PowerShell pour gérer les données dans les clouds indépendants Azure
titleSuffix: Azure Storage
description: Gestion du stockage dans le cloud chinois, le cloud Government et le cloud allemand avec Azure PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e924a5f6c765b5b964fe3b1492393b063d9d23b4
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783570"
---
# <a name="managing-storage-in-the-azure-independent-clouds-using-powershell"></a>Gestion du stockage dans les clouds indépendants Azure avec PowerShell

La plupart des gens utilisent le cloud public Azure pour leur déploiement Azure global. Pour des raisons de souveraineté, entre autres, il existe également des déploiements indépendants de Microsoft Azure. Ces déploiements indépendants sont appelés « environnements ». La liste suivante présente en détail les clouds indépendants actuellement disponibles.

* [Azure Government Cloud](https://azure.microsoft.com/features/gov/)
* [Cloud Azure China 21Vianet géré par 21Vianet en Chine](http://www.windowsazure.cn/)
* [Cloud Azure allemand](../../germany/germany-welcome.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="using-an-independent-cloud"></a>Utilisation d’un cloud indépendant

Pour utiliser le stockage Azure dans l’un des clouds indépendants, vous vous connectez à ce cloud au lieu du cloud Azure public. Pour utiliser l’un des clouds indépendants plutôt que le cloud Azure public :

* Spécifiez *l’environnement* auquel se connecter.
* Déterminez et utilisez les régions disponibles.
* Utilisez le suffixe de point de terminaison approprié, qui est différent de celui du cloud Azure public.

Ces exemples nécessitent le module Az Azure PowerShell version 0.7 ou ultérieure. Dans une fenêtre PowerShell, exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si aucune information ne s’affiche, ou si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="log-in-to-azure"></a>Connexion à Azure

Exécutez l’applet de commande [Get-AzEnvironment](/powershell/module/az.accounts/get-azenvironment) pour voir les environnements Azure disponibles :

```powershell
Get-AzEnvironment
```

Connectez-vous à votre compte qui a accès au cloud auquel vous souhaitez vous connecter, puis définissez l’environnement. Cet exemple montre comment se connecter à un compte qui utilise le cloud Azure Government.   

```powershell
Connect-AzAccount –Environment AzureUSGovernment
```

Pour accéder au cloud de la Chine, utilisez l’environnement **AzureChinaCloud** . Pour accéder au cloud allemand, utilisez **AzureGermanCloud** .

À ce stade, si vous avez besoin de la liste des emplacements pour créer un compte de stockage ou une autre ressource, vous pouvez interroger les emplacements disponibles pour le cloud sélectionné à l’aide de [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

```powershell
Get-AzLocation | select Location, DisplayName
```

Le tableau suivant présente les emplacements retournés pour le cloud allemand.

|Emplacement | Nom d’affichage |
|----|----|
| `germanycentral` | Centre de l’Allemagne|
| `germanynortheast` | Nord-Est de l’Allemagne |


## <a name="endpoint-suffix"></a>Suffixe de point de terminaison

Le suffixe de point de terminaison pour chacun de ces environnements est différent de celui du point de terminaison du cloud Azure public. Par exemple, le suffixe de point de terminaison blob du cloud Azure public est **blob.core.windows.net** . Pour le cloud Government, le suffixe de point de terminaison blob est **blob.core.usgovcloudapi.net** .

### <a name="get-endpoint-using-get-azenvironment"></a>Obtenir le point de terminaison avec Get-AzEnvironment

Récupérez le suffixe de point de terminaison à l’aide de [Get-AzEnvironment](/powershell/module/az.accounts/get-azenvironment). Le point de terminaison est la propriété *StorageEndpointSuffix* de l’environnement.

Les extraits de code suivants montrent comment récupérer le suffixe du point de terminaison. Toutes ces commandes retournent des valeurs similaires à « core.cloudapp.net» ou « core.cloudapi.de», etc. Ajoutez le suffixe au service de stockage pour accéder à ce dernier. Par exemple, « queue.core.cloudapi.de » accède au service de file d’attente du cloud allemand.

L’extrait de code suivant récupère tous les environnements et le suffixe de point de terminaison pour chacun d’eux.

```powershell
Get-AzEnvironment | select Name, StorageEndpointSuffix 
```

Cette commande retourne les résultats suivants.

| Nom| StorageEndpointSuffix|
|----|----|
| AzureChinaCloud | core.chinacloudapi.cn|
| AzureCloud | core.windows.net |
| AzureGermanCloud | core.cloudapi.de|
| AzureUSGovernment | core.usgovcloudapi.net |

Pour récupérer toutes les propriétés de l’environnement spécifié, appelez **Get-AzEnvironment** , puis spécifiez le nom du cloud. L’extrait de code suivant retourne une liste de propriétés. Recherchez **StorageEndpointSuffix** dans cette liste. L’exemple suivant concerne le cloud allemand.

```powershell
Get-AzEnvironment -Name AzureGermanCloud
```

Les résultats sont semblables aux valeurs suivantes :

|Nom de la propriété|Valeur|
|----|----|
| Nom | `AzureGermanCloud` |
| EnableAdfsAuthentication | `False` |
| ActiveDirectoryServiceEndpointResourceI | `http://management.core.cloudapi.de/` |
| GalleryURL | `https://gallery.cloudapi.de/` |
| ManagementPortalUrl | `https://portal.microsoftazure.de/` |
| ServiceManagementUrl | `https://manage.core.cloudapi.de/` |
| PublishSettingsFileUrl| `https://manage.microsoftazure.de/publishsettings/index` |
| ResourceManagerUrl | `http://management.microsoftazure.de/` |
| SqlDatabaseDnsSuffix | `.database.cloudapi.de` |
| **StorageEndpointSuffix** | `core.cloudapi.de` |
| ... | ... |

Pour récupérer uniquement la propriété de suffixe de point de terminaison de stockage, récupérez le cloud spécifique, puis demandez juste cette propriété.

```powershell
$environment = Get-AzEnvironment -Name AzureGermanCloud
Write-Host "Storage EndPoint Suffix = " $environment.StorageEndpointSuffix
```

Cette commande renvoie les informations suivantes :

`Storage Endpoint Suffix = core.cloudapi.de`

### <a name="get-endpoint-from-a-storage-account"></a>Obtenir le point de terminaison à partir d’un compte de stockage

Vous pouvez également consulter les propriétés d’un compte de stockage pour récupérer les points de terminaison :

```powershell
# Get a reference to the storage account.
$resourceGroup = "myexistingresourcegroup"
$storageAccountName = "myexistingstorageaccount"
$storageAccount = Get-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageAccountName 
  # Output the endpoints.
Write-Host "blob endpoint = " $storageAccount.PrimaryEndPoints.Blob 
Write-Host "file endpoint = " $storageAccount.PrimaryEndPoints.File
Write-Host "queue endpoint = " $storageAccount.PrimaryEndPoints.Queue
Write-Host "table endpoint = " $storageAccount.PrimaryEndPoints.Table
```

Pour un compte de stockage dans le cloud Government, cette commande retourne la sortie suivante :

```
blob endpoint = http://myexistingstorageaccount.blob.core.usgovcloudapi.net/
file endpoint = http://myexistingstorageaccount.file.core.usgovcloudapi.net/
queue endpoint = http://myexistingstorageaccount.queue.core.usgovcloudapi.net/
table endpoint = http://myexistingstorageaccount.table.core.usgovcloudapi.net/
```

## <a name="after-setting-the-environment"></a>Après la configuration de l’environnement

Vous pouvez désormais utiliser PowerShell pour gérer vos comptes de stockage et accéder aux données d’objet BLOB, de file d’attente, de fichier et de table. Pour plus d’informations, consultez [Az.Storage](/powershell/module/az.storage).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez créé un groupe de ressources et un compte de stockage pour cet exercice, vous pouvez supprimer les deux ressources en supprimant le groupe de ressources. La suppression du groupe de ressources supprime toutes les ressources qu’il contient.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

* [Persistance des informations de connexion utilisateur d’une session PowerShell à l’autre](/powershell/azure/context-persistence)
* [Stockage Azure Government](../../azure-government/compare-azure-government-global-azure.md)
* [Guide du développeur Microsoft Azure Government](../../azure-government/documentation-government-developer-guide.md)
* [Notes pour les développeurs relatives aux applications Azure China 21Vianet](https://msdn.microsoft.com/library/azure/dn578439.aspx)
* [Documentation Azure Allemagne](../../germany/germany-welcome.md)