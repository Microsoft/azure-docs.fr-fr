---
title: Déboguer votre application dans Eclipse
description: Améliorez la fiabilité et les performances de vos services en les développant et en procédant à leur débogage dans Eclipse sur un cluster de développement local.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-java
ms.openlocfilehash: d321e0c10b66a15e6cb309cefe711602fa12957c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91534107"
---
# <a name="debug-your-java-service-fabric-application-using-eclipse"></a>Débogage de votre application Java Service Fabric avec Eclipse
> [!div class="op_single_selector"]
> * [Visual Studio/CSharp](service-fabric-debugging-your-application.md) 
> * [Eclipse/Java](service-fabric-debugging-your-application-java.md)
> 

1. Démarrez un cluster de développement local en suivant les étapes de la section [Configuration de votre environnement de développement Service Fabric](service-fabric-get-started-linux.md).

2. Mettez à jour entryPoint.sh du service que vous souhaitez déboguer, afin qu’il démarre le processus java avec les paramètres de débogage à distance. Ce fichier se trouve à l’emplacement suivant : `ApplicationName\ServiceNamePkg\Code\entrypoint.sh`. Le port 8001 est défini pour le débogage dans cet exemple.

    ```sh
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar myapp.jar
    ```
3. Mettez à jour le manifeste de l’application en définissant le nombre d’instances ou le nombre de réplicas pour le service en cours de débogage sur 1. Ce paramètre évite les conflits pour le port utilisé pour le débogage. Par exemple, pour les services sans état, définissez `InstanceCount="1"` et pour les services avec état, définissez les tailles cible et de jeu de réplicas minimales sur 1 comme suit : `TargetReplicaSetSize="1" MinReplicaSetSize="1"`.

4. Déployez l’application.

5. Dans l’IDE Eclipse, sélectionnez **Exécuter-> Déboguer des Configurations-> Application Java distante et entrez les propriétés de connexion**, puis définissez les propriétés comme suit :

   ```
   Host: ipaddress
   Port: 8001
   ```
6.  Définissez des points d’arrêt sur les points de votre choix et déboguez l’application.

Si l’application se bloque, vous pouvez également activer le vidage de la mémoire de travail. Exécutez `ulimit -c` dans un interpréteur de commandes et si elle renvoie la valeur 0, les vidages de la mémoire de travail ne sont pas activés. Pour activer un nombre illimité de vidages de la mémoire de travail, exécutez la commande suivante : `ulimit -c unlimited`. Vous pouvez également vérifier l’état à l’aide de la commande `ulimit -a`.  Si vous souhaitez mettre à jour le chemin d’accès de la génération des vidages de la mémoire de travail, exécutez `echo '/tmp/core_%e.%p' | sudo tee /proc/sys/kernel/core_pattern`. 

### <a name="next-steps"></a>Étapes suivantes

* [Collecter les journaux avec Diagnostics Azure pour Linux](./service-fabric-diagnostics-event-aggregation-lad.md).
* [Surveiller et diagnostiquer les services localement](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md).
