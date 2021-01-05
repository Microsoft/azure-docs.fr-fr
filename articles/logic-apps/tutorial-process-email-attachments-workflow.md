---
title: Automatiser des tâches avec plusieurs services Azure
description: Tutoriel - Créer des workflows automatisés pour traiter les e-mails avec Azure Logic Apps, le Stockage Azure et Azure Functions
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 02/27/2020
ms.openlocfilehash: 7e58dcf8206ae9feab4d8a09517bf9efda244dd5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451583"
---
# <a name="tutorial-automate-tasks-to-process-emails-by-using-azure-logic-apps-azure-functions-and-azure-storage"></a>Tutoriel : Automatiser les tâches de traitement des e-mails avec Azure Logic Apps, Azure Functions et Stockage Azure

Azure Logic Apps vous aide à automatiser les flux de travail et à intégrer des données dans les services Azure et Microsoft, d’autres applications SaaS (software-as-a-service) et des systèmes locaux. Ce didacticiel montre comment créer une [application logique](../logic-apps/logic-apps-overview.md) qui gère les e-mails entrants et les éventuelles pièces jointes. Cette application logique analyse le contenu de l’e-mail, enregistre le contenu dans Stockage Azure et envoie des notifications de révision de ce contenu.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurez les applications [Stockage Azure](../storage/common/storage-introduction.md) et Explorateur Stockage pour vérifier les e-mails et pièces jointes enregistrés.
> * Créez une [fonction Azure](../azure-functions/functions-overview.md) qui supprime le code HTML des e-mails. Ce didacticiel inclut le code que vous pouvez utiliser pour cette fonction.
> * Créez une application logique vide.
> * Ajoutez un déclencheur qui surveille les e-mails et recherche des pièces jointes.
> * Ajoutez une condition qui vérifie si les e-mails contiennent des pièces jointes.
> * Ajoutez une action qui appelle la fonction Azure lorsqu’un e-mail comporte des pièces jointes.
> * Ajoutez une action qui crée des objets blob de stockage pour les e-mails et les pièces jointes.
> * Ajoutez une action qui envoie des notifications par e-mail.

Lorsque vous avez terminé, votre application logique ressemble au flux de travail suivant à un niveau élevé :

