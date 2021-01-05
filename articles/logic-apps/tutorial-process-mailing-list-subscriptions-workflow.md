---
title: Générer des workflows automatisés basés sur l’approbation
description: 'Tutoriel : Créer un workflow automatisé basé sur l’approbation qui traite les abonnements aux listes de diffusion à l’aide d’Azure Logic Apps'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/07/2020
ms.openlocfilehash: 102b1946021aff7f8ab5491ed70fbc6cf772e3a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842258"
---
# <a name="tutorial-create-automated-approval-based-workflows-by-using-azure-logic-apps"></a>Tutoriel : Créer des workflows automatisés basés sur l’approbation à l’aide d’Azure Logic Apps

Ce tutoriel montre comment générer un exemple d’[application logique](../logic-apps/logic-apps-overview.md) qui automatise un workflow basé sur l’approbation. Plus précisément, cet exemple d’application logique traite les demandes d’abonnement à une liste de diffusion gérée par le service [MailChimp](https://mailchimp.com/). Cette application logique comprend différentes étapes, la première d’entre elles étant la supervision d’un compte de messagerie pour les demandes. Elle envoie ensuite ces demandes pour approbation, vérifie si la demande est approuvée ou non, ajoute des membres approuvés à la liste de diffusion, puis vérifie si de nouveaux membres sont ajoutés à la liste.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créez une application logique vide.
> * Ajouter un déclencheur qui surveille les e-mails et détecte les requêtes d’abonnement.
> * Ajouter une action qui envoie des e-mails pour approuver ou rejeter ces requêtes.
> * Ajouter une condition qui vérifie la réponse d’approbation.
> * Ajouter une action qui ajoute les membres approuvés à la liste de diffusion.
> * Ajouter une condition qui vérifie si ces membres ont rejoint la liste.
> * Ajouter une action qui envoie des e-mails confirmant que ces membres ont rejoint la liste.

Lorsque vous avez terminé, votre application logique ressemble au flux de travail suivant à un niveau élevé :

![Vue d’ensemble globale de l’application logique terminée](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-overview.png)

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Un compte MailChimp dans lequel vous avez précédemment créé une liste nommée « test-members-ML » dans laquelle votre application logique peut ajouter les adresses e-mail des membres approuvés. Si vous n’avez pas de compte, [créez un compte gratuit](https://login.mailchimp.com/signup/) et découvrez [comment créer une liste MailChimp](https://us17.admin.mailchimp.com/lists/#).

* Un compte de messagerie d’un fournisseur de messagerie pris en charge par Logic Apps, par exemple Office 365 Outlook, Outlook.com ou Gmail. Pour les autres fournisseurs, [passez en revue la liste des connecteurs ici](/connectors/). Ce guide de démarrage rapide utilise Office 365 Outlook avec un compte professionnel ou scolaire. Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais l’interface utilisateur peut être légèrement différente.

* Un compte de courrier Office 365 Outlook ou Outlook.com, qui prend en charge les workflows d’approbation. Ce didacticiel utilise Outlook Office 365. Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais l’affichage de l’interface utilisateur peut être légèrement différent.

## <a name="create-your-logic-app"></a>Créer votre application logique

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure. Dans la page d’accueil Azure, sélectionnez **Créer une ressource**.

1. Dans le menu Place de marché Azure, sélectionnez **Intégration** > **Application logique**.

   ![Capture d’écran représentant le menu Place de marché Azure dans lequel « Intégration » et « Application logique » sont sélectionnées.](./media/tutorial-process-mailing-list-subscriptions-workflow/create-new-logic-app-resource.png)

1. Dans le volet **Application logique**, fournissez les informations décrites ici sur l’application logique que vous souhaitez créer.

   ![Capture d’écran représentant le volet de création d’application logique et les informations à fournir pour la nouvelle application logique.](./media/tutorial-process-mailing-list-subscriptions-workflow/create-logic-app-settings.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Abonnement** | <*Azure-subscription-name*> | Le nom de votre abonnement Azure. Cet exemple utilise `Pay-As-You-Go`. |
   | **Groupe de ressources** | LA-MailingList-RG | Nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md), qui est utilisé pour organiser les ressources connexes. Cet exemple crée un groupe de ressources nommé `LA-MailingList-RG`. |
   | **Nom** | LA-MailingList | Nom de votre application logique, qui peut contenir uniquement des lettres, des chiffres, des traits d’union (`-`), des traits de soulignement (`_`), des parenthèses (`(`, `)`) et des points (`.`). Cet exemple utilise `LA-MailingList`. |
   | **Lieu** | USA Ouest | Région dans laquelle stocker les informations sur votre application logique. Cet exemple utilise `West US`. |
   | **Log Analytics** | Off | Maintenez le paramètre de journalisation des diagnostics **Désactivé**. |
   ||||

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Une fois que Azure a validé les informations relatives à votre application logique, sélectionnez **Créer**.

1. Une fois que Azure a déployé votre application, sélectionnez **Accéder à la ressource**.

   Azure ouvre le volet de sélection du modèle Logic Apps, qui affiche une vidéo de présentation, des déclencheurs couramment utilisés et des modèles de modèle d’application logique.

1. Faites défiler la page vers le bas au-delà de la vidéo et des déclencheurs courants jusqu’à la section **Modèles**, puis sélectionnez **Application logique vide**.

   ![Capture d’écran représentant le volet de sélection de modèle Logic Apps dont l’option « Application logique vide » est sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/select-logic-app-template.png)

Ajoutez ensuite un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) Outlook qui écoute les e-mails entrants avec des requêtes d’abonnement. Chaque application logique doit commencer par un déclencheur, qui est activé quand un événement spécifique se produit ou quand de nouvelles données respectent une condition spécifique. Pour plus d’informations, consultez [Quickstart: Build your first logic app workflow - Azure portal](../logic-apps/quickstart-create-first-logic-app-workflow.md) (Démarrage rapide : Générer votre premier flux de travail d’application logique - portail Azure).

## <a name="add-trigger-to-monitor-emails"></a>Ajouter le déclencheur pour surveiller les e-mails

1. Dans la zone de recherche du Concepteur Logic Apps, entrez `when email arrives`, puis sélectionnez le déclencheur nommé **À l’arrivée d’un nouvel e-mail**.

   * Pour les comptes Azure professionnels ou scolaires, sélectionnez **Office 365 Outlook**.
   * Pour les comptes Microsoft personnels, sélectionnez **Outlook.com**.

   Cet exemple se poursuit avec la sélection d’Office 365 Outlook.

   ![Capture d’écran montrant la zone de recherche du Concepteur Logic Apps qui contient le terme de recherche « quand un e-mail arrive » et avec le déclencheur « À l’arrivée d’un nouvel e-mail » sélectionné.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-new-email.png)

1. Si vous n’avez pas encore de connexion, connectez-vous et authentifiez l’accès à votre compte de courrier lorsque vous y êtes invité.

   Azure Logic Apps crée une connexion à votre compte de messagerie.

1. Dans le déclencheur, fournissez les critères permettant de vérifier l’arrivée de nouveaux e-mails.

   1. Spécifiez le dossier pour la vérification des e-mails et conservez les valeurs par défaut des autres propriétés.

      ![Capture d’écran montrant le concepteur avec l’action « À l’arrivée d’un nouvel e-mail » et « Dossier » défini sur « Boîte de réception ».](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-set-up-email.png)

   1. Ajoutez la propriété **Filtre d’objet** du déclencheur afin de pouvoir filtrer les e-mails en fonction de la ligne d’objet. Ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez **Filtre d’objet**.

      ![Capture d’écran montrant la liste « Ajouter un nouveau paramètre » ouverte avec la propriété « Filtre d’objet » sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-add-properties.png)

      Pour plus d’informations sur les propriétés du déclencheur, consultez les [informations de référence sur le connecteur Office 365 Outlook](/connectors/office365/) ou les [informations de référence sur le connecteur Outlook.com](/connectors/outlook/).

   1. Une fois que la propriété s’affiche dans le déclencheur, entrez le texte suivant : `subscribe-test-members-ML`

      ![Capture d’écran montrant la propriété « Filtre d’objet » avec le texte « subscribe-test-members-ML » entré.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-subject-filter-property.png)

1. Pour masquer les informations du déclencheur pour le moment, réduisez la forme en cliquant dans sa barre de titre.

   ![Capture d’écran représentant la forme de déclencheur réduite.](./media/tutorial-process-mailing-list-subscriptions-workflow/collapse-trigger-shape.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

Votre application logique est à présent en ligne, mais elle ne fait rien d’autre que de vérifier vos e-mails entrants. Par conséquent, ajoutez une action qui répond à l’activation du déclencheur.

## <a name="send-approval-email"></a>Envoyer un message électronique d’approbation

Maintenant que vous disposez d’un déclencheur, ajoutez une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) qui envoie un e-mail pour approuver ou rejeter la requête.

1. Dans le Concepteur Logic Apps, sous le déclencheur **À l’arrivée d’un nouvel e-mail**, sélectionnez **Nouvelle étape**.

1. Sous **Choisir une opération**, dans la zone de recherche, entrez `send approval`, puis sélectionnez l’action nommée **Envoyer un e-mail d’approbation**.

   ![Capture d’écran montrant la liste « Choisir une opération » filtrée selon les actions d’« approbation » et l’action « Envoyer un e-mail d’approbation » sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-send-approval-email.png)

1. À présent, entrez les valeurs des propriétés spécifiées, comme indiqué et décrit ici. Conservez les valeurs par défaut de toutes les autres propriétés. Pour plus d’informations sur ces propriétés, consultez les [informations de référence sur le connecteur Office 365 Outlook](/connectors/office365/) ou les [informations de référence sur le connecteur Outlook.com](/connectors/outlook/).

   ![Capture d’écran montrant les propriétés « Envoyer un e-mail d’approbation »](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-approval-email-settings.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **To** | <*adresse_e-mail_approbation*> | Adresse e-mail de l’approbateur. À des fins de test, vous pouvez utiliser votre propre adresse. Cet exemple utilise l’adresse e-mail `sophiaowen@fabrikam.com` fictive. |
   | **Subject** | `Approve member request for test-members-ML` | Un objet d’e-mail descriptif |
   | **Options utilisateur** | `Approve, Reject` | Vérifiez que cette propriété spécifie les options de réponse que l’approbateur peut sélectionner, lesquelles sont **Approuver** ou **Rejeter** par défaut. |
   ||||

   > [!NOTE]
   > Quand vous cliquez dans certaines zones d’édition, la liste à contenu dynamique, que vous pouvez ignorer pour le moment, s’affiche. La liste affiche les sorties des actions précédentes que vous pouvez sélectionner comme entrées pour les actions suivantes de votre workflow.
 
1. Enregistrez votre application logique.

Ensuite, ajoutez une condition qui vérifie la réponse sélectionnée de l’approbateur.

## <a name="check-approval-response"></a>Vérifier la réponse d’approbation

1. Sous l’action **Envoyer un e-mail d’approbation**, sélectionnez **Nouvelle étape**.

1. Sous **Choisir une opération**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `condition`, puis sélectionnez l’action nommée **Condition**.

   ![Capture d’écran qui montre la zone de recherche « Choisir une opération » avec « Intégré » sélectionné et « condition » comme terme de recherche, tandis que l’action « Condition » apparaît sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/select-condition-action.png)

1. Dans la barre de titre **Condition**, sélectionnez le bouton de **sélection** ( **...** ), puis **Renommer**. Renommez la condition à l’aide de cette description : `If request approved`.

   ![Capture d’écran montrant le bouton de sélection sélectionné avec la liste « Paramètres » ouverte et la commande « Renommer » sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/rename-condition-description.png)

1. Créez une condition qui vérifie si l’approbateur a sélectionné **Approuver**.

   1. Dans la partie gauche de la condition, cliquez sur la zone **Choisir une valeur**.

   1. Dans la liste de contenu dynamique qui s’affiche, sous **Envoyer un e-mail d’approbation**, sélectionnez la propriété **SelectedOption**.

      ![Capture d’écran montrant la liste de contenu dynamique où, dans la section « Envoyer un e-mail d’approbation », la sortie « SelectedOption » apparaît sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response.png)

   1. Dans la zone de comparaison du milieu, sélectionnez l’opérateur **est égal à**.

   1. Dans la partie droite de la condition, dans la zone **Choisir une valeur**, entrez le texte `Approve`.

      Quand vous avez terminé, votre condition ressemble à cet exemple :

      ![Capture d’écran montrant la condition terminée de l’exemple de demande approuvée](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response-2.png)

1. Enregistrez votre application logique.

Ensuite, spécifiez l’action que votre application logique effectue lorsque le réviseur approuve la requête. 

## <a name="add-member-to-mailchimp-list"></a>Ajouter un membre à la liste MailChimp

Ajoutez maintenant une action qui ajoute le membre approuvé à votre liste de diffusion.

1. Dans la branche **True** de la condition, sélectionnez **Ajouter une action**.

1. Sous la zone de recherche **Choisir une opération**, sélectionnez **Toutes**. Dans la zone de recherche, entrez `mailchimp`, puis sélectionnez l’action nommée **Ajouter un membre à la liste**.

   ![Capture d’écran montrant la zone « Choisir une opération » avec le terme de recherche « mailchimp » et l’action « Ajouter un membre à la liste » sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member.png)

1. Si vous ne disposez pas déjà d’une connexion à votre compte MailChimp, vous êtes invité à vous connecter.

1. Dans l’action **Ajouter un membre à la liste**, fournissez les informations comme indiqué et décrit ici :

   ![Capture d’écran montrant les informations de l’action « Ajouter un membre à la liste ».](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member-settings.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **ID de liste** | Oui | <*nom_liste_diffusion*> | Sélectionnez le nom de votre liste de diffusion MailChimp. Cet exemple utilise `test-members-ML`. |
   | **Adresse e-mail** | Oui | <*new-member-email-address*> | Dans la liste de contenu dynamique qui s’ouvre, dans la section **À l’arrivée d’un nouvel e-mail**, sélectionnez **De**, qui est la sortie du déclencheur et spécifie l’adresse e-mail du nouveau membre. |
   | **État** | Oui | <*état_abonnement_membre*> | Sélectionnez l’état de l’abonnement à définir pour le nouveau membre. Cet exemple sélectionne `subscribed`. <p>Pour plus d’informations, consultez [Manage Subscribers with the MailChimp API](https://developer.mailchimp.com/documentation/mailchimp/guides/manage-subscribers-with-the-mailchimp-api/) (Gérer les abonnés avec l’API MailChimp). |
   |||||

   Pour plus d’informations sur les propriétés de l’action **Ajouter un membre à la liste**, consultez les [informations de référence sur le connecteur MailChimp](/connectors/mailchimp/).

1. Enregistrez votre application logique.

Ensuite, ajoutez une condition afin que vous puissiez vérifier si le nouveau membre a rejoint votre liste de diffusion. De cette façon, votre application logique peut vous signaler si cette opération a réussi ou a échoué.

## <a name="check-for-success-or-failure"></a>Vérifier la réussite ou l’échec

1. Dans la branche **True**, sous l’action **Ajouter un membre à la liste**, sélectionnez **Ajouter une action**.

1. Sous **Choisir une opération**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `condition`, puis sélectionnez l’action nommée **Condition**.

1. Renommez la condition à l’aide de cette description : `If add member succeeded`.

1. Générez une condition qui vérifie si le membre approuvé parvient ou non à rejoindre votre liste de diffusion :

   1. Dans la partie gauche de la condition, cliquez sur la zone **Choisir une valeur**. Dans la liste de contenu dynamique qui s’affiche, dans la section **Ajouter un membre à la liste**, sélectionnez la propriété **Statut**.

      Votre condition ressemble maintenant à cet exemple :

      ![Capture d’écran montrant la zone « Choisir une valeur » dans la partie gauche de la condition avec le « Statut » entré.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member.png)

   1. Dans la zone de comparaison du milieu, sélectionnez l’opérateur **est égal à**.

   1. Dans la partie droite de la condition, dans la zone **Choisir une valeur**, entrez ce texte : `subscribed`

      Quand vous avez terminé, votre condition ressemble à cet exemple :

      ![Capture d’écran montrant la condition terminée permettant de vérifier la réussite ou l’échec d’un abonnement.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member-2.png)

Ensuite, configurez les e-mails à envoyer quand le membre approuvé parvient, ou non, à rejoindre votre liste de diffusion.

## <a name="send-email-if-member-added"></a>Envoyer un e-mail si le membre est ajouté

1. Sous la condition **Si l’ajout du membre a réussi**, dans la branche **True**, sélectionnez **Ajouter une action**.

   ![Capture d’écran montrant la branche « True » de la condition « Si l’ajout du membre a réussi » avec « Ajouter une action » sélectionné.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success.png)

1. Dans la zone de recherche **Choisir une opération**, entrez `outlook send email`, puis sélectionnez l’action nommée **Envoyer un e-mail**.

   ![Capture d’écran montrant la zone de recherche « Choisir une opération » avec « outlook send email » (envoi d’e-mail outlook) entré et l’action « Envoyer un e-mail » sélectionnée pour la notification.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-2.png)

1. Renommez l’action à l’aide de cette description : `Send email on success`.

1. Dans l’action **Envoyer un e-mail en cas de succès**, fournissez les informations comme indiqué et décrit ici :

   ![Capture d’écran montrant l’action « Envoyer un e-mail en cas de succès » et les informations fournies pour l’e-mail de réussite.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-settings.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Corps** | Oui | <*corps_e-mail_réussite*> | Contenu du corps de l’e-mail de réussite. Pour ce tutoriel, effectuez les étapes suivantes : <p>1. Entrez ce texte avec un espace de fin : `New member has joined "test-members-ML":` <p>2. Dans la liste de contenu dynamique qui s’affiche, sélectionnez la propriété **Adresse e-mail**. <p>**Remarque** : Si cette propriété ne s’affiche pas, en regard de l’en-tête de la section **Ajouter un membre à la liste**, sélectionnez **Afficher plus**. <p>3. Sur la ligne suivante, entrez ce texte avec un espace de fin : `Member opt-in status: ` <p>4. Dans la liste de contenu dynamique, sous **Ajouter un membre à la liste**, sélectionnez la propriété **État**. |
   | **Subject** | Oui | <*objet_e-mail_réussite*> | Objet de l’e-mail de réussite. Pour ce tutoriel, effectuez les étapes suivantes : <p>1. Entrez ce texte avec un espace de fin : `Success! Member added to "test-members-ML": ` <p>2. Dans la liste de contenu dynamique, sous **Ajouter un membre à la liste**, sélectionnez la propriété **Adresse e-mail**. |
   | **To** | Oui | <*your-email-address*> | Adresse e-mail à laquelle envoyer l’e-mail de réussite. À des fins de test, vous pouvez utiliser votre propre adresse e-mail. |
   |||||

1. Enregistrez votre application logique.

## <a name="send-email-if-member-not-added"></a>Envoyer un e-mail si le membre n’est pas ajouté

1. Sous la condition **Si l’ajout du membre a réussi**, dans la branche **False**, sélectionnez **Ajouter une action**.

   ![Capture d’écran montrant la branche « False » de la condition « Si l’ajout du membre a réussi » avec « Ajouter une action » sélectionné.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed.png)

1. Dans la zone de recherche **Choisir une opération**, entrez `outlook send email`, puis sélectionnez l’action nommée **Envoyer un e-mail**.

   ![Capture d’écran montrant la zone de recherche « Choisir une opération » avec « outlook send email » (envoi d’e-mail outlook) entré et l’action « Envoyer un e-mail » sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-2.png)

1. Renommez l’action à l’aide de cette description : `Send email on failure`.

1. Fournissez des informations à propos de cette action comme illustré et décrit ici :

   ![Capture d’écran montrant l’action « Envoyer un e-mail en cas d’échec » et les informations fournies pour l’e-mail d’échec.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-settings.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Corps** | Oui | <*body-for-failure-email*> | Contenu du corps de l’e-mail d’échec. Pour ce didacticiel, entrez le texte suivant : <p>`Member might already exist. Check your MailChimp account.` |
   | **Subject** | Oui | <*subject-for-failure-email*> | Objet de l’e-mail d’échec. Pour ce tutoriel, effectuez les étapes suivantes : <p>1. Entrez ce texte avec un espace de fin : `Failed, member not added to "test-members-ML": ` <p>2. Dans la liste de contenu dynamique, sous **Ajouter un membre à la liste**, sélectionnez la propriété **Adresse e-mail**. |
   | **To** | Oui | <*your-email-address*> | Adresse e-mail à laquelle envoyer l’e-mail d’échec. À des fins de test, vous pouvez utiliser votre propre adresse e-mail. |
   |||||

1. Enregistrez votre application logique. 

Ensuite, testez votre application logique, qui ressemble désormais à l’exemple suivant :

![Capture d’écran montrant l’exemple de workflow d’application logique terminé.](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-completed.png)

## <a name="run-your-logic-app"></a>Exécuter votre application logique

1. Envoyez-vous un e-mail de requête pour rejoindre votre liste de diffusion. Attendez que la requête s’affiche dans votre boîte de réception.

1. Pour lancer manuellement votre application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur. 

   Si l’objet de votre e-mail correspond au filtre d’objet du déclencheur, votre application logique vous envoie un e-mail pour approuver la requête d’abonnement.

1. Dans l’e-mail d’approbation que vous recevez, sélectionnez **Approuver**.

1. Si l’adresse e-mail de l’abonné n’existe pas dans votre liste de diffusion, votre application logique ajoute l’adresse e-mail de cette personne et vous envoie un e-mail similaire à l’exemple suivant :

   ![Capture d’écran montrant l’exemple d’e-mail pour un abonnement réussi.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-success.png)

   Si votre application logique ne peut pas ajouter l’abonné, vous obtenez un e-mail semblable à l’exemple suivant :

   ![Capture d’écran montrant l’exemple d’e-mail pour un abonnement ayant échoué.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-failed.png)

  > [!TIP]
  > Si vous ne recevez pas d’e-mail, vérifiez le dossier Courrier indésirable de votre messagerie. Il se peut que le filtre de courrier indésirable redirige ces types d’e-mails. Sinon, si vous ne savez pas si votre application logique s’est correctement exécutée, consultez [Dépanner votre application logique](../logic-apps/logic-apps-diagnosing-failures.md).

Félicitations, vous venez de créer et d’exécuter une application logique qui intègre des informations des services Azure et Microsoft et d’autres applications SaaS.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Votre application logique poursuit son exécution jusqu’à ce que vous la désactiviez ou la supprimiez. Quand vous n’avez plus besoin de l’exemple d’application logique, supprimez le groupe de ressources qui contient votre application logique et les ressources associées.

1. Dans la zone de recherche de Portail Azure, entrez le nom du groupe de ressources que vous avez créé. Dans les résultats, sous **Groupes de ressources**, sélectionnez le groupe de ressources.

   Cet exemple crée le groupe de ressources `LA-MailingList-RG`.

   ![Capture d’écran dans laquelle la zone de recherche Azure contient « la-mailinglist-rg » et où **LA-MailingList-RG** est sélectionnée.](./media/tutorial-process-mailing-list-subscriptions-workflow/find-resource-group.png)

   > [!TIP]
   > Si la page d’accueil Azure affiche le groupe de ressources sous **Ressources récentes**, vous pouvez y sélectionner le groupe.

1. Dans le menu du groupe de ressources, vérifiez que **Vue d’ensemble** est sélectionné. Dans la barre d’outils du volet **Vue d’ensemble**, sélectionnez **Supprimer un groupe de ressources**.

   ![Capture d’écran représentant le volet « Vue d’ensemble » du groupe de ressources ; « Supprimer le groupe de ressources » est sélectionné dans la barre d’outils du volet.](./media/tutorial-process-mailing-list-subscriptions-workflow/delete-resource-group.png)

1. Dans le volet de confirmation qui s’affiche, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une application logique qui gère les approbations des demandes de liste de diffusion. Maintenant, découvrez comment créer une application logique qui traite et stocke les pièces jointes des e-mails en intégrant des services Azure, tels que Stockage Azure et Azure Functions.

> [!div class="nextstepaction"]
> [Process emails and attachments with a logic app](../logic-apps/tutorial-process-email-attachments-workflow.md) (Traiter les e-mails et les pièces jointes avec une application logique)
