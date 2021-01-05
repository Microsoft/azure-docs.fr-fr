---
title: Configurer l’Assistant Log Analytics dans Azure AD | Microsoft Docs
description: Découvrez comment configurer Log Analytics.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 08/05/2020
ms.author: markvi
author: MarkusVi
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2bdf3a763dc71eb842496775b6cc91b8ca39b4b3
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96343839"
---
# <a name="tutorial-configure-the-log-analytics-wizard"></a>Tutoriel : Configurer l’Assistant Log Analytics


Ce tutoriel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * configurer un espace de travail Log Analytics pour vos journaux d’audit et de connexion ;
> * exécuter des requêtes à l’aide du langage de requête Kusto (Kusto Query Language, KQL) ;
> * créer une règle d’alerte qui envoie des alertes en cas d’utilisation d’un compte spécifique ;
> * créer un classeur personnalisé à l’aide du modèle de démarrage rapide ;
> * ajouter une requête à un modèle de classeur existant.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure avec au moins un administrateur titulaire d’une licence P1. Si vous n’en avez pas, vous pouvez demander un [essai gratuit](https://azure.microsoft.com/free/).

- Un locataire Azure AD.

- Utilisateur considéré comme administrateur général ou administrateur de la sécurité pour un locataire Azure AD.


Familiarisez-vous avec les articles suivants :

- [Tutoriel : Collecter et analyser des journaux de ressources à partir d’une ressource Azure](../../azure-monitor/learn/tutorial-resource-logs.md)

- [Intégrer des journaux d’activité Azure AD aux journaux d’activité Azure Monitor](./howto-integrate-activity-logs-with-log-analytics.md)

- [Gérer des comptes d’accès d’urgence dans Azure AD](../roles/security-emergency-access.md)

- [Aide-mémoire sur KQL](/azure/data-explorer/kql-quick-reference)

- [Classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md)



## <a name="configure-a-workspace"></a>Configurer un espace de travail 

Cette procédure décrit comment configurer un espace de travail Log Analytics pour vos journaux d’audit et de connexion.
La configuration d’un espace de travail Log Analytics comprend deux grandes étapes :
 
1. Création d’un espace de travail Log Analytics
2. Définition des paramètres de diagnostic

**Pour configurer un espace de travail :** 


1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.

2. Recherchez des **espaces de travail Log Analytics**.

    ![Rechercher des ressources, des services et des documents](./media/tutorial-log-analytics-wizard/search-services.png)

3. Dans la page Espaces de travail Log Analytics, cliquez sur **Ajouter**.

    ![Capture d’écran montrant le bouton Ajouter de la page Espace de travail Log Analytics.](./media/tutorial-log-analytics-wizard/add.png)

4.  Dans la page **Espace de travail Log Analytics**, procédez comme suit :

    ![Créer un espace de travail Log Analytics](./media/tutorial-log-analytics-wizard/create-log-analytics-workspace.png)

    1. Sélectionnez votre abonnement.

    2. Sélectionnez un groupe de ressources.
 
    3. Dans la zone de texte **Nom**, tapez un nom (par exemple, MytestWorkspace1).

    4. Sélectionnez votre région.

5. Cliquez sur **Revoir + créer**.

    ![Examiner et créer](./media/tutorial-log-analytics-wizard/review-create.png)

6. Cliquez sur **Créer** et que le déploiement soit terminé. Il se peut que vous deviez actualiser la page pour afficher le nouvel espace de travail.

    ![Créer](./media/tutorial-log-analytics-wizard/create-workspace.png)

7. Recherchez **Azure Active Directory**.

    ![Capture d’écran montrant Azure Active Directory dans la Recherche Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

8. Dans la section **Surveillance**, cliquez sur **Paramètres de diagnostic**.

    ![Capture d’écran montrant les paramètres de diagnostic sélectionnés dans la section Supervision.](./media/tutorial-log-analytics-wizard/diagnostic-settings.png)

9. Dans la page **Paramètres de diagnostic**, cliquez sur **Ajouter un paramètre de diagnostic**.

    ![Ajouter un paramètre de diagnostic](./media/tutorial-log-analytics-wizard/add-diagnostic-setting.png)

10. Dans la page **Paramètres de diagnostic**, procédez comme suit :

    ![Sélectionner les paramètres de diagnostic](./media/tutorial-log-analytics-wizard/select-diagnostics-settings.png)

    1. Sous **Détails de la catégorie**, sélectionnez **AuditLogs**, puis **SigninLogs**.

    2. Sous **Détails de la destination**, sélectionnez **Envoyer à Log Analytics**, puis votre nouvel espace de travail Log Analytics. 
   
    3. Cliquez sur **Enregistrer**. 

## <a name="run-queries"></a>Exécuter des requêtes  

Cette procédure montre comment exécuter des requêtes à l’aide du **langage de requête Kusto (Kusto Query Language, KQL).**


**Pour exécuter une requête :**


1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.

2. Recherchez **Azure Active Directory**.

    ![Capture d’écran montrant Azure Active Directory dans la Recherche Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. Dans la section **Surveillance**, cliquez **Journaux**.

4. Dans la page **Journaux**, cliquez sur **Prise en main**.

5. Dans la zone de texte **Rechercher*, tapez votre requête.

6. Cliquez sur **Exécuter**.  


### <a name="kql-query-examples"></a>Exemples de requêtes KQL

Prendre 10 entrées au hasard parmi les données d’entrée :

`SigninLogs | take 10`

Examiner les connexions où l’accès conditionnel a réussi :

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus` 


Compter le nombre d’accès conditionnels réussis :

`SigninLogs | where ConditionalAccessStatus == "success" | project UserDisplayName, ConditionalAccessStatus | count`


Agréger le nombre de connexions réussies par utilisateur et par jour :

`SigninLogs | where ConditionalAccessStatus == "success" | summarize SuccessfulSign-ins = count() by UserDisplayName, bin(TimeGenerated, 1d)` 


afficher le nombre de fois qu’un utilisateur a effectué une opération donnée au cours d’une période spécifique :

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | summarize count() by OperationName, Identity`


Afficher un tableau croisé dynamique des résultats sur le nom de l’opération :

`AuditLogs | where TimeGenerated > ago(30d) | where OperationName contains "Add member to role" | project OperationName, Identity | evaluate pivot(OperationName)`


Fusionner les journaux d’audit et de connexion à l’aide d’une jointure interne :

`AuditLogs |where OperationName contains "Add User" |extend UserPrincipalName = tostring(TargetResources[0].userPrincipalName) | |project TimeGenerated, UserPrincipalName |join kind = inner (SigninLogs) on UserPrincipalName |summarize arg_min(TimeGenerated, *) by UserPrincipalName |extend SigninDate = TimeGenerated` 


Afficher le nombre de connexions par type d’application cliente :

`SigninLogs | summarize count() by ClientAppUsed`

Compter les connexions par jour :

`SigninLogs | summarize NumberOfEntries=count() by bin(TimeGenerated, 1d)`

Prendre 5 entrées au hasard et projeter les colonnes à afficher dans les résultats :

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `


Prendre les 5 premières entrées en ordre décroissant et projeter les colonnes à afficher :

`SigninLogs | take 5 | project ClientAppUsed, Identity, ConditionalAccessStatus, Status, TimeGenerated `

Créer une colonne en combinant les valeurs de deux autres colonnes :

`SigninLogs | limit 10 | extend RiskUser = strcat(RiskDetail, "-", Identity) | project RiskUser, ClientAppUsed`




## <a name="create-an-alert-rule"></a>Créer une règle d'alerte  

Cette procédure montre comment envoyer des alertes lorsque le compte breakglass est utilisé.

**Pour créer une règle d’alerte :**


1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.

2. Recherchez **Azure Active Directory**.

    ![Capture d’écran montrant Azure Active Directory dans la Recherche Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. Dans la section **Surveillance**, cliquez **Journaux**.

4. Dans la page **Journaux**, cliquez sur **Prise en main**.

5. Dans la zone de texte **Rechercher**, tapez : `SigninLogs |where UserDisplayName contains "BreakGlass" | project UserDisplayName`.

6. Cliquez sur **Exécuter**.  

7. Dans la barre d’outils, cliquez sur **Nouvelle règle d’alerte**.

    ![Nouvelle règle d’alerte](./media/tutorial-log-analytics-wizard/new-alert-rule.png)

8. Dans la page **Créer une règle d’alerte**, vérifiez que l’étendue est correcte.

9. Sous **Condition**, cliquez sur : **Chaque fois que la recherche dans les journaux personnalisée moyenne est supérieure à <logic undefined> nombre**

    ![Condition par défaut](./media/tutorial-log-analytics-wizard/default-condition.png)

10. Dans la page **Configurer la logique de signal**, dans la section **Logique d’alerte**, procédez comme suit :

    ![Logique d'alerte](./media/tutorial-log-analytics-wizard/alert-logic.png)

    1. Pour l’option **Basée sur**, sélectionnez **Nombre de résultats**.

    2. Pour l’option **Opérateur**, sélectionnez **Supérieur à**.

    3. Pour l’option **Valeur de seuil**, sélectionnez **0**. 

11. Dans la page **Configurer la logique du signal**, dans la section **Évaluée sur la base de**, procédez comme suit :

    ![Évaluée sur la base de](./media/tutorial-log-analytics-wizard/evaluated-based-on.png)

    1. Pour l’option **Période (en minutes)** , sélectionnez **5**.

    2. Pour l’option **Fréquence (en minutes)** , sélectionnez **5**.

    3. Cliquez sur **Done**. 

12. Sous **Groupe d’actions**, cliquez sur **Sélectionner un groupe d’actions**. 

    ![Groupe d’actions](./media/tutorial-log-analytics-wizard/action-group.png)

13. Dans **Sélectionner un groupe d’actions à associer à cette règle d’alerte**, cliquez sur **Créer un groupe d’actions**. 

    ![Créer un groupe d’actions](./media/tutorial-log-analytics-wizard/create-action-group.png)

14. Dans la page **Créer un groupe d’actions**, procédez comme suit :

    ![Détails de l’instance](./media/tutorial-log-analytics-wizard/instance-details.png)

    1. Dans la zone de texte **Nom du groupe d’actions**, tapez **Mon groupe d’actions**.

    2. Dans la zone de texte **Nom d’affichage**, tapez **Mon action**.

    3. Cliquez sur **Vérifier + créer**. 

    4. Cliquez sur **Créer**.


15. Sous **Personnaliser l’action**, procédez comme suit :

    ![Personnaliser les actions](./media/tutorial-log-analytics-wizard/customize-actions.png)

    1. Sélectionnez **Objet de l’e-mail**.

    2. Dans zone de texte **Ligne d’objet**, tapez : `Breakglass account has been used`.

16. Sous **Détails des règles d’alerte**, procédez comme suit :

    ![Détails des règles d’alerte](./media/tutorial-log-analytics-wizard/alert-rule-details.png)

    1. Dans la zone de texte **Nom de la règle d’alerte**, tapez : `Breakglass account`.

    2. Dans la zone de texte **Description**, tapez : `Your emergency access account has been used`.

17. Cliquez sur **Créer une règle d'alerte**.   


## <a name="create-a-custom-workbook"></a>Créer un classeur personnalisé

Cette procédure montre comment créer un classeur à l’aide du modèle de démarrage rapide.




1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.

2. Recherchez **Azure Active Directory**.

    ![Capture d’écran montrant Azure Active Directory dans la Recherche Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. Dans la section **Surveillance**, cliquez sur **Classeurs**.

    ![Capture d’écran montrant Surveillance dans le menu du portail Azure avec les classeurs sélectionnés.](./media/tutorial-log-analytics-wizard/workbooks.png)

4. Dans la section **Démarrage rapide**, cliquez sur **Vide**.

    ![Démarrage rapide](./media/tutorial-log-analytics-wizard/quick-start.png)

5. Cliquez sur **Ajouter**.

    ![Ajouter un classeur](./media/tutorial-log-analytics-wizard/add-workbook.png)

6. Cliquez **Ajouter du texte**.

    ![Ajouter du texte](./media/tutorial-log-analytics-wizard/add-text.png)


7. Dans la zone de texte, tapez : `# Client apps used in the past week`, puis cliquez sur **Fin de l’édition**.

    ![Texte du classeur](./media/tutorial-log-analytics-wizard/workbook-text.png)

8. Dans le nouveau classeur, cliquez sur **Ajouter**, puis sur **Ajouter une requête**.

    ![Ajouter une requête](./media/tutorial-log-analytics-wizard/add-query.png)

9. Dans la zone de texte de la requête, tapez : `SigninLogs | where TimeGenerated > ago(7d) | project TimeGenerated, UserDisplayName, ClientAppUsed | summarize count() by ClientAppUsed`.

10. cliquez sur **Run Query (Exécuter la requête)**.

    ![Capture d’écran montrant le bouton Exécuter la requête.](./media/tutorial-log-analytics-wizard/run-workbook-query.png)

11. Dans la barre d’outils, sous **Visualisation**, cliquez sur **Graphique à secteurs**.

    ![Graphique à secteurs](./media/tutorial-log-analytics-wizard/pie-chart.png)

12. Cliquez sur **Fin de l’édition**.

    ![Fin de l’édition](./media/tutorial-log-analytics-wizard/done-workbook-editing.png)



## <a name="add-a-query-to-a-workbook-template"></a>Ajouter une requête à un modèle de classeur  

Cette procédure montre comment ajouter une requête à un modèle de classeur. L’exemple est basé sur une requête qui montre la répartition des accès conditionnel réussis par rapport aux échecs.


1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.

2. Recherchez **Azure Active Directory**.

    ![Capture d’écran montrant Azure Active Directory dans la Recherche Azure.](./media/tutorial-log-analytics-wizard/search-azure-ad.png)

3. Dans la section **Surveillance**, cliquez sur **Classeurs**.

    ![La capture d’écran montrant Surveillance dans le menu avec les classeurs sélectionnés.](./media/tutorial-log-analytics-wizard/workbooks.png)

4. Dans la section **Accès conditionnel**, cliquez sur **Rapports et insights sur l’accès conditionnel**.

    ![Capture d’écran montre l’option Insights et rapports sur l’accès conditionnel.](./media/tutorial-log-analytics-wizard/conditional-access-template.png)

5. Dans la barre d’outils, cliquez sur **Modifier**.

    ![Capture d’écran montrant le bouton Modifier.](./media/tutorial-log-analytics-wizard/edit-workbook-template.png)

6. Dans la barre d’outils, cliquez sur les points de suspension, sur **Ajouter**, puis sur **Ajouter une requête**.

    ![Ajouter une requête de classeur](./media/tutorial-log-analytics-wizard/add-custom-workbook-query.png)

7. Dans la zone de texte de requête, tapez : `SigninLogs | where TimeGenerated > ago(20d) | where ConditionalAccessPolicies != "[]" | summarize dcount(UserDisplayName) by bin(TimeGenerated, 1d), ConditionalAccessStatus`.

8. cliquez sur **Run Query (Exécuter la requête)**.

    ![Capture d’écran montrant le bouton Exécuter la requête qui permet d’exécuter cette requête.](./media/tutorial-log-analytics-wizard/run-workbook-insights-query.png)

9. Cliquez sur **Intervalle de temps**, puis sélectionnez **Définir dans la requête**.

10. Cliquez sur **Visualisation**, puis sélectionnez **Graphique à barres**. 

11. Cliquez sur **Paramètres avancés**. Pour le titre du graphique, tapez `Conditional Access status over the last 20 days`, puis cliquez sur **Fin de l’édition**. 

    ![Définir le titre du graphique](./media/tutorial-log-analytics-wizard/set-chart-title.png)








## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour découvrir comment gérer les identités des appareils à l’aide du portail Azure.
> [!div class="nextstepaction"]
> [Surveillance](overview-monitoring.md)