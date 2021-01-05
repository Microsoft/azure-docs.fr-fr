---
title: Retarder l’action suivante dans des workflows
description: Retarder l'action suivante dans des workflows d’application logique à l'aide des actions Retarder ou Retarder jusqu'à dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, klam, logicappspm
ms.topic: conceptual
ms.date: 05/25/2019
tags: connectors
ms.openlocfilehash: 5348ade1ba6eec6cbd360849411b4520cb3c2b19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74787334"
---
# <a name="delay-running-the-next-action-in-azure-logic-apps"></a>Retarder l'exécution de l’action suivante dans Azure Logic Apps

Pour permettre à votre application logique d'attendre un certain temps avant d’exécuter l’action suivante, vous pouvez ajouter l'action intégrée **Retarder - Planification** devant une action de votre workflow d'application logique. Vous pouvez également ajouter l'action **Retarder jusqu'à - Planification** pour attendre jusqu'à une date et une heure spécifiques avant d’exécuter l’action suivante. Pour plus d’informations sur les actions et déclencheurs Planification intégrés, consultez [Planifier et exécuter des tâches et des workflows automatisés et récurrents avec Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

* **Retarder** : Attendre le nombre spécifié d'unités de temps, par exemple des secondes, minutes, heures, jours, semaines ou mois avant l'exécution de l'action suivante.

* **Retarder jusqu’à** : Attendre la date et l'heure spécifiées avant l'exécution de l'action suivante.

Voici quelques exemples illustrant l'utilisation de ces actions :

* Attendre un jour de semaine pour envoyer une mise à jour d’état par e-mail.

* Retarder votre workflow jusqu'à ce qu’un appel HTTP se termine avant de poursuivre et de récupérer des données.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Connaissances de base sur les [applications logiques](../logic-apps/logic-apps-overview.md). Pour utiliser une action, votre application logique doit d’abord démarrer avec un déclencheur. Vous pouvez utiliser le déclencheur de votre choix et ajouter d’autres actions avant d’ajouter une action Retarder. Cette rubrique utilise un déclencheur Office 365 Outlook. Si vous débutez avec les applications logiques, découvrez [comment créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="add-delay"></a>

## <a name="add-the-delay-action"></a>Ajouter l'action Retarder

1. Pour le Concepteur d'application logique, sous l’étape où vous souhaitez ajouter l'action Retarder, sélectionnez **Nouvelle étape**.

   Pour ajouter l’action Retarder entre des étapes, déplacez le pointeur sur la flèche qui connecte ces étapes. Cliquez sur le signe plus (+) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « retarder » comme filtre. Dans la liste des actions, sélectionnez cette action : **Retard**

   ![Ajouter l'action « Retarder »](./media/connectors-native-delay/add-delay-action.png)

1. Spécifiez le temps d’attente avant l'exécution de l'action suivante.

   ![Définir la durée du retard](./media/connectors-native-delay/delay-time-intervals.png)

   | Propriété | Nom JSON | Obligatoire | Type | Description |
   |----------|-----------|----------|------|-------------|
   | Count | count | Oui | Integer | Le nombre d’unités à retarder |
   | Unité | unité | Oui | String | Unité de temps, par exemple : `Second`, `Minute`, `Hour`, `Day`, `Week` ou `Month` |
   ||||||

1. Ajoutez les autres actions que vous souhaitez exécuter dans votre workflow.

1. Lorsque vous avez terminé, enregistrez votre application logique.

<a name="add-delay-until"></a>

## <a name="add-the-delay-until-action"></a>Ajouter l'action Retarder jusqu'à

1. Pour le Concepteur d'application logique, sous l’étape où vous souhaitez ajouter l'action Retarder, sélectionnez **Nouvelle étape**.

   Pour ajouter l’action Retarder entre des étapes, déplacez le pointeur sur la flèche qui connecte ces étapes. Cliquez sur le signe plus (+) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez « retarder » comme filtre. Dans la liste des actions, sélectionnez cette action : **Retarder jusqu’à**

   ![Ajouter l’action « Différer jusqu’à »](./media/connectors-native-delay/add-delay-until-action.png)

1. Indiquez la date et l'heure de fin auxquelles vous souhaitez reprendre le workflow.

   ![Spécifier le timestamp auquel mettre fin au retard](./media/connectors-native-delay/delay-until-timestamp.png)

   | Propriété | Nom JSON | Obligatoire | Type | Description |
   |----------|-----------|----------|------|-------------|
   | Timestamp | timestamp | Oui | String | Date et heure de fin pour reprendre le workflow au format : <p>AAAA-MM-JJThh:mm:ssZ <p>Par exemple, si vous optez pour le 18 septembre 2017 à 14:00, indiquez « 2017-09-18T14:00:00Z ». <p>**Remarque :** ce format d'heure doit être conforme à la [spécification de date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) au [format de date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), mais sans [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Sans fuseau horaire, vous devez ajouter la lettre « Z » à la fin sans espace. Ce « Z » fait référence au [temps nautique](https://en.wikipedia.org/wiki/Nautical_time) équivalent. |
   ||||||

1. Ajoutez les autres actions que vous souhaitez exécuter dans votre workflow.

1. Lorsque vous avez terminé, enregistrez votre application logique.

## <a name="next-steps"></a>Étapes suivantes

* [Créer, planifier et exécuter des tâches et des workflows récurrents avec le déclencheur Périodicité](../connectors/connectors-native-recurrence.md)
* [Connecteurs pour Logic Apps](../connectors/apis-list.md)