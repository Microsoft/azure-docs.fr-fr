---
title: Gérer plusieurs locataires avec Video Indexer – Azure
description: Cet article propose différentes options d’intégration pour la gestion de plusieurs locataires avec Video Indexer.
services: media-services
documentationcenter: ''
author: ika-microsoft
manager: femila
editor: ''
ms.service: media-services
ms.subservice: video-indexer
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 05/15/2019
ms.author: ikbarmen
ms.openlocfilehash: 18f2cf3daa281400151ba223e1735e7138d97e8e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76990502"
---
# <a name="manage-multiple-tenants"></a>Gérer plusieurs locataires

Cet article aborde différentes options de gestion de plusieurs locataires avec Video Indexer. Choisissez la méthode la plus adaptée à votre scénario :

* Un compte Video Indexer par locataire
* Un compte Video Indexer unique pour tous les locataires
* Un abonnement Azure par locataire

## <a name="video-indexer-account-per-tenant"></a>Un compte Video Indexer par locataire

Selon cette architecture, un compte Video Indexer est créé pour chaque locataire. Les locataires bénéficient d’une isolation totale dans la couche persistante et de calcul.  

![Un compte Video Indexer par locataire](./media/manage-multiple-tenants/video-indexer-account-per-tenant.png)

### <a name="considerations"></a>Considérations

* Les clients ne partagent pas les comptes de stockage (sauf configuration manuelle par le client).
* Les clients ne partagent pas le calcul (unités réservées) et n’ont pas d’incidence sur le temps de traitement des tâches les uns des autres.
* Pour retirer un locataire du système, il suffit de supprimer le compte Video Indexer.
* Il n’est pas possible de partager des modèles personnalisés entre locataires.

    Vérifiez que ce type de partage ne fait pas partie des exigences de l’entreprise.
* Ce modèle est plus difficile à gérer, en raison de la présence de plusieurs comptes Video Indexer (et Media Services associés) par locataire.

> [!TIP]
> Créez un utilisateur administrateur pour votre système sur [Video Indexer Developer Portal](https://api-portal.videoindexer.ai/) et utilisez l’API d’autorisation pour fournir à vos locataires le [jeton d’accès au compte](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Account-Access-Token) nécessaire.

## <a name="single-video-indexer-account-for-all-users"></a>Un compte Video Indexer unique pour tous les utilisateurs

Selon cette architecture, l’isolation des locataires incombe au client. Tous les locataires doivent utiliser un même compte Video Indexer avec un seul compte Azure Media Services. Pour effectuer une recherche, charger ou supprimer du contenu, le client devra créer un filtre permettant d’afficher les résultats correspondant au client.

![Un compte Video Indexer unique pour tous les utilisateurs](./media/manage-multiple-tenants/single-video-indexer-account-for-all-users.png)

Avec cette option, il est possible de partager ou d’isoler les modèles de personnalisation (personne, langue et marques) entre les locataires en filtrant les modèles par locataire.

Lors du [chargement de vidéos](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?), vous pouvez spécifier un autre attribut de partition par locataire, ce qui permet l’isolation dans [l’API de recherche](https://api-portal.videoindexer.ai/docs/services/operations/operations/Search-videos?). Ainsi, vous n’obtiendrez que les résultats de la partition spécifiée. 

### <a name="considerations"></a>Considérations

* Il est possible de partager du contenu et des modèles de personnalisation entre les locataires.
* Un locataire affecte les performances des autres locataires.
* Le client doit créer une couche de gestion complexe sur Video Indexer.

> [!TIP]
> Vous pouvez utiliser l’attribut [priority](upload-index-videos.md) pour classer les tâches des locataires par ordre de priorité.

## <a name="azure-subscription-per-tenant"></a>Un abonnement Azure par locataire 

Selon cette architecture, chaque locataire a son propre abonnement Azure. Pour chaque utilisateur, vous créez un compte Video Indexer dans l’abonnement du locataire.

![Un abonnement Azure par locataire](./media/manage-multiple-tenants/azure-subscription-per-tenant.png)

### <a name="considerations"></a>Considérations

* C’est la seule option permettant de séparer la facturation.
* Cette intégration demande plus de gestion que l’architecture Un compte Video Indexer par locataire. Si la facturation n’est pas obligatoire, il est recommandé d’utiliser une des autres options décrites dans cet article.

## <a name="next-steps"></a>Étapes suivantes

[Vue d'ensemble](video-indexer-overview.md)
