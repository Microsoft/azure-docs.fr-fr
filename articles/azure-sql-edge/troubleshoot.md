---
title: Résoudre les problèmes de déploiement d’Azure SQL Edge
description: Découvrez comment résoudre les problèmes possibles lors du déploiement d’Azure SQL Edge
keywords: SQL Edge, dépannage, problèmes de déploiement
services: sql-edge
ms.service: sql-edge
ms.topic: troubleshooting
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 517fed0dd9eb1736344546bde9f79e52ee17182f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333101"
---
# <a name="troubleshooting-azure-sql-edge-deployments"></a>Résoudre les problèmes de déploiement d’Azure SQL Edge 

Cet article fournit des informations sur les problèmes que vous pouvez rencontrer quand vous déployez et utilisez des conteneurs Azure SQL Edge, puis explique certaines techniques de dépannage pour vous aider à résoudre ces problèmes. 

Azure SQL Edge prend en charge deux modes de déploiement : 
- Déploiement connecté via Azure IoT Edge : Azure SQL Edge est disponible sur la Place de marché Azure et peut être déployé en tant que module pour [Azure IoT Edge](../iot-edge/about-iot-edge.md). Pour plus d'informations, consultez [Déployer Azure SQL Edge](deploy-portal.md).<br>

- Déploiement déconnecté : Les images conteneur Azure SQL Edge peuvent être tirées du hub Docker, puis déployées en tant que conteneur Docker autonome ou sur un cluster Kubernetes. Pour plus d’informations, consultez [Déployer Azure SQL Edge avec Docker](disconnected-deployment.md) et [Déployer un conteneur Azure SQL Edge dans Kubernetes](deploy-kubernetes.md).

## <a name="troubleshooting-iot-edge-device-and-deployments"></a>Résolution des problèmes avec les déploiements et l’appareil IoT Edge

Si vous obtenez une erreur lors du déploiement de SQL Edge par le biais d’Azure IoT Edge, vérifiez que `iotedge`le service est correctement configuré et opérationnel. La documentation suivante peut vous être utile pour résoudre les problèmes liés à Azure IoT Edge :
- [Problèmes courants et résolutions pour Azure IoT Edge](../iot-edge/troubleshoot-common-errors.md).
- [Résoudre les problèmes de votre appareil IoT Edge](../iot-edge/troubleshoot.md)

## <a name="docker-command-errors"></a>Erreurs de commande Docker

Si vous recevez des erreurs pour des commandes `docker`, assurez-vous que le service Docker est en cours d’exécution et essayez de l’exécuter avec des autorisations élevées.

Par exemple, sur Linux, vous pouvez obtenir l’erreur suivante lors de l’exécution des commandes `docker` :

```output
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```

Si vous recevez cette erreur sur Linux, essayez d’exécuter les mêmes commandes précédées de `sudo`. En cas d’échec, vérifiez que le service Docker est en cours d’exécution et démarrez-le si nécessaire.

```bash
sudo systemctl status docker
sudo systemctl start docker
```

Sur Windows, vérifiez que vous lancez PowerShell ou votre invite de commandes en tant qu’administrateur.

## <a name="azure-sql-edge-container-startup-errors"></a>Problèmes de démarrage du conteneur Azure SQL Edge

Si le conteneur SQL Edge ne parvient pas à démarrer, vérifiez les points suivants :

- Si vous utilisez Azure IoT Edge, assurez-vous que les images du module ont bien été téléchargées, et que les variables d’environnement et les options de création du conteneur sont correctement spécifiées dans le manifeste du module.

- Si vous utilisez le déploiement Docker ou Kubernetes, assurez-vous que la commande `docker run` est bien formée. Pour plus d’informations, consultez [Déployer Azure SQL Edge avec Docker](disconnected-deployment.md) et [Déployer un conteneur Azure SQL Edge dans Kubernetes](deploy-kubernetes.md).

- Si vous recevez une erreur comme `failed to create endpoint CONTAINER_NAME on network bridge. Error starting proxy: listen tcp 0.0.0.0:1433 bind: address already in use.`, vous essayez de mapper le port 1433 du conteneur sur un port qui est déjà en cours d’utilisation. Cela peut se produire si vous exécutez SQL Edge localement sur la machine hôte. Cela peut également se produire si vous démarrez deux conteneurs SQL Edge et que vous essayez de les mapper tous les deux sur le même port hôte. Dans ce cas, utilisez le paramètre `-p` pour mapper le port de conteneur 1433 sur un port d’hôte différent. Par exemple : 

    ```bash
    sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge-developer.
    ```

- Si vous recevez une erreur comme `Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30tdout=1&tail=all: dial unix /var/run/docker.sock: connect: permission denied` quand vous essayez de démarrer un conteneur, ajoutez votre utilisateur au groupe Docker dans Ubuntu. Ensuite, déconnectez-vous et reconnectez-vous, car cette modification affecte les nouvelles sessions. 

   ```bash
    usermod -aG docker $USER
   ```

- Vérifiez si des messages d’erreur s’affichent dans le conteneur.

   ```bash
   docker logs e69e056c702d
   ```

- Si vous utilisez un logiciel de gestion de conteneur, assurez-vous qu’il prend en charge les processus de conteneur s’exécutant en tant que root. Le processus sqlservr dans le conteneur s’exécute en tant que root.

