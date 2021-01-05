---
title: Générer des workflows d’automatisation basés sur la planification avec Azure
description: 'Tutoriel : créer un flux de travail d’automatisation périodique basé sur une planification qui s’intègre aux services cloud à l’aide d’Azure Logic Apps.'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/30/2020
ms.openlocfilehash: aad271875abb9024a1ecc7f45018c04d8c79ce95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842561"
---
# <a name="tutorial-create-schedule-based-and-recurring-automation-workflows-with-azure-logic-apps"></a>Tutoriel : Créer des workflows d’automatisation réguliers et basés sur la planification avec Azure Logic Apps

Ce tutoriel montre comment générer une [application logique exemple](../logic-apps/logic-apps-overview.md) qui automatise un workflow qui s’exécute selon une planification périodique. Plus précisément, cet exemple d’application logique vérifie le temps de trajet, en tenant compte des conditions de circulation, entre deux points, et s’exécute tous les matins de la semaine. Si la durée dépasse une limite définie, l’application logique vous envoie un e-mail indiquant la durée du trajet et le temps supplémentaire nécessaire pour atteindre votre destination. Le workflow comprend différentes étapes, qui commencent par un déclencheur basé sur une planification suivi d’une action Bing Maps, d’une action d’opérations de données, d’une action de flux de contrôle et d’une action de notification par e-mail.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez une application logique vide.
> * Ajoutez un déclencheur de périodicité qui spécifie la planification de votre application logique.
> * Ajoutez une action Bing Cartes qui permet d’obtenir le temps de trajet pour un itinéraire.
> * Ajoutez une action qui crée une variable, convertit le temps de trajet de secondes en minutes, puis stocke ce résultat dans la variable.
> * Ajoutez une condition qui compare la durée du trajet avec une limite définie.
> * Ajoutez une action qui vous envoie un e-mail si le temps de trajet dépasse la limite.

Lorsque vous avez terminé, votre application logique ressemble au flux de travail suivant à un niveau élevé :

![Capture d’écran représentant la vue d’ensemble générale d’un exemple de workflow d’application logique.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-overview.png)

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Un compte de messagerie d’un fournisseur de messagerie pris en charge par Logic Apps, par exemple Office 365 Outlook, Outlook.com ou Gmail. Pour les autres fournisseurs, [passez en revue la liste des connecteurs ici](/connectors/). Ce guide de démarrage rapide utilise Office 365 Outlook avec un compte professionnel ou scolaire. Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais l’interface utilisateur peut être légèrement différente.

  > [!IMPORTANT]
  > Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans Logic Apps. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser ce connecteur uniquement avec certains services approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification avec votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Pour obtenir la durée de trajet d’un itinéraire, vous avez besoin d’une clé d’accès pour l’API Bing Cartes. Pour obtenir cette clé, suivez les étapes décrites dans l’article indiquant [comment obtenir une clé Bing Cartes](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key).

## <a name="create-your-logic-app"></a>Créer votre application logique

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure. Dans la page d’accueil Azure, sélectionnez **Créer une ressource**.

1. Dans le menu Place de marché Azure, sélectionnez **Intégration** > **Application logique**.

   ![Capture d’écran représentant le menu Place de marché Azure dans lequel « Intégration » et « Application logique » sont sélectionnées.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-new-logic-app-resource.png)

