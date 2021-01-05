---
title: Activer la redondance de zone pour Azure Cache pour Redis (préversion)
description: Découvrez comment configurer la redondance de zone pour vos instances Azure Cache pour Redis de niveau Premium et Entreprise.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: 0cb7ee5b9fa02e726d03bf1ae9935c07ded6e4a6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93088010"
---
# <a name="enable-zone-redundancy-for-azure-cache-for-redis-preview"></a>Activer la redondance de zone pour Azure Cache pour Redis (préversion)
Dans cet article, vous allez apprendre à configurer une instance Azure Cache redondante interzone à l’aide du portail Azure.

Les niveaux Standard, Premium et Entreprise d’Azure Cache pour Redis offrent une redondance intégrée en hébergeant chaque cache sur deux machines virtuelles dédiées. Même si ces machines virtuelles se trouvent dans des [domaines d’erreur et de mise à jour Azure](../virtual-machines/manage-availability.md) distincts et hautement disponibles, elles sont sensibles aux défaillances au niveau du centre de données. Azure Cache pour Redis prend également en charge la redondance de zone dans ses niveaux Premium et Entreprise. Un cache redondant interzone s’exécute sur des machines virtuelles distribuées dans plusieurs [zones de disponibilité](../virtual-machines/manage-availability.md#use-availability-zones-to-protect-from-datacenter-level-failures). Il offre une plus grande résilience et disponibilité.

## <a name="prerequisites"></a>Prérequis
* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)

> [!NOTE]
> Cette fonctionnalité est actuellement disponible en préversion - [contactez-nous](mailto:azurecache@microsoft.com) si vous êtes intéressé.
>

## <a name="create-a-cache"></a>Création d'un cache
Pour créer un cache, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Créer une ressource**.
  
1. Dans la page **Nouvelle** , sélectionnez **Bases de données** , puis **Azure Cache pour Redis**.

    :::image type="content" source="media/cache-create/new-cache-menu.png" alt-text="Sélectionnez Azure Cache pour Redis.":::
   
1. Dans la page **Général** , configurez les paramètres du nouveau cache.
   
    | Paramètre      | Valeur suggérée  | Description |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Abonnement** | Sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
    | **Groupe de ressources** | Sélectionnez un groupe de ressources ou choisissez **Créer nouveau** , puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
    | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net*. | 
    | **Lieu** | Sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
    | **Type de cache** | Sélectionnez un cache de [niveau Premium ou Entreprise](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |
   
1. Sur la page **Avancé** , pour un cache de niveau Premium, choisissez **Nombre de réplicas**.
   
    :::image type="content" source="media/cache-how-to-multi-replicas/create-multi-replicas.png" alt-text="Nombre de réplicas":::

1. Sélectionnez **Zones de disponibilité**. 
   
    :::image type="content" source="media/cache-how-to-zone-redundancy/create-zones.png" alt-text="Zones de disponibilité":::

1. Conservez les paramètres par défaut pour les autres options. 

    > [!NOTE]
    > La prise en charge de la redondance de zone fonctionne uniquement avec les caches qui ne sont pas regroupés en cluster ni géorépliqués. En outre, elle ne prend pas en charge les liaisons privées, la mise à l’échelle, la persistance des données ou l’importation/exportation.
    >

1. Cliquez sur **Créer**. 
   
    La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution** , le cache est prêt pour utilisation.
   
    > [!NOTE]
    > Les zones de disponibilité ne peuvent pas être modifiées après la création du cache.
    >

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

> [!div class="nextstepaction"]
> [Niveaux de service Premium Azure Cache pour Redis](cache-overview.md#service-tiers)