- Par défaut, les conteneurs Azure SQL Edge s’exécutent en tant qu’utilisateur non racine nommé `mssql`. Si vous utilisez des points de montage ou des volumes de données pour la persistance des données, assurez-vous que l’utilisateur `mssql` dispose des autorisations appropriées sur le volume. Pour plus d’informations, consultez [Exécuter en tant qu’utilisateur non racine](configure.md#run-azure-sql-edge-as-non-root-user) et [Rendre les données persistantes](configure.md#persist-your-data).

- Si votre conteneur Docker SQL Edge s’arrête immédiatement après son démarrage, examinez vos journaux Docker. Si vous utilisez PowerShell sur Windows avec la commande `docker run`, utilisez des guillemets doubles plutôt que des simples. Avec PowerShell Core, utilisez des guillemets simples.

- Examinez les [journaux d’erreurs SQL Edge](#errorlogs).

## <a name="sql-edge-connection-failures"></a>Échecs de connexion à SQL Edge

Si vous ne réussissez pas à vous connecter à l’instance SQL Edge exécutée dans votre conteneur, vérifiez les points suivants :

- Assurez-vous que votre conteneur SQL Edge a été démarré en consultant la colonne **ÉTAT** de la sortie `docker ps -a`. Si ce n’est pas le cas, utilisez `docker start <Container ID>` pour le démarrer.

- Si vous avez mappé sur un autre port hôte que celui par défaut (pas 1433), assurez-vous que vous spécifiez le port dans votre chaîne de connexion. Vous pouvez voir votre mappage de port dans la colonne **PORTS** de la sortie `docker ps -a`. Pour plus d’informations sur la connexion à Azure SQL Edge, consultez [Se connecter à Azure SQL Edge et l’interroger](connect.md)

- Si vous avez déjà déployé SQL Edge avec un volume de données ou un conteneur de volume de données mappé et que vous utilisez maintenant ce volume ou ce conteneur existant, SQL Edge ignore la valeur de la variable d’environnement `MSSQL_SA_PASSWORD`. C’est le mot de passe d’administrateur système (AS) précédemment configuré qui est utilisé à la place. Cela s’explique par le fait que SQL Edge réutilise les fichiers de base de données master existants dans le volume ou le conteneur de volume de données mappé. Si vous rencontrez ce problème, vous avez les options suivantes :

    - Connectez-vous à l’aide du mot de passe utilisé précédemment, s’il est toujours disponible.
    - Configurez SQL Edge pour utiliser un autre volume ou conteneur de volume de données mappé.
    - Supprimez les fichiers de base de données master existants (master.mdf et mastlog.mdf) dans le volume ou conteneur de volume de données mappé.

- Examinez les [journaux d’erreurs SQL Edge](#errorlogs).

## <a name="sql-edge-setup-and-error-logs"></a><a id="errorlogs"></a> Journaux de configuration et d’erreurs SQL Edge

Par défaut, les journaux d’erreurs SQL Edge se trouvent dans le répertoire **/var/opt/mssql/log** du conteneur et sont accessibles de l’une des manières suivantes :

- Si vous avez monté un répertoire hôte sur **/var/opt/mssql** lorsque vous avez créé votre conteneur, vous pouvez à la place l’examiner dans le sous-répertoire **log** sur le chemin d’accès mappé sur l’ordinateur hôte.
- En utilisant une invite de commandes interactive pour vous connecter au conteneur. Si le conteneur n’est pas en cours d’exécution, démarrez d’abord le conteneur. Utilisez ensuite une invite de commandes interactive pour inspecter les journaux. Vous pouvez obtenir l’ID du conteneur en exécutant la commande `docker ps`.

    ```bash
    docker start <ContainerID>
    docker exec -it <ContainerID> "/bin/bash"
    ```

    À partir de la session bash à l’intérieur de votre conteneur, exécutez les commandes suivantes :

    ```bash
    cd /var/opt/mssql/log
    cat errorlog
    ```
- Si le conteneur SQL Edge est en cours d’exécution et que vous pouvez vous connecter à l’instance à l’aide des outils clients, utilisez la procédure stockée `sp_readerrorlog` pour lire le contenu du journal d’erreurs SQL Edge.

## <a name="execute-commands-in-a-container"></a>Exécuter des commandes dans un conteneur

Si vous disposez d’un conteneur en cours d’exécution, vous pouvez exécuter des commandes dans le conteneur à partir d’un terminal hôte.

Pour récupérer l’ID de conteneur, exécutez :

```bash
docker ps -a
```

Pour démarrer un terminal bash dans le conteneur, exécutez :

```bash
docker exec -it <Container ID> /bin/bash
```

Vous pouvez maintenant exécuter des commandes comme si vous les exécutiez sur le terminal à l’intérieur du conteneur. Quand vous avez terminé, tapez `exit`. Cela quitte la session de commande interactive, mais votre conteneur continue à s’exécuter.

### <a name="enabling-verbose-logging"></a>Activer la journalisation détaillée

Si le niveau de journalisation par défaut pour le moteur de streaming ne fournit pas suffisamment d’informations, la journalisation du débogage pour le moteur de streaming peut être activée dans SQL Edge. Pour activer la journalisation du débogage, ajoutez la variable d’environnement `RuntimeLogLevel=debug` à votre déploiement SQL Edge. Après avoir activé la journalisation du débogage, essayez de reproduire le problème et examinez les éventuels messages ou exceptions enregistrés dans les journaux. 

> [!NOTE]
> L’option de journalisation détaillée doit être utilisée uniquement pour la résolution des problèmes et non pour la charge de travail de production normale. 


## <a name="next-steps"></a>Étapes suivantes

- [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md)
- [Streaming de données dans Azure SQL Edge](stream-data.md)
- [Conservation et nettoyage des données](data-retention-overview.md)
- [Comblement des écarts temporels et imputation des valeurs manquantes](imputing-missing-values.md)







