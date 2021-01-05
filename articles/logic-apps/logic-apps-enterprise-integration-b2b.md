---
title: Échanger des messages pour les scénarios d’intégration d’entreprise B2B
description: Échanger des messages B2B entre partenaires commerciaux dans Azure Logic Apps à l’aide de l’Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 10/02/2020
ms.openlocfilehash: e16cc8934407a5c54c84fd045c99e28116e656c9
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93310497"
---
# <a name="receive-and-confirm--b2b-as2-messages-by-using-azure-logic-apps-and-enterprise-integration-pack"></a>Échanger et confirmer des messages B2B AS2 avec Azure Logic Apps et Enterprise Integration Pack

Si vous disposez d’un compte d’intégration définissant des partenaires et contrats commerciaux, vous pouvez créer un flux de travail B2B automatisé pour l’échange de messages entre partenaires commerciaux en utilisant [Azure Logic Apps](../logic-apps/logic-apps-overview.md) avec l’[Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md). Azure Logic Apps fonctionne avec les connecteurs qui prennent en charge les protocoles standard AS2, X12, EDIFACT et RosettaNet. Vous pouvez également combiner ces connecteurs avec d’autres [connecteurs disponibles dans Logic Apps](../connectors/apis-list.md), tels que Salesforce et Office 365 Outlook.

Cet article explique comment créer une application logique qui reçoit une requête HTTP à l’aide d’un déclencheur de requête, décode le contenu du message à l’aide des actions AS2 et X12, puis renvoie une réponse à l’aide de l’action Réponse.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Une application logique vide pour vous permettre de créer le flux de travail B2B à l’aide du déclencheur [Demande](../connectors/connectors-native-reqres.md) qui est suivi des actions suivantes :

  * [Décodage AS2](../logic-apps/logic-apps-enterprise-integration-as2.md)

  * [Condition](../logic-apps/logic-apps-control-flow-conditional-statement.md) qui envoie une [Réponse](../connectors/connectors-native-reqres.md) en fonction de la réussite ou de l’échec de l’action de décodage AS2

  * [Décoder le message X12](../logic-apps/logic-apps-enterprise-integration-x12.md) 

  Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [compte d’intégration](./logic-apps-enterprise-integration-create-integration-account.md) associé à votre abonnement Azure et lié à votre application logique. Votre application logique et votre compte d’intégration doivent tous deux exister dans le même emplacement ou dans la même région Azure.

* Au moins deux [partenaires commerciaux](../logic-apps/logic-apps-enterprise-integration-partners.md) que vous avez déjà définis dans votre compte d’intégration, ainsi que des [contrats AS2 et X12](logic-apps-enterprise-integration-agreements.md) pour ces partenaires.

## <a name="add-the-request-trigger"></a>Ajout du déclencheur Request

Cet exemple utilise le Concepteur d’application logique dans le portail Azure, mais vous pouvez suivre des étapes similaires pour le Concepteur d’application logique dans Visual Studio.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique vide dans le Concepteur d’application logique.

1. Dans la zone de recherche, entrez `when a http request`, puis sélectionnez **Lors de la réception d’une requête HTTP** à utiliser comme déclencheur.

   ![Sélectionner un déclencheur Demande pour démarrer le flux de travail de votre application logique](./media/logic-apps-enterprise-integration-b2b/select-http-request-trigger.png)

1. Laissez la zone **Schéma JSON du corps de la requête vide** , car le message X12 est un fichier plat.

   ![Laisser la zone « Schéma JSON du corps de la requête » vide](./media/logic-apps-enterprise-integration-b2b/receive-trigger-message-body-json-schema.png)

1. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

   Cette étape génère l’ **URL HTTP POST** à utiliser pour envoyer la requête qui déclenche l’application logique. Pour copier cette URL, sélectionnez l’icône de copie en regard de l’URL.

   ![URL générée pour le déclencheur de requête pour la réception d’appels](./media/logic-apps-enterprise-integration-b2b/generated-url-request-trigger.png)

## <a name="add-the-as2-decode-action"></a>Ajouter l’action de décodage AS2

Ajoutez maintenant les actions B2B que vous souhaitez utiliser. Cet exemple utilise des actions AS2 et X12.

1. Sous le déclencheur, sélectionnez **Nouvelle étape**. Pour masquer les détails du déclencheur, cliquez sur la barre de titre du déclencheur.

   ![Ajouter une autre étape au flux de travail de votre application logique](./media/logic-apps-enterprise-integration-b2b/add-new-action-under-trigger.png)

1. Sous **Choisir une action** , dans la zone de recherche, entrez `as2 decode`, puis sélectionnez **Décodage AS2 (v2)** .

   ![Rechercher et sélectionner « Décodage AS2 (v2) »](./media/logic-apps-enterprise-integration-b2b/add-as2-decode-action.png)

