---
title: Informations relatives à la suppression de la famille 1 des systèmes d’exploitation invités | Microsoft Docs
description: Fournit des informations sur la suppression de la famille 1 des SE invités d'Azure et sur la façon de déterminer si vous êtes concerné
services: cloud-services
documentationcenter: na
author: raiye
manager: timlt
ms.service: cloud-services
ms.topic: article
ms.date: 5/21/2017
ms.author: raiye
ms.openlocfilehash: 6de03969a22c5b699fdfd2be17a6ba65905d23e6
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92073391"
---
# <a name="guest-os-family-1-retirement-notice"></a>Informations relatives à la suppression de la famille 1 des systèmes d’exploitation invités
La suppression de la famille 1 des systèmes d'exploitation a été annoncée le 1er juin 2013.

**2 septembre 2014** La famille 1.x des systèmes d'exploitation invités (SE invités) d'Azure, qui est basée sur le système d'exploitation Windows Server 2008, a été officiellement supprimée. Toutes les tentatives de déploiement de nouveaux services ou de mise à niveau des services existants à l'aide de la famille 1 échoueront, et un message d'erreur vous informera que la famille 1 des systèmes d'exploitation invités a été supprimée.

**3 novembre 2014** La prise en charge étendue de la famille 1 des SE invités a pris fin et a été complètement supprimée. Tous les services toujours liés à la famille 1 seront affectés. Nous pouvons arrêter ces services à tout moment. Il n'existe aucune garantie que vos services continueront de s'exécuter, sauf si vous les mettez à niveau manuellement vous-même.

Si vous avez d’autres questions, visitez la [page de questions Microsoft Q&R sur les services cloud](/answers/topics/azure-cloud-services.html) ou [Contactez le support 	Azure](https://azure.microsoft.com/support/options/).

## <a name="are-you-affected"></a>Êtes-vous concerné ?
Vos services cloud sont concernés si l'une des conditions suivantes s'applique :

1. Vous avez une valeur de « osFamily = "1" » explicitement spécifiée dans le fichier ServiceConfiguration.cscfg pour votre service cloud.
2. Vous n'avez pas de valeur pour osFamily explicitement spécifiée dans le fichier ServiceConfiguration.cscfg pour votre service cloud. Actuellement, le système utilise la valeur par défaut « 1 » dans ce cas.
3. Le portail Azure répertorie votre valeur de famille des systèmes d’exploitation invités en tant que « Windows Server 2008 ».

Pour connaître la famille de systèmes d’exploitation exécutée par les services cloud, vous pouvez exécuter le script suivant dans Azure PowerShell. Vous devrez toutefois commencer par [configurer Azure PowerShell](/powershell/azure/). Pour plus d’informations sur le script, consultez [Fin de vie de la famille 1 des SE invités d’Azure : juin 2014](/archive/blogs/ryberry/azure-guest-os-family-1-end-of-life-june-2014).

```Powershell
foreach($subscription in Get-AzureSubscription) {
    Select-AzureSubscription -SubscriptionName $subscription.SubscriptionName

    $deployments=get-azureService | get-azureDeployment -ErrorAction Ignore | where {$_.SdkVersion -NE ""}

    $deployments | ft @{Name="SubscriptionName";Expression={$subscription.SubscriptionName}}, ServiceName, SdkVersion, Slot, @{Name="osFamily";Expression={(select-xml -content $_.configuration -xpath "/ns:ServiceConfiguration/@osFamily" -namespace $namespace).node.value }}, osVersion, Status, URL
}
```

Vos services cloud seront affectés par la suppression de la famille 1 si la colonne osFamily de la sortie du script est vide ou contient « 1 ».

## <a name="recommendations-if-you-are-affected"></a>Recommandations si vous êtes concerné
Nous vous recommandons de migrer vos rôles de service cloud vers l'une des familles de SE invités prises en charge :

**Famille 4.x du SE invité** - Windows Server 2012 R2 *(recommandé)*

1. Assurez-vous que votre application utilise SDK 2.1 ou une version ultérieure avec .NET framework 4.0, 4.5 ou 4.5.1.
2. Définissez l'attribut osFamily sur « 4 » dans le fichier ServiceConfiguration.cscfg et redéployez votre service cloud.

**Famille 3.x du SE invité** - Windows Server 2012

1. Assurez-vous que votre application utilise SDK 1.8 ou une version ultérieure avec .NET framework 4.0 ou 4.5.
2. Définissez l'attribut osFamily sur « 3 » dans le fichier ServiceConfiguration.cscfg et redéployez votre service cloud.

**Famille 2.x du SE invité** - Windows Server 2008 R2

1. Assurez-vous que votre application utilise SDK 1.3 ou une version ultérieure avec .NET framework 3.5 ou 4.0.
2. Définissez l'attribut osFamily sur « 2 » dans le fichier ServiceConfiguration.cscfg et redéployez votre service cloud.

## <a name="extended-support-for-guest-os-family-1-ended-nov-3-2014"></a>Fin de la prise en charge étendue pour la famille 1 des SE invités depuis le 3 novembre 2014
Les services cloud de la famille 1 des SE invités ne sont plus pris en charge. Quittez la famille 1 dès que possible pour éviter toute interruption de service.  

## <a name="next-steps"></a>Étapes suivantes
Consultez les dernières [versions du système d’exploitation invité](cloud-services-guestos-update-matrix.md).