---
title: Créer plus rapidement des workflows d’application logique à partir de modèles prédéfinis
description: Générer rapidement des workflows d’application logique à l’aide de modèles prédéfinis fournis par Azure Logic Apps
services: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 10/15/2017
ms.openlocfilehash: 7eca9b4a8cf09f2f35d56dc22fc25169be192f47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87066237"
---
# <a name="create-logic-app-workflows-from-prebuilt-templates"></a>Créer des workflows d’application logique à partir de modèles prédéfinis

Pour vous amener à commencer à créer des workflows plus rapidement, Logic Apps fournit des modèles qui sont des applications logiques prédéfinies qui suivent des modèles couramment utilisés. Utilisez ces modèles tels quels ou modifiez-les pour les adapter à votre scénario.

Voici certaines catégories de modèles :

| Type de modèle | Description | 
| ------------- | ----------- | 
| Modèles de cloud d’entreprise | Il s’agit de modèles pour l’intégration de Blob Azure, de Dynamics CRM, de Salesforce et de Box, ainsi que d’autres connecteurs pour les besoins cloud de votre entreprise. Par exemple, vous pouvez utiliser ces modèles pour organiser des responsables commerciaux ou sauvegarder vos données de dossier d’entreprise. | 
| Modèles de productivité personnelle | Améliorez la productivité personnelle en définissant des rappels quotidiens, en convertissant des éléments de travail importants en listes de tâches, et en automatisant des tâches fastidieuses de façon à les réduire à une étape d’approbation par un utilisateur unique. | 
| Modèles Cloud clients | Pour l’intégration de services de réseau social tels que Twitter, Slack et le courrier. Utile pour le renforcement des initiatives de marketing via réseau social. Ces modèles incluent également des tâches telles que la copie de cloud, qui augmente la productivité en permettant de gagner du temps sur des opérations traditionnellement répétitives. | 
| Modèles de pack d’intégration d’entreprise | Pour la configuration de pipelines VETER (valider, extraire, transformer, enrichir, router), la réception de documents EDI X12 via AS2 et leur conversion au format XML, ainsi que le traitement de messages X12, EDIFACT et AS2. | 
| Modèles de modèles de protocole | Pour l’implémentation de modèles de protocole tels que requête-réponse sur HTTP et les intégrations entre FTP et SFTP. Utilisez ces modèles tels quels, ou appuyez-vous dessus pour générer des modèles de protocole complexe. | 
||| 

