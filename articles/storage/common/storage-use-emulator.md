---
title: Utilisation de l’émulateur de stockage Azure pour le développement et le test
description: L’émulateur de stockage Azure fournit un environnement de développement local gratuit pour développer et tester vos applications Stockage Azure.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/16/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.openlocfilehash: 6d9e0e63a3586be139620c154616acfcba2ab10f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498215"
---
# <a name="use-the-azure-storage-emulator-for-development-and-testing"></a>Utilisation de l’émulateur de stockage Azure pour le développement et le test

L’émulateur de stockage Microsoft Azure est un outil qui émule les services Blob, File d’attente et Table d’Azure à des fins de développement. Vous pouvez tester localement votre application sur les services de stockage sans souscrire d’abonnement Azure ni engendrer de frais. Quand vous êtes satisfait du fonctionnement de votre application dans l’émulateur, commencez à utiliser un compte de stockage Azure dans le cloud.

> [!IMPORTANT]
> L’Émulateur de stockage Azure n’est plus développé activement. [**Azurite**](storage-use-azurite.md) est désormais la plateforme de l’émulateur de stockage. Azurite remplace l’Émulateur de stockage Azure. Azurite continuera d’être mis à jour pour prendre en charge les dernières versions des API de stockage Azure. Pour plus d’informations, consultez [**Utiliser l’émulateur Azurite à des fins de développement local pour Stockage Azure**](storage-use-azurite.md).

## <a name="get-the-storage-emulator"></a>Obtenir l’émulateur de stockage

