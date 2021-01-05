---
title: Planification de tâches et de workflows récurrents dans Azure Logic Apps
description: Vue d'ensemble de la planification de tâches, processus et workflows automatisés récurrents avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, jonfan, logicappspm
ms.topic: conceptual
ms.date: 03/25/2020
ms.openlocfilehash: a5f01e81564561fe43ef6e55e6e9b3b67d6e1d77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84945611"
---
# <a name="schedule-and-run-recurring-automated-tasks-processes-and-workflows-with-azure-logic-apps"></a>Créer et exécuter des tâches et des workflows récurrents avec Azure Logic Apps

Logic Apps vous aide à créer et à exécuter des tâches et des processus récurrents automatisés selon un calendrier. En créant un workflow d'application logique qui démarre avec un déclencheur intégré de type Périodicité ou Fenêtre glissante, c’est-à-dire des déclencheurs de type Planifier, vous pouvez exécuter des tâches immédiatement, ultérieurement ou à intervalles réguliers. Vous pouvez appeler des services à l'intérieur et à l'extérieur d'Azure, notamment des points de terminaison HTTP ou HTTPS, poster des messages sur des services Azure comme Azure Storage et Azure Service Bus, ou charger des fichiers vers un partage de fichiers. Le déclencheur Périodicité vous permet également de configurer des programmes complexes et des récurrences avancées pour exécuter des tâches. Pour plus d’informations sur les déclencheurs et actions de planification intégrés, consultez [Planifier des déclencheurs](#schedule-triggers) et [Planifier des actions](#schedule-actions). 

> [!TIP]
> Vous pouvez planifier et exécuter des charges de travail récurrentes sans créer d’application logique distincte pour chaque travail planifié et en cours d'exécution dans la [limite des workflows par région et par abonnement](../logic-apps/logic-apps-limits-and-config.md#definition-limits). Vous pouvez privilégier l'utilisation du modèle d’application logique créé par le [modèle de démarrage rapide Azure : Planificateur de travaux Logic Apps](https://github.com/Azure/azure-quickstart-templates/tree/master/301-logicapps-jobscheduler/).
>
> Le modèle du planificateur de travaux Logic Apps crée une application logique CreateTimerJob qui appelle une application logique TimerJob. Vous pouvez ensuite appeler l’application logique CreateTimerJob en tant qu’API en effectuant une requête HTTP et en utilisant une planification en tant qu'entrée pour la requête. Chaque appel fait à l’application logique CreateTimerJob appelle également l’application logique TimerJob, ce qui crée une nouvelle instance TimerJob qui s’exécute en continu en fonction de la planification spécifiée ou jusqu’à ce que la limite spécifiée soit atteinte. Ainsi, vous pouvez exécuter autant d’instances TimerJob que vous le souhaitez, sans vous soucier des limites de workflow car les instances ne correspondent pas à des définitions ou ressources de workflow d’application logique individuelle.

Cette liste présente quelques exemples de tâches que vous pouvez exécuter avec les déclencheurs de planification intégrés :

* Obtenir des données internes, par exemple exécuter une procédure stockée SQL tous les jours.

* Obtenir des données externes, par exemple extraire des rapports météorologiques de NOAA toutes les 15 minutes.

* Envoyer des rapports de données, par exemple envoyer par e-mail un résumé de toutes les commandes supérieures à une quantité spécifique de la semaine antérieure.

* Traiter des données, par exemple compresser les images téléchargées le jour même tous les jours aux heures creuses.

* Nettoyer des données, par exemple supprimer tous les tweets de plus de trois mois.

* Archiver des données, par exemple envoyer les factures à un service de sauvegarde tous les jours à 1 h du matin pendant les neuf prochains mois.

Vous pouvez également utiliser les actions intégrées Planifier pour suspendre votre workflow avant l'exécution de l'action suivante, par exemple :

* Attendre un jour de semaine pour envoyer une mise à jour d’état par e-mail.

* Retarder le workflow jusqu’à ce qu’un appel HTTP dispose d’assez de temps avant la reprise et la récupération du résultat.

Cet article décrit les capacités des déclencheurs et actions intégrés de type Planifier.

<a name="schedule-triggers"></a>

## <a name="schedule-triggers"></a>Planifier des déclencheurs

Vous pouvez démarrer votre workflow d'application logique en utilisant le déclencheur Périodicité ou le déclencheur Fenêtre glissante, qui n'est associé à aucun service ou système spécifique. Ces déclencheurs démarrent et exécutent votre workflow en fonction de la périodicité que vous avez spécifiée lors de la sélection de l’intervalle et de la fréquence, par exemple le nombre de secondes, de minutes, d'heures, de jours, de semaines ou de mois. Vous pouvez également définir la date et l'heure de début ainsi que le fuseau horaire. Chaque fois qu’un déclencheur est activé, Logic Apps crée et exécute une instance de workflow pour votre application logique.

Voici les différences entre ces types déclencheurs :

* **Périodicité** : Exécute votre workflow à intervalles réguliers selon la périodicité que vous avez spécifiée. Si des périodicités sont manquées, par exemple en raison d’interruptions ou de désactivation de workflows, le déclencheur Périodicité ne les traite pas, mais redémarre les périodicités selon l’intervalle planifié suivant. Vous pouvez spécifier la date et l'heure de début ainsi que le fuseau horaire. Si vous sélectionnez « Jour », vous pouvez spécifier les heures du jour et les minutes, par exemple, tous les jours à 14h30. Si vous sélectionnez « Semaine », vous pouvez également sélectionner les jours de la semaine, par exemple le mercredi et le samedi. Pour plus d'informations, voir [Créer, planifier et exécuter des tâches et des workflows récurrents avec le déclencheur Périodicité](../connectors/connectors-native-recurrence.md).

* **Fenêtre glissante** : Exécute votre workflow à des intervalles réguliers qui traitent les données en continu. Si des récurrences sont manquées pour une raison quelconque, par exemple suite à des interruptions ou à des flux de travail désactivés, le déclencheur Fenêtre glissante revient en arrière et traite ces récurrences manquées. Vous pouvez spécifier une date et une heure de début, un fuseau horaire et une durée pour retarder chaque périodicité de votre workflow. Ce déclencheur ne permet pas les planifications avancées, par exemple, les heures spécifiques du jour, les minutes de l'heure et les jours de la semaine. Pour plus d'informations, voir [Créer, planifier et exécuter des tâches et des workflows récurrents avec le déclencheur Fenêtre glissante](../connectors/connectors-native-sliding-window.md).

<a name="schedule-actions"></a>

## <a name="schedule-actions"></a>Planifier des actions

Après chaque action dans votre workflow d’application logique, vous pouvez utiliser les actions Retarder et Retarder jusqu’à pour mettre en pause votre workflow en attendant l’exécution de l'action suivante.

* **Retarder** : Attendre le nombre spécifié d'unités de temps, par exemple des secondes, minutes, heures, jours, semaines ou mois avant d’exécuter l'action suivante. Pour plus d'informations, voir [ Retarder la prochaine action dans les workflows](../connectors/connectors-native-delay.md).

* **Retarder jusqu’à** : Attendre la date et l'heure spécifiées avant d’exécuter l'action suivante. Pour plus d'informations, voir [ Retarder la prochaine action dans les workflows](../connectors/connectors-native-delay.md).

## <a name="patterns-for-start-date-and-time"></a>Modèles pour la date et l’heure de début

<a name="start-time"></a>

Voici quelques modèles qui montrent comment vous pouvez contrôler la périodicité avec la date et l’heure de début, et comment le service Logic Apps exécute ces périodicités :

| Heure de début | Périodicité sans planification | Périodicité avec planification (déclencheur Périodicité uniquement) |
|------------|-----------------------------|----------------------------------------------------|
| {aucune} | Exécute la première charge de travail instantanément. <p>Exécute les charges de travail suivantes en fonction de la dernière heure d’exécution. | Exécute la première charge de travail instantanément. <p>Exécute les charges de travail suivantes en fonction de la planification spécifiée. |
| Heure de début dans le passé | Déclencheur de **périodicité** : Calcule le temps d’exécution en fonction de l’heure de début spécifiée et ignore les heures d’exécution passées. Exécute la première charge de travail à la prochaine heure d’exécution. <p>Exécute les charges de travail suivantes selon des calculs basés sur la dernière heure d’exécution. <p><p>Déclencheur **Fenêtre glissante** : Calcule le temps d’exécution en fonction de l’heure de début spécifiée et respecte les heures d’exécution passées. <p>Exécute les charges de travail suivantes selon des calculs basés sur l’heure de début spécifiée. <p><p>Pour une explication plus détaillée, consultez l’exemple fourni après ce tableau. | Exécute la première charge de travail *exactement* à l’heure de début, en fonction de la planification calculée à partir de l’heure de début. <p>Exécute les charges de travail suivantes en fonction de la planification spécifiée. <p>**Remarque :** Si vous spécifiez une récurrence avec une planification, mais sans spécifier d’heures ni de minutes, les exécutions suivantes sont calculées avec les heures ou les minutes (respectivement) depuis le moment de la première d’exécution. |
| Heure de début actuelle ou future | Exécute la première charge de travail à l’heure de début spécifiée. <p>Exécute les charges de travail suivantes selon des calculs basés sur la dernière heure d’exécution. | Exécute la première charge de travail *exactement* à l’heure de début, en fonction de la planification calculée à partir de l’heure de début. <p>Exécute les charges de travail suivantes en fonction de la planification spécifiée. <p>**Remarque :** Si vous spécifiez une récurrence avec une planification, mais sans spécifier d’heures ni de minutes, les exécutions suivantes sont calculées avec les heures ou les minutes (respectivement) depuis le moment de la première d’exécution. |
||||

> [!IMPORTANT]
> Lorsque les récurrences ne spécifient pas d’options de planification avancées, les récurrences ultérieures sont basées sur l’heure de la dernière exécution.
> Les heures de début de ces récurrences peuvent dériver en raison de facteurs tels que la latence lors des appels de stockage. Pour vous assurer que votre application logique ne manque pas de récurrence, en particulier quand la fréquence est en jours ou plus, utilisez l’une des options suivantes :
> 
> * Indiquez une heure de début pour la récurrence.
> 
> * Spécifiez les heures et les minutes d’exécution de la récurrence à l’aide des propriétés **À ces heures** et **À ces minutes**.
> 
> * Utilisez le [déclencheur de fenêtre glissante](../connectors/connectors-native-sliding-window.md) plutôt que le déclencheur de récurrence.

*Exemple d’heure de début passée et de périodicité, mais sans planification*

Prenons par exemple le 8 septembre 2017 à 13h comme date et heure actuelles. Vous pouvez choisir le 7 septembre 2017 à 14h00 comme date et heure de début (dans le passé), et une périodicité d’exécution tous les deux jours.

| Heure de début | Heure actuelle | Périodicité | Planifier |
|------------|--------------|------------|----------|
| 2017-09-**07**T14:00:00Z <br>(**07**-09-2017 à 14h00) | 2017-09-**08**T13:00:00Z <br>(**08**-09-2017 à 13h00) | Tous les deux jours | {aucune} |
|||||

Pour le déclencheur Périodicité, le moteur Logic Apps calcule les heures d’exécution en fonction de l’heure de début, ignore les heures d’exécution passées, utilise l’heure de début suivante pour la première exécution, puis calcule les exécutions futures en fonction de la dernière exécution.

Voici à quoi ressemble cette périodicité :

| Heure de début | Première heure d'exécution | Heures d’exécution suivantes |
|------------|----------------|------------------|
| **07**/09/2017 à 14h00 | **09**/09/2017 à 14h00 | **11**/09/2017 à 14h00 </br>**13**/09/2017 à 14h00 </br>**15**/09/2017 à 14h00 </br>Etc. |
||||

Par conséquent, peu importe si l’heure de début spécifiée remonte à loin ou pas (par exemple, **05**/09/2017 à 14h00 ou **01**/09/2017 à 14h00), votre première exécution utilise toujours la prochaine heure de début.

Pour le déclencheur Fenêtre glissante, le moteur Logic Apps calcule les heures d’exécution en fonction de l’heure de début, respecte les heures d’exécution passées, utilise l’heure de début de la première exécution, puis calcule les exécutions futures en fonction de l’heure de début.

Voici à quoi ressemble cette périodicité :

| Heure de début | Première heure d'exécution | Heures d’exécution suivantes |
|------------|----------------|------------------|
| **07**/09/2017 à 14h00 | **07**/09/2017 à 14h00 | **09**/09/2017 à 14h00 </br>**11**/09/2017 à 14h00 </br>**13**/09/2017 à 14h00 </br>**15**/09/2017 à 14h00 </br>Etc. |
||||

Par conséquent, peu importe si l’heure de début spécifiée remonte à loin ou pas (par exemple, **05**/09/2017 à 14h00 ou **01**/09/2017 à 14h00), votre première exécution utilise toujours l’heure de début spécifiée.

<a name="example-recurrences"></a>

## <a name="example-recurrences"></a>Exemples de périodicité

Voici divers exemples de périodicité que vous pouvez configurer pour les déclencheurs prenant en charge ces options :

| Déclencheur | Périodicité | Intervalle | Fréquence | Heure de début | Aux jours indiqués | Aux heures indiquées | Aux minutes indiquées | Remarque |
|---------|------------|----------|-----------|------------|---------------|----------------|------------------|------|
| Périodicité, <br>Fenêtre glissante | Exécution toutes les 15 minutes (sans date ni heure de début) | 15 | Minute | {aucune} | {non disponible} | {aucune} | {aucune} | Cette planification démarre immédiatement, puis calcule les périodicités suivantes en fonction de la dernière heure d’exécution. |
| Périodicité, <br>Fenêtre glissante | Exécution toutes les 15 minutes (avec date et heure de début) | 15 | Minute | *startDate*T*startTime*Z | {non disponible} | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées. Les périodicités suivantes sont ensuite calculées en fonction de la dernière heure d’exécution. |
| Périodicité, <br>Fenêtre glissante | Exécution toutes les heures, à l’heure spécifiée (avec date et heure de début) | 1 | Heure | *startDate*Thh:00:00Z | {non disponible} | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées. Les prochaines périodicités interviennent toutes les heures à la minute « 00 », valeur calculée à partir de l'heure de début. <p>Si la fréquence est « Semaine » ou « Mois », cette planification s’exécute respectivement un seul jour par semaine ou un seul jour par mois. |
| Périodicité, <br>Fenêtre glissante | Exécution toutes les heures, tous les jours (sans date ni heure de début) | 1 | Heure | {aucune} | {non disponible} | {aucune} | {aucune} | Cette planification démarre immédiatement et calcule les périodicités suivantes en fonction de la dernière heure d’exécution. <p>Si la fréquence est « Semaine » ou « Mois », cette planification s’exécute respectivement un seul jour par semaine ou un seul jour par mois. |
| Périodicité, <br>Fenêtre glissante | Exécution toutes les heures, tous les jours (avec date et heure de début) | 1 | Heure | *startDate*T*startTime*Z | {non disponible} | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées. Les périodicités suivantes sont ensuite calculées en fonction de la dernière heure d’exécution. <p>Si la fréquence est « Semaine » ou « Mois », cette planification s’exécute respectivement un seul jour par semaine ou un seul jour par mois. |
| Périodicité, <br>Fenêtre glissante | Exécution 15 minutes après l’heure, toutes les heures (avec date et heure de début) | 1 | Heure | *startDate*T00:15:00Z | {non disponible} | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées. Les prochaines périodicités se produisent à la marque des « 15 » minutes, valeur calculée à partir de l'heure de début, soit à 00:15, 1:15, 2:15, et ainsi de suite. |
| Périodicité | Exécution 15 minutes après l’heure, toutes les heures (sans date ni heure de début) | 1 | jour | {aucune} | {non disponible} | 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 | 15 | Cette planification s’exécute à 00h15, 1h15, 2h15, etc. Cette planification est en outre l’équivalent d’une fréquence de type « Heure » et d’une heure de début avec « 15 » minutes. |
| Périodicité | Exécution toutes les 15 minutes à la marque de minutes spécifiée (sans date ni heure de début). | 1 | jour | {aucune} | {non disponible} | 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 | 0, 15, 30, 45 | Cette planification ne démarre pas avant la marque de 15 minutes spécifiée suivante. |
| Périodicité | Exécution quotidienne à 8h00 *plus* la marque de minutes à partir du moment où vous enregistrez votre application logique | 1 | jour | {aucune} | {non disponible} | 8 | {aucune} | Sans date et heure de début, cette planification s’exécute en fonction de l’heure à laquelle vous enregistrez l’application logique (opération PUT). |
| Périodicité | Exécution quotidienne à 8h00 (avec date et heure de début) | 1 | jour | *startDate*T08:00:00Z | {non disponible} | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées. Les occurrences ultérieures s’exécutent quotidiennement à 8h00. | 
| Périodicité | Exécution quotidienne à 8 h (sans date et heure de début) | 1 | jour | {aucune} | {non disponible} | 8 | 00 | Cette planification s’exécute à 8 h tous les jours. |
| Périodicité | Exécution quotidienne à 8 h et à 16 h | 1 | jour | {aucune} | {non disponible} | 8, 16 | 0 | |
| Périodicité | Exécution quotidienne à 8h30, 8h45, 16h30 et 16h45 | 1 | jour | {aucune} | {non disponible} | 8, 16 | 30, 45 | |
| Périodicité | Exécution hebdomadaire le samedi à 17 h (sans date ni heure de début) | 1 | Week | {aucune} | Samedi | 17 | 0 | Cette planification s’exécute tous les samedis à 17h00. |
| Périodicité | Exécution hebdomadaire le samedi à 17 h (avec date et heure de début) | 1 | Week | *startDate*T17:00:00Z | Samedi | {aucune} | {aucune} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées, à savoir le 9 septembre 2017 à 17h00 dans cet exemple. Les exécutions périodiques suivantes ont lieu tous les samedis à 17h00. |
| Périodicité | Exécution les mardi et jeudi à 17h00 *plus* la marque de minutes à partir du moment où vous enregistrez votre application logique| 1 | Week | {aucune} | Mardi, Jeudi | 17 | {aucune} | |
| Périodicité | Exécution toutes les heures pendant les heures de travail | 1 | Week | {aucune} | Sélectionnez tous les jours, sauf le samedi et le dimanche. | Sélectionnez les heures de la journée souhaitées. | Sélectionnez les minutes de l’heure souhaitées. | Par exemple, si vos heures de travail sont de 8 h à 17 h, sélectionnez « 8, 9, 10, 11, 12, 13, 14, 15, 16, 17 » comme heures de la journée *plus* « 0 » pour les minutes. |
| Périodicité | Exécution une fois par jour le week-end | 1 | Week | {aucune} | Samedi, Dimanche | Sélectionnez les heures de la journée souhaitées. | Sélectionnez les minutes de l’heure souhaitées. | Cette planification s’exécute tous les samedis et dimanches aux heures spécifiées. |
| Périodicité | Exécution toutes les 15 minutes toutes les deux semaines le lundi uniquement | 2 | Week | {aucune} | Lundi | 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 | 0, 15, 30, 45 | Cette planification s’exécute un lundi sur deux à chaque marque de 15 minutes. |
| Périodicité | Exécution tous les mois | 1 | Month | *startDate*T*startTime*Z | {non disponible} | {non disponible} | {non disponible} | Cette planification ne démarre *pas avant* la date et l’heure de début spécifiées, puis calcule les prochaines périodicités selon la date et à l’heure de début. Si vous ne spécifiez pas de date et heure de début, cette planification utilise la date et heure de création. |
| Périodicité | Exécution toutes les heures un jour par mois | 1 | Month | {voir remarque} | {non disponible} | 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 | {voir remarque} | Si vous ne spécifiez pas de date et heure de début, cette planification utilise la date et heure de création. Pour contrôler les minutes pour la planification de la périodicité, spécifiez les minutes de l’heure, une heure de début ou utilisez l’heure de création. Par exemple, si l’heure de début ou l’heure de création est 8h25, cette planification s’exécute à 8h25, 9h25, 10h25, etc. |
|||||||||

<a name="run-once"></a>

## <a name="run-one-time-only"></a>Une seule exécution

Si vous voulez exécuter votre application logique une seule fois dans le futur, vous pouvez utiliser le modèle **Scheduler : Exécuter des tâches une fois**. Après avoir créé une application logique, mais avant d’ouvrir le Doncepteur Logic Apps, sous la section **Modèles**, dans la liste **Catégorie**, sélectionnez **Planifier**, puis sélectionnez ce modèle :

![Sélectionner le modèle « Planificateur : Exécuter des tâches une fois »](./media/concepts-schedule-automated-recurring-tasks-workflows/choose-run-once-template.png)

Ou, si vous pouvez démarrer votre application logique avec le déclencheur **Lors de la réception d’une demande HTTP - Demande**, passez l'heure de début comme paramètre du déclencheur. Pour la première action, utiliser l’action **Retarder jusqu’à - Planifier** et indiquez l’heure de début d’exécution de l’action suivante.

## <a name="next-steps"></a>Étapes suivantes

* [Créer, planifier et exécuter des tâches et des workflows récurrents avec le déclencheur Périodicité](../connectors/connectors-native-recurrence.md)
* [Créer, planifier et exécuter des tâches et des workflows récurrents avec le déclencheur Fenêtre glissante](../connectors/connectors-native-sliding-window.md)
* [Suspendre les workflows avec des actions Retarder](../connectors/connectors-native-delay.md)
