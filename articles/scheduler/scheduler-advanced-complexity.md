---
title: Créer des planifications et des périodicités avancées pour les travaux
description: Découvrez comment créer des planifications et des périodicités avancées pour les travaux dans Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.suite: infrastructure-services
ms.topic: article
ms.date: 11/14/2018
ms.openlocfilehash: 5a74240e3f116121c0aaddd11c186e6e674ea26a
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368177"
---
# <a name="build-advanced-schedules-and-recurrences-for-jobs-in-azure-scheduler"></a>Créer des planifications et des périodicités avancées pour les travaux dans Azure Scheduler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

La planification constitue le cœur d’un travail [Azure Scheduler](../scheduler/scheduler-intro.md), car elle détermine quand et comment Azure Scheduler exécute le travail. Avec Scheduler, vous pouvez créer plusieurs planifications ponctuelles et récurrentes pour un travail. Les planifications ponctuelles se déclenchent une seule fois à un moment précis. Il s’agit en fait de planifications récurrentes qui ne s’exécutent qu’une seule fois. Les planifications récurrentes se déclenchent selon une fréquence définie. Cette flexibilité vous permet d’utiliser Scheduler dans divers scénarios d’entreprise, comme les exemples ci-après :

* **Nettoyage périodique des données** : créez un travail quotidien qui supprime tous les tweets qui datent de plus de trois mois.

* **Archivage des données** : créez un travail mensuel qui envoie (push) l’historique de facturation vers un service de sauvegarde.

* **Demande de données externes** : créez un travail qui s’exécute toutes les 15 minutes et tire (pull) un nouveau rapport météo de la NOAA.

* **Traitement des images** : créez un travail qui s’exécute tous les jours ouvrables, pendant les heures creuses, et utilise le cloud computing pour compresser les images chargées pendant la journée.

Cet article décrit des exemples de travaux que vous pouvez créer à l’aide de Scheduler et de l’[API REST Azure Scheduler](/rest/api/scheduler). Il fournit également la définition JSON (JavaScript Object Notation) de chaque planification. 

## <a name="supported-scenarios"></a>Scénarios pris en charge

Les exemples de cet article illustrent l’éventail de scénarios pris en charge par Azure Scheduler. Ils montrent comment créer des planifications pour divers modèles de comportement, notamment :

* Exécuter une seule fois à une date et une heure spécifiques
* Exécuter et répéter un certain nombre de fois
* Exécuter immédiatement et répéter
* Exécuter et répéter tous les *n* minutes, heures, jours, semaines ou mois, en commençant à un moment spécifique.
* Exécuter et répéter chaque semaine ou mois, mais uniquement certains jours de la semaine ou du mois.
* Exécuter et répéter plusieurs fois pendant une période spécifique. Par exemple, chaque mois le dernier vendredi et le dernier lundi, ou chaque jour à 5 h 15 et à 17 h 15.

Ces scénarios sont décrits plus en détail plus loin dans cet article.

<a name="create-scedule"></a>

## <a name="create-schedule-with-rest-api"></a>Créer une planification avec l’API REST

Pour créer une planification de base avec [l’API REST Azure Scheduler](/rest/api/scheduler), effectuez les étapes suivantes :

1. Inscrivez votre abonnement Azure auprès d’un fournisseur de ressources à l’aide de [l’opération Inscrire dans l’API REST Resource Manager](/rest/api/resources/providers). Le nom du fournisseur pour le service Azure Scheduler est **Microsoft.Scheduler** . 

1. Créez une collection de travaux en utilisant [l’opération Créer ou Mettre à jour pour les collections de travaux](/rest/api/scheduler/jobcollections) dans l’API REST Scheduler. 

1. Créez un travail à l’aide de [l’opération Créer ou Mettre à jour pour les travaux](/rest/api/scheduler/jobs/createorupdate). 

## <a name="job-schema-elements"></a>Éléments du schéma de travail

Ce tableau fournit une vue d’ensemble des principaux éléments JSON que vous pouvez utiliser pour configurer des planifications et des périodicités de travaux. 

