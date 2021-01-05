---
title: Changements importants à venir sur Azure Security Center
description: Changements à venir sur Azure Security Center dont vous devez peut-être tenir compte et pour lesquels une planification peut être nécessaire
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/14/2020
ms.author: memildin
ms.openlocfilehash: 052758079d8d413f7b0fead2a5abf3b47b9a691e
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511329"
---
# <a name="important-upcoming-changes-to-azure-security-center"></a>Changements importants à venir sur Azure Security Center

> [!IMPORTANT]
> Les informations de cette page concernent des produits ou fonctionnalités en préversion, qui peuvent être modifiés de façon substantielle avant d’être commercialisés, le cas échéant. Microsoft n’offre aucun engagement ni garantie, formel ou implicite, concernant les informations contenues dans ce document.

Dans cette page, vous allez découvrir les changements planifiés pour Security Center. Elle décrit les changements planifiés du produit qui peuvent avoir un impact sur votre degré de sécurisation ou vos workflows.

Si vous recherchez les notes de publication les plus récentes, vous les trouverez dans [Nouveautés d’Azure Security Center](release-notes.md).


## <a name="planned-changes"></a>Changements planifiés

- [Ressources « non applicables » à signaler comme « Conformes » dans les évaluations Azure Policy](#not-applicable-resources-to-be-reported-as-compliant-in-azure-policy-assessments)
- [Ajout de 35 recommandations (préversion) pour mieux détailler le benchmark de sécurité Azure](#35-preview-recommendations-being-added-to-increase-coverage-of-azure-security-benchmark)

### <a name="not-applicable-resources-to-be-reported-as-compliant-in-azure-policy-assessments"></a>Ressources « non applicables » à signaler comme « Conformes » dans les évaluations Azure Policy

**Date estimée de la modification :** Janvier 2021

Actuellement, les ressources qui sont évaluées pour une recommandation et considérées comme **non applicables** apparaissent dans Azure Policy comme « Non conformes ». Aucune action de l’utilisateur ne peut changer leur état en « Conforme ». À partir de ce changement planifié, elles seront signalées comme « Conformes » pour une plus grande clarté.

La seule incidence sera observée dans Azure Policy, où le nombre de ressources compatibles augmentera. Il n’y aura aucune incidence sur votre degré de sécurisation dans Azure Security Center.

### <a name="35-preview-recommendations-being-added-to-increase-coverage-of-azure-security-benchmark"></a>Ajout de 35 recommandations (préversion) pour mieux détailler le benchmark de sécurité Azure

**Date estimée de la modification :** Décembre 2020

Le Benchmark de sécurité Azure est l’ensemble des directives propres à Azure et créées par Microsoft contenant les bonnes pratiques de sécurité et de conformité basées sur les infrastructures de conformité courantes. [En savoir plus sur le Benchmark de sécurité Azure](../security/benchmarks/introduction.md).

Les 35 recommandations suivantes (préversion) seront ajoutées à Security Center pour mieux détailler ce benchmark.

Les recommandations en préversion ne rendent pas une ressource non saine et ne sont pas incluses dans les calculs de votre degré de sécurisation. Corrigez-les là où c’est possible, de sorte que quand la période de préversion se termine, elles soient prises en compte dans le calcul de votre degré de sécurisation. Découvrez comment répondre à ces recommandations dans [Corriger les recommandations dans Azure Security Center](security-center-remediate-recommendations.md).

| Contrôle de sécurité                     | Nouvelles recommandations                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Activer le chiffrement des données au repos            | - Les comptes Azure Cosmos DB doivent utiliser des clés gérées par le client pour chiffrer les données au repos<br>- Les espaces de travail Azure Machine Learning doivent être chiffrés avec une clé gérée par le client (CMK)<br>- La protection des données BYOK (Bring Your Own Key) doit être activée pour les serveurs MySQL<br>- La protection des données BYOK (Bring Your Own Key) doit être activée pour les serveurs PostgreSQL<br>- Les comptes Cognitive Services doivent activer le chiffrement des données avec une clé gérée par le client (CMK)<br>- Les registres de conteneurs doivent être chiffrés avec une clé gérée par le client (CMK)<br>- Les instances managées SQL doivent utiliser des clés gérées par le client pour chiffrer les données au repos<br>- Les serveurs SQL doivent utiliser des clés gérées par le client pour chiffrer les données au repos<br>- Les comptes de stockage doivent utiliser une clé gérée par le client (CMK) pour le chiffrement                                                                                                                                                              |
| Implémenter les bonnes pratiques de sécurité    | - Les abonnements doivent avoir l’adresse e-mail d’un contact pour le signalement des problèmes de sécurité<br> - Le provisionnement automatique de l’agent Log Analytics doit être activé sur votre abonnement<br> - La notification par e-mail pour les alertes à gravité élevée doit être activée<br> - La notification par e-mail au propriétaire de l’abonnement pour les alertes à gravité élevée doit être activée<br> - La protection contre la suppression définitive doit être activée sur les coffres de clés<br> - La suppression réversible doit être activée sur les coffres de clés |
| Gérer l’accès et les autorisations        | - Les applications de fonction doivent avoir l’option « Certificats clients (certificats clients entrants) » activée |
| Protéger les applications contre les attaques DDoS | - Le pare-feu d’applications web (WAF) doit être activé pour Application Gateway<br> - Web Application Firewall (WAF) doit être activé pour le service Azure Front Door Service |
| Restreindre l’accès réseau non autorisé | - Le pare-feu doit être activé sur Key Vault<br> - Le point de terminaison privé doit être configuré pour Key Vault<br> - App Configuration doit utiliser une liaison privée<br> - Azure Cache pour Redis doit se trouver dans un réseau virtuel<br> - Les domaines Azure Event Grid doivent utiliser une liaison privée<br> - Les rubriques Azure Event Grid doivent utiliser une liaison privée<br> - Les espaces de travail Azure Machine Learning doivent utiliser une liaison privée<br> - Azure SignalR Service doit utiliser une liaison privée<br> - Azure Spring Cloud doit utiliser l’injection de réseau<br> - Les registres de conteneurs ne doivent pas autoriser un accès réseau sans restriction<br> - Les registres de conteneurs doivent utiliser une liaison privée<br> - L’accès au réseau public doit être désactivé pour les serveurs MariaDB<br> - L’accès au réseau public doit être désactivé pour les serveurs MySQL<br> - L’accès au réseau public doit être désactivé pour les serveurs PostgreSQL<br> - Le compte de stockage doit utiliser une connexion de liaison privée<br> - Les comptes de stockage doivent utiliser des règles de réseau virtuel pour restreindre l’accès au réseau<br> - Les modèles VM Image Builder doivent utiliser une liaison privée|
|                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

Liens connexes :

- [En savoir plus sur le Benchmark de sécurité Azure](../security/benchmarks/introduction.md)
- [En savoir plus sur Azure Database for MariaDB](../mariadb/overview.md)
- [En savoir plus sur Azure Database pour MySQL](../mysql/overview.md)
- [En savoir plus sur Azure Database pour PostgreSQL](../postgresql/overview.md)





## <a name="next-steps"></a>Étapes suivantes

Pour tous les changements récents du produit, consultez [Nouveautés d’Azure Security Center](release-notes.md).