---
title: Choisir l’option de serveur PostgreSQL dans Azure
description: Fournit des instructions permettant de choisir l’option de serveur PostgreSQL pour vos déploiements.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: 8bc303f619d145cc280e6caab65781bd42d1b314
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489691"
---
# <a name="choose-the-right-postgresql-server-option-in-azure"></a>Choisir l’option de serveur PostgreSQL dans Azure

Avec Azure, vos charges de travail de serveur PostgreSQL peuvent s’exécuter dans une infrastructure en tant que service (IaaS) de machine virtuelle hébergée ou dans une plateforme en tant que service (PaaS) hébergée. PaaS propose différentes options de déploiement, chacune dotée de plusieurs niveaux de service. Quand vous choisissez entre IaaS et PaaS, vous devez déterminer si vous voulez gérer votre base de données, appliquer des correctifs et effectuer des sauvegardes, ou plutôt déléguer ces opérations à Azure.

Au moment de prendre votre décision, envisagez les trois options suivantes dans PaaS ou une exécution sur des machines virtuelles Azure (IaaS)
- [Azure Database pour PostgreSQL - Serveur unique](./overview-single-server.md)
- [Azure Database pour PostgreSQL - Serveur flexible](./flexible-server/overview.md)
- [Azure Database pour PostgreSQL - Hyperscale (Citus)]()

L’option **PostgreSQL sur machines virtuelles Azure** s’inscrit dans la catégorie de secteur IaaS. Avec ce service, vous pouvez exécuter le serveur PostgreSQL sur une machine virtuelle entièrement managée dans le cadre de la plateforme cloud Azure. Toutes les versions et éditions récentes de PostgreSQL peuvent être installées sur une machine virtuelle IaaS. La différence la plus significative par rapport à Azure Database pour PostgreSQL tient au fait que MySQL sur des machines virtuelles Azure offre le contrôle du moteur de base de données. Toutefois, ce contrôle est fourni au détriment de la responsabilité de gestion des machines virtuelles et de nombreuses tâches d’administration de base de données. Ces tâches incluent la maintenance et la mise à jour corrective des serveurs de base de données, la récupération des bases de données et une conception à haute disponibilité.

Le tableau suivant liste les principales différences entre ces options :

| **Attribut** | **Postgres sur machines virtuelles Azure** | **PostgreSQL en tant que PaaS** |
| ----- | ----- | ----- |
| <B> Contrat SLA de disponibilité |- 99,99 % avec les groupes à haute disponibilité <br> - 99,95 % avec les machines virtuelles uniques | - Serveur unique – 99,99 % <br> - Serveur flexible - Non disponible dans le cadre de la préversion <br> - Hyperscale (Citus) - 99,95 % (lorsque la haute disponibilité est activée)|
| <B> Mise à jour corrective du système d’exploitation et PostgreSQL | - Gérée par le client | - Serveur unique - Automatique <br> - Serveur flexible - Automatique avec fenêtre gérée par le client en option <br> - Hyperscale (Citus) - Automatique |
| <B> Haute disponibilité | - Les clients conçoivent, implémentent, testent et maintiennent la haute disponibilité. Les fonctionnalités peuvent inclure le clustering, la réplication, etc. | - Serveur unique : intégré <br> - Serveur flexible : intégré <br> - Hyperscale (Citus) : avec mode standby |
| <B>Redondance de zone | - Les machines virtuelles Azure peuvent être configurées pour s’exécuter dans différentes zones de disponibilité. Pour une solution locale, les clients doivent créer et gérer leur propre centre de données secondaire. | - Serveur unique : Non <br> - Serveur flexible : Oui <br> - Hyperscale (Citus) : Non |
| <B> Scénario hybride | - Géré par le client |- Serveur unique : Lecture-réplica <br> - Serveur flexible : Non disponible dans la cadre de la préversion <br> - Hyperscale (Citus) : Non |
| <B> Sauvegarde et restauration | - Gérées par le client | - Serveur unique : intégré avec la configuration utilisateur pour les paramètres locaux et géographiques <br> - Serveur flexible : intégré avec la configuration utilisateur sur le stockage redondant interzone <br> - Hyperscale (Citus) : intégré |
| <B> Supervision des opérations de base de données | - Gérées par le client | - Serveur unique, serveur flexible et Hyperscale (Citus) : Offre aux clients la possibilité de définir des alertes sur l’opération de base de données et d’agir dès qu’un seuil est atteint. |
| <B>Protection avancée contre les menaces | - Les clients doivent établir cette protection pour se prémunir eux-mêmes. |- Serveur unique : Oui <br> - Serveur flexible : Non disponible dans la cadre de la préversion <br> - Hyperscale (Citus) : Non |
| <B> Récupération d’urgence | - Gérée par le client | - Serveur unique : Sauvegarde géoredondante et lecture réplica géo <br> - Serveur flexible : Non disponible dans la cadre de la préversion <br> - Hyperscale (Citus) : Non |
| <B> Performances intelligentes | - Gérées par le client | - Serveur unique : Oui <br> - Serveur flexible : Non disponible dans la cadre de la préversion <br> - Hyperscale (Citus) : Non |

## <a name="total-cost-of-ownership-tco"></a>Coût total de possession (TCO)

Le coût total de possession est souvent le facteur principal qui détermine la meilleure solution d’hébergement de vos bases de données. C’est vrai que vous soyez une start-up à court de liquidités ou une équipe dans une société établie qui subit de fortes contraintes budgétaires. Cette section décrit les principes de base de facturation et de licence dans Azure qui s’appliquent à Azure Database pour PostgreSQL et à PostgreSQL sur des machines virtuelles Azure.

