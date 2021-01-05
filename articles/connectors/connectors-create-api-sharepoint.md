---
title: Se connecter à SharePoint à partir d’Azure Logic Apps
description: Automatiser les tâches et les flux de travail qui supervisent et gèrent des ressources dans SharePoint Online ou SharePoint Server en local à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: c72330792e508361830c1bf391f85eefe78bdd1e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87283977"
---
# <a name="monitor-and-manage-sharepoint-resources-with-azure-logic-apps"></a>Superviser et gérer les ressources SharePoint avec Azure Logic Apps

Avec Azure Logic Apps et le connecteur SharePoint, vous pouvez créer des tâches et des flux de travail automatisés qui supervisent et gèrent des ressources, telles que des fichiers, dossiers, listes, éléments ou personnes, dans SharePoint Online ou SharePoint Server en local, par exemple :

* Superviser à quel moment les fichiers ou éléments sont créés, changés ou supprimés.
* Créer, obtenir, mettre à jour ou supprimer des éléments.
* Ajouter, obtenir ou supprimer des pièces jointes. Obtenir le contenu de pièces jointes.
* Créer, copier, mettre à jour ou supprimer des fichiers. 
* Mettre à jour les propriétés de fichier. Obtenir le contenu, les métadonnées ou les propriétés d’un fichier.
* Lister ou extraire des dossiers.
* Obtenir des listes ou des vues de liste.
* Définir le statut d’approbation du contenu.
* Résoudre des personnes.
* Envoyer des requêtes HTTP à SharePoint.
* Obtenir les valeurs d’une entité.

Vous pouvez utiliser des déclencheurs qui obtiennent des réponses de SharePoint et mettent la sortie à la disposition d’autres actions. Vous pouvez utiliser des actions dans vos applications logiques pour effectuer des tâches dans SharePoint. Vous pouvez également faire en sorte que des actions utilisent la sortie d’actions SharePoint. Par exemple, si vous récupérez régulièrement des fichiers de SharePoint, vous pouvez envoyer des messages à votre équipe à l’aide du connecteur Slack.
Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/). 

* Vos informations d’identification utilisateur et adresse du site SharePoint

  Vos informations d’identification autorisent votre application logique à créer une connexion et à accéder à votre compte SharePoint. 

* Pour pouvoir connecter les applications logiques aux systèmes locaux tels que SQL Server, vous devez [installer et configurer une passerelle de données locale](../logic-apps/logic-apps-gateway-install.md). De cette façon, vous pouvez spécifier que soit utilisée votre installation de passerelle quand vous créez la connexion à SharePoint Server pour votre application logique.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte SharePoint. Pour démarrer avec un déclencheur SharePoint, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser une action SharePoint, démarrez votre application logique avec un déclencheur, tel qu’un déclencheur Salesforce, si vous avez un compte Salesforce.

  Par exemple, vous pouvez démarrer votre application logique avec le déclencheur Salesforce **Lorsqu’un enregistrement est créé**. 
  Ce déclencheur se déclenche chaque fois qu’un enregistrement, tel qu’un prospect, est créé dans Salesforce. 
  Vous pouvez ensuite suivre ce déclencheur avec l’action SharePoint **Créer un fichier**. Ainsi, quand l’enregistrement est créé, votre application logique crée dans SharePoint un fichier contenant des informations sur ce nouvel enregistrement.

## <a name="connect-to-sharepoint"></a>Se connecter à SharePoint

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Pour les applications logiques vides, dans la zone de recherche, entrez « sharepoint » comme filtre. Dans la liste des déclencheurs, sélectionnez le déclencheur souhaité. 

   -ou-

   Pour les applications logiques existantes, sous la dernière étape où vous souhaitez ajouter une action SharePoint, choisissez **Nouvelle étape**. 
   Dans la zone de recherche, entrez le filtre « sharepoint ». 
   Dans la liste des actions, sélectionnez l’action souhaitée.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. 
   Cliquez sur le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Quand vous êtes invité à vous connecter, fournissez les informations de connexion nécessaires. Si vous utilisez SharePoint Server, veillez à sélectionner **Se connecter via une passerelle de données locale**. Lorsque vous êtes prêt, choisissez **Créer**.

1. Fournissez les informations nécessaires pour le déclencheur ou l’action sélectionnés et continuez à générer le flux de travail de votre application logique.

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont décrits par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence](/connectors/sharepoint/) du connecteur.

## <a name="get-support"></a>Obtenir de l’aide

* Pour toute question, consultez la [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Pour voter pour des idées de fonctionnalités ou pour en soumettre, visitez le [site de commentaires des utilisateurs Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)

