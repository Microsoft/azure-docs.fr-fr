---
title: Créer un plan de migration avec Azure Migrate
description: Fournit des conseils sur la création d’un plan de migration avec Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 11/11/2020
ms.openlocfilehash: ef916e0e8b32c96382a731d4a307e2b2a98ba1ea
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753856"
---
# <a name="build-migration-plan-with-azure-migrate"></a>Créer un plan de migration avec Azure Migrate

Suivez cet article pour créer votre plan de migration vers Azure avec [Azure Migrate](migrate-services-overview.md). 

## <a name="define-cloud-migration-goals"></a>Définir des objectifs de migration cloud

Avant de commencer, la compréhension et l’évaluation de votre [motivation](/azure/cloud-adoption-framework/strategy/motivations) pour passer au cloud peut contribuer à la réussite de votre entreprise. Comme expliqué dans [Cloud Adoption Framework](/azure/cloud-adoption-framework), il existe plusieurs déclencheurs et résultats.   

**Événement commercial** | **Résultat de la migration**
--- | ---
Sortie du centre de données | Coût 
Fusion, acquisition ou cession | Réduction de la complexité liée aux fournisseurs/contraintes techniques
Réduction des dépenses d’investissement | Optimisation des opérations internes
Fin de la prise en charge de technologies stratégiques | Augmentation de l’agilité de l’entreprise
Réponse aux changements liés à la conformité réglementaire | Préparation à de nouvelles capacités techniques
Nouvelles exigences de souveraineté des données | Adaptation à de nouvelles demandes du marché
Réduction des interruptions et améliorations de la stabilité informatique | Adaptation à de nouvelles demandes géographiques

L’identification de votre motivation vous permet de déterminer vos objectifs de migration stratégiques. L’étape suivante consiste à identifier et à planifier un chemin de migration, adapté à vos charges de travail. L’outil [Azure Migrate Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool) vous aide à évaluer les charges de travail locales, et il fournit des conseils et des outils pour vous aider à effectuer la migration.

## <a name="understand-your-digital-estate"></a>Comprendre votre patrimoine numérique

Commencez par identifier votre infrastructure locale, vos applications et vos dépendances. Cela vous permet d’identifier des charges de travail pour la migration vers Azure et de constituer des projections des coûts optimisées. L’outil Server Assessment vous aide à identifier les charges de travail en cours d’utilisation, les dépendances entre elles et leur optimisation.

### <a name="workloads-in-use"></a>Charges de travail en cours d’utilisation

Azure Migrate utilise une appliance Azure Migrate légère pour effectuer la détection sans agent des machines virtuelles VMware locales, des machines virtuelles Hyper-V, d’autres machines virtualisées et des serveurs physiques. La détection en continu permet de collecter les informations de configuration des machines, les métadonnées de performances, ainsi que les données d’application. Voici ce que l’appliance collecte auprès des machines locales : 

- Métadonnées des machines, disques et cartes réseau.

- Applications, rôles et fonctionnalités installés.

- Données de performances, notamment l’utilisation du processeur et de la mémoire, les IOPS de disque et le débit.

Après avoir collecté ces données, vous pouvez exporter la liste d’inventaire des applications pour rechercher des applications et des instances SQL Server en cours d’exécution sur vos ordinateurs. Vous pouvez utiliser l’outil Azure Migrate : Database Assessment pour comprendre la disponibilité SQL Server.

 ![Inventaire des applications sur le Portail](./media/concepts-migration-planning/application-inventory-portal.png)

 ![Exporter l’inventaire des applications](./media/concepts-migration-planning/application-inventory-export.png)

Outre les données détectées avec l’outil Server Assessment, vous pouvez utiliser vos données CMDB (base de données de gestion de la configuration) pour créer une vue d’ensemble de votre patrimoine de serveurs et bases de données, et pour comprendre comment vos serveurs sont distribués entre les unités commerciales, propriétaires d’applications, zones géographiques, etc. Ainsi, vous pouvez déterminer plus facilement les charges de travail à privilégier pour la migration. 

