---
title: Automatiser les processus Azure Application Insights en utilisant Logic Apps
description: Découvrez comment automatiser rapidement des processus reproductibles en ajoutant le connecteur Application Insights à votre application logique.
ms.topic: conceptual
ms.date: 03/11/2019
ms.openlocfilehash: f6406c2e6fb933c561a8ae54009499768c81a204
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90970868"
---
# <a name="automate-application-insights-processes-by-using-logic-apps"></a>Automatiser les processus Application Insights en utilisant Logic Apps

Vous devez exécuter à plusieurs reprises les mêmes requêtes sur vos données de télémétrie pour vérifier le bon fonctionnement de votre service ? Cherchez-vous à automatiser ces requêtes pour rechercher les tendances et les anomalies, puis générer vos propres flux de travail autour d’elles ? Le connecteur Azure Application Insights pour Logic Apps est l’outil qu’il vous faut pour atteindre cet objectif.

> [!NOTE]
> Le connecteur Azure Application Insights a été remplacé par le [connecteur Azure Monitor](../platform/logicapp-flow-connector.md) intégré avec Azure Active Directory au lieu d’exiger une clé API. Il vous permet également de récupérer des données d’un espace de travail Log Analytics.

Avec cette intégration, vous pouvez automatiser de nombreux processus sans écrire la moindre ligne de code. Vous pouvez créer une application logique avec le connecteur Application Insights pour automatiser rapidement tous les processus Application Insights. 

Vous pouvez également ajouter des actions. La fonctionnalité Logic Apps d’Azure App Service met à votre disposition des centaines d’actions possibles. Par exemple, en utilisant une application logique, vous pouvez automatiquement envoyer une notification par e-mail ou créer un bogue dans Azure DevOps. Vous pouvez également utiliser l’un des nombreux [modèles](../../logic-apps/logic-apps-create-logic-apps-from-templates.md) disponibles pour contribuer à accélérer le processus de création de votre application logique. 

## <a name="create-a-logic-app-for-application-insights"></a>Créer une application logique pour Application Insights

Dans ce didacticiel, vous allez apprendre à créer une application logique qui utilise l’algorithme de cluster automatique Analytics pour regrouper des attributs dans les données pour une application web. Ce flux envoie automatiquement les résultats par courrier électronique. Il s’agit d’un exemple de la façon dont vous pouvez utiliser Application Insights Analytics et Logic Apps ensemble. 

### <a name="step-1-create-a-logic-app"></a>Étape 1 : Créer une application logique
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur **Créer une ressource**, sélectionnez **Web + Mobile**, puis sélectionnez **Application logique**.

    ![Fenêtre Nouvelle application logique](./media/automate-with-logic-apps/1createlogicapp.png)

### <a name="step-2-create-a-trigger-for-your-logic-app"></a>Étape 2 : Créer un déclencheur pour votre application logique
1. Dans la fenêtre **Concepteur d’application logique**, sous **Démarrer avec un déclencheur courant**, sélectionnez **Récurrence**.

    ![Fenêtre Concepteur d’application logique](./media/automate-with-logic-apps/2logicappdesigner.png)

1. Dans la zone **Intervalle**, sélectionnez **1**, puis, dans la zone **Fréquence**, sélectionnez **Jour**.

    ![Fenêtre « Récurrence » du Concepteur d’application logique](./media/automate-with-logic-apps/3recurrence.png)

### <a name="step-3-add-an-application-insights-action"></a>Étape 3 : Ajouter une action Application Insights
1. Cliquez sur **Nouvelle étape**.

1. Dans la zone de recherche **Choisir une action**, tapez **Azure Application Insights**.

1. Sous **Actions**, cliquez sur **Azure Application Insights - Visualiser la requête Analytics**.

    ![Fenêtre de « Choisir une action » du Concepteur d’application logique](./media/automate-with-logic-apps/4visualize.png)

### <a name="step-4-connect-to-an-application-insights-resource"></a>Étape 4 : Se connecter à une ressource Application Insights

Pour cette étape, vous avez besoin d’un ID d’application et d’une clé d’API pour votre ressource. Vous pouvez les récupérer sur le portail Azure, comme illustré dans le schéma suivant :

