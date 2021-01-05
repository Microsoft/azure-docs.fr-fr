---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 1f8f8d314a8bb37a08b3696f597b395a8a4beb8e
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "67177287"
---
## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

```powershell
Connect-AzAccount
```

Si vous ne savez pas quelle localisation utiliser, affichez la liste des localisations disponibles. Affichez la liste des localisations avec l’exemple de code suivant et trouvez celui que vous souhaitez utiliser. Cet exemple utilise **eastus**. Stockez la localisation dans une variable et utilisez cette variable pour changer son contenu dans un seul emplacement.

```powershell
Get-AzLocation | select Location
$location = "eastus"
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

```powershell
$resourceGroup = "myResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage universel standard avec réplication LRS à l’aide de l’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Ensuite, obtenez le contexte du compte de stockage qui définit le compte de stockage à utiliser. Si un compte de stockage est utilisé, référencez le contexte au lieu d’entrer les informations d’identification à plusieurs reprises. Utilisez l’exemple suivant pour créer un compte de stockage nommé *mystorageaccount* avec les options de stockage localement redondant (LRS) et de chiffrement des objets blob (activées par défaut).

```powershell
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name "mystorageaccount" `
  -SkuName Standard_LRS `
  -Location $location `

$ctx = $storageAccount.Context
```
