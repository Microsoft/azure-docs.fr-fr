---
title: Vue d’ensemble de la gestion Azure - Gouvernance Azure
description: Vue d’ensemble des domaines de gestion des applications et ressources Azure avec des liens vers du contenu sur les outils de gestion Azure.
ms.date: 09/09/2020
ms.topic: overview
ms.openlocfilehash: 8ffb37cd1db5740ed022b52cd9687bb3db1632bb
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91949494"
---
# <a name="what-are-the-azure-management-areas"></a>Présentation des domaines de gestion Azure

La gouvernance dans Azure est un aspect de la gestion Azure. Cet article décrit les différents domaines de gestion pour déployer et maintenir vos ressources dans Azure.

La gestion fait référence aux tâches et processus nécessaires pour maintenir vos applications métier et les ressources qui les prennent en charge. Azure a de nombreux services et outils qui fonctionnent ensemble pour offrir une gestion complète. Ces services ne sont pas uniquement destinés aux ressources dans Azure, mais également dans d’autres clouds et localement. La première étape pour concevoir un environnement de gestion complet est de bien comprendre les différents outils et comment ils fonctionnent ensemble.

Le schéma ci-dessous illustre les différents domaines de gestion requis pour maintenir toute application ou ressource. Ces différents domaines peuvent être considérés comme un cycle de vie. Chaque domaine est obligatoire successivement sur la durée de vie d’une ressource. Le cycle de vie d’une ressource commence par son déploiement initial, puis son exploitation dans la durée, et se termine par sa mise hors service.

:::image type="complex" source="../monitoring/media/management-overview/management-capabilities.png" alt-text="Diagramme des disciplines de gestion dans Azure." border="false":::
   Diagramme illustrant les éléments Migrer, Sécuriser, Protéger, Superviser, Configurer et Gouverner de la roue des services qui assurent la gestion et la gouvernance dans Azure. L’élément Sécuriser comporte les sous-éléments Gestion de la sécurité et Protection contre les menaces. L’élément Protéger comporte les sous-éléments Sauvegarde et Reprise d’activité. L’élément Superviser comporte les sous-éléments Supervision des applications, infrastructures et réseaux et Log Analytics et diagnostics. L’élément Configurer comporte les sous-éléments Configuration, Gestion des mises à jour, Automatisation et Scripts. Enfin, l’élément Gouverner comporte les sous-éléments Gestion des stratégies et Gestion des coûts.
:::image-end:::

Aucun service Azure ne répond complètement aux exigences d’un domaine de gestion en particulier. Au lieu de cela, chacune est réalisée par plusieurs services fonctionnant ensemble. Certains services, comme Application Insights, fournissent une fonctionnalité de supervision ciblée pour les applications web. D’autres, comme les journaux d’activité Azure Monitor, stockent les données de gestion pour d’autres services. Cette fonctionnalité vous permet d’analyser des données de différents types collectées par différents services.

Les sections suivantes décrivent en bref les différents domaines de gestion et contiennent des liens vers du contenu détaillé sur les principaux services Azure destinés à répondre à ces besoins.

## <a name="monitor"></a>Superviser

La supervision consiste à collecter et analyser des données afin de déterminer les performances, l’intégrité et la disponibilité de vos ressources. Une stratégie de supervision efficace vous permet de comprendre le fonctionnement des composants et d’augmenter la durée de fonctionnement avec des notifications. Lisez une vue d’ensemble de la supervision qui décrit les différents services utilisés dans la [Supervision des applications et des ressources Azure](../azure-monitor/overview.md).

## <a name="configure"></a>Configurer

La configuration fait référence au déploiement initial et à la configuration des ressources ainsi qu’à leur maintenance continue.
L’automatisation de ces tâches vous permet d’éviter la redondance, en réduisant le temps et les efforts et en augmentant votre précision et votre efficacité. [Azure Automation](../automation/automation-intro.md) fournit la majeure partie des services d’automatisation des tâches de configuration. Alors que les runbooks gèrent l’automatisation des processus, la gestion de la configuration et des mises à jour vous aide à gérer la configuration.

## <a name="govern"></a>Gouvernance

La gouvernance propose des mécanismes et des processus pour garder le contrôle sur vos applications et ressources dans Azure. Elle implique la planification de vos initiatives et la définition de priorités stratégiques.
La gouvernance dans Azure est principalement mise en œuvre à l’aide de deux services. [Azure Policy](./policy/overview.md) vous permet de créer, d’attribuer et de gérer des définitions de stratégie afin d’appliquer des règles pour vos ressources.
Cette fonctionnalité maintient ces ressources conformes aux standards de votre entreprise.
[Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md) vous permet de suivre l’utilisation du Cloud et les dépenses liées à vos ressources Azure et à d’autres fournisseurs de Cloud.

## <a name="secure"></a>Sécuriser

Gérez la sécurité de vos ressources et de vos données. Un programme de sécurité implique l’évaluation des menaces, la collecte et l’analyse des données, et la conformité de vos applications et ressources. [Azure Security Center](../security-center/security-center-introduction.md) assure la supervision de la sécurité et l’analyse des menaces, notamment par des fonctionnalités unifiées de gestion de la sécurité et de protection avancée contre les menaces sur l’ensemble des charges de travail cloud hybrides. Consultez [Présentation de la sécurité Azure](../security/fundamentals/overview.md) pour des informations complètes et des conseils sur la sécurisation des ressources Azure.

## <a name="protect"></a>Protéger

La protection consiste à garder vos applications et données disponibles, même en cas de pannes hors de votre contrôle. La protection dans Azure est fournie par deux services. [Sauvegarde Azure](../backup/backup-overview.md) permet la sauvegarde et la récupération de vos données, dans le cloud ou en local. [Azure Site Recovery](../site-recovery/site-recovery-overview.md) fournit une continuité d’activité et une reprise d’activité immédiate après sinistre.

## <a name="migrate"></a>Migrer

La migration fait référence à la transition des charges de travail exécutées actuellement en local vers le cloud Azure.
[Azure Migrate](../migrate/migrate-services-overview.md) est un service qui vous aide à évaluer la pertinence d’une migration de machines virtuelles locales vers Azure. Azure Site Recovery migre des machines virtuelles [locales](../site-recovery/migrate-tutorial-on-premises-azure.md) ou [d’Amazon Web Services](../site-recovery/migrate-tutorial-aws-azure.md). [Azure Database Migration](../dms/dms-overview.md) vous aide à migrer des sources de base de données vers des plateformes de données Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la Gouvernance Azure, consultez les articles suivants :

- [Hub de la Gouvernance Azure](./index.yml).
- [Gouvernance dans le Framework d’adoption du cloud pour Azure](/azure/cloud-adoption-framework/govern/)