1. Pour la propriété **Message à décoder** , indiquez l’entrée que l’action AS2 doit décode, qui est le contenu `body` que le déclencheur de requête HTTP reçoit. Vous avez plusieurs moyens de spécifier ce contenu comme entrée, soit à partir de la liste de contenu dynamique, soit en tant qu’expression :

   * Pour opérer une sélection dans une liste des sorties de déclencheur disponibles, cliquez à l’intérieur de la zone **Message à décoder**. Lorsque la liste de contenu dynamique s’affiche, sous **Lors de la réception d’une requête HTTP** , sélectionnez la valeur de propriété **Corps** , par exemple :

     ![Sélectionner la valeur « Corps » à partir du déclencheur](./media/logic-apps-enterprise-integration-b2b/select-body-content-from-trigger.png)

   * Pour entrer une expression qui fait référence à la sortie `body` du déclencheur, cliquez dans la zone **Message à décoder**. Lorsque la liste de contenu dynamique s’affiche, sélectionnez **Expression**. Dans l’éditeur d’expression, entrez l’expression ici, puis sélectionnez **OK**  :

     `triggerOutputs()['body']`

     Ou bien, dans la zone **Message à décoder** , entrez directement l’expression suivante :

     `@triggerBody()`

     L’expression est résolue en jeton **Corps**.

     ![Sortie Corps résolue à partir du déclencheur](./media/logic-apps-enterprise-integration-b2b/resolved-trigger-outputs-body-expression.png)

1. Pour la propriété **En-têtes de message** , entrez les en-têtes requis pour l’action AS2, qui sont décrits par le contenu de `headers` reçu par le déclencheur de requête HTTP.

   1. Pour entrer une expression qui fait référence à la sortie `headers` du déclencheur, sélectionnez **Basculer les en-têtes de message en mode texte**.

      ![Capture d’écran montrant la sélection de l’option « Basculer les en-têtes de message en mode texte ».](./media/logic-apps-enterprise-integration-b2b/as2-decode-switch-text-mode.png)

   1. Cliquez dans la zone **En-têtes de message**. Lorsque la liste de contenu dynamique s’affiche, sélectionnez **Expression**. Dans l’éditeur d’expression, entrez l’expression ici, puis sélectionnez **OK**  :

      `triggerOutputs()['Headers']`

      Dans l’action de décodage AS2, l’expression apparaît maintenant en tant que jeton :

      ![Capture d’écran montrant le jeton « @triggerOutputs()['Headers'] » dans la zone « En-têtes de message ».](./media/logic-apps-enterprise-integration-b2b/as2-decode-message-header-expression.png)

   1. Pour obtenir le jeton d’expression à résoudre dans le jeton **En-têtes** , basculez entre le concepteur et le mode Code. Après cette étape, l’action de décodage AS2 ressemble à cet exemple :

      ![Sortie d’en-têtes résolus à partir du déclencheur](./media/logic-apps-enterprise-integration-b2b/resolved-trigger-outputs-headers-expression.png)

## <a name="add-response-action-for-message-receipt-notification"></a>Ajouter une action Réponse pour une notification de réception de message

Pour informer le partenaire commercial de la réception du message, vous pouvez retourner une réponse contenant une notification d’état du message (MDN) AS2 à l’aide de l’action **Réponse**. En ajoutant cette action immédiatement après l’action **Décodage AS2** , si cette action échoue, l’application logique ne continue pas le traitement.

1. Sous l’action **Décodage AS2** , sélectionnez **Nouvelle étape**.

1. Sous **Choisir une action** , dans la zone de recherche, sélectionnez **Intégrée**. Dans la zone de recherche, entrez `condition`. Dans la liste **Actions** , sélectionnez **Condition**.

   ![Ajouter l’action « Condition »](./media/logic-apps-enterprise-integration-b2b/add-condition-action.png)

   La forme de condition s’affiche, qui inclut les chemins d’accès indiquant si la condition est remplie ou non.

   ![Capture d'écran représentant la forme de condition, dans laquelle les chemins d'accès sont vides.](./media/logic-apps-enterprise-integration-b2b/added-condition-action.png)

