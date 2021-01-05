---
title: Planifier des tâches et des workflows périodiques
description: Planifier et exécuter des tâches et des flux de travail automatisés et récurrents avec le déclencheur Récurrence dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 11/03/2020
ms.openlocfilehash: a9c6017ed46853b9dec991fc02097ee88c67a7cb
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342179"
---
# <a name="create-schedule-and-run-recurring-tasks-and-workflows-with-the-recurrence-trigger-in-azure-logic-apps"></a>Créer, planifier et exécuter des tâches et des flux de travail récurrents avec le déclencheur Récurrence dans Azure Logic Apps

Pour exécuter régulièrement des tâches, des processus ou des travaux en fonction d’une planification spécifique, vous pouvez démarrer le flux de travail de votre application logique avec le déclencheur **Récurrence - Planification** intégré. Vous pouvez définir une date et une heure, ainsi qu’un fuseau horaire, pour démarrer le flux de travail, et une récurrence pour la répétition de ce flux de travail. Si des périodicités sont manquées pour une quelconque raison, par exemple en raison d’interruptions ou de désactivation de workflows, le déclencheur ne les traite pas, mais redémarre les périodicités selon l’intervalle planifié suivant. Pour plus d’informations sur les déclencheurs et actions de Planification intégrés, voir [Planifier et exécuter des tâches et des workflows automatisés et récurrents avec Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

Voici quelques modèles pris en charge par ce déclencheur, ainsi que des récurrences plus avancées et des planifications complexes :

* Exécuter immédiatement et répéter tou(te)s les *n* secondes, minutes, heures, jours, semaines ou mois.

* Démarrer à une date et une heure spécifiques, exécuter immédiatement et répéter tou(te)s les *n* secondes, minutes, heures, jours, semaines ou mois.

* Exécuter et répéter une ou plusieurs fois par jour, par exemple à 8h00 et 17h00.

* Exécuter et répéter chaque semaine, mais uniquement certains jours, par exemple le samedi et le dimanche.

* Exécuter et répéter chaque semaine, mais uniquement certains jours et à des heures précises, par exemple du lundi au vendredi à 8h00 et 17h00.

Pour connaître les différences entre ce déclencheur et le déclencheur Fenêtre glissante, ou pour plus d’informations sur la planification des flux de travail récurrents, consultez [Schedule and run recurring automated tasks, processes, and workflows with Azure Logic Apps (Planifier et exécuter des tâches, des processus et des flux de travail automatisés et récurrents avec Azure Logic Apps)](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

> [!TIP]
> Pour déclencher votre application logique et l’exécuter une seule fois, voir [Exécuter des travaux une seule fois](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#run-once).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Connaissances de base sur les [applications logiques](../logic-apps/logic-apps-overview.md). Si vous débutez avec les applications logiques, découvrez [comment créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-recurrence-trigger"></a>Ajouter le déclencheur Récurrence

1. Connectez-vous au [portail Azure](https://portal.azure.com). Créez une application logique vide.

1. Une fois que le concepteur Logic Apps s’affiche, entrez `recurrence` comme filtre dans la zone de recherche. Dans la liste des déclencheurs, sélectionnez ce déclencheur pour la première étape dans votre workflow d’application logique : **Périodicité**

   ![Sélectionner le déclencheur « Récurrence »](./media/connectors-native-recurrence/add-recurrence-trigger.png)

1. Définissez l’intervalle et la fréquence de la périodicité. Dans cet exemple, définissez ces propriétés de sorte que votre flux de travail soit exécuté chaque semaine.

   ![Définir l’intervalle et la fréquence](./media/connectors-native-recurrence/recurrence-trigger-details.png)

   | Propriété | Nom JSON | Obligatoire | Type | Description |
   |----------|-----------|----------|------|-------------|
   | **Intervalle** | `interval` | Oui | Integer | Nombre entier positif qui décrit la fréquence à laquelle le flux de travail s’exécute en fonction de la fréquence. Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois <br>- Semaine : 1-71 semaines <br>Jour : 1-500 jours <br>- Heure : 1-12 000 heures <br>- Minute : 1-72 000 minutes <br>- Seconde : 1-9 999 999 secondes<p>Par exemple, si l’intervalle est de 6 et que la fréquence soit définie sur « Mois », la périodicité est alors tous les 6 mois. |
   | **Fréquence** | `frequency` | Oui | String | Unité de temps à utiliser pour la récurrence : **Seconde** , **Minute** , **Heure** , **Jour** , **Semaine** ou **Mois** |
   ||||||

   > [!IMPORTANT]
   > Quand les récurrences ne spécifient pas d’options de planification avancées, les récurrences ultérieures sont basées sur l’heure de la dernière exécution.
   > Les heures de début de ces récurrences peuvent dériver en raison de facteurs tels que la latence lors des appels de stockage. Pour vous assurer que votre application logique ne manque pas de récurrence, en particulier quand la fréquence est en jours ou plus, utilisez l’une des options suivantes :
   > 
   > * Indiquez une heure de début pour la récurrence.
   > 
   > * Spécifiez les heures et les minutes d’exécution de la récurrence à l’aide des propriétés **À ces heures** et **À ces minutes**.
   > 
   > * Utilisez le [déclencheur de fenêtre glissante](../connectors/connectors-native-sliding-window.md) plutôt que le déclencheur de récurrence.

1. Pour définir des options de planification avancées, ouvrez la liste **Ajouter un nouveau paramètre**. Toutes les options que vous sélectionnez s’affichent sur le déclencheur.

   ![Options avancées de planification](./media/connectors-native-recurrence/recurrence-trigger-more-options-details.png)

   | Propriété | Nom JSON | Obligatoire | Type | Description |
   |----------|-----------|----------|------|-------------|
   | **Fuseau horaire** | `timeZone` | Non | String | S’applique uniquement quand vous spécifiez une heure de début, car ce déclencheur n’accepte pas le [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Sélectionnez le fuseau horaire à appliquer. |
   | **Heure de début** | `startTime` | Non | String | Fournissez une date et une heure de début, qui ne peuvent pas dépasser 49 ans dans le futur et doivent être conformes à la [spécification de date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) au [format de date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), mais sans [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset) : <p><p>AAAA-MM-JJThh:mm:ss si vous sélectionnez un fuseau horaire <p>-ou- <p>AAAA-MM-JJThh:mm:ssZ si vous ne sélectionnez pas de fuseau horaire <p>Par exemple, si vous choisissez le 18 septembre 2020 à 14h00, alors spécifiez « 2020-09-18T14:00:00 » et sélectionnez un fuseau horaire tel que « Pacific Standard Time » (Heure standard du Pacifique). Vous pouvez également spécifier « 2020-09-18T14:00:00Z » sans fuseau horaire. <p><p>**Important :** Si vous ne sélectionnez pas de fuseau horaire, vous devez ajouter la lettre « Z » à la fin sans espace. Ce « Z » fait référence au [temps nautique](https://en.wikipedia.org/wiki/Nautical_time) équivalent. Si vous sélectionnez une valeur de fuseau horaire, vous n’avez pas besoin d’ajouter un « Z » à la fin de votre valeur **Heure de début**. Dans ce cas, Logic Apps ignore la valeur de fuseau horaire, car « Z » désigne un format d’heure UTC. <p><p>Pour les planifications simples, l’heure de début est la première occurrence, tandis que pour les planifications complexes, le déclencheur ne s’active pas avant l’heure de début. [*Comment puis-je utiliser la date et l’heure de début ?*](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time) |
   | **Aux jours indiqués** | `weekDays` | Non | Chaîne ou tableau de chaînes | Si vous sélectionnez « Semaine », vous pouvez sélectionner un ou plusieurs jours où vous voulez exécuter le workflow : **Lundi** , **Mardi** , **Mercredi** , **Jeudi** , **Vendredi** , **Samedi** et **Dimanche** |
   | **Aux heures indiquées** | `hours` | Non | Entier ou tableau d’entiers | Si vous sélectionnez « Jour » ou « Semaine », vous pouvez sélectionner un ou plusieurs entiers compris entre 0 et 23 pour les heures de la journée durant lesquelles exécuter le flux de travail. <p><p>Par exemple, si vous spécifiez « 10 », « 12 » et « 14 », vous obtenez 10h00, 12h00 et 14h00 pour les heures de la journée, mais les minutes de la journée sont calculées en fonction de démarrage de la récurrence. Pour définir des minutes spécifiques du jour, par exemple, 10h00, 12h00 et 14h00, spécifiez ces valeurs à l’aide de la propriété **Aux minutes indiquées**. |
   | **Aux minutes indiquées** | `minutes` | Non | Entier ou tableau d’entiers | Si vous sélectionnez « Jour » ou « Semaine », vous pouvez sélectionner un ou plusieurs entiers compris entre 0 et 59 pour les minutes de l’heure durant lesquelles exécuter le flux de travail. <p>Par exemple, vous pouvez spécifier « 30 » pour les minutes et à l’aide de l’exemple précédent des heures de la journée, vous obtenez 10h30, 12h30 et 14h30. <p>**Remarque**  : Parfois, l’horodatage de l’exécution déclenchée peut varier jusqu’à une minute par rapport à l’heure prévue. Si vous devez transmettre l’horodatage exactement comme prévu aux actions suivantes, vous pouvez utiliser des expressions de modèle pour modifier l’horodatage en conséquence. Pour plus d’informations, consultez [Fonctions de date et d’heure pour les expressions](../logic-apps/workflow-definition-language-functions-reference.md#date-time-functions). |
   |||||

   Par exemple, supposons que nous sommes le lundi 4 septembre 2020 aujourd’hui. Le déclencheur Récurrence suivant ne s’active *pas avant* la date et l’heure de début définies, à savoir le vendredi 18 septembre 2020 à 8h00 (PST). Notez toutefois que la périodicité planifiée est définie sur 10h30, 12h30 et 14h30 uniquement le lundi. Par conséquent, le déclencheur s’activera et créera une instance de flux de travail d’application logique pour la première fois le lundi à 10h30. Pour plus d’informations sur le fonctionnement des heures de début, consultez ces [exemples d’heure de début](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#start-time).

   Les exécutions suivantes auront lieu le même jour à 12h30 et 14h30. Chaque périodicité crée sa propre instance de flux de travail. L’ensemble de la planification se répète ensuite régulièrement chaque lundi. [*Existe-t-il d’autres exemples de périodicité ?*](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md#example-recurrences)

   ![Exemple de planification avancée](./media/connectors-native-recurrence/recurrence-trigger-advanced-schedule-options.png)

   > [!NOTE]
   > Le déclencheur affiche un aperçu de la périodicité que vous avez spécifiée uniquement quand vous sélectionnez « Jour » ou « Semaine » comme fréquence.

1. Créez maintenant votre flux de travail restant avec d’autres actions. Pour connaître les autres actions que vous pouvez ajouter, voir [Connecteurs pour Azure Logic Apps](../connectors/apis-list.md).

## <a name="workflow-definition---recurrence"></a>Définition du flux de travail - Récurrence

Dans la définition de flux de travail sous-jacente de votre application logique, qui utilise JSON, vous pouvez afficher la [définition du déclencheur Récurrence](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger) avec les options que vous avez choisies. Pour afficher cette définition, dans la barre d’outils Concepteur, choisissez **Mode Code**. Pour revenir au Concepteur, dans la barre d’outils Concepteur, choisissez **Concepteur**.

Cet exemple montre à quoi pourrait ressembler un déclencheur Récurrence dans une définition de flux de travail sous-jacente :

``` json
"triggers": {
   "Recurrence": {
      "type": "Recurrence",
      "recurrence": {
         "frequency": "Week",
         "interval": 1,
         "schedule": {
            "hours": [
               10,
               12,
               14
            ],
            "minutes": [
               30
            ],
            "weekDays": [
               "Monday"
            ]
         },
         "startTime": "2020-09-07T14:00:00Z",
         "timeZone": "Pacific Standard Time"
      }
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Suspendre les workflows avec des actions Retarder](../connectors/connectors-native-delay.md)
* [Connecteurs pour Logic Apps](../connectors/apis-list.md)
