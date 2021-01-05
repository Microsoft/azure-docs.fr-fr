---
title: Mise à l’échelle d’un service cloud Azure dans Windows PowerShell | Microsoft Docs
description: (classique) Découvrez comment utiliser PowerShell pour mettre à l’échelle un rôle web ou un rôle de travail dans Azure.
services: cloud-services
author: mmccrory
ms.service: cloud-services
ms.topic: article
ms.date: 12/01/2016
ms.author: memccror
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c2cec08d86760694ec3c41a98af62c6b86e0a4c9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89071089"
---
# <a name="how-to-scale-a-cloud-service-in-powershell"></a>Mise à l’échelle d’un service cloud dans PowerShell

Vous pouvez utiliser Windows PowerShell à pour mettre à l’échelle un rôle web ou un rôle de travail en ajoutant ou supprimant des instances.  

## <a name="log-in-to-azure"></a>Connexion à Azure

Avant de pouvoir effectuer des opérations sur votre abonnement via PowerShell, vous devez vous connecter :

```powershell
Add-AzureAccount
```

Si vous avez plusieurs abonnements associés à votre compte, vous devrez peut-être modifier l’abonnement actuel en fonction de l’emplacement de votre service cloud. Pour vérifier l’abonnement actuel, exécutez :

```powershell
Get-AzureSubscription -Current
```

Si vous devez modifier l’abonnement en cours, exécutez :

```powershell
Set-AzureSubscription -SubscriptionId <subscription_id>
```

## <a name="check-the-current-instance-count-for-your-role"></a>Vérifiez le nombre d’instances actuel pour votre rôle

Pour vérifier l’état actuel de votre rôle, exécutez :

```powershell
Get-AzureRole -ServiceName '<your_service_name>' -RoleName '<your_role_name>'
```

Vous devriez obtenir des informations sur le rôle, y compris la version actuelle de son système d’exploitation et le nombre d’instances. Dans ce cas, le rôle a une seule instance.

![Informations sur le rôle](./media/cloud-services-how-to-scale-powershell/get-azure-role.png)

## <a name="scale-out-the-role-by-adding-more-instances"></a>Effectuer un scale-out du rôle en ajoutant des instances

Pour effectuer un scale-out de votre rôle, passez le nombre souhaité d’instances en tant que paramètre **Count** paramètre à l’applet de commande **Set-AzureRole** :

```powershell
Set-AzureRole -ServiceName '<your_service_name>' -RoleName '<your_role_name>' -Slot <target_slot> -Count <desired_instances>
```

L’applet de commande se bloque momentanément pendant que les nouvelles instances sont configurées et démarrées. Pendant ce temps, si vous ouvrez une fenêtre PowerShell et appelez **Get-AzureRole** comme présenté précédemment, vous verrez le nouveau nombre d’instances cibles. Et si vous examinez l’état du rôle dans le portail, vous devriez voir la nouvelle instance démarrer :

![Instance de machine virtuelle en cours de démarrage dans le portail](./media/cloud-services-how-to-scale-powershell/role-instance-starting.png)

Une fois les nouvelles instances démarrées, l’applet de commande renvoie correctement :

![Augmentation du nombre d’instances du rôle réussie](./media/cloud-services-how-to-scale-powershell/set-azure-role-success.png)

## <a name="scale-in-the-role-by-removing-instances"></a>Effectuer un scale-in du rôle en supprimant des instances

Vous pouvez effectuer un scale-in d’un rôle en supprimant des instances de la même façon. Définissez le paramètre **Count** paramètre sur **Set-AzureRole** pour le nombre d’instances que vous souhaitez avoir une fois l’opération de scale-in terminée.

## <a name="next-steps"></a>Étapes suivantes

Il n’est pas possible de configurer la mise à l’échelle automatique pour les services cloud à partir de PowerShell. Pour ce faire, consultez [Mise à l’échelle automatique d’un service cloud](cloud-services-how-to-scale-portal.md).
