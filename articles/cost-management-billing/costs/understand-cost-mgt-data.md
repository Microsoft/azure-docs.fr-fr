---
title: Présentation des données d’Azure Cost Management
description: Cet article vous aide à mieux comprendre les données incluses dans Azure Cost Management et la fréquence à laquelle elles sont traitées, collectées, affichées et fermées.
author: bandersmsft
ms.author: banders
ms.date: 10/26/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.custom: contperf-fy21q2
ms.openlocfilehash: 97ae2ba26818bbc306da71af814d9b4f95858b6a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032573"
---
# <a name="understand-cost-management-data"></a>Comprendre les données Cost Management

Cet article vise à détailler les informations sur l’utilisation et les coûts incluses dans Azure Cost Management. Il indique la fréquence à laquelle les données sont traitées, collectées, affichées et fermées. L’utilisation d’Azure vous est facturée tous les mois. Bien que les cycles de facturation soient mensuels, les dates de début et de fin d’un cycle varient selon le type d’abonnement. La fréquence à laquelle Cost Management reçoit les données d’utilisation varie selon différents facteurs. Ces facteurs comprennent le temps nécessaire pour traiter les données et la fréquence à laquelle les services Azure envoient les données d’utilisation au système de facturation.

Azure Cost Management inclut toutes les informations sur les activités et achats, y compris les réservations et les offres tierces associées aux comptes des Contrats entreprise. Les comptes des Contrats client Microsoft et les abonnements individuels avec des taux de paiement à l’utilisation portent uniquement sur l’utilisation des services de Place de marché et Azure. Les coûts du support et les autres coûts ne sont pas inclus. Les coûts font l’objet d’une estimation tant qu’aucune facture n’a été générée et ne tiennent pas compte des crédits.

Si vous disposez d’un nouvel abonnement, vous ne pouvez pas utiliser les fonctionnalités de Cost Management tout de suite. Vous risquez de devoir attendre jusqu’à 48 heures avant de pouvoir utiliser toutes les fonctionnalités de Cost Management.

## <a name="supported-microsoft-azure-offers"></a>Offres Microsoft Azure prises en charge

