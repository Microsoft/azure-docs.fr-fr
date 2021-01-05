---
title: Analyser les coûts d’Azure avec l’application Power BI
description: Cet article explique comment installer et utiliser l’application Power BI Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 11/09/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: benshy
ms.openlocfilehash: 1c5adf58f670f3cadcedbe710568551004b77b78
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413239"
---
# <a name="analyze-cost-with-the-azure-cost-management-power-bi-app-for-enterprise-agreements-ea"></a>Analyser les coûts avec l’application Power BI Azure Cost Management pour les Contrats Entreprise

Cet article explique comment installer et utiliser l’application Power BI Azure Cost Management. L’application vous aide à analyser et à gérer vos coûts Azure dans Power BI. Vous pouvez utiliser l’application pour surveiller les coûts et les tendances d’utilisation, et pour identifier les options d’optimisation des coûts afin de réduire vos dépenses.

Vous pouvez utiliser l’application telle quelle, ou bien vous pouvez la modifier pour étendre les filtres, les vues et les visualisations par défaut afin de les personnaliser en fonction de vos besoins. Utilisez-la ensuite pour joindre des données supplémentaires pour créer des rapports personnalisés afin d’obtenir des vues holistiques de vos coûts globaux.

L’application Power BI Azure Cost Management prend actuellement en charge seulement les clients avec un [Contrat Entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/).

> [!NOTE]
> Les applications basées sur un modèle Power BI ne prennent pas en charge le téléchargement du fichier PBIX.

## <a name="prerequisites"></a>Prérequis

- Une [licence Power BI Pro](/power-bi/service-self-service-signup-for-power-bi) est requise pour installer et utiliser l’application.
- Pour vous connecter aux données, vous devez utiliser un compte [Administrateur d’entreprise](../manage/understand-ea-roles.md). Le rôle Administrateur d’entreprise (lecture seule) est pris en charge.

## <a name="installation-steps"></a>Procédure d’installation :

Pour installer l’application :