Si vous n’avez pas d’abonnement Azure, [créez un compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer. Pour plus d’informations sur la génération d’une application logique, voir [Créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-logic-apps-from-templates"></a>Créer des applications logiques à partir de modèles

1. Si ce n’est pas déjà fait, connectez-vous au [portail Azure](https://portal.azure.com "Portail Azure").

2. Dans le menu principal Azure, choisissez **Créer une ressource** > **Intégration Entreprise** > **Application logique**.

   ![portail Azure, nouveau, intégration d’entreprise, application logique](./media/logic-apps-create-logic-apps-from-templates/azure-portal-create-logic-app.png)

3. Créez votre application logique avec les paramètres spécifiés dans le tableau sous cette image :

   ![Spécifier les détails de l’application logique](./media/logic-apps-create-logic-apps-from-templates/logic-app-settings.png)

   | Paramètre | Valeur | Description | 
   | ------- | ----- | ----------- | 
   | **Nom** | *nom-de-votre-application-logique* | Donnez un nom unique à l’application logique. | 
   | **Abonnement** | *nom-de-votre-abonnement-Azure* | Sélectionnez l’abonnement Azure que vous souhaitez utiliser. | 
   | **Groupe de ressources** | *nom-de-votre-groupe-de-ressources-Azure* | Créez ou sélectionnez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) pour cette application logique et pour organiser toutes les ressources associées à celle-ci. | 
   | **Lieu** | *région-de-votre-centre-de-données-Azure* | Sélectionnez la région du centre de données où déployer votre application logique, par exemple USA Ouest. | 
   | **Log Analytics** | **Désactivé** (par défaut) ou **Activé** | Configurez la [journalisation des diagnostics](../logic-apps/monitor-logic-apps-log-analytics.md) pour votre application logique à l’aide de [journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md). Pour cela, vous devez déjà disposer d’un espace de travail Log Analytics. | 
   |||| 

4. Lorsque vous êtes prêt, sélectionnez **Épingler au tableau de bord**. De cette façon, votre application logique apparaît automatiquement sur votre tableau de bord Azure et s’ouvre après le déploiement. Cliquez sur **Créer**.

   > [!NOTE]
   > Si vous ne souhaitez pas épingler votre application logique, vous devez trouver et ouvrir manuellement votre application logique après le déploiement pour pouvoir continuer.

   Une fois qu’Azure a déployé votre application logique, le Concepteur Logic Apps s’ouvre et affiche une page contenant une vidéo de présentation. 
   Sous la vidéo, vous pouvez trouver des modèles courants d’applications logiques. 

5. Faites défiler au-delà de la vidéo de présentation et les déclencheurs courants jusqu’à **Modèles**. Choisissez un modèle prédéfini. Par exemple :

   ![Choisir un modèle d’application logique](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   > [!TIP]
   > Pour créer votre application logique à partir de rien, choisissez **Application logique vide**.

   Lorsque vous sélectionnez un modèle prédéfini, vous pouvez voir des informations supplémentaires sur ce modèle. 
   Par exemple :

   ![Choisir un modèle prédéfini](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

6. Pour continuer avec le modèle sélectionné, choisissez **Utiliser ce modèle**. 

7. Selon les connecteurs présents dans le modèle, vous êtes invité à effectuer l’une des étapes suivantes :

   * Connectez-vous avec vos informations d’identification aux systèmes ou services référencés par le modèle.

   * Créez des connexions pour les services ou systèmes référencés par le modèle. Pour créer une connexion, fournissez un nom pour celle-ci et, si nécessaire, sélectionnez la ressource que vous souhaitez utiliser. 

   * Si vous avez déjà configuré ces connexions, choisissez **Continuer**.

   Par exemple :

   ![Créer des connexions](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   Lorsque vous avez terminé, votre application logique s’ouvre et s’affiche dans le Concepteur d’applications logiques.

   > [!TIP]
   > Pour revenir à la visionneuse de modèle, choisissez **Modèles** dans la barre d’outils du concepteur. Cette action ayant pour effet de supprimer toutes les modifications non enregistrées, un message d’avertissement s’affiche pour confirmer votre demande.

8. Poursuivez l’élaboration de votre application logique.

   > [!NOTE] 
   > De nombreux modèles incluent des connecteurs qui pourraient déjà figurer dans des propriétés requises. Toutefois, certains modèles pourraient encore nécessiter que vous fournissiez des valeurs avant de pouvoir déployer correctement l’application logique. Si vous essayez de déployer sans compléter les champs de propriété manquants, vous obtenez un message d’erreur. 

## <a name="update-logic-apps-with-templates"></a>Mettre à jours des applications logiques avec des modèles

1. Dans le [portail Azure](https://portal.azure.com "Portail Azure"), recherchez et ouvrez votre application logique dans le concepteur d’application logique.

2. Dans la barre d’outils du concepteur, choisissez **Modèles**. Cette action ayant pour effet de supprimer toutes les modifications non enregistrées, un message d’avertissement s’affiche pour vous permettre confirmer votre volonté de continuer. Pour confirmer, choisissez **OK**. Par exemple :

   ![Choisissez « Modèles »](./media/logic-apps-create-logic-apps-from-templates/logic-app-update-existing-with-template.png)

3. Faites défiler au-delà de la vidéo de présentation et les déclencheurs courants jusqu’à **Modèles**. Choisissez un modèle prédéfini. Par exemple :

   ![Choisir un modèle d’application logique](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   Lorsque vous sélectionnez un modèle prédéfini, vous pouvez voir des informations supplémentaires sur ce modèle. 
   Par exemple :

   ![Choisir un modèle prédéfini](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

4. Pour continuer avec le modèle sélectionné, choisissez **Utiliser ce modèle**. 

5. Selon les connecteurs présents dans le modèle, vous êtes invité à effectuer l’une des étapes suivantes :

   * Connectez-vous avec vos informations d’identification aux systèmes ou services référencés par le modèle.

   * Créez des connexions pour les services ou systèmes référencés par le modèle. Pour créer une connexion, fournissez un nom pour celle-ci et, si nécessaire, sélectionnez la ressource que vous souhaitez utiliser. 

   * Si vous avez déjà configuré ces connexions, choisissez **Continuer**.

   ![Créer des connexions](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   À présent, votre application logique s’ouvre et apparaît dans le Concepteur d’applications logiques.

8. Poursuivez l’élaboration de votre application logique. 

   > [!TIP]
   > Si vous n’avez pas enregistré vos modifications, vous pouvez ignorer votre travail et revenir à votre application logique précédente. Dans la barre d’outils du concepteur, choisissez **Ignorer**.

> [!NOTE] 
> De nombreux modèles incluent des connecteurs qui pourraient déjà figurer dans des propriétés requises. Toutefois, certains modèles pourraient encore nécessiter que vous fournissiez des valeurs avant de pouvoir déployer correctement l’application logique. Si vous essayez de déployer sans compléter les champs de propriété manquants, vous obtenez un message d’erreur.

## <a name="deploy-logic-apps-built-from-templates"></a>Déployer des applications logiques générées à partir de modèles

Après avoir apporté vos modifications au modèle, vous pouvez les enregistrer. Cette action a aussi pour effet de publier automatiquement votre application logique.

Dans la barre d’outils du concepteur, choisissez **Enregistrer**.

![Enregistrer et publier votre application logique](./media/logic-apps-create-logic-apps-from-templates/logic-app-save.png)  

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des idées de fonctionnalités ou pour en soumettre, visitez le [site de commentaires des utilisateurs Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’applications logiques via des exemples, des scénarios, des témoignages de clients et des procédures pas à pas.

> [!div class="nextstepaction"]
> [Passez en revue des exemples d’applications logiques, des scénarios et des procédures pas à pas](../logic-apps/logic-apps-examples-and-scenarios.md)
