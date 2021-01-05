---
title: Exemples de stratégie dans la Gestion des API Azure | Microsoft Docs
description: Découvrez les stratégies disponibles dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: sample
ms.date: 10/31/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: b5b8c82aa420b62e3b6e68ee53352eb9f77988f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86506687"
---
# <a name="api-management-policy-samples"></a>Exemples de stratégie dans la Gestion des API

Les [stratégies](../api-management-howto-policies.md) sont une fonctionnalité puissante du système qui permet à l’éditeur de changer le comportement de l’API grâce à la configuration. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Le tableau suivant contient des liens vers des exemples et donne une brève description de chaque exemple.

| Stratégies de trafic entrant | Description |
| ---------------- | ----------- |
| [Ajouter un en-tête Forwarded pour autoriser l’API de service principal à concevoir des URL correctes](./set-header-to-enable-backend-to-construct-urls.md) | Montre comment ajouter un en-tête Forwarded à la demande entrante pour autoriser l’API de service principal à construire des URL correctes.                                                                                                        |
| [Ajouter un en-tête contenant un ID de corrélation](./add-correlation-id.md)                                                             | Montre comment ajouter un en-tête contenant un ID de corrélation à la requête entrante.                                                                                                                                        |
| [Ajouter des fonctionnalités à un service principal et mettre en cache la réponse](./cache-response.md)                                             | Montre comment ajouter des fonctionnalités à un service principal. Par exemple, accepter un nom d’emplacement au lieu des latitude et longitude dans une API de prévisions météorologiques.                                                                    |
| [Autoriser l’accès basé sur les revendications JWT](./authorize-request-based-on-jwt-claims.md)                                              | Montre comment autoriser l’accès à des méthodes HTTP spécifiques sur une API basée sur les revendications JWT.                                                                                                                                       |
| [Autoriser les demandes à l’aide d’agent d’autorisation externe](./authorize-request-using-external-authorizer.md)                                                   | Montre comment utiliser l’agent d’autorisation externe pour la sécurisation de l’accès à l’API.                                                                                                                                                               |
| [Autoriser l’accès à l’aide du jeton Google OAuth](./use-google-as-oauth-token-provider.md)                                            | Montre comment autoriser l’accès à vos points de terminaison en utilisant Google comme fournisseur de jeton OAuth.                                                                                                                                    |
| [Filtrer les adresses IP durant l’utilisation d’une passerelle Application Gateway](./filter-ip-addresses-when-using-appgw.md) | Montre comment filtrer les adresses IP dans les stratégies quand l’instance de Gestion des API est accessible via une passerelle Application Gateway
| [Générer une signature d’accès partagé et transférer la demande vers le stockage Azure](./generate-shared-access-signature.md)                  | Montre comment générer une [signature d’accès partagé](../../storage/common/storage-sas-overview.md) à l’aide d’expressions et transférer la demande vers le stockage Azure avec une stratégie rewrite-uri. |
| [Obtenir un jeton d’accès OAuth2 d’AAD et le transférer au backend](./use-oauth2-for-authorization.md)                             | Fournit un exemple d’utilisation d’OAuth2 pour l’autorisation entre la passerelle et un backend. Il montre comment obtenir un jeton d’accès d’AAD et le transférer au serveur principal.                                                    |
| [Obtenir un jeton X-CSRF de la passerelle SAP à l’aide d’une stratégie send-request](./get-x-csrf-token-from-sap-gateway.md)                           | Montre comment implémenter un modèle X-CSRF utilisé par de nombreuses API. Cet exemple est spécifique de la passerelle SAP.                                                                                                                           |
| [Router la demande en fonction de la taille de son corps](./route-requests-based-on-size.md)                                            | Montre comment router les demandes en fonction de la taille de leurs corps.                                                                                                                                                       |
| [Envoyer des informations de contexte de demande au backend](./send-request-context-info-to-backend-service.md)                    | Montre comment envoyer des informations de contexte au backend à des fins de journalisation ou de traitement.                                                                                                                                |
| [Définir la durée du cache de réponse](./set-cache-duration.md)                                                                          | Montre comment définir la durée du cache de réponse à l’aide de la valeur maxAge dans l’en-tête Cache-Control envoyé par le backend.                                                                                                             |
| **Stratégies de trafic sortant** | **Description** |
| [Filtrer le contenu de la réponse](./filter-response-content.md)                                                                         | Montre comment filtrer des éléments de données à partir de la charge utile de la réponse en fonction du produit associé à la demande.                                                                                                        |
| **Stratégies en cas d’erreur** | **Description** |
| [Journaliser les erreurs dans Stackify](./log-errors-to-stackify.md)                                                                           | Montre comment ajouter une stratégie de journalisation des erreurs pour envoyer les erreurs à Stackify à des fins de journalisation.                                                                                                                                            |
