---
title: Prise en charge de HTTP/2 dans Azure CDN | Microsoft Docs
description: Azure Content Delivery Network prend en charge HTTP/2, qui présente des avantages par rapport à HTTP/1, comme le multiplexage et l’accès concurrentiel, la compression des en-têtes et les dépendances de flux.
services: cdn
documentationcenter: ''
author: lichard
manager: erikre
editor: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 5/04/2017
ms.author: ril
ms.openlocfilehash: d4ed04185da251d9042f6b54a192d0e49ef20ede
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778545"
---
# <a name="http2-support-in-azure-cdn"></a>Prise en charge de HTTP/2 dans Azure CDN

HTTP/2 est une révision majeure de HTTP/1.1\. Il offre des performances web plus rapides, des meilleurs temps de réponse et une expérience utilisateur améliorée, tout en conservant les méthodes HTTP, les codes d’état et la sémantique habituels. Bien que HTTP/2 soit conçu pour fonctionner avec HTTP et HTTPS, de nombreux navigateurs web clients prennent en charge seulement HTTP/2 sur TLS.

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

## <a name="enabling-http2-support-in-azure-cdn"></a>Activation de la prise en charge de HTTP/2 dans Azure CDN

Actuellement, la prise en charge HTTP/2 est active pour tous les profils d’Azure CDN. Aucune action supplémentaire n’est nécessaire de la part des clients.

## <a name="next-steps"></a>Étapes suivantes

Pour voir les avantages de HTTP/2 en situation, consultez [cette démonstration d’Akamai](https://http2.akamai.com/demo).

Pour en savoir plus sur les HTTP/2, consultez les ressources suivantes :

*   [Page d’accueil de la spécification HTTP/2](https://http2.github.io/)
*   [Forum Aux Questions officiel de HTTP/2](https://http2.github.io/faq/)
*   [Informations sur HTTP/2 d’Akamai](https://http2.akamai.com/)

Pour plus d’informations sur les fonctionnalités disponibles dans Azure CDN, consultez [Vue d’ensemble du réseau de distribution de contenu (CDN) Azure](./cdn-overview.md).