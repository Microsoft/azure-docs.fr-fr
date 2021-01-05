---
title: Présentation des API Azure Consumption
description: Découvrez comment les API Azure Consumption vous permettent d’accéder par programme aux données de coût et d’utilisation de vos ressources Azure.
author: bandersmsft
tags: billing
ms.service: cost-management-billing
ms.subservice: cost-management
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: b00b5a46a0cdeb95c357da521326fe2bb678490b
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132616"
---
# <a name="azure-consumption-api-overview"></a>Présentation des API Azure Consumption

Les API Azure Consumption vous permettent d’accéder par programme aux données de coût et d’utilisation de vos ressources Azure. Pour le moment, ces API ne prennent en charge que les abonnements de type Accord de Mise en Œuvre Entreprise et Web direct (à quelques exceptions près). Ces API sont continuellement mises à jour pour prendre en charge d’autres types d’abonnements Azure.

Les API Azure Consumption donnent accès aux éléments suivants :
- Clients Entreprise en Web Direct
    - Usage Details
    - Marketplace Charges
    - Reservation Recommendations
    - Reservation Details
    - Reservation Summaries
- Clients Entreprise uniquement
    - Price sheet
    - Budgets
    - Balances

## <a name="usage-details-api"></a>API Usage Details

L’API Usage Details permet d’obtenir les données de coût et d’utilisation de toutes les ressources internes Azure. Les informations sont sous la forme d’enregistrements détaillés de l’utilisation, qui sont actuellement émis une fois par jour par compteur et par ressource. Les informations peuvent être utilisées pour ajouter les coûts pour toutes les ressources ou pour examiner les coûts/l’utilisation de ressources spécifiques.

L’API comprend :

-   **Données de consommation au niveau du compteur** : consultez les données, notamment le coût d’utilisation, le compteur d’émission des frais et la ressource Azure associée aux frais. Tous les enregistrements de détail de l’utilisation sont mappés à un compartiment quotidien.
-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Filtrage** : affinez vos résultats d’API pour obtenir un ensemble plus petit d’enregistrements des détails d’utilisation à l’aide des filtres suivants :
    - Début/Fin de l’utilisation
    - Groupe de ressources
    - Nom de la ressource
-   **Agrégation de données** : utilisez le protocole OData pour appliquer des expressions aux détails d’utilisation d’agrégation par balises ou propriétés de filtrage.
-   **Utilisation pour différents types d’offre** : les informations détaillées de l’utilisation sont actuellement disponibles pour les clients Entreprise et Web Direct.

Pour plus d’informations, consultez la spécification technique de [l’API Usage Details](/rest/api/consumption/usagedetails).

## <a name="marketplace-charges-api"></a>API Marketplace Charges

Utilisez l’API Marketplace Charges pour obtenir les données d’utilisation et de frais de toutes les ressources de la Place de marché (offres tierces Azure). Ces données peuvent être utilisées pour ajouter les coûts pour toutes les ressources de la Place de marché ou pour examiner les coûts/l’utilisation de ressources spécifiques.

L’API comprend :

-   **Données de consommation au niveau du compteur** : consultez les données, notamment le coût d’utilisation de la Place de marché, le compteur d’émission des frais et la ressource associée aux frais. Tous les enregistrements de détail de l’utilisation sont mappés à un compartiment quotidien.
-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Filtrage** : affinez vos résultats d’API pour obtenir un ensemble plus petit d’enregistrements de la Place de marché à l’aide des filtres suivants :
    - Début/Fin de l’utilisation
    - Groupe de ressources
    - Nom de la ressource
-   **Utilisation pour différents types d’offre** : les informations de la Place de marché sont actuellement disponibles pour les clients Entreprise et Web Direct.

Pour plus d’informations, consultez la spécification technique de [l’API Marketplace Charges](/rest/api/consumption/marketplaces).

## <a name="balances-api"></a>API Balances

Les clients Entreprise peuvent utiliser l’API Balances pour obtenir un résumé mensuel des informations sur les soldes, nouveaux achats, frais de service de la Place de marché Azure, ajustements et dépassement des frais. Vous pouvez obtenir ces informations pour la période de facturation en cours ou n’importe quelle période passée. Les entreprises peuvent utiliser ces données pour les comparer avec les frais résumés calculés. Cette API ne fournit pas d’informations propres à la ressource ni de vue agrégée des coûts.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Clients Entreprise uniquement** : cette API est disponible uniquement pour les clients EA.
    - Les clients doivent disposer des autorisations d’administrateur d’entreprise pour appeler cette API.

Pour plus d’informations, consultez la spécification technique de [l’API Balances](/rest/api/consumption/balances).

## <a name="budgets-api"></a>API Budgets

Les clients Entreprise peuvent utiliser cette API pour créer des budgets de coût ou d’utilisation pour les ressources, les groupes de ressources ou les compteurs de facturation. Une fois ces informations déterminées, la génération d’alertes peut être configurée pour avertir lorsque les seuils de budget définis par l’utilisateur sont atteints.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Clients Entreprise uniquement** : cette API est disponible uniquement pour les clients EA.
-   **Notifications configurables** : indiquez le ou les utilisateurs à informer en cas de dépassement du budget.
-   **Budgets basés sur l’utilisation ou les coûts** : créez votre budget basé sur votre consommation ou vos coûts en fonction de votre situation.
-   **Filtrage** : filtrez votre budget pour obtenir un plus petit sous-ensemble de ressources à l’aide des filtres configurables suivants :
    - Groupe de ressources
    - Nom de la ressource
    - Compteur
