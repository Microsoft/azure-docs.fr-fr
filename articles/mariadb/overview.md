---
title: Vue d’ensemble – Azure Database for MariaDB
description: Découvrez le service Azure Database for MariaDB, service de base de données relationnelle dans le cloud Microsoft qui repose sur MySQL Community Edition.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: overview
ms.custom: mvc
ms.date: 3/18/2020
ms.openlocfilehash: 8115625099543d378728a6313a8cc4c95fec0cd2
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536997"
---
# <a name="what-is-azure-database-for-mariadb"></a>Qu’est-ce qu’Azure Database for MariaDB ?

Azure Database for MariaDB est un service de base de données relationnelle dans Microsoft Cloud. Azure Database for MariaDB est basé sur le moteur de base de données [MariaDB Community Edition](https://mariadb.org/download/) (disponible dans le cadre de la licence GPLv2), version 10.2 et 10.3.

Azure Database for MariaDB offre les fonctionnalités suivantes :

- Une haute disponibilité intégrée sans coût supplémentaire ;
- Des performances prévisibles, grâce aux tarifs du paiement à l’utilisation ;
- Mise à l’échelle selon les besoins en quelques secondes
- Protection sécurisée des données sensibles au repos et en mouvement
- Des sauvegardes automatiques et une restauration à un point dans le temps jusqu’à 35 jours ;
- Une sécurité et une conformité de classe Entreprise.

Ces fonctionnalités ne nécessitent presque aucune administration. Elles sont fournies sans coût supplémentaire. Azure Database for MariaDB peut vous aider à rapidement développer votre application et à accélérer le délai de mise sur le marché. Vous n’êtes pas obligé d’allouer un temps précieux et des ressources à la gestion des machines virtuelles et de l’infrastructure. Vous pouvez également continuer à développer votre application en utilisant les outils open source et la plateforme de votre choix. Répondez aux besoins de vitesse et d’efficacité exigés par votre activité, sans apprendre de nouvelles compétences.

Pour en savoir plus sur les principaux concepts et fonctionnalités dans Azure Database for MariaDB, y compris les performances, l’évolutivité et la facilité de gestion, consultez ces guides de démarrage rapide :

- [Créer un serveur Azure Database for MariaDB à l’aide du portail Azure](quickstart-create-mariadb-server-database-using-azure-portal.md)
- [Créer un serveur Azure Database for MariaDB à l’aide d’Azure CLI](quickstart-create-mariadb-server-database-using-azure-cli.md)

<!--
For a set of Azure CLI samples, see:
- [Azure CLI samples for Azure Database for MariaDB](sample-scripts-azure-cli.md) 
-->

## <a name="adjust-performance-and-scale-within-seconds"></a>Ajustez les performances et la mise à l’échelle en quelques secondes

Le service Azure Database for MariaDB offre plusieurs niveaux de service : De base, Usage général et À mémoire optimisée. Chaque niveau offre différentes performances et fonctionnalités pour prendre en charge des charges de travail de base de données plus ou moins denses. Vous pouvez créer votre première application sur une petite base de données pour un faible coût mensuel, puis adapter l’échelle aux besoins de votre solution. L’évolutivité dynamique aide votre base de données à répondre en toute transparence à l’évolution rapide des besoins en ressources. Vous payez uniquement pour les ressources dont vous avez besoin et seulement quand vous en avez besoin. Pour plus d’informations, consultez [Niveaux tarifaires](concepts-pricing-tiers.md).

## <a name="monitoring-and-alerting"></a>Surveillance et alerte

Comment savoir quand monter ou baisser la puissance des instances ? Vous pouvez utiliser les fonctionnalités intégrées de surveillance et d’alerte de performances d’Azure Database for MariaDB, combinées avec les évaluations de performance basées sur des vCores. Ces outils vous permettent d’évaluer rapidement les effets des mises à l’échelle des vCores (montées ou descentes en charge) en fonction de vos besoins en performances actuels ou pour un projet. Pour plus d’informations, consultez [Alertes](howto-alert-metric.md).

## <a name="keep-your-app-and-business-running"></a>Votre application et votre activité ne s’arrêtent jamais

Avec un temps de disponibilité de 99,99 %, le SLA (contrat de niveau de service) d’Azure est assuré par un réseau mondial de centres de données gérés par Microsoft. Le réseau permet d’exécuter votre application 24 heures sur 24 et 7 jours sur 7. Vous profitez de la sécurité intégrée, de la tolérance de panne et de la protection des données dans Azure Database for MariaDB. Avec Azure Database for MariaDB, vous pouvez utiliser la limite de restauration dans le temps pour restaurer un serveur à un état antérieur pouvant remonter jusqu’à 35 jours.

## <a name="secure-your-data"></a>Sécurisez vos données

Azure Database for MariaDB respecte la tradition de sécurité des données qu’offrent les services de base de données Azure. Azure Database for MariaDB offre des fonctionnalités qui limitent l’accès, protègent les données au repos et en mouvement et vous aident à surveiller l’activité. Visitez le [Centre de confidentialité Azure](https://www.microsoft.com/trustcenter/security) pour plus d’informations sur la sécurité de la plateforme Azure. Pour plus d’informations sur les fonctionnalités de sécurité d’Azure Database for MariaDB, consultez la [vue d’ensemble de sécurité](concepts-security.md).

## <a name="contacts"></a>Contacts

Vous pouvez envoyer toutes vos questions ou suggestions au sujet de l’utilisation d’Azure Database for MariaDB à l’[équipe Azure Database for MariaDB](mailto:AskAzureDBforMariaDB@service.microsoft.com) (n’est pas un alias pour l’équipe du support technique).

Vous pouvez également utiliser les points de contact suivants :
- Pour contacter le support Azure, [ouvrez une demande de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) dans le portail Azure.
- Pour résoudre un problème relatif à votre compte, [ouvrez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) sur le portail Azure.
- Pour donner votre avis ou demander de nouvelles fonctionnalités, créez une entrée via le [Forum de commentaires Azure](https://feedback.azure.com/forums/915439-azure-database-for-mariadb).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez lu une introduction à Azure Database for MariaDB, vous êtes prêt à :
- Consulter la page de [tarification](https://azure.microsoft.com/pricing/details/mariadb/) pour des comparaisons de coûts et des calculatrices. 
- Commencer par [créer votre premier serveur](quickstart-create-mariadb-server-database-using-azure-portal.md).

<!--- - Build your first app using your preferred language: [Python](./connect-python.md) | [Node.JS](./connect-nodejs.md) | [Java](./connect-java.md) | [Ruby](./connect-ruby.md) | [PHP](./connect-php.md) | [.NET (C#)](./connect-csharp.md) | [Go](./connect-go.md) --->
