---
title: Mettre à jour Media Services après la substitution de clés d’accès de stockage | Microsoft Docs
description: Cet article vous donne des conseils sur la mise à jour de Media Services après la substitution de clés d’accès de stockage.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a892ebb0-0ea0-4fc8-b715-60347cc5c95b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: milanga;cenkdin
ms.openlocfilehash: c68bedb37722fb6a8b7ad9dccdeaaaa4fab9d020
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264128"
---
# <a name="update-media-services-after-rolling-storage-access-keys"></a>Mettre à jour Media Services après la substitution de clés d’accès de stockage

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Quand vous créez un compte Azure Media Services (AMS), vous êtes également invité à sélectionner un compte de stockage Azure qui est utilisé pour stocker votre contenu multimédia. Vous pouvez ajouter plusieurs comptes de stockage à votre compte Media Services. Cet article explique comment orchestrer la rotation des clés de stockage. Elle montre également comment ajouter des comptes de stockage à un compte multimédia. 

Pour effectuer les actions décrites dans cet article, vous devez utiliser des [API Azure Resource Manager](/rest/api/media/operations/azure-media-services-rest-api-reference) et [Powershell](/powershell/module/az.media).  Pour plus d’informations, consultez [Guide pratique pour gérer les ressources Azure avec PowerShell et Resource Manager](../../azure-resource-manager/management/manage-resource-groups-powershell.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble

Lors de la création d’un compte de stockage, Azure génère deux clés d’accès de stockage 512 bits, qui sont utilisées pour authentifier l’accès à votre compte de stockage. Pour sécuriser vos connexions de stockage, il est recommandé de régénérer et d’alterner périodiquement vos clés d’accès de stockage. Vous bénéficiez de deux clés d’accès (primaire et secondaire), ce qui vous permet de conserver vos connexions au compte de stockage à l’aide d’une clé d’accès lorsque vous régénérez l’autre clé. Cette procédure est également appelée « substitution des clés d’accès ».

Media Services dépend d'une clé de stockage qui lui est fournie. Plus précisément, les localisateurs utilisés pour diffuser en continu ou télécharger vos ressources dépendent de la clé d’accès de stockage spécifiée. Lors de la création d’un compte AMS, Media Services choisit une dépendance sur la clé d’accès de stockage principale par défaut mais, en tant qu’utilisateur, vous pouvez mettre à jour la clé de stockage d’AMS. Vous devez veiller à indiquer à Media Services la clé à utiliser en suivant les étapes décrites dans cet article.  

>[!NOTE]
> Si vous possédez plusieurs comptes de stockage, vous devez effectuer cette procédure pour chacun d’eux. L’ordre de régénération des clés de stockage n’est pas fixe. Vous pouvez commencer par la clé primaire ou la clé secondaire.
>
> Avant de suivre les étapes décrites dans cet article sur un compte de production, veillez à les tester sur un compte de pré-production.
>

## <a name="steps-to-rotate-storage-keys"></a>Étapes de régénération des clés de stockage 
 
 1. Modifiez la clé primaire du compte de stockage par le biais de l’applet de commande Powershell ou du portail [Azure](https://portal.azure.com/).
 2. Appelez la cmdlet Sync-AzMediaServiceStorageKeys avec les paramètres appropriés pour forcer le compte multimédia à utiliser les clés de compte de stockage.
 
    L’exemple suivant montre comment synchroniser des clés avec des comptes de stockage.
  
    `Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId`
  
 3. Attendez environ une heure. Vérifiez que les scénarios de streaming fonctionnent.
 4. Modifiez la clé secondaire du compte de stockage par le biais de l’applet de commande Powershell ou du portail Azure.
 5. Appelez la cmdlet Sync-AzMediaServiceStorageKeys avec les paramètres appropriés pour forcer le compte multimédia à utiliser de nouvelles clés de compte de stockage. 
 6. Attendez environ une heure. Vérifiez que les scénarios de streaming fonctionnent.
 
### <a name="a-powershell-cmdlet-example"></a>Exemple d’applet de commande PowerShell 

L’exemple suivant montre comment obtenir le compte de stockage et le synchroniser avec le compte AMS.

```console
$regionName = "West US"
$resourceGroupName = "SkyMedia-USWest-App"
$mediaAccountName = "sky"
$storageAccountName = "skystorage"
$storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId
```
 
## <a name="steps-to-add-storage-accounts-to-your-ams-account"></a>Étapes à suivre pour ajouter des comptes de stockage à votre compte AMS

L’article suivante montre comment ajouter des comptes de stockage à votre compte AMS : [Associez plusieurs comptes de stockage à un compte Media Services](./media-services-managing-multiple-storage-accounts.md).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>Remerciements
Nous aimerions remercier les personnes suivantes qui ont contribué à la création de ce document : Cenk Dingiloglu, Milan Gada, Seva Titov.
