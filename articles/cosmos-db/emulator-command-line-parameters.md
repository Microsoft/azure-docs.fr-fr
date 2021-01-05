---
title: Informations de référence sur la ligne de commande et PowerShell pour l’émulateur Azure Cosmos DB
description: Découvrez les paramètres de ligne de commande de l’émulateur Azure Cosmos DB, comment contrôler l’émulateur avec PowerShell et comment changer le nombre de conteneurs que vous pouvez créer dans l’émulateur.
ms.service: cosmos-db
ms.topic: how-to
author: markjbrown
ms.author: mjbrown
ms.date: 09/17/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 2579326e92144149472874611b5307920a93c049
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031400"
---
# <a name="command-line-and-powershell-reference-for-azure-cosmos-db-emulator"></a>Informations de référence sur la ligne de commande et PowerShell pour l’émulateur Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement local. Après avoir [installé l’émulateur](local-emulator.md), vous pouvez le contrôler à l’aide de la ligne de commande et des commandes PowerShell. Cet article explique comment utiliser la ligne de commande et les commandes PowerShell pour démarrer et arrêter ’émulateur, configurer les options et effectuer d’autres opérations. Vous devez exécuter les commandes à partir de l’emplacement d’installation.

##  <a name="manage-the-emulator-with-command-line-syntax"></a><a id="command-line"></a>Gérer l’émulateur avec la syntaxe de ligne de commande

```cmd
Microsoft.Azure.Cosmos.Emulator.exe [/Shutdown] [/DataPath] [/Port] [/MongoPort] [/DirectPorts] [/Key] [/EnableRateLimiting] [/DisableRateLimiting] [/NoUI] [/NoExplorer] [/EnableMongoDbEndpoint] [/?]
```

Pour afficher la liste des options, tapez `Microsoft.Azure.Cosmos.Emulator.exe /?` dans l’invite de commandes.

