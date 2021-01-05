---
title: Vue d’ensemble de la passerelle auto-hébergée | Microsoft Docs
description: Découvrez comment la fonctionnalité de passerelle auto-hébergée Gestion des API Azure aide les organisations à gérer les API dans des environnements hybrides et multiclouds.
services: api-management
documentationcenter: ''
author: vlvinogr
manager: gwallace
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 04/26/2020
ms.author: apimpm
ms.openlocfilehash: b560b02544eeb96167e68ed305d4d9942d2b1e0f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82232970"
---
# <a name="self-hosted-gateway-overview"></a>Vue d’ensemble de la passerelle auto-hébergée

Cet article explique comment la fonctionnalité de passerelle auto-hébergée de Gestion des API Azure permet la gestion des API hybrides et multiclouds, présente son architecture de haut niveau et met en évidence ses capacités.

## <a name="hybrid-and-multi-cloud-api-management"></a>Gestion des API hybrides et multiclouds

La fonctionnalité de passerelle auto-hébergée étend la prise en charge Gestion des API pour les environnements hybrides et multiclouds, et permet aux organisations de gérer efficacement et en toute sécurité les API hébergées localement et entre les clouds à partir d’un seul service de gestion des API dans Azure.

Avec la passerelle auto-hébergée, les clients ont la possibilité de déployer une version en conteneur du composant de la passerelle Gestion des API dans les mêmes environnements que ceux où ils hébergent leurs API. Toutes les passerelles auto-hébergées sont managées à partir du service Gestion des API avec lequel elles sont fédérées. Ainsi, les clients bénéficient d’une visibilité et d’une expérience de gestion unifiée sur l’ensemble des API internes et externes. Le placement des passerelles à proximité des API permet aux clients d’optimiser les flux de trafic d’API et de répondre aux exigences de sécurité et de conformité.

Chaque service Gestion des API se compose des éléments clés suivants :

-   Plan de gestion, exposé sous la forme d’une API, utilisé pour configurer le service via le Portail Azure, PowerShell et d’autres mécanismes pris en charge.
-   La passerelle (ou le plan de données) est responsable du proxy des demandes d’API, de l’application des stratégies et de la collecte des données de télémétrie
-   Portail des développeurs utilisé par les développeurs pour découvrir, apprendre et intégrer afin d’utiliser les API

Par défaut, tous ces composants sont déployés dans Azure, ce qui entraîne le flux de tout le trafic API (représenté par des flèches noires pleines sur l’image ci-dessous) dans Azure, quel que soit l’endroit où les serveurs principaux implémentant les API sont hébergés. La simplicité opérationnelle de ce modèle a un coût : une latence accrue, des problèmes de conformité et, dans certains cas, des frais de transfert de données supplémentaires.

![Flux de trafic d’API sans passerelles auto-hébergées](media/self-hosted-gateway-overview/without-gateways.png)

Le déploiement de passerelles auto-hébergées dans les mêmes environnements où sont hébergées les implémentations d’API back-end permet au trafic d’API de circuler directement vers les API back-end. Ceci améliore la latence, optimise les coûts de transfert de données et assure la conformité tout en conservant les avantages d’un point unique de gestion, d’observabilité et de découverte de toutes les API au sein de l’organisation, quel que soit l’emplacement où leurs implémentations sont hébergées.

![Flux de trafic d’API avec passerelles auto-hébergées](media/self-hosted-gateway-overview/with-gateways.png)

## <a name="packaging-and-features"></a>Empaquetage et caractéristiques

