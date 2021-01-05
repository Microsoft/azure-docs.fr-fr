---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/30/2020
ms.author: alkohli
ms.openlocfilehash: 92ccb6127e624ace9e719ffd23324b3a1b971f72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89272178"
---
Sur un appareil Azure Stack Edge sur lequel le rôle de calcul est configuré, vous pouvez dépanner ou surveiller l’appareil à l’aide de deux jeux de commandes distincts.

- À l’aide des commandes `iotedge`. Ces commandes sont disponibles pour les opérations de base pour votre appareil.
- À l’aide des commandes `dkrdbe`. Ces commandes sont disponibles pour un ensemble complet d’opérations pour votre appareil.

Pour exécuter les jeux de commandes ci-dessus, vous devez [vous connecter à l’interface PowerShell](#connect-to-the-powershell-interface).

### <a name="use-iotedge-commands"></a>Utiliser les commandes `iotedge`

Pour afficher la liste des commandes disponibles, [connectez-vous à l’interface PowerShell](#connect-to-the-powershell-interface) et utilisez la fonction `iotedge`.

```powershell
[10.100.10.10]: PS>iotedge -?                                                                                                                                                                                                 Usage: iotedge COMMAND

Commands:
   check
   list
   logs
   restart

[10.100.10.10]: PS>
```

Le tableau ci-après contient une brève description des commandes disponibles pour `iotedge` :

|command  |Description |
|---------|---------|
|`check`     | Effectuer des contrôles automatisés de problèmes de configuration et de connectivité courants       |
|`list`     | Faire la liste des modules         |
|`logs`     | Extraire les journaux d’un module        |
|`restart`     | Arrêter et redémarrer un module         |


### <a name="use-dkrdbe-commands"></a>Utiliser les commandes `dkrdbe`

Pour afficher la liste des commandes disponibles, [connectez-vous à l’interface PowerShell](#connect-to-the-powershell-interface) et utilisez la fonction `dkrdbe`.

```powershell
[10.100.10.10]: PS>dkrdbe -?
Usage: dkrdbe COMMAND

Commands:
   image [prune]
   images
   inspect
   login
   logout
   logs
   port
   ps
   pull
   start
   stats
   stop
   system [df]
   top

[10.100.10.10]: PS>
```
Le tableau ci-après contient une brève description des commandes disponibles pour `dkrdbe` :

|command  |Description |
|---------|---------|
|`image`     | Gérer les images Pour supprimer les images non utilisées, utilisez : `dkrdbe image prune -a -f`       |
|`images`     | Répertorier des images         |
|`inspect`     | Retourner des informations détaillées sur les objets Docker         |
|`login`     | Se connecter à un registre Docker         |
|`logout`     | Se déconnecter d’un registre Docker         |
|`logs`     | Extraire les journaux d’un conteneur        |
|`port`     | Répertorier des mappages de port ou un mappage spécifique pour le conteneur        |
|`ps`     | Répertorier les conteneurs        |
|`pull`     | Extraire une image ou un référentiel à partir d’un registre         |
|`start`     | Démarrer un ou plusieurs conteneurs arrêtés         |
|`stats`     | Afficher un flux en direct des statistiques d’utilisation des ressources de conteneurs         |
|`stop`     | Arrêter un ou plusieurs conteneurs en cours d’exécution        |
|`system`     | Gérer Docker         |
|`top`     | Afficher les processus en cours d’exécution d’un conteneur         |

Pour obtenir de l’aide pour n’importe quelle commande disponible, utilisez `dkrdbe <command-name> --help`.

Par exemple, pour comprendre l’utilisation de la commande `port`, tapez :

```powershell
[10.100.10.10]: P> dkrdbe port --help

Usage:  dkr port CONTAINER [PRIVATE_PORT[/PROTO]]

List port mappings or a specific mapping for the container
[10.100.10.10]: P> dkrdbe login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
[10.100.10.10]: PS>
```

Les commandes disponibles pour la fonction `dkrdbe` utilisent les mêmes paramètres que ceux utilisés pour les commandes docker normales. Pour les options et les paramètres utilisés avec la commande docker, accédez à [Utiliser la ligne de commande Docker](https://docs.docker.com/engine/reference/commandline/docker/).

### <a name="to-check-if-the-module-deployed-successfully"></a>Pour vérifier si le module a été déployé avec succès

Les modules de calcul sont des conteneurs avec une logique métier implémentée. Pour vérifier si un module de calcul a été déployé avec succès, exécutez la commande `ps` et vérifiez l’exécution du conteneur (correspondant au module de calcul).

Pour obtenir la liste de tous les conteneurs (y compris ceux en pause), exécutez la commande `ps -a`.

```powershell
[10.100.10.10]: P> dkrdbe ps -a
CONTAINER ID        IMAGE                                                COMMAND                   CREATED             STATUS              PORTS                                                                  NAMES
d99e2f91d9a8        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  movefile
0a06f6d605e9        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  filemove
2f8a36e629db        mcr.microsoft.com/azureiotedge-hub:1.0               "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days           0.0.0.0:443->443/tcp, 0.0.0.0:5671->5671/tcp, 0.0.0.0:8883->8883/tcp   edgeHub
acce59f70d60        mcr.microsoft.com/azureiotedge-agent:1.0             "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days                                                                                  edgeAgent
[10.100.10.10]: PS>
```

Si une erreur s’est produite lors de la création de l’image de conteneur ou lors de l’extraction de l’image, exécutez `logs edgeAgent`.  `EdgeAgent` est le conteneur de runtime IoT Edge qui est responsable de la mise en service des autres conteneurs.

Étant donné que `logs edgeAgent` vide les journaux, un bon moyen de voir les erreurs récentes est d’utiliser l’option `--tail 20`.


```powershell
[10.100.10.10]: PS>dkrdbe logs edgeAgent --tail 20
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Util.Uds.HttpUdsMessageHandler] - Connected socket /var/run/iotedge/mgmt.sock
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Util.Uds.HttpUdsMessageHandler] - Sending request http://mgmt.sock/modules?api-version=2018-06-28
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Agent] - Getting edge agent config...
2019-02-28 23:38:23.464 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Agent] - Obtained edge agent config
2019-02-28 23:38:23.469 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Edgelet.ModuleManagementHttpClient] - Received a valid Http response from unix:///var/run/iotedge/mgmt.soc
k for List modules
--------------------CUT---------------------
--------------------CUT---------------------
08:28.1007774+00:00","restartCount":0,"lastRestartTimeUtc":"2019-02-26T20:08:28.1007774+00:00","runtimeStatus":"running","version":"1.0","status":"running","restartPolicy":"always
","type":"docker","settings":{"image":"edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64","imageHash":"sha256:47778be0602fb077d7bc2aaae9b0760fbfc7c058bf4df192f207ad6cbb96f7cc","c
reateOptions":"{\"HostConfig\":{\"Binds\":[\"/home/hcsshares/share4-dl460:/home/input\",\"/home/hcsshares/share4-iot:/home/output\"]}}"},"env":{}}
2019-02-28 23:38:28.480 +00:00 [DBG] [Microsoft.Azure.Devices.Edge.Agent.Core.Planners.HealthRestartPlanner] - HealthRestartPlanner created Plan, with 0 command(s).
```

### <a name="to-get-container-logs"></a>Pour obtenir les journaux d’activité de conteneur

Pour obtenir les journaux d'activité d’un conteneur spécifique, répertoriez d’abord le conteneur et obtenez les journaux du conteneur qui vous intéresse.

1. [Connectez-vous à l’interface PowerShell](#connect-to-the-powershell-interface).
2. Pour obtenir la liste des conteneurs en cours d’exécution, exécutez la commande `ps`.

    ```powershell
    [10.100.10.10]: P> dkrdbe ps
    CONTAINER ID        IMAGE                                                COMMAND                   CREATED             STATUS              PORTS                                                                  NAMES
    d99e2f91d9a8        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  movefile
    0a06f6d605e9        edgecompute.azurecr.io/filemovemodule2:0.0.1-amd64   "dotnet FileMoveModuâ€¦"    2 days ago          Up 2 days                                                                                  filemove
    2f8a36e629db        mcr.microsoft.com/azureiotedge-hub:1.0               "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days           0.0.0.0:443->443/tcp, 0.0.0.0:5671->5671/tcp, 0.0.0.0:8883->8883/tcp   edgeHub
    acce59f70d60        mcr.microsoft.com/azureiotedge-agent:1.0             "/bin/sh -c 'echo \"$â€¦"   2 days ago          Up 2 days                                                                                  edgeAgent
    ```

3. Notez l’ID de conteneur du conteneur contenant les journaux dont vous avez besoin.

4. Pour obtenir les journaux d’un conteneur spécifique, exécutez la commande `logs` en renseignant l’ID du conteneur.

    ```powershell
    [10.100.10.10]: PS>dkrdbe logs d99e2f91d9a8
    02/26/2019 18:21:45: Info: Opening module client connection.
    02/26/2019 18:21:46: Info: Initializing with input: /home/input, output: /home/output.
    02/26/2019 18:21:46: Info: IoT Hub module client initialized.
    02/26/2019 18:22:24: Info: Received message: 1, SequenceNumber: 0 CorrelationId: , MessageId: 081886a07e694c4c8f245a80b96a252a Body: [{"ChangeType":"Created","ShareRelativeFilePath":"\\__Microsoft Data Box Edge__\\Upload\\Errors.xml","ShareName":"share4-dl460"}]
    02/26/2019 18:22:24: Info: Moving input file: /home/input/__Microsoft Data Box Edge__/Upload/Errors.xml to /home/output/__Microsoft Data Box Edge__/Upload/Errors.xml
    02/26/2019 18:22:24: Info: Processed event.
    02/26/2019 18:23:38: Info: Received message: 2, SequenceNumber: 0 CorrelationId: , MessageId: 30714d005eb048e7a4e7e3c22048cf20 Body: [{"ChangeType":"Created","ShareRelativeFilePath":"\\f [10]","ShareName":"share4-dl460"}]
    02/26/2019 18:23:38: Info: Moving input file: /home/input/f [10] to /home/output/f [10]
    02/26/2019 18:23:38: Info: Processed event.
    ```

### <a name="to-monitor-the-usage-statistics-of-the-device"></a>Pour surveiller les statistiques d’utilisation de l’appareil

Pour surveiller la mémoire, l’utilisation du processeur et les E/S sur l’appareil, utilisez la commande `stats`.

1. [Connectez-vous à l’interface PowerShell](#connect-to-the-powershell-interface).
2. Exécutez la commande `stats` pour désactiver la diffusion en direct et extraire uniquement le premier résultat.

   ```powershell
   dkrdbe stats --no-stream
   ```

   L’exemple suivant montre comment utiliser cette cmdlet :

    ```
    [10.100.10.10]: P> dkrdbe stats --no-stream
    CONTAINER ID        NAME          CPU %         MEM USAGE / LIMIT     MEM %         NET I/O             BLOCK I/O           PIDS
    d99e2f91d9a8        movefile      0.0           24.4MiB / 62.89GiB    0.04%         751kB / 497kB       299kB / 0B          14
    0a06f6d605e9        filemove      0.00%         24.11MiB / 62.89GiB   0.04%         679kB / 481kB       49.5MB / 0B         14
    2f8a36e629db        edgeHub       0.18%         173.8MiB / 62.89GiB   0.27%         4.58MB / 5.49MB     25.7MB / 2.19MB     241
    acce59f70d60        edgeAgent     0.00%         35.55MiB / 62.89GiB   0.06%         2.23MB / 2.31MB     55.7MB / 332kB      14
    [10.100.10.10]: PS>
    ```