-   **Périodes budgétaires configurables** : spécifiez la fréquence à laquelle le budget doit être réinitialisé ainsi que la durée de validité de celui-ci.

Pour plus d’informations, consultez la spécification technique de [l’API Budgets](/rest/api/consumption/budgets).

## <a name="reservation-recommendations-api"></a>API Reservation Recommendations

Utilisez cette API pour obtenir des recommandations sur l’achat d’instances de machine virtuelle réservées. Les recommandations sont conçues pour permettre aux clients d’analyser les économies et les montants d’achat attendus.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Filtrage** : personnalisez les résultats des recommandations à l’aide des filtres suivants :
    - Étendue
    - Période de recherche arrière
-   **Informations de réservation pour différents types d’offre** : les informations de réservation sont actuellement disponibles pour les clients Entreprise et Web Direct.

Pour plus d’informations, consultez la spécification technique de [l’API Reservation Recommendations](/rest/api/consumption/reservationrecommendations).

## <a name="reservation-details-api"></a>API Reservation Details

Utilisez l’API Reservation Details pour afficher des informations sur les réservations de machines virtuelles déjà achetées, comme la quantité de consommation réservée par rapport à la quantité actuellement utilisée. Vous pouvez voir les données au niveau de la machine virtuelle.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Filtrage** : affinez vos résultats d’API pour obtenir un ensemble plus petit de réservations à l’aide du filtre suivant :
    - Plage de dates
-   **Informations de réservation pour différents types d’offre** : les informations de réservation sont actuellement disponibles pour les clients Entreprise et Web Direct.

Pour plus d’informations, consultez la spécification technique de [l’API Reservation Details](/rest/api/consumption/reservationsdetails).

## <a name="reservation-summaries-api"></a>API Reservation Summaries

Utilisez cette API pour afficher des informations agrégées sur les réservations de machines virtuelles déjà achetées, comme la quantité de consommation réservée par rapport à la quantité actuellement utilisée dans l’agrégation.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Filtrage** : personnalisez vos résultats lors de l’utilisation de la base quotidienne avec le filtre suivant :
    - Date d'utilisation
-   **Informations de réservation pour différents types d’offre** : les informations de réservation sont actuellement disponibles pour les clients Entreprise et Web Direct.
-   **Agrégations quotidiennes ou mensuelles** : les appelants peuvent spécifier s’ils veulent que les données de synthèse de leurs réservations soient quotidiennes ou mensuelles.

Pour plus d’informations, consultez la spécification technique de [l’API Reservation Summaries](/rest/api/consumption/reservationssummaries).

## <a name="price-sheet-api"></a>API Price Sheet
Les clients Entreprise peuvent utiliser cette API pour récupérer leur tarification personnalisée pour tous les compteurs. Les entreprises peuvent utiliser cette API en association avec les détails sur l’utilisation et l’utilisation des Places de marché pour effectuer des calculs de coût à l’aide des données relatives à l’utilisation et à la Place de marché.

L’API comprend :

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : configurez les stratégies d’accès dans le [Portail Azure](https://portal.azure.com), [l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md) ou les [cmdlets Azure PowerShell](/powershell/azure/) pour spécifier les utilisateurs ou applications qui peuvent accéder aux données d’utilisation de l’abonnement. Les appelants doivent utiliser les jetons Azure Active Directory standard pour l’authentification. Ajoutez l’appelant au rôle Lecteur de facture, Lecteur, Propriétaire ou Collaborateur pour être en mesure d’accéder aux données d’utilisation relatives à un abonnement Azure spécifique.
-   **Clients Entreprise uniquement** : cette API est disponible uniquement pour les clients EA. Les clients Web Direct doivent utiliser l’API RateCard pour obtenir la tarification.

Pour plus d’informations, consultez la spécification technique de [l’API Price Sheet](/rest/api/consumption/pricesheet).

## <a name="scenarios"></a>Scénarios

Voici quelques scénarios qui sont rendus possibles via les API Consumption :

-   **Rapprochement de facture** : Microsoft m’a-t-il facturé le montant correct ?  Quelle est ma facture et puis-je la calculer moi-même ?
-   **Frais croisés** : maintenant que je sais combien je suis facturé, qui dans mon organisation doit payer ?
-   **Optimisation des coûts** : je connais le montant de ma facture... comment puis-je rentabiliser davantage l’argent que je dépense dans Azure ?
-   **Suivi des coûts** : je souhaite voir combien je dépense et utilise sur Azure au fil du temps. Quelles sont les tendances ? Comment faire mieux ?
-   **Dépenses Azure au cours du mois** : à combien s’élèvent mes dépenses sur Azure pour le mois en cours jusqu’à la date actuelle ? Dois-je procéder à quelques ajustements dans mes dépenses et/ou mon utilisation d’Azure ? Quand, dans le mois, ma consommation d’Azure est-elle la plus élevée ?
-   **Configurer des alertes** : je souhaite configurer des alertes monétaires ou de consommation basées sur les ressources en fonction d’un budget.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’utilisation des API Facturation Azure pour obtenir par programme des insights sur votre utilisation d’Azure, consultez [Présentation de l’API Facturation Azure](usage-rate-card-overview.md).