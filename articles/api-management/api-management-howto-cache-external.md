---
title: Utiliser un cache externe dans Gestion des API Azure | Microsoft Docs
description: Découvrez comment configurer et utiliser un cache externe dans Gestion des API Azure. L’utilisation d’un cache externe vous permet de pallier quelques limites du cache intégré.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.assetid: 740f6a27-8323-474d-ade2-828ae0c75e7a
ms.service: api-management
ms.topic: conceptual
ms.date: 04/26/2020
ms.author: apimpm
ms.openlocfilehash: 6288a10e111e42629abf5e09b84a6a7791dcfe95
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018220"
---
# <a name="use-an-external-redis-compatible-cache-in-azure-api-management"></a>Utiliser un cache externe compatible Redis dans Gestion des API Azure

En plus de l’utilisation du cache intégré, Gestion des API Azure permet la mise en cache externe compatible Redis, par ex. dans un Cache Azure pour Redis.

L’utilisation d’un cache externe vous permet de pallier quelques limites du cache intégré :

* Éviter que votre cache soit régulièrement effacé lors des mises à jour de Gestion des API
* Obtenir un plus grand contrôle sur la configuration de votre cache
* Mettre en cache plus de données que ce que votre niveau Gestion des API vous permet
* Utiliser la mise en cache avec le niveau Consommation de Gestion des API
* Activer la mise en cache dans les [passerelles auto-hébergées de gestion des API](self-hosted-gateway-overview.md)

Pour plus d’informations sur la mise en cache, consultez [Stratégies de mise en cache dans Gestion des API](api-management-caching-policies.md) et [Mise en cache personnalisée dans Gestion des API Azure](api-management-sample-cache-by-key.md).

![Apport de votre propre cache à APIM](media/api-management-howto-cache-external/overview.png)

Ce que vous allez apprendre :

> [!div class="checklist"]
> * Ajouter un cache externe dans Gestion des API

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

+ [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)
+ Comprendre la [mise en cache dans Gestion des API Azure](api-management-howto-cache.md)

## <a name="create-azure-cache-for-redis"></a><a name="create-cache"> </a> Créer un cache Azure pour Redis

Cette section explique comment créer un Cache Azure pour Redis dans Azure. Si vous disposez déjà d’un Cache Azure pour Redis, dans ou en dehors d’Azure, vous pouvez <a href="#add-external-cache">passer</a> à la section suivante.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="deploy-redis-cache-to-kubernetes"></a><a name="create-cache"> </a> Déployer le cache Redis sur Kubernetes

Pour la mise en cache, les passerelles auto-hébergées s’appuient exclusivement sur les caches externes. Pour que la mise en cache prenne en compte les passerelles auto-hébergées et le cache sur lequel elles s’appuient, elle doit être située à proximité de chaque élément afin de minimiser les latences de recherche et de stockage. Le déploiement d’un cache Redis dans le même cluster Kubernetes ou dans un cluster distinct à proximité est le meilleur choix. Pour savoir comment déployer le cache Redis sur un cluster Kubernetes, suivez ce [lien](https://github.com/kubernetes/examples/tree/master/guestbook).

## <a name="add-an-external-cache"></a><a name="add-external-cache"> </a> Ajouter un cache externe

Suivez les étapes ci-dessous pour ajouter un Cache Azure pour Redis externe dans Gestion des API Azure.

![Capture d’écran montrant comment ajouter une compte Azure Cache pour Redis externe dans Gestion des API Azure.](media/api-management-howto-cache-external/add-external-cache.png)

> [!NOTE]
> Le paramètre **Utiliser à partir de** spécifie une région Azure ou un emplacement de passerelle auto-hébergée qui utilisera le cache configuré. Les caches configurés **par défaut** sont remplacés par des caches avec une valeur de région ou d’emplacement correspondante spécifique.
>
> Par exemple, si le service Gestion des API est hébergé dans les régions USA Est, Asie Sud-Est et Europe Ouest, et que deux caches sont configurés, l’un **Par défaut** et l’autre pour **Asie Sud-Est**, l’instance Gestion des API de la région **Asie Sud-Est** utilise son propre cache tandis que les deux autres régions utilisent l’entrée de cache **Par défaut**.

### <a name="add-an-azure-cache-for-redis-from-the-same-subscription"></a>Ajouter un Cache Azure pour Redis à partir du même abonnement

1. Accédez à votre instance Gestion des API dans le portail Azure.
2. Sélectionnez l’onglet **Cache externe** dans le menu de gauche.
3. Cliquez sur le bouton **+ Ajouter** .
4. Sélectionnez votre cache dans la liste déroulante **Instance de cache**.
5. Sélectionnez **Par défaut** ou précisez la région souhaitée dans la zone de liste déroulante **Utiliser depuis**.
6. Cliquez sur **Enregistrer**.

### <a name="add-an-azure-cache-for-redis-hosted-outside-of-the-current-azure-subscription-or-azure-in-general"></a>Ajouter un Cache Azure pour Redis hébergé en dehors de l’abonnement Azure actuel ou d’Azure en général

1. Accédez à votre instance Gestion des API dans le portail Azure.
2. Sélectionnez l’onglet **Cache externe** dans le menu de gauche.
3. Cliquez sur le bouton **+ Ajouter** .
4. Sélectionnez **Personnalisé** dans la zone de liste déroulante **Instance de cache**.
5. Sélectionnez **Par défaut** ou précisez la région souhaitée dans la zone de liste déroulante **Utiliser depuis**.
6. Indiquez la chaîne de connexion de votre Cache Azure pour Redis dans le champ **Chaîne de connexion**.
7. Cliquez sur **Enregistrer**.

### <a name="add-a-redis-cache-to-a-self-hosted-gateway"></a>Ajouter un cache Redis à une passerelle auto-hébergée

1. Accédez à votre instance Gestion des API dans le portail Azure.
2. Sélectionnez l’onglet **Cache externe** dans le menu de gauche.
3. Cliquez sur le bouton **+ Ajouter** .
4. Sélectionnez **Personnalisé** dans la zone de liste déroulante **Instance de cache**.
5. Spécifiez l’emplacement de la passerelle auto-hébergée souhaitée ou **Par défaut** dans le champ déroulant **Utiliser à partir de**.
6. Indiquez la chaîne de connexion de votre cache Redis dans le champ **Chaîne de connexion**.
7. Cliquez sur **Enregistrer**.

## <a name="use-the-external-cache"></a>Utiliser le cache externe

Une fois le cache externe configuré dans Gestion des API Azure, il peut être utilisé par le biais des stratégies de mise en cache. Pour les instructions détaillées, consultez [Ajouter une mise en cache pour améliorer les performances de Gestion des API Azure](api-management-howto-cache.md).

## <a name="next-steps"></a><a name="next-steps"> </a>Étapes suivantes

* Pour plus d’informations sur les stratégies de mise en cache, voir la section [Stratégies de mise en cache][Caching policies] dans [Référence de stratégie de Gestion des API][API Management policy reference].
* Pour plus d’informations sur la mise en cache des éléments par clé à l’aide d’expressions de stratégie, consultez [Mise en cache personnalisée dans la gestion des API Azure](api-management-sample-cache-by-key.md).

[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-caching-policies.md