|**Option** | **Description** | **Commande**| **Arguments**|
|---|---|---|---|
|[aucun argument] | Démarre l’émulateur Azure Cosmos DB avec des paramètres par défaut. |Microsoft.Azure.Cosmos.Emulator.exe| |
|[Aide] |Affiche la liste des arguments de ligne de commande pris en charge.|Microsoft.Azure.Cosmos.Emulator.exe /? | |
| GetStatus |Permet d’obtenir l’état de l’émulateur Azure Cosmos DB. L’état est indiqué par le code de sortie : 1 = démarrage, 2 = exécution, 3 = arrêté. Un code de sortie négatif indique qu’une erreur s’est produite. Aucune autre sortie n’est générée. | Microsoft.Azure.Cosmos.Emulator.exe /GetStatus| |
| Shutdown| Arrête l’émulateur Azure Cosmos DB.| Microsoft.Azure.Cosmos.Emulator.exe /Shutdown | |
|DataPath | Spécifie le chemin d’accès dans lequel stocker les fichiers de données. La valeur par défaut est %LocalAppdata%\CosmosDBEmulator. | Microsoft.Azure.Cosmos.Emulator.exe /DataPath=\<datapath\> | \<datapath\>: Un chemin accessible |
|Port | Spécifie le numéro de port à utiliser pour l'émulateur. La valeur par défaut est 8081. |Microsoft.Azure.Cosmos.Emulator.exe /Port=\<port\> | \<port\>: Numéro de port unique |
| ComputePort | Spécifie le numéro de port à utiliser pour le service de passerelle Interop Compute. Le port de sonde du point de terminaison HTTP de la passerelle est calculé comme étant égal à ComputePort + 79. Par conséquent, les ports ComputePort et ComputePort + 79 doivent être ouverts et disponibles. La valeur par défaut est 8900. | Microsoft.Azure.Cosmos.Emulator.exe /ComputePort=\<computeport\> | \<computeport\>: Numéro de port unique |
| EnableMongoDbEndpoint=3.2 | Active l’API MongoDB 3.2 | Microsoft.Azure.Cosmos.Emulator.exe /EnableMongoDbEndpoint=3.2 | |
| EnableMongoDbEndpoint=3.6 | Active l’API MongoDB 3.6 | Microsoft.Azure.Cosmos.Emulator.exe /EnableMongoDbEndpoint=3.6 | |
| MongoPort | Spécifie le numéro de port à utiliser pour l'API de compatibilité MongoDB. La valeur par défaut est 10255. |Microsoft.Azure.Cosmos.Emulator.exe /MongoPort=\<mongoport\>|\<mongoport\>: Numéro de port unique|
| EnableCassandraEndpoint | Active l’API Cassandra | Microsoft.Azure.Cosmos.Emulator.exe /EnableCassandraEndpoint | |
| CassandraPort | Spécifie le numéro de port à utiliser pour le point de terminaison Cassandra. La valeur par défaut est 10350. | Microsoft.Azure.Cosmos.Emulator.exe /CassandraPort=\<cassandraport\> | \<cassandraport\>: Numéro de port unique |
| EnableGremlinEndpoint | Active l’API Gremlin | Microsoft.Azure.Cosmos.Emulator.exe /EnableGremlinEndpoint | |
| GremlinPort | Numéro de port à utiliser pour le point de terminaison Gremlin. La valeur par défaut est 8901. | Microsoft.Azure.Cosmos.Emulator.exe /GremlinPort=\<port\> | \<port\>: Numéro de port unique |
|EnableTableEndpoint | Active l’API Table Azure | Microsoft.Azure.Cosmos.Emulator.exe /EnableTableEndpoint | |
|TablePort | Numéro de port à utiliser pour le point de terminaison Table Azure. La valeur par défaut est 8902. | Microsoft.Azure.Cosmos.Emulator.exe /TablePort=\<port\> | \<port\>: Numéro de port unique|
| KeyFile | Lit la clé d’autorisation dans le fichier spécifié. Utilisez l’option /GenKeyFile pour générer un fichier de clé | Microsoft.Azure.Cosmos.Emulator.exe /KeyFile=\<file_name\> | \<file_name\>: chemin du fichier |
| ResetDataPath | Supprime de manière récursive tous les fichiers du chemin spécifié. Si vous ne spécifiez pas de chemin, le chemin par défaut est %LOCALAPPDATA%\CosmosDbEmulator | Microsoft.Azure.Cosmos.Emulator.exe /ResetDataPath=\<path> | \<path\>: Chemins d'accès au fichier  |
| StartTraces  |  Démarre la collecte des journaux de trace du débogage à l’aide de LOGMAN. | Microsoft.Azure.Cosmos.Emulator.exe /StartTraces | |
| StopTraces     | Arrête la collecte des journaux de traces du débogage à l’aide de LOGMAN. | Microsoft.Azure.Cosmos.Emulator.exe /StopTraces  | |
| StartWprTraces  |  Démarre la collecte des journaux de traces du débogage à l’aide de l’outil d’enregistrement des performances Windows. | Microsoft.Azure.Cosmos.Emulator.exe /StartWprTraces | |
| StopWprTraces     | Arrête la collecte des journaux de traces du débogage à l’aide de l’outil d’enregistrement des performances Windows. | Microsoft.Azure.Cosmos.Emulator.exe /StopWprTraces  | |
|FailOnSslCertificateNameMismatch | Par défaut, l’émulateur regénère son certificat TLS/SSL auto-signé, si le réseau SAN du certificat n’inclut pas les informations de l’hôte émulateur (nom de domaine, adresse IPv4 locale, « localhost » et « 127.0.0.1 »). Avec cette option, l’émulateur échoue au démarrage. Vous devez alors utiliser l’option /GenCert pour créer et installer un nouveau certificat TLS/SSL auto-signé. | Microsoft.Azure.Cosmos.Emulator.exe /FailOnSslCertificateNameMismatch  | |
| GenCert | Génère et installe un nouveau certificat TLS/SSL auto-signé, incluant de manière facultative une liste de noms DNS supplémentaires séparés par des virgules pour accéder à l’émulateur via le réseau. | Microsoft.Azure.Cosmos.Emulator.exe /GenCert=\<dns-names\> |\<dns-names\>: Liste de noms DNS supplémentaires séparés par des virgules (facultatif)  |
| DirectPorts |Spécifie les ports à utiliser pour une connectivité directe. Les valeurs par défaut sont 10251,10252,10253,10254. | Microsoft.Azure.Cosmos.Emulator.exe /DirectPorts:\<directports\> | \<directports\>: Liste de 4 ports séparée par des virgules |
| Clé |Clé d’autorisation pour l’émulateur. La clé doit être le codage en base 64 d’un vecteur de 64 octets. | Microsoft.Azure.Cosmos.Emulator.exe /Key:\<key\> | \<key\>: La clé doit être le codage en base 64 d’un vecteur de 64 octets|
| EnableRateLimiting | Spécifie que le comportement de limitation de taux de demandes est activé. |Microsoft.Azure.Cosmos.Emulator.exe /EnableRateLimiting | |
| DisableRateLimiting |Spécifie que le comportement de limitation de taux de demandes est désactivé. |Microsoft.Azure.Cosmos.Emulator.exe /DisableRateLimiting | |
| NoUI | Ne pas afficher l’interface utilisateur de l’émulateur. | Microsoft.Azure.Cosmos.Emulator.exe /NoUI | |
| NoExplorer | Ne pas afficher l’Explorateur de données au démarrage. |Microsoft.Azure.Cosmos.Emulator.exe /NoExplorer | | 
| PartitionCount | Spécifie le nombre maximal de conteneurs partitionnés. Pour plus d’informations, consultez [Changer le nombre de conteneurs](#set-partitioncount). | Microsoft.Azure.Cosmos.Emulator.exe /PartitionCount=\<partitioncount\> | \<partitioncount\>: nombre maximal de conteneurs à partition unique autorisés. La valeur par défaut est 25. Valeur maximale autorisée : 250.|
| DefaultPartitionCount| Spécifie le nombre par défaut de partitions pour un conteneur partitionné. | Microsoft.Azure.Cosmos.Emulator.exe /DefaultPartitionCount=\<defaultpartitioncount\> | \<defaultpartitioncount\> La valeur par défaut est 25.|
| AllowNetworkAccess | Permet d’accéder à l’émulateur sur un réseau. Vous devez également passer /Key=\<key_string\> ou /KeyFile=\<file_name\> pour activer l’accès réseau. | Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /Key=\<key_string\> ou Microsoft.Azure.Cosmos.Emulator.exe /AllowNetworkAccess /KeyFile=\<file_name\>| |
| NoFirewall | Ne pas ajuster les règles de pare-feu quand l’option /AllowNetworkAccess est utilisée. |Microsoft.Azure.Cosmos.Emulator.exe /NoFirewall | |
| GenKeyFile | Générer une nouvelle clé d’autorisation et Générer une nouvelle clé d’autorisation et l’enregistrer dans le fichier spécifié. La clé générée peut être utilisée avec les options /Key or /KeyFile. | Microsoft.Azure.Cosmos.Emulator.exe /GenKeyFile=\<path to key file\> | |
| Cohérence | Définir le niveau de cohérence par défaut pour le compte. | Microsoft.Azure.Cosmos.Emulator.exe /Consistency=\<consistency\> | \<consistency\>: La valeur doit être l’un des [niveaux de cohérence](consistency-levels.md) suivants : Session, Strong, Eventual ou BoundedStaleness. La valeur par défaut est Session. |
| ? | Afficher le message d’aide.| | |

## <a name="manage-the-emulator-with-powershell"></a>Gérer l’émulateur avec PowerShell

L’émulateur est fourni avec un module PowerShell que vous pouvez utiliser pour démarrer, arrêter et désinstaller le service, et récupérer son état. Exécutez l’applet de commande suivante pour utiliser le module PowerShell :

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
```

Vous pouvez aussi placer le répertoire `PSModules` dans `PSModulesPath` et importer le module en utilisant la commande suivante :

```powershell
$env:PSModulesPath += "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules"
Import-Module Microsoft.Azure.CosmosDB.Emulator
```

Voici un récapitulatif des commandes permettant de contrôler l’émulateur à partir de PowerShell :

### `Get-CosmosDbEmulatorStatus`

**Syntaxe**

`Get-CosmosDbEmulatorStatus`

**Remarques**

Retourne une des valeurs ServiceControllerStatus suivantes : ServiceControllerStatus.StartPending, ServiceControllerStatus.Running ou ServiceControllerStatus.Stopped.

### `Start-CosmosDbEmulator`

**Syntaxe**

`Start-CosmosDbEmulator [-DataPath <string>] [-DefaultPartitionCount <uint16>] [-DirectPort <uint16[]>] [-MongoPort <uint16>] [-NoUI] [-NoWait] [-PartitionCount <uint16>] [-Port <uint16>] [<CommonParameters>]`

**Remarques**

Démarre l’émulateur. Par défaut, la commande attend que l’émulateur soit prêt à accepter les requêtes. Utilisez l’option -NoWait si vous souhaitez que l’applet de commande soit retournée dès le démarrage de l’émulateur.

### `Stop-CosmosDbEmulator`

**Syntaxe**

 `Stop-CosmosDbEmulator [-NoWait]`

**Remarques**

Arrête l’émulateur. Par défaut, cette commande attend que l’émulateur soit complètement arrêté. Utilisez l’option -NoWait si vous souhaitez que l’applet de commande soit retournée dès le début du processus d’arrêt de l’émulateur.

### `Uninstall-CosmosDbEmulator`

**Syntaxe**

`Uninstall-CosmosDbEmulator [-RemoveData]`

**Remarques**

Désinstalle l’émulateur et supprime éventuellement l’intégralité du contenu de $env:LOCALAPPDATA\CosmosDbEmulator.
L’applet de commande garantit que l’émulateur est arrêté avant d’être désinstallé.

## <a name="change-the-number-of-default-containers"></a><a id="set-partitioncount"></a>Modifier le nombre de conteneurs par défaut

Par défaut, vous pouvez créer jusqu’à 25 conteneurs de taille fixe (pris en charge uniquement avec les kits SDK Azure Cosmos DB), ou 5 conteneurs illimités avec l’émulateur Azure Cosmos DB. En changeant la valeur **PartitionCount**, vous pouvez créer jusqu’à 250 conteneurs de taille fixe ou 50 conteneurs illimités, ou n’importe quelle combinaison des deux à hauteur de 250 conteneurs de taille fixe au total (sachant qu’un conteneur illimité est égal à cinq conteneurs de taille fixe). Toutefois, il est déconseillé de configurer l’émulateur pour s’exécuter avec plus de 200 conteneurs de taille fixe. En effet, cela ajoute une surcharge aux opérations d’E/S sur le disque, qui entraîne des délais d’attente imprévisibles lors de l’utilisation des API de point de terminaison.

Si vous tentez de créer un conteneur après le dépassement du nombre de partitions actuel, l’émulateur lève une exception ServiceUnavailable et affiche un message similaire à celui-ci.

> Nous sommes désolés, mais nous rencontrons actuellement une demande élevée dans cette région et nous ne pouvons pas répondre à votre demande pour l’instant. Nous nous efforçons d’offrir une capacité en ligne toujours plus grande, et nous vous encourageons à réessayer plus tard.
> ActivityId : 12345678-1234-1234-1234-123456789abc

Pour changer le nombre de conteneurs disponibles dans l’émulateur Azure Cosmos DB, effectuez les étapes suivantes :

1. Supprimez toutes les données locales de l’émulateur Azure Cosmos DB en cliquant avec le bouton droit sur l’icône **Émulateur Azure Cosmos DB** dans la zone d’état, puis en cliquant sur **Reset Data...** (Réinitialiser les données...).

1. Supprimez toutes les données de l’émulateur dans le dossier `%LOCALAPPDATA%\CosmosDBEmulator`.

1. Quittez toutes les instances ouvertes en cliquant avec le bouton droit sur l’icône **Émulateur Azure Cosmos DB** dans la zone d’état, puis en cliquant sur **Quitter**. Quitter l’ensemble des instances peut prendre une minute.

1. Installez la dernière version de [l’émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator).

1. Lancez l’émulateur avec l’indicateur PartitionCount en définissant une valeur <= 250. Par exemple : `C:\Program Files\Azure Cosmos DB Emulator> Microsoft.Azure.Cosmos.Emulator.exe /PartitionCount=100`.
 
## <a name="next-steps"></a>Étapes suivantes

* [Exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec des applications Java, Python et Node.js](local-emulator-export-ssl-certificates.md)
* [Problèmes de débogage avec l’émulateur](troubleshoot-local-emulator.md)