1. Dans le volet **Application logique**, fournissez les informations décrites ici sur l’application logique que vous souhaitez créer.

   ![Capture d’écran représentant le volet de création d’application logique et les informations à fournir pour la nouvelle application logique.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-logic-app-settings.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Abonnement** | <*Azure-subscription-name*> | Le nom de votre abonnement Azure. Cet exemple utilise `Pay-As-You-Go`. |
   | **Groupe de ressources** | LA-TravelTime-RG | Nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md), qui est utilisé pour organiser les ressources connexes. Cet exemple crée un groupe de ressources nommé `LA-TravelTime-RG`. |
   | **Nom** | LA-TravelTime | Nom de votre application logique, qui peut contenir uniquement des lettres, des chiffres, des traits d’union (`-`), des traits de soulignement (`_`), des parenthèses (`(`, `)`) et des points (`.`). Cet exemple utilise `LA-TravelTime`. |
   | **Lieu** | USA Ouest | Région dans laquelle stocker les informations sur votre application logique. Cet exemple utilise `West US`. |
   | **Log Analytics** | Off | Maintenez le paramètre de journalisation des diagnostics **Désactivé**. |
   ||||

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Une fois que Azure a validé les informations relatives à votre application logique, sélectionnez **Créer**.

1. Une fois que Azure a déployé votre application, sélectionnez **Accéder à la ressource**.

   Azure ouvre le volet de sélection du modèle Logic Apps, qui affiche une vidéo de présentation, des déclencheurs couramment utilisés et des modèles de modèle d’application logique.

1. Faites défiler la page vers le bas au-delà de la vidéo et des déclencheurs courants jusqu’à la section **Modèles**, puis sélectionnez **Application logique vide**.

   ![Capture d’écran représentant le volet de sélection de modèle Logic Apps dont l’option « Application logique vide » est sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-logic-app-template.png)

Ajoutez ensuite le [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) de périodicité qui exécute le workflow selon une planification spécifiée. Chaque application logique doit commencer par un déclencheur, qui est activé lorsqu’un événement spécifique se produit ou lorsque de nouvelles données respectent une condition particulière. Pour plus d’informations, consultez [Quickstart: Build your first logic app workflow - Azure portal](../logic-apps/quickstart-create-first-logic-app-workflow.md) (Démarrage rapide : Générer votre premier flux de travail d’application logique - portail Azure).

## <a name="add-the-recurrence-trigger"></a>Ajouter le déclencheur de périodicité

1. Dans la zone de recherche du concepteur Logic Apps, entrez `recurrence`, puis sélectionnez le déclencheur **Périodicité**.

   ![Capture d’écran représentant la zone de recherche du concepteur Logic Apps qui contient le terme de recherche « périodicité » et le déclencheur « Périodicité » sélectionné dans la liste « Déclencheurs ».](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-schedule-recurrence-trigger.png)

1. Dans la forme **Périodicité**, sélectionnez le bouton représentant des **points de suspension** ( **...** ), puis **Renommer**. Renommez le déclencheur à l’aide de cette description : `Check travel time every weekday morning`.

   ![Capture d’écran représentant le bouton ellipse sélectionné, la liste « Paramètres » ouverte et la commande « Renommer » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/rename-recurrence-schedule-trigger.png)