![Application logique terminée de niveau élevé](./media/tutorial-process-email-attachments-workflow/overview.png)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un compte de messagerie d’un fournisseur de messagerie pris en charge par Azure Logic Apps, par exemple Office 365 Outlook, Outlook.com ou Gmail. Pour les autres fournisseurs, [passez en revue la liste des connecteurs ici](/connectors/).

  Cette application logique utilise un compte professionnel ou scolaire. Si vous utilisez un autre compte de messagerie, les étapes générales sont identiques, mais l’affichage de l’interface utilisateur peut être légèrement différent.

  > [!IMPORTANT]
  > Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans Logic Apps. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser ce connecteur uniquement avec certains services approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification avec votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Téléchargez et installez [l’Explorateur Stockage Microsoft Azure gratuit](https://storageexplorer.com/). Cet outil vous permet de vérifier que votre conteneur de stockage est correctement configuré.

## <a name="set-up-storage-to-save-attachments"></a>Configurer le stockage pour y enregistrer les pièces jointes

Vous pouvez enregistrer les e-mails entrants et les pièces jointes en tant qu’objets blob dans un [conteneur de stockage Azure](../storage/common/storage-introduction.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Avant de créer un conteneur de stockage, [créez un compte de stockage](../storage/common/storage-account-create.md) avec ces paramètres sous l’onglet **De base** dans le portail Azure :

   | Paramètre | Valeur | Description |
   |---------|-------|-------------|
   | **Abonnement** | <*Azure-subscription-name*> | Nom de votre abonnement Azure. |  
   | **Groupe de ressources** | <*Azure-resource-group*> | Nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md) utilisé pour organiser et gérer les ressources connexes. Cet exemple utilise « LA-Tutorial-RG ». <p>**Remarque :** Un groupe de ressources existe dans une région spécifique. Même si les éléments de ce didacticiel ne sont pas forcément disponibles dans toutes les régions, essayez d’utiliser la même région dans la mesure du possible. |
   | **Nom du compte de stockage** | <*Azure-storage-account-name*> | Nom de votre compte de stockage, qui doit comporter entre 3 et 24 caractères, et ne peut contenir que des lettres minuscules et des chiffres. Cet exemple utilise « attachmentstorageacct ». |
   | **Lieu** | <*Azure-region*> | Région dans laquelle stocker les informations sur votre compte de stockage. Cet exemple utilise la région « USA Ouest ». |
   | **Performances** | Standard | Ce paramètre spécifie les types de données pris en charge et les médias de stockage des données. Voir [Types de compte de stockage](../storage/common/storage-introduction.md#types-of-storage-accounts). |
   | **Type de compte** | Usage général | [Type de compte de stockage](../storage/common/storage-introduction.md#types-of-storage-accounts). |
   | **Réplication** | Stockage localement redondant (LRS) | Ce paramètre spécifie comment vos données sont copiées, stockées, gérées et synchronisées. Consultez [Stockage localement redondant (LRS) : redondance des données à faible coût pour le stockage Azure](../storage/common/storage-redundancy.md). |
   | **Niveau d’accès (par défaut)** | Conservez le paramètre actuel. |
   ||||

   Sous l’onglet **Avancé**, sélectionnez ce paramètre :

   | Paramètre | Valeur | Description |
   |---------|-------|-------------|
   | **Transfert sécurisé requis** | Désactivé | Ce paramètre spécifie la sécurité requise pour les demandes de connexions. Voir [Exiger un transfert sécurisé dans Stockage Azure](../storage/common/storage-require-secure-transfer.md). |
   ||||

   Pour créer votre compte de stockage, vous pouvez également utiliser [Azure PowerShell](../storage/common/storage-account-create.md?tabs=powershell) ou [Azure CLI](../storage/common/storage-account-create.md?tabs=azure-cli).

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**.

1. Une fois qu’Azure a déployé votre compte de stockage, recherchez-le, puis obtenez sa clé d’accès :

   1. Dans le menu de votre compte de stockage, sous **Paramètres**, sélectionnez **Clés d’accès**.

   1. Copiez le nom de votre compte de stockage et **key1**, puis enregistrez ces valeurs dans un endroit sûr.

      ![Copier et enregistrer un nom de compte de stockage et une clé](./media/tutorial-process-email-attachments-workflow/copy-save-storage-name-key.png)

   Pour obtenir la clé d’accès de votre compte de stockage, vous pouvez également utiliser [Azure PowerShell](/powershell/module/az.storage/get-azstorageaccountkey) ou [Azure CLI](/cli/azure/storage/account/keys?view=azure-cli-latest.md#az-storage-account-keys-list).

1. Créez un conteneur de stockage d’objets blob pour vos pièces jointes.

   1. Dans le menu de votre compte de stockage, sélectionnez **Vue d’ensemble**. Dans le volet Vue d’ensemble, sélectionnez **Conteneurs**.

      ![Ajouter un conteneur de stockage d’objets blob](./media/tutorial-process-email-attachments-workflow/create-storage-container.png)

   1. Lorsque la page **Conteneurs** s’ouvre, dans la barre d’outils, sélectionnez **Conteneur**.

   1. Sous **Nouveau conteneur**, entrez `attachments` comme nom de votre conteneur. Sous **Niveau d’accès public**, sélectionnez **Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)**  > **OK**.

      Lorsque vous avez terminé, vous pouvez trouver votre conteneur de stockage dans votre compte de stockage ici dans le portail Azure :

      ![Conteneur de stockage terminé](./media/tutorial-process-email-attachments-workflow/created-storage-container.png)

   Pour créer un conteneur de stockage, vous pouvez également utiliser [Azure PowerShell](/powershell/module/az.storage/new-azstoragecontainer) ou [Azure CLI](/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create).

À présent, connectez l’Explorateur Stockage à votre compte de stockage.

## <a name="set-up-storage-explorer"></a>Configurer l’Explorateur Stockage

Ensuite, connectez l’Explorateur Stockage à votre compte de stockage afin de vérifier que votre application logique peut enregistrer correctement les pièces jointes en tant qu’objets blob dans votre conteneur de stockage.

1. Lancez Microsoft Azure Storage Explorer.

   L’Explorateur Stockage vous invite à vous connecter à votre compte de stockage.

1. Dans le volet **Se connecter au Stockage Azure**, sélectionnez **Utiliser un nom et une clé de compte de stockage** > **Suivant**.

   ![Explorateur Stockage - Connexion au compte de stockage](./media/tutorial-process-email-attachments-workflow/storage-explorer-choose-storage-account.png)

   > [!TIP]
   > Si aucune invite ne s’affiche, sur la barre d’outils de l’Explorateur Stockage, sélectionnez **Ajouter un compte**.

1. Sous **Nom complet**, fournissez un nom convivial pour votre connexion. Sous **Nom du compte**, fournissez le nom de votre compte de stockage. Sous **Clé de compte**, fournissez la clé d’accès que vous avez enregistrée précédemment, puis sélectionnez **Suivant**.

1. Confirmez vos informations de connexion, puis sélectionnez **Se connecter**.

   L’Explorateur Stockage crée la connexion et affiche votre compte de stockage dans la fenêtre de l’Explorateur sous **Local et attaché** > **Comptes de stockage**.

1. Pour rechercher votre conteneur de stockage d’objets blob, sous **Comptes de stockage**, développez votre compte de stockage, qui est **attachmentstorageacct** ici, puis développez **Conteneurs d’objets blob** où se trouve le conteneur **pièces jointes** ; par exemple :

   ![Explorateur Stockage - rechercher le conteneur de stockage](./media/tutorial-process-email-attachments-workflow/storage-explorer-check-contianer.png)

Créez une [fonction Azure](../azure-functions/functions-overview.md) qui supprime le code HTML des e-mails entrants.

## <a name="create-function-to-clean-html"></a>Créer une fonction pour nettoyer le code HTML

Utilisez l’extrait de code fourni par ces étapes pour créer une fonction Azure qui supprime le code HTML dans chaque e-mail entrant. De cette façon, le contenu des e-mails est plus net et plus facile à traiter. Vous pouvez ensuite appeler cette fonction à partir de votre application logique.

1. Avant de pouvoir créer une fonction, [créez une application de fonction](../azure-functions/functions-create-function-app-portal.md) avec les paramètres suivants :

   | Paramètre | Valeur | Description |
   | ------- | ----- | ----------- |
   | **Nom de l’application** | <*function-app-name*> | Nom de votre application de fonction, qui doit être globalement unique dans Azure. Cet exemple utilise déjà « CleanTextFunctionApp ». Par conséquent, fournissez un autre nom, comme « MyCleanTextFunctionApp-<*votre_nom*> » |
   | **Abonnement** | <*your-Azure-subscription-name*> | Abonnement Azure que vous avez utilisé précédemment. |
   | **Groupe de ressources** | LA-Tutorial-RG | Groupe de ressources Azure que vous avez utilisé précédemment. |
   | **SE** | <*votre_système_exploitation*> | Sélectionnez le système d’exploitation qui prend en charge votre langage de programmation de fonction favori. Pour cet exemple, sélectionnez **Windows**. |
   | **Plan d’hébergement** | Plan de consommation | Ce paramètre détermine l’affectation et la mise à l’échelle des ressources, telles que la puissance de calcul, pour l’exécution de votre application de fonction. Voir [Comparaison des plans d’hébergement](../azure-functions/functions-scale.md). |
   | **Lieu** | USA Ouest | Région que vous avez utilisée précédemment. |
   | **Pile d’exécution** | Langage préféré | Sélectionnez un runtime qui prend en charge votre langage de programmation de fonction favori. Sélectionnez **.NET** pour les fonctions C# et F#. |
   | **Stockage** | cleantextfunctionstorageacct | Créez un compte de stockage pour votre application de fonction. Utilisez uniquement des lettres minuscules et des chiffres. <p>**Remarque :** Ce compte de stockage contient vos applications de fonctions et diffère du compte de stockage créé précédemment pour les pièces jointes d’e-mail. |
   | **Application Insights** | Disable | Active la supervision des applications avec [Application Insights](../azure-monitor/app/app-insights-overview.md), mais pour ce tutoriel, sélectionnez **Désactiver** > **Appliquer**. |
   ||||

   Si votre application de fonction ne s’ouvre pas automatiquement après le déploiement, dans la zone de recherche [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Application de fonction**. Sous **Application de fonction**, sélectionnez votre application de fonction.

   ![Sélectionner l’application de fonction](./media/tutorial-process-email-attachments-workflow/select-function-app.png)

   Sinon, Azure ouvre automatiquement votre application de fonction comme illustré ici :

   ![Application de fonction créée](./media/tutorial-process-email-attachments-workflow/function-app-created.png)

   Pour créer une application de fonction, vous pouvez également utiliser [Azure CLI](../azure-functions/create-first-function-cli-csharp.md) ou des [modèles PowerShell et Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

1. Dans la liste **Applications de fonctions**, développez votre application de fonction, si ce n’est déjà fait. Sous votre application de fonction, sélectionnez **Fonction**. Dans la barre d’outils des fonctions, sélectionnez **Nouvelle fonction**.

   ![Créer une fonction](./media/tutorial-process-email-attachments-workflow/function-app-new-function.png)

1. Sous **Choisir un modèle ci-dessous ou accéder au démarrage rapide**, sélectionnez le modèle **Déclencheur HTTP**.

   ![Sélectionnez le modèle de déclencheur HTTP](./media/tutorial-process-email-attachments-workflow/function-select-httptrigger-csharp-function-template.png)

   Azure crée une fonction à l’aide d’un modèle propre à un langage pour une fonction déclenchée via HTTP.

1. Dans le volet **Nouvelle fonction**, sous **Nom**, entrez `RemoveHTMLFunction`. Conservez le **Niveau d’autorisation** défini sur **Fonction**, puis sélectionnez **Créer**.

   ![Nommer votre fonction](./media/tutorial-process-email-attachments-workflow/function-provide-name.png)

1. Dans l’éditeur ouvert, remplacez le code du modèle par cet exemple de code, ce qui supprime le code HTML et retourne les résultats à l’appelant :

   ```csharp
   #r "Newtonsoft.Json"

   using System.Net;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Extensions.Primitives;
   using Newtonsoft.Json;
   using System.Text.RegularExpressions;

   public static async Task<IActionResult> Run(HttpRequest req, ILogger log) {

      log.LogInformation("HttpWebhook triggered");

      // Parse query parameter
      string emailBodyContent = await new StreamReader(req.Body).ReadToEndAsync();

      // Replace HTML with other characters
      string updatedBody = Regex.Replace(emailBodyContent, "<.*?>", string.Empty);
      updatedBody = updatedBody.Replace("\\r\\n", " ");
      updatedBody = updatedBody.Replace(@"&nbsp;", " ");

      // Return cleaned text
      return (ActionResult)new OkObjectResult(new { updatedBody });
   }
   ```

1. Quand vous avez terminé, sélectionnez **Enregistrer**. Pour tester votre fonction, sur le côté droit de l’éditeur, sous l’icône de flèche ( **<** ), sélectionnez **Test**.

   ![Ouvrir le volet Test](./media/tutorial-process-email-attachments-workflow/function-choose-test.png)

1. Dans le volet **Test**, sous **Corps de la requête**, entrez la ligne suivante, puis sélectionnez **Exécuter**.

   `{"name": "<p><p>Testing my function</br></p></p>"}`

   ![Tester votre fonction](./media/tutorial-process-email-attachments-workflow/function-run-test.png)

   La fenêtre **Sortie** affiche le résultat de la fonction :

   ```json
   {"updatedBody":"{\"name\": \"Testing my function\"}"}
   ```

Après avoir vérifié le bon fonctionnement de votre fonction, créez votre application logique. Même si ce didacticiel montre comment créer une fonction qui supprime le code HTML des e-mails, Logic Apps fournit également un connecteur **HTML to Text (HTML à texte)** .

## <a name="create-your-logic-app"></a>Créer votre application logique

1. Dans la zone de recherche Azure du haut, entrez `logic apps`, puis sélectionnez **Logic Apps**.

   ![Rechercher et sélectionner « Logic Apps »](./media/tutorial-process-email-attachments-workflow/find-select-logic-apps.png)

1. Dans le volet **Logic Apps**, sélectionnez **Ajouter**.

   ![Ajouter une nouvelle application logique](./media/tutorial-process-email-attachments-workflow/add-new-logic-app.png)

1. Dans le volet **Application logique**, indiquez les détails de votre application logique comme indiqué ici. Quand vous avez terminé, sélectionnez **Vérifier + créer**.

   ![Spécifier les informations de l’application logique](./media/tutorial-process-email-attachments-workflow/create-logic-app-settings.png)

   | Paramètre | Valeur | Description |
   | ------- | ----- | ----------- |
   | **Abonnement** | <*your-Azure-subscription-name*> | Abonnement Azure que vous avez utilisé précédemment. |
   | **Groupe de ressources** | LA-Tutorial-RG | Groupe de ressources Azure que vous avez utilisé précédemment. |
   | **Nom de l’application logique** | LA-ProcessAttachment | Nom de l’application logique. |
   | **Sélectionner l’emplacement** | USA Ouest | Région que vous avez utilisée précédemment. |
   | **Log Analytics** | Désactivé | Pour ce tutoriel, sélectionnez le paramètre **Désactivé**. |
   ||||

1. Une fois qu’Azure a déployé votre application, dans la barre d’outils Azure, sélectionnez l’icône de notifications, puis **Accéder à la ressource**.

   ![Dans la liste des notifications Azure, sélectionnez « Accéder à la ressource »](./media/tutorial-process-email-attachments-workflow/go-to-new-logic-app-resource.png)

1. Le Concepteur Logic Apps s’ouvre ensuite et affiche une page contenant une vidéo de présentation et des modèles d’applications logiques courantes. Sous **Modèles**, sélectionnez **Application logique vide**.

   ![Sélectionner un modèle d’application logique vide](./media/tutorial-process-email-attachments-workflow/choose-logic-app-template.png)

Ajoutez maintenant un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) qui écoute les e-mails entrants comportant des pièces jointes. Chaque application logique doit commencer par un déclencheur, qui est activé lorsqu’un événement spécifique se produit ou lorsque de nouvelles données respectent une condition particulière. Pour plus d’informations, consultez [Quickstart: Build your first logic app workflow - Azure portal](../logic-apps/quickstart-create-first-logic-app-workflow.md) (Démarrage rapide : Générer votre premier flux de travail d’application logique - portail Azure).

## <a name="monitor-incoming-email"></a>Surveiller les e-mails entrants

1. Dans la zone de recherche du concepteur, entrez `when new email arrives` comme filtre. Sélectionnez ce déclencheur pour votre fournisseur de messagerie : **Lorsqu’un nouvel e-mail arrive - <*your-email-provider*>**

   Par exemple :

   ![Sélectionnez ce déclencheur pour votre fournisseur de messagerie : « When a new email arrives » (Quand un nouvel e-mail arrive)](./media/tutorial-process-email-attachments-workflow/add-trigger-when-email-arrives.png)

   * Pour les comptes Azure professionnels ou scolaires, sélectionnez Office 365 Outlook.

   * Pour les comptes Microsoft personnels, sélectionnez Outlook.com.

1. Si vous êtes invité à entrer vos informations d’identification, connectez-vous à votre compte de messagerie afin que Logic Apps puisse se connecter à ce compte.

1. Indiquez maintenant les critères que le déclencheur doit utiliser pour filtrer les nouveaux e-mails.

   1. Spécifiez les paramètres décrits ci-dessous pour la vérification des e-mails.

      ![Spécifier le dossier, l’intervalle et la fréquence de vérification des e-mails](./media/tutorial-process-email-attachments-workflow/set-up-email-trigger.png)

      | Paramètre | Valeur | Description |
      | ------- | ----- | ----------- |
      | **Folder** | Inbox | Dossier d’e-mail à vérifier |
      | **Contient une pièce jointe** | Oui | Récupère uniquement les e-mails comportant des pièces jointes. <p>**Remarque :** Le déclencheur ne supprime pas les e-mails de votre compte. Il vérifie uniquement les nouveaux messages et ne traite que les e-mails qui correspondent au filtre Objet. |
      | **Inclure des pièces jointes** | Oui | Récupérez les pièces jointes en tant qu’entrée de votre flux de travail au lieu de les rechercher simplement. |
      | **Intervalle** | 1 | Nombre d’intervalles d’attente entre les vérifications. |
      | **Fréquence** | Minute | Unité de temps de chaque intervalle entre les vérifications. |
      ||||

   1. Dans la liste **Ajouter un nouveau paramètre**, sélectionnez **Filtre Objet**.

   1. Une fois que la zone **Filtre Objet** s’affiche dans l’action, spécifiez l’objet comme indiqué ici :

      | Paramètre | Valeur | Description |
      | ------- | ----- | ----------- |
      | **Filtre Objet** | `Business Analyst 2 #423501` | Texte à rechercher dans l’objet de l’e-mail. |
      ||||

1. Pour masquer les informations du déclencheur pour le moment, cliquez dans sa barre de titre.

   ![Réduire la forme pour masquer les informations](./media/tutorial-process-email-attachments-workflow/collapse-trigger-shape.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

   Votre application logique est à présent active, mais elle vérifie uniquement vos e-mails. Ajoutez une condition qui spécifie les critères pour poursuivre le flux de travail.

## <a name="check-for-attachments"></a>Rechercher des pièces jointes

Ajoutez maintenant une condition qui sélectionne uniquement les e-mails contenant des pièces jointes.

1. Sous le déclencheur, sélectionnez **Nouvelle étape**.

   ![« Nouvelle étape »](./media/tutorial-process-email-attachments-workflow/add-condition-under-trigger.png)

1. Sous **Choisir une action**, dans la zone de recherche, entrez `condition`. Sélectionnez cette action : **Condition**

   ![Sélectionnez « Condition »](./media/tutorial-process-email-attachments-workflow/select-condition.png)

   1. Renommez la condition en utilisant une meilleure description. Dans la barre de titre de la condition, sélectionnez le bouton représentant des points de suspension ( **...** ) > **Renommer**.

      ![Renommer la condition](./media/tutorial-process-email-attachments-workflow/condition-rename.png)

   1. Renommez votre condition à l’aide de cette description : `If email has attachments and key subject phrase`

1. Créez une condition qui recherche les e-mails contenant des pièces jointes.

   1. Sur la première ligne, sous **et**, cliquez à l’intérieur de la zone de gauche. Dans la liste de contenu dynamique qui s’affiche, sélectionnez la propriété **Comporte une pièce jointe**.

      ![Capture d’écran représentant la propriété « and » pour la condition et la sélection de la propriété « Comporte une pièce jointe ».](./media/tutorial-process-email-attachments-workflow/build-condition.png)

   1. Dans la zone du milieu, conservez l’opérateur **est égal à**.

   1. Dans la zone de droite, entrez **True** comme valeur à comparer avec la valeur de la propriété **Comporte une pièce jointe** du déclencheur.

      ![Créer une condition](./media/tutorial-process-email-attachments-workflow/finished-condition.png)

      Si les deux valeurs sont égales, l’e-mail comporte au moins une pièce jointe, la condition est transmise et le flux de travail se poursuit.

   Dans votre définition d’application logique sous-jacente, que vous pouvez afficher dans la fenêtre de l’éditeur de code, cette condition se présente comme dans cet exemple :

   ```json
   "Condition": {
      "actions": { <actions-to-run-when-condition-passes> },
      "expression": {
         "and": [ {
            "equals": [
               "@triggerBody()?['HasAttachment']",
                 "true"
            ]
         } ]
      },
      "runAfter": {},
      "type": "If"
   }
   ```

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

### <a name="test-your-condition"></a>Tester votre condition

Vérifiez si la condition fonctionne correctement :

1. Si votre application logique n’est pas déjà en cours d’exécution, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

   Cette étape démarre manuellement votre application logique sans avoir à attendre la transmission de l’intervalle spécifié. Toutefois, rien ne se produit tant que l’e-mail de test n’est pas arrivé dans votre boîte de réception.

1. Envoyez-vous un e-mail qui respecte les critères suivants :

   * L’objet de votre e-mail contient le texte que vous avez spécifié dans le **Filtre Objet** du déclencheur : `Business Analyst 2 #423501`

   * Votre e-mail comporte une pièce jointe. Pour le moment, créez simplement un fichier texte vide et joignez-le à votre e-mail.

   Lorsque l’e-mail arrive, votre application logique recherche des pièces jointes et le texte de l’objet spécifié. Si la condition est transmise, le déclencheur est activé et provoque la création d’une instance d’application logique et le démarrage du flux de travail par le moteur Logic Apps.

1. Pour vérifier que le déclencheur a été activé et que l’application logique s’est exécutée avec succès, sélectionnez **Vue d’ensemble** dans le menu de l’application logique.

   ![Vérifier l’historique du déclencheur et l’historique des exécutions](./media/tutorial-process-email-attachments-workflow/checkpoint-run-history.png)

   Si votre application logique n’a pas été activée ni exécutée en dépit d’un déclencheur réussi, voir [Résoudre les problèmes et diagnostiquer les échecs d’applications logiques](../logic-apps/logic-apps-diagnosing-failures.md).

À présent, définissez les actions à entreprendre pour la branche **Si true**. Pour enregistrer l’e-mail ainsi que les pièces jointes, supprimez le code HTML dans le corps de l’e-mail, puis créez des objets blob dans le conteneur de stockage correspondant à l’e-mail et aux pièces jointes.

> [!NOTE]
> Votre application logique n’a rien à faire pour la branche **Si false** lorsqu’un e-mail ne contient pas de pièces jointes.
> À titre d’exercice supplémentaire à l’issue de ce didacticiel, vous pouvez ajouter une action appropriée que vous souhaitez entreprendre pour la branche **Si false**.

## <a name="call-removehtmlfunction"></a>Appeler RemoveHTMLFunction

Cette étape ajoute votre fonction Azure créée précédemment à votre application logique et transmet le contenu du corps de l’e-mail depuis le déclencheur d’e-mail à votre fonction.

1. Dans le menu de l’application logique, sélectionnez **Concepteur d’application logique**. Dans la branche **Si true**, sélectionner **Ajouter une action**.

   ![À l’intérieur de « Si true », ajouter une action](./media/tutorial-process-email-attachments-workflow/if-true-add-action.png)

1. Dans la zone de recherche, recherchez « Azure Functions » et sélectionnez cette action : **Choisissez une fonction Azure - Azure Functions**

   ![Sélectionner une action pour « Choisir une fonction Azure »](./media/tutorial-process-email-attachments-workflow/add-action-azure-function.png)

1. Sélectionnez votre application de fonction créée précédemment, qui est `CleanTextFunctionApp` dans cet exemple :

   ![Sélectionner votre application de fonction Azure](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function-app.png)

1. Sélectionnez votre fonction : **RemoveHTMLFunction**

   ![Sélectionner votre application de fonction](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function.png)

1. Renommez la forme de votre fonction avec cette description : `Call RemoveHTMLFunction to clean email body`.

1. Maintenant, spécifiez l’entrée pour votre fonction à traiter.

   1. Pour le champ **Corps de la demande**, entrez ce texte avec un espace de fin :

      `{ "emailBody":`

      Pendant que vous travaillez sur cette entrée dans les étapes suivantes, une erreur de JSON non valide s’affiche jusqu’à ce que votre entrée soit correctement formatée en JSON. Lorsque vous avez testé précédemment cette fonction, l’entrée spécifiée pour cette fonction utilisait JavaScript Objet Notation (JSON). Par conséquent, le corps de la requête doit également utiliser le même format.

      De plus, lorsque le curseur se trouve à l’intérieur de la zone **Corps de la demande**, la liste de contenu dynamique s’affiche afin que vous puissiez sélectionner les valeurs de propriété disponibles à partir des actions précédentes.

   1. Dans la liste de contenu dynamique, sous **À la réception d’un e-mail**, sélectionnez la propriété **Corps**. Après cette propriété, pensez à ajouter l’accolade fermante : `}`

      ![Spécifier le corps de la requête à transmettre à la fonction](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing.png)

   Une fois que vous avez terminé, l’entrée ajoutée à votre fonction ressemble à cet exemple :

   ![Fin du corps de la demande à transmettre à votre fonction](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing-2.png)

1. Enregistrez votre application logique.

Ensuite, ajoutez une action qui crée un objet blob dans votre conteneur de stockage pour vous permettre d’enregistrer le corps de l’e-mail.

## <a name="create-blob-for-email-body"></a>Créer un objet blob pour le corps de l’e-mail

1. Dans le bloc **Si true** et sous votre fonction Azure, sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez `create blob` comme filtre, puis sélectionnez cette action : **Créer un objet blob**

   ![Ajouter une action pour créer un objet blob pour le corps de l’e-mail](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-email-body.png)

1. Créez une connexion à votre compte de stockage avec ces paramètres comme indiqué et décrit ici. Sélectionnez **Créer** lorsque vous avez terminé.

   ![Créer une connexion au compte de stockage](./media/tutorial-process-email-attachments-workflow/create-storage-account-connection-first.png)

   | Paramètre | Valeur | Description |
   | ------- | ----- | ----------- |
   | **Nom de connexion** | AttachmentStorageConnection | Nom descriptif de la connexion. |
   | **Compte de stockage** | attachmentstorageacct | Nom du compte de stockage que vous avez créé précédemment pour enregistrer des pièces jointes. |
   ||||

1. Renommez l’action **Créer un objet blob** avec cette description : `Create blob for email body`.

1. Dans l’action **Créer un objet blob**, indiquez ces informations, puis sélectionnez ces champs pour créer l’objet blob comme indiqué et décrit :

   ![Fournir des informations d’objet blob pour le corps de l’e-mail](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body.png)

   | Paramètre | Valeur | Description |
   | ------- | ----- | ----------- |
   | **Chemin d’accès du dossier** | /pièces jointes | Chemin d’accès et nom du conteneur que vous avez créé précédemment. Pour cet exemple, cliquez sur l’icône de dossier, puis sélectionnez le conteneur « /pièces jointes ». |
   | **Nom de l’objet blob** | Champ **De** | Pour cet exemple, utilisez le nom de l’expéditeur comme nom de l’objet blob. Cliquez dans cette zone pour que la liste de contenu dynamique s’affiche, puis sélectionnez le champ **De** sous l’action **À l’arrivée d’un e-mail**. |
   | **Contenu de l’objet blob** | Champ **Contenu** | Pour cet exemple, utilisez le corps de l’e-mail sans code HTML comme contenu d’objet blob. Cliquez dans cette zone pour que la liste de contenu dynamique s’affiche, puis sélectionnez **Corps** sous l’action **Call RemoveHTMLFunction to clean email body** (Appeler RemoveHTMLFunction pour nettoyer le corps de l’e-mail). |
   ||||

   Une fois que vous avez terminé, l’action ressemble à cet exemple :

   ![Capture d’écran représentant un exemple d’action « Créer un objet blob » terminé.](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body-done.png)

1. Enregistrez votre application logique.

### <a name="check-attachment-handling"></a>Vérifier le traitement des pièces jointes

Vérifiez à présent si votre application logique gère les e-mails comme vous l’avez spécifié :

1. Si votre application logique n’est pas déjà en cours d’exécution, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

1. Envoyez-vous un e-mail qui respecte les critères suivants :

   * L’objet de votre e-mail contient le texte que vous avez spécifié dans le **Filtre Objet** du déclencheur : `Business Analyst 2 #423501`

   * Votre e-mail comporte au moins une pièce jointe. Pour le moment, créez simplement un fichier texte vide, puis joignez-le à votre e-mail.

   * Le corps de votre e-mail comporte du contenu de test, par exemple : `Testing my logic app`

   Si votre application logique n’a pas été activée ni exécutée en dépit d’un déclencheur réussi, voir [Résoudre les problèmes et diagnostiquer les échecs d’applications logiques](../logic-apps/logic-apps-diagnosing-failures.md).

1. Vérifiez que votre application logique a enregistré l’e-mail dans le conteneur de stockage approprié.

   1. Dans l’Explorateur Stockage, développez **Local et attaché** > **Comptes de stockage** > **attachmentstorageacct (Clé)**  > **Conteneurs d’objets blob** > **pièces jointes**.

   1. Vérifiez le conteneur **pièces jointes** de l’e-mail.

      À ce stade, seul l’e-mail s’affiche dans le conteneur, car l’application logique ne traite pas encore les pièces jointes.

      ![Rechercher l’e-mail enregistré dans l’Explorateur Stockage](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-email.png)

   1. Lorsque vous avez terminé, supprimez l’e-mail dans l’Explorateur Stockage.

1. Si vous le souhaitez, pour tester la branche **Si false** qui n’effectue aucune action pour le moment, vous pouvez envoyer un e-mail qui ne réponde pas aux critères.

Ajoutez une boucle pour traiter toutes les pièces jointes.

## <a name="process-attachments"></a>Traiter des pièces jointes

Pour traiter chaque pièce jointe de l’e-mail, ajoutez une boucle **For Each** au flux de votre application logique.

1. Sous la forme **Créer un objet blob pour le corps de l’e-mail**, sélectionnez **Ajouter une action**.

   ![Ajouter une boucle « For Each »](./media/tutorial-process-email-attachments-workflow/add-for-each-loop.png)

1. Sous **Choisir une action**, dans la zone de recherche, entrez `for each` comme filtre, puis sélectionnez cette action : **For each**

   ![Sélectionnez « For each »](./media/tutorial-process-email-attachments-workflow/select-for-each.png)

1. Renommez votre boucle à l’aide de cette description : `For each email attachment`.

1. Indiquez les données de la boucle à traiter. Cliquez dans la zone **Sélectionnez un résultat à partir des étapes précédentes** afin que la liste de contenu dynamique s’ouvre, puis sélectionnez **Pièces jointes**.

   ![Sélectionner « Pièces jointes »](./media/tutorial-process-email-attachments-workflow/select-attachments.png)

   Le champ **Pièces jointes** transmet un tableau qui contient toutes les pièces jointes incluses dans un e-mail. La boucle **For Each** répète des actions sur chaque élément transmis avec le tableau.

1. Enregistrez votre application logique.

Ajoutez l’action qui enregistre chaque pièce jointe sous la forme d’un objet blob dans votre conteneur de stockage **pièces jointes**.

## <a name="create-blob-for-each-attachment"></a>Créer un objet blob pour chaque pièce jointe

1. Dans la boucle **Pour chaque pièce jointe d’e-mail**, sélectionnez **Ajouter une action** afin de pouvoir spécifier la tâche à effectuer sur chaque pièce jointe trouvée.

   ![Ajouter une action à la boucle](./media/tutorial-process-email-attachments-workflow/for-each-add-action.png)

1. Dans la zone de recherche, entrez `create blob` comme filtre, puis sélectionnez cette action : **Créer un objet blob**

   ![Ajouter une action pour créer un objet blob](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-attachments.png)

1. Renommez l’action **Créer un objet blob 2** avec cette description : `Create blob for each email attachment`.

1. Dans l’action **Créer un objet blob pour chaque pièce jointe d’e-mail**, indiquez ces informations, puis sélectionnez les propriétés pour chaque objet blob à créer comme indiqué et décrit :

   ![Fournir des informations sur l’objet blob](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment.png)

   | Paramètre | Valeur | Description |
   | ------- | ----- | ----------- |
   | **Chemin d’accès du dossier** | /pièces jointes | Chemin d’accès et nom du conteneur que vous avez créé précédemment. Pour cet exemple, cliquez sur l’icône de dossier, puis sélectionnez le conteneur « /pièces jointes ». |
   | **Nom de l’objet blob** | Champ **Nom** | Pour cet exemple, utilisez le nom de la pièce jointe comme nom de l’objet blob. Cliquez dans cette zone pour que la liste de contenu dynamique s’affiche, puis sélectionnez le champ **Nom** sous l’action **À l’arrivée d’un e-mail**. |
   | **Contenu de l’objet blob** | Champ **Contenu** | Pour cet exemple, utilisez le champ **Contenu** comme contenu d’objet blob. Cliquez dans cette zone pour que la liste de contenu dynamique s’affiche, puis sélectionnez le champ **Contenu** sous l’action **À l’arrivée d’un e-mail**. |
   ||||

   Une fois que vous avez terminé, l’action ressemble à cet exemple :

   ![Fin de l’action « Créer un objet blob »](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment-done.png)

1. Enregistrez votre application logique.

### <a name="check-attachment-handling"></a>Vérifier le traitement des pièces jointes

À présent, vérifiez si votre application logique gère les pièces jointes comme vous l’avez spécifié :

1. Si votre application logique n’est pas déjà en cours d’exécution, sélectionnez **Exécuter** dans la barre d’outils du concepteur.

1. Envoyez-vous un e-mail qui respecte les critères suivants :

   * L’objet de votre e-mail contient le texte que vous avez spécifié dans la propriété **Filtre d’objet** du déclencheur : `Business Analyst 2 #423501`

   * Votre e-mail comporte au moins deux pièces jointes. Pour le moment, créez simplement deux fichiers texte vides et joignez-les à votre e-mail.

   Si votre application logique n’a pas été activée ni exécutée en dépit d’un déclencheur réussi, voir [Résoudre les problèmes et diagnostiquer les échecs d’applications logiques](../logic-apps/logic-apps-diagnosing-failures.md).

1. Vérifiez que votre application logique a enregistré l’e-mail et les pièces jointes dans le conteneur de stockage approprié.

   1. Dans l’Explorateur Stockage, développez **Local et attaché** > **Comptes de stockage** > **attachmentstorageacct (Clé)**  > **Conteneurs d’objets blob** > **pièces jointes**.

   1. Dans le conteneur **pièces de jointes**, recherchez l’e-mail et les pièces jointes.

      ![Rechercher l’e-mail et les pièces jointes enregistrés](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-attachments.png)

   1. Lorsque vous avez terminé, supprimez l’e-mail et les pièces jointes dans l’Explorateur Stockage.

Ajoutez une action afin que votre application logique envoie un e-mail pour passer en revue les pièces jointes.

## <a name="send-email-notifications"></a>Envoyer des notifications par e-mail

1. Dans la branche **Si true**, sous la boucle **Pour chaque pièce jointe d’e-mail**, sélectionnez **Ajouter une action**.

   ![Ajouter une action sous la boucle « For Each »](./media/tutorial-process-email-attachments-workflow/add-action-send-email.png)

1. Dans la zone de recherche, entrez `send email` comme filtre, puis sélectionnez l’action « send email » (envoi d’e-mail) pour votre fournisseur de messagerie.

   Pour filtrer la liste d’actions pour un service spécifique, vous pouvez sélectionner tout d’abord le connecteur.

   ![Sélectionner l’action « Envoyer un e-mail » pour votre fournisseur de messagerie](./media/tutorial-process-email-attachments-workflow/add-action-select-send-email.png)

   * Pour les comptes Azure professionnels ou scolaires, sélectionnez Office 365 Outlook.

   * Pour les comptes Microsoft personnels, sélectionnez Outlook.com.

1. Si vous êtes invité à entrer vos informations d’identification, connectez-vous à votre compte de messagerie afin que Logic Apps puisse établir une connexion avec votre compte de messagerie.

1. Renommez l’action **Envoyer un e-mail** avec cette description : `Send email for review`.

1. Indiquez les informations de cette action, puis sélectionnez les champs que vous souhaitez inclure dans l’e-mail comme indiqué et décrit. Pour ajouter des lignes vides dans une zone d’édition, appuyez sur Maj + Entrée.

   ![Envoyer une notification par e-mail](./media/tutorial-process-email-attachments-workflow/send-email-notification.png)

   Si vous ne trouvez pas un champ attendu dans la liste de contenu dynamique, sélectionnez **Voir plus** en regard de **À la réception d’un e-mail**.

   | Paramètre | Valeur | Notes |
   | ------- | ----- | ----- |
   | **To** | <*recipient-email-address*> | À des fins de test, vous pouvez utiliser votre propre adresse e-mail. |
   | **Subject**  | ```ASAP - Review applicant for position:``` **Objet** | Objet de l’e-mail que vous souhaitez inclure. Cliquez dans cette zone, entrez l’exemple de texte et dans la liste de contenu dynamique, sélectionnez le champ **Objet** sous **À l’arrivée d’un e-mail**. |
   | **Corps** | ```Please review new applicant:``` <p>```Applicant name:``` **De** <p>```Application file location:``` **Chemin** <p>```Application email content:``` **Corps** | Contenu du corps de l’e-mail. Cliquez dans cette zone, entrez l’exemple de texte et dans la liste de contenu dynamique, sélectionnez ces champs : <p>- Champ **De** situé sous **À la réception d’un e-mail** </br>- Champ **Chemin d’accès** situé sous **Créer un objet blob pour le corps de l’e-mail** </br>- Champ **Corps** situé sous **Call RemoveHTMLFunction to clean email body (Appeler RemoveHTMLFunction pour nettoyer le corps de l’e-mail)** |
   ||||

   > [!NOTE]
   > Si vous sélectionnez un champ qui contient un tableau, tel que le champ **Contenu**, qui est un tableau contenant des pièces jointes, le concepteur ajoute automatiquement une boucle For Each autour de l’action qui référence ce champ.
   > De cette façon, votre application logique peut effectuer cette action sur chaque élément du tableau.
   > Pour supprimer la boucle, supprimez le champ du tableau, déplacez l’action de référencement en dehors de la boucle, sélectionnez les points de suspension ( **...** ) dans la barre de titre de la boucle, puis **Supprimer**.

1. Enregistrez votre application logique.

À présent, testez votre application logique, qui ressemble à cet exemple :

![Application logique terminée](./media/tutorial-process-email-attachments-workflow/complete.png)

## <a name="run-your-logic-app"></a>Exécuter votre application logique

1. Envoyez-vous un e-mail qui respecte les critères suivants :

   * L’objet de votre e-mail contient le texte que vous avez spécifié dans la propriété **Filtre d’objet** du déclencheur : `Business Analyst 2 #423501`

   * Votre e-mail contient une ou plusieurs pièces jointes. Vous pouvez réutiliser un fichier texte vide de votre test précédent. Pour un scénario plus réaliste, joignez un fichier de CV.

   * Le corps de l’e-mail contient le texte suivant que vous pouvez copier et coller :

     ```text

     Name: Jamal Hartnett

     Street address: 12345 Anywhere Road

     City: Any Town

     State or Country: Any State

     Postal code: 00000

     Email address: jamhartnett@outlook.com

     Phone number: 000-000-0000

     Position: Business Analyst 2 #423501

     Technical skills: Dynamics CRM, MySQL, Microsoft SQL Server, JavaScript, Perl, Power BI, Tableau, Microsoft Office: Excel, Visio, Word, PowerPoint, SharePoint, and Outlook

     Professional skills: Data, process, workflow, statistics, risk analysis, modeling; technical writing, expert communicator and presenter, logical and analytical thinker, team builder, mediator, negotiator, self-starter, self-managing  

     Certifications: Six Sigma Green Belt, Lean Project Management

     Language skills: English, Mandarin, Spanish

     Education: Master of Business Administration
     ```

1. Exécutez votre application logique. En cas de réussite, votre application logique vous envoie un e-mail qui ressemble à l’exemple suivant :

   ![Notification par e-mail envoyée par l’application logique](./media/tutorial-process-email-attachments-workflow/email-notification.png)

   Si vous ne recevez pas d’e-mail, vérifiez le dossier Courrier indésirable de votre messagerie. Il se peut que le filtre de courrier indésirable redirige ces types d’e-mails. Sinon, si vous ne savez pas si votre application logique s’est correctement exécutée, consultez [Dépanner votre application logique](../logic-apps/logic-apps-diagnosing-failures.md).

Félicitations ! Vous avez maintenant créé et exécuté une application logique qui automatise les tâches dans différents services Azure et appelle un code personnalisé.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources qui contient votre application logique et les ressources associées.

1. Dans la zone de recherche Azure du haut, entrez `resources groups`, puis sélectionnez **Groupes de ressources**.

   ![Rechercher et sélectionner « Groupes de ressources »](./media/tutorial-process-email-attachments-workflow/find-azure-resource-groups.png)

1. Dans la liste des **Groupes de ressources**, sélectionnez le groupe de ressources pour ce tutoriel. 

   ![Rechercher le groupe de ressources pour le tutoriel](./media/tutorial-process-email-attachments-workflow/find-select-tutorial-resource-group.png)

1. Dans le volet **Vue d’ensemble**, sélectionnez **Supprimer un groupe de ressources**.

   ![Supprimer le groupe de ressources de l’application logique](./media/tutorial-process-email-attachments-workflow/delete-resource-group.png)

1. Dans le volet de confirmation, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une application logique qui traite et stocke les pièces jointes en intégrant des services Azure, tels que Stockage Azure et Azure Functions. À présent, découvrez d’autres connecteurs que vous pouvez utiliser pour créer des applications logiques.

> [!div class="nextstepaction"]
> [En savoir plus sur les connecteurs de Logic Apps](../connectors/apis-list.md)