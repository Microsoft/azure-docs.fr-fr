---
title: Qu’est-ce qu’Azure Database pour PostgreSQL ?
description: Fournit une vue d’ensemble du service de base de données relationnelle d’Azure Database pour PostgreSQL dans un contexte de serveur flexible.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: c3ea7930f41fe89538a817da032e993e534db9cd
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92491323"
---
# <a name="what-is-azure-database-for-postgresql"></a>Qu’est-ce qu’Azure Database pour PostgreSQL ?

Azure Database pour PostgreSQL est un service de base de données relationnelle du cloud Microsoft qui repose sur le moteur de base de données [PostgreSQL Community Edition](https://www.postgresql.org/) (disponible avec la licence GPLv2). Azure Database pour PostgreSQL offre :

- Une haute disponibilité intégrée.
- La protection des données à l’aide de sauvegardes automatiques et une restauration à un point dans le temps jusqu’à 35 jours.
- Maintenance automatisée du matériel sous-jacent, du système d’exploitation et du moteur de base de données pour assurer la sécurité et la mise à jour du service.
- Des performances prévisibles, grâce aux tarifs du paiement à l’utilisation ;
- Mise à l’échelle élastique en quelques secondes.
- Sécurité de niveau entreprise et conformité à la pointe pour protéger les données sensibles au repos et en mouvement.
- Surveillance et automatisation pour simplifier la gestion et la surveillance des déploiements à grande échelle.
- Expérience de support à la pointe du secteur.

 :::image type="content" source="./media/overview/overview-what-is-azure-postgres.png" alt-text="Base de données Azure pour PostgreSQL":::

Ces fonctionnalités ne nécessitent presque aucune administration, et toutes sont fournies sans coût supplémentaire. Elles vous permettent de vous concentrer sur le développement rapide de vos applications et d’accélérer leur mise sur le marché, plutôt que de consacrer du temps et des ressources à gérer des machines virtuelles et une infrastructure. Par ailleurs, vous pouvez continuer à développer votre application avec les outils open source et la plateforme de votre choix pour travailler avec la rapidité et l’efficacité exigées par votre activité, et ce sans avoir à acquérir de nouvelles compétences.

## <a name="deployment-models"></a>Modèles de déploiement

Trois modes de déploiement sont disponibles pour Azure Database pour PostgreSQL optimisé par PostgreSQL Community Edition :

- Serveur unique
- Serveur flexible (préversion)
- Hyperscale (Citus)

### <a name="azure-database-for-postgresql---single-server"></a>Azure Database pour PostgreSQL - Serveur unique

Un serveur unique Azure Database pour PostgreSQL est un service de base de données complètement managé avec des exigences minimales pour les personnalisations de base de données. La plateforme de serveur unique est conçue pour prendre en charge la plupart des fonctions de gestion de base de données, comme les correctifs, les sauvegardes, la haute disponibilité et la sécurité, avec un minimum de configuration et de contrôle de la part de l’utilisateur. L’architecture est optimisée à des fins de haute disponibilité intégrée avec une disponibilité de 99,99 % sur une seule zone de disponibilité. Elle prend en charge la version de la communauté de PostgreSQL 9.5, 9.6, 10 et 11. Le service est aujourd'hui généralement disponible dans un grand nombre de [régions Azure](https://azure.microsoft.com/global-infrastructure/services/).

L’option de déploiement Serveur unique offre trois niveaux tarifaires : De base, Usage général et À mémoire optimisée. Chaque niveau offre des fonctionnalités de ressources différentes pour prendre en charge vos charges de travail de base de données. Vous pouvez créer votre première application sur une petite base de données pour un faible coût mensuel, puis adapter l’échelle aux besoins de votre solution. L’évolutivité dynamique permet de répondre en toute transparence à l’évolution rapide des besoins en ressources de votre base de données. Vous payez uniquement pour les ressources dont vous avez besoin et seulement quand vous en avez besoin. Pour plus d’informations, consultez [Niveaux tarifaires](./concepts-pricing-tiers.md).

Les serveurs uniques sont particulièrement adaptés aux applications natives Cloud conçues pour gérer les correctifs automatisés sans qu’il soit nécessaire d’exercer un contrôle précis sur le calendrier des correctifs et les paramètres de configuration PostgreSQL personnalisés.

Pour une présentation détaillée du mode de déploiement Serveur unique, consultez [Vue d’ensemble du mode Serveur unique](./overview-single-server.md).

### <a name="azure-database-for-postgresql---flexible-server-preview"></a>Azure Database pour PostgreSQL - Serveur flexible (préversion)

Un serveur flexible Azure Database pour PostgreSQL est un service de base de données entièrement géré conçu pour offrir un contrôle et une flexibilité plus granulaires des fonctions de gestion de base de données et des paramètres de configuration. En général, le service fournit des personnalisations en termes de flexibilité suivant les besoins de l’utilisateur. L’architecture de serveur flexible permet aux utilisateurs d’opter pour une haute disponibilité au sein d’une même zone de disponibilité et dans plusieurs zones de disponibilité. Les serveurs flexibles offrent de meilleurs contrôles d’optimisation des coûts grâce à la possibilité de d’arrêter/de démarrer le serveur et un niveau de calcul expansible, ce qui est idéal pour les charges de travail ne nécessitant pas en permanence une capacité de calcul complète. Le service prend actuellement en charge la version de la communauté de PostgreSQL 11 et 12, et prévoit d’ajouter prochainement des versions plus récentes. Le service, en préversion publique, est aujourd'hui généralement disponible dans un grand nombre de régions Azure.

Les serveurs flexibles sont idéalement adaptés pour ce qui suit :

- Développement d’applications nécessitant un meilleur contrôle et des personnalisations.
- Contrôles d’optimisation des coûts avec possibilité d’arrêter/de démarrer le serveur.
- Haute disponibilité redondante interzone.
- Fenêtres de maintenance managées.
  
Pour une présentation détaillée du mode de déploiement Serveur flexible, consultez [Vue d'ensemble du mode Serveur flexible](./flexible-server/overview.md).

### <a name="azure-database-for-postgresql--hyperscale-citus"></a>Azure Database pour PostgreSQL - Hyperscale (Citus)

L’option Hyperscale (Citus) met horizontalement à l’échelle les requêtes sur plusieurs machines à l’aide du partitionnement. Son moteur de requête parallélise les requêtes SQL entrantes sur ces serveurs afin de générer des réponses plus rapides sur les jeux de données volumineux. Elle s’adresse aux applications qui nécessitent des performances et une mise à l’échelle plus grandes, généralement des charges de travail qui approchent, ou dépassent déjà, 100 Go de données.

L’option de déploiement Hyperscale (Citus) offre les fonctionnalités suivantes :

- Mise à l’échelle horizontale sur plusieurs machines à l’aide du partitionnement
- Parallélisation des requêtes sur ces serveurs pour générer des réponses plus rapides sur les jeux de données volumineux
- Excellente prise en charge des applications multilocataires, analytique opérationnelle en temps réel et charges de travail transactionnelles à débit élevé
  
Les applications générées pour PostgreSQL peuvent exécuter des requêtes distribuées sur Hyperscale (Citus) avec des [bibliothèques de connexions](./concepts-connection-libraries.md) standard et des modifications minimes.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les trois modes de déploiement pour Azure Database pour PostgreSQL et choisir les options appropriées en fonction de vos besoins.

- [Serveur unique](./overview-single-server.md)
- [Serveur flexible](./flexible-server/overview.md)
- Hyperscale (Citus)