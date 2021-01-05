---
title: Automatiser des rapports personnalisés avec des données Azure Application Insights
description: Automatiser des rapports quotidients/hebdomadaires/mensuels personnalisés avec des données Azure Application Insights
ms.topic: conceptual
ms.date: 05/20/2019
ms.reviewer: sdash
ms.openlocfilehash: d465666ce5d631a234644d1adc85afcf6e3c1e27
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010603"
---
# <a name="automate-custom-reports-with-azure-application-insights-data"></a>Automatiser des rapports personnalisés avec des données Azure Application Insights

Les rapports périodiques permettent de tenir une équipe informée sur le fonctionnement des services critiques pour l’entreprise dont elle s’occupe. Les développeurs, les équipes DevOps/SRE et leurs responsables peuvent produire des rapports automatisés qui fournissent des insights fiables, sans demander à quiconque de se connecter au portail. Ces rapports servent également à mieux identifier les augmentations progressives des latences, des taux de chargement ou d’échec susceptibles de ne pas déclencher de règles d’alerte.

Chaque entreprise a des besoins uniques en termes de rapports, notamment :

* Agrégations en pourcentage spécifiques des métriques ou métriques personnalisées dans un rapport.
* Disponibilité de rapports différents pour les cumuls de données quotidiens, hebdomadaires et mensuels destinés à différents publics.
* Segmentation selon des attributs personnalisés comme la région ou l’environnement.
* Regroupement de ressources IA dans un seul rapport, même si elles appartiennent à des abonnements ou groupes de ressources différents, etc.
* Rapports distincts contenant des métriques sensibles envoyés à un public sélectionné.
* Rapports destinés à des parties prenantes qui n’ont pas accès aux ressources du portail.

> [!NOTE] 
> L’e-mail de synthèse Application Insights hebdomadaire ne permettait aucune personnalisation et sera supprimé en faveur des options personnalisées répertoriées ci-dessous. Vous recevrez le dernier e-mail de synthèse hebdomadaire le 11 juin 2018. Configurez l’une des options suivantes pour obtenir des rapports personnalisés similaires (utilisez la requête suggérée ci-dessous).

## <a name="to-automate-custom-report-emails"></a>Pour automatiser les e-mails de rapports personnalisés

