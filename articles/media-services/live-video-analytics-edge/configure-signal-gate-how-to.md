---
title: Configurer une porte de signal pour l’enregistrement vidéo basé sur les événements – Azure
description: Cet article fournit des conseils sur la configuration d’une porte de signal dans un graphe multimédia.
ms.topic: how-to
ms.date: 11/3/2020
ms.openlocfilehash: afcec7c03f1353f08b58311278f5a533e0c911bc
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410791"
---
# <a name="configure-a-signal-gate-for-event-based-video-recording"></a>Configurer une porte de signal pour l’enregistrement vidéo basé sur les événements

Dans un graphe multimédia, un [nœud processeur de porte de signal](media-graph-concept.md#signal-gate-processor) vous permet de transférer un média d’un nœud à un autre lorsque la porte est déclenchée par un événement. Une fois déclenchée, la porte s’ouvre, laissant le média circuler pendant une durée spécifiée. En l’absence d’événements pour déclencher la porte, celle-ci se ferme et le média cesse de circuler. Vous pouvez utiliser le processeur de porte de signal pour l’enregistrement vidéo basé sur les événements.

Cet article explique comment configurer un processeur de porte de signal.

## <a name="suggested-prereading"></a>Lecture préalable suggérée
-   [Graphe multimédia](media-graph-concept.md)
-   [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)


## <a name="problem"></a>Problème
Un utilisateur peut démarrer l’enregistrement à un moment donné avant ou après le déclenchement de la porte par un événement. L’utilisateur connaît la latence acceptable au sein de son système. Il souhaite ainsi spécifier la latence du processeur de porte de signal. Il souhaite également spécifier les durées minimale et maximale de son enregistrement, quel que soit le nombre de nouveaux événements reçus.
 
### <a name="use-case-scenario"></a>Cas d’utilisation
Supposons que vous souhaitiez enregistrer une vidéo chaque fois que la porte d’entrée de votre bâtiment s’ouvre. Vous souhaitez que l’enregistrement : 

- inclue les *X* secondes précédent l’ouverture de la porte ; 
- dure au moins *Y* secondes si la porte n’est pas rouverte ; 
- dure au plus *Z* secondes si la porte est ouverte à plusieurs reprises. 
 
Vous savez que le capteur de la porte a une latence de *K* secondes. Pour réduire le risque que des événements soient ignorés en raison d’une arrivée tardive, vous voulez qu’au moins *K* secondes puissent s’écouler jusqu’à leur arrivée.


## <a name="solution"></a>Solution

Pour résoudre le problème, modifiez les paramètres de votre processeur de porte de signal.

Pour configurer un processeur de porte de signal, utilisez les quatre paramètres suivants :
- fenêtre d’évaluation de l’activation ;
- décalage du signal d’activation ;
- fenêtre d’activation minimale ;
- fenêtre d’activation maximale.

Une fois déclenché, le processeur de porte de signal reste ouvert pendant la durée d’activation minimale. L’événement d’activation commence à l’horodatage de l’événement le plus ancien, plus le décalage du signal d’activation. 

Si le processeur de porte de signal est déclenché à nouveau alors qu’il est ouvert, le minuteur est remis à zéro et la porte reste ouverte pendant au moins la durée d’activation minimale. Le processeur de porte de signal ne reste jamais ouvert plus longtemps que la durée d’activation maximale. 

Un événement (événement 1) qui se produit avant un autre (événement 2) sur la base des horodatages du média, peut être ignoré si le système accuse un retard et si l’événement 1 arrive au processeur de porte de signal après l’événement 2. Si l’événement 1 n’arrive pas entre l’arrivée de l’événement 2 et la fenêtre d’évaluation de l’activation, l’événement 1 est ignoré. Il n’est pas transmis via le processeur de porte de signal. 

Des ID de corrélation sont définis pour chaque événement. Ces ID sont définis à partir de l’événement initial. Ils sont séquentiels pour chaque événement suivant.

> [!IMPORTANT]
> L’heure du média est basée sur l’horodatage d’un événement se produit dans le média. La séquence des événements qui arrivent à la porte de signal peut ne pas refléter la séquence des événements qui arrivent à l’heure du média.


### <a name="parameters-based-on-the-physical-time-that-events-arrive-at-the-signal-gate"></a>Paramètres en fonction du temps physique d’arrivée des événements à la porte de signal

* **minimumActivationTime (durée la plus brève possible d’un enregistrement)**  : nombre minimal de secondes pendant lesquelles le processeur de porte de signal reste ouvert après qu’il a été déclenché pour recevoir de nouveaux événements, sauf interruption par le paramètre maximumActivationTime.
* **maximumActivationTime (durée la plus longue possible d’un enregistrement)**  : nombre maximal de secondes, à partir de l’événement initial, pendant lesquelles le processeur de porte de signal reste ouvert après qu’il a été déclenché pour recevoir de nouveaux événements, quels que soient les événements reçus.
* **activationSignalOffset** : nombre de secondes entre l’activation du processeur de porte de signal et le début de l’enregistrement vidéo. Généralement, cette valeur est négative, car elle démarre l’enregistrement avant l’événement déclencheur.
* **activationEvaluationWindow** : à partir de l’événement déclencheur initial, nombre de secondes à l’issue desquelles un événement qui s’est produit avant l’événement initial, en temps du média, doit arriver au processeur de porte de signal avant d’être ignoré et considéré comme une arrivée tardive.

> [!NOTE]
> Une *arrivée tardive* est un événement qui arrive après la fin de la fenêtre d’évaluation de l’activation, mais avant l’événement initial en temps du média.

### <a name="limits-of-parameters"></a>Limites des paramètres

* **activationEvaluationWindow** : de 0 à 10 secondes.
* **activationSignalOffset** : de -1 minute à 1 minute.
* **minimumActivationTime** : de 1 seconde à 1 heure.
* **maximumActivationTime** : de 1 seconde à 1 heure.


Dans le cas d’usage, vous devez définir les paramètres comme suit :

* **activationEvaluationWindow** : *K* secondes.
* **activationSignalOffset** : *-X* secondes.
* **minimumActivationWindow** : *Y* secondes.
* **maximumActivationWindow** : *Z* secondes.


Voici un exemple de section de nœud **Processeur de porte de signal** dans une topologie de graphe multimédia pour les valeurs de paramètre suivantes :
* **activationEvaluationWindow** : 1 seconde
* **activationSignalOffset** : -5 secondes.
* **minimumActivationTime** : 20 secondes.
* **maximumActivationTime** : 40 secondes.

> [!IMPORTANT]
> Le [format de durée ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations
) est attendu pour chaque valeur de paramètre. Par exemple, PT1S = 1 seconde.


```
"processors":              
[
          {
            "@type": "#Microsoft.Media.MediaGraphSignalGateProcessor",
            "name": "signalGateProcessor",
            "inputs": [
              {
                "nodeName": "iotMessageSource"
              },
              {
                "nodeName": "rtspSource"
              }
            ],
            "activationEvaluationWindow": "PT1S",
            "activationSignalOffset": "-PT5S",
            "minimumActivationTime": "PT20S",
            "maximumActivationTime": "PT40S"
          }
]
```


À présent, considérez la façon dont cette configuration de processeur de porte de signal se comporte dans différents scénarios d’enregistrement.

### <a name="recording-scenarios"></a>Scénarios d’enregistrement

**Un événement d’une source (*activation normale*)**

Un processeur de porte de signal qui reçoit un événement entraîne un enregistrement qui démarre 5 secondes (signal d’activation = 5 secondes) avant que l’événement arrive à la porte. Le reste de l’enregistrement dure 20 secondes (durée d’activation minimale = 20 secondes), car aucun autre événement n’arrive avant la fin de la durée d’activation minimale pour redéclencher la porte.

Exemple de diagramme :
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/normal-activation.svg" alt-text="Diagramme montrant l’activation normale d’un événement d’une source.":::

* Durée de l’enregistrement = -décalage + minimumActivationTime = [E1 + décalage, E1 + minimumActivationTime]


**Deux événements d’une source (*activation redéclenchée*)**

Un processeur de porte de signal qui reçoit deux événements entraîne un enregistrement qui démarre 5 secondes (décalage du signal d’activation = 5 secondes) avant que l’événement arrive à la porte. Par ailleurs, l’événement 2 arrive 5 secondes après l’événement 1. Étant donné que l’événement 2 arrive avant la fin de la durée d’activation minimale de l’événement 1 (20 secondes), la porte est redéclenchée. Le reste de l’enregistrement dure 20 secondes (durée d’activation minimale = 20 secondes), car aucun autre événement n’arrive avant la fin de la durée d’activation minimale de l’événement 2 pour redéclencher la porte.

Exemple de diagramme :
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/retriggering-activation.svg" alt-text="Diagramme montrant l’activation redéclenchée de deux événements d’une source.":::

* Durée de l’enregistrement = -décalage + (arrivée de l’événement 2 - arrivée de l’événement 1) + minimumActivationTime


***N* événements d’une source (*activation maximale*)**

Un processeur de porte de signal qui reçoit *N* événements entraîne un enregistrement qui démarre 5 secondes (décalage du signal d’activation = 5 secondes) avant que le premier événement arrive à la porte. Comme chaque événement arrive avant la fin de la durée d’activation minimale de 20 secondes de l’événement précédent, la porte est continuellement redéclenchée. Elle reste ouverte pendant la durée d’activation maximale de 40 secondes après le premier événement. Ensuite, la porte se ferme et n’accepte plus aucun nouvel événement.

Exemple de diagramme :
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/maximum-activation.svg" alt-text="Diagramme montrant l’activation maximale de N événements d’une source.":::
 
* Durée de l’enregistrement = -décalage + maximumActivationTime

> [!IMPORTANT]
> Les diagrammes précédents présupposent que chaque événement arrive au même instant en temps physique et en temps du média. Autrement dit, ils présupposent qu’il n’y a pas d’arrivées tardives.

## <a name="next-steps"></a>Étapes suivantes

Essayez de suivre le [Tutoriel d’enregistrement vidéo basé sur les événements](event-based-video-recording-tutorial.md). Commencez par modifier le fichier [topology.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json). Modifiez les paramètres du nœud signalgateProcessor, puis suivez le reste du tutoriel. Examinez les enregistrements vidéo pour analyser l’effet des paramètres.



