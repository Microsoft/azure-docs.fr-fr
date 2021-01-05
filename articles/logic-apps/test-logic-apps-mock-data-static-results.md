---
title: Tester des applications logiques avec des données fictives
description: Configurer les résultats statiques pour le test d’applications logiques avec des données fictives sans affecter les environnements de production
services: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 05/13/2019
ms.openlocfilehash: 711d753203aeaeba50cea692053a37fcab2e9c7b
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027701"
---
# <a name="test-logic-apps-with-mock-data-by-setting-up-static-results"></a>Tester des applications logiques avec des données fictives en configurant des résultats statiques

Lorsque vous testez vos applications logiques, vous n’êtes peut-être pas prêt à appeler des applications, services et systèmes ou à y accéder pour diverses raisons. Généralement, dans ces scénarios, vous devez exécuter différents chemins d’accès de condition, forcer des erreurs, fournir des corps de réponse de message spécifiques ou même tenter d’ignorer certaines étapes. En configurant des résultats statiques pour une action dans votre application logique, vous pouvez simuler des données de sortie pour cette action. L’activation des résultats statiques sur une action n’exécute pas l’action, mais renvoie des données fictives.

Par exemple, si vous avez configuré des résultats statiques pour l’action d’envoi d’e-mails d’Outlook 365, le moteur Logic Apps renvoie juste les données fictives que vous avez spécifiées en tant que résultats statiques, au lieu d’appeler Outlook et d’envoyer un e-mail.

## <a name="prerequisites"></a>Conditions préalables requises

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, <a href="https://azure.microsoft.com/free/" target="_blank">inscrivez-vous pour bénéficier d’un compte Azure gratuit</a>.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique dans laquelle vous souhaitez configurer des résultats statiques

<a name="set-up-static-results"></a>

## <a name="set-up-static-results"></a>Configurer des résultats statiques

1. Si ce n’est pas déjà fait, dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur Logic Apps.

1. Procédez comme suit pour l’action pour laquelle vous souhaitez configurer des résultats statiques : 

   1. En haut à droite de l’action, choisissez le bouton représentant des points de suspension ( *...* ), puis sélectionnez **Résultat statique** , par exemple :

      ![Sélectionner « Résultat statique » > « Activer le résultat statique »](./media/test-logic-apps-mock-data-static-results/select-static-result.png)

   1. Choisissez **Activer le résultat statique**. Pour les propriétés requises (*), spécifiez les valeurs de sortie fictives que vous souhaitez en réponse à l’action.

      Par exemple, voici les propriétés requises pour l’action HTTP :

      | Propriété | Description |
      |----------|-------------|
      | **État** | État de l’action à renvoyer |
      | **Code d’état** | Code d’état spécifique à renvoyer |
      | **En-têtes** | Contenu d’en-tête à renvoyer |
      |||

      ![Sélectionnez « Activer le résultat statique »](./media/test-logic-apps-mock-data-static-results/enable-static-result.png)

      Pour entrer des données fictives au format JavaScript Objet Notation (JSON), choisissez **Basculer en mode JSON** (![choisissez « Basculer en mode JSON »](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button.png)).

   1. Pour les propriétés facultatives, ouvrez la liste **Sélectionner des champs facultatifs** , puis sélectionnez les propriétés que vous souhaitez simuler.

      ![Sélectionner des propriétés facultatives](./media/test-logic-apps-mock-data-static-results/optional-properties.png)

