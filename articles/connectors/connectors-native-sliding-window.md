---
title: Planifier des tâches pour gérer les données contiguës
description: Créer et exécuter des tâches récurrentes qui gèrent les données contiguës à l’aide de fenêtres glissantes dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 03/25/2020
ms.openlocfilehash: 103805fbf395dc120acc96fbcee273abcf14939d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010416"
---
# <a name="schedule-and-run-tasks-for-contiguous-data-by-using-the-sliding-window-trigger-in-azure-logic-apps"></a>Planifier et exécuter des tâches pour les données contiguës à l’aide du déclencheur Fenêtre glissante dans Azure Logic Apps

Pour exécuter régulièrement des tâches, processus ou travaux devant gérer des données dans des blocs contigus, vous pouvez démarrer votre workflow d’application logique avec le déclencheur **Fenêtre glissante**. Vous pouvez définir une date et une heure, ainsi qu’un fuseau horaire, pour démarrer le flux de travail, et une récurrence pour la répétition de ce flux de travail. Si des récurrences sont manquées pour une raison quelconque, par exemple suite à des interruptions ou à des flux de travail désactivés, ce déclencheur traite ces récurrences manquées. Par exemple, lors de la synchronisation de données entre votre base de données et le stockage de sauvegarde, utilisez le déclencheur Fenêtre glissante afin que les données soient synchronisées sans que cela génère des écarts. Pour plus d’informations sur les déclencheurs et actions de Planification intégrés, voir [Planifier et exécuter des tâches et des workflows automatisés et récurrents avec Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

Voici quelques modèles que ce déclencheur prend en charge :

* Exécuter immédiatement et répéter tou(te)s les *n* secondes, minutes, heures, jours, semaines ou mois.

* Démarrer à une date et une heure spécifiques, exécuter immédiatement et répéter tou(te)s les *n* secondes, minutes, heures, jours, semaines ou mois. Avec ce déclencheur, vous pouvez spécifier une heure de début dans le passé, qui exécute toutes les périodicités passées.

* Retarder chaque périodicité pendant une durée spécifique avant de l’exécuter.

Pour connaître les différences entre ce déclencheur et le déclencheur Périodicité, ou pour plus d’informations sur la planification des flux de travail récurrents, voir [Planifier et exécuter des tâches, processus et workflows automatisés et récurrents avec Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

> [!TIP]
> Pour déclencher votre application logique et l’exécuter une seule fois, voir [Exécuter des travaux une seule fois](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#run-once).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Connaissances de base sur les [applications logiques](../logic-apps/logic-apps-overview.md). Si vous débutez avec les applications logiques, découvrez [comment créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-sliding-window-trigger"></a>Ajouter un déclencheur Fenêtre glissante

1. Connectez-vous au [portail Azure](https://portal.azure.com). Créez une application logique vide.

1. Une fois que le concepteur Logic Apps s’affiche, entrez `sliding window` comme filtre dans la zone de recherche. Dans la liste des déclencheurs, sélectionnez le déclencheur **Fenêtre glissante** pour la première étape dans votre workflow d’application logique.

   ![Sélectionner le déclencheur « Fenêtre glissante »](./media/connectors-native-sliding-window/add-sliding-window-trigger.png)

1. Définissez l’intervalle et la fréquence de la périodicité. Dans cet exemple, définissez ces propriétés de sorte que votre flux de travail soit exécuté chaque semaine.

   ![Définir l’intervalle et la fréquence](./media/connectors-native-sliding-window/sliding-window-trigger-details.png)

   | Propriété | Nom JSON | Obligatoire | Type | Description |
   |----------|----------|-----------|------|-------------|
   | **Intervalle** | `interval` | Oui | Integer | Nombre entier positif qui décrit la fréquence à laquelle le flux de travail s’exécute en fonction de la fréquence. Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois <br>- Semaine : 1-71 semaines <br>Jour : 1-500 jours <br>- Heure : 1-12 000 heures <br>- Minute : 1-72 000 minutes <br>- Seconde : 1-9 999 999 secondes <p>Par exemple, si l’intervalle est de 6 et que la fréquence soit définie sur « Mois », la périodicité est alors tous les 6 mois. |
   | **Fréquence** | `frequency` | Oui | String | Unité de temps à utiliser pour la récurrence : **Seconde**, **Minute**, **Heure**, **Jour**, **Semaine** ou **Mois** |
   ||||||

   ![Options de périodicité avancées](./media/connectors-native-sliding-window/sliding-window-trigger-more-options-details.png)

   Pour plus d’options de périodicité, ouvrez la liste **Ajouter un nouveau paramètre**. Toutes les options que vous sélectionnez s’affichent sur le déclencheur.

   | Propriété | Obligatoire | Nom JSON | Type | Description |
   |----------|----------|-----------|------|-------------|
   | **Retard** | Non | delay | String | Valeur de délai pour retarder chaque périodicité à l’aide de la [spécification date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations) |
   | **Fuseau horaire** | Non | timeZone | String | S’applique uniquement quand vous spécifiez une heure de début, car ce déclencheur n’accepte pas le [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Sélectionnez le fuseau horaire à appliquer. |
   | **Heure de début** | Non | startTime | String | Indiquez une date et une heure de début au format suivant : <p>AAAA-MM-JJThh:mm:ss si vous sélectionnez un fuseau horaire <p>-ou- <p>AAAA-MM-JJThh:mm:ssZ si vous ne sélectionnez pas de fuseau horaire <p>Par exemple, si vous choisissez le 18 septembre 2017 à 14h00, alors spécifiez « 2017-09-18T14:00:00 » et sélectionnez un fuseau horaire tel que « Pacific Standard Time » (Heure standard du Pacifique). Vous pouvez également spécifier « 2017-09-18T14:00:00Z » sans fuseau horaire. <p>**Remarque :** Cette heure de début doit être conforme à la [spécification de date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) au [format de date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), mais sans [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Si vous ne sélectionnez pas de fuseau horaire, vous devez ajouter la lettre « Z » à la fin sans espace. Ce « Z » fait référence au [temps nautique](https://en.wikipedia.org/wiki/Nautical_time) équivalent. <p>Pour les planifications simples, l’heure de début est la première occurrence, tandis que, pour les périodicités complexes, le déclencheur ne s’active pas avant l’heure de début. [*Comment puis-je utiliser la date et l’heure de début ?*](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time) |
   |||||

1. Créez maintenant votre flux de travail restant avec d’autres actions. Pour connaître les autres actions que vous pouvez ajouter, voir [Connecteurs pour Azure Logic Apps](../connectors/apis-list.md).

## <a name="workflow-definition---sliding-window"></a>Définition de workflow – Fenêtre glissante

Dans la définition de workflow sous-jacente de votre application logique, qui utilise JSON, vous pouvez afficher la définition de déclencheur Fenêtre glissante avec les options que vous avez choisies. Pour afficher cette définition, dans la barre d’outils Concepteur, choisissez **Mode Code**. Pour revenir au Concepteur, dans la barre d’outils Concepteur, choisissez **Concepteur**.

Cet exemple montre à quoi une définition de déclencheur Fenêtre coulissante peut ressembler dans une définition de workflow sous-jacente dans laquelle le délai pour chaque périodicité est de cinq secondes pour une périodicité horaire :

``` json
"triggers": {
   "Recurrence": {
      "type": "SlidingWindow",
      "Sliding_Window": {
         "inputs": {
            "delay": "PT5S"
         },
         "recurrence": {
            "frequency": "Hour",
            "interval": 1,
            "startTime": "2019-05-13T14:00:00Z",
            "timeZone": "Pacific Standard Time"
         }
      }
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Retarder l’action suivante dans des workflows](../connectors/connectors-native-delay.md)
* [Connecteurs pour Logic Apps](../connectors/apis-list.md)