1. Dans le déclencheur, modifiez ces propriétés comme décrit et montré ici.

   ![Capture d’écran représentant les modifications apportées à l’intervalle et à la fréquence du déclencheur.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/change-interval-frequency.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Intervalle** | Oui | 1 | Nombre d’intervalles d’attente entre les vérifications. |
   | **Fréquence** | Oui | Week | Unité de temps à utiliser pour la périodicité. |
   |||||

1. Sous **Intervalle** et **Fréquence**, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez ces propriétés pour ajouter le déclencheur.

   * **Aux jours indiqués**
   * **Aux heures indiquées**
   * **Aux minutes indiquées**

   ![Capture d’écran représentant la liste « Ajouter un nouveau paramètre » ouverte et les propriétés suivantes sélectionnées : « Aux jours indiqués », « Aux heures indiquées » et « Aux minutes indiquées ».](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-trigger-properties.png)

1. À présent, définissez les valeurs des propriétés supplémentaires comme indiqué et décrit ici.

   ![Capture d’écran représentant les propriétés supplémentaires définies sur les valeurs décrites dans le tableau suivant.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/recurrence-trigger-property-values.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Aux jours indiqués** | Lundi,mardi,mercredi,jeudi,vendredi | Ce paramètre est disponible uniquement lorsque vous définissez **Fréquence** sur **Semaine**. |
   | **Aux heures indiquées** | 7,8,9 | Ce paramètre est disponible uniquement lorsque vous définissez **Fréquence** sur **Semaine** ou **Jour**. Sélectionnez les heures de la journée pour cette périodicité. Cet exemple s’exécute aux marques de `7`, `8` et `9` heures. |
   | **Aux minutes indiquées** | 0,15,30,45 | Ce paramètre est disponible uniquement lorsque vous définissez **Fréquence** sur **Semaine** ou **Jour**. Sélectionnez les minutes de la journée pour cette périodicité. Cet exemple s’exécute toutes les 15 minutes à partir de la marque zéro heure. |
   ||||

   Ce déclencheur s’active tous les jours ouvrables, toutes les 15 minutes, de à 7 h 00 à 9 h 45. La zone **Aperçu** indique la planification de la périodicité. Pour plus d’informations, voir [Planifier des tâches et flux de travail à exécuter régulièrement avec des applications logiques](../connectors/connectors-native-recurrence.md) et [Actions et déclencheurs de flux de travail](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger).

1. Pour masquer les informations du déclencheur pour le moment, réduisez la forme en cliquant dans sa barre de titre.

   ![Capture d’écran représentant la forme de déclencheur réduite.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/collapse-trigger-shape.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

Votre application logique est maintenant active dans le Portail Azure, mais elle ne fait rien d’autre que le déclencheur en fonction de la planification spécifiée. Par conséquent, ajoutez une action qui répond à l’activation du déclencheur.

## <a name="get-the-travel-time-for-a-route"></a>Obtenir la durée du trajet d’un itinéraire

Maintenant que vous disposez d’un déclencheur, ajoutez une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) qui permet de récupérer la durée du trajet entre deux lieux. Logic Apps fournit un connecteur pour l’API Bing Cartes afin que vous puissiez accéder facilement à ces informations. Avant de commencer cette tâche, vérifiez que vous disposez d’une clé de l’API Bing Cartes comme décrit dans les conditions préalables requises de ce didacticiel.

1. Dans le Concepteur d’application logique, sous le déclencheur Périodicité, sélectionnez **Nouvelle étape**.

1. Sous **Choisir une opération**, sélectionnez **Standard**. Dans la zone de recherche, entrez `bing maps`, puis sélectionnez l’action **Obtenir un itinéraire**.

   ![Capture d’écran représentant la liste « Choisir une opération » filtrée par les actions « Bing Maps » et l’action « Récupérer un itinéraire » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-get-route-action.png)

1. Si vous ne disposez pas d’une connexion à Bing Cartes, vous êtes invité à en créer une. Fournissez les détails de connexion comme indiqué et décrit, puis sélectionnez **Créer**.

   ![Capture d’écran représentant la zone de connexion Bing Maps dans laquelle le nom de connexion et la clé d’API Bing Maps sont indiqués.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-maps-connection.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom de connexion** | Oui | BingMapsConnection | Donnez un nom à votre connexion. Cet exemple utilise `BingMapsConnection`. |
   | **Clé API** | Oui | <*Bing-Maps-API-key*> | Entrez la clé d’API Bing Maps que vous avez reçue précédemment. Si vous ne possédez pas une clé Bing Cartes, découvrez [comment en obtenir une](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key). |
   |||||

1. Renommez l’action avec cette description : `Get route and travel time with traffic`.

1. Dans l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les propriétés suivantes.

   * **Optimize**
   * **Unité de distance**
   * **Mode de déplacement**

   ![Capture d’écran représentant l’action « Obtenir un itinéraire... » pour laquelle les propriétés « Optimiser », « unité de distance » et « Mode de déplacement » sont sélectionnées.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-bing-maps-action-properties.png) 

1. À présent, entrez les valeurs des propriétés comme indiqué et décrit ici.

   ![Capture d’écran représentant les valeurs de propriété supplémentaires pour l’action « Obtenir un itinéraire ».](./media/tutorial-build-scheduled-recurring-logic-app-workflow/get-route-action-settings.png) 

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Étape 1** | Oui | <*start-location*> | Début de l’itinéraire. Cet exemple spécifie un exemple d’adresse de début. |
   | **Étape 2** | Oui | <*end-location*> | Destination de l’itinéraire. Cet exemple spécifie un exemple d’adresse de destination. |
   | **Optimize** | Non | timeWithTraffic | Paramètre permettant d’optimiser votre itinéraire, par exemple la distance, la durée du trajet avec le trafic en cours, etc. Sélectionnez la valeur du paramètre, **timeWithTraffic**. |
   | **Unité de distance** | Non | <*your-preference*> | Unité de distance utilisé pour l’itinéraire. Cet exemple utilise **Mile** comme unité. |
   | **Mode de déplacement** | Non | Conduite | Mode de déplacement pour parcourir l’itinéraire. Sélectionnez le mode **Driving** (Conduite). |
   |||||

   Pour plus d’informations sur ces paramètres et valeurs, voir [Calculate a route](/bingmaps/rest-services/routes/calculate-a-route) (Calculer un itinéraire).

1. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

Créez une variable afin de pouvoir convertir et stocker la durée du trajet actuel en minutes plutôt qu’en secondes. De cette façon, vous pouvez éviter de répéter la conversion et utiliser la valeur plus facilement dans les étapes ultérieures. 

## <a name="create-a-variable-to-store-travel-time"></a>Créer une variable pour stocker le temps de trajet

Vous souhaiterez parfois exécuter des opérations sur les données de votre workflow et utiliser les résultats dans des actions ultérieures. Pour enregistrer ces résultats afin de pouvoir facilement les réutiliser ou y faire référence, vous pouvez créer des variables pour stocker ces résultats après traitement. Vous ne pouvez créer des variables qu’au niveau supérieur de votre application logique.

Par défaut, l’action **Get route** (Obtenir l’itinéraire) retourne le temps de trajet actuel avec le trafic exprimé en secondes à partir de la propriété **Travel Duration Traffic** (Trafic correspondant au temps de trajet). Si vous préférez convertir et stocker cette valeur en minutes, vous facilitez la réutilisation ultérieure de la valeur sans la reconvertir.

1. Dans le concepteur, sous l’action **Get route** (Obtenir l’itinéraire), sélectionnez **Nouvelle étape**.

1. Sous **Choisir une opération**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `variables`, puis sélectionnez l’action **Initialiser la variable**.

   ![Capture d’écran représentant l’action « Initialiser la variable » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-initialize-variable-action.png)

1. Renommez l’action à l’aide de cette description : `Create variable to store travel time`.

1. Fournissez ces informations pour votre variable, comme indiqué dans ce tableau et dans les étapes sous ce dernier :

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom** | Oui | travelTime | Nom de la variable. Cet exemple utilise `travelTime`. |
   | **Type** | Oui | Integer | Type de données de la variable. |
   | **Valeur** | Non | Expression qui convertit la durée du trajet actuel de secondes en minutes (voir les étapes décrites sous ce tableau). | Valeur initiale de votre variable. |
   |||||

   1. Pour créer l’expression pour la propriété **Valeur**, cliquez dans la zone pour faire apparaître la liste à contenu dynamique. Si nécessaire, élargissez la fenêtre du navigateur jusqu’à ce que la liste dynamique apparaisse. Dans la liste de contenu dynamique, sélectionnez **Expression**, qui affiche l’éditeur d’expression.

      ![Capture d’écran représentant l’action « Initialiser la variable » avec le curseur à l’intérieur de la propriété « Value », qui ouvre la liste de contenu dynamique.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings.png)

      La liste de contenu dynamique affiche les sorties des actions précédentes que vous pouvez sélectionner comme entrées pour les actions suivantes dans votre flux de travail. La liste de contenu dynamique comprend un éditeur d’expression qui vous permet de sélectionner des fonctions qui effectuent des opérations dans votre expression. Cet éditeur d’expressions est disponible uniquement dans la liste de contenu dynamique.

   1. Dans l’éditeur d’expressions, entrez cette expression : `div(,60)`.

      ![Capture d’écran représentant l’expression « div(,60) » entrée dans l’éditeur d’expression.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-2.png)

   1. Dans l’expression, placez votre curseur entre la parenthèse ouvrante ( **(** ) et la virgule ( **,** ), puis sélectionnez **Contenu dynamique**.

      ![Capture d’écran indiquant où placer le curseur dans l’expression « div(,60) » alors que « Contenu dynamique » est sélectionné.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-3.png)

   1. Dans la liste de contenu dynamique, sélectionnez la valeur de propriété **Travel Duration Traffic** (Trafic correspondant à la durée du trajet).

      ![Capture d’écran représentant la valeur de propriété « Travel Duration Traffic » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-4.png)

   1. Une fois la valeur de la propriété résolue dans l’expression, sélectionnez **OK**.

      ![Capture d’écran représentant le bouton « OK » sélectionné.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-5.png)

      La propriété **Valeur** apparaît à présent comme indiqué ici :

      ![Capture d’écran représentant la propriété « Value » avec l’expression résolue.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-6.png)

1. Enregistrez votre application logique.

Ajoutez une condition qui vérifie si la durée du trajet actuel est supérieure à une limite définie.

## <a name="compare-the-travel-time-with-limit"></a>Comparer le temps de trajet à une limite

1. Sous l’action **Créer une variable pour stocker la durée du trajet** , sélectionnez **Nouvelle étape**.

1. Sous **Choisir une opération**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `condition`. et dans la liste des actions, sélectionnez l’action **Condition**.

   ![Capture d’écran représentant l’action « Condition » sélectionnée](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-condition-action.png)

1. Renommez la condition à l’aide de cette description : `If travel time exceeds limit`.

1. Générez une condition qui vérifie si la valeur de la propriété **travelTime** dépasse la limite spécifiée comme décrit et indiqué ici :

   1. Dans la condition, cliquez sur la zone **Choisir une valeur** située dans la partie gauche de la condition.

   1. Dans la liste de contenu dynamique qui s’affiche, sous **Variables**, sélectionnez la propriété **travelTime**.

      ![Capture d’écran représentant la zone « Choisissez une valeur » sur le côté gauche de la condition avec la liste de contenu dynamique ouverte et la propriété « travelTime » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-left-side.png)

   1. Dans la zone de comparaison du milieu, sélectionnez l’opérateur **est supérieur à**.

   1. Dans la zone **Choisir une valeur** dans la partie droite de la condition, entrez cette limite : `15`

      Quand vous avez terminé, votre condition ressemble à cet exemple :

      ![Capture d’écran représentant la condition de fin de la comparaison entre le temps de trajet et la limite spécifiée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-check-travel-time.png)

1. Enregistrez votre application logique.

Ajoutez ensuite l’action à exécuter quand le temps de trajet dépasse la limite.

## <a name="send-email-when-limit-exceeded"></a>Envoyer un e-mail en cas de dépassement de la limite

Ajoutez une action qui vous avertit par e-mail lorsque la durée du trajet dépasse la limite que vous avez définie. Cet e-mail inclut la durée du trajet actuel et le temps supplémentaire nécessaire pour parcourir l’itinéraire spécifié.

1. Dans la branche **True** de la condition, sélectionnez **Ajouter une action**.

1. Sous **Choisir une opération**, sélectionnez **Standard**. Dans la zone de recherche, entrez `send email`. Comme la liste retourne un grand nombre de résultats, pour filtrer la liste, sélectionnez d’abord le connecteur de messagerie qui vous intéresse.

   Par exemple, si vous disposez d’un compte de courrier Outlook, sélectionnez le connecteur correspondant à votre type de compte :

   * Pour les comptes Azure professionnels ou scolaires, sélectionnez **Office 365 Outlook**.
   * Pour les comptes Microsoft personnels, sélectionnez **Outlook.com**.

   Cet exemple se poursuit avec la sélection d’Office 365 Outlook.

   ![Capture d’écran représentant la catégorie « Choisir une liste d’opérations » dont la catégorie « Standard » et le connecteur « Office 365 Outlook » sont sélectionnés.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-action-send-email.png)

1. Quand les actions du connecteur apparaissent, sélectionnez l’action d’envoi d’un e-mail, par exemple :

   ![Capture d’écran représentant l’action « Envoyer un e-mail » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-send-email-action.png)

1. Si vous n’avez pas encore de connexion, connectez-vous et authentifiez l’accès à votre compte de courrier lorsque vous y êtes invité.

   Azure Logic Apps crée une connexion à votre compte de messagerie.

1. Renommez l’action à l’aide de cette description : `Send email with travel time`.

1. Pour la propriété **To** (À), entrez l’adresse e-mail du destinataire. À des fins de test, vous pouvez utiliser votre adresse e-mail.

1. Pour la propriété **Subject** (Objet), spécifiez l’objet de l’e-mail, puis insérez la variable **travelTime** comme suit :

   1. Entrez le texte `Current travel time (minutes):` avec un espace de fin. Maintenez votre curseur dans la zone d’édition **Subject** (Objet) pour que la liste de contenu dynamique reste ouverte.

   1. Dans l’en-tête **Variables** de la liste de contenu dynamique, sélectionnez **Voir plus** afin d’afficher la variable **travelTime**.

      ![Capture d’écran représentant la liste de contenu dynamique avec la section « Variable » et « Voir plus » sélectionnés.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/find-travelTime-variable.png)

      > [!NOTE]
      > La liste de contenu dynamique n’affiche pas automatiquement la variable **travelTime** , car la propriété **Subject** attend une valeur chaîne, tandis que **travelTime** est une valeur entière.

      ![Capture d’écran représentant la liste de contenu dynamique avec la variable « travelTime » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-travelTime-variable.png)

1. Pour la propriété **Body**, spécifiez le contenu du corps de l’e-mail comme suit :

   1. Entrez le texte `Add extra travel time (minutes):` avec un espace de fin. Maintenez votre curseur dans la zone d’édition **Body** (Corps) pour que la liste de contenu dynamique reste ouverte.

   1. Dans la liste de contenu dynamique, sélectionnez **Expression**, qui affiche l’éditeur d’expression.

      ![Capture d’écran représentant la liste de contenu dynamique avec l’option « Expression » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings.png)

   1. Dans l’éditeur d’expressions, entrez `sub(,15)` afin de pouvoir calculer le nombre de minutes au-delà de la limite que vous avez définie : 

      ![Capture d’écran représentant l’expression « sub(,15) » entrée dans l’éditeur d’expression.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-2.png)

   1. Dans l’expression, placez votre curseur entre la parenthèse ouvrante ( **(** ) et la virgule ( **,** ), puis sélectionnez **Contenu dynamique**.

      ![Capture d’écran indiquant où placer le curseur dans l’expression « sub(,15) » alors que « Contenu dynamique » est sélectionné.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-3.png)

   1. Sous **Variables**, sélectionnez **travelTime**.

      ![Capture d’écran représentant la liste de contenu dynamique avec la variable « travelTime » sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-4.png)

   1. Une fois la propriété résolue dans l’expression, sélectionnez **OK**.

      ![Capture d’écran représentant la liste de contenu dynamique et « OK » sélectionné.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-5.png)

      La propriété **Corps** apparaît à présent comme indiqué ici :

      ![Capture d’écran représentant la liste de contenu dynamique avec l’expression résolue dans la propriété « Body » de l’action de messagerie.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-6.png)

1. Enregistrez votre application logique.

Ensuite, testez et exécutez votre application logique, qui ressemble désormais à l’exemple suivant :

![Capture d’écran représentant l’exemple de workflow d’application logique terminé](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-finished.png)

## <a name="run-your-logic-app"></a>Exécuter votre application logique

Pour lancer manuellement votre application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

* Si le temps de trajet actuel reste en dessous de la limite que vous avez définie, votre application logique ne fait rien d’autre et attend l’intervalle suivant avant de procéder à une nouvelle vérification. 

* Mais si le temps de trajet actuel dépasse la limite que vous avez définie, vous obtenez un e-mail indiquant le temps de trajet actuel et le nombre de minutes au-dessus de la limite définie. Voici un exemple d’e-mail envoyé par votre application logique :

  ![Capture d’écran représentant un exemple d’e-mail qui indique le temps de trajet actuel et le temps de trajet supplémentaire qui dépasse la limite spécifiée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/received-example-email-notification.png)

  > [!TIP]
  > Si vous ne recevez pas d’e-mail, vérifiez le dossier Courrier indésirable de votre messagerie. Il se peut que le filtre de courrier indésirable redirige ces types d’e-mails. Sinon, si vous ne savez pas si votre application logique s’est correctement exécutée, consultez [Dépanner votre application logique](../logic-apps/logic-apps-diagnosing-failures.md).

Félicitations ! Vous venez de créer et d’exécuter une application logique périodique basée sur une planification. 

Pour créer d’autres applications logiques qui utilisent le déclencheur **Périodicité**, découvrez ces modèles, qui sont disponibles après que vous avez créé une application logique :

* Recevez des rappels quotidiens.
* Supprimez d’anciens objets blob Azure.
* Ajoutez un message à une file d’attente Stockage Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Votre application logique poursuit son exécution jusqu’à ce que vous la désactiviez ou la supprimiez. Quand vous n’avez plus besoin de l’exemple d’application logique, supprimez le groupe de ressources qui contient votre application logique et les ressources associées.

1. Dans la zone de recherche de Portail Azure, entrez le nom du groupe de ressources que vous avez créé. Dans les résultats, sous **Groupes de ressources**, sélectionnez le groupe de ressources.

   Cet exemple crée le groupe de ressources `LA-TravelTime-RG`.

   ![Capture d’écran dans laquelle la zone de recherche Azure comporte « la-travel-time-rg » et **LA-TravelTime-RG** est sélectionnée.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/find-resource-group.png)

   > [!TIP]
   > Si la page d’accueil Azure affiche le groupe de ressources sous **Ressources récentes**, vous pouvez y sélectionner le groupe.

1. Dans le menu du groupe de ressources, vérifiez que **Vue d’ensemble** est sélectionné. Dans la barre d’outils du volet **Vue d’ensemble**, sélectionnez **Supprimer un groupe de ressources**.

   ![Capture d’écran représentant le volet « Vue d’ensemble » du groupe de ressources ; « Supprimer le groupe de ressources » est sélectionné dans la barre d’outils du volet.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/delete-resource-group.png)

1. Dans le volet de confirmation qui s’affiche, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une application logique qui vérifie le trafic selon une planification spécifiée (les matins des jours ouvrables), et effectue une action (envoi d’un e-mail) lorsque la durée du trajet excède une limite définie. À présent, découvrez comment créer une application logique qui envoie des demandes de liste de diffusion pour approbation en intégrant des services Azure et Microsoft et d’autres applications SaaS (Software-as-a-Service).

> [!div class="nextstepaction"]
> [Gérer les demandes de liste de diffusion](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
