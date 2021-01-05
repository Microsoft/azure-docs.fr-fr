---
title: Optimiser votre investissement dans le cloud avec Azure Cost Management
description: Cet article vous aide à tirer le meilleur parti de vos investissements dans le cloud, à réduire vos coûts et à évaluer vos postes de dépenses.
author: bandersmsft
ms.author: banders
ms.date: 05/27/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18
ms.openlocfilehash: 4dd22427038bc7c4052ee714c3ec3ce45d7a1837
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132820"
---
# <a name="how-to-optimize-your-cloud-investment-with-azure-cost-management"></a>Guide pratique pour optimiser votre investissement dans le cloud avec Azure Cost Management

Azure Cost Management met à votre disposition les outils nécessaires pour la planification, l’analyse et la réduction de vos dépenses en vue d’optimiser votre investissement dans le cloud. Ce document fournit une approche méthodique de la gestion des coûts et met en valeur les outils dont vous pouvez vous servir pour résoudre les problèmes de coût auxquels votre organisation peut être confrontée. Azure permet de générer et de déployer facilement des solutions cloud. Toutefois, il est important que ces solutions soient optimisées afin de réduire les coûts pour votre organisation. Si vous suivez les principes présentés dans ce document et utilisez nos outils, vous pourrez mener votre organisation à la réussite.

## <a name="methodology"></a>Méthodologie

La gestion des coûts concerne l’ensemble de l’organisation et doit être un travail constant qui commence avant la moindre dépense dans des ressources cloud. Pour implémenter la gestion des coûts et optimiser les coûts correctement, votre organisation doit :

- Être préparée avec les outils appropriés
- Être comptable des coûts
- Prendre les mesures appropriées pour optimiser les dépenses

Trois groupes clés, décrits ci-après, doivent être alignés dans votre organisation pour que vous gériez les coûts correctement.

- **Finance** : personnes responsables de l’approbation des demandes de budget au sein de l’organisation en fonction des prévisions de dépenses dans le cloud. Elles paient la facture correspondante et affectent les coûts aux différentes équipes pour responsabiliser celles-ci.
- **Gestionnaires** : décideurs dans une organisation qui doivent comprendre les dépenses cloud afin d’en tirer les meilleurs résultats.
- **Équipes d’application** : ingénieurs qui gèrent les ressources cloud sur une base quotidienne et développent des services pour répondre aux besoins de l’organisation. Ces équipes ont besoin de flexibilité pour optimiser leurs budgets définis.

### <a name="key-principles"></a>Principes clés

Utilisez les principes énoncés ci-après pour que votre organisation gère correctement les coûts liés au cloud.