![Capture d’écran montrant la page accès d’API dans le portail Azure avec le bouton Créer une clé API sélectionné.](./media/automate-with-logic-apps/5apiaccess.png)

![ID d’application dans le portail Azure](./media/automate-with-logic-apps/6apikey.png)

Renseignez le nom de votre connexion, l’ID d’application et la clé d’API.

![Fenêtre de connexion de flux du Concepteur d’application logique](./media/automate-with-logic-apps/7connection.png)

### <a name="step-5-specify-the-analytics-query-and-chart-type"></a>Étape 5 : Spécifier le type de requête et de graphique Analytics
Dans l’exemple suivant, la requête sélectionne les requêtes ayant échoué au cours du dernier jour et les met en corrélation avec les exceptions qui se sont produites dans le cadre de l’opération. Analytics met en corrélation les requêtes ayant échoué en fonction de l’identificateur operation_Id. La requête segmente ensuite les résultats à l’aide de l’algorithme de cluster automatique. 

Lorsque vous créez vos propres requêtes, vérifiez qu’elles fonctionnent correctement dans Analytics avant de les ajouter à votre flux.

1. Dans la zone **Requête**, ajoutez la requête Analytics suivante :

    ```
    requests
    | where timestamp > ago(1d)
    | where success == "False"
    | project name, operation_Id
    | join ( exceptions
        | project problemId, outerMessage, operation_Id
    ) on operation_Id
    | evaluate autocluster()
    ```

1. Dans la zone **Type de graphique**, sélectionnez **Table HTML**.

    ![Fenêtre de configuration de requête Analytics](./media/automate-with-logic-apps/8query.png)

### <a name="step-6-configure-the-logic-app-to-send-email"></a>Étape 6 : Configurer l’application logique pour envoyer un e-mail

1. Cliquez sur **Nouvelle étape**.

1. Dans la zone de recherche, tapez **Office 365 Outlook**.

1. Cliquez sur **Office 365 Outlook - Envoyer un message électronique**.

    ![Sélection d’Office 365 Outlook](./media/automate-with-logic-apps/9sendemail.png)

1. Dans la fenêtre **Envoyer un message électronique**, effectuez les étapes suivantes :

   a. Tapez l’adresse e-mail du destinataire.

   b. Tapez l’objet de l’e-mail.

   c. Cliquez n’importe où dans la zone **Corps**, puis, dans le menu de contenu dynamique qui s’ouvre sur la droite, sélectionnez **Corps**.
    
   d. Cliquez sur la liste déroulante **Ajouter un nouveau paramètre**, puis sélectionnez Pièces jointes et Est au format HTML.

      ![Capture d’écran montrant la fenêtre Envoyer un e-mail avec la zone Corps en surbrillance et le menu Contenu dynamique avec le Corps en surbrillance sur le côté droit.](./media/automate-with-logic-apps/10emailbody.png)

      ![Configuration d’Office 365 Outlook](./media/automate-with-logic-apps/11emailparameter.png)

1. Dans le menu de contenu dynamique, effectuez les étapes suivantes :

    a. Sélectionnez **Nom de la pièce jointe**.

    b. Sélectionnez **Attachment Content** (Contenu de la pièce jointe).
    
    c. Dans la zone **Is HTML** (Est HTML), sélectionnez **Oui**.

      ![Écran de configuration d’e-mail Office 365](./media/automate-with-logic-apps/12emailattachment.png)

### <a name="step-7-save-and-test-your-logic-app"></a>Étape 7 : Enregistrer et tester votre application logique
* Cliquez sur **Enregistrer** pour enregistrer vos modifications.

Vous pouvez attendre que le déclencheur exécute l’application logique ou vous pouvez l’exécuter immédiatement en sélectionnant **Exécuter**.

![Écran de création d’application logique](./media/automate-with-logic-apps/13save.png)

Lorsque votre application logique s’exécute, les destinataires que vous avez spécifiés dans la liste des e-mails reçoivent un e-mail similaire à :

![Message électronique de l’application logique](./media/automate-with-logic-apps/flow9.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez la création de [requêtes Analytics](../log-query/get-started-queries.md).
- Découvrez plus en détail les [applications logiques](../../logic-apps/logic-apps-overview.md).



<!--Link references-->

