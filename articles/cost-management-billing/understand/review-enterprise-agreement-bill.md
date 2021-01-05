---
title: Consulter votre facture Contrat Entreprise Azure
description: Découvrez comment lire et comprendre vos données d’utilisation et de facture pour un Contrat Entreprise Azure.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: d707051e20b4ce9d1289557aec6f8256c2e6bfba
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150069"
---
# <a name="understand-your-azure-enterprise-agreement-bill"></a>Comprendre votre facture Contrat Entreprise Azure

Les clients Azure qui ont un Contrat Entreprise reçoivent une facture quand ils dépassent le crédit de l’organisation ou utilisent des services qui ne sont pas couverts par le crédit.

Le crédit de votre organisation comprend votre engagement financier. L’engagement financier correspond au montant que votre organisation a payé à l’avance pour l’utilisation des services Azure. Vous pouvez ajouter des fonds d’engagement financier à votre Contrat Entreprise en contactant votre revendeur ou votre responsable de compte Microsoft.

Ce tutoriel s’applique uniquement aux clients Azure disposant d’un Contrat Entreprise Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Passer en revue les frais facturés.
> * Passer en revue les frais de dépassement de service.
> * Passer en revue la facture de la Place de marché.

## <a name="prerequisites"></a>Prérequis

Pour consulter et vérifier les frais figurant sur votre facture, vous devez être administrateur d’entreprise. Pour plus d’informations, consultez [Comprendre les rôles d’administrateur Contrat Entreprise Azure dans Azure](../manage/understand-ea-roles.md). Si vous ignorez qui est l’administrateur d’entreprise de votre organisation, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="review-invoiced-charges-for-most-customers"></a>Passer en revue les frais facturés pour la plupart des clients

Cette section ne s’applique pas aux clients Azure en Australie, au Japon ou à Singapour.

Vous recevez une facture Azure si l’un des événements suivants se produit au cours de la période de facturation :

- **Dépassement de service** : les frais d’utilisation de votre organisation dépassent votre solde de crédit.
- **Frais facturés séparément** : les services utilisés par votre organisation ne sont pas couverts par le crédit. Les services suivants vous sont facturés, malgré votre solde de crédit :
    - Canonical
    - Citrix XenApp Essentials
    - Citrix XenDesktop
    - Utilisateur inscrit
    - Openlogic
    - Utilisateur inscrit de Remote Access Rights XenApp Essentials
    - Ubuntu Advantage
    - Visual Studio Enterprise (mensuel)
    - Visual Studio Enterprise (annuel)
    - Visual Studio Professional (mensuel)
    - Visual Studio Professional (annuel)
- **Frais de la Place de marché** : les achats et l’utilisation liés à la Place de marché Azure ne sont pas couverts par le crédit de votre organisation. Les frais de la Place de marché vous sont donc facturés, malgré votre solde de crédit. Sur Enterprise Portal, un administrateur d’entreprise peut activer et désactiver les achats liés à la Place de marché.

Votre facture présentera les frais d’utilisation d’Azure avec les coûts associés affichés en premier, suivis des frais relatifs à la Place de marché. Si vous avez un solde créditeur, il s’appliquera à l’utilisation d’Azure et votre facture présentera l’utilisation d’Azure et celle de la Place de marché en dernier, sans aucun coût.

Comparez votre montant total combiné indiqué sur Enterprise Portal dans **Rapports** > **Récapitulatif de l’utilisation** avec votre facture Azure. Les montants dans le **Récapitulatif de l’utilisation** n’incluent pas les taxes.