Vous pouvez [interroger par programmation des données Application Insights](https://dev.applicationinsights.io/) pour générer des rapports personnalisés selon une planification. Les options suivantes peuvent vous aider à démarrer rapidement :

* [Automatiser des rapports avec Power Automate](../platform/logicapp-flow-connector.md)
* [Automatiser des rapports avec Logic Apps](automate-with-logic-apps.md)
* Utilisez le modèle de [fonction Azure](../../azure-functions/functions-create-first-azure-function.md) « synthèse planifiée Application Insights » dans le scénario de surveillance. Cette fonction utilise SendGrid pour remettre l’e-mail. 

    ![Modèle de fonction Azure](./media/automate-custom-reports/azure-function-template.png)

## <a name="sample-query-for-a-weekly-digest-email"></a>Exemple de requête pour un e-mail de synthèse hebdomadaire
La requête suivante montre la jointure entre plusieurs jeux de données utilisée pour un e-mail de synthèse hebdomadaire comme un rapport. Personnalisez-la selon les besoins et utilisez-la avec les options répertoriées ci-dessus pour automatiser un rapport hebdomadaire.

```AIQL
let period=7d;
requests
| where timestamp > ago(period)
| summarize Row = 1, TotalRequests = sum(itemCount), FailedRequests = sum(toint(success == 'False')),
    RequestsDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
| join (
dependencies
| where timestamp > ago(period)
| summarize Row = 1, TotalDependencies = sum(itemCount), FailedDependencies = sum(success == 'False'),
    DependenciesDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
) on Row | join (
pageViews
| where timestamp > ago(period)
| summarize Row = 1, TotalViews = sum(itemCount)
) on Row | join (
exceptions
| where timestamp > ago(period)
| summarize Row = 1, TotalExceptions = sum(itemCount)
) on Row | join (
availabilityResults
| where timestamp > ago(period)
| summarize Row = 1, OverallAvailability = iff(isnan(avg(toint(success))), '------', tostring(toint(avg(toint(success)) * 10000) / 100.0)),
    AvailabilityDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
) on Row
| project TotalRequests, FailedRequests, RequestsDuration, TotalDependencies, FailedDependencies, DependenciesDuration, TotalViews, TotalExceptions, OverallAvailability, AvailabilityDuration
```

## <a name="application-insights-scheduled-digest-report"></a>Rapport de synthèse planifié Application Insights

1. Créez une application de fonction Azure. (Application Insights _Activé_ est requis uniquement si vous souhaitez superviser votre nouvelle application de fonction avec Application Insights)

   Consultez la documentation Azure Functions pour découvrir comment [créer une application de fonction](../../azure-functions/functions-create-first-azure-function.md#create-a-function-app).

2. Dès que la nouvelle application Function App a terminé le déploiement, sélectionnez **Accéder à la ressource**.

3. Sélectionnez **Nouvelle fonction**.

   ![Créer une nouvelle capture d’écran Function](./media/automate-custom-reports/new-function.png)

4. Sélectionnez le **_modèle de synthèse planifié Application Insights_** .

     > [!NOTE]
     > Par défaut, les applications de fonction sont créées avec la version 3.x du runtime. Vous devez [cibler la version](../../azure-functions/set-runtime-version.md)1.x **du runtime d'Azure Functions** pour utiliser le modèle de synthèse planifié Application Insights. Accédez à Configuration > Paramètres du runtime de fonction pour modifier la version du Runtime. ![Capture d’écran du runtime](./media/automate-custom-reports/change-runtime-v.png)

   ![Capture d’écran Nouvelle fonction - Application Insights - Modèle](./media/automate-custom-reports/function-app-04.png)

5. Entrez une adresse e-mail de destinataire appropriée pour votre rapport et sélectionnez **Créer**.

   ![Capture d’écran Paramètres - Fonction](./media/automate-custom-reports/scheduled-digest.png)

6. Sélectionnez votre **Application de fonction** > **Fonctionnalités de la plateforme** > **Configuration**.

    ![Capture d’écran Paramètres d’application Azure Function](./media/automate-custom-reports/config.png)

7. Créez trois nouveaux paramètres d’application avec les valeurs appropriées correspondants ``AI_APP_ID``, ``AI_APP_KEY``, et ``SendGridAPI``. Sélectionnez **Enregistrer**.

     ![Capture d’écran Interface d’intégration de la fonction](./media/automate-custom-reports/app-settings.png)
    
    Les valeurs AI_ sont accessibles sous Accès API pour la ressource Application Insights pour laquelle vous voulez créer un rapport. Si vous n’avez pas de clé API Application Insights, il est possible de **Créer une clé API**.
    
   * AI_APP_ID = ID d’application
   * AI_APP_KEY = Clé API
   * SendGridAPI = Clé API SendGrid

     > [!NOTE]
     > Si vous ne possédez pas de compte SendGrid, vous pouvez en créer un. La documentation relative à SendGrid pour Azure Functions est disponible [ici](../../azure-functions/functions-bindings-sendgrid.md). Si une brève explication concernant la configuration de SendGrid et la génération d’une clé API vous suffit, reportez-vous à la fin de cet article. 

8. Sélectionnez **Integrate** et sous Sorties, cliquez sur **SendGrid ($return)** .

     ![Capture d’écran Sortie](./media/automate-custom-reports/integrate.png)

9. Sous **Paramètre d’application clé SendGridAPI**, sélectionnez le paramètre d’application que vous venez de créer pour **SendGridAPI**.

     ![Capture d’écran Exécuter Function App](./media/automate-custom-reports/sendgrid-output.png)

10. Exécutez et tester votre application Function App.

     ![Capture d’écran Tester](./media/automate-custom-reports/function-app-11.png)

11. Consultez votre messagerie électronique pour confirmer que le message a bien été envoyé/reçu.

     ![Capture d’écran Objet d’e-mail](./media/automate-custom-reports/function-app-12.png)

## <a name="sendgrid-with-azure"></a>SendGrid avec Azure

Ces étapes s’appliquent uniquement si vous n’avez pas encore configuré de compte SendGrid.

1. Dans le portail Azure, sélectionnez **Créer une ressource** > recherchez **Remise de courrier électronique SendGrid** > cliquez sur **Créer** > remplissez les instructions de création propres à SendGrid.

     ![Capture d’écran Créer une ressource SendGrid](./media/automate-custom-reports/sendgrid.png)

2. Une fois créée sous Comptes SendGrid, sélectionnez **Gérer**.

     ![Capture d’écran Paramètres - Clé API](./media/automate-custom-reports/sendgrid-manage.png)

3. Cette action permet de lancer le site de SendGrid. Sélectionnez **Paramètres** > **Clés API**.

     ![Capture d’écran Créer et afficher - Application de clé API](./media/automate-custom-reports/function-app-15.png)

4. Créez une clé API > choisissez **Créer et afficher**. (Consultez la documentation relative à SendGrid portant sur la restriction d’accès restreint pour déterminer si le niveau des autorisations est approprié pour votre clé API. Accès total est sélectionné ici par exemple.

   ![Capture d’écran Accès complet](./media/automate-custom-reports/function-app-16.png)

5. Copiez l’intégralité de la clé, cette valeur est ce dont vous avez besoin dans vos paramètres Function App comme valeur pour SendGridAPI

   ![Capture d’écran Copier - Clé API](./media/automate-custom-reports/function-app-17.png)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez la création de [requêtes Analytics](../log-query/get-started-queries.md).
* En savoir plus sur l’[interrogation par programmation de données Application Insights](https://dev.applicationinsights.io/)
* Découvrez plus en détail les [applications logiques](../../logic-apps/logic-apps-overview.md).
* En savoir plus sur [Microsoft Power Automate](https://ms.flow.microsoft.com).

