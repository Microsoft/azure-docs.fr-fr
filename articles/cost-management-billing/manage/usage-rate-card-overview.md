---
title: Obtenir l’utilisation d’Azure avec API Facturation Azure
description: Découvrez les API Azure Billing Usage et RateCard, qui fournissent des vues d’ensemble des tendances et de la consommation des ressources Azure.
author: tonguyen
ms.reviewer: mumami
tags: billing
ms.service: cost-management-billing
ms.subservice: common
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: b3ae2b8323c9f278dcec432dfaac05e9fcfb4b49
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132106"
---
# <a name="use-azure-billing-apis-to-programmatically-get-insight-into-your-azure-usage"></a>Utilisez les API de facturation Azure pour obtenir par programme des informations sur votre utilisation d’Azure
Utilisez les API de facturation Azure pour extraire les données d’utilisation et de ressources dans vos outils d’analyse de données préférés. Les API d’utilisation des ressources Azure et RateCard peuvent vous aider à prévoir vos coûts avec précision et à les gérer. Les API sont implémentées en tant que fournisseur de ressources et font partie intégrante de la famille d’API exposées par Azure Resource Manager.  

## <a name="azure-invoice-download-api-preview"></a>API Azure Invoice Download (version préliminaire)
Une fois l’[adhésion terminée](manage-billing-access.md#opt-in), téléchargez des factures à l’aide de la version préliminaire de l’[API Invoice](/rest/api/billing). Cette API offre les fonctionnalités suivantes :

* **Contrôle d’accès en fonction du rôle Azure (RBAC Azure)**  : configurez des stratégies d’accès sur le [portail Azure](https://portal.azure.com) ou via des [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications pouvant accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
* **Filtrage par date** : utilisez le paramètre `$filter` pour obtenir toutes les factures par ordre chronologique inverse à partir de la date de fin de la période de facturation.

> [!NOTE]
> Cette fonctionnalité se trouve dans la première version de la version préliminaire et peut être sujette à des modifications à incompatibilité descendante. Elle n’est actuellement pas disponible pour certaines offres d’abonnement (EA, CSP, AIO non pris en charge) et Azure Allemagne.

## <a name="azure-resource-usage-api-preview"></a>API Azure Resource Usage (version préliminaire)
Utiliser l’[API Azure Resource Usage](/previous-versions/azure/reference/mt219003(v=azure.100)) pour obtenir une estimation de vos données de consommation Azure. L’API comprend :

* **Contrôle d’accès en fonction du rôle Azure (RBAC Azure)**  : configurez des stratégies d’accès sur le [portail Azure](https://portal.azure.com) ou via des [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications pouvant accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
* **Agrégations horaires ou quotidiennes** : les appelants peuvent indiquer s’ils souhaitent visualiser leurs données d’utilisation Azure par intervalles de temps horaires ou quotidiens. Par défaut, les données sont présentées par jour.
* **Métadonnées d’instance (balises de ressource incluses)**  : obtenez des détails de niveau instance, comme l’URI de ressource complet (/subscriptions/{subscription-id}/..) et des informations de groupe de ressources et les balises de ressource. Les métadonnées vous aideront à allouer de façon déterministe et par programme l’utilisation par les balises, pour les cas d’utilisation comme la facturation interne.
* **Métadonnées sur les ressources** : des détails sur les ressources, comme le nom du compteur, la catégorie du compteur, la sous-catégorie du compteur, l’unité et la région offrent à l’appelant une meilleure compréhension des ressources consommées. Nous nous efforçons également d’aligner la terminologie des métadonnées de ressource dans l’ensemble du portail Azure, des fichiers CSV sur l’utilisation d’Azure et sur la facturation Contrat Entreprise (EA), ainsi que dans toutes les autres expériences destinées au public, afin de vous permettre de mettre en corrélation les données des différentes expériences.
* **Utilisation pour différents types d’offre** : les données d’utilisation sont accessibles pour tous les types d’offre, tels que Paiement à l’utilisation, MSDN, Engagement monétaire, Crédit monétaire et Contrat Entreprise (EA), excepté [CSP](/partner-center).

## <a name="azure-resource-ratecard-api-preview"></a>API Azure Resource RateCard (version préliminaire)
Utilisez l’[API Azure Resource RateCard](/previous-versions/azure/reference/mt219005(v=azure.100)) pour obtenir la liste des ressources Azure disponibles et une estimation des informations de prix pour chacune de ces ressources. L’API comprend :

* **Contrôle d’accès en fonction du rôle Azure (RBAC Azure)**  : configurez vos stratégies d’accès sur le [portail Azure](https://portal.azure.com) ou via des [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou les applications qui peuvent accéder aux données de RateCard. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure particulier.
* **Prise en charge des offres Paiement à l’utilisation, MSDN, Engagement monétaire et Crédit monétaire (offre EA et [CSP](/partner-center) non prises en charge)**  : cette API fournit des informations de tarif au niveau des offres Azure.  L’appelant de cette API doit transmettre les informations d’offre pour obtenir les détails et les tarifs des ressources. Nous sommes actuellement pas en mesure de fournir les tarifs EA, car les offres EA présentent des tarifs par inscription personnalisés.

## <a name="scenarios"></a>Scénarios
Voici quelques-uns des scénarios autorisés par l’utilisation combinée des API Usage et RateCard :

* **Dépenses Azure au cours du mois** : utilisez la combinaison des API d’utilisation et RateCard pour obtenir une meilleure idée de vos dépenses de cloud au cours du mois. Vous pouvez analyser les compartiments horaires et quotidiens des estimations d’utilisation et de facturation.
* **Configurer des alertes** : utilisez les API d’utilisation et RateCard pour obtenir une estimation de la consommation et de la facturation, et configurer des alertes en fonction des ressources ou des valeurs monétaires.
* **Prédiction de facture** : obtenez votre estimation de consommation et votre dépense cloud et appliquez des algorithmes d’apprentissage automatique pour prédire le montant de la facture à la fin du cycle de facturation.
* **Analyse des coûts avant consommation** : utilisez l’API RateCard pour prédire le montant de votre facture pour votre utilisation prévue lorsque vous déplacez vos charges de travail dans Azure. Si vous disposez de charges de travail existantes dans d’autres clouds publics ou privés, vous pouvez également mapper votre utilisation sur les tarifs Azure afin d’obtenir une meilleure estimation de vos dépenses Azure. Cette estimation vous donne la possibilité d’ajouter un tableau croisé dynamique sur l’offre et de comparer les différents types d’offre au-delà du paiement à l’utilisation, notamment l’engagement monétaire et crédit monétaire. L’API vous donne également la possibilité de voir les différences de coût par région et vous permet d’effectuer une analyse de scénarios des coûts pour vous aider à prendre des décisions en matière de déploiement.
* **Analyse de scénarios** -

  * Vous pouvez déterminer s’il est plus rentable d’exécuter les charges de travail dans une autre région ou dans une autre configuration de la ressource Azure. Les coûts des ressources Azure peuvent varier en fonction de la région Azure que vous utilisez.
  * Vous pouvez également déterminer si un autre type d’offre Azure propose un meilleur tarif pour une ressource Azure.


## <a name="next-steps"></a>Étapes suivantes
* Consultez les exemples de code sur GitHub :
  * [Exemple de code de l’API Invoice](https://go.microsoft.com/fwlink/?linkid=845124)

  * [Exemple de code de l’API Usage](https://github.com/Azure-Samples/billing-dotnet-usage-api)

  * [Exemple de code de l’API RateCard](https://github.com/Azure-Samples/billing-dotnet-ratecard-api)

* Pour en savoir plus sur Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../../azure-resource-manager/management/overview.md) .