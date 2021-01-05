---
title: Vue d’ensemble – Azure Database pour MySQL
description: Découvrez le service Azure Database pour MySQL, service de base de données relationnelle dans le cloud Microsoft qui repose sur MySQL Community Edition.
author: savjani
ms.service: mysql
ms.author: pariks
ms.custom: mvc
ms.topic: overview
ms.date: 3/18/2020
ms.openlocfilehash: 51c194ca9b091bc685f293320750da55925ad49d
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91565613"
---
# <a name="what-is-azure-database-for-mysql"></a>Qu’est-ce qu’Azure Database pour MySQL ?

Azure Database pour MySQL est un service de base de données relationnelle du cloud Microsoft qui repose sur le moteur de base de données [MySQL Community Edition](https://www.mysql.com/products/community/) versions 5.6, 5.7 et 8.0 (disponible avec la licence GPLv2). Azure Database pour MySQL offre :

- Une haute disponibilité intégrée.
- La protection des données à l’aide de sauvegardes automatiques et une restauration à un point dans le temps jusqu’à 35 jours.
- Maintenance automatisée du matériel sous-jacent, du système d’exploitation et du moteur de base de données pour assurer la sécurité et la mise à jour du service.
- Des performances prévisibles, grâce aux tarifs du paiement à l’utilisation ;
- Mise à l’échelle élastique en quelques secondes.
- Contrôles d’optimisation des coûts avec possibilité d’arrêter/de démarrer le serveur. 
- Sécurité de niveau entreprise et conformité à la pointe pour protéger les données sensibles au repos et en mouvement.
- Surveillance et automatisation pour simplifier la gestion et la surveillance des déploiements à grande échelle.
- Expérience de support à la pointe du secteur.

Ces fonctionnalités ne demandent pratiquement aucune administration, et toutes sont fournies sans coût supplémentaire. Elles vous permettent de vous concentrer sur le développement rapide de vos applications et d’accélérer leur mise sur le marché, plutôt que de consacrer du temps et des ressources à gérer des machines virtuelles et une infrastructure. Par ailleurs, vous pouvez continuer à développer votre application avec les outils open source et la plateforme de votre choix pour travailler avec la rapidité et l’efficacité exigées par votre activité, et ce sans avoir à acquérir de nouvelles compétences.

:::image type="content" source="media/overview/1-azure-db-for-mysql-conceptual-diagram.png" alt-text="Diagramme conceptuel d’Azure Database pour MySQL":::

## <a name="deployment-models"></a>Modèles de déploiement

Deux modes de déploiement sont disponibles pour Azure Database pour MySQL optimisé par MySQL Community Edition :
- Serveur unique 
- Serveur flexible (préversion)
  
### <a name="azure-database-for-mysql---single-server"></a>Serveur unique Azure Database pour MySQL

Un serveur unique Azure Database pour MySQL est un service de base de données complètement managé avec des exigences minimales pour les personnalisations de base de données. La plateforme de serveur unique est conçue pour prendre en charge la plupart des fonctions de gestion de base de données, comme les correctifs, les sauvegardes, la haute disponibilité et la sécurité, avec un minimum de configuration et de contrôle de la part de l’utilisateur. L’architecture est optimisée à des fins de haute disponibilité intégrée avec une disponibilité de 99,99 % sur une seule zone de disponibilité. Elle prend en charge la version de la communauté de MySQL 5.6, 5.7 et 8.0. Le service est aujourd'hui généralement disponible dans un grand nombre de [régions Azure](https://azure.microsoft.com/global-infrastructure/services/).

L’option de déploiement Serveur unique offre trois niveaux tarifaires : De base, Usage général et À mémoire optimisée. Chaque niveau offre des fonctionnalités de ressources différentes pour prendre en charge vos charges de travail de base de données. Vous pouvez créer votre première application sur une petite base de données pour un faible coût mensuel, puis adapter l’échelle aux besoins de votre solution. L’évolutivité dynamique permet de répondre en toute transparence à l’évolution rapide des besoins en ressources de votre base de données. Vous payez uniquement pour les ressources dont vous avez besoin et seulement quand vous en avez besoin. Pour plus d’informations, consultez [Niveaux tarifaires](concepts-pricing-tiers.md).

Les serveurs uniques sont particulièrement adaptés aux applications natives Cloud conçues pour gérer les correctifs automatisés sans qu’il soit nécessaire d’exercer un contrôle précis sur le calendrier des correctifs et les paramètres de configuration MySQL personnalisés. 

Pour une présentation détaillée du mode de déploiement Serveur unique, consultez [Vue d’ensemble du mode Serveur unique](single-server-overview.md).

### <a name="azure-database-for-mysql---flexible-server-preview"></a>Azure Database pour MySQL - Serveur flexible (préversion)

Azure Database pour MySQL - Serveur flexible est un service de base de données entièrement géré conçu pour offrir un contrôle et une flexibilité plus granulaires des fonctions de gestion de base de données et des paramètres de configuration. En général, le service fournit des personnalisations en termes de flexibilité suivant les besoins de l’utilisateur. L’architecture de serveur flexible permet aux utilisateurs d’opter pour une haute disponibilité au sein d’une même zone de disponibilité et dans plusieurs zones de disponibilité. Les serveurs flexibles offrent de meilleurs contrôles d’optimisation des coûts grâce à la possibilité de d’arrêter/de démarrer le serveur et un niveau de calcul expansible, ce qui est idéal pour les charges de travail ne nécessitant pas en permanence une capacité de calcul complète. Le service prend actuellement en charge la version de la communauté de MySQL 5.7 et prévoit d’ajouter prochainement des versions plus récentes. Le service, en préversion publique, est aujourd'hui généralement disponible dans un grand nombre de [régions Azure](https://azure.microsoft.com/global-infrastructure/services/).

Les serveurs flexibles sont adaptés de façon optimale pour ce qui suit : 
- Développement d’applications nécessitant un meilleur contrôle et des personnalisations.
- Haute disponibilité redondante interzone.
- Fenêtres de maintenance managées.

Pour une présentation détaillée du mode de déploiement Serveur flexible, consultez [Vue d'ensemble du mode Serveur flexible](flexible-server/overview.md).

## <a name="contacts"></a>Contacts
Pour toute question ou suggestion au sujet de l’utilisation d’Azure Database pour MySQL, envoyez un e-mail à l’équipe Azure Database pour MySQL ([@Ask Azure DB pour MySQL](mailto:AskAzureDBforMySQL@service.microsoft.com)). Cette adresse e-mail n’est pas un alias de support technique.

En outre, tenez compte des points de contact suivants le cas échéant :

- Pour contacter le support technique Azure, [émettez un ticket à partir du Portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Pour résoudre un problème relatif à votre compte, enregistrez une [demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) sur le portail Azure.
- Pour donner votre avis ou demander de nouvelles fonctionnalités, créez une entrée via [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les deux modes de déploiement pour Azure Database pour MySQL et choisir les options appropriées en fonction de vos besoins.

- [Serveur unique](single-server/index.yml)
- [Serveur flexible](flexible-server/index.yml)
- [Choisir l’option de déploiement MySQL appropriée pour votre charge de travail](select-right-deployment-type.md)