| Élément | Obligatoire | Description | 
|---------|----------|-------------|
| **startTime** | Non | Valeur de chaîne DateHeure au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) qui spécifie quand le travail démarre la première fois dans une planification de base. <p>Pour les planifications complexes, le travail ne démarre pas avant **startTime** . | 
| **recurrence** | Non | Spécifie les règles de périodicité selon lesquelles le travail est exécuté. L’objet **recurrence** prend en charge les éléments suivants : **frequency** , **interval** , **schedule** , **count** et **endTime** . <p>Si vous définissez l’élément **recurrence** , vous devez également définir l’élément **frequency** . Les autres éléments **recurrence** sont facultatifs. |
| **frequency** | Oui, si vous définissez **recurrence** | Unité de temps entre les occurrences. Les valeurs prises en charge sont : « Minute », « Hour », « Day », « Week », « Month » et « Year ». | 
| **interval** | Non | Entier positif qui détermine le nombre d’unités de temps entre les occurrences, en fonction de l’élément **frequency** . <p>Par exemple, si **interval** a la valeur 10 et que **frequency** est défini sur « Week », le travail se répète toutes les 10 semaines. <p>Voici le plus grand nombre d’intervalles pour chaque fréquence : <p>- 18 mois <br>- 78 semaines <br>- 548 jours <br>- Pour les heures et les minutes, la plage est 1 <= < *interval* > <= 1 000. | 
| **schedule** | Non | Définit les changements de périodicité selon les minutes, heures, jours de la semaine et jours du mois spécifiés. | 
| **count** | Non | Entier positif qui spécifie le nombre de fois où le travail s’exécute avant de finir. <p>Par exemple, quand un travail quotidien a une valeur **count** égale à 7 et une date de début définie au lundi, le travail finit de s’exécuter le dimanche. Si la date de début est passée, la première exécution est calculée d’après l’heure de création. <p>Si la valeur **endTime** ou **count** n’est pas spécifiée, le travail s’exécute indéfiniment. Vous ne pouvez pas utiliser à la fois **count** et **endTime** dans le même travail, mais la règle qui finit en premier est appliquée. | 
| **endTime** | Non | Valeur de chaîne Date ou DateHeure au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) qui spécifie quand le travail arrête de s’exécuter. Vous pouvez définir une valeur **endTime** qui se trouve dans le passé. <p>Si la valeur **endTime** ou **count** n’est pas spécifiée, le travail s’exécute indéfiniment. Vous ne pouvez pas utiliser à la fois **count** et **endTime** dans le même travail, mais la règle qui finit en premier est appliquée. |
|||| 

Par exemple, ce schéma JSON décrit une planification et une périodicité de base pour un travail : 

```json
"properties": {
   "startTime": "2012-08-04T00:00Z", 
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "weekDays": ["Monday", "Wednesday", "Friday"],
         "hours": [10, 22]                      
      },
      "count": 10,       
      "endTime": "2012-11-04"
   },
},
``` 

*Valeurs Dates et DateTime*

