---
title: Comparaison des fonctionnalités des niveaux de la Gestion des API Azure | Microsoft Docs
description: Comparez les niveaux de service de Gestion des API en fonction des fonctionnalités qu’ils proposent. Consultez un tableau qui récapitule les principales fonctionnalités disponibles dans chaque niveau tarifaire.
services: api-management
documentationcenter: ''
author: vladvino
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/15/2020
ms.author: apimpm
ms.openlocfilehash: eec913237ba978e52bb64fbd4c1f043a9214ffc8
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077845"
---
# <a name="feature-based-comparison-of-the-azure-api-management-tiers"></a>Comparaison des fonctionnalités des niveaux de la Gestion des API Azure

Chacun des [niveaux tarifaires](https://aka.ms/apimpricing) de la Gestion des API propose un ensemble distinct de fonctionnalités et une [capacité](api-management-capacity.md) par unité. Le tableau suivant récapitule les principales fonctionnalités disponibles dans chacun d’entre eux. Certaines fonctionnalités fonctionnent différemment ou sont différentes selon les niveaux. Dans ce cas, les différences sont mises en évidence dans les articles de la documentation qui décrivent ces différentes fonctionnalités.

> [!IMPORTANT]
> Veuillez noter que le niveau développeur est uniquement pour les cas d’utilisation et les évaluations hors production. Il ne propose pas de SLA.

| Fonctionnalité                                                                                      | Consommation | Développeur | De base | Standard | Premium |
| -------------------------------------------------------------------------------------------- | ----------- | --------- | ----- | -------- | ------- |
| Intégration à Azure AD<sup>1</sup>                                                             | Non          | Oui       | Non    | Oui      | Oui     |
| Prise en charge du réseau virtuel                                                               | Non          | Oui       | Non    | Non       | Oui     |
| Déploiement multi-régions                                                                      | Non          | Non        | Non    | Non       | Oui     |
| Noms de domaine personnalisés multiples                                                                 | Non          | Oui        | Non    | Non       | Oui     |
| Portail des développeurs<sup>2</sup>                                                                 | Non          | Oui       | Oui   | Oui      | Oui     |
| Cache intégré                                                                               | Non          | Oui       | Oui   | Oui      | Oui     |
| Analytique intégrée                                                                           | Non          | Oui       | Oui   | Oui      | Oui     |
| [Passerelle auto-hébergée](self-hosted-gateway-overview.md)<sup>3</sup>                           | Non          | Oui       | Non    | Non       | Oui     |
| [Paramètres TLS](api-management-howto-manage-protocols-ciphers.md)                             | Oui         | Oui       | Oui   | Oui      | Oui     |
| [Cache externe](./api-management-howto-cache-external.md)                                                    | Oui         | Oui       | Oui   | Oui      | Oui     |
| [Authentification par certificat client](api-management-howto-mutual-certificates-for-clients.md) | Oui         | Oui       | Oui   | Oui      | Oui     |
| [Sauvegarde et restauration](api-management-howto-disaster-recovery-backup-restore.md)               | Non          | Oui       | Oui   | Oui      | Oui     |
| [Gestion sur Git](api-management-configuration-repository-git.md)                        | Non          | Oui       | Oui   | Oui      | Oui     |
| API de gestion directe                                                                        | Non          | Oui       | Oui   | Oui      | Oui     |
| Journaux d’activité et métriques Azure Monitor                                                               | Oui         | Oui       | Oui   | Oui      | Oui     |
| Adresse IP statique                                                                                    | Non          | Oui       | Oui   | Oui      | Oui     |

<sup>1</sup> Permet l’utilisation de Microsoft Azure AD (et Azure AD B2C) en tant que fournisseur d’identité pour la connexion utilisateur sur le portail des développeurs.<br/>
<sup>2</sup> Y compris les fonctionnalités liées, par exemple, les utilisateurs, les groupes, les problèmes, les modèles d’application et de courrier électronique et les notifications.<br/>
<sup>3</sup> Au niveau Développeur, les passerelles auto-hébergées sont limitées à un seul nœud de passerelle.<br/>