Les informations suivantes présentent les [offres Microsoft Azure](https://azure.microsoft.com/support/legal/offer-details/) actuellement prises en charge dans Azure Cost Management. Une offre Azure correspond au type d’abonnement Azure que vous avez. Les données sont disponibles dans Cost Management à compter de la date indiquée dans **Données disponibles à partir de/du**. Si les offres d’un abonnement changent, les coûts avant la date de modification de l’offre ne sont pas disponibles.

| **Catégorie**  | **Nom de l’offre** | **ID du quota** | **Numéro de l’offre** | **Données disponibles à partir de/du** |
| --- | --- | --- | --- | --- |
| **Azure Government** | Azure Government Entreprise                                                         | EnterpriseAgreement_2014-09-01 | MS-AZR-USGOV-0017P | Mai 2014<sup>1</sup> |
| **Contrat Entreprise (EA)** | Enterprise Dev/Test                                                        | MSDNDevTest_2014-09-01 | MS-AZR-0148P | Mai 2014<sup>1</sup> |
| **Contrat Entreprise (EA)** | Microsoft Azure Enterprise | EnterpriseAgreement_2014-09-01 | MS-AZR-0017P | Mai 2014<sup>1</sup> |
| **Contrat client Microsoft** | Offre Microsoft Azure | EnterpriseAgreement_2014-09-01 | N/A | Mars 2019<sup>3</sup> |
| **Contrat client Microsoft** | Offre Microsoft Azure pour Dev/Test | MSDNDevTest_2014-09-01 | N/A | Mars 2019<sup>3</sup> |
| **Contrat client Microsoft pris en charge par les partenaires** | Offre Microsoft Azure | CSP_2015-05-01, CSP_MG_2017-12-01 et CSPDEVTEST_2018-05-01<br><br>L’ID de quota est réutilisé pour le contrat client Microsoft et les abonnements CSP hérités. Actuellement, seuls les abonnements aux contrats clients Microsoft sont pris en charge. | N/A | 2 octobre 2019 |
| **MSDN (Microsoft Developer Network)** | Plateformes MSDN<sup>4</sup> | MSDN_2014-09-01 | MS-AZR-0062P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Pay-As-You-Go                  | PayAsYouGo_2014-09-01 | MS-AZR-0003P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Dev/Test - Paiement à l’utilisation         | MSDNDevTest_2014-09-01 | MS-AZR-0023P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Microsoft Partner Network      | MPN_2014-09-01 | MS-AZR-0025P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Version d’évaluation gratuite<sup>4</sup>         | FreeTrial_2014-09-01 | MS-AZR-0044P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Azure dans Open<sup>4</sup>      | AzureInOpen_2014-09-01 | MS-AZR-0111P | 2 octobre 2018<sup>2</sup> |
| **Paiement à l’utilisation** | Pass Azure<sup>4</sup>                                                            | AzurePass_2014-09-01 | MS-AZR-0120P, MS-AZR-0122P - MS-AZR-0125P, MS-AZR-0128P - MS-AZR-0130P | 2 octobre 2018<sup>2</sup> |
| **Visual Studio** | Visual Studio Enterprise – MPN<sup>4</sup>     | MPN_2014-09-01 | MS-AZR-0029P | 2 octobre 2018<sup>2</sup> |
| **Visual Studio** | Visual Studio Professional<sup>4</sup>         | MSDN_2014-09-01 | MS-AZR-0059P | 2 octobre 2018<sup>2</sup> |
| **Visual Studio** | Visual Studio Test Professional<sup>4</sup>    | MSDNDevTest_2014-09-01 | MS-AZR-0060P | 2 octobre 2018<sup>2</sup> |
| **Visual Studio** | Visual Studio Enterprise<sup>4</sup>           | MSDN_2014-09-01 | MS-AZR-0063P | 2 octobre 2018<sup>2</sup> |
| **Visual Studio** | Visual Studio Enterprise : BizSpark<sup>4</sup> | MSDN_2014-09-01 | MS-AZR-0064P | 2 octobre 2018<sup>2</sup> |

_<sup>**1**</sup> Pour les données antérieures à mai 2014, visitez [Azure Enterprise Portal](https://ea.azure.com)._

_<sup>**2**</sup>Pour les données antérieures au 2 octobre 2018, visitez le [Centre des comptes Azure](https://account.azure.com/subscriptions)._

_<sup>**3**</sup> Les Contrats client Microsoft ont commencé en mars 2019 et ne disposent pas de données historiques préalables à cette date._

_<sup>**4**</sup> Les données historiques des abonnements basés sur le crédit et avec règlement à l’avance peuvent ne pas correspondre à votre facture. Consultez [Les données historiques peuvent ne pas correspondre à la facture](#historical-data-might-not-match-invoice) ci-dessous._

Les offres suivantes ne sont pas encore prises en charge :

| Category  | **Nom de l’offre** | **ID du quota** | **Numéro de l’offre** |
| --- | --- | --- | --- |
| **Azure Allemagne** | Azure Allemagne - Paiement à l'utilisation | PayAsYouGo_2014-09-01 | MS-AZR-DE-0003P |
| **Azure Government** | Azure Government – Paiement à l’utilisation | PayAsYouGo_2014-09-01 | MS-AZR-USGOV-0003P |
| **Fournisseur de solutions cloud (CSP)** | Microsoft Azure                                    | CSP_2015-05-01 | MS-AZR-0145P |
| **Fournisseur de solutions cloud (CSP)** | Azure Government CSP                               | CSP_2015-05-01 | MS-AZR-USGOV-0145P |
| **Fournisseur de solutions cloud (CSP)** | Azure Germany en tant que fournisseur CSP pour Microsoft Cloud Germany   | CSP_2015-05-01 | MS-AZR-DE-0145P |
| **Paiement à l’utilisation**                 | Kit de démarrage Azure pour les étudiants | DreamSpark_2015-02-01 | MS-AZR-0144P |
| **Paiement à l’utilisation** | Azure for Students<sup>4</sup> | AzureForStudents_2018-01-01 | MS-AZR-0170P |
| **Paiement à l’utilisation**                 | Microsoft Azure Sponsorship | Sponsored_2016-01-01 | MS-AZR-0036P |
| **Plans de support** | Support standard                    | Default_2014-09-01 | MS-AZR-0041P |
| **Plans de support** | Support Professional Direct         | Default_2014-09-01 | MS-AZR-0042P |
| **Plans de support** | Developer Support                   | Default_2014-09-01 | MS-AZR-0043P |
| **Plans de support** | Plan de support Allemagne                | Default_2014-09-01 | MS-AZR-DE-0043P |
| **Plans de support** | Support standard Azure Government   | Default_2014-09-01 | MS-AZR-USGOV-0041P |
| **Plans de support** | Support Professional Direct Azure Government | Default_2014-09-01 | MS-AZR-USGOV-0042P |
| **Plans de support** | Developer Support Azure Government  | Default_2014-09-01 | MS-AZR-USGOV-0043P |

### <a name="free-trial-to-pay-as-you-go-upgrade"></a>Passer d’un essai gratuit au paiement à l’utilisation

Pour des informations sur la disponibilité des services de niveau gratuit après être passé d’un essai gratuit au paiement à l’utilisation, consultez les [Questions fréquentes (FAQ) sur les comptes gratuits Azure](https://azure.microsoft.com/free/free-account-faq/).

### <a name="determine-your-offer-type"></a>Déterminer le type de votre offre

Si vous ne voyez pas de données pour un abonnement et que vous souhaitez déterminer si votre abonnement figure dans les offres prises en charge, vous pouvez valider que votre abonnement est pris en charge. Pour valider la prise en charge d’un abonnement Azure, connectez-vous au portail Azure. Sélectionnez ensuite **Tous les services** dans le volet du menu de gauche. Dans la liste des services, sélectionnez **Abonnements**. Dans le menu de la liste des abonnements, sélectionnez l’abonnement que vous souhaitez vérifier. Votre abonnement est affiché sous l’onglet Vue d’ensemble et vous pouvez voir les éléments **Offre** et **ID de l’offre**. L’image suivante en montre un exemple.

![Exemple de l’onglet Vue d’ensemble de l’abonnement montrant l’offre et l’ID de l’offre](./media/understand-cost-mgt-data/offer-and-offer-id.png)

## <a name="costs-included-in-cost-management"></a>Coûts inclus dans Cost Management

Les tableaux suivants indiquent les données qui sont comprises ou non dans Cost Management. Les coûts font l’objet d’une estimation tant qu’aucune facture n’a été générée. Les coûts affichés n’incluent pas les crédits prépayés et gratuits.

| **Incluses** | **Non incluses** |
| --- | --- |
| Utilisation des services Azure<sup>5</sup>        | Frais de support. Pour plus d’informations, consultez [Conditions de facturation expliquées](../understand/understand-invoice.md). |
| Utilisation des produits de la Place de marché<sup>6</sup> | Taxes. Pour plus d’informations, consultez [Conditions de facturation expliquées](../understand/understand-invoice.md). |
| Achats de la Place de marché<sup>6</sup>      | Crédits. Pour plus d’informations, consultez [Conditions de facturation expliquées](../understand/understand-invoice.md). |
| Achats de réservation<sup>7</sup>      |  |
| Amortissement des achats de réservation<sup>7</sup>      |  |

_<sup>**5**</sup> L’utilisation des services Azure est basée sur la réservation et des prix négociés._

_<sup>**6**</sup> Les achats de la Place de marché ne sont pas disponibles pour les offres MSDN et Visual Studio à l’heure actuelle._

_<sup>**7**</sup> Les achats de réservation sont uniquement disponibles pour les comptes Contrat Entreprise et Contrat client Microsoft à l’heure actuelle._

## <a name="how-tags-are-used-in-cost-and-usage-data"></a>Utilisation des étiquettes dans les données de coût et d’utilisation

Azure Cost Management reçoit des étiquettes dans le cadre de chaque enregistrement d’utilisation soumis par les services individuels. Les contraintes suivantes s’appliquent à ces étiquettes :

- Les étiquettes doivent être appliquées directement aux ressources et ne sont pas héritées implicitement du groupe de ressources parent.
- Les étiquettes de ressource sont prises en charge seulement pour les ressources déployées sur des groupes de ressources.
- Certaines ressources déployées peuvent ne pas prendre en charge les étiquettes ou ne pas inclure d’étiquettes dans les données d’utilisation.
- Les étiquettes de ressource sont incluses seulement dans les données d’utilisation quand l’étiquette est appliquée : les étiquettes ne sont pas appliquées aux données d’historique.
- Les étiquettes de ressource sont disponibles dans Cost Management seulement une fois que les données sont actualisées.
- Les étiquettes de ressource sont disponibles dans Cost Management seulement quand la ressource est active/en cours d’exécution et produit des enregistrements d’utilisation (par exemple pas quand une machine virtuelle est désallouée).
- La gestion des étiquettes nécessite un accès Contributeur à chaque ressource.
- La gestion des stratégies d’étiquettes nécessite l’accès Propriétaire ou Contributeur de stratégie à un groupe d’administration, un abonnement ou un groupe de ressources.
    
Si vous ne voyez pas une étiquette spécifique dans Cost Management, prenez en considération les éléments suivants :

- L’étiquette a-t-elle été appliquée directement à la ressource ?
- L’étiquette a-t-elle été appliquée il y a plus de 24 heures ?
- Le type de ressource prend-il en charge les étiquettes ? Les types de ressources suivants ne prennent pas en charge les étiquettes dans les données d’utilisation à la date du 1er décembre 2019. Pour obtenir la liste complète de ce qui est pris en charge, consultez [Prise en charge des étiquettes pour les ressources Azure](../../azure-resource-manager/management/tag-support.md).
    - Annuaires Azure Active Directory B2C
    - Azure Bastion
    - Pare-feux Azure
    - Azure NetApp Files
    - Data Factory
    - Databricks
    - Équilibreurs de charge
    - Network Watcher
    - Notification Hubs
    - Service Bus
    - Time Series Insights
    
Voici quelques conseils pour l’utilisation des étiquettes :

- Planifiez en amont et définissez une stratégie d’étiquetage qui vous permet de ventiler les coûts par organisation, application, environnement, etc.
- Utilisez Azure Policy pour copier les étiquettes des groupes de ressources vers des ressources individuelles et pour appliquer votre stratégie d’étiquetage.
- Utilisez l’API Tags conjointement avec Query ou UsageDetails pour obtenir tous les coûts en fonction des étiquettes actuelles.


## <a name="cost-and-usage-data-updates-and-retention"></a>Mises à jour et rétention des données de coût et d’utilisation

Les données de coût et d’utilisation sont généralement disponibles dans Gestion des coûts + facturation dans le Portail Azure et les API de prise en charge dans les 8 à 24 heures. Gardez à l’esprit les points suivants quand vous passez en revue les coûts :

- Chaque service Azure (tel que Stockage, Compute et SQL) émet une utilisation à différents intervalles. Vous pouvez voir des données pour certains services plus tôt que d’autres.
- Les frais estimés pour la période de facturation en cours sont mis à jour six fois par jour.
- Les frais estimés pour la période de facturation en cours peuvent changer au fur et à mesure que votre utilisation augmente.
- Chaque mise à jour est cumulative et inclut tous les éléments de ligne et les informations de la mise à jour précédente.
- Azure finalise ou _ferme_ la période de facturation en cours jusqu’à 72 heures (trois jours calendaires) après la fin de la période de facturation.

Les exemples suivants illustrent la façon dont les périodes de facturation peuvent se terminer :

* Abonnements Contrat Entreprise (EA). Si le mois de facturation se termine le 31 mars, les frais estimés sont mis à jour jusqu’à 72 heures plus tard. Dans cet exemple, à minuit (UTC) le 4 avril.
* Abonnements avec paiement à l’utilisation. Si le mois de facturation se termine le 15 mai, les frais estimés peuvent être mis à jour jusqu’à 72 heures plus tard. Dans cet exemple, à minuit (UTC) le 19 mai.

Une fois les données de coût et d’utilisation disponibles dans Cost Management + facturation, elles sont conservées pendant au moins 7 ans.

### <a name="rerated-data"></a>Données réestimées

Que vous utilisiez les API Cost Management, Power BI ou le portail Azure pour récupérer des données, prévoyez une réestimation des frais de la période de facturation en cours, et donc des changements, jusqu’à la clôture de la facture.

## <a name="cost-rounding"></a>Arrondi

Les coûts indiqués dans Cost Management sont arrondis. Les coûts retournés par l’API de requête ne le sont pas. Par exemple :

- Analyse des coûts sur le portail Azure – Les frais sont arrondis selon les règles d’arrondi standard : les valeurs avec une partie décimale de 0,5 et plus sont arrondies par excès. Sinon, les coûts sont arrondis par défaut. L’arrondi se produit uniquement au moment où les valeurs sont affichées. Il ne se produit pas pendant le traitement et l’agrégation de données. Par exemple, l’analyse des coûts agrège les coûts comme suit :
  -    Frais 1 : 0,004 USD
  - Frais 2 : 0,004 USD
  -    Rendu des frais agrégés : 0,004 + 0.004 = 0,008. Les frais affichés sont de 0,01 USD.
- API de requête – Les frais affichés présentent huit décimales et ne sont pas arrondis.

## <a name="historical-data-might-not-match-invoice"></a>Les données historiques peuvent ne pas correspondre à la facture

Les données historiques des offres basées sur le crédit et avec règlement à l’avance peuvent ne pas correspondre à votre facture. Certaines offres de paiement à l'utilisation, MSDN et Visual Studio peuvent entraîner l’application de crédits et paiements avancés Azure à la facture. Toutefois, les données historiques indiquées dans Azure Cost Management sont uniquement basées sur les frais de consommation estimés. Elles n’incluent pas les paiements et les crédits. Les données historiques indiquées pour les offres suivantes peuvent donc ne pas correspondre exactement à votre facture.

- Azure for Students (MS-AZR-0170P)
- Azure dans Open (MS-AZR-0111P)
- Pass Azure (MS-AZR-0120P, MS-AZR-0123P, MS-AZR-0125P, MS-AZR-0128P, MS-AZR-0129P)
- Essai gratuit (MS-AZR-0044P)
- MSDN (MS-AZR-0062P)
- Visual Studio (MS-AZR-0029P, MS-AZR-0059P, MS-AZR-0060P, MS-AZR-0063P, MS-AZR-0064P)

## <a name="see-also"></a>Voir aussi

- Si vous n’avez pas encore effectué le premier guide de démarrage rapide relatif à Cost Management, lisez-le à partir de [Démarrer l’analyse des coûts](./quick-acm-cost-analysis.md).