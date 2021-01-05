---
title: Installer et développer localement avec l’émulateur Azure Cosmos DB
description: Découvrez comment installer et utiliser l’émulateur Azure Cosmos DB dans les environnements Windows, Linux, macOS et Docker pour Windows. L’émulateur vous permet de développer et de tester votre application localement, sans créer d’abonnement Azure et sans frais.
ms.service: cosmos-db
ms.topic: how-to
author: markjbrown
ms.author: mjbrown
ms.date: 09/22/2020
ms.custom: devx-track-csharp, contperf-fy21q1
ms.openlocfilehash: 89e82e210dbf7c404ebd5978d87bf391d8565ee2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033963"
---
# <a name="install-and-use-the-azure-cosmos-db-emulator-for-local-development-and-testing"></a>Installer et utiliser l’émulateur Azure Cosmos DB à des fins de développement et de test en local
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement. L’émulateur Azure Cosmos DB vous permet de développer et de tester votre application localement, sans créer d’abonnement Azure et sans frais. Une fois que vous êtes satisfait du fonctionnement de votre application dans l’émulateur Azure Cosmos DB, vous pouvez commencer à utiliser un compte Azure Cosmos dans le cloud. Cet article décrit l’installation et l’utilisation de l’émulateur sur les environnements Windows, Linux, macOS et Windows Docker.

## <a name="download-the-emulator"></a>Télécharger l’émulateur

Pour commencer, téléchargez et installez la dernière version de l’émulateur Azure Cosmos DB sur votre ordinateur local. L’article [Notes de publication de l’émulateur](local-emulator-release-notes.md) répertorie toutes les versions disponibles et les mises à jour de fonctionnalités qui ont été effectuées dans chaque version.

:::image type="icon" source="media/local-emulator/download-icon.png" border="false"::: **[Télécharger l’émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator)**