## <a name="billing"></a>Facturation

Azure Database pour PostgreSQL est disponible en tant que service dans plusieurs niveaux avec différents prix pour les ressources. Toutes les ressources sont facturées à un tarif horaire fixe. Pour obtenir les dernières informations sur les niveaux de service, les tailles de calcul et les quantités de stockage actuellement pris en charge, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/postgresql/server/) Vous pouvez ajuster dynamiquement les niveaux de service et les tailles de calcul en fonction des besoins de débit variés de votre application. Vous êtes facturé pour le trafic internet sortant aux [tarifs de transfert de données](https://azure.microsoft.com/pricing/details/data-transfers/) standard.

Avec Azure Database pour PostgreSQL, Microsoft configure, corrige et met à niveau automatiquement le logiciel de base de données. Ces actions automatisées réduisent vos coûts d’administration. En outre, Azure Database pour PostgreSQL offre des fonctionnalités de [sauvegardes automatisées](). Ces fonctionnalités vous permettent de réaliser d’importantes économies, notamment si vous avez un grand nombre de bases de données. Au contraire, avec PostgreSQL sur des machines virtuelles Azure, vous pouvez choisir et exécuter n’importe quelle version de PostgreSQL. Toutefois, vous devez payer pour la machine virtuelle approvisionnée, le coût de stockage associé aux données, la sauvegarde, la surveillance des données et le stockage des journaux, ainsi que les coûts associés au type de licence PostgreSQL spécifique utilisé (le cas échéant).

Azure Database pour PostgreSQL offre une haute disponibilité intégrée pour tout type d’interruption de niveau nœud tout en conservant le niveau SLA garanti de 99,99 % pour le service. Toutefois, pour la haute disponibilité de base de données au sein de machines virtuelles, vous devez utiliser des options de haute disponibilité telles que la [réplication en streaming](https://www.postgresql.org/docs/12/warm-standby.html#STREAMING-REPLICATION) disponibles sur une base de données PostgreSQL. L’utilisation d’une option de haute disponibilité prise en charge ne fournit pas de SLA supplémentaire. Toutefois, pour des coûts supplémentaires et une plus grande charge administrative, elle vous permet d’atteindre une disponibilité de base de données supérieure à 99,99 %.

Pour plus d’informations sur les tarifs, consultez les articles suivants :
- [Tarifs d'Azure Database pour PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/)
- [Tarifs des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/)
- [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/)

## <a name="administration"></a>Administration

Pour de nombreuses entreprises, la décision de migrer vers un service cloud vise autant à simplifier l’administration qu’à réduire son coût.

Avec IaaS, Microsoft :

- Administre l’infrastructure sous-jacente.
- Fournit une mise à jour corrective automatisée pour le matériel et le système d’exploitation sous-jacents.

Avec PaaS, Microsoft :

- Administre l’infrastructure sous-jacente.
- Fournit une mise à jour corrective automatisée pour le matériel, le système d’exploitation et le moteur de base de données sous-jacents.
- Gère la haute disponibilité de la base de données.
- Effectue automatiquement des sauvegardes et réplique toutes les données pour assurer la récupération d’urgence.
- Chiffre les données au repos et en mouvement par défaut.
- Analyse votre serveur et fournit des fonctionnalités pour obtenir des informations sur les performances des requêtes et des recommandations en matière de performances.

Avec Azure Database pour PostgreSQL, vous pouvez continuer à administrer votre base de données. Mais vous n’avez plus besoin de gérer le moteur de base de données, le système d’exploitation ni le matériel. Voici quelques exemples d’éléments que vous pouvez continuer à administrer :

- Bases de données
- Connexion
- Réglage des index
- Paramétrage des requêtes
- Audit
- Sécurité

Par ailleurs, la configuration de la haute disponibilité vers un autre centre de données requiert une configuration ou une administration minimale ou nulle.

- Avec PostgreSQL sur des machines virtuelles Azure, vous disposez d’un contrôle total sur le système d’exploitation et la configuration des instances de serveur PostgreSQL. Avec une machine virtuelle, vous décidez quand mettre à jour ou à niveau le système d’exploitation et le logiciel de base de données, ainsi que les correctifs à appliquer. Vous décidez également quand installer des logiciels supplémentaires tels qu’une application antivirus. Certaines fonctionnalités automatisées simplifient considérablement la gestion des correctifs, la sauvegarde et la haute disponibilité. Vous pouvez contrôler la taille de la machine virtuelle, le nombre de disques et leurs configurations de stockage. Pour plus d’informations, consultez [Tailles des machines virtuelles et des services cloud pour Azure](../virtual-machines/sizes.md).

## <a name="time-to-move-to-azure-postgresql-service-paas"></a>Il est temps de migrer vers Azure PostgreSQL Service (PaaS)

- Azure Database pour PostgreSQL est la solution idéale pour des applications cloud quand la productivité des développeurs et la rapidité de mise sur le marché de nouvelles solutions sont essentielles. Avec des fonctionnalités de programmation similaires à DBA, le service est adapté pour les architectes et les développeurs cloud, car il tempère la nécessité de gérer le système d’exploitation et la base de données sous-jacents.

- Lorsque vous souhaitez éviter le temps et les frais liés à l’acquisition d’un nouveau matériel local, PostgreSQL sur machines virtuelles Azure est la solution adaptée aux applications qui nécessitent un contrôle granulaire et une personnalisation du moteur PostgreSQL, non pris en charge par le service ou nécessitant un accès du système d’exploitation sous-jacent.

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Tarifs d'Azure Database pour PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).
- Commencez par créer votre premier serveur.