Pour en savoir plus, regardez la vidéo [Cost Management - Configuration pour une utilisation réussie](https://www.youtube.com/watch?v=dVuwITdSAZ4). Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/dVuwITdSAZ4]

#### <a name="planning"></a>Planification

Une planification initiale et complète vous permet d’adapter l’utilisation du cloud en fonction des besoins de votre entreprise. Posez-vous les questions suivantes :

- Quel est le problème métier que je souhaite résoudre ?
- Quels sont les modèles d’utilisation que j’attends de mes ressources ?

Vos réponses vous aideront à sélectionner les offres qui vous conviennent. Elles déterminent l’infrastructure à utiliser et comment s’en servir pour optimiser votre efficacité Azure.

#### <a name="visibility"></a>Visibilité

Quand la gestion des coûts est correctement structurée, elle vous aide à informer les personnes au sujet des coûts Azure dont elles sont responsables et de l’argent qu’elles dépensent. Azure propose des services qui vous aident à déterminer *où* votre argent est dépensé. Tirez parti de ces outils. Ils peuvent vous aider à trouver des ressources qui sont trop utilisées, à supprimer les gaspillages et à optimiser les opportunités d’économies de coûts.

#### <a name="accountability"></a>Responsabilité

Assignez les coûts dans votre organisation afin que des personnes responsables soient garantes des dépenses de leur équipe. Pour bien comprendre les dépenses Azure de votre organisation, vous devez organiser vos ressources afin d’optimiser l’insight de l’affectation des coûts. Une bonne organisation permet de gérer et de réduire les coûts et de responsabiliser les personnes quant à l’efficacité des dépenses dans votre organisation.

#### <a name="optimization"></a>Optimization

Agissez pour réduire vos dépenses. Optimisez-les à partir des résultats collectés par le biais de la planification et d’une meilleure visibilité des coûts. Vous pouvez envisager des optimisations des achats et de la gestion des licences, ainsi que des modifications du déploiement de l’infrastructure qui sont décrites en détail plus loin dans ce document.

#### <a name="iteration"></a>Itération

Tous les membres de votre organisation doivent s’engager dans le cycle de vie de la gestion des coûts. Ils doivent participer de manière continue à l’optimisation des coûts. Faites preuve de rigueur vis-à-vis de ce processus itératif et faites-en un principe clé de gouvernance cloud responsable dans votre organisation.

![Schéma des principes clés montrant la visibilité, la responsabilité et l’optimisation](./media/cost-mgt-best-practices/principles.png)

## <a name="plan-with-cost-in-mind"></a>Planifier avec le coût à l’esprit

Avant de déployer des ressources cloud, évaluez les éléments suivants :

- L’offre Azure qui répond le mieux à vos besoins
- Les ressources que vous prévoyez d’utiliser
- Leur coût potentiel

Azure fournit des outils pour vous aider dans ce processus d’évaluation. Ces outils peuvent vous donner une idée de l’investissement nécessaire pour activer vos charges de travail. Ensuite, vous pouvez sélectionner la meilleure configuration suivant votre situation.

### <a name="azure-onboarding-options"></a>Options d’intégration Azure

Pour optimiser votre expérience au sein de Cost Management, la première étape consiste à déterminer et à choisir l’offre Azure qui vous convient le mieux. Pensez à la façon dont vous prévoyez d’utiliser Azure. Penchez-vous également sur la configuration que vous souhaitez pour votre modèle de facturation. Posez-vous les questions suivantes quand vous prenez votre décision :

- Combien de temps est-ce que j’envisage d’utiliser Azure ? Mon souhait est-il d’effectuer des tests ou de créer une infrastructure à plus long terme ?
- Comment est-ce que je souhaite payer l’utilisation d’Azure ? Dois-je opter pour le prépaiement afin d’obtenir un prix réduit ou être facturé à la fin du mois ?

Pour en savoir plus sur les différentes options, consultez la page [Acheter Azure](https://azure.microsoft.com/pricing/purchase-options/). Plusieurs des modèles de facturation les plus courants sont présentés ci-après.

#### <a name="free"></a>[Gratuit](https://azure.microsoft.com/free/)

- 12 mois de services gratuits populaires
- Crédit de 170 € pour explorer les services pendant 30 jours
- Plus de 25 services sont toujours gratuits

#### <a name="pay-as-you-go"></a>[Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p)

- Aucun minimum, aucun engagement
- Tarifs compétitifs
- Payez uniquement pour ce que vous utilisez
- Annulation possible à tout moment

#### <a name="enterprise-agreement"></a>[Contrat Entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/)

- Options pour des engagements monétaires initiaux
- Accès à des tarifs Azure réduits

#### <a name="azure-in-csp"></a>[Azure in CSP](https://azure.microsoft.com/offers/ms-azr-0145p/)

- Les partenaires CSP sont le premier point de contact pour les besoins de leurs clients et constituent le centre de la relation client.
- Les partenaires CSP provisionnent de nouveaux clients, commandent des abonnements, gèrent des abonnements et effectuent des tâches d’administration pour le compte de leurs clients.
- Les partenaires CSP regroupent les services avec des solutions uniques ou revendent Azure tout en contrôlant les prix, les conditions et la facturation.

## <a name="estimate-the-cost-of-your-solution"></a>Estimer le coût de votre solution

Avant de déployer une infrastructure, évaluez le coût de votre solution. L’évaluation vous aide à créer un budget initial pour votre organisation en ce qui concerne la charge de travail. Ensuite, vous pouvez utiliser un budget au fil du temps pour évaluer la validité de votre estimation initiale. Et vous pouvez la comparer au coût réel de votre solution déployée.

### <a name="azure-pricing-calculator"></a>Calculatrice de prix Azure

La calculatrice de prix Azure vous permet de combiner différents services Azure pour voir une estimation des coûts. Vous pouvez implémenter votre solution de différentes façons dans Azure, chacune pouvant influer sur vos dépenses globales. En réfléchissant à l’avance aux besoins en infrastructure de votre déploiement cloud, vous pouvez optimiser l’utilisation de l’outil. Vous pouvez ainsi obtenir une estimation solide de vos dépenses dans Azure.

Pour plus d’informations, consultez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator).

### <a name="azure-migrate"></a>Azure Migrate

Azure Migrate est un service qui évalue les charges de travail actuelles de votre organisation dans les centres de données locaux. Il vous donne un insight de ce que vous pourriez attendre d’une solution de remplacement Azure. Tout d’abord, Migrate analyse vos machines locales pour déterminer si la migration est possible. Ensuite, il recommande un dimensionnement des machines virtuelles dans Azure pour optimiser les performances. Enfin, il crée également une estimation de coût pour une solution basée sur Azure.

Pour plus d’informations, consultez [Azure Migrate](../../migrate/migrate-services-overview.md).

## <a name="analyze-and-manage-your-costs"></a>Analyser et gérer les coûts

Restez informé de la façon dont les coûts de votre organisation évoluent au fil du temps. Utilisez les techniques suivantes pour comprendre et gérer vos dépenses correctement.

### <a name="organize-resources-to-maximize-cost-insights-and-accountability"></a>Organiser les ressources pour optimiser les insights et la responsabilité

Une structure organisationnelle bien planifiée pour vos hiérarchies des ressources et de facturation Azure vous aide à mieux comprendre et contrôler les coûts à mesure que vous créez votre infrastructure cloud. Regardez la vidéo [Configuration de hiérarchies d’entités](https://www.youtube.com/watch?v=n3TLRaYJ1NY) pour mieux comprendre les outils organisationnels disponibles et la manière d’en tirer profit. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/n3TLRaYJ1NY]

À mesure que vous évaluez et créez une hiérarchie répondant à vos besoins, posez-vous les questions suivantes.

*Quelle hiérarchie de facturation est à ma disposition et quelles sont les différentes portées que je peux utiliser ?*

Identifiez l’organisation de la facturation pour votre organisation en déterminant le type de votre offre Azure. Les portées disponibles pour chaque organisation de facturation Azure sont détaillées dans [Comprendre et utiliser les portées](understand-work-scopes.md).

*Si j’ai plusieurs équipes, comment dois-je organiser mes abonnements et groupes de ressources ?*

La création d’un abonnement ou d’un groupe de ressources pour chaque équipe est une pratique courante. Elle peut vous aider à différencier les coûts et à encourager les équipes à être responsables. Toutefois, les coûts sont liés à l’abonnement ou au groupe de ressources.

Si vous avez déjà des équipes avec plusieurs abonnements, envisagez de regrouper les abonnements en groupes d’administration pour analyser les coûts de manière groupée. Les groupes d’administration, abonnements et groupes de ressources font tous partie de la hiérarchie RBAC Azure. Utilisez-les de manière collective pour le contrôle d'accès dans vos équipes.

Les ressources peuvent être réparties sur plusieurs portées, en particulier si elles sont partagées par plusieurs équipes ou charges de travail. Envisagez d’identifier les ressources avec des étiquettes. Les étiquettes sont davantage détaillées dans la section suivante.

*Ai-je des environnements de développement et de production ?*

Envisagez de créer des abonnements dev/test pour vos environnements de développement afin de tirer parti des réductions de prix. Si les charges de travail sont réparties sur plusieurs équipes ou portées Azure, envisagez d’utiliser des étiquettes pour les identifier.

### <a name="tag-shared-resources"></a>Identifiez les ressources partagées

Les étiquettes représentent une manière pratique de comprendre les coûts répartis entre plusieurs équipes et portées Azure. Par exemple, vous pouvez avoir une ressource, telle qu’un serveur de messagerie utilisé par plusieurs équipes. Vous pouvez ajouter une ressource partagée, telle que le serveur de messagerie, à un abonnement dédié aux ressources partagées ou l’ajouter à un abonnement existant. Si vous l’ajoutez à un abonnement existant, il est possible que le propriétaire de l’abonnement ne souhaite pas que ses coûts s’appliquent à son équipe chaque mois. Pour cet exemple, vous pouvez utiliser une étiquette pour identifier les ressource comme partagées.

De même, il est possible que vous ayez des environnements ou applications web, par exemple de test ou de production, qui utilisent des ressources de plusieurs abonnements en possession de différentes équipes. Pour mieux comprendre le coût total des charges de travail, identifiez les ressources qu’elles utilisent. Lorsque les étiquettes sont correctement appliquées, elles peuvent servir à filtrer l’analyse des coûts pour mieux comprendre les tendances.

Une fois l’identification des ressources planifiée, vous pouvez configurer une stratégie Azure pour appliquer l’identification des ressources. Regardez la vidéo [Comment examiner les stratégies d’identification avec Azure Cost Management](https://www.youtube.com/watch?v=nHQYcYGKuyw) pour comprendre les outils dont vous disposez pour appliquer une identification des ressources évolutive. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/nHQYcYGKuyw]

### <a name="use-cost-analysis"></a>Utiliser l’analyse des coûts

L’analyse des coûts vous permet d’analyser les coûts de votre organisation de manière approfondie en disséquant les coûts à l’aide de propriétés de ressource standard. Appuyez-vous sur les questions courantes suivantes pour effectuer votre analyse. En répondant régulièrement à ces questions, vous pouvez vous tenir plus informé et prendre des décisions tenant davantage compte des coûts.

- **Coûts estimés pour le mois en cours** : combien ai-je dépensé jusqu’ici ce mois-ci ? Est-ce que je vais respecter mon budget ?
- **Examiner les anomalies** : effectuez des vérifications régulières pour vous assurer que les coûts correspondent à des usages normaux et raisonnables. Quelles sont les tendances ? Existe-t-il des aberrations ?
- **Rapprochement de facture** : mon dernier coût facturé est-il supérieur à celui du mois précédent ? Comment les habitudes de dépenses ont-elles évolué sur un mois ?
- **Rétrofacturation interne** : le montant de la facturation étant connu, comment ces coûts doivent-ils être ventilés pour mon organisation ?

Pour plus d’informations, consultez [Analyse des coûts](quick-acm-cost-analysis.md).

### <a name="export-billing-data-on-a-schedule"></a>Exporter les données de facturation selon une planification

Devez-vous importer vos données de facturation dans un système externe, tel qu’un tableau de bord ou un système financier ? Configurez des exportations automatisées dans le stockage Azure et évitez de télécharger manuellement les fichiers chaque mois. Vous pouvez ensuite facilement configurer les intégrations automatique avec d’autres systèmes pour préserver la synchronisation de vos données de facturation.

Pour plus d’informations sur l’exportation des données de facturation, consultez [Créer et gérer des données exportées](tutorial-export-acm-data.md).

### <a name="create-budgets"></a>Créer des budgets

Une fois que vous avez identifié et analysé vos modèles de dépenses, il est important de commencer à définir des limites pour vos équipes et vous-même. Dans Azure, vous pouvez définir un budget basé sur le coût ou l’utilisation avec de nombreux seuils et alertes. Veillez à examiner les budgets que vous créez régulièrement pour voir leur progression burndown et apporter les modifications éventuellement nécessaires. Les budgets Azure vous permettent également de configurer un déclencheur d’automatisation quand un seuil de budget donné est atteint. Par exemple, vous pouvez configurer votre service pour qu’il arrête les machines virtuelles. Ou vous pouvez déplacer votre infrastructure vers un autre niveau tarifaire en réponse à un déclencheur de budget.

Pour plus d’informations, consultez [Budgets Azure](tutorial-acm-create-budgets.md).

Pour plus d’informations sur l’automatisation basée sur les budgets, consultez [Automatisation basée sur les budgets](../manage/cost-management-budget-scenario.md).

## <a name="act-to-optimize"></a>Agir pour optimiser
Utilisez les méthodes suivantes pour optimiser les dépenses.

### <a name="cut-out-waste"></a>Supprimer les gaspillages

Une fois que vous avez déployé votre infrastructure dans Azure, il est important de s’assurer qu’elle est utilisée. La manière la plus simple de faire des économies immédiatement consiste à passer en revue vos ressources et à supprimer celles qui ne sont pas utilisées. Vous devriez alors déterminer si vos ressources sont utilisées aussi efficacement que possible.

#### <a name="azure-advisor"></a>Azure Advisor

Azure Advisor est un service qui, entre autres choses, identifie les machines virtuelles ayant une faible utilisation de l’UC ou du réseau. Vous pouvez alors décider de les arrêter ou de les redimensionner en fonction du coût estimé de la poursuite de leur exécution. Advisor fournit également des recommandations en termes d’achat d’instances réservées. Les recommandations sont basées sur vos 30 derniers jours d’utilisation des machines virtuelles. Quand elles sont suivies, les recommandations peuvent aider à réduire vos dépenses.

Pour plus d’informations, consultez [Azure Advisor](../../advisor/advisor-overview.md).

### <a name="size-your-vms-properly"></a>Dimensionner vos machines virtuelles correctement

Le dimensionnement des machines virtuelles a un impact significatif sur votre coût Azure global. Le nombre de machines virtuelles nécessaires dans Azure peut ne pas correspondre à la configuration que vous avez déployée dans un centre de données local. Veillez à choisir la bonne taille pour les charges de travail que vous projetez d’exécuter.

Pour plus d’informations, consultez [Azure IaaS : dimensionnement approprié et coût](https://azure.microsoft.com/resources/videos/azurecon-2015-azure-iaas-proper-sizing-and-cost/).

### <a name="use-purchase-discounts"></a>Utiliser des remises d’achat

Azure dispose de nombreuses remises dont votre organisation peut tirer parti pour réaliser des économies.

#### <a name="azure-reservations"></a>Réservations Azure

Les réservations Azure vous permettent de prépayer un ou trois ans de capacité de calcul de machine virtuelle ou de base de données SQL Database. Le prépaiement vous permet d’obtenir une remise sur les ressources que vous utilisez. Avec un engagement initial d’une durée de 1 ou 3 ans, les réservations Azure réduisent considérablement (jusqu’à 72 % par rapport au tarif du paiement à l’utilisation) les coûts de calcul de vos machines virtuelles ou de votre base de données SQL. Les réservations permettent de bénéficier d’une remise sur la facturation et n’ont aucune incidence sur l’état de runtime de vos machines virtuelles ou bases de données SQL.

Pour plus d’informations, consultez [Qu’est-ce qu’une réservation Azure ?](../reservations/save-compute-costs-reservations.md).

#### <a name="use-azure-hybrid-benefit"></a>Utiliser Azure Hybrid Benefit

Si vous avez déjà des licences Windows Server ou SQL Server dans vos déploiements locaux, vous pouvez utiliser le programme Azure Hybrid Benefit pour faire des économies dans Azure. Avec l’avantage Windows Server, chaque licence couvre le coût du système d’exploitation (jusqu’à deux machines virtuelles) et vous payez uniquement les coûts liés au calcul de base. Vous pouvez utiliser des licences SQL Server existantes pour économiser jusqu’à 55 % sur les options SQL Database basées sur vCore. Parmi ces options figurent SQL Server dans Machines virtuelles Azure et SQL Server Integration Services.

Pour plus d’informations, consultez [Calculatrice des économies réalisées avec Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).

### <a name="other-resources"></a>Autres ressources

Azure propose également un service avec lequel vous pouvez créer des services qui tirent parti de la capacité excédentaire dans Azure et bénéficier ainsi de tarifs réduits. Pour plus d’informations, consultez [Utiliser des machines virtuelles de faible priorité avec Batch](../../batch/batch-low-pri-vms.md).

## <a name="next-steps"></a>Étapes suivantes
- Si vous ne connaissez pas Cost Management, consultez [Qu’est-ce que la gestion des coûts Azure ?](../cost-management-billing-overview.md) pour découvrir dans quelle mesure ce processus aide à superviser et à maîtriser les dépenses Azure, mais aussi à optimiser l’utilisation des ressources.