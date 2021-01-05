---
title: Gérer les métadonnées d’artefact de compte d’intégration
description: Ajouter et obtenir des métadonnées d’artefact à partir de comptes d’intégration dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/17/2019
ms.openlocfilehash: bc119f1ce8efb821781dabfb9dd259cc5c8d9c23
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74792469"
---
# <a name="manage-artifact-metadata-in-integration-accounts-with-azure-logic-apps-and-enterprise-integration-pack"></a>Gérer des métadonnées d’artefact à partir de comptes d’intégration avec Azure Logic Apps et Enterprise Integration Pack

Vous pouvez définir des métadonnées personnalisées pour les artefacts dans les comptes d’intégration et obtenir ces métadonnées pendant l’exécution pour votre application logique à utiliser. Par exemple, vous pouvez fournir des métadonnées pour les artefacts tels que des partenaires, des contrats, des schémas et des cartes ; tous stockent les métadonnées à l’aide de paires clé-valeur. 

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez <a href="https://azure.microsoft.com/free/" target="_blank">vous inscrire pour obtenir un compte Azure gratuitement</a>.

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) de base qui a les artefacts à l’emplacement où vous voulez ajouter des métadonnées ; par exemple : 

  * [Partenaire](logic-apps-enterprise-integration-partners.md)
  * [Contrat](logic-apps-enterprise-integration-agreements.md)
  * [Schéma](logic-apps-enterprise-integration-schemas.md)
  * [Map](logic-apps-enterprise-integration-maps.md)

