---
title: 'Démarrage rapide : Créer votre premier workflow Logic Apps dans le portail Azure'
description: Suivez ce guide de démarrage rapide pour créer votre premier workflow Logic Apps automatisé dans le portail Azure. Découvrez les concepts de base des solutions d’intégration de systèmes et d’intégration d’applications d’entreprise (EAI) dans Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 10/30/2020
ms.openlocfilehash: 9d402599c4d6732ce92b8c64af6f660bcedbc4ba
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455059"
---
# <a name="quickstart-create-your-first-logic-apps-workflow---azure-portal"></a>Démarrage rapide : Créer votre premier workflow Logic Apps dans le portail Azure

Ce guide de démarrage rapide explique comment créer votre premier workflow dans [Azure Logic Apps](logic-apps-overview.md) par le biais du [portail Azure](https://portal.azure.com). Ce guide d’introduction explique également les concepts de base du service Logic Apps, notamment comment créer une application logique, puis comment lui ajouter un déclencheur et une action, et enfin comment la tester. Suivez ce guide de démarrage rapide pour créer un exemple d’application logique qui vérifie régulièrement un flux RSS et envoie un e-mail de notification pour chaque nouvel élément publié. La capture d’écran ci-dessous montre le workflow général de cet exemple d’application logique :

![Capture d’écran du concepteur d’applications logiques, qui illustre un exemple d’application logique où le déclencheur est un flux RSS et où l’action est l’envoi d’un e-mail.](./media/quickstart-create-first-logic-app-workflow/quickstart-workflow-overview.png)

<a name="prerequisites"></a>

Si vous souhaitez apprendre à créer et à gérer votre première application logique par le biais d’autres interfaces et applications, consultez ces guides de démarrage rapide pour Logic Apps : 

* [Créer et gérer des applications logiques à l’aide d’Azure CLI](quickstart-logic-apps-azure-cli.md)
* [Créer et gérer des applications logiques dans Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md)
* [Créer et gérer des applications logiques dans Visual Studio](quickstart-create-logic-apps-with-visual-studio.md)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’en avez pas, [inscrivez-vous pour obtenir un compte Azure gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un compte de messagerie d’un service pris en charge par Logic Apps (comme Office 365 Outlook ou Outlook.com). Pour connaître les autres fournisseurs de messagerie pris en charge, [consultez la liste des connecteurs](/connectors/).

    > [!IMPORTANT]
    > Si vous utilisez le [connecteur Gmail](/connectors/gmail/), sachez que seuls les comptes G Suite peuvent utiliser ce connecteur sans restriction dans Logic Apps. Si vous avez un compte Gmail personnel, vous pouvez uniquement utiliser ce connecteur avec certains services approuvés par Google, sauf si vous [créez une application cliente Google pour les besoins d’authentification auprès de votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

<a name="create-logic-app"></a>

## <a name="create-your-logic-app"></a>Créer votre application logique

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche du portail Azure, entrez `logic apps`, puis sélectionnez **Logic Apps**.

   ![Capture d’écran du portail Azure, montrant la zone de recherche avec le terme de recherche « logic apps », et le résultat de recherche « Logic Apps » sélectionné.](./media/quickstart-create-first-logic-app-workflow/find-select-logic-apps.png)

1. Dans la page **Logic Apps**, sélectionnez **Ajouter**.

   ![Capture d’écran de la page du service Logic Apps dans le portail Azure, montrant la liste des applications logiques et le bouton « Ajouter » sélectionné.](./media/quickstart-create-first-logic-app-workflow/add-new-logic-app.png)

1. Dans le volet **Logic Apps**, entrez les détails de base et les paramètres de votre application logique. Créez un [groupe de ressources](../azure-resource-manager/management/overview.md#terminology) pour les besoins de cet exemple d’application logique.
    
   | Property | Valeur | Description |
   |----------|-------|-------------|
   | **Nom** | <*logic-app-name*> | Nom de votre application logique. Ce nom doit être unique d’une région à l’autre. Le nom peut contenir uniquement des lettres, des chiffres, des traits d’union (`-`), des traits de soulignement (`_`), des parenthèses (`(`, `)`) et des points (`.`). Cet exemple utilise « My-First-Logic-App ». |
   | **Abonnement** | <*Azure-subscription-name*> | Nom de votre abonnement Azure. |
   | **Groupe de ressources** | <*nom-groupe-de-ressources-Azure*> | Nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md#terminology) où vous créez l’application logique. Ce nom doit être unique d’une région à l’autre. Cet exemple utilise « My-First-LA-RG ». |
   | **Lieu** | <*Azure-region*> | Région Azure où seront stockées les données de votre application logique. Cet exemple utilise la région « USA Ouest ». |
   | **Log Analytics** | Désactivé | Paramètre de la journalisation des diagnostics, défini sur **Désactivé** par défaut. Pour cet exemple, conservez le paramètre **Désactivé**. |
   ||||

   ![Capture d’écran de la page de création dans Logic Apps, montrant les détails de la nouvelle application logique.](./media/quickstart-create-first-logic-app-workflow/create-logic-app-settings.png)

1. Quand vous êtes prêt, sélectionnez **Vérifier + créer**. Vérifiez les détails que vous avez fournis, puis sélectionnez **Créer**.

1. Une fois qu’Azure a déployé votre application, sélectionnez **Accéder à la ressource**. Ou bien, vous pouvez rechercher et sélectionner votre application logique en tapant son nom dans la zone de recherche.

   ![Capture d’écran de la page de déploiement de ressource, montrant le bouton « Accéder à la ressource » sélectionné.](./media/quickstart-create-first-logic-app-workflow/go-to-new-logic-app-resource.png)

   Le Concepteur d’application logique s’ouvre et affiche une page contenant une vidéo de présentation et les déclencheurs couramment utilisés. Sous **Modèles**, sélectionnez **Application logique vide**.

   ![Capture d’écran du concepteur d’applications logiques, montrant la galerie de modèles et le modèle « Application logique vide » sélectionné.](./media/quickstart-create-first-logic-app-workflow/choose-logic-app-template.png)

Ensuite, [ajoutez un déclencheur à votre application logique](#add-rss-trigger).

<a name="add-rss-trigger"></a>

## <a name="add-the-rss-trigger"></a>Ajouter le déclencheur RSS

Chaque application logique doit démarrer avec un [déclencheur](../logic-apps/logic-apps-overview.md#how-do-logic-apps-work), qui s’active lorsqu’un événement spécifique se produit ou qu’une condition particulière est remplie. Chaque fois que le déclencheur détecte de nouveaux éléments, il s’active, et le moteur Logic Apps crée une instance d’application logique qui démarre et exécute votre workflow. Si le déclencheur ne détecte aucun nouvel élément, il ne s’active pas, et le moteur ne crée pas d’instance ni n’exécute le workflow à ce point de vérification.

Dans cet exemple de démarrage rapide, après avoir [créé une application logique](#create-your-logic-app), vous ajoutez un déclencheur qui recherche les nouveaux éléments dans un flux RSS et qui se déclenche quand il en trouve. Vous pouvez également créer des applications logiques avec différents types de déclencheurs, comme cela est expliqué dans le tutoriel sur la [création de workflows basés sur l’approbation automatique](tutorial-process-mailing-list-subscriptions-workflow.md).

1. Dans le **Concepteur d’application logique**, sous la zone de recherche, sélectionnez **Tout**.

1. Pour rechercher le connecteur RSS, dans la zone de recherche, entrez `rss`. Dans la liste **Déclencheurs**, sélectionnez le déclencheur RSS **Lors de la publication d’un élément de flux**.

   ![Capture d’écran du concepteur d’applications logiques, montrant la zone de recherche avec le terme « rss », et le déclencheur RSS « Lors de la publication d’un élément de flux » sélectionné.](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-new-feed-item.png)

1. Entrez l’URL du flux RSS pour votre déclencheur. Ensuite, planifiez le déclencheur en définissant l’intervalle et la fréquence.

   | Property | Valeur | Description |
   | -------- | ----- | ----------- |
   | **URL du flux RSS** | <*URL-flux-RSS*> | URL du flux RSS à superviser. Cet exemple utilise le flux RSS du Wall Street Journal sur `https://feeds.a.dj.com/rss/RSSMarketsMain.xml`. Pour cet exemple, vous pouvez choisir tout autre flux RSS qui ne nécessite pas d’autorisation HTTP. Choisissez un flux RSS qui publie fréquemment, afin de pouvoir tester votre application logique facilement plus tard. |
   | **Intervalle** | 1 | Nombre d’intervalles d’attente entre les vérifications du flux RSS. Cet exemple utilise des intervalles d’une minute. |
   | **Fréquence** | Minute | Unité de temps de chaque intervalle entre les vérifications du flux RSS. Cet exemple utilise des intervalles d’une minute. |
   ||||

   ![Capture d’écran montrant le Concepteur d’applications logiques avec les paramètres du déclencheur RSS, notamment l’URL, la fréquence et l’intervalle RSS](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-settings.png)

1. Pour le moment, réduisez les détails du déclencheur en cliquant à l’intérieur de sa barre de titre.

   ![Capture d’écran montrant le Concepteur d’applications logiques avec la forme d’application logique réduite.](./media/quickstart-create-first-logic-app-workflow/collapse-trigger-shape.png)

1. Enregistrez votre application logique en sélectionnant **Enregistrer** dans la barre d’outils du concepteur.

Votre application logique est à présent opérationnelle. Toutefois, elle ne fait rien d’autre que vérifier le flux RSS. Vous allez maintenant lui [ajouter une action](#add-email-action) pour définir ce qui se produit à l’activation du déclencheur.

<a name="add-email-action"></a>

## <a name="add-the-send-email-action"></a>Ajouter l’action « Envoyer un courrier électronique »

Après avoir [ajouté un déclencheur à votre application logique](#add-rss-trigger), vous devez ajouter une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) pour définir ce que doit faire l’application logique quand elle vérifie le flux RSS et qu’elle détecte un nouvel élément publié. Vous pouvez également créer des applications logiques avec des actions bien plus complexes, comme cela est décrit dans le tutoriel sur le [traitement des e-mails avec Logic Apps, Azure Functions et Stockage Azure](./tutorial-process-email-attachments-workflow.md).

> [!NOTE]
> Cet exemple utilise Office 365 Outlook comme service de messagerie. Si vous utilisez un autre service de messagerie pris en charge dans votre application logique, l’interface utilisateur se présentera peut-être différemment. Les concepts de base pour la connexion à un autre service de messagerie restent néanmoins les mêmes.

1. Sous le déclencheur **Lors de la publication d’un élément de flux**, sélectionnez **Nouvelle étape**.

   ![Capture d’écran du concepteur d’applications logiques, montrant un workflow avec le bouton « Nouvelle étape » sélectionné.](./media/quickstart-create-first-logic-app-workflow/add-new-step-under-trigger.png)

1. Sous **Choisir une action** et la zone de recherche, sélectionnez **Tous**.

1. Dans la zone de recherche, entrez `send an email` afin de pouvoir rechercher les connecteurs qui proposent cette action. Pour filtrer la liste d’actions pour un service ou une application spécifique, vous pouvez commencer par sélectionner ce service ou cette application.

   Par exemple, si vous utilisez un compte professionnel ou scolaire Microsoft et souhaitez utiliser Office 365 Outlook, sélectionnez **Office 365 Outlook**. Ou bien, si vous utilisez un compte Microsoft personnel, vous pouvez sélectionner Outlook.com. Cet exemple se poursuit avec Office 365 Outlook :

   ![Capture d’écran du concepteur d’applications logiques, montrant l’étape de l’action avec le connecteur de messagerie « Office 365 Outlook » sélectionné.](./media/quickstart-create-first-logic-app-workflow/select-connector.png)

   Vous pouvez désormais rechercher et sélectionner plus facilement l’action que vous souhaitez utiliser, par exemple `send an email` :

   ![Capture d’écran du concepteur d’applications logiques, montrant la liste des actions filtrées pour le connecteur de messagerie « Office 365 Outlook ».](./media/quickstart-create-first-logic-app-workflow/filtered-actions-list.png)

1. Si le connecteur de messagerie sélectionné vous invite à authentifier votre identité, faites-le aussitôt. Pour que cet exemple fonctionne, vous devez créer une connexion entre votre application logique et votre service de messagerie. 

    > [!NOTE]
    > Cet exemple montre l’authentification manuelle auprès du connecteur Office 365 Outlook. Toutefois, les autres connecteurs peuvent prendre en charge des types d’authentification différents.
    > Vous pouvez également gérer l’authentification pour vos applications logiques d’une autre manière, en fonction de votre cas d’usage. Par exemple, si vous utilisez des modèles Azure Resource Manager dans le cadre du déploiement, vous pouvez les paramétrer afin de renforcer la sécurité des entrées qui changent souvent, comme les détails de connexion. Pour plus d’informations, consultez les rubriques suivantes :
   > * [Paramètres du modèle pour le déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters)
   > * [Autoriser des connexions OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections)
   > * [Authentifier l’accès avec des identités managées](../logic-apps/create-managed-service-identity.md)
   > * [Authentifier les connexions pour le déploiement d’applications logiques](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#authenticate-connections)

1. Dans l’action **Envoyer un courrier électronique**, entrez les informations à inclure dans l’e-mail de notification.

   1. Dans la zone **À**, entrez l’adresse e-mail du destinataire. Pour cet exemple, utilisez votre propre adresse e-mail.

        > [!NOTE]
        > La liste **Ajouter du contenu dynamique** s’affiche quand vous cliquez à l’intérieur de la zone **À** ainsi que dans d’autres zones d’entrée dans le concepteur d’applications logiques. Cet exemple utilise du contenu dynamique dans une étape ultérieure. La liste **Ajouter du contenu dynamique** affiche toutes les sorties disponibles trouvées à l’étape précédente, et que vous pouvez utiliser comme entrées pour l’action actuelle.

   1. Dans la zone **Objet**, entrez l’objet de votre e-mail de notification. Pour cet exemple, entrez le texte suivant avec un espace vide à la fin : `New RSS item: `

      ![Capture d’écran du concepteur d’applications logiques, montrant l’action « Envoyer un courrier électronique » et le curseur d’entrée placé à l’intérieur de la zone de propriété « Objet ».](./media/quickstart-create-first-logic-app-workflow/send-email-subject.png)

   1. Dans la liste **Ajouter du contenu dynamique**, sélectionnez **Titre du flux**, qui est la sortie du déclencheur **Lors de la publication d’un élément de flux**. Votre e-mail de notification récupère le titre de l’élément RSS dans cette sortie.

      ![Capture d’écran du concepteur d’applications logiques, montrant l’action « Envoyer un courrier électronique » et le curseur d’entrée placé à l’intérieur de la zone de propriété « Objet » avec une liste de contenu dynamique ouverte et la sortie « Titre du flux » sélectionnée.](./media/quickstart-create-first-logic-app-workflow/send-email-subject-dynamic-content.png)

      > [!TIP]
      > Si la liste de contenu dynamique n’affiche aucune sortie retournée par le déclencheur **Lors de la publication d’un élément de flux**, en regard de l’en-tête de l’action, sélectionnez **Afficher plus**.
      > 
      > ![Capture d’écran du concepteur d’applications logiques, montrant la liste de contenu dynamique ouverte et le bouton « Afficher plus » sélectionné pour le déclencheur.](./media/quickstart-create-first-logic-app-workflow/dynamic-content-list-see-more-actions.png)

      Lorsque vous avez terminé, l’objet de l’e-mail ressemble à l’exemple suivant :

      ![Capture d’écran du concepteur d’applications logiques, montrant l’action « Envoyer un courrier électronique » et un exemple d’objet d’e-mail contenant la propriété « Titre du flux ».](./media/quickstart-create-first-logic-app-workflow/send-email-feed-title.png)

      Si une boucle « For each » s’affiche dans le concepteur, c’est que vous avez sélectionné un jeton de tableau, par exemple, le jeton **categories-Item**. Pour ces types de jeton, le concepteur ajoute automatiquement cette boucle autour de l’action qui référence ce jeton. De cette façon, votre application logique effectue la même action sur chaque élément du tableau. Pour supprimer la boucle, sélectionnez le bouton **points de suspension** (**...**) sur la barre de titre de la boucle, puis **Supprimer**.

   1. Dans la zone **Corps**, entrez le contenu du corps de l’e-mail. Dans cet exemple, le contenu comprend trois propriétés avec un texte descriptif pour chacune : `Title:` (propriété **Titre du flux**), `Date published:` (propriété **Flux publié le**) et `Link:` (propriété **Lien de flux principal**). Pour ajouter des lignes vides dans une zone d’édition, appuyez sur Maj + Entrée.

      | Property | Description |
      |----------|-------------|
      | **Titre du flux** | Titre de l’élément. |
      | **Flux publié le** | Date et heure de publication de l’élément. |
      | **Lien du flux principal** | URL de l’élément. |
      |||

      ![Capture d’écran du concepteur d’applications logiques, montrant l’action « Envoyer un courrier électronique » et des propriétés sélectionnées dans la zone « Corps ».](./media/quickstart-create-first-logic-app-workflow/send-email-body.png)

1. Enregistrez votre application logique. Dans le menu du concepteur, sélectionnez **Enregistrer**.

Vous allez à présent [vérifier que votre application logique fonctionne comme prévu](#test-logic-app).

<a name="test-logic-app"></a>

## <a name="run-your-logic-app"></a>Exécuter votre application logique

Après avoir créé votre exemple d’application logique, vous devez vérifier que votre workflow est correctement configuré. Vous pouvez attendre que l’application logique vérifie votre flux RSS au moment planifié. Vous pouvez aussi exécuter manuellement votre application logique en sélectionnant **Exécuter** dans la barre d’outils du concepteur d’applications logiques, comme illustré dans la capture d’écran ci-dessous. 

Si le flux RSS a de nouveaux éléments, votre application logique envoie un e-mail pour chaque nouvel élément. Dans le cas contraire, l’application logique attend le prochain intervalle pour lancer une nouvelle vérification du flux RSS. 

![Capture d’écran du concepteur d’applications logiques, montrant le bouton « Exécuter » sélectionné dans la barre d’outils du concepteur.](./media/quickstart-create-first-logic-app-workflow/run-logic-app-test.png)

La capture d’écran suivante illustre un exemple d’e-mail de notification provenant de cet exemple d’application logique. L’e-mail contient les détails de chaque élément de flux RSS sélectionné dans le concepteur, ainsi que le texte descriptif ajouté pour chacun.

![Capture d’écran d’Outlook, montrant un exemple d’e-mail reçu après la publication d’un nouvel élément de flux RSS, avec le titre de l’élément, la date de sa publication et le lien pour y accéder.](./media/quickstart-create-first-logic-app-workflow/monitor-rss-feed-email.png)

Si vous ne recevez pas l’e-mail de notification attendu de l’application logique :

* Vérifiez si le message n’a pas été incorrectement filtré et placé dans votre dossier de courrier indésirable ou de spam.
* Assurez-vous que le flux RSS que vous utilisez a publié des éléments depuis la dernière vérification planifiée ou manuelle.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez fini de tester cet exemple d’application logique, nettoyez l’application logique et toutes les ressources associées en supprimant le groupe de ressources que vous avez créé pour cet exemple.

> [!NOTE]
> Quand vous [supprimez une application logique](manage-logic-apps-with-azure-portal.md#delete-logic-apps), aucune nouvelle exécution n’est instanciée. Toutes les exécutions en cours et en attente sont annulées. Si vous avez des milliers d’exécutions, l’annulation peut prendre beaucoup de temps.

1. Dans la zone de recherche Azure, entrez `resource groups`, puis sélectionnez **Groupes de ressources**.

   ![Capture d’écran montrant la zone de recherche du portail Azure avec le terme de recherche « resource groups ».](./media/quickstart-create-first-logic-app-workflow/find-resource-groups.png)

1. Recherchez et sélectionnez le groupe de ressources de votre application logique. Dans le volet **Vue d’ensemble**, sélectionnez **Supprimer un groupe de ressources**.

   ![Capture d’écran montrant le portail Azure avec le groupe de ressources sélectionné et le bouton « Supprimer le groupe de ressources ».](./media/quickstart-create-first-logic-app-workflow/delete-resource-group.png)

1. Dans le volet de confirmation, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.

   ![Capture d’écran montrant le portail Azure avec le volet de confirmation et le nom du groupe de ressources entré à supprimer.](./media/quickstart-create-first-logic-app-workflow/delete-resource-group-2.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé votre première application logique par le biais du portail Azure. Cette application avait pour but de vérifier un flux RSS pour détecter les mises à jour selon une planification, puis d’envoyer un e-mail de notification pour chaque nouvel élément de flux publié. 

Pour savoir comment créer des workbooks plus complexes basés sur des planifications dans Logic Apps, consultez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Contrôler le trafic à l’aide d’une application logique planifiée](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)