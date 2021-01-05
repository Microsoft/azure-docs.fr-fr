---
title: Vue d’ensemble d’Azure Cost Management + Facturation
description: Vous utilisez les fonctionnalités d’Azure Cost Management + Facturation pour effectuer des tâches de facturation et gérer l’accès de la facturation aux coûts. Vous utilisez également les fonctionnalités pour superviser et contrôler les dépenses Azure, ainsi que pour optimiser l’utilisation des ressources Azure.
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 10/26/2020
ms.topic: overview
ms.service: cost-management-billing
ms.subservice: common
ms.custom: contperf-fy21q2
ms.openlocfilehash: 34034a99641d75e44783cb5b87af8948b4db1628
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029938"
---
# <a name="what-is-azure-cost-management--billing"></a>Présentation d’Azure Cost Management + Facturation

En utilisant le cloud Microsoft, vous pouvez améliorer considérablement les performances techniques des charges de travail de votre entreprise. Il peut également réduire vos coûts et la surcharge requise pour gérer les ressources organisationnelles. Toutefois, l’opportunité professionnelle crée un risque en raison de la possibilité de gaspillage et de manque d’efficacité introduits dans vos déploiements cloud. Azure Cost Management + Facturation est une suite d’outils fournis par Microsoft qui vous aident à analyser, gérer et optimiser les coûts de vos charges de travail. L’utilisation de la suite permet de garantir que votre organisation tire parti des avantages offerts par le cloud.

Vous pouvez considérer vos charges de travail Azure comme les lumières de votre foyer. Quand vous sortez pour la journée, les laissez-vous allumées ? Pourriez-vous utiliser des ampoules différentes qui sont plus efficaces pour réduire votre facture d’énergie mensuelle ? Avez-vous plus d’éclairages que nécessaire dans une pièce ? Vous pouvez utiliser Azure Cost Management + Facturation pour appliquer un processus de pensée semblable aux charges de travail utilisées par votre organisation.

Avec les produits et services Azure, vous payez uniquement ce que vous utilisez. Lorsque vous créez et utilisez des ressources Azure, celles-ci vous sont facturées. En raison de la facilité de déploiement pour les nouvelles ressources, les coûts de vos charges de travail peuvent augmenter de manière significative sans analyse et supervision appropriées. Vous utilisez les fonctionnalités d’Azure Cost Management + Facturation pour :

- Effectuer des tâches de facturation telles que le paiement de votre facture
- Gérer l’accès de la facturation aux coûts
- Télécharger les données d’utilisation et de coût qui ont été utilisées pour générer votre facture mensuelle
- Appliquer de manière proactive l’analyse des données à vos coûts
- Définir des seuils de dépense
- Identifier les opportunités de modification de la charge de travail qui peuvent optimiser vos dépenses

Pour découvrir plus en détail comment une organisation doit approcher la gestion des coûts, consultez l’article [Bonnes pratiques pour Azure Cost Management](./costs/cost-mgt-best-practices.md).

## <a name="understand-azure-billing"></a>Comprendre la facturation Azure

Les fonctionnalités de la facturation Azure permettent d’examiner vos factures et de gérer l’accès aux informations de facturation. Dans les grandes entreprises, les équipes chargées de l’approvisionnement et des finances s’occupent généralement des tâches de facturation.

Un compte de facturation est créé lorsque vous vous inscrivez pour utiliser Azure. Votre compte de facturation vous permet de gérer vos factures et paiements, et de suivre les coûts. Vous pouvez accéder à plusieurs comptes de facturation. Par exemple, vous pouvez vous être inscrit à Azure pour vos projets personnels. Par conséquent, chaque abonnement Azure peut avoir son compte de facturation. Vous pouvez aussi avoir accès à Azure via le Contrat Entreprise de votre organisation ou le Contrat client Microsoft. Pour chaque scénario, vous disposez d’un compte de facturation distinct.

### <a name="billing-accounts"></a>Comptes de facturation

Le portail Azure prend en charge les types de comptes de facturation suivants :

