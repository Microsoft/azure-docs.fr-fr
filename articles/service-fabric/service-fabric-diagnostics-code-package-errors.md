---
title: Diagnostiquer les erreurs de package de code courantes à l’aide de Service Fabric
description: Découvrez comment résoudre les erreurs de package de code les plus courantes avec Azure Service Fabric
author: grzuber
ms.topic: article
ms.date: 05/09/2019
ms.author: grzuber
ms.openlocfilehash: 983d45a7a240701fa6441d2d9edeeda16f1ed18b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86256490"
---
# <a name="diagnose-common-code-package-errors-by-using-service-fabric"></a>Diagnostiquer les erreurs de package de code courantes à l’aide de Service Fabric

Cet article décrit ce que signifie l’interruption inattendue d’un package de code. Il fournit des informations sur les causes possibles des codes d’erreur courants, ainsi que des étapes de résolution des problèmes.

## <a name="when-does-a-process-or-container-terminate-unexpectedly"></a>Quand est-ce qu’un processus ou conteneur s’arrête de façon inattendue ?

Quand Azure Service Fabric reçoit une requête de démarrage d’un package de code, il commence à préparer l’environnement sur le système local en fonction des options définies dans les manifestes de l’application et du service. Ces préparatifs peuvent inclure la réservation de points de terminaison ou de ressources réseau, la configuration de règles de pare-feu ou la définition de contraintes de gouvernance des ressources. 

Une fois que l’environnement a été configuré correctement, Service Fabric tente d’apporter le package de code. Cette étape est considérée comme réussie si le système d’exploitation ou le runtime du conteneur signale que le processus ou conteneur a été activé avec succès. Si l’activation a échoué, vous devriez voir un message de contrôle d’intégrité dans SFX similaire à ce qui suit :

```
There was an error during CodePackage activation. Service host failed to activate. Error: 0xXXXXXXXX
```

Une fois que le package de code a bien été correctement activé, Service Fabric commence la surveillance de sa durée de vie. À ce stade, un processus ou conteneur peut se terminer à tout moment pour plusieurs raisons. Par exemple, il se peut qu’il n’ait pas pu initialiser une DLL ou que le système d’exploitation ait dépassé l’espace du segment de mémoire du bureau. Si votre package de code s’est arrêté, vous devriez voir le message de contrôle d’intégrité dans SFX suivant :

```
The process/container terminated with exit code: XXXXXXXX. Please look at your application logs/dump or debug your code package for more details. For information about common termination errors, please visit https://aka.ms/service-fabric-termination-errors
```

Le code de sortie dans ce message d’intégrité est le seul indice que le processus ou le conteneur fournit sur la raison pour laquelle il s’est terminé. Il peut être généré par n’importe quel niveau de la pile. Par exemple, ce code de sortie peut être lié à une erreur de système d’exploitation ou à un problème .NET, ou il a peut-être été déclenché par votre code. Utilisez cet article comme point de départ pour le diagnostic de la source des codes de sortie de terminaison et des solutions possibles. Mais gardez à l’esprit qu’il s’agit de solutions générales pour des scénarios courants qui peuvent ne pas s’appliquer à l’erreur que vous rencontrez.

## <a name="how-can-i-tell-if-service-fabric-terminated-my-code-package"></a>Comment puis-je savoir si Service Fabric a interrompu mon package de code ?

Service Fabric peut être responsable de l’interruption de votre package de code pour diverses raisons. Par exemple, il peut décider de placer le package de code sur un autre nœud à des fins d’équilibrage de charge. Vous pouvez vérifier que Service Fabric mis fin à votre package de code si vous voyez l’un des codes de sortie dans le tableau suivant.

>[!NOTE]
> Si votre processus ou votre conteneur se termine avec un code de sortie autre que ceux dans le tableau suivant, Service Fabric n’en est pas responsable.

Code de sortie | Description
--------- | -----------
7147 | Indique que Service Fabric a arrêté le processus ou le conteneur de façon appropriée en lui envoyant un signal Ctrl + C.
7148 | Indique que Service Fabric a arrêté le processus ou le conteneur. Parfois, ce code d’erreur indique que le processus ou le conteneur n’a pas répondu dans un délai raisonnable après l’envoi d’un signal Ctrl + C, et qu’il a donc dû être arrêté de façon forcée.


## <a name="other-common-error-codes-and-their-potential-fixes"></a>Autres codes d’erreur courants et corrections éventuelles

Code de sortie | Valeur hexadécimale | Description courte | Cause racine | Correctif potentiel
--------- | --------- | ----------------- | ---------- | -------------
3221225794 | 0xc0000142 | STATUS_DLL_INIT_FAILED | Cette erreur peut potentiellement signifier que la machine était à court d’espace dans le segment de mémoire. Cette cause est probable si vous avez un grand nombre de processus appartenant à votre application exécutée sur le nœud. | Si votre programme n’a pas été conçu pour répondre aux signaux Ctrl + C, vous pouvez activer le paramètre **« EnableActivateNoWindow »** dans le manifeste de cluster. L’activation de ce paramètre signifie que votre package de code s’exécutera sans fenêtre GUI et ne recevra pas de signaux Ctrl + C. Cette action réduit également la quantité d’espace dans le segment de mémoire du bureau consommée par chaque processus. Si votre package de code a besoin de recevoir les signaux Ctrl + C, vous pouvez augmenter la taille de tas de votre nœud.
3762504530 | 0xe0434352 | N/A | Cette valeur représente le code d’erreur pour une exception non gérée dans le code managé (autrement dit, .NET). | Ce code de sortie indique que votre application a levé une exception qui reste non gérée et qui a mis fin au processus. En guise de première étape pour déterminer ce qui a déclenché cette erreur, déboguez les journaux et les fichiers de vidage de votre application.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [diagnostic des autres scénarios courants](service-fabric-diagnostics-common-scenarios.md).
* Obtenez une vue d’ensemble plus détaillée des journaux de Azure Monitor et de ce qu’ils offrent en lisant [Vue d’ensemble Azure Monitor](../azure-monitor/overview.md).
* Découvrez plus en détail la [création d’alertes](../azure-monitor/platform/alerts-overview.md) dans les journaux Azure Monitor pour faciliter la détection et les diagnostics.
* Familiarisez-vous avec les fonctionnalités de [requête et de recherche dans les journaux](../azure-monitor/log-query/log-query-overview.md) proposées par Journaux Azure Monitor.
