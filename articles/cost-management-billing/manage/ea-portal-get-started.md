---
title: Bien démarrer avec le portail Azure Enterprise
description: Cet article explique comment les clients Contrat Entreprise Azure (Azure EA) utilisent le portail Azure Enterprise.
author: bandersmsft
ms.reviewer: baolcsva
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 10/28/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: 88bb4c24489b973aa4230ba6f2b34756f83c7dda
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030686"
---
# <a name="get-started-with-the-azure-enterprise-portal"></a>Bien démarrer avec le portail Azure Enterprise

Cet article aide les clients Contrat Entreprise Azure (Azure EA) directs et indirects à commencer à utiliser le [portail Azure Enterprise](https://ea.azure.com). Obtenir des informations de base sur les thèmes suivants :

- La structure du portail Azure Enterprise.
- Les rôles utilisés dans le portail Azure Enterprise.
- Création d’un abonnement.
- Analyse des coûts dans les portails Azure Enterprise et Azure.

## <a name="get-started-with-ea-onboarding"></a>Prise en main de l’intégration d’EA

Pour obtenir un guide d’intégration d’Azure EA, consultez [Guide d’intégration d’Azure EA (PDF)](https://ea.azure.com/api/v3Help/v2AzureEAOnboardingGuide).

Regardez cette vidéo pour visionner une session d’intégration complète dans le portail Azure Enterprise :

> [!VIDEO https://www.youtube.com/embed/OiZ1GdBpo-I]

## <a name="understanding-ea-user-roles-and-introduction-to-user-hierarchy"></a>Compréhension des rôles d’utilisateur EA et présentation de la hiérarchie utilisateur

Pour faciliter la gestion de l’utilisation et des dépenses d’une organisation, les clients Azure disposant d’un Contrat Entreprise peuvent attribuer cinq rôles d’administrateur distincts :

- Administrateur d’entreprise
- Administrateur d’entreprise (lecture seule)
- Administrateur de service
- Administrateur de service (lecture seule)
- Propriétaire du compte

Chaque rôle a un degré variable de limites et d’autorisations utilisateur. Pour plus d’informations, consultez [Structure de l’organisation et autorisations par rôle](./understand-ea-roles.md#organization-structure-and-permissions-by-role).

## <a name="activate-your-enrollment-create-a-subscription-and-other-administrative-tasks"></a>Activer votre inscription, créer un abonnement et d’autres tâches d’administration

Pour plus d’informations sur l’activation de votre inscription, la création d’un service ou d’un abonnement, l’ajout d’administrateurs et de propriétaires de comptes, et d’autres tâches d’administration, consultez [Administration du portail Azure EA](./ea-portal-administration.md).

Si vous souhaitez en savoir plus sur le transfert d’un abonnement Enterprise sur un abonnement avec paiement à l’utilisation, consultez [Transferts Azure Enterprise](./ea-transfers.md).

## <a name="view-usage-summary-and-download-reports"></a>Consulter le résumé de l’utilisation et télécharger les rapports

Vous pouvez gérer et intervenir sur votre facture Azure EA. Votre facture représente ce que vous devez payer, il convient donc d’en vérifier l’exactitude.

Pour voir le récapitulatif de l’utilisation, télécharger des rapports et gérer des factures d’inscription, consultez [Factures d’inscription Azure Enterprise](./ea-portal-enrollment-invoices.md).

## <a name="now-that-youre-familiar-with-the-basics-here-are-some-additional-links-to-help-you-get-onboarded"></a>À présent que vous êtes familiarisé avec les principes de base, voici quelques liens supplémentaires pour vous aider à vous lancer

[Les tarifs Azure EA](./ea-pricing-overview.md) fournissent des informations détaillées sur la façon dont l’utilisation est calculée et explique les frais de divers services Azure du Contrat Entreprise, où les calculs sont plus complexes.

Si vous souhaitez savoir comment les réservations Azure pour les instances réservées de machine virtuelle peuvent vous aider à faire des économies au niveau de votre inscription d’entreprise, consultez [Instances réservées de machine virtuelle Azure EA](./ea-portal-vm-reservations.md).

Pour obtenir des informations sur les API REST à utiliser avec votre inscription Azure Enterprise, et des explications sur la façon de résoudre les problèmes courants liés aux API REST, consultez [API REST Azure Enterprise](./ea-portal-rest-apis.md).

La page [Contrats et avenants Azure EA](./ea-portal-agreements.md) décrit dans quelle mesure les contrats et avenants Azure EA peuvent affecter votre accès, votre utilisation et vos paiements pour les services Azure.

La page [Place de marché Azure](./ea-azure-marketplace.md) explique comment les clients et partenaires EA peuvent afficher les frais de la Place de marché et activer les achats sur la Place de marché Azure.

Pour obtenir des explications sur les tâches courantes qu’un administrateur EA partenaire effectue dans le portail Azure EA, consultez [Administration du portail Azure EA pour les partenaires](./ea-partner-portal-administration.md).

## <a name="get-started-on-azure-ea---faq"></a>FAQ de prise en main d’Azure EA

Cette section fournit des informations sur les questions types posées par les clients au cours du processus d’intégration.  

### <a name="i-accidentally-associated-my-existing-azure-account-with-azure-ea-enrollment-as-a-result-i-lost-my-monthly-credit-can-i-get-my-monthly-credit-back"></a>J’ai accidentellement associé mon compte Azure existant à l’inscription Azure EA. Par conséquent, j’ai perdu mon crédit mensuel. Puis-je récupérer mon crédit mensuel ?

Si vous vous êtes connecté en tant que propriétaire d’un compte Azure EA avec les mêmes informations d’identification que pour votre abonnement Visual Studio, vous pouvez récupérer votre avantage Azure d’abonnement Visual Studio en effectuant l’une des actions suivantes :

- Supprimez le propriétaire de votre compte du portail Azure Enterprise après avoir supprimé ou déplacé des abonnements Azure associés. Inscrivez-vous ensuite à nouveau aux avantages individuels de Visual Studio Azure.
- Supprimez l’abonné Visual Studio du site d’administration dans VLSC et réaffectez l’abonnement à un compte, cette fois avec des informations d’identification différentes. Inscrivez-vous ensuite à nouveau aux avantages individuels de Visual Studio Azure.

### <a name="what-type-of-subscription-should-i-create"></a>Quel type d’abonnement devrais-je créer ?

Le portail Azure Enterprise offre deux types d’abonnements pour les clients d’entreprise :

- Microsoft Azure Enterprise, qui est idéal pour :
  - Toutes les utilisations de production
  - Meilleurs prix basés sur les dépenses d’infrastructure

  Pour plus d'informations, [contactez l’équipe commerciale Azure](https://azure.microsoft.com/pricing/enterprise-agreement/).

- Enterprise Dev/Test, qui est idéal pour :
  - Toutes les charges de travail de développement/test d'équipe
  - Charges de travail de test/développement individuelles moyennes à fortes
  - Accès aux images MSDN spéciales et aux tarifs de service préférentiels

  Pour plus d’informations, consultez [l’offre Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/).

### <a name="my-subscription-name-is-the-same-as-the-offer-name-should-i-change-the-subscription-name-to-something-meaningful-to-my-organization"></a>Mon nom d’abonnement est le même que le nom de l’offre. Dois-je remplacer le nom de l’abonnement par un nom significatif pour mon organisation ?

Lorsque vous créez un abonnement, le nom par défaut est fonction du type d’offre que vous choisissez. Nous vous recommandons de remplacer le nom d’abonnement par un nom qui en facilite le suivi.

Pour modifier le nom :

1. Connectez-vous à [https://account.windowsazure.com](https://account.windowsazure.com).
1. Sélectionnez la liste des abonnements.
1. Sélectionnez l’abonnement que vous souhaitez modifier.
1. Sélectionnez l’icône **Gérer l’abonnement**.
1. Modifiez les détails de l’abonnement.

### <a name="how-can-i-track-costs-incurred-by-a-cost-center"></a>Comment puis-je suivre les coûts engendrés par le centre de coûts ?

Pour suivre les coûts par centre de coût, vous devez définir le centre de coûts à l’un des niveaux suivants :

- department
- Compte
- Abonnement

En fonction de vos besoins, vous pouvez utiliser le même centre de coûts pour suivre l’utilisation et les coûts associés à un centre de coûts particulier.

Par exemple, pour suivre le coût d’un projet spécial dans lequel plusieurs services sont impliqués, vous souhaitez peut-être définir le centre de coûts au niveau de l’abonnement pour suivre l’utilisation et les coûts.

Vous ne pouvez pas définir un centre de coûts au niveau du service. Si vous souhaitez suivre l’utilisation au niveau du service, vous pouvez utiliser la fonctionnalité _Balise_ disponible au niveau du service.

### <a name="how-do-i-track-usage-and-spend-by-different-departments-in-my-organization"></a>Comment faire suivre l’utilisation et les dépenses de différents services dans mon organisation ?

Vous pouvez créer autant de services que nécessaire dans le cadre de votre inscription Azure EA. Pour suivre correctement l’utilisation, assurez-vous que plusieurs services ne se partagent pas les abonnements.

Après avoir créé des services et des abonnements, vous pouvez voir les données dans le rapport d’utilisation. Ces informations peuvent vous aider à suivre l’utilisation ainsi qu’à gérer les coûts et les dépenses au niveau du service.

Vous pouvez également accéder aux données d’utilisation par le biais de l’API de création de rapports. Pour obtenir des informations détaillées et des exemples de code, consultez [API REST Azure Enterprise](./ea-portal-rest-apis.md).

### <a name="can-i-set-a-spending-quota-and-get-alerts-as-i-approach-my-limit"></a>Puis-je définir un quota de dépenses et recevoir des alertes lorsque je m’approche de ma limite ?

Vous pouvez définir un quota de dépense au niveau du service. Le système vous informera automatiquement lorsque vos limites de dépense atteindront 50 %, 75 %, 90 % et 100 % du quota que vous avez défini.

Pour définir votre quota de dépenses, sélectionnez un service, puis sélectionnez l’icône de modification. Après avoir modifié les détails de la limite de dépenses, sélectionnez **Enregistrer**.

### <a name="i-used-resource-groups-to-implement-rbac-and-track-usage-how-can-i-view-the-associated-usage-details"></a>J’ai utilisé des groupes de ressources pour implémenter le RBAC et suivre l’utilisation. Comment puis-je afficher les détails d’utilisation associés ?

Si vous utilisez des _groupes de ressources_ et des _balises_, ces informations sont suivies au niveau du service, et vous pouvez y accéder dans le fichier de téléchargement d’utilisation détaillé (CSV). Consultez le [rapport d’utilisation à télécharger](https://ea.azure.com/report/downloadusage) dans le portail Azure Enterprise.

Vous pouvez également accéder à l’utilisation via l’API. Pour obtenir des informations détaillées et des exemples de code, consultez [API REST Azure Enterprise](./ea-portal-rest-apis.md).

> [!NOTE]
> Vous ne pouvez appliquer des balises qu’à des ressources qui prennent en charge les opérations Azure Resource Manager. Si vous avez créé une machine virtuelle, un réseau virtuel ou un stockage par le biais du modèle de déploiement classique (tel que via le portail Azure classique), vous ne pouvez pas appliquer de balise à cette ressource. Vous devez redéployer ces ressources via Resource Manager pour prendre en charge le balisage. Toutes les autres ressources prennent en charge le balisage.

### <a name="can-i-perform-analyses-using-power-bi"></a>Puis-je effectuer des analyses à l’aide de Power BI ?

Oui. Avec le Pack de contenu Microsoft Azure Enterprise pour Power BI, vous pouvez effectuer les opérations suivantes :

- Importer et analyser rapidement la consommation Azure pour l’inscription de votre entreprise.
- Déterminer le service, le compte ou l’abonnement dont la consommation est la plus élevée.
- Découvrir le service le plus utilisé par votre organisation.
- Suivre les dépenses et les tendances d’utilisation.

Pour utilisez Power BI :

1. rendez-vous sur le site web de Power BI.
1. Connectez-vous avec un compte professionnel ou scolaire valide.

   Le compte professionnel ou scolaire peut être soit identique à celui utilisé pour accéder à l’inscription via le portail Azure Enterprise, soit différent.
1. Dans le tableau de bord des services, choisissez la vignette Microsoft Azure Enterprise et sélectionnez **Connecter**.
1. Sur l’écran **Se connecter à Azure Enterprise**, entrez les informations suivantes :
    - URL d’environnement Azure : [https://ea.azure.com](https://ea.azure.com)
    - Nombre de mois : entre 1 et 36
    - Numéro d’inscription : votre numéro d’inscription
1. Sélectionnez **Suivant**.
1. Entrez la clé API dans la zone **Clé de compte**.

   Vous trouverez la clé API dans le portail Azure Enterprise. Regardez sous l’onglet **Télécharger l’utilisation**, puis sélectionnez **Clé d’accès API**. Copiez la clé, puis collez-la dans la zone **Clé de compte** dans Power BI.

Selon la taille du jeu de données, le chargement des données peut prendre entre 5 et 30 minutes dans Power BI.

La création de rapport de Power BI est disponible pour les clients Azure EA directs, les partenaires et les clients indirects qui sont en mesure d’afficher les informations de facturation.

## <a name="next-steps"></a>Étapes suivantes

- Les administrateurs du portail Azure Enterprise doivent lire la [documentation consacrée à l’administration du portail Azure Enterprise](ea-portal-administration.md) pour découvrir les tâches d’administration courantes.
- Si vous avez besoin d’aide pour résoudre des problèmes rencontrés avec le portail Azure Enterprise, consultez [Résoudre les problèmes d’accès au portail Azure Enterprise](ea-portal-troubleshoot.md).