1. Ouvrez [Application Power BI Azure Cost Management](https://aka.ms/costmgmt/ACMApp).
2. Dans la page AppSource de Power BI, sélectionnez **Obtenir**.
3. Sélectionnez **Continuer** pour accepter les conditions d’utilisation et la politique de confidentialité.
4. Dans la zone **Installer cette application Power BI**, sélectionnez **Installer**.
5. Si nécessaire, créez un espace de travail et sélectionnez **Continuer**.
6. Une fois l’installation terminée, une notification apparaît, indiquant que votre nouvelle application est prête.
7. Sélectionnez **Accéder à l’application**.
8. Dans **Démarrer avec votre nouvelle application**, sous **Connecter vos données**, sélectionnez **Connecter**.  
  ![Démarrer avec votre nouvelle application - Connecter](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/connect-data2.png)
9. Dans la boîte de dialogue qui apparaît, entrez votre numéro d’inscription Contrat Entreprise pour **BillingProfileIdOrEnrollmentNumber**. Spécifiez le nombre de mois de données à obtenir. Conservez la valeur d’**Étendue** par défaut de **Numéro d’inscription**, puis sélectionnez **Suivant**.  
  ![Entrer les informations d’inscription du Contrat Entreprise](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-number.png)  
10. La boîte de dialogue suivante se connecte à Azure et obtient les données nécessaires pour les recommandations d’instances réservées. *Laissez les valeurs par défaut comme elles sont configurées* et sélectionnez **Sign in** (Se connecter).  
  ![Capture d’écran illustrant la boîte de dialogue Se connecter à l’application Azure Cost Management avec les valeurs par défaut.](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/autofit.png)  
11. La dernière étape de l’installation est de se connecter à votre inscription Contrat Entreprise ; elle nécessite un compte [Administrateur d’entreprise](../manage/understand-ea-roles.md). Sélectionnez **Sign in** (Se connecter) pour vous authentifier avec votre inscription Contrat Entreprise. Cette étape lance également une action d’actualisation des données dans Power BI.  
  ![Se connecter à l’inscription Contrat Entreprise](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-auth.png)  
    > [!NOTE]
    > Le processus d’actualisation des données peut prendre un certain temps. Sa durée dépend du nombre de mois spécifiés et de la quantité de données à synchroniser.
12. Pour vérifier l’état d’actualisation des données, sélectionnez l’onglet **Datasets** (Jeux de données) dans l’espace de travail. Regardez ensuite l’horodatage Actualisé. S’il est toujours en cours de mise à jour, vous voyez un indicateur montrant que l’actualisation est en cours.  
  ![Actualiser les données](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/data-refresh2.png)

Une fois l’actualisation des données terminée, sélectionnez l’application Azure Cost Management pour visualiser les rapports créés au préalable.

## <a name="reports-available-with-the-app"></a>Rapports disponibles avec l’application

Les rapports suivants sont disponibles dans l’application.

**Prise en main** - Fournit des liens utiles vers la documentation et des liens permettant de spécifier un feedback.

**Account overview** (Vue d’ensemble du compte) - Le rapport montre un récapitulatif mensuel des informations, notamment :

- Frais et crédits
- Nouveaux achats
- Frais de la place de marché Azure
- Dépassements et frais totaux

**Utilisation par abonnements et par groupes de ressources** - Fournit une vue des coûts dans le temps et des graphiques montrant les coûts par abonnement et par groupe de ressources.

**Usage by Services** (Utilisation par services) - Fournit une vue de l’utilisation dans le temps par MeterCategory. Vous pouvez suivre vos données d’utilisation et explorer les anomalies pour comprendre les pics ou les diminutions de l’utilisation.

**Top 5 Usage drivers** (Top 5 des centres de coûts d’utilisation) - Le rapport montre une totalisation des coûts filtrés pour le top 5 des MeterCategory et des MeterName correspondants.

**Utilisation d’Azure Hybrid Benefit sur Windows Server** - Le rapport montre le nombre de machines virtuelles pour lesquelles Azure Hybrid Benefit activée. Il montre également le nombre de cœurs/processeurs virtuels utilisés par les machines virtuelles.

![Rapport complet Azure Hybrid Benefit](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ahb-report-full.png)

Le rapport identifie également les machines virtuelles Windows où Hybrid Benefit est **Activé** mais où il y a _moins de_ 8 processeurs virtuels. Il montre aussi les endroits où Hybrid Benefit est **non activé** et qui ont 8 processeurs virtuels _ou plus_. Ces informations vous aident à utiliser pleinement Hybrid Benefit. Appliquez l’avantage à vos machines virtuelles les plus coûteuses pour optimiser vos économies potentielles.

![Avantages Azure Hybrid Benefit : Moins de 8 processeurs virtuels et processeurs virtuels non activés](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ahb-report.png)

**RI Chargeback** (Refacturation des instances réservées) - Le rapport vous aide à comprendre où et dans quelle mesure un avantage des instances réservées est appliqué par région, par abonnement, par groupe de ressources ou par ressource. Le rapport utilise les données d’utilisation amorties pour montrer la vue.

Vous pouvez appliquer un filtre sur _chargetype_ pour voir les données de sous-utilisation des instances réservées.

Pour plus d’informations sur les données amorties, consultez [Obtenir les données d’utilisation et de coûts de la réservation pour les Contrats Entreprise](../reservations/understand-reserved-instance-usage-ea.md).

**RI Savings** (Économies liées aux instances réservées) - Le rapport montre les économies accumulées par les réservations pour l’abonnement, le groupe de ressources et au niveau de la ressource. Il affiche :

- Coût avec réservation
- Estimation du coût à la demande si la réservation n’a pas été appliquée à l’utilisation
- Économies accumulées résultant de la réservation

 Le rapport soustrait des économies totales le coût des pertes des réservations sous-utilisées. La perte ne se serait pas produite sans une réservation.

Vous pouvez utiliser les données d’utilisation amorties comme base pour les données.

<a name="shared-recommendation"></a>
**VM RI Coverage (shared recommendation)** (Couverture des instances réservées de machines virtuelles [recommandation partagée]) - Le rapport est divisé entre l’utilisation des machines virtuelles à la demande et l’utilisation des machines virtuelles des instances réservées sur la période sélectionnée. Il fournit des recommandations pour les achats d’instances réservées de machines virtuelles au niveau d’une étendue partagée.

Pour utiliser le rapport, sélectionnez le filtre d’exploration.

![Rapport de couverture des instances réservées de machines virtuelles - Sélection de l’exploration](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ri-drill-down2.png)

Sélectionnez la région que vous voulez analyser. Sélectionnez ensuite le groupe de flexibilité de taille d’instance, etc.

Pour chaque niveau d’exploration, les filtres suivants sont appliqués au rapport :

- Les données de couverture à droite sont le filtre qui indique la quantité d’utilisation facturée en utilisant le pourcentage à la demande par rapport à la quantité couverte par la réservation.
- Les recommandations sont également filtrées.

Le tableau des recommandations fournit des recommandations pour l’achat des réservations en fonction des tailles de machine virtuelle utilisées.

Les valeurs de _Normalized Size_ (Taille normalisée) et de _Recommended Quantity Normalized_ (Quantité normalisée recommandée) vous aident à normaliser l’achat à la plus petite taille possible pour un groupe de flexibilité de taille d’instance. Les informations sont utiles si vous prévoyez d’acheter une seule réservation pour toutes les tailles dans le groupe de flexibilité de taille d’instance.

![Recommandations pour les instances réservées](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ri-recomendations.png)

**VM RI Coverage (single recommendation)** (Couverture des instances réservées de machines virtuelles [recommandation unique]) - Le rapport est divisé entre l’utilisation des machines virtuelles à la demande et l’utilisation des machines virtuelles des instances réservées sur la période de temps sélectionnée. Il fournit des recommandations pour les achats d’instances réservées de machines virtuelles au niveau d’une étendue d’abonnement.

Pour plus d’informations sur l’utilisation du rapport, consultez la section [Couverture des instances réservées de machines virtuelles (recommandation partagée)](#shared-recommendation).

**RI purchases** (Achats d’instances réservées) - Le rapport montre les achats d’instances réservées sur la période spécifiée.

**Price sheet** (Liste des prix) - Le rapport montre une liste détaillée des prix spécifiques à un compte de facturation ou à une inscription Contrat Entreprise.

## <a name="troubleshoot-problems"></a>Résoudre les problèmes

Si vous rencontrez des problèmes avec l’application Power BI, les informations de dépannage suivantes peuvent vous aider.

### <a name="error-processing-the-data-in-the-dataset"></a>Erreur lors du traitement des données dans le jeu de données

Vous risquez d’obtenir une erreur indiquant :

```
There was an error when processing the data in the dataset.
Data source error: {"error":{"code":"ModelRefresh_ShortMessage_ProcessingError","pbi.error":{"code":"ModelRefresh_ShortMessage_ProcessingError","parameters":{},"details":[{"code":"Message","detail":{"type":1,"value":"We cannot convert the value \"Required Field: 'Enr...\" to type List."}}],"exceptionCulprit":1}}} Table: <TableName>.
```

Un nom de table s’affiche à la place de `<TableName>`.

#### <a name="cause"></a>Cause

La valeur de l’**étendue** par défaut `Enrollment Number` a été modifiée dans la connexion à Cost Management.

#### <a name="solution"></a>Solution

Reconnectez-vous à Cost Management et définissez la valeur de l’**étendue** sur `Enrollment Number`. N’entrez pas le numéro d’inscription de votre organisation, mais tapez plutôt `Enrollment Number` exactement comme il apparaît dans l’image suivante.

![Entrer les informations d’inscription du Contrat Entreprise](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-number.png)  

### <a name="budgetamount-error"></a>Erreur BudgetAmount

Vous risquez d’obtenir une erreur indiquant :

```
Something went wrong
There was an error when processing the data in the dataset.
Please try again later or contact support. If you contact support, please provide these details.
Data source error: The 'budgetAmount' column does not exist in the rowset. Table: Budgets.
```

#### <a name="cause"></a>Cause

Cette erreur se produit en raison d’un bogue avec les métadonnées sous-jacentes. Le problème est lié à l’absence de budget disponible sous **Gestion des coûts > Budget** dans le portail Azure. La résolution du bogue est en cours de déploiement sur Power BI Desktop et le service Power BI. 

#### <a name="solution"></a>Solution

- Tant que le bogue n’a pas été corrigé, vous pouvez contourner le problème en ajoutant un budget de test dans le portail Azure au niveau du compte de facturation/de l’inscription EA. Le budget de test débloque la connexion avec Power BI. Pour plus d’informations sur la création d’un budget, consultez [Tutoriel : Créer et gérer des budgets Azure](tutorial-acm-create-budgets.md).


### <a name="invalid-credentials-for-azureblob-error"></a>Erreur d’informations d’identification non valides pour AzureBlob

Vous risquez d’obtenir une erreur indiquant :

```
Failed to update data source credentials: The credentials provided for the AzureBlobs source are invalid.
```

#### <a name="cause"></a>Cause

Cette erreur se produit si vous changez la méthode d’authentification pour la connexion blob AutoFitComboMeter.

#### <a name="solution"></a>Solution

1. Connectez-vous à vos données.
1. Après avoir entré votre inscription EA et le nombre de mois, veillez à conserver la valeur par défaut **Anonyme** pour la méthode d’authentification et **Aucun** pour le paramètre niveau de confidentialité.  
  ![Capture d’écran illustrant la boîte de dialogue Se connecter à l’application Azure Cost Management avec les valeurs Anonyme et Aucun.](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/autofit-troubleshoot.png)  
1. Dans la page suivante, définissez **OAuth2** pour la méthode d’authentification et **Aucun** pour le niveau de confidentialité. Ensuite, connectez-vous pour vous authentifier auprès de votre inscription. Cette étape lance également une action d’actualisation des données Power BI.


## <a name="data-reference"></a>Référence de données

Les informations suivantes récapitulent les données disponibles via l’application. Il y a également des liens vers des API qui fournissent des informations plus détaillées pour les champs et les valeurs des données.

| **Informations de référence du tableau** | **Description** |
| --- | --- |
| **AutoFitComboMeter** | Données incluses dans l’application pour normaliser la recommandation sur les instances réservées et l’utilisation de la plus petite taille possible dans le groupe de la famille d’instances. |
| [**Balance summary (Récapitulatif du solde)**](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary#response) | Récapitulatif du solde pour les Contrats Entreprise. |
| [**Budgets**](/rest/api/consumption/budgets/get#definitions) | Détails des budgets pour visualiser les coûts réels ou l’utilisation par rapport aux cibles budgétaires existantes. |
| [**Pricesheets (Listes de prix)**](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet#see-also) | Tarifs applicables pour le profil de facturation ou l’inscription Contrat Entreprise spécifié. |
| [**RI charges (Coûts facturés pour les instances réservées)**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges#response) | Coûts facturés associés à vos instances réservées au cours des 24 derniers mois. |
| [**RI recommendations (shared) - Recommandations (partagées) pour les instances réservées**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#response) | Recommandations d’achat d’instances réservées, basées sur les tendances d’utilisation observées dans tous vos abonnements sur les 7 derniers jours. |
| [**RI recommendations (single) - Recommandations (abonnement unique) pour les instances réservées**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#response-1) | Recommandations d’achat d’instances réservées, basées sur les tendances d’utilisation de votre abonnement unique au cours des 7 derniers jours. |
| [**RI usage details (Détails d’utilisation des instances réservées)**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#response) | Détails de la consommation pour vos instances réservées existantes au cours du dernier mois. |
| [**RI usage summary (Récapitulatif de l’utilisation des instances réservées)**](/rest/api/consumption/reservationssummaries/list) | Pourcentage d’utilisation quotidienne des réservations Azure |
| [**Usage Details (Détails d’utilisation)**](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#usage-details-field-definitions) | Ventilation des quantités consommées et des coûts estimés pour le profil de facturation donné dans l’inscription Contrat Entreprise. |
| [**Usage details amortized (Détails d’utilisation avec amortissement)**](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#usage-details-field-definitions) | Ventilation des quantités consommées et des coûts amortis estimés pour le profil de facturation donné dans l’inscription Contrat Entreprise. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration des données, l’actualisation, le partage des rapports et la personnalisation des rapports, consultez les articles suivants :

- [Configurer une actualisation planifiée](/power-bi/refresh-scheduled-refresh)
- [Partager des tableaux de bord et des rapports Power BI avec des collègues et d’autres utilisateurs](/power-bi/service-share-dashboards)
- [Abonner vous-même et d’autres utilisateurs à des rapports et des tableaux de bord dans le service Power BI](/power-bi/service-report-subscribe)
- [Télécharger un rapport à partir du service Power BI vers Power BI Desktop](/power-bi/service-export-to-pbix)
- [Enregistrer un rapport dans le service Power BI et dans Power BI Desktop](/power-bi/service-report-save)
- [Créer un rapport dans le service Power BI en important un jeu de données](/power-bi/service-report-create-new)