---
title: Créer des workflows Logic Apps (préversion) dans le portail Azure
description: Générez et exécutez des workflows pour les scénarios d’automatisation et d’intégration avec Azure Logic Apps (préversion) dans le portail Azure.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, az-logic-apps-dev
ms.topic: conceptual
ms.date: 12/07/2020
ms.openlocfilehash: d10689937a037469399863395e0190e399334bd3
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96924008"
---
# <a name="create-stateful-and-stateless-workflows-in-the-azure-portal-with-azure-logic-apps-preview"></a>Créer des workflows avec état et sans état dans le portail Azure à l’aide d’Azure Logic Apps (préversion)

> [!IMPORTANT]
> Cette fonctionnalité en préversion publique est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Grâce à [Azure Logic Apps (préversion)](logic-apps-overview-preview.md), vous pouvez créer des solutions d’automatisation et d’intégration entre les applications, les données, les services cloud et les systèmes en créant et en exécutant des applications logiques qui incluent des [workflows *avec état* et *sans état*](logic-apps-overview-preview.md#stateful-stateless) dans le portail Azure en commençant par le nouveau type de ressource **Application logique (préversion)** . Grâce à ce nouveau type d’application logique, vous pouvez créer plusieurs workflows alimentés par le runtime d’Azure Logic Apps (préversion) remanié, qui offre de la portabilité, de meilleures performances et de la flexibilité pour le déploiement et l’exécution dans divers environnements d’hébergement, non seulement dans Azure, mais également dans les conteneurs Docker. Pour en savoir plus sur le nouveau type d’application logique, consultez [Vue d’ensemble de la préversion Azure Logic Apps](logic-apps-overview-preview.md).

![Capture d’écran montrant le portail Azure et le concepteur de workflow pour la ressource « Application logique (préversion) ».](./media/create-stateful-stateless-workflows-azure-portal/azure-portal-logic-apps-overview.png)

Dans le portail Azure, vous pouvez commencer par créer une nouvelle ressource **Application logique (préversion)** . Vous pouvez également commencer par [créer un projet dans Visual Studio Code à l’aide de l’extension Azure Logic Apps (préversion)](create-stateful-stateless-workflows-visual-studio-code.md), mais les deux approches vous permettent de déployer et d’exécuter votre application logique dans les mêmes types d’environnements d’hébergement.

En attendant, vous pouvez toujours créer le type d’application logique d’origine. Bien que les expériences de développement dans le portail diffèrent entre le type d’application logique d’origine et le nouveau type d’application logique, votre abonnement Azure peut inclure les deux types. Vous pouvez afficher toutes les applications logiques déployées dans votre abonnement Azure et y accéder, mais elles sont organisées dans leurs propres catégories et sections.

Cet article explique comment créer votre application logique et votre workflow dans le portail Azure à l’aide du type de ressource **Application logique (préversion)** et en effectuant les tâches de haut niveau suivantes :

* Créez la nouvelle ressource d’application logique et ajoutez un workflow vide.

* Ajoutez un déclencheur et une action.

* Déclenchez une exécution de workflow.

* Affichez l’historique des exécutions du workflow.

* Activez ou ouvrez Application Insights après le déploiement.

* Activez l’historique des exécutions pour les workflows sans état.

> [!NOTE]
> Pour plus d’informations sur les problèmes connus actuels, consultez la [page relative aux problèmes connus de la préversion publique de Logic Apps dans GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md).

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Un [compte de stockage Azure](../storage/common/storage-account-overview.md), car la ressource **Application logique (préversion)** est alimentée par Azure Functions et a [des exigences de stockage similaires à celles des applications de fonction](../azure-functions/storage-considerations.md). Vous pouvez utiliser un compte de stockage existant ou vous pouvez créer un compte de stockage à l’avance ou pendant la création d’une application logique.

  > [!NOTE]
  > Les [applications logiques avec état](logic-apps-overview-preview.md#stateful-stateless) effectuent des transactions de stockage, telles que l’utilisation de files d’attente pour la planification et le stockage d’états de workflow dans des tables et des blobs. Ces transactions occasionne des [frais pour le service Stockage Azure](https://azure.microsoft.com/pricing/details/storage/). Pour plus d’informations sur la façon dont les applications logiques avec état stockent les données dans un stockage externe, consultez [Avec état par rapport à sans état](logic-apps-overview-preview.md#stateful-stateless).

* Pour générer le même exemple d’application logique que celui présenté dans cet article, vous avez besoin d’un compte de courrier Office 365 Outlook qui utilise un compte Microsoft professionnel ou scolaire pour se connecter.

  Si vous choisissez d’utiliser un autre [connecteur de messagerie pris en charge par Azure Logic Apps](/connectors/), tel qu’Outlook.com ou [Gmail](../connectors/connectors-google-data-security-privacy-policy.md), vous pouvez toujours suivre l’exemple. Les étapes générales sont les mêmes, mais l’interface utilisateur et les options peuvent différer de plusieurs manières. Par exemple, si vous utilisez le connecteur Outlook.com, utilisez votre compte Microsoft personnel au lieu de vous connecter.

* Pour tester l’exemple d’application logique que vous créez dans cet article, vous avez besoin d’un outil capable d’envoyer des appels au déclencheur de demande, ce qui constitue la première étape de l’exemple d’application logique. Si vous n’avez pas d’outil de ce type, vous pouvez télécharger, installer et utiliser [Postman](https://www.postman.com/downloads/).

* Si vous créez votre application logique avec des paramètres qui prennent en charge l’utilisation d’[Application Insights](../azure-monitor/app/app-insights-overview.md), vous pouvez choisir d’activer la journalisation et le suivi des diagnostics pour votre application logique. Vous pouvez le faire lorsque vous créez votre application logique ou après le déploiement. Vous devez disposer d’une instance d’Application Insights, mais vous pouvez créer cette ressource [à l’avance](../azure-monitor/app/create-workspace-resource.md), lorsque vous créez votre application logique, ou après le déploiement.

## <a name="create-the-logic-app-resource"></a>Créer la ressource d’application logique

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche du portail Azure, entrez `logic app preview`, puis sélectionnez **Application logique (préversion)** .

   ![Capture d’écran montrant la zone de recherche du portail Azure contenant le texte de recherche « application logique préversion » et la ressource « Application logique (préversion) » sélectionnée.](./media/create-stateful-stateless-workflows-azure-portal/find-logic-app-resource-template.png)

1. Dans la page **Application logique (préversion)** , sélectionnez **Ajouter**.

1. Sur la page **Créer une application logique (préversion)** , sous l’onglet **Général**, fournissez les informations suivantes relatives à votre application logique.

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Abonnement** | Oui | <*Azure-subscription-name*> | Abonnement Azure à utiliser pour votre application logique. |
   | **Groupe de ressources** | Oui | <*nom-groupe-de-ressources-Azure*> | Groupe de ressources Azure dans lequel vous créez votre application logique et les ressources associées. Ce nom de ressource doit être unique d’une région à l’autre et peut uniquement contenir des lettres, des chiffres, des traits d’union ( **-** ), des traits de soulignement ( **_** ), des parenthèses [ **()** ] et des points ( **.** ). <p><p>Cet exemple crée un groupe de ressources nommé `Fabrikam-Workflows-RG`. |
   | **Nom de l’application logique** | Oui | <*logic-app-name*> | Nom à utiliser pour votre application logique. Ce nom de ressource doit être unique d’une région à l’autre et peut uniquement contenir des lettres, des chiffres, des traits d’union ( **-** ), des traits de soulignement ( **_** ), des parenthèses [ **()** ] et des points ( **.** ). <p><p>Cet exemple crée une application logique nommée `Fabrikam-Workflows`. <p><p>**Remarque** : Le nom de votre application logique reçoit automatiquement le suffixe `.azurewebsites.net`, car la ressource **Application logique (préversion)** est alimentée par Azure Functions, qui utilise la même convention d’affectation de noms d’application. |
   | **Publier** | Oui | <*deployment-environment*> | Destination de déploiement pour votre application logique. Vous pouvez effectuer le déploiement sur Azure en sélectionnant **Workflow** ou un conteneur Docker. <p><p>Cet exemple utilise **Workflow**, qui est la ressource **Application logique (préversion)** dans Azure. <p><p>Si vous sélectionnez **Conteneur Docker**, [spécifiez le conteneur à utiliser dans les paramètres de votre application logique](#set-docker-container). |
   | **Région** | Oui | <*Azure-region*> | Région Azure à utiliser lors de la création de votre groupe de ressources et de vos ressources. <p><p>Cet exemple utilise la région **USA Ouest**. |
   |||||

   Voici un exemple :

   ![Capture d’écran montrant le portail Azure et la page « Créer une application logique (préversion) ».](./media/create-stateful-stateless-workflows-azure-portal/create-logic-app-resource-portal.png)

1. Ensuite, dans l’onglet **Hébergement**, fournissez ces informations sur la solution de stockage et le plan d’hébergement à utiliser pour votre application logique.

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Compte de stockage** | Oui | <*Azure-storage-account-name*> | [Compte de stockage Azure](../storage/common/storage-account-overview.md) à utiliser pour les transactions de stockage. Ce nom de ressource doit être unique d’une région à l’autre et comporter entre 3 et 24 caractères avec uniquement des chiffres et des lettres minuscules. Sélectionnez un compte existant ou créez un nouveau compte. <p><p>Cet exemple crée un compte de stockage nommé `fabrikamstorageacct`. |
   | **Type de plan** | Oui | <*Azure-hosting-plan*> | [Plan d’hébergement](../app-service/overview-hosting-plans.md) à utiliser pour déployer votre application logique, qui est un [**plan Premium**](../azure-functions/functions-scale.md#premium-plan) ou [**App Service**](../azure-functions/functions-scale.md#app-service-plan). Votre choix a une incidence sur les niveaux tarifaires que vous pouvez choisir ultérieurement. <p><p>Cet exemple utilise le **plan App Service**. <p><p>**Remarque** : Comme Azure Functions, le type de ressource **Application logique (préversion)** requiert un plan d’hébergement et un niveau tarifaire. Les plans d’hébergement dits de consommation ne sont pas pris en charge ni disponibles pour ce type de ressource. Pour plus d’informations, consultez les rubriques suivantes : <p><p>- [Échelle et hébergement dans Azure Functions](../azure-functions/functions-scale.md) <br>- [Détails de tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/) <p><p> |
   | **Plan Windows** | Oui | <*plan-name*> | Nom du plan à utiliser. Sélectionnez un plan existant ou fournissez le nom d’un nouveau plan. <p><p>L’exemple suivant utilise le nom `Fabrikam-Service-Plan`. |
   | **SKU et taille** | Oui | <*pricing-tier*> | [Niveau tarifaire](../app-service/overview-hosting-plans.md) à utiliser pour héberger votre application logique. Vos choix sont influencés par le type de plan que vous avez choisi précédemment. Pour modifier le niveau par défaut, sélectionnez **Modifier la taille**. Vous pouvez ensuite sélectionner d’autres niveaux tarifaires en fonction de la charge de travail dont vous avez besoin. <p><p>Cet exemple utilise le **niveau tarifaire F1** gratuit pour des charges de travail **Dev/Test**. Pour en savoir plus, consultez les [détails de tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/). |
   |||||

1. Ensuite, si vos paramètres de création et de déploiement prennent en charge l’utilisation d’[Application Insights](../azure-monitor/app/app-insights-overview.md), vous pouvez activer en option la journalisation et le suivi des diagnostics pour votre application logique.

   1. Dans l’onglet **Surveillance**, sous **Application Insights**, définissez **Activer Application Insights** sur **Oui** si ce n’est pas déjà fait.

   1. Pour le paramètre **Application Insights**, sélectionnez une instance existante d’Application Insights ou, si vous souhaitez créer une nouvelle instance, sélectionnez **Créer nouveau** et indiquez le nom que vous souhaitez utiliser.

1. Une fois qu’Azure a validé les paramètres de votre application logique, sous l’onglet **Vérifier + créer**, sélectionnez **Créer**.

   Par exemple :

   ![Capture d’écran montrant le portail Azure et les nouveaux paramètres de ressource d’application logique.](./media/create-stateful-stateless-workflows-azure-portal/check-logic-app-resource-settings.png)

   Une fois le déploiement terminé, votre application logique est automatiquement active et en cours d’exécution, mais elle ne fait rien pour l’instant, car il n’existe aucun workflow.

1. Sur la page d’achèvement du déploiement, sélectionnez **Accéder à la ressource** afin de pouvoir commencer à créer votre workflow.

   ![Capture d’écran montrant le portail Azure et le déploiement terminé.](./media/create-stateful-stateless-workflows-azure-portal/logic-app-completed-deployment.png)

<a name="set-docker-container"></a>

## <a name="specify-docker-container-for-deployment"></a>Spécifier un conteneur Docker pour le déploiement

Si vous avez sélectionné **Conteneur Docker** lors de la création de votre application logique, assurez-vous de fournir des informations sur le conteneur que vous souhaitez utiliser pour le déploiement après que le portail Azure a créé votre ressource **Application logique (préversion)** .

1. Dans le portail Azure, accédez à votre ressource d’application logique.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Paramètres de conteneur**. Fournissez les détails et l’emplacement de votre image conteneur Docker.

   ![Capture d’écran montrant le menu de l’application logique avec l’élément « Paramètres de conteneur » sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/logic-app-deploy-container-settings.png)

1. Lorsque vous avez terminé, enregistrez vos paramètres.

<a name="add-workflow"></a>

## <a name="add-a-blank-workflow"></a>Ajouter un workflow vide

1. Une fois qu’Azure a ouvert la ressource, dans le menu de votre application logique, sélectionnez **Workflow**. Dans la barre d’outils **Workflows**, sélectionnez **Ajouter**.

   ![Capture d’écran montrant le menu de la ressource d’application logique avec l’option « Workflows » sélectionnée, puis, dans la barre d’outils, « Ajouter » est sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/logic-app-add-blank-workflow.png)

1. Après l’ouverture du volet **Nouveau workflow**, donnez un nom à votre workflow et choisissez le type de workflow [**avec état** ou **sans état**](logic-apps-overview-preview.md#stateful-stateless). Sélectionnez **Créer** lorsque vous avez terminé.

   Cet exemple ajoute un workflow avec état vide nommé `Fabrikam-Stateful-Workflow`. Par défaut, le workflow est activé, mais ne fait rien tant que vous n’avez pas ajouté un déclencheur et des actions.

   ![Capture d’écran montrant le workflow avec état vide récemment ajouté « Fabrikam-Stateful-Workflow ».](./media/create-stateful-stateless-workflows-azure-portal/logic-app-blank-workflow-created.png)

1. Ensuite, ouvrez le workflow vide dans le concepteur afin d’ajouter un déclencheur et une action.

   1. Dans la liste des workflows, sélectionnez le workflow vide.

   1. Dans le menu des workflows, sous **Développeur**, sélectionnez **Concepteur**.

      Sur l’aire du concepteur, l’invite **Choisir une opération** s’affiche déjà et est sélectionnée par défaut de sorte que le volet **Ajouter un déclencheur** apparaît également ouvert.

      ![Capture d’écran montrant le concepteur de workflow ouvert avec l’option « Choisir une opération » sélectionnée sur l’aire du concepteur.](./media/create-stateful-stateless-workflows-azure-portal/opened-logic-app-designer-blank-workflow.png)

<a name="add-trigger-actions"></a>

## <a name="add-a-trigger-and-an-action"></a>Ajouter un déclencheur et une action

Cet exemple crée un workflow qui comporte les étapes suivantes :

* [Déclencheur de demande](../connectors/connectors-native-reqres.md) intégré, **Lors de la réception d’une requête HTTP**, qui reçoit les appels ou requêtes entrants, et crée un point de terminaison que d’autres services ou applications logiques peuvent appeler.

* [Action Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), **Envoyer un e-mail**.

* [Action Réponse](../connectors/connectors-native-reqres.md) intégrée, qui permet d’envoyer une réponse et de retourner des données à l’appelant.

### <a name="add-the-request-trigger"></a>Ajout du déclencheur Request

Avant de pouvoir ajouter un déclencheur à un workflow vide, assurez-vous que le concepteur de workflow est ouvert et que l’invite **Choisir une opération** est sélectionnée sur l’aire du concepteur.

1. À côté de l’aire du concepteur, dans le volet **Ajouter un déclencheur**, sous la zone de recherche **Choisir une opération**, vérifiez que l’onglet **Intégré** est sélectionné. Cet onglet affiche les déclencheurs qui s’exécutent en mode natif dans Azure Logic Apps.

1. Dans la zone de recherche **Choisir une opération**, entrez `when a http request`, puis sélectionnez le déclencheur de demande intégré nommé **Lors de la réception d’une requête HTTP**.

   ![Capture d’écran montrant le concepteur et le volet **Ajouter un déclencheur** avec le déclencheur « Lors de la réception d’une requête HTTP » sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/find-request-trigger.png)

   Lorsque le déclencheur s’affiche sur le concepteur, le volet d’informations du déclencheur s’ouvre pour afficher les propriétés, les paramètres et d’autres actions du déclencheur.

   ![Capture d’écran montrant le concepteur avec le déclencheur « Lors de la réception d’une requête HTTP » sélectionné et le volet d’informations du déclencheur ouvert.](./media/create-stateful-stateless-workflows-azure-portal/request-trigger-added-to-designer.png)

   > [!TIP]
   > Si le volet d’informations ne s’affiche pas, assurez-vous que le déclencheur est sélectionné sur le concepteur.

1. Si vous devez supprimer un élément du concepteur, [procédez comme suit](#delete-from-designer).

1. Pour enregistrer votre travail, sélectionnez **Enregistrer** sur la barre d’outils du concepteur.

   Lorsque vous enregistrez un workflow pour la première fois et que celui-ci démarre avec un déclencheur Requête, le service Logic Apps génère automatiquement une URL pour un point de terminaison créé par le déclencheur Requête. Plus tard, lorsque vous testez votre workflow, vous envoyez une requête à cette URL, qui active le déclencheur et démarre l’exécution du workflow.

### <a name="add-the-office-365-outlook-action"></a>Ajouter l’action Office 365 Outlook

1. Dans le concepteur, sous le déclencheur que vous avez ajouté, sélectionnez **Nouvelle étape**.

   L’invite **Choisir une opération** s’affiche sur le concepteur, et le volet **Ajouter une action** se rouvre pour vous permettre de sélectionner l’action suivante.

   > [!NOTE]
   > Si le volet **Ajouter une action** affiche le message d’erreur « Impossible de lire la propriété "filtre" de non défini », enregistrez votre workflow, rechargez la page, rouvrez votre workflow, puis réessayez.

1. Dans le volet **Ajouter une action**, sous la zone de recherche **Choisir une opération**, sélectionnez **Azure**. Cet onglet affiche les connecteurs gérés qui sont disponibles et déployés dans Azure.

   > [!NOTE]
   > Si le volet **Ajouter une action** affiche le message d’erreur `The access token expiry UTC time '{token-expiration-date-time}' is earlier than current UTC time '{current-date-time}'`, enregistrez votre workflow, rechargez la page, rouvrez votre workflow, puis réessayez d’ajouter l’action.

   Cet exemple utilise l’action Office 365 Outlook intitulée **Envoyer un e-mail (V2)** .

   ![Capture d’écran montrant le concepteur et le volet **Ajouter une action** avec l’action Office 365 Outlook « Envoyer un e-mail » sélectionnée.](./media/create-stateful-stateless-workflows-azure-portal/find-send-email-action.png)

1. Dans le volet d’informations de l’action, dans l’onglet **Créer une connexion**, sélectionnez **Se connecter** pour pouvoir créer une connexion à votre compte de messagerie.

   ![Capture d’écran montrant le concepteur et le volet d’informations « Envoyer un e-mail (V2) » avec l’option « Se connecter » sélectionnée.](./media/create-stateful-stateless-workflows-azure-portal/send-email-action-sign-in.png)

1. Lorsque vous êtes invité à donner votre consentement pour accéder à votre compte de messagerie, connectez-vous avec vos informations d’identification.

   > [!NOTE]
   > Si vous recevez le message d’erreur `Failed with error: 'The browser is closed.'. Please sign in again`, vérifiez si votre navigateur bloque les cookies tiers. Si ces cookies sont bloqués, essayez d’ajouter `https://portal.azure.com` à la liste des sites autorisés à utiliser des cookies. Si vous utilisez le mode Incognito, assurez-vous que les cookies tiers ne sont pas bloqués.
   > 
   > Si nécessaire, rechargez la page, ouvrez votre workflow, rajoutez l’action d’e-mail et essayez de créer la connexion.

   Une fois qu’Azure a créé la connexion, l’action **Envoyer un e-mail** apparaît sur le concepteur et est sélectionnée par défaut. Si l’action n’est pas sélectionnée, sélectionnez l’action afin que son volet d’informations soit également ouvert.

1. Dans le volet d’informations de l’action, sous l’onglet **Paramètres**, fournissez les informations requises pour l’action, par exemple :

   ![Capture d’écran montrant le concepteur et le volet d’informations « Envoyer un e-mail » avec l’onglet « Paramètres » sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/send-email-action-details.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **To** | Oui | <*your-email-address*> | Destinataire de l’e-mail, qui peut être votre adresse de messagerie à des fins de test. Cet exemple utilise l’e-mail fictif `sophiaowen@fabrikam.com`. |
   | **Subject** | Oui | `An email from your example workflow` | L’objet de l’e-mail |
   | **Corps** | Oui | `Hello from your example workflow!` | Contenu du corps de l’e-mail. |
   ||||

   > [!NOTE]
   > Lorsque vous apportez des modifications dans le volet d’informations sous l’onglet **Paramètres**, **Résultat statique** ou **Exécuter après**, veillez à sélectionner **Terminé** pour valider ces modifications avant de changer d’onglet ou de passer au concepteur. Dans le cas contraire, le concepteur ne conservera pas vos modifications.

1. Enregistrez votre travail. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

Ensuite, pour tester votre workflow, déclenchez manuellement une exécution.

## <a name="trigger-the-workflow"></a>Déclencher le workflow

Dans cet exemple, le workflow s’exécute lorsque le déclencheur Requête reçoit une demande entrante, qui est envoyée à l’URL du point de terminaison créé par le déclencheur. Lorsque vous avez enregistré le workflow pour la première fois, le service Logic Apps a automatiquement généré cette URL. Ainsi, avant de pouvoir envoyer cette demande pour déclencher le workflow, vous devez trouver cette URL.

1. Dans le concepteur de workflow, sélectionnez le déclencheur Requête intitulé **Lors de la réception d’une requête HTTP**.

1. Une fois que le volet d’informations s’ouvre, sous l’onglet **Paramètres**, recherchez la propriété **HTTP POST URL**. Pour copier l’URL générée, sélectionnez **Copier l’URL** (icône de copie de fichier) et enregistrez l’URL ailleurs pour le moment. L’URL suit ce format :

   `http://<logic-app-name>.azurewebsites.net:443/api/<workflow-name>/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<shared-access-signature>`

   ![Capture d’écran montrant le concepteur avec le déclencheur Requête et l’URL du point de terminaison dans la propriété « HTTP POST URL ».](./media/create-stateful-stateless-workflows-azure-portal/find-request-trigger-url.png)

   Pour cet exemple, l’URL se présente comme suit :

   `https://fabrikam-workflows.azurewebsites.net:443/api/Fabrikam-Stateful-Workflow/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=xxxxxXXXXxxxxxXXXXxxxXXXXxxxxXXXX`

   > [!TIP]
   > Vous pouvez également trouver l’URL du point de terminaison dans le volet **Vue d’ensemble** de votre application logique dans la propriété **URL du workflow**.
   >
   > 1. Dans le menu des ressources, sélectionnez **Vue d’ensemble**.
   > 1. Dans le volet **Vue d’ensemble**, recherchez la propriété **URL du workflow**.
   > 1. Pour copier l’URL du point de terminaison, déplacez votre pointeur sur la fin du texte de l’URL du point de terminaison et sélectionnez **Copier dans le presse-papiers** (icône de copie de fichier).

1. Pour tester l’URL en envoyant une demande, ouvrez [Postman](https://www.postman.com/downloads/) ou l’outil de votre choix pour créer et envoyer des demandes.

   Cet exemple se poursuit en utilisant Postman. Pour plus d’informations, consultez [Prise en main de Postman](https://learning.postman.com/docs/getting-started/introduction/).

   1. Dans la barre d’outils de Postman, sélectionnez **Nouveau**.

      ![Capture d’écran montrant Postman avec le bouton Nouveau sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/postman-create-request.png)

   1. Dans le volet **Créer**, sous **Blocs de construction**, sélectionnez **Requête**.

   1. Dans la fenêtre **Enregistrer la requête**, sous **Nom de la requête**, entrez un nom pour la requête, par exemple `Test workflow trigger`.

   1. Sous **Sélectionner une collection ou un dossier dans lesquels enregistrer**, sélectionnez **Créer une collection**.

   1. Sous **Toutes les collections**, fournissez un nom pour la collection à créer afin d’organiser vos demandes, appuyez sur Entrée, puis sélectionnez **Enregistrer dans <*nom-collection*>** . Cet exemple utilise `Logic Apps requests` comme nom de collection.

      Le volet de demande de Postman s’ouvre pour que vous puissiez envoyer une demande à l’URL du point de terminaison du déclencheur Requête.

      ![Capture d’écran montrant Postman avec le volet de demande ouvert](./media/create-stateful-stateless-workflows-azure-portal/postman-request-pane.png)

   1. Dans le volet de demande, dans la zone d’adresse à côté de la liste de méthodes qui affiche actuellement la méthode de demande par défaut **GET**, collez l’URL précédemment copiée, puis sélectionnez **Envoyer**.

      ![Capture d’écran montrant Postman et l’URL du point de terminaison dans la zone d’adresse avec le bouton Envoyer sélectionné.](./media/create-stateful-stateless-workflows-azure-portal/postman-test-endpoint-url.png)

      Lorsque le déclencheur est activé, l’exemple de workflow s’exécute et envoie un e-mail qui ressemble à l’exemple suivant :

      ![Capture d’écran montrant l’e-mail Outlook décrit dans l’exemple.](./media/create-stateful-stateless-workflows-azure-portal/workflow-app-result-email.png)

## <a name="review-run-history"></a>Examiner l’historique des exécutions

Pour un workflow avec état, après chaque exécution du workflow, vous pouvez afficher l’historique des exécutions, y compris l’état de l’exécution globale, du déclencheur et de chaque action, ainsi que leurs entrées et les sorties.

1. Dans le portail Azure, dans le menu de votre workflow, sélectionnez **Analyse**.

   Le volet **Analyse** affiche l’historique des exécutions de ce workflow.

   ![Capture d’écran montrant le volet « Analyse » et l’historique des exécutions du workflow.](./media/create-stateful-stateless-workflows-azure-portal/find-run-history.png)

   > [!TIP]
   > Si l’état d’exécution le plus récent ne s’affiche pas, dans la barre d’outils du volet **Analyse**, sélectionnez **Actualiser**. Aucune exécution ne se produit pour un déclencheur ignoré en raison de critères non respectés ou d’une absence de données.

   | État de l’exécution | Description |
   |------------|-------------|
   | **Abandonné** | L’exécution s’est arrêtée ou ne s’est pas terminée en raison de problèmes externes, par exemple, une panne système ou un abonnement Azure expiré. |
   | **Annulé** | L’exécution a été déclenchée et a démarré, mais a reçu une demande d’annulation. |
   | **Échec** | Au moins une action de l’exécution a échoué. Aucune action subséquente n’était configurée dans le flux de travail pour gérer l’échec. |
   | **Exécution** | L’exécution a été déclenchée et est en cours, mais cet état peut également apparaître pour une exécution limitée en raison de [limites d’action](logic-apps-limits-and-config.md) ou du [plan de tarification actuel](https://azure.microsoft.com/pricing/details/logic-apps/). <p><p>**Conseil** : Si vous configurez la [journalisation des diagnostics](monitor-logic-apps-log-analytics.md), vous pouvez obtenir des informations sur tous les événements de limitation qui se produisent. |
   | **Réussi** | L’exécution a réussi. Si une action a échoué, une action subséquente dans le flux de travail a géré cet échec. |
   | **Délai dépassé** | L’exécution a expiré parce qu’elle a dépassé la limite de durée d’exécution qui est contrôlée par le [paramètre **Run history retention in days** (rétention de l’historique d’exécution en jours)](logic-apps-limits-and-config.md#run-duration-retention-limits). La durée d’une exécution est calculée en utilisant l’heure de début et la limite de durée d’exécution à cette heure. <p><p>**Remarque** : Si la durée de l’exécution dépasse aussi la *limite actuelle de rétention de l’historique d’exécution*, qui est également contrôlée par le paramètre [**Run history retention in days** (Rétention de l’historique d’exécution en jours)](logic-apps-limits-and-config.md#run-duration-retention-limits), l’exécution est effacée de l’historique d’exécution par un travail de nettoyage quotidien. Que l’exécution expire ou se termine, la période de rétention est toujours calculée sur la base de l’heure de début de l’exécution et de la limite de rétention *actuelle*. Par conséquent, si vous réduisez la limite de durée pour une exécution en cours, l’exécution expire. Toutefois, l’exécution reste dans l’historique des exécutions ou est effacée de celui-ci selon que la durée de l’exécution a ou non dépassé la limite de rétention. |
   | **En attente** | L’exécution n’a pas démarré ou est suspendue, par exemple, en raison d’une instance de flux de travail antérieure qui est toujours en cours d’exécution. |
   |||

1. Pour passer en revue l’état de chaque étape d’une exécution, sélectionnez l’exécution que vous souhaitez examiner.

   L’affichage détaillé de l’exécution s’ouvre et indique l’état de chaque étape de l’exécution.

   ![Capture d’écran montrant l’affichage détaillé de l’exécution avec l’état de chaque étape du workflow.](./media/create-stateful-stateless-workflows-azure-portal/review-run-details.png)

   Voici les états possibles que chaque étape du flux de travail :

   | État d’action | Icône | Description |
   |---------------|------|-------------|
   | Abandonné | ![Icône d’état d’action « Abandonnée »][aborted-icon] | L’action a été arrêtée ou ne s’est pas terminée en raison de problèmes externes, par exemple, une panne du système ou l’expiration d’un abonnement Azure. |
   | Annulé | ![Icône d’état d’action « Annulée »][cancelled-icon] | L’action était en cours d’exécution mais a reçu une demande d’annulation. |
   | Échec | ![Icône d’état d’action « Échec »][failed-icon] | L’action a échoué. |
   | Exécution en cours | ![Icône d’état d’action « Exécution »][running-icon] | L’action est en cours d’exécution. |
   | Ignoré | ![Icône d’état d’action « Ignorée »][skipped-icon] | L’action a été ignorée parce qu’elle précède immédiatement l’action qui a échoué. Une action a une condition `runAfter` qui exige que l’action précédente se termine correctement avant que l’action en cours puisse s’exécuter. |
   | Opération réussie | ![Icône d’état d’action « Réussie »][succeeded-icon] | L’action a réussi. |
   | Réussie avec de nouvelles tentatives | ![Icône d’état d’action « Réussie avec de nouvelles tentatives »][succeeded-with-retries-icon] | L’action a réussi, mais uniquement après une ou plusieurs tentatives. Pour consulter l’historique des tentatives, dans la vue des détails de l’historique des exécutions, sélectionnez cette action afin de voir les entrées et sorties. |
   | Délai dépassé | ![Icône d’état de l’action « Expirée »][timed-out-icon] | L’action a été arrêtée en raison du délai d’expiration spécifié par les paramètres de cette action. |
   | En attente | ![Icône d’état d’action « En attente »][waiting-icon] | S’applique à une action de webhook qui attend une demande entrante d’un appelant. |
   ||||

   [aborted-icon]: ./media/create-stateful-stateless-workflows-azure-portal/aborted.png
   [cancelled-icon]: ./media/create-stateful-stateless-workflows-azure-portal/cancelled.png
   [failed-icon]: ./media/create-stateful-stateless-workflows-azure-portal/failed.png
   [running-icon]: ./media/create-stateful-stateless-workflows-azure-portal/running.png
   [skipped-icon]: ./media/create-stateful-stateless-workflows-azure-portal/skipped.png
   [succeeded-icon]: ./media/create-stateful-stateless-workflows-azure-portal/succeeded.png
   [succeeded-with-retries-icon]: ./media/create-stateful-stateless-workflows-azure-portal/succeeded-with-retries.png
   [timed-out-icon]: ./media/create-stateful-stateless-workflows-azure-portal/timed-out.png
   [waiting-icon]: ./media/create-stateful-stateless-workflows-azure-portal/waiting.png

1. Pour examiner les entrées et les sorties d’une étape spécifique, sélectionnez cette dernière.

   ![Capture d’écran montrant les entrées et sorties de l’action « Envoyer un e-mail » sélectionnée.](./media/create-stateful-stateless-workflows-azure-portal/review-step-inputs-outputs.png)

1. Pour examiner plus en détail les entrées et sorties brutes de cette étape, sélectionnez **Afficher les entrées brutes** ou **Afficher les sorties brutes**.

<a name="enable-open-application-insights"></a>

## <a name="enable-or-open-application-insights-after-deployment"></a>Activer ou ouvrir Application Insights après le déploiement

Pendant l’exécution du workflow, votre application logique émet des données de télémétrie ainsi que d’autres événements. Vous pouvez utiliser ces données de télémétrie pour obtenir une meilleure visibilité sur l’exécution de votre workflow et sur les différentes façons dont fonctionne le runtime de Logic Apps. Vous pouvez surveiller votre workflow à l’aide d’[Application Insights](../azure-monitor/app/app-insights-overview.md), qui fournit des données de télémétrie en quasi-temps réel (métriques en temps réel). Cette capacité vous permet d’examiner plus facilement les défaillances et les problèmes de performances lorsque vous utilisez ces données pour diagnostiquer des problèmes, configurer des alertes et créer des graphiques.

Si les paramètres de création et de déploiement de votre application logique prennent en charge l’utilisation d’[Application Insights](../azure-monitor/app/app-insights-overview.md), vous pouvez activer en option la journalisation et le suivi des diagnostics pour votre application logique. Vous pouvez le faire lorsque vous créez votre application logique dans le portail Azure ou après le déploiement. Vous devez disposer d’une instance d’Application Insights, mais vous pouvez créer cette ressource [à l’avance](../azure-monitor/app/create-workspace-resource.md), lorsque vous créez votre application logique, ou après le déploiement.

Pour activer Application Insights sur une application logique déployée ou pour ouvrir le tableau de bord Application Insights si la fonctionnalité est déjà activée, procédez comme suit :

1. Dans le portail Azure, recherchez votre application logique déployée.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Application Insights**.

1. Si la fonctionnalité Application Insights n’est pas activée, dans le volet **Application Insights**, sélectionnez **Activer Application Insights**. Une fois le volet mis à jour, dans la partie inférieure, sélectionnez **Appliquer**.

   Si la fonctionnalité Application Insights est activée, dans le volet **Application Insights**, sélectionnez **Afficher les données Application Insights**.

Après l’ouverture d’Application Insights, vous pouvez passer en revue les différentes métriques de votre application logique.

<a name="enable-run-history-stateless"></a>

## <a name="enable-run-history-for-stateless-workflows"></a>Activer l’historique des exécutions pour les workflows sans état

Pour déboguer plus facilement un workflow sans état, vous pouvez activer l’historique des exécutions du workflow en question, puis le désactiver lorsque vous avez terminé. Procédez comme suit pour le portail Azure ou, si vous travaillez dans Visual Studio Code, consultez [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#enable-run-history-stateless).

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre ressource **Logic Apps (préversion)** .

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Configuration**.

1. Sous l’onglet **Paramètres de l’application**, sélectionnez **Nouveau paramètre d’application**.

1. Dans le volet **Ajouter/modifier un paramètre d’application**, dans la zone **Nom**, entrez le nom de l’option d’opération : 

   `Workflows.{yourWorkflowName}.OperationOptions`

1. Dans la zone **Valeur**, entrez la valeur suivante : `WithStatelessRunHistory`

   Par exemple :

   ![Capture d’écran montrant le portail Azure et la ressource Application logique (préversion) avec le volet « Configuration » > « Nouveau paramètre d’application » > « Ajouter/modifier un paramètre d’application » ouvert et l’option « Workflows.{yourWorkflowName}.OperationOptions » définie sur « WithStatelessRunHistory ».](./media/create-stateful-stateless-workflows-azure-portal/stateless-operation-options-run-history.png)

1. Pour terminer cette tâche, sélectionnez **OK**. Dans la barre d’outils du volet **Configuration**, sélectionnez **Enregistrer**.

1. Pour désactiver l’historique des exécutions lorsque vous avez terminé, supprimez la propriété `Workflows.{yourWorkflowName}.OperationOptions` et sa valeur, ou définissez-la sur `None`.

<a name="delete-from-designer"></a>

## <a name="delete-items-from-the-designer"></a>Supprimer des éléments du concepteur

Pour supprimer un élément de votre workflow à partir du concepteur, procédez de l’une des façons suivantes :

* Sélectionnez l’élément, ouvrez le menu contextuel de l’élément (Maj+F10), puis sélectionnez **Supprimer**. Pour confirmer, sélectionnez **OK**.

* Sélectionnez l’élément, puis appuyez sur la touche Suppr. Pour confirmer, sélectionnez **OK**.

* Sélectionnez l’élément afin que le volet d’informations le concernant s’ouvre. Dans l’angle supérieur droit du volet, ouvrez le menu de sélection ( **…** ), puis sélectionnez **Supprimer**. Pour confirmer, sélectionnez **OK**.

  ![Capture d’écran montrant un élément sélectionné dans le concepteur avec le volet d’informations ouvert, ainsi que le bouton de sélection et la commande « Supprimer » sélectionnés.](./media/create-stateful-stateless-workflows-azure-portal/delete-item-from-designer.png)

  > [!TIP]
  > Si le menu de sélection n’est pas visible, développez la fenêtre de votre navigateur suffisamment pour que le volet d’informations affiche le bouton de sélection ( **…** ) dans le coin supérieur droit.

<a name="troubleshoot"></a>

## <a name="troubleshoot-problems-and-errors"></a>Résoudre les problèmes et les erreurs

<a name="missing-triggers-actions"></a>

### <a name="new-triggers-and-actions-are-missing-from-the-designer-picker-for-previously-created-workflows"></a>De nouveaux déclencheurs et actions sont absents du sélecteur de concepteur pour les workflows créés précédemment

Azure Logic Apps (préversion) prend en charge les actions intégrées pour Azure Function Operations, Liquid Operations et XML Operations, telles que **Validation XML** et **Transformer au format XML**. Toutefois, pour les applications logiques créées précédemment, ces actions peuvent ne pas apparaître dans le concepteur en vue d’une sélection si votre application logique utilise une version obsolète du pack d’extension, `Microsoft.Azure.Functions.ExtensionBundle.Workflows`.

Pour corriger ce problème, procédez comme suit pour supprimer la version obsolète afin que le pack d’extension puisse être automatiquement mis à jour vers la version la plus récente.

> [!NOTE]
> Cette solution spécifique s’applique uniquement aux ressources **Application logique (préversion)** que vous créez à l’aide du portail Azure, et non aux applications logiques que vous créez et déployez à l’aide de Visual Studio Code et de l’extension Azure Logic Apps (préversion). Voir [Des déclencheurs et actions pris en charge sont absents du concepteur dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#missing-triggers-actions).

1. Dans le portail Azure, arrêtez votre application logique.

   1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**.

   1. Dans la barre d’outils du volet **Vue d’ensemble**, sélectionnez **Arrêter**.

1. Dans le menu de l’application logique, sélectionnez **Outils avancés** sous **Outils de développement**.

1. Dans le volet **Outils avancés**, sélectionnez **Accéder**, ce qui ouvre l’environnement Kudu pour votre application logique.

1. Dans la barre d’outils Kudu, ouvrez le menu **Console de débogage**, puis sélectionnez **CMD**. 

   Une fenêtre de console s’ouvre pour vous permettre d’accéder au dossier du pack à l’aide de l’invite de commandes. Sinon, vous pouvez parcourir la structure des répertoires qui s’affiche dans la fenêtre de console.

1. Accédez au dossier suivant, qui contient les dossiers de version du pack existant :

   `...\home\data\Functions\ExtensionBundles\Microsoft.Azure.Functions.ExtensionBundle.Workflows`

1. Supprimez le dossier de version pour le pack existant. Dans la fenêtre de la console, vous pouvez exécuter cette commande dans laquelle vous remplacez `{bundle-version}` par la version existante :

   `rm -rf {bundle-version}`

   Par exemple : `rm -rf 1.1.3`

   > [!TIP]
   > Si vous recevez une erreur de type « autorisation refusée » ou « fichier en cours d’utilisation », actualisez la page de votre navigateur, puis réessayez d’exécuter les étapes précédentes jusqu’à ce que le dossier soit supprimé.

1. Dans le portail Azure, revenez à la page **Vue d’ensemble** de votre application logique, puis sélectionnez **Redémarrer**.

   Le portail récupère et utilise automatiquement le pack le plus récent.

## <a name="next-steps"></a>Étapes suivantes

Nous aimerions que vous nous fassiez part de votre expérience de la préversion publique.

* Pour tout bogue ou problème que vous rencontreriez, [créez un problème dans GitHub](https://github.com/Azure/logicapps/issues).
* Pour toute question, demande, commentaire ou retour d’expérience, utilisez de [formulaire de commentaires](https://aka.ms/lafeedback).