Connectez-vous au [portail Azure EA](https://ea.azure.com). Ensuite, sélectionnez **Rapports**. En haut à droite de l’onglet, changez de vue en passant de **M** à **C**, et faites correspondre la période sur la facture.  

![Capture d’écran qui montre l’option M + C dans le récapitulatif de l’utilisation.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)

Le montant combiné de l’**Utilisation totale** et de la **Place de marché Azure** doit correspondre au **Total final** sur votre facture. Pour obtenir plus de détails sur vos frais, accédez à **Télécharger l’utilisation**.  

![Capture d’écran montrant l’onglet Télécharger l’utilisation](./media/review-enterprise-agreement-bill/ea-portal-download-usage.png)

## <a name="review-invoiced-charges-for-other-customers"></a>Passer en revue les frais facturés pour d’autres clients

Cette section s’applique uniquement aux clients Azure en Australie, au Japon ou à Singapour.

Vous recevez une ou plusieurs factures Azure dans les cas suivants :

- **Dépassement de service** : les frais d’utilisation de votre organisation dépassent votre solde de crédit.
- **Frais facturés séparément** : les services utilisés par votre organisation ne sont pas couverts par le crédit. Les services suivants vous sont facturés :
    - Canonical
    - Citrix XenApp Essentials
    - Citrix XenDesktop
    - Utilisateur inscrit
    - Openlogic
    - Utilisateur inscrit de Remote Access Rights XenApp Essentials
    - Ubuntu Advantage
    - Visual Studio Enterprise (mensuel)
    - Visual Studio Enterprise (annuel)
    - Visual Studio Professional (mensuel)
    - Visual Studio Professional (annuel)
- **Frais de la Place de marché** : Les achats et l’utilisation liés à la Place de marché Azure ne sont pas couverts par le crédit de votre organisation et sont facturés séparément. Sur Enterprise Portal, un administrateur d’entreprise peut activer et désactiver les achats liés à la Place de marché.

Quand vous avez des frais de dépassements de service et des frais facturés séparément au cours de la période de facturation, vous recevez une facture. Elle inclut les deux types de frais. Les frais de la Place de marché sont toujours facturés séparément.

## <a name="review-service-overage-charges-for-other-customers"></a>Passer en revue les frais de dépassement de service pour d’autres clients

Cette section s’applique uniquement si vous vous trouvez en Australie, au Japon ou à Singapour.

Comparez le montant total de votre utilisation sur Enterprise Portal dans **Rapports** > **Récapitulatif de l’utilisation** avec votre facture de dépassement de service. La facture de dépassement de service inclut l’utilisation qui dépasse le crédit de votre organisation et/ou les services qui ne sont pas couverts par le crédit. Les montants du **Récapitulatif de l’utilisation** n’incluent pas les taxes.

Connectez-vous au [portail Azure EA](https://ea.azure.com), puis sélectionnez **Rapports**. En haut à droite de l’onglet, changez de vue en passant de **M** à **C**, et faites correspondre la période sur la facture.  

![Capture d’écran qui montre l’option M + C dans le récapitulatif de l’utilisation.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)

Le montant **Utilisation totale** doit correspondre au **Total final** de votre facture de dépassement de service. Pour plus d’informations sur vos frais, accédez à **Télécharger l’utilisation** > **Téléchargement de rapport avancé**. Le rapport n’inclut pas les taxes ou frais de réservation, ni les frais de la Place de marché.  

![Capture d’écran montrant le téléchargement de rapport avancé sous l’onglet Téléchargement de l’utilisation.](./media/review-enterprise-agreement-bill/ea-portal-download-usage-advanced.png)

Le tableau suivant liste les termes et descriptions figurant sur la facture ainsi que sur le **Récapitulatif de l’utilisation** dans Enterprise Portal :

|Terme relatif à la facturation|Terme relatif au récapitulatif de l’utilisation|Description|
|---|---|---|
|Total final|Utilisation totale|Total des frais d’utilisation hors taxes pour la période spécifique avant l’application du crédit.|
|Utilisation de l’engagement|Utilisation de l’engagement|Crédit appliqué pendant cette période spécifique.|
|Total de la vente|Dépassement total|Montant total des frais d’utilisation dépassant le montant de votre crédit. Ce montant n’inclut pas les taxes.|
|Montant des taxes|Non applicable|Taxes qui s’appliquent au montant total de la vente pour la période spécifique.|
|Montant total|Non applicable|Montant dû pour la facture une fois le crédit appliqué et la taxe ajoutée.|

### <a name="review-marketplace-invoice"></a>Passer en revue la facture de la Place de marché.

Cette section s’applique uniquement si vous vous trouvez en Australie, au Japon ou à Singapour.

Comparez le total de la Place de marché Azure sur **Rapports** > **Récapitulatif de l’utilisation** dans Enterprise Portal avec votre facture de la Place de marché. La facture de la Place de marché concerne uniquement les achats et l’utilisation liés à la Place de marché Azure. Les montants figurant dans **Résumé de l’utilisation** incluent déjà une taxe qui est déterminée par le serveur de publication.

Connectez-vous au [portail Entreprise](https://ea.azure.com), puis sélectionnez **Rapports**. En haut à droite de l’onglet, changez de vue en passant de **M** à **C**, et faites correspondre la période sur la facture.  

![Capture d’écran qui montre l’option M + C dans le récapitulatif de l’utilisation.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)  

Le total de la **Place de marché Azure** doit correspondre au **Total de la vente** sur votre facture de la Place de marché. Pour plus d’informations sur vos frais d’utilisation, accédez à **Télécharger l’utilisation**. Sous **Frais de la Place de marché**, sélectionnez **Télécharger**. Le prix de la Place de marché comprend une taxe déterminée par l’éditeur. Les clients ne recevront pas de facture distincte de la part de l’éditeur pour collecter les taxes sur la transaction.

![Capture d’écran montrant l’option de téléchargement sous les frais de la Place de marché.](./media/review-enterprise-agreement-bill/ea-portal-download-usage-marketplace.png)

## <a name="view-price-sheet-information"></a>Voir des informations de grille tarifaire

Les administrateurs d’entreprise peuvent voir les tarifs associés à leur inscription pour les services Azure.

Pour voir la grille tarifaire actuelle :

1. Dans le portail Azure Enterprise, sélectionnez **Rapports**, puis **Grille tarifaire**.
2. Affichez la grille tarifaire ou sélectionnez **Télécharger**.

![Exemple présentant des informations de grille tarifaire](./media/review-enterprise-agreement-bill/ea-create-view-price-sheet-information.png)

Pour télécharger des tarifs historiques :

1. Dans le portail Azure Enterprise, sélectionnez **Rapports**, puis **Télécharger l'utilisation**.
2. Téléchargez la grille tarifaire.

![Exemple montrant l’emplacement de téléchargement d’une ancienne grille tarifaire](./media/review-enterprise-agreement-bill/create-ea-view-price-sheet-download-historical-price-list.png)

Voici quelques raisons qui expliquent les différences de tarifs :

- Les tarifs ont peut-être changé entre l’inscription précédente et la nouvelle inscription. Des changements de tarifs peuvent survenir du fait qu’ils sont contractuels pendant une inscription spécifique depuis la date de début jusqu’à la date de fin d’un contrat.
- Lorsque vous passez à une nouvelle inscription, la tarification change en fonction du nouvel accord. Le tarif est défini par votre grille tarifaire, et peut être plus élevé dans la nouvelle inscription.
- Si une inscription est prolongée, les tarifs changent aussi. Les prix varient en fonction de ceux du paiement à l’utilisation.

## <a name="request-detailed-usage-information"></a>Demander des informations détaillées sur l’utilisation

Les administrateurs d’entreprise peuvent voir un résumé de leurs données d’utilisation, l’engagement financier consommé et les frais associés à l’utilisation supplémentaire dans le portail Azure Enterprise. Les frais sont présentés au niveau du résumé pour tous les comptes et abonnements.

Pour voir l’utilisation détaillée de comptes spécifiques, téléchargez le rapport de détails de l’utilisation en accédant à **Rapports** > **Télécharger l’utilisation**.

> [!NOTE]
> Le rapport des détails de l’utilisation n’inclut pas les taxes applicables. Une latence pouvant aller jusqu’à huit heures peut être observée entre le moment où l’utilisation est engagée et celui où elle apparaît dans le rapport.

Pour les inscriptions indirectes, votre partenaire doit activer la fonction de balisage pour que vous puissiez voir toutes les informations relatives aux coûts.

## <a name="reports"></a>Rapports

Les administrateurs d’entreprise peuvent voir un résumé de leurs données d’utilisation, l’engagement financier consommé et les frais associés à l’utilisation supplémentaire dans le portail Azure Enterprise. Les frais sont présentés au niveau du résumé pour tous les comptes et abonnements.

### <a name="azure-enterprise-reports"></a>Rapports Azure Enterprise

- Résumé de l’utilisation et graphiques
- Rapport d’utilisation du service
- Rapport de solde et de frais
- Rapport de détails de l’utilisation
- Rapport de frais de la Place de marché Azure
- Price sheet
- Téléchargement des rapports avancés
- Mise en forme du rapport CSV

### <a name="to-view-the-usage-summary-reports-and-graphs"></a>Pour voir les rapports et graphes du résumé de l’utilisation :

1. Accédez au portail Azure Enterprise.
1. Sélectionnez **Rapports** dans le volet de gauche.
1. Sélectionnez l’onglet **Résumé de l’utilisation**.
1. Sélectionnez la période d’engagement dans le menu des plages de dates en haut à gauche.
1. Sélectionnez la période ou le mois sur le graphe pour afficher des détails supplémentaires.
1. Dans cet onglet, vous pouvez :
   - Afficher un graphique montrant l’utilisation par mois, avec le détail de l’engagement, du dépassement des frais de service, des frais facturés séparément et des frais de la Place de marché Azure.
   - Filtrez par services, comptes et abonnements sous le graphe.
   - Basculez entre la **répartition des frais par services** et la **répartition des frais par hiérarchie**.
   - Affichez les détails des services Azure, des frais facturés séparément et des frais de la Place de marché Azure.

## <a name="service-usage-report"></a>Rapport d’utilisation du service

La page du rapport d’utilisation du service permet aux administrateurs d’entreprise d’afficher un résumé de leurs données d’utilisation du service. L’utilisation est présentée au niveau du résumé pour tous les comptes et abonnements. Pour consulter les détails de l'utilisation, vous pouvez filtrer le rapport par comptes ou par abonnements.

> [!NOTE]
> Une latence jusqu’à cinq jours entre la date d’utilisation et le moment où cette utilisation est reflétée dans ce rapport est possible.

### <a name="to-view-the-report"></a>Pour afficher le rapport :

1. Connectez-vous au portail Azure Enterprise.
1. Sélectionnez **Rapports** dans la barre de navigation à gauche.
1. Sélectionnez l’onglet **Résumé de l’utilisation**.
1. Sélectionnez la plage de dates.
1. Choisissez les comptes ou abonnements à afficher.
1. Vous pouvez éventuellement :
   - Basculer entre la vue des **frais par services** et celle des **frais par hiérarchie** pour afficher différentes répartitions.
   - Affichez les détails tels que le nom du service, l’unité de mesure, les unités consommées, le taux effectif et le coût total.

## <a name="download-csv-reports"></a>Télécharger les rapports CSV

La page de téléchargement de rapport mensuel permet aux administrateurs de l’entreprise de télécharger plusieurs rapports dans des fichiers au format .CSV. Les rapports téléchargeables incluent :

- Rapport de solde et de frais
- Rapport de détails de l’utilisation
- Rapport de frais de la Place de marché Azure
- Price sheet

### <a name="to-download-reports"></a>Pour télécharger des rapports :

1. Sur le portail Azure Enterprise, sélectionnez **Rapport**.
1. Sélectionnez **Usage Download** (Téléchargement de l’utilisation) dans le ruban supérieur.
1. Sélectionnez **Télécharger** en regard du rapport mensuel approprié.

### <a name="csv-report-formatting-issues"></a>Problèmes de mise en forme des rapports CSV

Les clients qui consultent les rapports CSV du portail Azure Enterprise en euros peuvent rencontrer des problèmes de mise en forme liés aux virgules et aux points.

Par exemple, vous pouvez voir :

| **ServiceResource** | **ResourceQtyConsumed** | **ResourceRate** | **ExtendedCost** |
| --- | --- | --- | --- |
| Heures | 24 | 0,0535960591133005 | 12,863,054,187,192,100,000,000 |

Ce qui suit doit s’afficher :

| ServiceResource | ResourceQtyConsumed | ResourceRate | ExtendedCost |
| --- | --- | --- | --- |
| Heures | 24 | 0,0535960591133005 | 1,2863054187192120000000 |

Ce problème de mise en forme est dû aux paramètres par défaut de la fonctionnalité d'importation Excel. Excel importe tous les champs en texte « général » et suppose qu’un nombre est séparé dans la norme mathématique. Par exemple : « 1,000.00 ».

Si une devise européenne utilise un point (.) comme séparateur de milliers et une virgule comme séparateur décimal (,), ce nombre s’affichera de manière incorrecte. Par exemple : "1.000,00". Les résultats de l’importation peuvent varier en fonction de votre paramètre de langue régionale.

### <a name="to-import-the-csv-file-without-formatting-issues"></a>Pour importer le fichier CSV sans problèmes de mise en forme :

1. Dans Microsoft Excel, sélectionnez **Fichier** > **Ouvrir**.
   L’Assistant Importation de texte s’affiche.
1. Sous **Type de données d’origine**, choisissez **Délimité**.  La valeur par défaut est **Largeur fixe**.
1. Sélectionnez **Suivant**.
1. Sous Délimiteurs, cochez la case de la **virgule**. Désactivez l’option **Tabulation** si elle est activée.
1. Sélectionnez **Suivant**.
1. Faites défiler jusqu’aux colonnes **ResourceRate** et **ExtendedCost**.
1. Sélectionnez la colonne **ResourceRate**. Elle apparaît en surbrillance en noir.
1. Sous la section **Format des données en colonne**, sélectionnez **Texte** au lieu de **Général**. L'en-tête de la colonne passera de **Général** à **Texte**.
1. Répétez les étapes 8 et 9 pour la colonne **Extended Cost**, puis sélectionnez **Terminer**.

> [!TIP]
> Si Excel est configuré pour ouvrir automatiquement des fichiers CSV, vous devez utiliser la fonction **Ouvrir** dans Excel à la place. Dans Excel, sélectionnez **Fichier** > **Ouvrir**.

## <a name="balance-and-charge-report"></a>Rapport de solde et de frais

Le rapport de solde et de frais fournit un résumé mensuel des informations sur les soldes, nouveaux achats, frais de service de la Place de marché Azure, ajustements et frais de dépassement.

Toutes les lignes du tableau de l’engagement de service Azure restent statiques mois après mois. Vous trouverez les détails de tous les achats et ajustements sous les sections de **détails des nouveaux achats** et de **détails des ajustements**.

### <a name="download-the-balance-and-charge-report"></a>Télécharger le rapport de solde et de frais

1. Connectez-vous au portail Azure Enterprise en tant qu’administrateur d’entreprise.
1. Sélectionnez **Rapports** dans le volet de gauche.
1. Sélectionnez l’onglet **Report Download** (Téléchargement du rapport).
1. Sélectionnez le mois approprié sous la colonne **Balance and Charge** (Solde et frais), puis téléchargez le rapport.

## <a name="usage-detail-report"></a>Rapport de détails de l’utilisation

Le rapport de détails de l’utilisation offre un résumé mensuel de la consommation des services et des quantités d’une inscription. Il comprend des informations sur les noms des compteurs, les types de compteurs et les quantités consommées.

### <a name="download-the-usage-detail-report"></a>Télécharger le rapport de détails de l’utilisation

1. Connectez-vous au portail Azure Enterprise en tant qu’administrateur d’entreprise.
1. Sélectionnez **Rapports** dans la barre de navigation à gauche.
1. Sélectionnez l’onglet **Télécharger l’utilisation**.
1. Sélectionnez le mois approprié sous la colonne **Usage Detail** (Détails de l’utilisation), puis téléchargez le rapport.

## <a name="azure-marketplace-charges-in-azure-enterprise-portal-reports"></a>Frais de la Place de marché Azure dans les rapports Azure Enterprise

Il existe deux types de frais de la Place de marché Azure :

- **Basés sur l’utilisation :** Produits mesurés en unités transactionnelles.  Par exemple, les machines virtuelles sont facturées toutes les heures, les recherches d’API Bing sont facturées par nombre de recherches.
- **Frais mensuels** : Facturation mensuelle basée sur l’utilisation ou l’accès.

Pour plus d'informations sur les frais de la Place de marché Azure, voir les [FAQ sur la Place de marché Azure](https://azure.microsoft.com/marketplace/faq/).

Pour afficher les différents frais dans le portail Azure Enterprise :

- **Rapport de résumé de l’utilisation** : affiche **à la fois** les frais de la Place de marché Azure basés sur l’utilisation et les frais mensuels.
- **Rapport de frais de la Place de marché** : affiche **uniquement** les frais de la Place de marché Azure basés sur l’utilisation.  Les frais à usage unique ne sont pas affichés.

> [!NOTE]
> La Place de marché Azure n’est pas disponible pour les inscriptions MPSA.

## <a name="advanced-report-download"></a>Téléchargement des rapports avancés

Pour la création de rapports sur des plages de dates ou des comptes spécifiques, vous pouvez utiliser le téléchargement de rapports avancés. Depuis le 30 août 2016, le format du fichier de sortie est CSV afin de prendre en charge les jeux d’enregistrements plus volumineux.

1. Dans le portail Azure Enterprise, sélectionnez **Téléchargement des rapports avancés**.
1. Sélectionnez **Appropriate Date Range** (Plage de dates appropriée).
1. Sélectionnez **Appropriate Accounts** (Comptes appropriés).
1. Sélectionnez **Request Usage Data** (Demander les données d’utilisation).
1. Sélectionnez le bouton **Actualiser** jusqu’à ce que l’état du rapport affiche **Télécharger**.
1. Téléchargez le rapport.

## <a name="reporting-for-non-enterprise-administrators"></a>Rapports pour les administrateurs non d’entreprise

Les administrateurs d’entreprise peuvent autoriser les administrateurs de service (DA) et les propriétaires de comptes (AO) à afficher les frais sur une inscription. Les propriétaires de compte disposant d’un accès peuvent télécharger des rapports CSV spécifiques à leur compte et à leurs abonnements. Ils peuvent également consulter les informations visuellement dans la section des rapports du portail Azure Enterprise.

### <a name="to-enable-access"></a>Pour activer l’accès :

 1. Connectez-vous au portail Azure Enterprise en tant qu’administrateur d’entreprise.
 1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
 1. Sélectionnez l’onglet **Inscription**.
 1. Sous la section des **détails de l’inscription**, sélectionnez l’icône du crayon en regard de **DA View Charges** (Afficher les frais de l’administrateur de service) ou de **AO View Charges** (Afficher les frais du propriétaire du compte).
 1. Sélectionnez **Enabled**.
 1. Sélectionnez **Enregistrer**.

### <a name="to-view-reports"></a>Pour afficher des rapports :

1. Connectez-vous au portail Azure Enterprise en tant qu’administrateur de service ou propriétaire du compte.
1. Sélectionnez **Rapports** dans la barre de navigation à gauche.
1. Sélectionnez l’onglet **Résumé de l’utilisation** pour afficher des informations sur le compte et l’abonnement.
1. Sélectionnez **Usage Download** (Téléchargement de l’utilisation) pour afficher les rapports .CSV.

Les administrateurs de service et les propriétaires de comptes ne peuvent pas afficher les frais lors de l’utilisation de la fonction **Téléchargement des rapports avancés**. Les coûts s’affichent sous la forme $0.

Les autorisations accordées au propriétaire du compte pour afficher les frais s’étendent aux propriétaires de comptes et à tous les utilisateurs qui disposent d’autorisations sur les abonnements associés. Si vous êtes un client indirect, les fonctionnalités de coût doivent être activées par votre partenaire de canal.

## <a name="power-bi-reporting"></a>Rapports Power BI

La création de rapports de Power BI est disponible pour les clients EA directs, les partenaires et les clients indirects qui ont accès aux informations de facturation.

### <a name="power-bi-pro"></a>Power BI Pro

Power BI Pro est disponible pour les clients EA. Avec Power BI Pro, vous pouvez générer et partager des rapports pour gérer efficacement vos données de coûts. Vous bénéficiez également de fonctionnalités de collaboration et d’actualisation des données supplémentaires. Power BI Pro offre une capacité de données et des limites de diffusion des données supérieures.

<!--We plan to add new cost management features for Azure Enterprise customers.

Current Power BI (free) users who use the Microsoft Azure Consumption Insights content pack can get a 60-day free trial of Power BI Pro. After the trial is over, you can continue using Power BI Pro by adding a license.

To sign up for the free Power BI Pro trial:

1. From the gear icon in Power BI, select **Manage personal storage**.
1. Select **Try Pro for free** on the right.

See [Power BI self-service sign up](https://powerbi.microsoft.com/documentation/powerbi-service-self-service-signup-for-power-bi/#power-bi-pro-60-day-trial) for more information on the Power BI Pro free trial.

### Azure EA Power BI Pro trial terms

- **General purpose**: The extended Power BI Pro for the "Microsoft Azure Enterprise" content pack trial offer (the "Offer") is available to existing qualified users during the term of the Offer, to allow them to access certain insights related to their Azure consumption through the use of a specific Power BI content pack.
- **Eligibility**: Users under an Enterprise Agreement (EA) can participate in the Offer if they have a function related to their organization's Azure billing, service, or cost management.
- **Exclusions**:
  - Users already participating in the Extended Power BI Pro trial will continue to qualify under the pre-existing offer and can't enter into the Azure EA Power BI Pro trial offer.
  - Users participating in the Offer can only use Power BI Pro with the Microsoft Azure Enterprise content pack. Any other use of Power BI Pro is prohibited.
  - Term: The Offer began on June 1, 2017 and ended on May 31, 2018.  Acceptance can occur at any time during the 12-month period, though the offer will terminate on May 31, 2018 for all users regardless of when they accepted the Offer.
  -->

### <a name="to-access-microsoft-azure-consumption-insights"></a>Pour accéder à Microsoft Azure Consumption Insights :

1. Accédez à [Microsoft Azure Consumption Insights](https://app.powerbi.com/getdata/services/azureconsumption?cpcode=MicrosoftAzureConsumptionInsights&amp;getDataForceConnect=true&amp;WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
1. Sélectionnez **Obtenir maintenant**.
1. Indiquez un numéro d'inscription et le nombre de mois, puis sélectionnez **Suivant**.
1. Fournissez votre clé d’accès API pour la connexion. Vous pouvez trouver la clé de votre inscription dans le [portail Enterprise](https://ea.azure.com/?WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
1. Sélectionnez **Sign In** (Connexion) pour lancer automatiquement le processus d'importation.
1. Une fois l’opération terminée, un nouveau tableau de bord, un nouveau rapport et un nouveau modèle s’affichent dans le volet de navigation. Sélectionnez le tableau de bord pour afficher vos données importées.

> [!TIP]
>
> - Pour plus d’informations sur la génération de la clé API pour votre inscription, consultez le fichier d’aide des rapports d’API sur le [portail Enterprise](https://ea.azure.com/?WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
> - Pour plus d'informations sur la connexion de Power BI à votre consommation Azure, voir [Microsoft Azure Consumption Insights](/power-bi/desktop-connect-azure-cost-management).

### <a name="to-access-the-legacy-power-bi-ea-content-pack"></a>Pour accéder au pack de contenu Power BI EA hérité :

1. Rendez-vous sur le [site web de Power BI](https://app.powerbi.com/getdata/services/azure-enterprise).
1. Connectez-vous avec un compte professionnel ou scolaire valide.

   Le compte professionnel ou scolaire peut être identique ou différent de celui que vous utilisez pour accéder à l’inscription via le portail Azure Enterprise.
1. Dans le tableau de bord de services, sélectionnez **Microsoft Azure Enterprise**, puis **Connexion**.
1. Sur l’écran **Se connecter à Azure Enterprise**, renseignez les champs :
    - URL d’environnement Azure : [https://ea.azure.com](https://ea.azure.com/)
    - Nombre de mois : de 1 à 36
    - Numéro d’inscription : entrez votre numéro d’inscription
1. Sélectionnez **Suivant**.
1. Dans la **zone de la clé d’authentification**, entrez la clé API.

    Vous pouvez accéder à la clé API dans le portail Azure Enterprise sous l’onglet **Télécharger l’utilisation**. Sélectionnez **Clé d’accès API**, puis collez la clé dans la zone **Clé du compte**.
1. Le chargement des données dure entre 5 et 30 minutes environ dans Power BI, en fonction de la taille des jeux de données.

## <a name="reports-faq"></a>FAQ sur les rapports

Cette section répond aux questions les plus courantes sur les rapports.

### <a name="why-is-my-cost-showing-as-0"></a>Pourquoi mon coût affiche-t-il $0 ?

Pour les clients avec **inscription directe**, les administrateurs d'entreprise peuvent fournir aux propriétaires de compte et aux administrateurs de service un accès aux informations sur le coût et le prix des rapports d'utilisation. Procédez comme suit :

1. Dans le portail Enterprise, sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez le crayon bleu en regard de l’affichage des frais pour le DA (administrateur de service).
1. Sélectionnez **Activé** et enregistrez.
1. Sélectionnez le crayon bleu en regard de l’affichage des frais pour l’AO (propriétaire du compte).
1. Sélectionnez **Activé** et enregistrez.

> [!NOTE]
> Si vous êtes propriétaire de compte ou administrateur de service, contactez votre administrateur d’entreprise pour activer la fonctionnalité de tarification.

Pour les clients avec **inscription indirecte**, vérifiez auprès de votre partenaire s’il a activé la fonctionnalité de tarification pour vous. Cette opération ne peut être réalisée que par le partenaire. Une fois cette fonctionnalité activée, vous pouvez afficher le coût et le prix de votre inscription en tant qu'administrateur d'entreprise.

Pour les partenaires, si vous souhaitez activer la fonctionnalité d'affichage des frais pour un propriétaire de compte ou un administrateur de service, suivez les étapes de l'**inscription directe**.

### <a name="why-is-there-no-sku-information-on-my-usage-detail-report"></a>Pourquoi mon rapport de détails de l’utilisation ne contient-il aucune information de référence SKU ?

Le rapport de détails de l’utilisation ne contient aucune information de référence SKU. En revanche, étant donné qu’il contient des informations sur l'utilisation, vous pouvez télécharger le rapport de grille tarifaire afin d’obtenir des informations de référence SKU.

### <a name="why-doesnt-the-total-amount-on-azure-marketplace-match-the-reports-for-usage-summary-and-detail"></a>Pourquoi le montant total sur la Place de marché Azure ne correspond-il pas aux rapports pour le résumé et les détails de l'utilisation ?

Le rapport de frais de la Place de marché Azure affiche uniquement les frais basés sur l’utilisation. Les frais à usage unique ne sont pas affichés. Reportez-vous à la page Résumé de l’utilisation pour connaître les utilisations à jour sur les frais basés sur l’utilisation et les frais à usage unique.

### <a name="why-is-there-no-information-on-my-api-report"></a>Pourquoi mon rapport d’API n’affiche-t-il aucune information ?

Les clés API expirent tous les six mois. En cas de problème, un administrateur d'entreprise doit générer une nouvelle clé API. N'oubliez pas de suivre les étapes de la FAQ sur le rapport API.

### <a name="why-isnt-my-power-bi-report-working"></a>Pourquoi mon rapport Power BI ne fonctionne-t-il pas ?

En cas de problème avec Power BI, ouvrez un ticket auprès de l'[équipe de support Power BI](https://support.powerbi.com).

### <a name="why-dont-my-resource-tags-show-on-my-reports"></a>Pourquoi mes balises de ressources ne s'affichent-elles pas sur mes rapports ?

Les balises de ressources sont gérées sur le portail Azure. Vous pouvez contacter l'équipe des abonnements Azure sur le [portail Azure](https://portal.azure.com). Suivez les étapes de l'article [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).

### <a name="why-does-my-resource-rate-change-every-day"></a>Pourquoi mon taux de ressource change-t-il tous les jours ?

Le taux de ressource indiqué dans le rapport de détails de l’utilisation est une valeur calculée. Il représente le tarif mensuel moyen qui a été facturé pour le service. Le taux de ressource est calculé à l’aide de la moyenne de votre engagement mensuel et de vos frais de dépassement mensuels pour une unité de service. La partie de l’utilisation facturée par rapport à votre engagement et aux tarifs de dépassement change à chaque fin de journée. Ainsi, le taux de ressource affiché change également au cours du mois. Le taux de ressource est verrouillé le cinquième jour suivant la fin du mois.

### <a name="glossary-of-processes-for-calculating-the-resource-rate"></a>Glossaire des processus de calcul du taux de ressource

- **Nombre total d’unités brutes :** Quantité consommée dans le rapport de détails de l’utilisation.
- **Ressource MOCP par unité :** Le système d’utilisation en amont émet les utilisations de chaque service dans des unités différentes. (Préréglage)
- **Consommation par unité :** Unité de mesure Azure Enterprise. (Préréglage)
- **Prix** : Prix unitaire dans le portail Azure Enterprise.
- **Coût total :** Coût total du rapport de détails de l’utilisation, ou utilisation de l’engagement + dépassement dans le portail Azure Enterprise.

### <a name="charges-calculations"></a>Calculs des frais

- **Convertir en ressources MOCP consommées** = `ROUND(Total RAW Units * MOCP Resource Per Unit,4)`
- **Convertir en unités consommées** = `Consumed MOCP Resources / Consumption per Unit`
- **Calculer le coût total** = `Consumed Units * Price`

### <a name="logic-in-the-usage-calculation-logic"></a>Logique de calcul de l'utilisation

**Taux de ressource** = `Total Cost /(Total RAW Units / MOCP Resource Per Unit)`

Le taux de ressource est calculé sur la base de vos frais. Il peut ne pas correspondre au prix unitaire réel figurant dans la grille tarifaire.

Dans le rapport de téléchargement des données d’utilisation, vous pouvez voir l'utilisation des ressources brutes jusqu'à six décimales. Ces données sont utilisées pour le calcul des frais de dépassement. Toutefois, les données d’utilisation affichées dans le portail Azure Enterprise sont arrondies à quatre décimales pour les unités d’engagement et tronquées à zéro décimale pour les unités de dépassement. Sous le portail Azure Enterprise, tous les dépassements d’utilisation sont facturés pour des unités complètes uniquement. Vous risquez de constater une différence importante entre le prix unitaire et le taux de ressource pour l’utilisation facturée en tant que dépassement ou pour plusieurs mois.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Passer en revue les frais facturés.
> * Passer en revue les frais de dépassement de service.
> * Passer en revue la facture de la Place de marché.

Passez à l’article suivant pour en savoir plus sur l’utilisation du portail Azure EA.

> [!div class="nextstepaction"]
> [Bien démarrer avec le portail Azure EA](../manage/ea-portal-get-started.md)