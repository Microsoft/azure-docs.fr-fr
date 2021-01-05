---
title: Configurer Azure SQL Edge
description: En savoir plus sur la configuration d’Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 0c49f5ab9f10456c32f7f8516cba0e851fa80e74
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392331"
---
# <a name="configure-azure-sql-edge"></a>Configurer Azure SQL Edge

Azure SQL Edge prend en charge la configuration par le biais d'une deux options suivantes :

- Variables d'environnement
- Un fichier mssql.conf placé dans le dossier /var/opt/mssql

> [!NOTE]
> La définition de variables d’environnement remplace les paramètres spécifiés dans le fichier mssql.conf.

## <a name="configure-by-using-environment-variables"></a>Configurer à l'aide des variables d’environnement

Azure SQL Edge expose plusieurs variables d’environnement qui peuvent être utilisées pour configurer le conteneur SQL Edge. Ces variables d’environnement correspondent à un sous-ensemble de celles disponibles pour SQL Server sur Linux. Pour plus d’informations sur les variables d’environnement SQL Server sur Linux, consultez [Variables d’environnement](/sql/linux/sql-server-linux-configure-environment-variables/).

Les nouvelles variables d’environnement suivantes ont été ajoutées à Azure SQL Edge. 

| Variable d’environnement | Description | Valeurs |     
|-----|-----| ---------- | 
| **PlanId** | Spécifie la référence SKU Azure SQL Edge à utiliser lors de l'initialisation. Cette variable d'environnement est uniquement requise lors du déploiement d'Azure SQL Edge à l'aide d'Azure IoT Edge. | **asde-developer-on-iot-edge** ou **asde-premium-on-iot-edge** | 
| **MSSQL_TELEMETRY_ENABLED** | Activez ou désactivez la collecte des données d’utilisation et de diagnostic. | TRUE ou FALSE |  
| **MSSQL_TELEMETRY_DIR** | Définit le répertoire cible pour les fichiers d’audit de la collecte des données d’utilisation et de diagnostic. | Emplacement du dossier dans le conteneur SQL Edge. Ce dossier peut être mappé à un volume hôte à l’aide de points de montage ou de volumes de données. | 
| **MSSQL_PACKAGE** | Spécifie l'emplacement du package dacpac ou bacpac à déployer. | Dossier, fichier ou URL SAP contenant les packages dacpac ou bacpac. Pour plus d’informations, consultez [Déployer des packages SQL Database DACPAC et BACPAC dans SQL Edge](deploy-dacpac.md). |


La variable d’environnement SQL Server sur Linux suivante n’est pas prise en charge par Azure SQL Edge. Si elle est définie, cette variable d’environnement sera ignorée lors de l’initialisation du conteneur.

| Variable d’environnement | Description |
|-----|-----|
| **MSSQL_ENABLE_HADR** | Activez le groupe de disponibilité. Par exemple, **1** est activé et **0** est désactivé. |

> [!IMPORTANT]
> La variable d’environnement **MSSQL_PID** pour SQL Edge accepte uniquement **Premium** et **Developer** en tant que valeurs valides. Azure SQL Edge ne prend pas en charge l’initialisation à l’aide d’une clé de produit.

### <a name="specify-the-environment-variables"></a>Spécifier les variables d’environnement

Spécifiez les variables d’environnement pour SQL Edge lorsque vous déployez le service via le [portail Azure](deploy-portal.md). Vous pouvez les ajouter dans la section **Variables d’environnement** du déploiement de module ou dans le cadre des **Options de création de conteneur**.

Ajoutez les valeurs dans des **Variables d’environnement**.