1. Spécifiez à présent la condition à évaluer. Dans la zone **Choisir une valeur** , entrez l’expression suivante :

   `@body('AS2_Decode')?['AS2Message']?['MdnExpected']`

   Dans la zone centrale, assurez-vous que l’opération de comparaison est définie sur `is equal to`. Dans la zone de droite, entrez la valeur `Expected`. Pour obtenir l’expression à résoudre en ce jeton, basculez entre le concepteur et le mode Code.

   ![Capture d'écran représentant la forme de condition, dans laquelle une condition a été ajoutée.](./media/logic-apps-enterprise-integration-b2b/expression-for-evaluating-condition.png)

1. Maintenant, spécifiez les réponses à retourner selon que l’action **Décodage AS2** réussi ou non.

   1. Pour le cas où l’action **Décodage AS2** réussit, dans la forme **Si vrai** , sélectionnez **Ajouter une action**. Sous **Choisir une action** , dans la zone de recherche, entrez `response`, puis sélectionnez **Réponse**.

      ![Rechercher et sélectionner l’action « Réponse »](./media/logic-apps-enterprise-integration-b2b/select-http-response-action.png)

   1. Pour accéder au MDN AS2 à partir de la sortie de l’action **Décodage AS2** , spécifiez les expressions suivantes :

      * Dans la propriété **En-têtes** de l’action **Réponse** , entrez l’expression suivante :

        `@body('AS2_Decode')?['OutgoingMdn']?['OutboundHeaders']`

      * Dans la propriété **Corps** de l’action **Réponse** , entrez l’expression suivante :

        `@body('AS2_Decode')?['OutgoingMdn']?['Content']`

   1. Pour obtenir les expressions à résoudre en tant que jetons, basculez entre le concepteur et le mode Code :

      ![Expression résolue pour accéder à MDN AS2](./media/logic-apps-enterprise-integration-b2b/response-action-success-resolved-expression.png)

   1. Dans le cas où l’action **Décodage AS2** échoue, dans la forme **Si faux** , sélectionnez **Ajouter une action**. Sous **Choisir une action** , dans la zone de recherche, entrez `response`, puis sélectionnez **Réponse**. Configurez l’action **Réponse** pour retourner l’état et l’erreur de votre choix.

1. Enregistrez votre application logique.

## <a name="add-decode-x12-message-action"></a>Ajouter l’action Décoder le message X12

1. Ajoutez maintenant l’action **Décoder le message X12**. Sous l’action **Réponse** , sélectionnez **Ajouter une action**.

1. Sous **Choisir une action** , dans la zone de recherche, entrez `x12 decode`, puis sélectionnez **Décoder le message X12**.

   ![Rechercher et sélectionner l’action « Décoder le message X12 »](./media/logic-apps-enterprise-integration-b2b/add-x12-decode-action.png)

1. Si l’action X12 vous demande les informations de connexion, indiquez le nom de la connexion, sélectionnez le compte d’intégration à utiliser, puis sélectionnez **Créer**.

   ![Créer une connexion X12 à un compte d’intégration](./media/logic-apps-enterprise-integration-b2b/create-x12-integration-account-connection.png)

1. Spécifiez à présent l’entrée pour l’action X12. Cet exemple utilise la sortie de l’action AS2, qui est le contenu du message, mais ce contenu est au format d’objet JSON et encodé en base64. Par conséquent, vous devez convertir ce contenu en chaîne.

   Dans la zone **Message de fichier plat X12 à décoder** , entrez l’expression suivante pour convertir la sortie AS2 :

   `@base64ToString(body('AS2_Decode')?['AS2Message']?['Content'])`

    Pour obtenir l’expression à résoudre en ce jeton, basculez entre le concepteur et le mode Code.

    ![Convertir du contenu encodé en base64 en chaîne](./media/logic-apps-enterprise-integration-b2b/x12-decode-message-content.png)

1. Enregistrez votre application logique.

   Si vous avez besoin d’étapes supplémentaires pour cette application logique, par exemple, pour décoder le contenu du message et sortir ce contenu au format d’objet JSON, continuez à créer votre application logique.

Vous avez maintenant terminé la configuration de votre application logique B2B. Dans une application réelle, vous pourriez stocker les données X12 décodées dans une application métier ou un magasin de données. Par exemple, consultez les articles suivants :

* [Se connecter aux systèmes SAP à partir d’Azure Logic Apps](../logic-apps/logic-apps-using-sap-connector.md)
* [Superviser, créer et gérer des fichiers SFTP à l’aide de SSH et d’Azure Logic Apps](../connectors/connectors-sftp-ssh.md)

Pour vous connecter à vos propres applications métier et utiliser ces API dans votre application logique, vous pouvez ajouter des actions ou [écrire des API personnalisées](../logic-apps/logic-apps-create-api-app.md).

## <a name="next-steps"></a>Étapes suivantes

* [Recevoir des appels HTTPs entrants et y répondre](../connectors/connectors-native-reqres.md)
* [Échanger des messages AS2 pour l’intégration d’entreprise B2B](../logic-apps/logic-apps-enterprise-integration-as2.md)
* [Échanger des messages XS2 pour l’intégration d’entreprise B2B](../logic-apps/logic-apps-enterprise-integration-x12.md)
* En savoir plus sur [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