Vous pouvez développer des applications à l’aide de l’émulateur Azure Cosmos DB et des comptes d’API [SQL](local-emulator.md#sql-api), [Cassandra](local-emulator.md#cassandra-api), [MongoDB](local-emulator.md#azure-cosmos-dbs-api-for-mongodb), [Gremlin](local-emulator.md#gremlin-api) et [Table](local-emulator.md#table-api). Actuellement, l’explorateur de données dans l’émulateur prend entièrement en charge l’affichage des données SQL uniquement ; les données créées à l’aide d’applications clientes MongoDB, Gremlin/Graph et Cassandra ne sont pas visibles pour l’instant. Pour plus d’informations, consultez [Comment se connecter au point de terminaison de l’émulateur](#connect-with-emulator-apis) à partir de différentes API.

## <a name="how-does-the-emulator-work"></a>Comment l’émulateur fonctionne-t-il ?

L’émulateur Azure Cosmos DB fournit une émulation haute fidélité du service Azure Cosmos DB. Il prend en charge les mêmes fonctionnalités qu’Azure Cosmos DB, notamment la prise en charge de la création et de l’interrogation des données, la configuration et la mise à l’échelle des conteneurs, et l’exécution des procédures stockées et des déclencheurs. Vous pouvez développer et tester des applications à l’aide de l’émulateur Azure Cosmos DB, et les déployer sur Azure à l’échelle mondiale en mettant à jour le point de terminaison de connexion Azure Cosmos DB.

Si l’émulation du service Azure Cosmos DB est fidèle, l’implémentation de l’émulateur est différente de celle du service. Par exemple, l’émulateur utilise les composants du système d’exploitation standard, notamment le système de fichiers local pour la persistance, et la pile de protocole HTTPS pour la connectivité. Les fonctionnalités qui s’appuient sur l’infrastructure Azure, comme la réplication globale, la latence en quelques millisecondes pour les lectures/écritures et les niveaux de cohérence ajustables ne s’appliquent pas lorsque vous utilisez l’émulateur.

Vous pouvez migrer des données entre l’émulateur Azure Cosmos DB et le service Azure Cosmos DB à l’aide de l’[outil de migration des données Azure Cosmos DB](https://github.com/azure/azure-documentdb-datamigrationtool).

## <a name="differences-between-the-emulator-and-the-cloud-service"></a>Différences entre l’émulateur et le service cloud

Dans la mesure où l’émulateur Azure Cosmos DB offre un environnement émulé qui s’exécute sur la station de travail de développement locale, il existe quelques différences entre les fonctionnalités de l’émulateur et celles d’un compte Azure Cosmos dans le cloud :

* Actuellement, le volet **Explorateur de données** dans l’émulateur prend entièrement en charge les clients d’API SQL uniquement. La vue de **l’Explorateur de données** et les opérations pour les API MongoDB, Table, Graph et Cassandra d’Azure Cosmos DB ne sont pas entièrement prises en charge.

* L'émulateur prend en charge uniquement un compte fixe et une clé principale connue. Vous ne pouvez pas regénérer la clé quand vous utilisez l’émulateur Azure Cosmos DB. Toutefois, vous pouvez changer la clé par défaut à l’aide de l’option de [ligne de commande](emulator-command-line-parameters.md).

* Avec l’émulateur, vous pouvez créer un compte Azure Cosmos en mode [débit provisionné](set-throughput.md) uniquement ; actuellement, il ne prend pas en charge le mode [sans serveur](serverless.md).

* L’émulateur n’est pas un service de stockage scalable et ne prend pas en charge un grand nombre de conteneurs. Par défaut, vous pouvez créer jusqu’à 25 conteneurs de taille fixe à 400 RU/s (pris en charge uniquement avec les kits SDK Azure Cosmos DB), ou 5 conteneurs illimités avec l’émulateur Azure Cosmos DB. Pour plus d’informations sur la modification de cette valeur, consultez l’article [Définir la valeur PartitionCount](emulator-command-line-parameters.md#set-partitioncount).

* L’émulateur n’offre pas de [niveaux de cohérence Azure Cosmos DB](consistency-levels.md) différents comme le service cloud le fait.

* L’émulateur ne permet pas la [réplication entre plusieurs régions](distribute-data-globally.md).

* Votre copie de l’émulateur Azure Cosmos DB n’étant pas forcément à jour par rapport aux derniers changements apportés au service Azure Cosmos DB, nous vous conseillons de consulter l’outil de [planification de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md) pour évaluer avec précision les besoins en débit (RU) de votre application.

* L’émulateur prend en charge la taille maximale de propriété d’ ID qui est de 254 caractères.

## <a name="install-the-emulator"></a>Installer l’émulateur

Avant d’installer l’émulateur, vérifiez que vous disposez de la configuration matérielle et logicielle requise suivante :

* Configuration logicielle requise :
  * Actuellement, les systèmes d’exploitation hôtes Windows Server 2016, 2019 ou Windows 10 sont pris en charge. Le système d’exploitation hôte avec Active Directory activé n’est pas pris en charge pour le moment.
  * Système d’exploitation 64 bits

* Conditions matérielles minimales requises :
  * 2 Go de RAM
  * 10 Go d’espace disque disponible

* Pour installer, configurer et exécuter l’émulateur Azure Cosmos DB, vous devez disposer de privilèges d’administrateur sur l’ordinateur. L’émulateur ajoute un certificat et définit également les règles de pare-feu afin d’exécuter ses services. Par conséquent, des droits d’administrateur sont nécessaires pour que l’émulateur puisse exécuter ces opérations.

Pour commencer, téléchargez et installez la dernière version de l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) sur votre ordinateur local. Si vous rencontrez des problèmes lors de l’installation de l’émulateur, consultez l’article [Résolution des problèmes de l’émulateur](troubleshoot-local-emulator.md) pour déboguer.

En fonction de la configuration requise, vous pouvez exécuter l’émulateur sur [Windows](#run-on-windows), [Docker pour Windows](#run-on-windows-docker), [Linux ou macOS](#run-on-linux-macos), comme décrit dans les sections suivantes de cet article.

## <a name="check-for-emulator-updates"></a>Rechercher les mises à jour de l’émulateur

Chaque version de l’émulateur est fournie avec un ensemble de mises à jour de fonctionnalités ou de correctifs de bogues. Pour afficher les versions disponibles, lisez l’article [Notes de publication de l’émulateur](local-emulator-release-notes.md).

Après l’installation, si vous avez utilisé les paramètres par défaut, les données correspondant à l’émulateur sont enregistrées à l’emplacement %LOCALAPPDATA%\CosmosDBEmulator. Vous pouvez configurer un autre emplacement à l’aide des paramètres facultatifs de chemin d’accès aux données, avec `/DataPath=PREFERRED_LOCATION` comme [paramètre de ligne de commande](emulator-command-line-parameters.md). Il n’est pas garanti que vous puissiez accéder aux données créées dans une version de l’émulateur Azure Cosmos DB à partir d’une autre version. Si vous devez rendre vos données persistantes à long terme, nous vous recommandons de les stocker dans un compte Azure Cosmos plutôt que dans l’émulateur Azure Cosmos DB.

## <a name="use-the-emulator-on-windows"></a><a id="run-on-windows"></a>Utiliser l’émulateur sur Windows

L’émulateur Azure Cosmos DB est installé à l’emplacement `C:\Program Files\Azure Cosmos DB Emulator` par défaut. Pour démarrer l’émulateur Azure Cosmos DB sur Windows, sélectionnez le bouton **Démarrer**, ou appuyez sur la touche Windows. Commencez à taper **Émulateur Azure Cosmos DB**, puis sélectionnez l’émulateur dans la liste des applications.

:::image type="content" source="./media/local-emulator/database-local-emulator-start.png" alt-text="Sélectionnez le bouton Démarrer, ou appuyez sur la touche Windows, commencez à taper émulateur Azure Cosmos DB, puis sélectionnez l’émulateur dans la liste des applications":::

Lorsque l’émulateur est démarré, une icône est affichée dans la zone de notification de la barre des tâches Windows. L’Explorateur de données Azure Cosmos s’ouvre automatiquement dans votre navigateur à cette URL `https://localhost:8081/_explorer/index.html`.

:::image type="content" source="./media/local-emulator/database-local-emulator-taskbar.png" alt-text="Notification dans la barre des tâches de l’émulateur local Azure Cosmos DB":::

Vous pouvez également démarrer et arrêter l’émulateur à partir de la ligne de commande ou de commandes PowerShell. Pour plus d’informations, consultez l’article [Référence de l’outil en ligne de commande](emulator-command-line-parameters.md).

L’émulateur Azure Cosmos DB s’exécute par défaut sur l’ordinateur local (« localhost ») et écoute sur le port 8081. L’adresse apparaît sous la forme `https://localhost:8081/_explorer/index.html`. Si vous fermez l’explorateur et que vous souhaitez le rouvrir ultérieurement, vous pouvez ouvrir l’URL dans votre navigateur ou la lancer à partir de l’émulateur Azure Cosmos DB dans l’icône de la barre d’état système Windows, comme indiqué ci-dessous.

:::image type="content" source="./media/local-emulator/database-local-emulator-data-explorer-launcher.png" alt-text="Lanceur de l’Explorateur de données de l’émulateur local Azure Cosmos":::

## <a name="use-the-emulator-on-docker-for-windows"></a><a id="run-on-windows-docker"></a>Utiliser l’émulateur sur Docker pour Windows

Vous pouvez exécuter l’émulateur Azure Cosmos DB dans le conteneur Docker pour Windows. Pour plus d’informations, consultez [Docker Hub](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/) pour la commande docker pull et [GitHub](https://github.com/Azure/azure-cosmos-db-emulator-docker) pour `Dockerfile`. Actuellement, l’émulateur ne fonctionne pas sur Docker pour Oracle Linux. Pour exécuter l’émulateur sur Docker pour Windows, suivez les instructions ci-dessous :

1. Une fois [Docker pour Windows](https://www.docker.com/docker-windows) installé, basculez vers les conteneurs Windows en double-cliquant sur l’icône Docker dans la barre d’outils et en sélectionnant **Switch to Windows containers (Basculer vers les conteneurs Windows)** .

1. Ensuite, extrayez l’image de l’émulateur à partir de Docker Hub en exécutant la commande suivante à partir de l’interpréteur de commandes de votre choix.

   ```bash
   docker pull mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```

1. Pour démarrer l’image, exécutez les commandes suivantes en fonction de la ligne de commande ou de l’environnement PowerShell :

   # <a name="command-line"></a>[Ligne de commande](#tab/cli)

   ```bash

   md %LOCALAPPDATA%\CosmosDBEmulator\bind-mount

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%LOCALAPPDATA%\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```
   Les images Docker basées sur Windows peuvent ne pas être compatibles avec chaque système d’exploitation hôte Windows. Par exemple, l’image par défaut de l’émulateur Azure Cosmos DB est compatible uniquement avec Windows 10 et Windows Server 2016. Si vous avez besoin d’une image qui est compatible avec Windows Server 2019, exécutez la commande suivante à la place :

   ```bash
   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%hostDirectory%,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/winsrv2019/azure-cosmos-emulator:latest
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell

   md $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount 2>null

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=$env:LOCALAPPDATA\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator

   ```

   La réponse ressemble à ce qui suit :

   ```bash
   Starting emulator
   Emulator Endpoint: https://172.20.229.193:8081/
   Primary Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   Exporting SSL Certificate
   You can import the SSL certificate from an administrator command prompt on the host by running:
   cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
   powershell .\importcert.ps1
   --------------------------------------------------------------------------------------------------
   Starting interactive shell
   ```
   ---

   > [!NOTE]
   > Lors de l’exécution de la commande `docker run`, si vous voyez une erreur de conflit de port (c’est-à-dire si le port spécifié est déjà utilisé), transmettez un port personnalisé en modifiant les numéros de port. Par exemple, vous pouvez remplacer le paramètre « -p 8081:8081 » par « -p 443:8081 »

1. À présent, utilisez le point de terminaison de l’émulateur et la clé primaire de la réponse, puis importez le certificat TLS/SSL dans votre hôte. Pour importer le certificat TLS/SSL, procédez comme suit à partir d’une invite de commandes administrateur :

   # <a name="command-line"></a>[Ligne de commande](#tab/cli)

   ```bash
   cd  %LOCALAPPDATA%\CosmosDBEmulator\bind-mount
   powershell .\importcert.ps1
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell
   cd $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount
   .\importcert.ps1
   ```
   ---

1. Si vous fermez l’interpréteur de commandes interactif après le démarrage de l’émulateur, celui-ci arrêtera le conteneur de l’émulateur. Pour rouvrir l’Explorateur de données, accédez à l’URL suivante dans votre navigateur. Le point de terminaison de l’émulateur est fourni dans le message de réponse ci-dessus.

   `https://<emulator endpoint provided in response>/_explorer/index.html`

Si vous avez une application cliente .NET en cours d’exécution sur un conteneur Docker pour Linux et si vous exécutez l’émulateur Azure Cosmos DB sur une machine hôte, suivez les instructions de la prochaine section afin d’importer le certificat dans le conteneur Docker pour Linux.

### <a name="regenerate-the-emulator-certificates-when-running-on-a-docker-container"></a>Régénérer les certificats de l’émulateur lors de l’exécution sur un conteneur Docker

Lors de l’exécution de l’émulateur dans un conteneur Docker, les certificats associés à l’émulateur sont régénérés chaque fois que vous arrêtez et redémarrez le conteneur respectif. Pour cette raison, vous devez réimporter les certificats après chaque démarrage de conteneur. Pour contourner cette limitation, vous pouvez utiliser un fichier Docker Compose pour lier le conteneur Docker à une adresse IP spécifique et à une image conteneur.

Par exemple, vous pouvez utiliser la configuration suivante dans le fichier Docker Compose. Veillez à le formater selon vos besoins : 

```yml
version: '2.4' # Do not upgrade to 3.x yet, unless you plan to use swarm/docker stack: https://github.com/docker/compose/issues/4513

networks:
  default:
    external: false
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"

services:

  # First create a directory that will hold the emulator traces and certificate to be imported
  # set hostDirectory=C:\emulator\bind-mount
  # mkdir %hostDirectory%

  cosmosdb:
    container_name: "azurecosmosemulator"
    hostname: "azurecosmosemulator"
    image: 'mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator'
    platform: windows
    tty: true
    mem_limit: 3GB
    ports:
        - '8081:8081'
        - '8900:8900'
        - '8901:8901'
        - '8902:8902'
        - '10250:10250'
        - '10251:10251'
        - '10252:10252'
        - '10253:10253'
        - '10254:10254'
        - '10255:10255'
        - '10256:10256'
        - '10350:10350'
    networks:
      default:
        ipv4_address: 172.16.238.246
    volumes:
        - '${hostDirectory}:C:\CosmosDB.Emulator\bind-mount'
```

## <a name="use-the-emulator-on-linux-or-macos"></a><a id="run-on-linux-macos"></a>Utiliser l’émulateur sur Linux ou macOS

L’émulateur Azure Cosmos DB peut uniquement être exécuté sur Windows. Si vous utilisez Linux ou macOS, vous pouvez exécuter l’émulateur sur une machine virtuelle Windows hébergée dans un hyperviseur tel que Parallels ou VirtualBox.

> [!NOTE]
> Chaque fois que vous redémarrez la machine virtuelle Windows qui est hébergée dans un hyperviseur, vous devez réimporter le certificat, car l’adresse IP de la machine virtuelle change. L’importation du certificat n’est pas nécessaire si vous avez configuré la machine virtuelle pour conserver l’adresse IP.

Procédez comme suit pour utiliser l’émulateur sur des environnements Linux ou macOS :

1. Exécutez la commande suivante à partir de la machine virtuelle Windows et prenez note de l’adresse IPv4 :

   ```bash
   ipconfig.exe
   ```

1. Dans votre application, modifiez l’URL du point de terminaison pour utiliser l’adresse IPv4 renvoyée par `ipconfig.exe` au lieu de `localhost`.

1. À partir de la machine virtuelle Windows, depuis la ligne de commande, lancez l’émulateur Azure Cosmos DB avec les options suivantes. Pour plus d’informations sur les paramètres pris en charge par la ligne de commande, consultez l’article [Référence de l’outil en ligne de commande de l’émulateur](emulator-command-line-parameters.md) :

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   ```

1. Enfin, vous devez résoudre le processus d’approbation de certificat entre l’application en cours d’exécution dans l’environnement Linux ou Mac et l’émulateur. Vous pouvez utiliser l’une des deux options suivantes pour résoudre le certificat :

   1. [Importer le certificat TLS/SSL de l’émulateur dans l’environnement Linux ou Mac](#import-certificate) ou
   2. [Désactiver la validation TLS/SSL dans l’application](#disable-ssl-validation)

### <a name="option-1-import-the-emulator-tlsssl-certificate"></a><a id="import-certificate"></a>Option 1 : Importer le certificat TLS/SSL de l’émulateur

Les sections suivantes montrent comment importer le certificat TLS/SSL de l’émulateur dans des environnements Linux et macOS.

#### <a name="linux-environment"></a>Environnement Linux

Si vous travaillez sur Linux, .NET passe le relais à OpenSSL pour effectuer la validation :

1. [Exporter le certificat au format .PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). L’option PFX est disponible lorsque vous choisissez d’exporter la clé privée.

1. Copiez ce fichier PFX dans votre environnement Linux.

1. Convertir le fichier PFX en fichier CRT

   ```bash
   openssl pkcs12 -in YourPFX.pfx -clcerts -nokeys -out YourCTR.crt
   ```

1. Copiez le fichier CRT dans le dossier qui contient les certificats personnalisés dans votre distribution Linux. Sur les distributions Debian, il se trouve en général sur `/usr/local/share/ca-certificates/`.

   ```bash
   cp YourCTR.crt /usr/local/share/ca-certificates/
   ```

1. Mettez à jour les certificats TLS/SSL, ce qui mettra à jour le dossier `/etc/ssl/certs/`.

   ```bash
   update-ca-certificates
   ```

#### <a name="macos-environment"></a>Environnement macOS

Suivez les étapes ci-dessous si vous travaillez sur Mac :

1. [Exporter le certificat au format .PFX](local-emulator-export-ssl-certificates.md#export-emulator-certificate). L’option PFX est disponible lorsque vous choisissez d’exporter la clé privée.

1. Copiez ce fichier PFX dans votre environnement Mac.

1. Ouvrez l’application *Keychain Access* et importez le fichier PFX.

1. Ouvrez la liste des certificats et identifiez celui qui a le nom `localhost`.

1. Ouvrez le menu contextuel pour cet élément particulier, sélectionnez *Obtenir l’élément* et, sous *Approuver* > *Lors de l’utilisation de ce certificat*, sélectionnez *Toujours approuver*. 

   :::image type="content" source="./media/local-emulator/mac-trust-certificate.png" alt-text="Ouvrez le menu contextuel pour cet élément particulier, sélectionnez Obtenir l’élément et, sous l’option Approuver - Lors de l’utilisation de ce certificat, sélectionnez Toujours approuver":::
  
### <a name="option-2-disable-the-ssl-validation-in-the-application"></a><a id="disable-ssl-validation"></a>Option 2 : Désactiver la validation SSL dans l’application

La désactivation de la validation SSL est uniquement recommandée à des fins de développement et ne doit pas être effectuée lors de l’exécution dans un environnement de production. Les exemples suivants montrent comment désactiver la validation SSL pour les applications .NET et Node.js.

# <a name="net-standard-21"></a>[.NET Standard 2.1+](#tab/ssl-netstd21)

Pour toute application s’exécutant dans une infrastructure compatible avec .NET Standard 2.1 ou une version ultérieure, nous pouvons tirer parti de `CosmosClientOptions.HttpClientFactory` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard21)]

# <a name="net-standard-20"></a>[.NET Standard 2.0](#tab/ssl-netstd20)

Pour toute application s’exécutant dans une infrastructure compatible avec .NET Standard 2.0, nous pouvons tirer parti de `CosmosClientOptions.HttpClientFactory` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/HttpClientFactory/Program.cs?name=DisableSSLNETStandard20)]

# <a name="nodejs"></a>[Node.JS](#tab/ssl-nodejs)

Pour les applications Node.js, vous pouvez modifier votre fichier `package.json` pour définir `NODE_TLS_REJECT_UNAUTHORIZED` lors du démarrage de l’application :

```json
"start": NODE_TLS_REJECT_UNAUTHORIZED=0 node app.js
```
--- 

## <a name="enable-access-to-emulator-on-a-local-network"></a>Activer l’accès à l’émulateur sur un réseau local

Supposons que vous avez plusieurs ordinateurs utilisant un même réseau, et que vous configurez l’émulateur sur un ordinateur et souhaitez y accéder à partir d’un autre ordinateur. Dans ce cas, vous devez activer l’accès à l’émulateur sur un réseau local.

Vous pouvez exécuter l’émulateur sur un réseau local. Pour activer l’accès réseau, spécifiez l’option `/AllowNetworkAccess` sur la [ligne de commande](emulator-command-line-parameters.md), avec `/Key=key_string` ou `/KeyFile=file_name`. Vous pouvez initialement utiliser `/GenKeyFile=file_name` pour générer un fichier avec une clé aléatoire. Passez ensuite cette clé à `/KeyFile=file_name` ou `/Key=contents_of_file`.

Avant d’activer l’accès réseau pour la première fois, l’utilisateur doit arrêter l’émulateur et supprimer le répertoire de données de l’émulateur *%LOCALAPPDATA%\CosmosDBEmulator*.

## <a name="authenticate-connections-when-using-emulator"></a><a id="authenticate-requests"></a>Authentifier les connexions lors de l’utilisation de l’émulateur

Tout comme avec Azure Cosmos DB dans le cloud, chaque demande que vous effectuez auprès de l’émulateur Azure Cosmos DB doit être authentifiée. L’émulateur Azure Cosmos DB prend uniquement en charge une communication sécurisée via TLS. L’émulateur Azure Cosmos DB prend en charge un seul compte fixe et une clé d’authentification connue pour l’authentification de la clé primaire. Ce compte et cette clé sont les seules informations d’identification autorisées pour une utilisation avec l’émulateur Azure Cosmos DB. Il s'agit des éléments suivants :

```bash
Account name: localhost:<port>
Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
```

> [!NOTE]
> La clé primaire prise en charge par l’émulateur Azure Cosmos DB peut être utilisée uniquement avec l’émulateur. Vous ne pouvez pas utiliser votre compte et votre clé Azure Cosmos DB de production avec l’émulateur Azure Cosmos DB.

> [!NOTE]
> Si vous avez démarré l’émulateur avec l’option /Key, utilisez la clé générée à la place de la clé par défaut `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`. Pour plus d’informations sur l’option /Key, consultez la section [Référence sur l’outil en ligne de commande](emulator-command-line-parameters.md).

## <a name="connect-to-different-apis-with-the-emulator"></a><a id="connect-with-emulator-apis"></a>Se connecter à différentes API avec l’émulateur

### <a name="sql-api"></a>API SQL

Une fois que l’émulateur Azure Cosmos DB est exécuté sur votre bureau, vous pouvez utiliser n’importe quel [SDK Azure Cosmos DB](sql-api-sdk-dotnet-standard.md) pris en charge ou l’[API REST Azure Cosmos DB](/rest/api/cosmos-db/) pour interagir avec lui. L’émulateur Azure Cosmos DB comprend également un Explorateur de données intégré à partir duquel vous pouvez créer des conteneurs pour l’API SQL ou l’API Azure Cosmos DB pour Mongo DB. L’Explorateur de données vous permet d’afficher et de modifier des éléments sans écrire de code.

```csharp
// Connect to the Azure Cosmos DB Emulator running locally
CosmosClient client = new CosmosClient(
   "https://localhost:8081", 
    "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

```

### <a name="azure-cosmos-dbs-api-for-mongodb"></a>API d’Azure Cosmos DB pour MongoDB

Une fois que l’émulateur Azure Cosmos DB s’exécute sur votre poste de travail, vous pouvez utiliser l’[API Azure Cosmos DB pour MongoDB](mongodb-introduction.md) afin d’interagir avec lui. Démarrez l’émulateur à partir d’une [invite de commandes](emulator-command-line-parameters.md) administrateur avec « /EnableMongoDbEndpoint ». Utilisez ensuite la chaîne de connexion suivante pour vous connecter au compte de l’API MongoDB :

```bash
mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true
```

### <a name="table-api"></a>API de table

Une fois que l’émulateur Azure Cosmos DB s’exécute sur votre poste de travail, vous pouvez utiliser le [kit SDK de l’API Table pour Azure Cosmos DB](./tutorial-develop-table-dotnet.md) afin d’interagir avec lui. Démarrez l’émulateur à partir d’une [invite de commandes](emulator-command-line-parameters.md) administrateur avec « /EnableTableEndpoint ». Ensuite, exécutez le code suivant pour vous connecter au compte d’API Table :

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Table;
using CloudTable = Microsoft.WindowsAzure.Storage.Table.CloudTable;
using CloudTableClient = Microsoft.WindowsAzure.Storage.Table.CloudTableClient;

string connectionString = "DefaultEndpointsProtocol=http;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=http://localhost:8902/;";

CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("testtable");
table.CreateIfNotExists();
table.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
```

### <a name="cassandra-api"></a>API Cassandra

Démarrez l’émulateur à partir d’une [invite de commandes](emulator-command-line-parameters.md) administrateur avec « /EnableCassandraEndpoint ». Vous pouvez également définir la variable d’environnement `AZURE_COSMOS_EMULATOR_CASSANDRA_ENDPOINT=true`.

1. [Installez Python 2.7](https://www.python.org/downloads/release/python-2716/)

1. [Installez Cassandra CLI/CQLSH](https://cassandra.apache.org/download/)

1. Exécutez les commandes ci-dessous à partir d’une fenêtre d’invite de commandes :

   ```bash
   set Path=c:\Python27;%Path%
   cd /d C:\sdk\apache-cassandra-3.11.3\bin
   set SSL_VERSION=TLSv1_2
   set SSL_VALIDATE=false
   cqlsh localhost 10350 -u localhost -p C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== --ssl
   ```

1. Dans l’interpréteur de commandes CQLSH, exécutez les commandes suivantes pour vous connecter au point de terminaison Cassandra :

   ```bash
   CREATE KEYSPACE MyKeySpace WITH replication = {'class':'MyClass', 'replication_factor': 1};
   DESCRIBE keyspaces;
   USE mykeyspace;
   CREATE table table1(my_id int PRIMARY KEY, my_name text, my_desc text);
   INSERT into table1 (my_id, my_name, my_desc) values( 1, 'name1', 'description 1');
   SELECT * from table1;
   EXIT
   ```

### <a name="gremlin-api"></a>API Gremlin

Démarrez l’émulateur à partir d’une [invite de commandes](emulator-command-line-parameters.md) administrateur avec « /EnableGremlinEndpoint ». Vous pouvez également définir la variable d’environnement `AZURE_COSMOS_EMULATOR_GREMLIN_ENDPOINT=true`

1. [Installez apache-tinkerpop-gremlin-console-3.3.4](https://archive.apache.org/dist/tinkerpop/3.3.4).

1. Dans l’Explorateur de données de l’émulateur, créez une base de données « db1 » et une collection « coll1 » ; pour la clé de partition, définissez le nom avec « /name »

1. Exécutez les commandes ci-dessous à partir d’une fenêtre d’invite de commandes :

   ```bash
   cd /d C:\sdk\apache-tinkerpop-gremlin-console-3.3.4-bin\apache-tinkerpop-gremlin-console-3.3.4
  
   copy /y conf\remote.yaml conf\remote-localcompute.yaml
   notepad.exe conf\remote-localcompute.yaml
     hosts: [localhost]
     port: 8901
     username: /dbs/db1/colls/coll1
     password: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
     connectionPool: {
     enableSsl: false}
     serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
     config: { serializeResultToString: true  }}

   bin\gremlin.bat
   ```

1. Dans l’interpréteur de commandes Gremlin, exécutez les commandes suivantes pour vous connecter au point de terminaison Gremlin :

   ```bash
   :remote connect tinkerpop.server conf/remote-localcompute.yaml
   :remote console
   :> g.V()
   :> g.addV('person1').property(id, '1').property('name', 'somename1')
   :> g.addV('person2').property(id, '2').property('name', 'somename2')
   :> g.V()
   ```

## <a name="uninstall-the-local-emulator"></a><a id="uninstall"></a>Désinstaller l’émulateur local

Utilisez les étapes suivantes pour désinstaller l’émulateur :

1. Quittez toutes les instances ouvertes de l’émulateur local en cliquant avec le bouton droit sur l’icône de **l’émulateur Azure Cosmos DB** dans la zone de notification, puis en sélectionnant **Quitter**. Quitter l’ensemble des instances peut prendre une minute.

1. Dans la zone de recherche Windows, tapez **Applications et fonctionnalités**, puis sélectionnez **Applications et fonctionnalités (paramètres système)** .

1. Dans la liste des applications, faites défiler la page jusqu’à trouver **Émulateur Azure Cosmos DB**, sélectionnez-le, cliquez sur **Désinstaller**, puis confirmez en sélectionnant de nouveau **Désinstaller**.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser l’émulateur local pour un développement local gratuit. Vous pouvez maintenant passer aux articles suivants :

* [Exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec des applications Java, Python et Node.js](local-emulator-export-ssl-certificates.md)
* [Utiliser des paramètres de ligne de commande et des commandes PowerShell pour contrôler l’émulateur](emulator-command-line-parameters.md)
* [Problèmes de débogage avec l’émulateur](troubleshoot-local-emulator.md)
