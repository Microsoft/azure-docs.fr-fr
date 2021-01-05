---
title: Azure Front Door - Prise en charge du protocole HTTP2 | Microsoft Docs
description: Cet article vous aide à en savoir plus sur la prise en charge du protocole HTTP/2 dans Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 20d45f5966aca3df89e17e03aa6120a4ddf5c5b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91448700"
---
# <a name="http2-support-in-azure-front-door"></a>Prise en charge du protocole HTTP/2 dans Azure Front Door

Actuellement, la prise en charge de HTTP/2 est active pour toutes les configurations Azure Front Door. Aucune action supplémentaire n’est nécessaire de la part des clients.

HTTP/2 est une révision majeure de HTTP/1.1 qui vous permet de bénéficier de performances web plus rapides en réduisant le temps de réponse. Pour améliorer l'expérience utilisateur, HTTP/2 conserve les méthodes HTTP, les codes d'état et la sémantique qui vous sont familiers. Bien que HTTP/2 soit conçu pour fonctionner avec HTTP et HTTPS, de nombreux navigateurs web clients prennent uniquement en charge HTTP/2 sur TLS (Transport Layer Security).

> [!NOTE]
> La prise en charge du protocole HTTP/2 est disponible uniquement pour les requêtes des clients à Front Door. La communication entre Front Door et les serveurs principaux dans le pool back-end s’effectue via HTTP/1.1. 

### <a name="http2-benefits"></a>Avantages de HTTP/2

Les avantages de HTTP/2 sont les suivants :

*   **Multiplexage et accès concurrentiel**

    Avec HTTP 1.1, effectuer plusieurs demandes de ressources nécessite plusieurs connexions TCP, et chaque connexion représente une surcharge qui a un impact sur les performances. HTTP/2 permet de demander plusieurs ressources sur une même connexion TCP.

*   **Compression des en-têtes**

    Grâce à la compression des en-têtes HTTP pour les ressources traitées, le temps passé sur le réseau est sensiblement réduit.

*   **Dépendances de flux**

    Les dépendances de flux permettent au client indiquer au serveur quelles ressources ont la priorité.


## <a name="http2-browser-support"></a>Prise en charge de HTTP/2 par les navigateurs

Tous les principaux navigateurs ont implémenté la prise en charge de HTTP/2 dans leur version actuelle. Les navigateurs non pris en charge passent automatiquement à HTTP/1.1.

|Browser|Version minimale|
|-------------|------------|
|Microsoft Edge| 12|
|Google Chrome| 43|
|Mozilla Firefox| 38|
|Opera| 32|
|Safari| 9|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les HTTP/2, consultez les ressources suivantes :

- [Page d’accueil de la spécification HTTP/2](https://http2.github.io/)
- [Forum Aux Questions officiel de HTTP/2](https://http2.github.io/faq/)
- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