L’émulateur de stockage fait partie du [Kit de développement logiciel (SDK) Microsoft Azure](https://azure.microsoft.com/downloads/). Vous pouvez également installer l’émulateur de stockage en tant que [programme d’installation autonome](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409) (téléchargement direct). Pour installer l’émulateur de stockage, vous devez disposer de privilèges administratifs sur votre ordinateur.

Actuellement, l’émulateur de stockage s’exécute uniquement sous Windows. Si vous avez besoin d’un émulateur de stockage pour Linux, il est possible d’utiliser l’émulateur de stockage open source [Azurite](https://github.com/azure/azurite), géré par la communauté.

> [!NOTE]
> Il n’est pas garanti que vous puissiez accéder aux données créées dans une version de l’émulateur de stockage à partir d’une autre version. Si vous devez rendre vos données persistantes à long terme, nous vous recommandons de stocker ces données dans un compte de stockage Azure plutôt que dans l’émulateur de stockage.
> 
> L’émulateur de stockage dépend des versions spécifiques des bibliothèques OData. Le remplacement des DLL OData utilisées par l’émulateur de stockage par des versions ultérieures n’est pas pris en charge et peut provoquer un comportement inattendu. Toutefois, vous pouvez utiliser n’importe quelle version OData prise en charge par le service de stockage pour envoyer des demandes à l’émulateur.

## <a name="how-the-storage-emulator-works"></a>Fonctionnement de l’émulateur de stockage

L’émulateur de stockage utilise une instance locale de Microsoft SQL Server 2012 Express LocalDB pour émuler les services de stockage Azure. Vous pouvez configurer l’émulateur de stockage de sorte qu’il accède à une instance locale de SQL Server plutôt qu’à l’instance LocalDB. Pour en savoir plus, consultez la section [Démarrer et initialiser l’émulateur de stockage](#start-and-initialize-the-storage-emulator) plus loin dans cet article.

L'émulateur de stockage se connecte à SQL Server ou LocalDB par l'intermédiaire de l'authentification Windows.

Il existe quelques différences de fonctionnalités entre l’émulateur de stockage et les services de stockage Azure. Pour plus d’informations sur ces différences, consultez la section [Différences entre l’émulateur de stockage et le Stockage Azure](#differences-between-the-storage-emulator-and-azure-storage) plus loin dans cet article.

## <a name="start-and-initialize-the-storage-emulator"></a>Démarrer et initialiser l’émulateur de stockage

Pour démarrer l’émulateur de stockage Azure :

1. Sélectionnez le bouton **Démarrer** ou appuyez sur la touche **Windows**.
2. Commencez à taper `Azure Storage Emulator`.
3. Sélectionnez l’émulateur dans la liste des applications affichées.

Au démarrage de l’émulateur de stockage, une fenêtre d’invite de commande apparaît. Vous pouvez utiliser cette fenêtre de console pour démarrer et arrêter l’émulateur de stockage. Vous pouvez également effacer des données, initialiser l’émulateur et afficher son état à partir de l’invite de commandes. Pour plus d’informations, consultez la section [Référence de l’outil en ligne de commande de l’émulateur de stockage](#storage-emulator-command-line-tool-reference) plus loin dans cet article.

> [!NOTE]
> L’émulateur de stockage Azure peut ne pas démarrer correctement si un autre émulateur de stockage comme Azurite s’exécute sur le système.

Lorsque l’émulateur est en cours d’exécution, une icône est affichée dans la zone de notification de la barre des tâches Windows.

Lorsque vous fermez la fenêtre d’invite de commande de l’émulateur de stockage, celui-ci continuera à s’exécuter. Pour faire réapparaître la fenêtre de console de l’émulateur de stockage, suivez les étapes ci-dessus comme si vous démarriez l’émulateur de stockage.

La première fois que vous exécutez l'émulateur de stockage, l'environnement de stockage local est initialisé pour vous. Le processus d'initialisation crée une base de données dans LocalDB et réserve des ports HTTP pour chaque service de stockage local.

L’émulateur de stockage est installé par défaut dans `C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator`.

> [!TIP]
> Vous pouvez utiliser [l’Explorateur Stockage Microsoft Azure](https://storageexplorer.com) pour travailler avec les ressources d’émulateur de stockage local. Recherchez « (Émulateur - Ports par défaut) » sous « Locaux et joints » dans l’arborescence de ressources de l’Explorateur de stockage après avoir installé et démarré l’émulateur de stockage.
>

### <a name="initialize-the-storage-emulator-to-use-a-different-sql-database"></a>Initialiser l’émulateur de stockage de manière à utiliser une autre base de données SQL

Vous pouvez utiliser l’outil en ligne de commande de l’émulateur de stockage pour initialiser l’émulateur de stockage afin qu’il pointe vers une instance de base de données SQL autre que l’instance LocalDB par défaut :

1. Ouvrez la fenêtre de console de l’émulateur de stockage, comme décrit dans la section [Démarrer et initialiser l’émulateur de stockage](#start-and-initialize-the-storage-emulator).
1. Dans la fenêtre de console, tapez la commande suivante, où `<SQLServerInstance>` est le nom de l’instance SQL Server. Pour utiliser LocalDB, spécifiez `(localdb)\MSSQLLocalDb` comme instance SQL Server.

   `AzureStorageEmulator.exe init /server <SQLServerInstance>`

   Vous pouvez également exécuter la commande suivante, qui indique à l'émulateur d'utiliser l'instance SQL Server par défaut :

   `AzureStorageEmulator.exe init /server .`

   En guise d’alternative, vous pouvez exécuter la commande suivante, qui initialise la base de données en rétablissant l’instance LocalDB par défaut :

   `AzureStorageEmulator.exe init /forceCreate`

Pour plus d’informations sur ces commandes, consultez la section [Référence de l’outil en ligne de commande de l’émulateur de stockage](#storage-emulator-command-line-tool-reference).

> [!TIP]
> Vous pouvez utiliser [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) pour gérer vos instances SQL Server, y compris l’installation de LocalDB. Dans la boîte de dialogue **Se connecter au serveur** de SMSS, spécifiez `(localdb)\MSSQLLocalDb` dans le champ **Nom du serveur :** pour vous connecter à l’instance LocalDB.

## <a name="authenticating-requests-against-the-storage-emulator"></a>Authentification des demandes auprès de l’émulateur de stockage

Une fois l’émulateur de stockage installé et démarré, vous pouvez tester votre code sur celui-ci. Chaque demande que vous effectuez auprès de l’émulateur de stockage doit être autorisée, sauf s’il s’agit d’une demande anonyme. Vous pouvez autoriser les demandes auprès de l’émulateur de stockage à l’aide de l’authentification par clé partagée ou d’une signature d’accès partagé (SAP).

### <a name="authorize-with-shared-key-credentials"></a>Autorisation à l’aide d’informations d’identification de clé partagée

[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

Pour plus d’informations sur les chaînes de connexion, consultez [Configuration des chaînes de connexion Stockage Azure](./storage-configure-connection-string.md).

### <a name="authorize-with-a-shared-access-signature"></a>Autorisation à l’aide d’une signature d’accès partagé

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Certaines bibliothèques clientes de stockage Azure, telles que la bibliothèque Xamarin, prennent uniquement en charge l’authentification par jeton de signature d’accès partagé (SAP). Vous pouvez créer le jeton SAS à l’aide de l’[Explorateur Stockage](https://storageexplorer.com/) ou d’une autre application prenant en charge l’authentification avec clé partagée.

Vous pouvez également générer un jeton SAP à l’aide d’Azure PowerShell. L’exemple suivant génère un jeton SAP avec des autorisations complètes sur un conteneur de blobs :

1. Installez Azure PowerShell si vous ne l’avez pas encore (à l’aide de la version la plus récente de cmdlets d’Azure PowerShell). Pour connaître la procédure d’installation, consultez l’article [Install and configure Azure PowerShell](/powershell/azure/install-Az-ps) (Installation et configuration d’Azure PowerShell).
2. Ouvrez Azure PowerShell et exécutez les commandes suivantes, en remplaçant `CONTAINER_NAME` par un nom de votre choix :

```powershell
$context = New-AzStorageContext -Local

New-AzStorageContainer CONTAINER_NAME -Permission Off -Context $context

$now = Get-Date

New-AzStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri
```

L’URI de la signature d’accès partagé obtenue pour le nouveau conteneur doit être semblable à ce qui suit :

```
http://127.0.0.1:10000/devstoreaccount1/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss
```

La signature d’accès partagé créée avec cet exemple est valide une journée. La signature accorde un accès complet (lecture, écriture, suppression et liste) aux blobs du conteneur.

Pour plus d’informations sur les signatures d’accès partagé, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](storage-sas-overview.md).

## <a name="addressing-resources-in-the-storage-emulator"></a>Adressage des ressources dans l’émulateur de stockage

Les points de terminaison de service de l’émulateur de stockage sont différents de ceux d’un compte de stockage Azure. Dans la mesure où l’ordinateur local n’effectue pas de résolution de noms de domaine, les points de terminaison de l’émulateur de stockage doivent être des adresses locales.

Lorsque vous adressez une ressource dans un compte de stockage Azure, vous utilisez le schéma suivant. Le nom du compte fait partie du nom d’hôte de l’URI et la ressource étant adressée fait partie du chemin d’accès de l’URI :

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

Par exemple, l’URI suivante est une adresse valide pour un objet blob dans un compte de stockage Azure :

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

Dans la mesure où l’ordinateur local n’effectue pas de résolution de noms de domaine, le nom du compte fait partie du chemin de l’URI au lieu du nom d’hôte. Utilisez le format d’URI suivant pour une ressource dans l’émulateur de stockage :

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

Par exemple, l'adresse suivante peut être utilisée pour accéder à un objet blob dans l'émulateur de stockage :

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

Les points de terminaison de service de l’émulateur de stockage sont :

* Service BLOB : `http://127.0.0.1:10000/<account-name>/<resource-path>`
* Service de File d’attente : `http://127.0.0.1:10001/<account-name>/<resource-path>`
* Service de Table : `http://127.0.0.1:10002/<account-name>/<resource-path>`

### <a name="addressing-the-account-secondary-with-ra-grs"></a>Adressage du compte secondaire avec RA-GRS

À partir de la version 3.1, l’émulateur de stockage prend en charge la réplication géo-redondante avec accès en lecture (RA-GRS). Vous pouvez accéder à l’emplacement secondaire en ajoutant -secondary au nom du compte. Par exemple, vous pouvez utiliser l'adresse suivante pour accéder à un objet blob en utilisant l'emplacement secondaire en lecture seule dans l'émulateur de stockage :

`http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt`

> [!NOTE]
> Pour un accès par programmation au stockage secondaire avec l'émulateur de stockage, utilisez la bibliothèque cliente de stockage pour .NET version 3.2 ou ultérieure. Pour plus d’informations, consultez [Bibliothèque cliente Microsoft Azure Storage pour .NET](/previous-versions/azure/dn261237(v=azure.100)) .
>
>

## <a name="storage-emulator-command-line-tool-reference"></a>Référence de l’outil en ligne de commande de l’émulateur de stockage

À partir de la version 3.0, une fenêtre de console s’affiche lorsque vous démarrez l’émulateur de stockage. Utilisez la ligne de commande dans la fenêtre de console pour démarrer et arrêter l’émulateur. Vous pouvez également interroger l’état et effectuer d’autres opérations à partir de la ligne de commande.

> [!NOTE]
> Si l’émulateur de calcul Microsoft Azure est installé, une icône apparaît dans la zone de notification lorsque vous lancez l’émulateur de stockage. Cliquez avec le bouton droit sur l’icône pour faire apparaître un menu, qui fournit un moyen graphique pour démarrer et arrêter l’émulateur de stockage.
>
>

### <a name="command-line-syntax"></a>Syntaxe de ligne de commande

`AzureStorageEmulator.exe [start] [stop] [status] [clear] [init] [help]`

### <a name="options"></a>Options

Pour afficher la liste des options, tapez `/help` dans l’invite de commandes.

| Option | Description | Commande | Arguments |
| --- | --- | --- | --- |
| **Start** |Permet de démarrer l’émulateur de stockage. |`AzureStorageEmulator.exe start [-inprocess]` |*-Reprocess* : Démarrez l’émulateur dans le processus actuel au lieu de créer un processus. |
| **Stop** |Permet d’arrêter l’émulateur de stockage. |`AzureStorageEmulator.exe stop` | |
| **État** |Permet d’imprimer l’état de l’émulateur de stockage. |`AzureStorageEmulator.exe status` | |
| **Clear** |Permet d’effacer les données de tous les services spécifiés sur la ligne de commande. |`AzureStorageEmulator.exe clear [blob] [table] [queue] [all]` |*blob* : Efface les données d’objet blob. <br/>*queue* : Efface les données de file d’attente. <br/>*table* : Efface les données de table. <br/>*all* : Efface toutes les données de tous les services. |
| **Init** |Effectue une initialisation ponctuelle pour configurer l’émulateur. |<code>AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate&#124;-skipcreate] [-reserveports&#124;-unreserveports] [-inprocess]</code> |*-server serverName\instanceName* : Spécifie le serveur qui héberge l’instance SQL. <br/>*-sqlinstance instanceName* : Spécifie le nom de l’instance SQL à utiliser dans l’instance de serveur par défaut. <br/>*-forcecreate* : Force la création de la base de données SQL, même si celle-ci existe déjà. <br/>*-skipcreate* : Ignore la création de la base de données SQL. Cet argument est prioritaire sur -forcecreate.<br/>*-reserveports* : Tente de réserver les ports HTTP associés aux services.<br/>*-unreserveports* : Tente de supprimer les réservations des ports HTTP associés aux services. Cet argument est prioritaire sur -reserveports.<br/>*-inprocess* : Effectue l’initialisation dans le processus actuel au lieu de générer un nouveau processus. Vous devez lancer le processus actuel avec des autorisations élevées en cas de modification des réservations des ports. |

## <a name="differences-between-the-storage-emulator-and-azure-storage"></a>Différences entre l’émulateur de stockage et Azure Storage

L’émulateur de stockage étant un environnement émulé local, il existe des différences entre l’émulateur et un compte de stockage Azure dans le cloud :

* L'émulateur de stockage prend en charge uniquement un compte fixe et une clé d'authentification connue.
* L’émulateur de stockage n’est pas un service de stockage scalable et ne prend pas en charge un grand nombre de clients simultanés.
* Comme décrit dans [Adressage des ressources dans l’émulateur de stockage](#addressing-resources-in-the-storage-emulator), les ressources ne sont pas adressées de la même manière dans l’émulateur de stockage et dans un compte de stockage Azure. La différence est due au fait que la résolution de noms de domaine est disponible dans le cloud, mais pas sur l’ordinateur local.
* À partir de la version 3.1, le compte d'émulateur de stockage prend en charge la réplication géo-redondante avec accès en lecture. Dans l’émulateur, RA-GRS est activé pour tous les comptes et il n’y a jamais de latence entre le réplica principal et le réplica secondaire. Les opérations Get Blob Service Stats, Get Queue Service Stats et Get Table Service Stats sont prises en charge sur le compte secondaire et retournent toujours la valeur de l’élément de réponse `LastSyncTime` comme heure actuelle en fonction de la base de données SQL sous-jacente.
* Les points de terminaison du service de fichiers et de protocole SMB ne sont pas pris en charge dans l’émulateur de stockage pour le moment.
* Si vous utilisez une version des services de stockage qui n’est pas prise en charge par l’émulateur, l’émulateur retourne une erreur VersionNotSupportedByEmulator (code d’état HTTP 400 - Demande incorrecte).

### <a name="differences-for-blob-storage"></a>Différences pour le stockage d’objets blob

Les différences suivantes s’appliquent au stockage d’objets blob dans l’émulateur :

* L’émulateur de stockage prend uniquement en charge les objets blob d’une taille inférieure ou égale à 2 Go.
* La longueur maximale d’un nom d’objet blob dans l’émulateur de stockage est de 256 caractères, contre 1 024 caractères dans le stockage Azure.
* La copie incrémentielle permet de copier des instantanés à partir d’objets blob remplacés, ce qui renvoie une erreur sur le service.
* L’opération Get Page Ranges Diff ne fonctionne pas entre des instantanés copiés à l’aide de la copie incrémentielle d’objets blob.
* Une opération Put Blob peut réussir sur un objet blob qui existe dans l’émulateur de stockage avec un bail actif, même si l’ID du bail n’a pas été spécifié dans la demande.
* L’émulateur ne prend pas en charge les opérations d’ajout de blob. Toute tentative d’exécution d’une opération sur un objet blob d’ajout renvoie une erreur FeatureNotSupportedByEmulator (code d’état HTTP 400 – demande incorrecte).

### <a name="differences-for-table-storage"></a>Différences pour le stockage de tables

Les différences suivantes s’appliquent au stockage de tables dans l’émulateur :

* Les propriétés de date du service de Table dans l’émulateur de stockage ne prennent en charge que la plage autorisée par SQL Server 2005 (les dates postérieures au 1er janvier 1753). Toutes les dates antérieures au 1er janvier 1753 sont remplacées par cette valeur. La précision des dates est limitée à la précision de SQL Server 2005, ce qui signifie que les dates sont précises au 1/300e de seconde.
* L’émulateur de stockage prend en charge des valeurs de propriétés de clé de partition et de clé de ligne de moins de 512 octets chacune. La taille totale du nom du compte, du nom de table et de l’ensemble des noms de propriétés de clé ne peut pas dépasser 900 octets.
* La taille totale d’une ligne de table dans l’émulateur de stockage est limitée à moins de 1 Mo.
* Dans l’émulateur de stockage, les propriétés du type de données `Edm.Guid` ou `Edm.Binary` ne prennent en charge que les opérateurs de comparaison `Equal (eq)` et `NotEqual (ne)` dans les chaînes de filtre de requête.

### <a name="differences-for-queue-storage"></a>Différences pour le stockage de files d’attente

Le stockage de files d’attente dans l’émulateur ne présente aucune différence spécifique.

## <a name="storage-emulator-release-notes"></a>Notes de publication de l’émulateur de stockage

### <a name="version-510"></a>Version 5.10

* L’émulateur de stockage ne refuse pas la version 2019-07-07 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-59"></a>Version 5.9

* L’émulateur de stockage ne refuse pas la version 2019-02-02 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-58"></a>Version 5.8

* L’émulateur de stockage ne refuse pas la version 2018-11-09 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-57"></a>Version 5.7

* Correction d’un bogue générant une erreur en cas d’activation de la journalisation.

### <a name="version-56"></a>Version 5.6

* L’émulateur de stockage prend maintenant en charge la version 2018-03-28 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-55"></a>Version 5.5

* L’émulateur de stockage prend maintenant en charge la version 2017-11-09 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.
* La prise en charge a été ajoutée pour la propriété **Créé** de l’objet blob, qui retourne l’heure de création de l’objet blob.

### <a name="version-54"></a>Version 5.4

* Pour améliorer la stabilité de l’installation, l’émulateur ne tente plus de réserver les ports au moment de l’installation. Si vous souhaitez des réservations de port, utilisez l’option *-reserveports* de la commande **init** pour les spécifier.

### <a name="version-53"></a>Version 5.3

* L’émulateur de stockage prend maintenant en charge la version 2017-07-29 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-52"></a>Version 5.2

* L’émulateur de stockage prend maintenant en charge la version 2017-04-17 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.
* Correction d’un bogue impliquant le mauvais encodage des valeurs de propriété de table.

### <a name="version-51"></a>Version 5.1

* Correction d’un bogue dans lequel l’émulateur de stockage retournait l’en-tête `DataServiceVersion` dans certaines réponses où le service ne se trouvait pas.

### <a name="version-50"></a>Version 5.0

* Le programme d’installation de l’émulateur de stockage ne vérifie plus s’il existe déjà des installations de MSSQL et de .NET Framework.
* Le programme d’installation de l’émulateur de stockage ne crée plus la base de données dans le cadre de l’installation. Si nécessaire, la base de données sera toujours créée dans le cadre du démarrage.
* La création de la base de données ne nécessite plus une élévation de privilèges.
* Les réservations de ports ne sont plus nécessaires pour le démarrage.
* Ajoute les options suivantes pour `init` : `-reserveports` (nécessite une élévation), `-unreserveports` (nécessite une élévation), `-skipcreate`.
* L’option d’interface utilisateur Émulateur de stockage de la barre d’état système lance maintenant l’interface de ligne de commande. L’ancienne interface graphique utilisateur (GUI) n’est plus disponible.
* Certaines DLL ont été supprimées ou renommées.

### <a name="version-46"></a>Version 4.6

* L’émulateur de stockage prend maintenant en charge la version 2016-05-31 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-45"></a>Version 4.5

* Corrige un bogue provoquant l’échec de l’installation et de l’initialisation au moment du renommage de la base de données de sauvegarde.

### <a name="version-44"></a>Version 4.4

* L’émulateur de stockage prend maintenant en charge la version 2015-12-11 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.
* Le garbage collection par l’émulateur de stockage des données blob est désormais plus efficace quand le nombre d’objets blob est élevé.
* Correction d’un bogue qui provoquait la validation du XML ACL de conteneur légèrement différemment de la façon dont procède le service de stockage.
* Correction d’un bogue qui entraînait parfois le signalement des valeurs de date/heure max et min dans le fuseau horaire incorrect.

### <a name="version-43"></a>Version 4.3

* L’émulateur de stockage prend maintenant en charge la version 2015-07-08 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-42"></a>Version 4.2

* L’émulateur de stockage prend maintenant en charge la version 2015-04-05 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table.

### <a name="version-41"></a>Version 4.1

* L’émulateur de stockage prend maintenant en charge la version 2015-02-21 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table. Il ne prend pas en charge les nouvelles fonctionnalités d’ajout de blob.
* L’émulateur retourne à présent un message d’erreur significatif pour les versions non prises en charge des services de stockage. Nous vous recommandons d’utiliser la dernière version de l’émulateur. Si vous obtenez une erreur VersionNotSupportedByEmulator (code d’état HTTP 400 - demande incorrecte), téléchargez la dernière version de l’émulateur.
* Correction d’un bogue dans lequel une condition de course a généré des données d’entité de table incorrectes lors d’opérations de fusion simultanées.

### <a name="version-40"></a>Version 4.0

* L’exécutable de l’émulateur de stockage est renommé en *AzureStorageEmulator.exe*.

### <a name="version-32"></a>Version 3.2

* L’émulateur de stockage prend maintenant en charge la version 2014-02-14 des services de stockage sur les points de terminaison des services Blob, File d’attente et Table. Les points de terminaison du service de fichiers ne sont pas pris en charge dans l’émulateur de stockage pour le moment. Pour plus d’informations sur la version 2014-02-14, consultez la page [Contrôle de version pour les services Azure Storage](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) .

### <a name="version-31"></a>Version 3.1

* Le stockage géo-redondant avec accès en lecture (RA-GRS) est maintenant pris en charge dans l’émulateur de stockage. Les API `Get Blob Service Stats`, `Get Queue Service Stats` et `Get Table Service Stats` sont prises en charge pour le compte secondaire et renvoient la valeur de l’élément de réponse LastSyncTime comme heure actuelle en fonction de la base de données SQL sous-jacente. Pour un accès par programmation au stockage secondaire avec l'émulateur de stockage, utilisez la bibliothèque cliente de stockage pour .NET version 3.2 ou ultérieure. Pour plus d’informations, consultez Bibliothèque cliente Microsoft Azure Storage pour .NET.

### <a name="version-30"></a>Version 3.0

* L’émulateur de stockage Azure n’est plus inclus dans le même package que l’émulateur de calcul.
* L’interface utilisateur graphique de l’émulateur de stockage est dépréciée. Elle a été remplacée par une interface de ligne de commande scriptable. Pour plus d’informations sur l’interface de ligne de commande, consultez la section Référence de l’outil en ligne de commande de l’émulateur de stockage. L’interface graphique est toujours présente dans la version 3.0, mais elle est uniquement accessible lorsque l’émulateur de calcul est installé en cliquant avec le bouton droit sur l’icône de la zone de notification, puis en sélectionnant Afficher l’IU de l’émulateur de stockage.
* La version 2013-08-15 des services de stockage Azure est maintenant entièrement prise en charge. (Auparavant, cette version était uniquement prise en charge par la version préliminaire de l’émulateur de stockage version 2.2.1.)

## <a name="next-steps"></a>Étapes suivantes

* Évaluez l’émulateur de stockage open source [Azurite](https://github.com/azure/azurite) multiplateforme et géré par la communauté. 
* L’article [Exemples de stockage Azure avec .NET](./storage-samples-dotnet.md) contient des liens vers plusieurs exemples de code que vous pouvez utiliser lorsque vous développez votre application.
* Vous pouvez utiliser [l’Explorateur Stockage Microsoft Azure](https://storageexplorer.com) pour travailler avec des ressources dans votre compte de stockage cloud et dans l’émulateur de stockage.

## <a name="see-also"></a>Voir aussi

* [Développement de Stockage Azure local avec Azurite, les kits de développement logiciel (SDK) Azure et l’Explorateur Stockage Azure](https://blog.jongallant.com/2020/04/local-azure-storage-development-with-azurite-azuresdks-storage-explorer/)
