---
title: Connexion à Outlook.com
description: Automatiser les tâches et les workflows qui gèrent les e-mails, les calendriers et les contacts dans Outlook.com à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/18/2016
tags: connectors
ms.openlocfilehash: 8d3b180b6f1e9dc4ec4b09dd81786cc81e8588da
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75707184"
---
# <a name="manage-email-calendars-and-contacts-in-outlookcom-by-using-azure-logic-apps"></a>Gérer les e-mails, les calendriers et les contacts dans Outlook.com à l’aide d’Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le [connecteur Outlook.com](/connectors/outlook/), vous pouvez créer des tâches et workflows automatisés qui gèrent votre compte @outlook.com ou @hotmail.com en générant des applications logiques. Par exemple, vous automatisez les tâches suivantes :

* Recevoir, envoyer et répondre à des e-mails
* Planifier des réunions sur votre calendrier
* Ajouter et modifier des contacts.

Vous pouvez utiliser n’importe quel déclencheur pour démarrer votre workflow, par exemple, lorsqu’un nouvel e-mail arrive, lorsqu’un élément de calendrier est mis à jour ou lorsqu’un événement se produit dans un autre service. Vous pouvez utiliser des actions qui répondent à l’événement déclencheur, par exemple, envoyer un e-mail ou créer un événement de calendrier.

> [!NOTE]
> Pour automatiser les tâches d’un compte professionnel Microsoft, par exemple @fabrikam.onmicrosoft.com, utilisez le [connecteur Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md).

## <a name="prerequisites"></a>Prérequis

* Un [compte Outlook.com](https://outlook.live.com/owa/)

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/). 

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte Outlook.com. Pour démarrer votre workflow avec un déclencheur Outlook.com, vous avez besoin d’une [application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour ajouter une action Outlook.com à votre workflow, votre application logique doit déjà disposer d’un déclencheur.

## <a name="add-a-trigger"></a>Ajouter un déclencheur

Un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) désigne un événement qui démarre le workflow dans votre application logique. Cet exemple d’application logique utilise un déclencheur d’interrogation qui recherche tout nouvel e-mail dans votre compte de messagerie, en fonction de l’intervalle et de la fréquence spécifiés.

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez votre application logique vide dans le Concepteur d’applications logiques.

1. Dans la zone de recherche, entrez « outlook.com » comme filtre. Dans cet exemple, sélectionnez **À la réception d’un e-mail**.

1. Si vous êtes invité à vous connecter, indiquez vos informations d’identification Outlook.com, afin que votre application logique puisse se connecter à votre compte. Ou bien, si votre connexion existe déjà, fournissez les informations pour les propriétés du déclencheur :

1. Dans le déclencheur, définissez les valeurs **Fréquence** et **Intervalle**.

   Par exemple, si vous souhaitez que le déclencheur interroge le service toutes les 15 minutes, définissez le champ **Fréquence** sur **Minute**, et le champ **Intervalle** sur **15**.

1. Dans la barre d’outils du concepteur, sélectionnez **Enregistrer**, ce qui permet d’enregistrer votre application logique.

Pour répondre au déclencheur, ajoutez une autre action. Par exemple, vous pouvez ajouter l’action Twilio **Envoyer un message**, qui envoie un texte à la réception d’un e-mail.

## <a name="add-an-action"></a>Ajouter une action

Une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) est une opération qui est exécutée par le workflow dans votre application logique. Cet exemple d’application logique envoie un e-mail à partir de votre compte Outlook.com. Vous pouvez utiliser la sortie d’un autre déclencheur pour remplir l’action. Supposons, par exemple, que votre application logique utilise le déclencheur SalesForce **Quand un objet est créé**. Vous pouvez ajouter l’action Outlook.com **Envoyer un e-mail** et utiliser les sorties du déclencheur SalesForce dans l’e-mail.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Pour ajouter une action comme dernière étape à votre workflow, sélectionnez **Nouvelle étape**. 

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « outlook.com » comme filtre. Pour cet exemple, sélectionnez **Envoyer un e-mail**. 

1. Si vous êtes invité à vous connecter, indiquez vos informations d’identification Outlook.com, afin que votre application logique puisse se connecter à votre compte. Ou bien, si votre connexion existe déjà, fournissez les informations pour les propriétés de l’action.

1. Dans la barre d’outils du concepteur, sélectionnez **Enregistrer**, ce qui permet d’enregistrer votre application logique.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques, telles que les déclencheurs, actions et limites, comme décrit dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/outlook/). 

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