- **Programme Microsoft Online Services** : Un seul compte de facturation pour un Programme Microsoft Online Services est créé lorsque vous vous inscrivez à Azure via le site web Azure. Par exemple, quand vous souscrivez à un compte gratuit Azure, un compte avec tarifs de paiement à l’utilisation ou en tant qu’abonné Visual Studio.

- **Contrat Entreprise** : Un compte de facturation pour un Contrat Entreprise est créé quand votre organisation signe un Contrat Entreprise (EA) pour utiliser Azure.

- **Contrat client Microsoft** : Un compte de facturation pour un Contrat client Microsoft est créé lorsque votre organisation signe un Contrat client Microsoft avec un commercial Microsoft. Des clients dans certaines régions, qui s’inscrivent via le site web Azure pour un compte avec tarifs de paiement à l’utilisation ou mettent à niveau leur compte gratuit Azure, peuvent avoir un compte de facturation pour un Contrat client Microsoft.

### <a name="scopes-for-billing-accounts"></a>Étendues pour les comptes de facturation
Une étendue est un nœud à l’intérieur d’un compte de facturation que vous utilisez pour afficher et gérer la facturation. C’est là que vous gérez les données de facturation, les paiements, les factures, et effectuez la gestion des comptes de manière générale.

#### <a name="microsoft-online-services-program"></a>Programme Microsoft Online Services

|Étendue  |Définition  |
|---------|---------|
|Compte de facturation     | Représente un seul propriétaire (administrateur de compte) pour un ou plusieurs abonnements Azure. Un administrateur de compte est autorisé à effectuer diverses tâches de facturation comme créer des abonnements, afficher des factures ou modifier la facturation des abonnements.  |
|Abonnement     |  Représente un regroupement de ressources Azure. La facture est générée dans l’étendue de l’abonnement. Il a ses propres modes de paiement qui permettent de régler la facture associée.|

#### <a name="enterprise-agreement"></a>Contrat Entreprise

|Étendue  |Définition  |
|---------|---------|
|Compte de facturation    | Représente l’inscription à un Contrat Entreprise. La facture est générée dans l’étendue du compte de facturation. Il est structuré à l’aide des services et des comptes d’inscription.  |
|department     |  Regroupement facultatif des comptes d’inscription.      |
|Compte d’inscription     |  Représente un seul propriétaire de compte. Les abonnements Azure sont créés dans l’étendue des comptes d’inscription.  |

#### <a name="microsoft-customer-agreement"></a>Contrat client Microsoft

|Étendue  |Tâches  |
|---------|---------|
|Compte de facturation     |   Représente un contrat client pour plusieurs produits et services Microsoft. Le compte de facturation est structuré à l’aide de profils de facturation et de sections de facture.   |
|Profil de facturation     |  Représente une facture et ses modes de paiement. La facture est générée dans cette étendue. Le profil de facturation peut comprendre plusieurs sections de facture.      |
|Section de facture     |   Représente un groupe de coûts dans une facture. Les abonnements et autres achats sont associés à l’étendue de la section Facture.    |

## <a name="understand-azure-cost-management"></a>Présentation d’Azure Cost Management

Bien que liées, la facturation diffère de la gestion des coûts. La facturation est le processus qui vous permet de facturer des biens ou des services aux clients et de gérer les relations commerciales.

Cost Management révèle les modèles de coût et d’utilisation de l’organisation avec une analytique avancée. Les rapports dans Cost Management affichent les coûts basée sur l’utilisation consommés par les services Azure et les offres tierces de la Place de marché. Les coûts sont basés sur les prix négociés et prennent en compte la réservation et les remises Azure Hybrid Benefit. Ensemble, les rapports indiquent vos coûts d’utilisation internes et externes ainsi que les frais de la Place de marché Azure. Les autre frais, comme les achats de réservation, le support et les taxes, n’apparaissent pas encore dans les rapports. Ils vous aident à comprendre vos dépenses et votre utilisation des ressources, et à trouver des anomalies dans les dépenses. Des analyses prédictives sont aussi disponibles. Cost Management utilise des groupes d’administration, des budgets et des recommandations Azure pour montrer avec clarté comment vos dépenses sont organisées et comment vous pourriez réduire les coûts.