### <a name="dependencies-between-workloads"></a>Dépendances entre les charges de travail

À l’issue de la détection des serveurs, vous pouvez [analyser les dépendances](concepts-dependency-visualization.md) pour visualiser et identifier ces dépendances entre serveurs, ainsi que des stratégies d’optimisation dans le but de déplacer les serveurs interdépendants vers Azure. La visualisation permet de déterminer si certaines machines sont en cours d’utilisation ou si elles peuvent être mises hors service au lieu d’être migrées.  L’analyse des dépendances permet de s’assurer que rien n’est laissé de côté et d’éviter les pannes inopinées durant la migration. Une fois l’inventaire des applications et l’analyse des dépendances terminées, vous pouvez créer des groupes de serveurs fiables et commencer à les évaluer.

 ![Mappage des dépendances](./media/concepts-migration-planning/expand-client-group.png)

### <a name="optimization-and-sizing"></a>Optimisation et dimensionnement

Azure offre la flexibilité nécessaire pour redimensionner votre capacité cloud au fil du temps, et la migration vous permet d’optimiser les ressources processeur et mémoire allouées à vos serveurs. La création d’une évaluation sur les serveurs ainsi identifiés vous aide à comprendre l’historique des performances de vos charges de travail. Ce point est essentiel pour le bon dimensionnement des références SKU de machine virtuelle Azure et les recommandations de disque dans Azure.

## <a name="assess-migration-readiness"></a>Évaluer la préparation à la migration


### <a name="readinesssuitability-analysis"></a>Analyse de la préparation/pertinence

Vous pouvez exporter le rapport d’évaluation et filtrer ces catégories pour comprendre la préparation pour Azure :

- **Disponible pour Azure** : Les machines peuvent être migrées en l’état vers Azure sans aucune modification. 
- **Préparé pour Azure sous condition**: Les machines peuvent être migrées vers Azure, mais elles nécessitent des modifications mineures, conformément aux instructions de correction fournies dans l’évaluation.
- **Non disponible pour Azure** : Les machines ne peuvent pas être migrées vers Azure en l’état. Des problèmes doivent être corrigés conformément aux instructions de correction, avant la migration. 
- **État de la préparation inconnu** : Azure Migrate ne parvient pas à déterminer l’état de préparation des machines en raison d’un manque de métadonnées.

Grâce aux évaluations des bases de données, vous pouvez estimer l’état de préparation de votre patrimoine de données SQL Server vers Azure SQL Database ou Azure SQL Managed Instance. L’évaluation indique le pourcentage d’état de préparation à la migration pour chacune de vos instances SQL Server. De plus, pour chaque instance, vous pouvez voir la cible recommandée dans Azure, les bloqueurs de migration potentiels, le nombre de changements cassants, le niveau de préparation pour Azure SQL DB ou Azure SQL VM et un niveau de compatibilité. Vous pouvez approfondir l’étude de l’impact des bloqueurs de migration et des recommandations pour les résoudre.

 ![Évaluations des bases de données](./media/concepts-migration-planning/database-assessment-portal.png)

### <a name="sizing-recommendations"></a>Recommandations de dimensionnement

Une fois qu’une machine est marquée comme prête pour Azure, Server Assessment fait des recommandations de dimensionnement afin d’identifier la référence SKU de machine virtuelle Azure et le type de disque adaptés à vos machines. Vous pouvez recevoir des recommandations de dimensionnement en fonction de l’historique des performances (pour optimiser les ressources pendant la migration) ou en fonction des paramètres des machines locales, sans historique des performances. Dans une évaluation de base de données, vous pouvez voir des recommandations sur la référence SKU de base de données, le niveau tarifaire et le niveau de calcul.  

### <a name="get-compute-costs"></a>Obtenir les coûts de calcul

L’option de dimensionnement basé sur les performances dans les évaluations Azure Migrate vous aide à dimensionner correctement les machines virtuelles. L’utilisation de cette option constitue une bonne pratique pour optimiser les charges de travail dans Azure. En plus du bon dimensionnement, il existe d’autres options pour vous aider à réduire les coûts Azure : 

