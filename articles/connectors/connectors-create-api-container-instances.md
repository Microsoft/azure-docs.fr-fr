---
title: Déployer et gérer Azure Container Instances
description: Automatiser les tâches et les workflows qui créent et gèrent les déploiements de conteneurs dans Azure Container Instances à l’aide d’Azure Logic Apps
services: logic-apps, container-instances
ms.service: logic-apps
ms.suite: integration
author: dlepow
ms.author: danlep
ms.manager: gwallace
ms.reviewer: estfan, macolso
ms.topic: article
tags: connectors
ms.date: 01/14/2020
ms.openlocfilehash: ecb1049d64197f2a60438df7eedfb244907f7327
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76046203"
---
# <a name="deploy-and-manage-azure-container-instances-by-using-azure-logic-apps"></a>Déployer et gérer Azure Container Instances à l’aide d’Azure Logic Apps

Avec Azure Logic Apps et le connecteur Azure Container Instances, vous pouvez configurer des tâches et des workflows automatisés qui déploient et gèrent des [groupes de conteneurs](../container-instances/container-instances-container-groups.md). Le connecteur Container Instance prend en charge les actions suivantes :

* Créer ou supprimer un groupe de conteneurs
* Obtenir les propriétés d’un groupe de conteneurs
* Obtenir une liste des groupes de conteneurs
* Obtenir les journaux d’une instance de conteneur

Utilisez ces actions dans vos applications logiques pour effectuer des tâches, telles que l’exécution d’une charge de travail de conteneur en réponse à un déclencheur Logic Apps. Vous pouvez également faire en sorte que des actions utilisent la sortie d’actions Container Instance. 

Ce connecteur ne fournit que des actions ; ainsi, pour démarrer votre application logique, utilisez un déclencheur distinct, tel que le déclencheur **Périodicité** pour exécuter une charge de travail de conteneur régulièrement. Ou bien, vous devrez peut-être déclencher un déploiement de groupe de conteneurs après un événement, tel que l’arrivée d’un e-mail Outlook. 

Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Conditions préalables requises

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/). 

* Des connaissances de base sur la [création des applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md) ainsi que sur la [création et la gestion des instances de conteneur](../container-instances/container-instances-quickstart.md)

* L’application logique à partir de laquelle vous souhaitez accéder à vos instances de conteneur. Pour utiliser une action, démarrez votre application logique avec un autre déclencheur, par exemple le déclencheur **Périodicité**.

## <a name="add-a-container-instance-action"></a>Ajouter une action Container Instance

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Choisissez une procédure : 

   * Sous la dernière étape où vous souhaitez ajouter une action, choisissez **Nouvelle étape**. 

     -ou-

   * Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. 
   Cliquez sur le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « container instance » comme filtre. Dans la liste des actions, sélectionnez l’action du connecteur Azure Container Instance de votre choix.

1. Donnez un nom à votre connexion. 

1. Fournissez les informations nécessaires pour votre action sélectionné et continuez à générer le flux de travail de votre application logique.

  Par exemple, sélectionnez **Créer un groupe de conteneurs** et entrez les propriétés d’un groupe de conteneurs ainsi qu’une ou plusieurs instances de conteneur dans le groupe, comme illustré dans l’image suivante (détail partiel) :

  ![Créer un groupe de conteneurs](./media/connectors-create-api-container-instances/logic-apps-aci-connector.png)

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont détaillés par la description OpenAPI (anciennement Swagger) du connecteur, consultez la [page de référence](/connectors/aci/) du connecteur ou les [Informations de référence YAML](../container-instances/container-instances-reference-yaml.md) du groupe de conteneurs.

## <a name="next-steps"></a>Étapes suivantes

* Consultez un [exemple d’application logique](https://github.com/Azure-Samples/aci-logicapps-integration) qui exécute un conteneur dans Azure Container Instances pour analyser le sentiment d’un e-mail ou d’un texte Twitter

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)