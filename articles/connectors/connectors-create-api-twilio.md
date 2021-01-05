---
title: Se connecter à Twilio à partir d’Azure Logic Apps
description: Automatiser des tâches et des flux de travail qui gèrent les messages SMS, MMS et IP globaux via votre compte Twilio à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: d144960972f5a1b45e88cc3a0ea015925cae3b91
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87288139"
---
# <a name="manage-messages-in-twilio-with-azure-logic-apps"></a>Gérer les messages dans Twilio avec Azure Logic Apps

Avec Azure Logic Apps et le connecteur Twilio, vous pouvez créer des tâches et des flux de travail automatisés qui reçoivent, envoient et répertorient des messages dans Twilio, notamment des messages SMS, MMS et IP globaux. Vous pouvez utiliser des actions pour effectuer des tâches avec votre compte Twilio. Vous pouvez également faire en sorte que des actions utilisent le résultat d’actions Twilio. Par exemple, lorsqu’un nouveau message arrive, vous pouvez envoyer son contenu avec le connecteur Slack. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/). 

* À partir de [Twilio](https://www.twilio.com/) : 

  * Votre ID de compte Twilio et le [jeton d’authentification](https://support.twilio.com/hc/en-us/articles/223136027-Auth-Tokens-and-How-to-Change-Them), que vous pouvez trouver sur votre tableau de bord Twilio

    Vos informations d’identification autorisent votre application logique à créer une connexion et à accéder à votre compte Twilio depuis votre application logique. 
    Si vous utilisez un compte d’évaluation Twilio, vous ne pouvez envoyer de SMS qu’à des numéros de téléphone *vérifiés*.

  * Un numéro de téléphone Twilio vérifié pouvant envoyer des SMS

  * Un numéro de téléphone Twilio vérifié pouvant recevoir des SMS

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte Twilio. Pour utiliser une action Twilio, démarrez votre application logique avec un autre déclencheur, par exemple, le déclencheur **Périodicité**.

## <a name="connect-to-twilio"></a>Connectez-vous à Twilio

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Choisissez une procédure : 

     * Sous la dernière étape où vous souhaitez ajouter une action, choisissez **Nouvelle étape**. 

       -ou-

     * Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. 
     Cliquez sur le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.
     
       Dans la zone de recherche, entrez «twilio » comme filtre. 
       Dans la liste des actions, sélectionnez l’action souhaitée.

1. Fournissez les informations nécessaires pour votre connexion, puis choisissez **Créer** :

   * Nom à utiliser pour votre connexion
   * Votre ID de compte Twilio 
   * Votre jeton d’accès (d’authentification) Twilio

1. Fournissez les informations nécessaires pour votre action sélectionné et continuez à générer le flux de travail de votre application logique.

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont décrits par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence](/connectors/twilio/) du connecteur.

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des idées de fonctionnalités ou pour en soumettre, visitez le [site de commentaires des utilisateurs Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
