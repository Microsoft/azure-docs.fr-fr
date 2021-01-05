---
title: 'Démarrage rapide : Créer un cache de niveau Entreprise'
description: Dans ce guide de démarrage rapide, vous allez apprendre à créer une instance Azure Cache pour Redis de niveau Entreprise
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: 95f6e2cf7803ff0b152f33c08f170725b5a0e94e
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95795662"
---
# <a name="quickstart-create-an-enterprise-tier-cache-preview"></a>Démarrage rapide : Créer un cache de niveau Entreprise (préversion)

Les niveaux Enterprise d’Azure Cache pour Redis offrent des instances [Redis Enterprise](https://redislabs.com/redis-enterprise/) entièrement intégrées et gérées. Ils sont actuellement disponibles en préversion. Il existe deux nouveaux niveaux dans cette préversion :
* Enterprise, qui utilise la mémoire volatile (DRAM) sur une machine virtuelle pour stocker les données
* Enterprise Flash, qui utilise à la fois la mémoire volatile et la mémoire non volatile (NVMe ou SSD) pour stocker les données.

## <a name="prerequisites"></a>Prérequis

Vous aurez besoin d’un abonnement Azure avant de commencer. Si vous n’en avez pas, créez un [compte](https://azure.microsoft.com/). Pour plus d’informations, consultez [Conditions du niveau Enterprise](cache-overview.md#enterprise-tier-requirements).

## <a name="create-a-cache"></a>Création d'un cache
1. Pour créer un cache, connectez-vous au portail Azure via le lien dans votre invitation à la préversion, puis sélectionnez **Créer une ressource**.

1. Dans la page **Nouvelle**, sélectionnez **Bases de données**, puis **Azure Cache pour Redis**.
   
   :::image type="content" source="media/cache-create/new-cache-menu.png" alt-text="Sélectionner Azure Cache pour Redis":::
   
1. Dans la page **Nouveau cache Redis**, configurez les paramètres du nouveau cache.
   
   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.<Azure region>.redisenterprise.cache.azure.net*. | 
   | **Lieu** | Dans la liste déroulante, sélectionnez un emplacement. | Les niveaux Enterprise sont disponibles dans un nombre limité de régions Azure pendant la préversion. |
   | **Type de cache** | Dans la liste déroulante, sélectionnez un niveau *Enterprise* ou *Enterprise Flash* et une taille. |  Le niveau détermine la taille, les performances et les fonctionnalités disponibles pour le cache. |
   
   :::image type="content" source="media/cache-create/enterprise-tier-basics.png" alt-text="Notions de base sur le niveau Enterprise":::

   > [!NOTE] 
   > Veillez à cocher la case sous « Conditions » avant de continuer.
   >

1. Sélectionnez **Suivant : Mise en réseau** et ignorez.

   > [!NOTE] 
   > L’option Liaison privée est en cours de déploiement et peut ne pas être tout de suite disponible dans votre région.
   >

1. Sélectionnez **Suivant : Avancé**, puis définissez **Stratégie de clustering** sur **Entreprise**.
   
   Vous pouvez conserver les paramètres par défaut ou les modifier en fonction de vos besoins. Lorsque vous activez **Autoriser l’accès uniquement via TLS**, vous devez utiliser TLS pour accéder au nouveau cache à partir de votre application.

   :::image type="content" source="media/cache-create/enterprise-tier-advanced.png" alt-text="Niveau Enterprise avancé":::

   > [!NOTE] 
   > Les modules Redis ne sont pas encore pris en charge dans le niveau Enterprise Flash. Si vous envisagez d’utiliser un module Redis, veillez à choisir un cache de niveau Enterprise.
   >
   
1. Sélectionnez **Suivant : Étiquettes** et ignorez.

1. Sélectionnez **Suivant : Vérifier + créer**.

   :::image type="content" source="media/cache-create/enterprise-tier-summary.png" alt-text="Résumé du niveau Enterprise":::

1. Vérifiez les paramètres, puis cliquez sur **Créer**.
   
   La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution**, le cache est prêt pour utilisation.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer une instance de niveau Entreprise pour Azure Cache pour Redis.

> [!div class="nextstepaction"]
> [Créer une application web ASP.NET qui utilise un cache Azure pour Redis.](./cache-web-app-howto.md)