* Une application logique liée aux métadonnées d’artefact et de compte d’intégration que vous voulez utiliser. Si votre application logique n’est pas déjà liée, découvrez [comment lier des applications logiques à des comptes d’intégration](logic-apps-enterprise-integration-create-integration-account.md#link-account). 

  Si vous n’avez pas encore d’application logique, découvrez [comment créer des applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md). 
  Ajoutez le déclencheur et les actions que vous voulez utiliser pour la gestion des métadonnées d’artefact. Ou, pour simplement faire des tests, ajoutez un déclencheur comme **Request** ou **HTTP** à votre application logique.

## <a name="add-metadata-to-artifacts"></a>Ajouter des métadonnées à des artefacts

1. Connectez-vous au <a href="https://portal.azure.com" target="_blank">portail Azure</a> avec les informations d’identification de votre compte Azure. Recherchez et ouvrez votre compte d’intégration.

1. Sélectionnez l’artefact où vous voulez ajouter des métadonnées, puis choisissez **Modifier**. Entrez les détails des métadonnées de cet artefact ; par exemple :

   ![Saisie des métadonnées](media/logic-apps-enterprise-integration-metadata/add-partner-metadata.png)

1. Une fois que vous avez terminé, sélectionnez **OK**.

1. Pour afficher ces métadonnées dans la définition JSON (JavaScript Objet Notation) pour le compte d’intégration, choisissez **Modifier au format JSON** afin que l’éditeur JSON s’ouvre : 

   ![JSON pour les métadonnées du partenaire](media/logic-apps-enterprise-integration-metadata/partner-metadata.png)

## <a name="get-artifact-metadata"></a>Obtenir des métadonnées d’artefact

1. Dans le portail Azure, ouvrez l’application logique liée au compte d’intégration souhaité. 

1. Dans le Concepteur d’application logique, si vous ajoutez l’étape d’obtention de métadonnées sous le déclencheur ou la dernière action du workflow, choisissez **Nouvelle étape** > **Ajouter une action**. 

1. Dans la zone de recherche, entrez « compte d’intégration ». Sous la zone de recherche, choisissez **Tout**. Dans la liste des actions, sélectionnez cette action : **Recherche d’artefact de compte d’intégration - Compte d’intégration**

   ![Sélectionner « Recherche d’artefact de compte d’intégration »](media/logic-apps-enterprise-integration-metadata/integration-account-artifact-lookup.png)

1. Fournissez les informations suivantes pour l’artefact que vous voulez rechercher :

   | Propriété | Obligatoire | Valeur | Description | 
   |----------|---------|-------|-------------| 
   | **Type d’artefact** | Oui | **Schéma**, **Carte**, **Partenaire**, **Contrat** ou un type personnalisé | Le type souhaité pour l’artefact | 
   | **Nom de l’artefact** | Oui | <*nom_artefact*> | Nom souhaité pour l’artefact | 
   ||| 

   Par exemple, supposons que vous voulez obtenir les métadonnées d’un artefact de partenaire commercial :

   ![Sélectionner le type d’artefact et entrer son nom](media/logic-apps-enterprise-integration-metadata/artifact-lookup-information.png)

1. Ajoutez l’action souhaitée pour la gestion de ces métadonnées ; par exemple :

   1. Sous l’action **Recherche d’artefact de compte d’intégration** , choisissez **Étape suivante**, puis sélectionnez **Ajouter une action**. 

   1. Dans la zone de recherche, entrez « http ». Sous la zone de recherche, choisissez **Éléments intégrés**, puis sélectionnez cette action : **HTTP - HTTP**

      ![Ajouter une action HTTP](media/logic-apps-enterprise-integration-metadata/http-action.png)

   1. Fournissez les informations relatives au métadonnées d’artefact que vous voulez gérer. 

      Par exemple, supposons que vous voulez obtenir les métadonnées `routingUrl` qui ont été ajoutées précédemment dans cette rubrique. Voici les valeurs de propriété que vous pouvez spécifier : 

      | Propriété | Obligatoire | Valeur | Description | 
      |----------|----------|-------|-------------| 
      | **Méthode** | Oui | <*operation-to-run*> | Opération HTTP à exécuter sur l’artefact. Par exemple, cette action HTTP utilise la méthode **GET**. | 
      | **URI** | Oui | <*metadata-location*> | Pour accéder à la valeur de métadonnées `routingUrl` à partir de l’artefact que vous avez récupéré, vous pouvez utiliser une expression ; par exemple : <p>`@{outputs('Integration_Account_Artifact_Lookup')['properties']['metadata']['routingUrl']}` | 
      | **En-têtes** | Non | <*header-values*> | Toute sortie d’en-tête du déclencheur que vous voulez passer dans l’action HTTP. Par exemple, pour passer la valeur de propriété `headers` du déclencheur, vous pouvez utiliser une expression ; par exemple : <p>`@triggeroutputs()['headers']` | 
      | **Corps** | Non | <*body-content*> | Tout autre contenu que vous voulez passer par le biais de la propriété `body` de l’action HTTP. Cet exemple passe les valeurs `properties` de l’artefact dans l’action HTTP : <p>1. Cliquez dans la propriété **Body** pour afficher la liste de contenu dynamique. Si aucune propriété ne s’affiche, choisissez **Afficher plus**. <br>2. Dans la liste de contenu dynamique, sous **Recherche d’artefact de compte d’intégration**, sélectionnez **Propriétés**. | 
      |||| 

      Par exemple :

      ![Spécifier des valeurs et des expressions pour l’action HTTP](media/logic-apps-enterprise-integration-metadata/add-http-action-values.png)

   1. Pour vérifier les informations que vous avez fournies pour l’action HTTP, affichez la définition JSON de votre application logique. Dans la barre d’outils du Concepteur d’application logique, choisissez **Mode Code** afin que la définition JSON de l’application s’affiche ; par exemple :

      ![Définition JSON de l’application logique](media/logic-apps-enterprise-integration-metadata/finished-logic-app-definition.png)

      Quand vous revenez dans le Concepteur d’application logique, toutes les expressions que vous avez utilisées apparaissent alors résolues ; par exemple :

      ![Expressions résolues dans le Concepteur d’application logique](media/logic-apps-enterprise-integration-metadata/resolved-expressions.png)

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les contrats](logic-apps-enterprise-integration-agreements.md)