- **Instances réservées** : Avec des [instances réservées](https://azure.microsoft.com/pricing/reserved-vm-instances/), vous pouvez réduire considérablement les coûts par rapport à ceux du [paiement à l’utilisation](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/).
- **Azure Hybrid Benefit** : Avec [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/), vous pouvez utiliser des licences Windows Server locales avec des abonnements Software Assurance ou Linux actifs sur Azure et les combiner avec les options des instances réservées.
- **Contrat Entreprise** : Un [Contrat Entreprise (EA)](../cost-management-billing/manage/ea-portal-agreements.md) Azure peut permettre de réaliser des économies sur les abonnements et services Azure.
- **Offres** : Il existe plusieurs [offres Azure](https://azure.microsoft.com/support/legal/offer-details/). Par exemple, [Dev/Test - Paiement à l’utilisation](https://azure.microsoft.com/pricing/dev-test/) ou [Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/), pour réduire les tarifs des machines virtuelles de développement/test.
- **Durée de fonctionnement de la machine virtuelle** : Vous pouvez déterminer les jours par mois et les heures par jour pendant lesquels les machines virtuelles Azure s’exécutent. Arrêter les machines quand elles ne sont pas utilisées peut réduire vos coûts (non applicable aux instances réservées).
- **Région cible** : Vous pouvez créer des évaluations dans différentes régions pour déterminer si la migration vers une région particulière peut s’avérer plus rentable. 

### <a name="visualize-data"></a>Visualiser les données

Vous pouvez consulter les rapports Server Assessment (contenant les informations sur la préparation pour Azure et la distribution des coûts mensuels) dans le portail. Vous pouvez aussi exporter une évaluation et enrichir votre plan de migration avec d’autres visualisations. Vous pouvez créer plusieurs évaluations, avec différentes combinaisons de propriétés, puis choisir l’ensemble de propriétés convenant le mieux à votre entreprise.  

 ![Vue d’ensemble des évaluations](./media/concepts-migration-planning/assessment-summary.png)

### <a name="evaluate-gapsblockers"></a>Évaluer les lacunes/les obstacles

À mesure que vous déterminez les applications et les charges de travail à migrer, identifiez leurs contraintes de temps d’arrêt et recherchez toutes les dépendances opérationnelles entre vos applications et l’infrastructure sous-jacente. Cette analyse vous aide à planifier des migrations qui répondent à votre objectif de délai de récupération (RTO) et à garantir une perte de données minime voire nulle. Avant de procéder à la migration, nous vous recommandons de passer en revue et d’atténuer les éventuels problèmes de compatibilité, ou fonctionnalités non prises en charge, susceptibles de bloquer la migration des serveurs/bases de données SQL. Le rapport Azure Migrate Server Assessment et Azure Migrate Database Assessment peuvent vous y aider. 

### <a name="prioritize-workloads"></a>Hiérarchiser les charges de travail

Une fois que vous avez collecté des informations sur votre inventaire, vous pouvez identifier les applications et charges de travail à migrer en premier. Adoptez une approche « appliquer et apprendre » pour migrer des applications de manière systématique et contrôlable, afin de pouvoir éliminer toutes les failles avant de commencer une migration à grande échelle.

Pour hiérarchiser l’ordre de migration, vous pouvez utiliser des facteurs stratégiques comme la complexité, la durée de migration, l’urgence pour l’entreprise, les considérations liés à la production/non-production, la conformité, les exigences de sécurité, la connaissance des applications, etc. 

Voici quelques recommandations :

- **Donner la priorité aux bénéfices rapides** : Utilisez les rapports d’évaluation pour identifier les ressources à portée de main, notamment les serveurs et les bases de données qui sont entièrement prêts et nécessitent un effort minimal pour migrer vers Azure. Le tableau récapitule les différentes façons de procéder.

    **State** | **Action**
    --- | ---
    **Machines virtuelles prêtes pour Azure** | Exportez le rapport d’évaluation et filtrez toutes les machines dont l’état est *Disponible pour Azure*. Il peut s’agir du premier groupe de machines à déplacer vers Azure à l’aide de l’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool).
    **Systèmes d’exploitation en fin de prise en charge** | Exportez le rapport d’évaluation et filtrez toutes les machines exécutant Windows Server 2008 R2/Windows Server 2008. Ces systèmes d’exploitation sont en fin de prise en charge et seul Azure propose gratuitement trois ans de mises à jour de sécurité quand vous les migrez vers Azure. Si vous combinez Azure Hybrid Benefit et utilisez des instances réservées, les économies peuvent s’avérer beaucoup plus importantes.
    **Migration de SQL Server** | Utilisez les recommandations issues de l’évaluation des bases de données afin de migrer celles qui sont prêtes pour Azure SQL Database à l’aide de l’outil Azure Migrate : Database Migration. Migrez les bases de données prêtes pour une machine virtuelle Azure SQL à l’aide de l’outil Azure Migrate : d’Azure Migrate.
    **Logiciel en fin de prise en charge** | Exportez l’inventaire des applications et filtrez les logiciels/extensions susceptibles d’être en fin de prise en charge. Hiérarchisez ces applications pour la migration.
    **Machines sous-provisionnées** | Exportez votre rapport d’évaluation et filtrez les machines qui présentent une faible utilisation du processeur (%) et une faible utilisation de la mémoire (%).  Effectuez une migration vers une machine virtuelle Azure de taille adéquate et réduisez les coûts liés aux ressources sous-exploitées.
    **Machines sur-provisionnées** | Exportez votre rapport d’évaluation et filtrez les machines qui présentent une forte utilisation du processeur (%) et une forte utilisation de la mémoire (%).  Résolvez les contraintes de capacité, prévenez l’arrêt des machines en tension et augmentez les performances en migrant ces machines vers Azure. Dans Azure, utilisez les fonctionnalités de mise à l’échelle automatique pour répondre à la demande.<br/><br/> Analysez les rapports d’évaluation pour identifier les contraintes de stockage. Analysez les IOPS de disque et le débit, ainsi que le type de disque recommandé.

- **Commencez petit, puis voyez les choses en grand** : Commencez par déplacer les applications et charges de travail qui présentent le moins de risques et de complexité, afin de prendre confiance dans votre stratégie de migration. Analysez les recommandations issues de l’évaluation Azure Migrate avec votre référentiel CMDB, pour rechercher et migrer les charges de travail de développement/test susceptibles d’être candidates à des migrations pilotes. Les commentaires et leçons tirées des migrations pilotes peuvent s’avérer utiles quand vous commencez à migrer des charges de travail de production.  
- **Pensez conformité** : Azure offre la plus grande gamme de solutions de conformité du secteur d’activité en termes d’offres horizontales et verticales. Utilisez des exigences de conformité pour hiérarchiser les migrations, afin que les applications et charges de travail soient conformes à vos normes et législations nationales, régionales et sectorielles. Cela est particulièrement vrai pour les organisations qui gèrent des processus vitaux pour l’entreprise, qui conservent des informations sensibles ou qui évoluent dans des secteurs d’activité très réglementés. Dans ces types d’organisations, les normes et législations sont nombreuses et peuvent changer souvent, ce qui les rend difficiles à suivre.  

## <a name="finalize-the-migration-plan"></a>Finaliser le plan de migration

Avant de finaliser votre plan de migration, veillez à prendre en compte et à atténuer les autres bloqueurs potentiels suivants : 

- **Configuration réseau requise** : Évaluez les contraintes de bande passante et de latence réseau, susceptibles d’entraîner des retards imprévus et des perturbations du rythme de réplication de la migration.
- **Tests/Adaptations post-migration** : Laissez un délai tampon pour effectuer des tests de performances et d’acceptation par les utilisateurs sur les applications migrées, ou bien pour configurer/adapter les applications après la migration, notamment mettre à jour les chaînes de connexion de base de données, configurer les serveurs web, effectuer des basculements/nettoyages, etc.
- **Autorisations** : Passez en revue les autorisations Azure recommandées, ainsi que les rôles d’accès serveur/base de données et les autorisations nécessaires à la migration.
- **Formation** : Préparez votre organisation à la transformation numérique. Une formation solide est essentielle à la réussite des changements organisationnels. Consultez la formation gratuite sur [Microsoft Learn](/learn/azure/?ocid=CM_Discovery_Checklist_PDF), comprenant des cours sur les notions de base d’Azure, les architectures de solution et la sécurité. Encouragez votre équipe à se renseigner sur les  [certifications Azure](https://www.microsoft.com/learning/certification-overview.aspx?ocid=CM_Discovery_Checklist_PDF).  
- **Prise en charge de l’implémentation** : Bénéficiez d’un support pour votre implémentation, si nécessaire. De nombreuses organisations choisissent une aide externe pour prendre en charge la migration vers le cloud. Pour passer à Azure rapidement et en toute confiance avec une assistance personnalisée, envisagez de recourir à un  [fournisseur de services managés Azure](https://www.microsoft.com/solution-providers/search?cacheId=9c2fed4f-f9e2-42fb-8966-4c565f08f11e&ocid=CM_Discovery_Checklist_PDF) ou à  [FastTrack for Azure](https://azure.microsoft.com/programs/azure-fasttrack/?ocid=CM_Discovery_Checklist_PDF).  


Créez un plan de migration cloud efficace qui contient des informations détaillées sur les applications à migrer, la disponibilité des applications/bases de données, les contraintes de temps d’arrêt et les étapes majeures de la migration. Le plan prend en compte la durée de la copie des données et inclut un tampon réaliste pour les tests post-migration et les activités de basculement. 

Un plan de tests post-migration doit inclure des cas d’utilisation et des tests fonctionnels, d’intégration, de sécurité et de performances pour vérifier que les applications migrées fonctionnent comme prévu et que tous les objets de base de données et les relations de données sont transférés vers le cloud.  

Créez une feuille de route de migration et déclarez une fenêtre de maintenance afin de migrer vos applications et bases de données avec un temps d’arrêt minimal voire nul et limiter l’impact opérationnel et commercial potentiel pendant la migration.  

## <a name="migrate"></a>Migrer

Nous vous recommandons d’exécuter une migration de test dans Azure Migrate, avant de commencer la migration à grande échelle. Une migration de test vous permet d’estimer le temps nécessaire et d’adapter votre plan de migration. Elle offre la possibilité de détecter d’éventuels problèmes et de les résoudre avant la migration complète.

Quand vous êtes prêt pour la migration, utilisez l’outil Azure Migrate : Server Migration et Azure Data Migration Service (DMS) pour bénéficier d’une expérience de migration harmonieuse et intégrée, avec un suivi de bout en bout.

- Avec l’outil Server Migration, vous pouvez migrer des machines virtuelles et des serveurs locaux, ou des machines virtuelles situées dans un autre cloud privé ou public (notamment AWS, GCP) sans aucun temps d’arrêt.
- Azure DMS fournit un service complètement managé conçu pour permettre des migrations harmonieuses de plusieurs sources de base de données vers des plateformes de données Azure avec un temps d’arrêt minimal.  

## <a name="next-steps"></a>Étapes suivantes

- Examinez le  [parcours de migration cloud](/azure/architecture/cloud-adoption/getting-started/migrate) dans le framework d’adoption du cloud Azure.
- Obtenez une [vue d’ensemble rapide](migrate-services-overview.md) d’Azure Migrate et visionnez une [vidéo de prise en main](https://youtu.be/wFfq3YPxYHE).
- Découvrez-en plus sur l’évaluation des machines virtuelles pour la migration vers des [machines virtuelles Azure](concepts-assessment-calculation.md).