![Définir en utilisant une liste de variables d'environnement](media/configure/set-environment-variables.png)

Ajoutez des valeurs dans **Options de création de conteneur**.

![Définir en utilisant des options de création de conteneur](media/configure/set-environment-variables-using-create-options.png)

> [!NOTE]
> En mode de déploiement déconnecté, les variables d'environnement peuvent être spécifiées à l'aide de l'option `-e`, `--env` ou `--env-file` de la commande `docker run`.

## <a name="configure-by-using-an-mssqlconf-file"></a>Configurer à l’aide d’un fichier mssql.conf

Azure SQL Edge n’inclut pas [l’utilitaire de configuration mssql-conf](/sql/linux/sql-server-linux-configure-mssql-conf/) comme le fait SQL Server sur Linux. Vous devez configurer manuellement le fichier mssql.conf et le placer dans le lecteur de stockage persistant qui est mappé au dossier /var/opt/mssql/ dans le module SQL Edge. Lors du déploiement de SQL Edge à partir de la Place de marché Azure, ce mappage est spécifié en tant qu’option **Mounts** dans les **Options de création de conteneur**.

```json
    {
        "Mounts": [
          {
            "Type": "volume",
            "Source": "sqlvolume",
            "Target": "/var/opt/mssql"
          }
        ]
      }
    }
```

Les nouvelles options MSSQL. conf suivantes ont été ajoutées pour Azure SQL Edge. 

|Option|Description|
|:---|:---|
|**customerfeedback** | Choisissez si SQL Server envoie des commentaires à Microsoft. Pour plus d'informations, consultez [Désactiver la collecte des données d'utilisation et de diagnostic](usage-and-diagnostics-data-configuration.md#disable-usage-and-diagnostic-data-collection)|      
|**userrequestedlocalauditdirectory** | Définit le répertoire cible pour les fichiers d’audit de la collecte des données d’utilisation et de diagnostic. Pour plus d'informations, consultez [Audit local de la collecte des données d'utilisation et de diagnostic](usage-and-diagnostics-data-configuration.md#local-audit-of-usage-and-diagnostic-data-collection) |        

Les options mssql.conf suivantes ne s’appliquent pas à SQL Edge :

|Option|Description|
|:---|:---|
|**Feedback des clients** | Choisissez si SQL Server envoie des commentaires à Microsoft. |
|**Profil Database Mail** | Définir le profil de messagerie de base de données par défaut pour SQL Server sur Linux. |
|**Haute disponibilité** | Activer les groupes de disponibilité. |
|**Microsoft Distributed Transaction Coordinator** | Configurer et dépanner MSDTC sur Linux. Les autres options de configuration liées aux transactions distribuées ne sont pas prises en charge par SQL Edge. Pour plus d’informations sur ces options de configuration supplémentaires, consultez [Configurer MSDTC](/sql/linux/sql-server-linux-configure-mssql-conf#msdtc). |
|**CLUF ML Services** | Accepter les CLUF R et Python pour les packages Azure Machine Learning. S'applique à SQL Server 2019 uniquement.|
|**outboundnetworkaccess** |Activez l'accès réseau sortant pour les extensions [Machine Learning Services](/sql/linux/sql-server-linux-setup-machine-learning/) R, Python et Java.|

L’exemple suivant de fichier mssql.conf fonctionne pour SQL Edge. Pour plus d’informations sur le format du fichier mssql.conf, consultez [Format mssql.conf](/sql/linux/sql-server-linux-configure-mssql-conf#mssql-conf-format).

```ini
[EULA]
accepteula = Y

[coredump]
captureminiandfull = true
coredumptype = full

[filelocation]
defaultbackupdir = /var/opt/mssql/backup/
defaultdatadir = /var/opt/mssql/data/
defaultdumpdir = /var/opt/mssql/data/
defaultlogdir = /var/opt/mssql/log/

[language]
lcid = 1033

[memory]
memorylimitmb = 6144

[sqlagent]
errorlogfile = /var/opt/mssql/log/sqlagentlog.log
errorlogginglevel = 7

[traceflag]
traceflag0 = 3604
traceflag1 = 3605
traceflag2 = 1204
```

## <a name="run-azure-sql-edge-as-non-root-user"></a>Exécutez Azure SQL Edge en tant qu’utilisateur non racine

Par défaut, les conteneurs Azure SQL Edge s'exécutent avec un utilisateur/groupe non racine. En cas de déploiement via la Place de marché Azure (ou via la commande docker run), si aucun autre utilisateur/groupe n'est spécifié, les conteneurs SQL Edge démarrent en tant qu'utilisateur MSSQL (non racine). Pour spécifier un autre utilisateur non racine pendant le déploiement, ajoutez la paire clé-valeur `*"User": "<name|uid>[:<group|gid>]"*` sous les options de création de conteneur. Dans l’exemple ci-dessous, SQL Edge est configuré de manière à démarrer en tant qu’utilisateur `*IoTAdmin*`.

```json
{
    ..
    ..
    ..
    "User": "IoTAdmin",
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only",
        "MSSQL_PID=Premium"
    ]
}
```

Pour permettre à l’utilisateur non racine d’accéder aux fichiers de base de données qui se trouvent sur des volumes montés, vérifiez que l’utilisateur/le groupe sous lequel vous exécutez le conteneur dispose des autorisations de lecture et d’écriture sur le stockage de fichiers persistant. Dans l’exemple ci-dessous, nous avons défini l’utilisateur non racine avec user_id 10001 comme propriétaire des fichiers. 

```bash
chown -R 10001:0 <database file dir>
```

### <a name="upgrading-from-earlier-ctp-releases"></a>Mise à niveau à partir de versions CTP antérieures

Les CTP précédentes d'Azure SQL Edge étaient configurées de manière à s'exécuter en tant qu'utilisateurs racine. Les options suivantes sont disponibles lors de la mise à niveau à partir de CTP antérieures.

- Continuer à utiliser l’utilisateur racine - Pour continuer à utiliser l’utilisateur racine, ajoutez la paire clé-valeur `*"User": "0:0"*` sous les options de création de conteneur.
- Utiliser l’utilisateur MSSQL par défaut - Pour utiliser l’utilisateur MSSQL par défaut, suivez les étapes ci-dessous
  - Ajoutez un utilisateur nommé MSSQL sur l’hôte Docker. Dans l’exemple ci-dessous, nous ajoutons un utilisateur MSSQL avec l’ID 10001. Cet utilisateur est également ajouté au groupe racine.
    ```bash
    sudo useradd -M -s /bin/bash -u 10001 -g 0 mssql
    ```
  - Modifier l'autorisation sur le répertoire/volume de montage dans lequel se trouve le fichier de base de données 
    ```bash
    sudo chgrp -R 0 /var/lib/docker/volumes/kafka_sqldata/
    sudo chmod -R g=u /var/lib/docker/volumes/kafka_sqldata/
    ```
- Utiliser un autre compte d’utilisateur non racine - Pour utiliser un autre compte d’utilisateur non racine
  - Mettez à jour les options de création de conteneur pour spécifier l’ajout d’une paire clé-valeur `*"User": "user_name | user_id*` sous les options de création de conteneur. Remplacez user_name ou user_id par un user_name ou un user_id réel de votre hôte Docker. 
  - Modifiez les autorisations sur le répertoire/volume de montage.

## <a name="persist-your-data"></a> Rendre vos données persistantes

Vos changements de configuration et fichiers de base de données Azure SQL Edge sont conservés dans le conteneur même si vous redémarrez le conteneur avec `docker stop` et `docker start`. Toutefois, si vous supprimez le conteneur avec `docker rm`, tout ce qui se trouve dans le conteneur est supprimé, y compris Azure SQL Edge et vos bases de données. La section suivante explique comment utiliser des **volumes de données** pour conserver les fichiers de votre base de données même si les conteneurs associés sont supprimés.

> [!IMPORTANT]
> Par Azure SQL Edge, il est essentiel que vous compreniez la persistance des données dans Docker. En plus de la discussion de cette section, consultez la documentation de Docker pour savoir [comment gérer les données dans les conteneurs Docker](https://docs.docker.com/engine/tutorials/dockervolumes/).

### <a name="mount-a-host-directory-as-data-volume"></a>Monter un répertoire hôte en tant que volume de données

La première option consiste à monter un répertoire sur votre hôte en tant que volume de données dans votre conteneur. Pour ce faire, utilisez la commande `docker run` avec l'indicateur `-v <host directory>:/var/opt/mssql`. Cela permet de restaurer les données entre les exécutions de conteneur.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge
```

Cette technique vous permet également de partager et d’afficher les fichiers sur l’ordinateur hôte en dehors de Docker.

> [!IMPORTANT]
> Actuellement, le mappage du volume hôte pour **Docker sur Windows** ne prend pas en charge le mappage de l’intégralité du répertoire `/var/opt/mssql`. Toutefois, vous pouvez mapper un sous-répertoire comme `/var/opt/mssql/data` à votre ordinateur hôte.

> [!IMPORTANT]
> Le mappage du volume hôte pour **Docker sur Mac** avec l’image Azure SQL Edge n’est pas pris en charge pour l’instant. Utilisez des conteneurs de volume de données à la place. Cette restriction est spécifique au répertoire `/var/opt/mssql`. La lecture à partir d’un répertoire monté fonctionne bien. Par exemple, vous pouvez monter un répertoire hôte à l’aide de -v sur Mac et restaurer une sauvegarde à partir d’un fichier .bak résidant sur l’hôte.

### <a name="use-data-volume-containers"></a>Utiliser des conteneurs de volume de données

La deuxième option consiste à utiliser un conteneur de volume de données. Vous pouvez créer un conteneur de volume de données en spécifiant un nom de volume à la place d’un répertoire hôte avec le paramètre `-v`. L’exemple suivant crée un volume de données partagé nommé **sqlvolume**.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge
```

> [!NOTE]
> Cette technique pour la création implicite d’un volume de données dans la commande d’exécution ne fonctionne pas avec les versions antérieures de Docker. Dans ce cas, utilisez les étapes explicites décrites dans la documentation de Docker, [Création et montage d’un conteneur de volume de données](https://docs.docker.com/engine/tutorials/dockervolumes/#creating-and-mounting-a-data-volume-container).

Même si vous arrêtez et supprimez ce conteneur, le volume de données persiste. Vous pouvez l’afficher avec la commande `docker volume ls`.

```bash
docker volume ls
```

Si vous créez ensuite un autre conteneur avec le même nom de volume, le nouveau conteneur utilise les données Azure SQL Edge contenues dans le volume.

Pour supprimer un conteneur de volume de données, utilisez la commande `docker volume rm`.

> [!WARNING]
> Si vous supprimez le conteneur de volume de données, toutes les données Azure SQL Edge dans le conteneur sont *définitivement* supprimées.


## <a name="next-steps"></a>Étapes suivantes

- [Se connecter à Azure SQL Edge](connect.md)
- [Générer une solution IoT de bout en bout avec SQL Edge](tutorial-deploy-azure-resources.md)