La passerelle auto-hébergée est une version en conteneur, fonctionnellement équivalente, de la passerelle managée déployée sur Azure dans le cadre de chaque service Gestion des API. La passerelle auto-hébergée est disponible sous la forme d’un [conteneur](https://aka.ms/apim/sputnik/dhub) Docker basé sur Linux à partir de Microsoft Container Registry. Elle peut être déployée sur Docker, Kubernetes ou toute autre solution d’orchestration de conteneurs s’exécutant sur un cluster de serveurs local, sur une infrastructure cloud ou, à des fins d’évaluation et de développement, sur un ordinateur personnel.

Les fonctionnalités suivantes, présentes dans les passerelles gérées, **ne sont pas disponibles** dans les passerelles auto-hébergées :

- Journaux d’activité Azure Monitor
- Gestion du chiffrement et de la version TLS en amont (côté principal)
- Validation des certificats serveur et client à l’aide de [certificats racine de l’autorité de certification](api-management-howto-ca-certificates.md) chargés vers le service Gestion des API. Pour ajouter la prise en charge d’une autorité de certification personnalisée, ajoutez une couche à l’image conteneur de la passerelle auto-hébergée qui installe le certificat racine de l’autorité de certification
- Intégration à [Service Fabric](../service-fabric/service-fabric-api-management-overview.md)
- Reprise de session TLS
- Renégociation du certificat client. Cela signifie que, pour que l’[authentification par certificat client](api-management-howto-mutual-certificates-for-clients.md) fonctionne, les utilisateurs d’API doivent présenter leurs certificats dans le cadre de la négociation TLS initiale. Pour vous en assurer, activez le paramètre de négociation du certificat client lors de la configuration d’un nom d’hôte personnalisé pour une passerelle auto-hébergée
- Cache intégré. Consultez ce [document](api-management-howto-cache-external.md) pour en savoir plus sur l’utilisation du cache externe dans les passerelles auto-hébergées

## <a name="connectivity-to-azure"></a>Connectivité à Azure

Les passerelles auto-hébergées nécessitent une connectivité TCP/IP sortante vers Azure sur le port 443. Chaque passerelle auto-hébergée doit être associée à un seul service Gestion des API et est configurée via son plan de gestion. La passerelle auto-hébergée utilise la connectivité à Azure pour les actions suivantes :

-   Signalement de son état en envoyant des messages de pulsations toutes les minutes
-   Vérification régulière (toutes les 10 secondes) et application des mises à jour de configuration chaque fois qu’elles sont disponibles
-   Envoi de journaux de demandes et de métriques à Azure Monitor, si elle est configurée pour ce faire
-   Envoi d’événements à Application Insights, si elle est configurée pour ce faire

Lorsque la connectivité à Azure est perdue, la passerelle auto-hébergée ne peut plus recevoir de mises à jour de configuration, signaler son état ou charger les données de télémétrie.

La passerelle auto-hébergée est conçue pour « basculer en mode statique » et peut survivre à une perte temporaire de connectivité à Azure. Elle peut être déployée avec ou sans sauvegarde de configuration locale. Dans le premier cas, les passerelles auto-hébergées enregistrent régulièrement une copie de sauvegarde de la configuration téléchargée la plus récente sur un volume persistant attaché à son conteneur ou à son pod.

Lorsque la sauvegarde de la configuration est désactivée et que la connectivité à Azure est interrompue :

-   Les passerelles auto-hébergées en cours d’exécution continuent à fonctionner à l’aide d’une copie en mémoire de la configuration
-   Les passerelles auto-hébergées arrêtées ne peuvent pas démarrer

Lorsque la sauvegarde de la configuration est activée et que la connectivité à Azure est interrompue :

-   Les passerelles auto-hébergées en cours d’exécution continuent à fonctionner à l’aide d’une copie en mémoire de la configuration
-   Les passerelles auto-hébergées arrêtées peuvent démarrer à l’aide d’une copie de sauvegarde de la configuration

Lorsque la connectivité est restaurée, chaque passerelle auto-hébergée affectée par la panne se reconnecte automatiquement à son service Gestion des API associé et télécharge toutes les mises à jour de configuration qui se sont produites pendant que la passerelle était « hors connexion ».

## <a name="next-steps"></a>Étapes suivantes

-   [Lire un livre blanc pour plus d’informations sur cette rubrique](https://aka.ms/hybrid-and-multi-cloud-api-management)
-   [Déployer une passerelle auto-hébergée sur Docker](how-to-deploy-self-hosted-gateway-docker.md)
-   [Déployer une passerelle auto-hébergée sur Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md)
