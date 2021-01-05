---
title: Fichier include
description: Fichier include
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: include
ms.date: 05/09/2019
ms.author: sharadag
ms.custom: include file
ms.openlocfilehash: 361fb18574e90fc46e45bff8914c51ee1afa2f96
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89411739"
---
| Ressource | Limite |
| --- | --- |
| Ressources Azure Front Door par abonnement | 100 |
| Hôtes front-end, y compris les domaines personnalisés par ressource | 500 |
| Règles de routage par ressource | 500 |
| Pools back-end par ressource | 50 |
| Serveurs principaux par pool back-end | 100 |
| Modèles de chemin à utiliser pour une règle de routage | 25 |
| URL dans un appel de vidage du cache unique | 100 |
| Règles personnalisées de pare-feu d’applications web par stratégie | 100 |
| Stratégie de pare-feu d’applications web par abonnement | 100 |
| Conditions de correspondance du pare-feu d'applications web par règle personnalisée | 10 |
| Plages d’adresses IP du pare-feu d'applications web par condition de correspondance | 600 |
| Valeurs de correspondance de chaîne du pare-feu d'applications web par condition de correspondance | 10 |
| Longueur de valeur de correspondance de chaîne du pare-feu d'applications web | 256 |
| Longueur de nom de paramètre du corps POST du pare-feu d’applications web | 256 |
| Longueur du nom d’en-tête HTTP du pare-feu d’applications web | 256 |
| Longueur du nom de cookie du pare-feu d’applications web | 256 |
| Taille du corps de la requête HTTP inspectée du pare-feu d'applications web | 128 Ko |
| Longueur du corps de la réponse personnalisée du pare-feu d’applications web | 2 Ko |

### <a name="timeout-values"></a>Valeurs de délai d’expiration
#### <a name="client-to-front-door"></a>Du client à Front Door
* Front Door a un délai de 61 secondes pour les connexions TCP inactives.

#### <a name="front-door-to-application-back-end"></a>De Front Door au serveur d’applications back-end
* Si la réponse est mémorisée en bloc, un code d’état de 200 est renvoyé lors de la réception du premier bloc.
* Une fois que la requête HTTP est transférée au serveur principal, Front Door attend 30 secondes l’arrivée du premier paquet depuis le serveur principal. Ensuite, il renvoie une erreur 503 au client. Vous pouvez configurer cette valeur par le biais du champ sendRecvTimeoutSeconds dans l’API.
    * Pour les scénarios de mise en cache, ce délai d’expiration n’est pas configurable. Par conséquent, si une requête est mise en cache et qu’elle prend plus de 30 secondes pour le premier paquet en provenance de Front Door ou du back-end, une erreur 504 est retournée au client. 
* Lorsqu’il reçoit le premier paquet du serveur principal, Front Door attend 30 secondes (délai d’inactivité). Ensuite, il renvoie une erreur 503 au client. Cette valeur de délai d’expiration n’est pas configurable.
* Le délai d’expiration d’une session TCP entre Front Door et le back-end est de 90 secondes.

### <a name="upload-and-download-data-limit"></a>Taille limite pour le chargement et le téléchargement des données

|  | Avec encodage de transfert mémorisé en bloc (CTE) | Sans segmentation HTTP |
| ---- | ------- | ------- |
| **Télécharger** | La taille du téléchargement n’est pas limitée. | La taille du téléchargement n’est pas limitée. |
| **Charger** |    Aucune limite tant que la taille de chaque chargement avec encodage de transfert mémorisé en bloc est inférieure à 2 Go. | Elle ne peut pas être supérieure à 2 Go. |

### <a name="other-limits"></a>Autres limites
* Taille d’URL maximale : 8 192 octets - Spécifie la longueur maximale de l’URL brute (schéma + nom d’hôte + port + chemin + chaîne de requête de l’URL)
* Taille maximale de chaîne de requête : 4 096 octets - Spécifie la longueur maximale de la chaîne de requête en octets.
* Taille maximale d’en-tête de réponse HTTP à partir de l’URL de la sonde d’intégrité - 4096 octets - Spécifie la longueur maximale de tous les en-têtes de réponse des sondes d’intégrité. 
