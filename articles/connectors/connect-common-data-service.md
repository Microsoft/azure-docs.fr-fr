---
title: Se connecter à Common Data Service
description: Créer et gérer des enregistrements Common Data Service à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jdaly, logicappspm
ms.topic: conceptual
ms.date: 05/08/2020
tags: connectors
ms.openlocfilehash: de85a61cbd699ec9ac2669f8abb6217254038de9
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91334580"
---
# <a name="create-and-manage-records-in-common-data-service-by-using-azure-logic-apps"></a>Créer et gérer des enregistrements dans Common Data Service à l’aide d’Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le [connecteur Common Data Service](/connectors/commondataservice/), vous pouvez créer des flux de travail automatisés qui gèrent les enregistrements dans votre base de données [Common Data Service](/powerapps/maker/common-data-service/data-platform-intro). Ces flux de travail peuvent créer des enregistrements, mettre à jour des enregistrements et exécuter d’autres opérations. Vous pouvez également obtenir des informations à partir de votre base de données Common Data Service et rendre la sortie disponible pour d’autres actions à utiliser dans votre application logique. Par exemple, lorsqu’un enregistrement est mis à jour dans votre base de données Common Data Service, vous pouvez envoyer un e-mail à l’aide du connecteur Office 365 Outlook.

Cet article vous explique comment générer une application logique qui crée un enregistrement de tâche quand un enregistrement de prospect est créé.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [environnement Common Data Service](/power-platform/admin/environments-overview), qui est un espace dans lequel votre organisation stocke, gère et partage des données d’entreprise et une base de données de Common Data Service. Pour plus d’informations, consultez ces ressources :<p>

  * [Découvrir : Prise en main de Common Data Service](/learn/modules/get-started-with-powerapps-common-data-service/)
  * [Power Platform - Présentation des environnements](/power-platform/admin/environments-overview)

* Connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md) et application logique à partir de laquelle vous voulez accéder aux enregistrements dans votre base de données Common Data Service. Pour démarrer votre application logique avec un déclencheur Common Data Service, vous avez besoin d’une application logique vide. Si vous débutez avec les applications logiques Azure, consultez le guide de [Démarrage rapide : Créer votre premier workflow à l’aide d’Azure Logic Apps](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-common-data-service-trigger"></a>Ajouter un déclencheur Common Data Service

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

Pour cet exemple, ajoutez le déclencheur Common Data Service qui se déclenche lors de la création d’un enregistrement.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique vide dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Dans la zone de recherche, entrez `common data service`. Pour cet exemple, dans la liste des déclencheurs, sélectionnez ce déclencheur : **Lorsqu’un enregistrement est créé**

   ![Sélectionner le déclencheur « Lorsqu'un enregistrement est créé »](./media/connect-common-data-service/select-when-record-created-trigger.png)

1. Si vous y êtes invité, connectez-vous à Common Data Service.

1. Dans le déclencheur, fournissez des informations sur l’environnement dans lequel vous souhaitez surveiller les nouveaux enregistrements de « prospects », par exemple :

   ![Informations de déclencheur pour l’environnement à surveiller](./media/connect-common-data-service/when-record-created-trigger-details.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Environment** | Oui | L’environnement à surveiller, par exemple « Production Ventes Fabrikam ». Pour plus d’informations, consultez [Power Platform - Présentation des environnements](/power-platform/admin/environments-overview). |
   | **Nom de l’entité** | Oui | L’entité à superviser, par exemple, « Leads » (Prospects) |
   | **Portée** | Oui | La source qui a créé le nouvel enregistrement, par exemple, un utilisateur de votre unité commerciale ou n’importe quel utilisateur de votre organisation. Cet exemple utilise « Business unit » (Unité commerciale). |
   ||||

## <a name="add-common-data-service-action"></a>Ajouter une action Common Data Service

Ajoutez maintenant une action Common Data Service qui crée un enregistrement de tâche pour un nouvel enregistrement « Leads ».

1. Sous le déclencheur **Lorsqu’un enregistrement est créé**, sélectionnez **Nouvelle étape**.

1. Dans la zone de recherche, entrez `common data service`. Dans la liste des actions, sélectionnez cette action : **Créer un enregistrement**

   ![Sélectionner une action « Créer un enregistrement »](./media/connect-common-data-service/select-create-new-record-action.png)

1. Dans l’action, fournissez les informations sur l’environnement dans lequel vous souhaitez créer le nouvel enregistrement de tâche. Le cas échéant, d’autres propriétés s’affichent également en fonction de l’entité que vous avez sélectionnée pour cette action, par exemple :

   ![Informations d’action pour l’environnement dans lequel créer l’enregistrement](./media/connect-common-data-service/create-new-record-action-details.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom de l’organisation** | Oui | Environnement où vous souhaitez créer l’enregistrement ; bien que cet environnement puisse différer de l’environnement de votre déclencheur, il porte le nom « Fabrikam Sales Production » dans cet exemple. |
   | **Nom de l’entité** | Oui | Entité où vous souhaitez créer l’enregistrement, par exemple, « Tasks » (Tâches) |
   | **Subject** | Oui, en fonction de l’entité sélectionnée dans cet exemple | Brève description de l’objectif de cette tâche |
   ||||

   1. Pour la propriété **Objet**, entrez ce texte avec un espace de fin :

      `Follow up with new lead:`

   1. Gardez votre pointeur à l’intérieur de la zone **Objet** pour que la liste de contenu dynamique reste visible.
   
   1. Dans la liste, à partir de la section **Lorsqu’un enregistrement est créé**, sélectionnez les sorties du déclencheur que vous souhaitez inclure dans l’enregistrement de tâche, par exemple :

      ![Sélectionner les sorties de déclencheur à utiliser dans l’enregistrement de tâche](./media/connect-common-data-service/create-new-record-action-select-trigger-outputs.png)

      | Sortie du déclencheur | Description |
      |----------------|-------------|
      | **Prénom** | Premier nom de l’enregistrement de prospect à utiliser comme contact principal dans l’enregistrement de tâche |
      | **Last Name** | Nom de famille de l’enregistrement de prospect en tant que contact principal dans l’enregistrement de tâche |
      | **Description** | Autres sorties à inclure dans l’enregistrement de tâche, telles que l’adresse de messagerie et le numéro de téléphone professionnel |
      |||

   Lorsque vous avez terminé, l’action peut se présenter comme dans l’exemple suivant :

   ![Action « Créer un enregistrement » terminée](./media/connect-common-data-service/finished-create-record-action-details.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

1. Pour lancer manuellement l’application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur. Pour tester votre application logique, créez un enregistrement « Prospects ».

## <a name="trigger-only-on-updated-attributes"></a>Déclencher uniquement sur les attributs mis à jour

Pour les déclencheurs qui s’exécutent lorsque des enregistrements sont mis à jour, tels que l’action **Lorsqu’un enregistrement est mis à jour**, vous pouvez utiliser des attributs de filtre pour que votre application logique s’exécute uniquement lorsque les attributs spécifiés sont mis à jour. Cette fonctionnalité vous aide à empêcher l’exécution d’applications logiques inutiles.

1. Dans le déclencheur, dans la liste **Ajouter un nouveau paramètre**, sélectionnez **Filtres d’attribut**.

   ![Capture d’écran montrant l’action « Lorsqu’un enregistrement est mis à jour » et la liste « Ajouter un nouveau paramètre » ouverte avec la propriété « Filtres d’attribut » sélectionnée.](./media/connect-common-data-service/when-record-updated-trigger-add-attribute-filters.png)

1. Pour chaque **Élément Filtres d’attribut**, sélectionnez l’attribut dont vous souhaitez analyser les mises à jour, par exemple :

   ![Ajouter la propriété « Filtres d’attribut »](./media/connect-common-data-service/when-record-updated-trigger-select-attribute-filter.png)

## <a name="list-records-based-on-a-filter"></a>Répertorier les enregistrements en fonction d’un filtre

Pour les actions qui retournent des enregistrements, telles que l’action **Liste des enregistrements**, vous pouvez utiliser une requête ODATA qui retourne des enregistrements en fonction du filtre spécifié. Par exemple, vous pouvez faire en sorte que l’action répertorie uniquement les enregistrements pour les comptes actifs.

1. Dans l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez la propriété **Requête de filtre**.

   ![Ajouter la propriété « Requête de filtre »](./media/connect-common-data-service/list-records-action-filter-query.png)

1. Dans la propriété **Requête de filtre** qui apparaît maintenant dans l’action, entrez cette requête de filtre ODATA : `statuscode eq 1`

   ![Entrer une requête de filtre ODATA pour le filtrage des enregistrements](./media/connect-common-data-service/list-records-action-filter-query-value.png)

Pour plus d’informations sur les `$filter`options de requête système , consultez [Common Data Service - Filtrer les résultats](/powerapps/developer/common-data-service/webapi/query-data-web-api#filter-results).

## <a name="list-records-based-on-an-order"></a>Répertorier les enregistrements en fonction d’une commande

Pour les actions qui retournent des enregistrements, telles que l’action **Liste des enregistrements**, vous pouvez utiliser une requête ODATA qui retourne les enregistrements dans un ordre spécifié, qui varie en fonction des enregistrements renvoyés par l’action. Par exemple, vous pouvez faire en sorte que l’action répertorie les enregistrements en fonction du nom du compte.

1. Dans l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez la propriété **Trier par**.

   ![Ajouter la propriété « Trier par »](./media/connect-common-data-service/list-records-action-order-by.png)

1. Dans la propriété **Trier par** qui apparaît maintenant dans l’action, entrez cette requête de filtre ODATA : `name`

   ![Entrer une requête de filtre ODATA pour le tri des enregistrements](./media/connect-common-data-service/list-records-action-order-by-value.png)

Pour plus d’informations sur les `$orderby`options de requête système , consultez [Common Data Service - Trier les résultats](/powerapps/developer/common-data-service/webapi/query-data-web-api#order-results).

## <a name="field-data-types"></a>Types de données de champ

Même si vous entrez manuellement une valeur ou si vous sélectionnez une valeur dans la liste de contenu dynamique pour un champ dans un déclencheur ou une action, le type de données de la valeur doit correspondre au type de données requis du champ.

Ce tableau décrit certains des types de champs et les types de données requis pour leurs valeurs.

| Champ | Type de données | Description |
|-------|-----------|-------------|
| Champ texte | Ligne de texte unique | Requiert une seule ligne de texte ou du contenu dynamique qui a le type de données texte, par exemple, les propriétés suivantes : <p><p>- **Description** <br>- **Catégorie** |
| Champ entier | Nombre entier | Requiert un entier ou un contenu dynamique qui a le type de données Entier, par exemple, les propriétés suivantes : <p><p>- **Pourcentage effectué** <br>- **Durée** |
| Champ de date | Date et heure | Requiert une date au format MM/JJ/YYY ou un contenu dynamique qui a le type de données date, par exemple, les propriétés suivantes : <p><p>- **Créée le** <br>- **Date de début** <br>- **Début réel** <br>- **Fin réelle** <br>- **Date d’échéance** |
| Champ qui fait référence à un autre enregistrement d’entité | Clé primaire | Nécessite à la fois un ID d’enregistrement, tel qu’un GUID, et un type de recherche, ce qui signifie que les valeurs de la liste de contenu dynamique ne fonctionneront pas, par exemple, les propriétés suivantes : <p><p>- **Propriétaire** : Doit être un ID d’utilisateur valide ou un ID d’enregistrement d’équipe. <br>- **Type de propriétaire** : Doit être un type de recherche, par exemple `systemusers` ou `teams`, respectivement. <p><p>- **Concernant** : Doit être un ID d’enregistrement valide, tel qu’un ID de compte ou un ID d’enregistrement de contact. <br>- **Type concernant** : Doit être un type de recherche, par exemple `accounts` ou `contacts`, respectivement. <p><p>- **Client** : Doit être un ID d’enregistrement valide, tel qu’un ID de compte ou un ID d’enregistrement de contact. <br>- **Type de client** : Doit être le type de recherche, par exemple `accounts` ou `contacts`, respectivement. |
||||

Cet exemple montre comment l'action **Créer un enregistrement** crée un nouvel enregistrement « Tâches » associé à d’autres enregistrements d’entité, en particulier un enregistrement utilisateur et un enregistrement de compte. L’action spécifie les ID et les types de recherche pour ces enregistrements d’entité en utilisant des valeurs qui correspondent aux types de données attendus pour les propriétés pertinentes.

* Sur la base de la propriété **Propriétaire**, qui spécifie un ID d’utilisateur et la propriété **Type de propriétaire**, qui spécifie le type de recherche `systemusers`, l’action associe le nouvel enregistrement « Tâches » à un utilisateur spécifique.

* Sur la base de la propriété **Concernant**, qui spécifie un ID d’enregistrement, et de la propriété **Concernant le type**, qui spécifie le type de recherche `accounts`, l’action associe le nouvel enregistrement « Tâches » à un compte spécifique.

![Créer un enregistrement « Tâches » associé à des ID et des types de recherche](./media/connect-common-data-service/create-new-record-task-properties.png)

## <a name="connector-reference"></a>Référence de connecteur

Pour des informations techniques sur la description Swagger du connecteur, comme les déclencheurs, les actions, les limites et d’autres détails, consultez la [page de référence du connecteur](/connectors/commondataservice/).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [tous les autres connecteurs pour Azure Logic apps](../connectors/apis-list.md)