Vous pouvez utiliser le portail Azure ou différentes API pour automatiser l’exportation visant à intégrer les données de coût à des processus et systèmes externes. Vous avez aussi à disposition une exportation automatisée des données de facturation ainsi que des rapports planifiés.

Regardez la vidéo de présentation d’Azure Cost Management pour avoir une vue d’ensemble rapide de la façon dont Azure Cost Management peut vous aider à économiser de l’argent dans Azure. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/el4yN5cHsJ0]

### <a name="plan-and-control-expenses"></a>Planifier et maîtriser les dépenses

Cost Management vous aide à planifier et à contrôler vos coûts de différentes façons : Analyse des coûts, budgets, suggestions et exportation des données de gestion des coûts.

Utilisez l’analyse du coût pour explorer et analyser les coûts de votre organisation. Vous pouvez voir les coûts agrégés par l’organisation afin de comprendre où les coûts ont augmenté et d’identifier les tendances de dépenses. Vous pouvez aussi voir les coûts cumulés au fil du temps pour évaluer des tendances mensuelles, trimestrielles ou même annuelles par rapport à un budget.

Les budgets vous aident à planifier et à suivre la comptabilité de votre organisation. Ils empêchent de dépasser les plafonds ou les limites de coût. Les budgets peuvent aussi vous permettre d’informer les autres de leurs dépenses pour gérer les coûts de manière proactive. Vous pouvez aussi voir la progression des dépenses.

Les recommandations vous montrent comment optimiser et améliorer votre efficacité en identifiant les ressources inactives et sous-utilisées. Elles peuvent aussi indiquer les options de ressource les moins chères. Si vous agissez sur des recommandations, vous changez la façon dont vous utilisez vos ressources pour économiser de l’argent. Pour agir, consultez d’abord les recommandations d’optimisation des coûts pour voir les inefficacités d’utilisation potentielles. Ensuite, agissez sur une recommandation pour modifier votre utilisation des ressources Azure et bénéficier d’une option plus rentable. Vérifiez ensuite que l’action que vous prenez change votre situation en mieux.

Si vous utilisez des systèmes externes pour accéder ou consulter les données de gestion des coûts, vous pouvez facilement exporter les données à partir d’Azure. Vous pouvez également définir une exportation planifiée quotidienne au format CSV et stocker les fichiers de données dans le stockage Azure. Ensuite, vous pouvez accéder aux données à partir de votre système externe.

### <a name="cloudyn-deprecation"></a>Dépréciation de Cloudyn

Cloudyn est un service Azure lié à Cost Management, qui sera déprécié d’ici la fin de l’année 2020. Les fonctionnalités existantes de Cloudyn sont intégrées directement au portail Azure dans la mesure du possible. Aucun nouveau client n’est actuellement intégré pour le moment. Cependant, le produit restera pris en charge jusqu’à ce qu’il soit complètement déprécié.
 
### <a name="additional-azure-tools"></a>Autres outils Azure

Azure dispose d’autres outils qui ne font pas partie de l’ensemble des fonctionnalités d’Azure Cost Management + Facturation. Toutefois, ils jouent un rôle important dans le processus de gestion des coûts. Pour en savoir plus sur ces outils, consultez les liens suivants.

- [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/) - Utilisez cet outil pour estimer vos coûts cloud à l’avance.
- [Azure Migrate](../migrate/migrate-services-overview.md) - Évaluez la charge actuelle de votre centre de données pour savoir ce dont vous avez besoin pour une solution de remplacement Azure.
- [Azure Advisor](../advisor/advisor-overview.md) - Identifiez les machines virtuelles inutilisées et recevez des recommandations sur les achats d’instance réservée Azure.
- [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) - Utilisez vos licences Windows Server ou SQL Server locales actuelles pour les machines virtuelles dans Azure afin d’économiser.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec Cost Management + Facturation, vous pouvez commencer à utiliser le service.

- Commencez à utiliser Azure Cost Management pour [analyser les coûts](./costs/quick-acm-cost-analysis.md).
- Vous pouvez également en savoir plus sur les [bonnes pratiques pour Azure Cost Management](./costs/cost-mgt-best-practices.md).