1. Quand vous êtes prêt à enregistrer votre travail, choisissez **Terminé**.

   En haut à droite de l’action, la barre de titre affiche maintenant une icône de bécher test (![Icône pour résultats statiques](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)), ce qui indique que vous avez activé les résultats statiques.

   ![Icône montrant l’activation des résultats statiques](./media/test-logic-apps-mock-data-static-results/static-results-enabled.png)

   Pour rechercher les exécutions précédentes qui utilisent des données fictives, consultez la section [Rechercher les exécutions qui utilisent des résultats statiques](#find-runs-mock-data), plus loin dans cette rubrique.

<a name="reuse-sample-outputs"></a>

## <a name="reuse-previous-outputs"></a>Réutiliser des sorties précédentes

Si votre application logique a une exécution précédente avec des sorties, vous pouvez les réutiliser en tant que sorties fictives en copiant et collant les sorties de cette exécution.

1. Si ce n’est pas déjà fait, dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur Logic Apps.

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**.

1. Dans la section **Historique des exécutions** , sélectionnez l’application logique souhaitée.

1. Dans le flux de travail de votre application logique, recherchez et développez l’action qui a les sorties souhaitées.

1. Choisissez le lien **Afficher les sorties brutes**.

1. Copiez l’objet JavaScript Objet Notation (JSON) complet ou la sous-section spécifique que vous souhaitez utiliser, par exemple, la section des sorties ou même simplement la section des en-têtes.

1. Suivez les étapes pour ouvrir la section **Résultat statique** de votre action dans [Configurer des résultats statiques](#set-up-static-results).

1. Une fois la section **Résultat statique** ouverte, choisissez de procéder à l’une des opérations suivantes :

   * Pour coller un objet JSON complet, choisissez **Basculer en mode JSON** (![choisissez « Basculer en mode JSON »](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button.png)) :

     ![Choisir « Basculer en mode JSON » pour l’objet complet](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button-complete.png)

   * Pour coller uniquement une section JSON, en regard de l’étiquette de cette section, sélectionnez **Basculer en mode JSON** pour cette section, par exemple :

     ![Choisir « Basculer en mode JSON » pour les sorties](./media/test-logic-apps-mock-data-static-results/switch-to-json-mode-button-outputs.png)

1. Dans l’éditeur JSON, collez votre élément JSON copié précédemment.

   ![Mode JSON](./media/test-logic-apps-mock-data-static-results/json-editing-mode.png)

1. Quand vous avez fini, choisissez **Terminé**. Ou, pour revenir au concepteur, choisissez **Basculer en mode Éditeur** (![choisissez « Basculer en mode Éditeur »](./media/test-logic-apps-mock-data-static-results/switch-editor-mode-button.png)).

<a name="find-runs-mock-data"></a>

## <a name="find-runs-that-use-static-results"></a>Rechercher les exécutions qui utilisent des résultats statiques

L’historique des exécutions de votre application logique identifie les exécutions où les actions utilisent des résultats statiques. Pour rechercher ces exécutions, procédez comme suit :

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. 

1. Dans le volet droit, sous **Historique des exécutions** , recherchez la colonne **Résultats statiques**. 

   Toute exécution qui inclut des actions avec des résultats a la colonne **Résultats statiques** définie sur **Activée** , par exemple :

   ![Historique des exécutions - colonne de résultats statiques](./media/test-logic-apps-mock-data-static-results/run-history.png)

1. Pour afficher les actions qui utilisent des résultats statiques, sélectionnez l’exécution pour laquelle vous souhaitez que la colonne **Résultats statiques** soit définie sur **Activée**.

   Les actions qui utilisent les résultats statiques affichent un bécher test (![Icône pour résultats statiques](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)), par exemple :

   ![Historique des exécutions - actions qui utilisent les résultats statiques](./media/test-logic-apps-mock-data-static-results/static-results-enabled-run-details.png)

## <a name="disable-static-results"></a>Désactiver les résultats statiques

La désactivation des résultats statiques ne supprime pas les valeurs de votre dernière configuration. Par conséquent, la prochaine fois que vous activerez les résultats statiques, vous retrouverez vos valeurs précédentes.

1. Recherchez l’action pour laquelle vous souhaitez désactiver les sorties statiques. En haut à droite de l’action, cliquez sur l’icône de bécher test (![Icône pour résultats statiques](./media/test-logic-apps-mock-data-static-results/static-results-test-beaker-icon.png)).

   ![Capture d’écran montrant une action HTTP dans laquelle vous pouvez sélectionner l’icône en forme de bécher de laboratoire.](./media/test-logic-apps-mock-data-static-results/disable-static-results.png)

1. Choisissez **Désactiver le résultat statique** > **Terminé**.

   ![Capture d’écran montrant l’option Désactiver le résultat statique que vous pouvez sélectionner.](./media/test-logic-apps-mock-data-static-results/disable-static-results-button.png)

## <a name="reference"></a>Informations de référence

Pour plus d’informations sur ce paramètre dans vos définitions de flux de travail sous-jacent, consultez [Schema reference for Workflow Definition Language in Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md#static-results) (Référence de schéma du langage de définition de flux de travail dans Azure Logic Apps) et [runtimeConfiguration.staticResult - Runtime configuration settings](../logic-apps/logic-apps-workflow-actions-triggers.md#runtime-configuration-settings) (runtimeConfiguration.staticResult - paramètres de configuration du runtime)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Azure Logic Apps](../logic-apps/logic-apps-overview.md)