* Les valeurs Dates dans les travaux Scheduler contiennent uniquement la date et respectent la [spécification ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

* Les valeurs DateTime dans les travaux Scheduler contiennent à la fois la date et l’heure, respectent la [spécification ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et ont par défaut le type UTC quand aucun décalage UTC n’est spécifié. 

Pour plus d’informations, consultez [Concepts, terminologie et entités](../scheduler/scheduler-concepts-terms.md).

<a name="start-time"></a>

## <a name="details-starttime"></a>Détails sur startTime

Ce tableau décrit comment **startTime** contrôle la façon dont un travail s’exécute :

| startTime | Aucune périodicité | Périodicité, aucune planification | Périodicité avec planification |
|-----------|---------------|-------------------------|--------------------------|
| **Aucune heure de début** | Exécuter une fois immédiatement. | Exécuter une fois immédiatement. Lancer ultérieurement les exécutions calculées à partir de la dernière exécution. | Exécuter une fois immédiatement. Lancer ultérieurement les exécutions en fonction d’une planification de périodicité. | 
| **Heure de début dans le passé** | Exécuter une fois immédiatement. | Calculer la première exécution suivante après l’heure de début, puis lancer l’exécution à ce moment. <p>Lancer ultérieurement les exécutions calculées à partir de la dernière exécution. <p>Consultez l’exemple après ce tableau. | Démarrer le travail *pas avant* l’heure de début spécifiée. La première occurrence dépend de la planification calculée à partir de l’heure de début. <p>Lancer ultérieurement les exécutions en fonction d’une planification de périodicité. | 
| **Heure de début dans le futur ou heure actuelle** | Exécuter une fois à l’heure de début spécifiée. | Exécuter une fois à l’heure de début spécifiée. <p>Lancer ultérieurement les exécutions calculées à partir de la dernière exécution. | Démarrer le travail *pas avant* l’heure de début spécifiée. La première occurrence dépend de la planification, calculée à partir de l’heure de début. <p>Lancer ultérieurement les exécutions en fonction d’une planification de périodicité. |
||||| 

Prenons cet exemple d’exécution avec ces conditions : une heure de début dans le passé avec une périodicité, mais aucune planification.

```json
"properties": {
   "startTime": "2015-04-07T14:00Z", 
   "recurrence": {
      "frequency": "Day",
      "interval": 2
   }
}
```

* La date actuelle est le 8 avril 2015 et l’heure actuelle est 13h00.

* La date et l’heure de début sont respectivement le 7 avril 2015 à 14h00, soit avant la date et l’heure actuelles.

* La périodicité est tous les deux jours.

1. Dans ces conditions, la première exécution a lieu le 9 avril 2015 à 14h00. 

   Scheduler calcule les occurrences d’exécution à partir de l’heure de début, ignore toutes les instances dans le passé et utilise l’instance suivante dans le futur. 
   Dans ce cas, **startTime** correspond au 7 avril 2015 à 14h00, si bien que l’instance suivante a lieu deux jours plus tard, c’est-à-dire le 9 avril 2015 à 14h00.

   La première exécution se produit au même moment que **startTime** soit défini au 05-04-2015 à 14 h ou au 01-04-2015 à 14 h. Après la première exécution, les exécutions ultérieures sont calculées en fonction de la planification. 
   
1. Les exécutions suivantes ont lieu dans cet ordre : 
   
   1. Le 11-04-2015 à 14 h
   1. Le 13-04-2015 à 14 h 
   1. Le 15-04-2015 à 14 h
   1. Et ainsi de suite...

1. Pour finir, quand un travail a une planification, mais que les heures et minutes ne sont pas spécifiées, ces valeurs sont définies par défaut aux heures et minutes de la première exécution, respectivement.

<a name="schedule"></a>

## <a name="details-schedule"></a>Détails sur schedule

Vous pouvez utiliser **schedule** pour *limiter* le nombre d’exécutions du travail. Par exemple, si un travail avec une valeur **frequency** définie sur « month » a une planification qui s’exécute uniquement le 31, le travail s’exécute uniquement pendant les mois dotés d’un 31e jour.

Vous pouvez également utiliser **schedule** pour *étendre* le nombre d’exécutions du travail. Par exemple, si un travail avec une valeur **frequency** définie sur "month" a une planification qui s’exécute les jours 1 et 2 du mois, le travail s’exécute le 1er et le 2ème jour du mois au lieu d’une seule fois par mois.

Si vous spécifiez plusieurs éléments de planification, l’ordre d’évaluation va du plus grand au plus petit : numéro de semaine, jour du mois, jour de la semaine, heure et minute.

Le tableau suivant décrit les éléments schedule en détail :

| Nom JSON | Description | Valeurs valides |
|:--- |:--- |:--- |
| **minutes** |Minutes de l’heure où le travail s’exécute. |Tableau d’entiers. |
| **hours** |Heures de la journée où le travail s’exécute. |Tableau d’entiers. |
| **weekDays** |Jours de la semaine où le travail s’exécute. Peut être spécifié uniquement avec une fréquence hebdomadaire. |Tableau de l’une des valeurs suivantes (la taille de tableau maximale est 7) :<br />- "Monday"<br />- "Tuesday"<br />- "Wednesday"<br />- "Thursday"<br />- "Friday"<br />- "Saturday"<br />- "Sunday"<br /><br />Ne respecte pas la casse. |
| **monthlyOccurrences** |Détermine les jours du mois où le travail s’exécute. Peut être spécifié uniquement avec une fréquence mensuelle. |Tableau d’objets **monthlyOccurrence**  :<br /> `{ "day": day, "occurrence": occurrence}`<br /><br /> **day** est le jour de la semaine où le travail s’exécute. Par exemple, *{Sunday}* correspond à tous les dimanche du mois. Obligatoire.<br /><br />**occurrence** est l’occurrence du jour au cours du mois. Par exemple, *{Sunday, -1}* correspond au dernier dimanche du mois. facultatif. |
| **monthDays** |Jour du mois où le travail s’exécute. Peut être spécifié uniquement avec une fréquence mensuelle. |Tableau des valeurs suivantes :<br />- Toute valeur <= -1 et >= -31<br />- Toute valeur >= 1 et <= 31|

## <a name="examples-recurrence-schedules"></a>Exemples : planifications de périodicité

Les exemples suivants montrent différentes planifications de périodicité. Ils se concentrent sur l’objet de planification et ses sous-éléments.

Ces planifications partent du principe que **interval** a la valeur 1\., et que les valeurs de **frequency** pour les valeurs spécifiées dans **schedule** sont correctes. Par exemple, vous ne pouvez pas utiliser une valeur **frequency** "day" et avoir une modification **monthDays** dans **schedule** . Nous décrivons ces restrictions plus haut dans l’article.

| Exemple | Description |
|:--- |:--- |
| `{"hours":[5]}` |Exécution à 5h00 tous les jours.<br /><br />Scheduler fait correspondre chaque valeur dans "hours" avec chaque valeur dans "minutes", une par une, pour créer une liste de toutes les fois où le travail s’exécute. |
| `{"minutes":[15], "hours":[5]}` |Exécution à 5h15 tous les jours. |
| `{"minutes":[15], "hours":[5,17]}` |Exécution à 5h15 et 17h15 tous les jours. |
| `{"minutes":[15,45], "hours":[5,17]}` |Exécution à 5h15, 5h45, 17h15 et 17h45 tous les jours. |
| `{"minutes":[0,15,30,45]}` |Exécution toutes les 15 minutes. |
| `{hours":[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]}` |Exécution toutes les heures.<br /><br />Ce travail s'exécute toutes les heures. La minute est contrôlée par la valeur de **startTime** , si elle est spécifiée. Si aucune valeur **startTime** n’est spécifiée, les minutes sont contrôlées par l’heure de création. Par exemple, si l’heure de début ou l’heure de création (selon le cas) est 00h25, le travail s’exécute à 00h25, 01h25, 02h25, ..., 23h25.<br /><br />La planification est la même qu’un travail avec une valeur **frequency** définie sur « hour », une valeur **interval** égale à 1 et aucune value **schedule** . La différence est que vous pouvez utiliser cette planification avec des valeurs **frequency** et **interval** différentes pour créer d’autres travaux. Par exemple, si **frequency** est définie sur "month", la planification s’exécute une seule fois par mois, plutôt que tous les jours (si **frequency** est définie sur "day"). |
| `{minutes:[0]}` |Exécution toutes les heures sur l’heure.<br /><br />Ce travail s’exécute également toutes les heures, mais sur l’heure (par exemple, minuit, 1h, 2h, et ainsi de suite). Cette planification équivaut à un travail avec une valeur **frequency** définie sur « hour », une valeur **startTime** de zéro minute, et aucune valeur **schedule** , si la fréquence est « day ». Si la valeur **frequency** est définie sur "week" » ou "month", la planification s’exécute respectivement un seul jour par semaine ou un seul jour par mois. |
| `{"minutes":[15]}` |Exécution 15 minutes après l’heure toutes les heures.<br /><br />Ce travail s’exécute toutes les heures, en commençant à 00h15, 01h15, 02h15, et ainsi de suite. Il se termine à 23h15. |
| `{"hours":[17], "weekDays":["saturday"]}` |Exécution à 17h00 le samedi chaque semaine. |
| `{hours":[17], "weekDays":["monday", "wednesday", "friday"]}` |Exécution à 17h00 le lundi, le mercredi et le vendredi chaque semaine. |
| `{"minutes":[15,45], "hours":[17], "weekDays":["monday", "wednesday", "friday"]}` |Exécution à 17h15 et 17h45 le lundi, le mercredi et le vendredi chaque semaine. |
| `{"hours":[5,17], "weekDays":["monday", "wednesday", "friday"]}` |Exécution à 5h00 et 17h00 le lundi, le mercredi et le vendredi chaque semaine. |
| `{"minutes":[15,45], "hours":[5,17], "weekDays":["monday", "wednesday", "friday"]}` |Exécution à 5h15, 5h45, 17h15 et 17h45 le lundi, le mercredi et le vendredi chaque semaine. |
| `{"minutes":[0,15,30,45], "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` |Exécution toutes les 15 minutes les jours de semaine. |
| `{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` |Exécution toutes les 15 minutes les jours de semaine entre 9h00 et 16h45. |
| `{"weekDays":["sunday"]}` |Exécution le dimanche à l’heure de début. |
| `{"weekDays":["tuesday", "thursday"]}` |Exécution le mardi et le jeudi à l’heure de début. |
| `{"minutes":[0], "hours":[6], "monthDays":[28]}` |Exécution à 6h00 le 28e jour de chaque mois (en supposant que la valeur **frequency** est définie sur « month »). |
| `{"minutes":[0], "hours":[6], "monthDays":[-1]}` |Exécution à 6h00 le dernier jour du mois.<br /><br />Si vous souhaitez exécuter un travail le dernier jour du mois, utilisez -1 au lieu de jour 28, 29, 30 ou 31. |
| `{"minutes":[0], "hours":[6], "monthDays":[1,-1]}` |Exécution à 6h00 le premier et le dernier jour de chaque mois. |
| `{monthDays":[1,-1]}` |Exécution le premier et le dernier jour de chaque mois à l’heure de début. |
| `{monthDays":[1,14]}` |Exécution le premier et le 14e jour de chaque mois à l’heure de début. |
| `{monthDays":[2]}` |Exécution le deuxième jour du mois à l’heure de début. |
| `{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` |Exécution le premier vendredi de chaque mois à 5h00. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` |Exécution le premier vendredi de chaque mois à l’heure de début. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":-3}]}` |Exécution le troisième vendredi à partir de la fin du mois, chaque mois, à l’heure de début. |
| `{"minutes":[15], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` |Exécution le premier et le dernier vendredi de chaque mois à 5h15. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` |Exécution le premier et le dernier vendredi de chaque mois à l’heure de début. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":5}]}` |Exécution le cinquième vendredi de chaque mois à l’heure de début.<br /><br />S’il n’y a pas de cinquième vendredi dans un mois, le travail ne s’exécute pas. Vous pouvez utiliser -1 plutôt que 5 pour l’occurrence si vous souhaitez exécuter le travail le dernier vendredi du mois. |
| `{"minutes":[0,15,30,45], "monthlyOccurrences":[{"day":"friday", "occurrence":-1}]}` |Exécution toutes les 15 minutes le dernier vendredi du mois. |
| `{"minutes":[15,45], "hours":[5,17], "monthlyOccurrences":[{"day":"wednesday", "occurrence":3}]}` |Exécution à 5h15, 5h45, 17h15 et 17h45 le troisième mercredi de chaque mois. |

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)
* [Limites, valeurs par défaut et codes d’erreur d’Azure Scheluler](scheduler-limits-defaults-errors.md)