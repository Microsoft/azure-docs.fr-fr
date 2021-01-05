---
title: Utiliser l’émulateur azurite pour le développement de stockage Azure local
description: L’émulateur open source Azurite fournit un environnement local gratuit, qui vous permet de tester vos applications du Stockage Azure.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/15/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: b5051a5e29f9e34c43963a9a264ee47e2ad7b04a
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490779"
---
# <a name="use-the-azurite-emulator-for-local-azure-storage-development"></a>Utiliser l’émulateur azurite pour le développement de stockage Azure local

L’émulateur open source Azurite (préversion) fournit un environnement local gratuit, qui vous permet de tester vos applications du Stockage Blob et de file d'attente Azure. Une fois que vous êtes satisfait de la manière dont votre application fonctionne localement, passez à l’utilisation d’un compte de stockage Azure dans le cloud. L’émulateur offre une prise en charge multiplateforme, qui inclut Windows, Linux et macOS.

Azurite est la future plateforme d’émulateur de stockage. Azurite remplace l’[Émulateur de stockage Azure](storage-use-emulator.md). Azurite continuera d’être mis à jour pour prendre en charge les dernières versions des API de stockage Azure.

Il existe plusieurs manières différentes d’installer et d’exécuter Azurite sur votre système local :

  1. [Installer et exécuter l’extension Visual Studio Code Azurite](#install-and-run-the-azurite-visual-studio-code-extension)
  1. [Installer et exécuter Azurite à l’aide de NPM](#install-and-run-azurite-by-using-npm)
  1. [Installer et exécuter l’image Docker Azurite](#install-and-run-the-azurite-docker-image)
  1. [Cloner, générer et exécuter Azurite à partir du dépôt GitHub](#clone-build-and-run-azurite-from-the-github-repository)

## <a name="install-and-run-the-azurite-visual-studio-code-extension"></a>Installer et exécuter l’extension Visual Studio Code Azurite

Dans Visual Studio Code, sélectionnez le volet **EXTENSIONS** , puis recherchez *Azurite* dans **EXTENSIONS : PLACE DE MARCHÉ** .

![Place de marché des extensions Visual Studio Code](media/storage-use-azurite/azurite-vs-code-extension.png)

Vous pouvez également accéder au marché des extensions pour [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=Azurite.azurite) dans votre navigateur. Sélectionnez le bouton **Installer** pour ouvrir Visual Studio Code et accéder directement à la page de l’extension Azurite.

L’extension prend en charge les commandes Visual Studio Code suivantes : Dans Visual Studio Code, appuyez sur F1 pour ouvrir la palette de commandes. 

   - **Azurite : Nettoyer** - Réinitialiser toutes les données de persistance des services Azurite
   - **Azurite : Nettoyer le service blob** - Nettoyer le service blob
   - **Azurite : Nettoyer le service de file d'attente** - Nettoyer le service de file d'attente
   - **Azurite : Fermer** - Fermer tous les services Azurite
   - **Azurite : Fermer le service blob** - Fermer le service BLOB
   - **Azurite : Fermer le service de file d’attente** - Fermer le service de file d’attente
   - **Azurite : Démarrer** - Démarrer tous les services Azurite
   - **Azurite : Démarrer le service Blob** - Démarrer le service Blob
   - **Azurite : Démarrer le service de file d’attente** - Démarrer le service de file d’attente

Pour configurer Azurite dans Visual Studio Code, sélectionnez le volet Extensions. Sélectionnez l'icône **Gérer** (engrenage) pour **Azurite** . Sélectionnez **Extension Settings** (Paramètres d’extension)

![Configuration des paramètres de l’extension Azurite](media/storage-use-azurite/azurite-configure-extension-settings.png)

Les paramètres suivants sont pris en charge :

   - **Azurite : Blob Host** (Hôte Blob) - Point de terminaison d’écoute du service Blob. Le paramètre par défaut est 127.0.0.1.
   - **Azurite : Blob Port** (Port Blob) - Port d’écoute du service Blob. Le port par défaut est 10000.
   - **Azurite : Cert**  : chemin d’accès vers un fichier PEM ou PFX approuvé localement pour activer le mode HTTPS.
   - **Azurite : Debug** (Débogage) - Sortie du journal de débogage sur le canal Azurite. La valeur par défaut est **false** .
   - **Azurite : Key**  : chemin d’accès vers un fichier de clé PEM approuvé localement, requis lorsque **Azurite: Cert** pointe vers un fichier PEM.
   - **Azurite : Location** (Emplacement) - Chemin de l’emplacement de l’espace de travail. Par défaut, il s’agit du dossier de travail de Visual Studio Code.
   - **Azurite : Loose**  : active le mode libre, qui ignore les en-têtes et les paramètres non pris en charge.
   - **Azurite : OAuth**  : niveau OAuth facultatif.
   - **Azurite : Pwd**  : mot de passe pour un fichier PFX. Obligatoire lorsque **Azurite: Cert** pointe vers un fichier PFX.
   - **Azurite : Hôte de file d'attente** - Point de terminaison d’écoute du service de file d'attente. Le paramètre par défaut est 127.0.0.1.
   - **Azurite : Port de la file d’attente**  - Le port d’écoute du service de file d’attente. Le port par défaut est 10001.
   - **Azurite : Silent** (Sans assistance) - Ce mode entraîne la désactivation du journal des accès. La valeur par défaut est **false** .
   - **Azurite : Ignorer la vérification de la version de l’API** : ignorer la vérification de la version de l’API de requête. La valeur par défaut est **false** .

## <a name="install-and-run-azurite-by-using-npm"></a>Installer et exécuter Azurite à l’aide de NPM

Cette méthode d’installation nécessite l’installation préalable de [Node.js version 8.0 ou ultérieure](https://nodejs.org). Node Package Manager (npm) est l’outil de gestion des packages inclus dans chaque installation Node.js. Après avoir installé Node.js, exécutez la commande `npm` suivante pour installer Azurite.

```console
npm install -g azurite
```

Après avoir installé Azurite, consultez [Exécuter Azurite à partir d’une ligne de commande](#run-azurite-from-a-command-line).

## <a name="install-and-run-the-azurite-docker-image"></a>Installer et exécuter l’image Docker Azurite

Utilisez [DockerHub](https://hub.docker.com/) pour effectuer le tirage (pull) de l’[image Azurite la plus récente](https://hub.docker.com/_/microsoft-azure-storage-azurite) à l’aide de la commande suivante :

```console
docker pull mcr.microsoft.com/azure-storage/azurite
```

**Exécuter l’image Docker Azurite**  :

La commande suivante exécute l’image Docker Azurite. Le paramètre `-p 10000:10000` redirige les requêtes du port 10000 de la machine hôte vers l’instance de Docker.

```console
docker run -p 10000:10000 -p 10001:10001 \
    mcr.microsoft.com/azure-storage/azurite
```

**Spécifier l’emplacement de l’espace de travail**  :

Dans l’exemple suivant, le paramètre `-v c:/azurite:/data` spécifie *c:/azurite* en tant qu’emplacement des données persistantes d’Azurite. Le répertoire *c:/azurite* doit être créé avant l’exécution de la commande Docker.

```console
docker run -p 10000:10000 -p 10001:10001 \
    -v c:/azurite:/data mcr.microsoft.com/azure-storage/azurite
```

**Exécuter uniquement le service blob**

```console
docker run -p 10000:10000 mcr.microsoft.com/azure-storage/azurite \
    azurite-blob --blobHost 0.0.0.0 --blobPort 10000
```

Pour plus d’informations sur la configuration d’Azurite au démarrage, consultez la section [Options de ligne de commande](#command-line-options).

## <a name="clone-build-and-run-azurite-from-the-github-repository"></a>Cloner, générer et exécuter Azurite à partir du dépôt GitHub

Cette méthode d’installation nécessite l’installation préalable de [Git](https://git-scm.com/). Clonez le [dépôt GitHub](https://github.com/azure/azurite) du projet Azurite à l’aide de la commande de console suivante.

```console
git clone https://github.com/Azure/Azurite.git
```

Après avoir cloné le code source, exécutez les commandes suivantes à partir de la racine du dépôt cloné pour générer et installer Azurite.

```console
npm install
npm run build
npm install -g
```

Après avoir installé et généré Azurite, consultez [Exécuter Azurite à partir d’une ligne de commande](#run-azurite-from-a-command-line).

## <a name="run-azurite-from-a-command-line"></a>Exécuter Azurite à partir d’une ligne de commande

> [!NOTE]
> Vous ne pouvez pas exécuter Azurite à partir de la ligne de commande si vous avez installé uniquement l’extension Visual Studio Code. Utilisez plutôt la palette de commandes de Visual Studio Code. Pour plus d’informations, consultez [Installer et exécuter l’extension Visual Studio Code Azurite](#install-and-run-the-azurite-visual-studio-code-extension).

Pour commencer immédiatement à l’aide de la ligne de commande, créez un répertoire appelé *c:\azurite* , puis lancez Azurite en émettant la commande suivante :

```console
azurite --silent --location c:\azurite --debug c:\azurite\debug.log
```

Cette commande indique à Azurite de stocker toutes les données dans un répertoire particulier, *c:\azurite* . Si l’option `--location` est omise, le répertoire de travail actif est utilisé.

## <a name="command-line-options"></a>Options de ligne de commande

Cette section détaille les commutateurs de ligne de commande disponibles quand vous lancez Azurite.

### <a name="help"></a>Aide

**Facultatif**  : permet d’obtenir de l’aide sur la ligne de commande en utilisant le commutateur `-h` ou `--help`.

```console
azurite -h
azurite --help
```

### <a name="blob-listening-host"></a>Hôte d’écoute Blob

**Facultatif** Par défaut, Azurite écoute le serveur local à l’adresse 127.0.0.1. Utilisez le commutateur `--blobHost` pour définir l’adresse en fonction de vos exigences.

Accepter les requêtes sur la machine locale uniquement :

```console
azurite --blobHost 127.0.0.1
```

Autoriser les requêtes distantes :

```console
azurite --blobHost 0.0.0.0
```

> [!CAUTION]
> L’autorisation des requêtes distantes peut rendre votre système vulnérable aux attaques externes.

### <a name="blob-listening-port-configuration"></a>Configuration du port d’écoute Blob

**Facultatif** Par défaut, Azurite écoute le service Blob sur le port 10000. Utilisez le commutateur `--blobPort` pour spécifier le port d’écoute nécessaire.

> [!NOTE]
> Après avoir utilisé un port personnalisé, vous devez mettre à jour la chaîne de connexion ou la configuration correspondante dans vos outils du service Stockage Azure ou vos kits SDK.

Personnaliser le port d’écoute du service Blob :

```console
azurite --blobPort 8888
```

Laisser le système sélectionner automatiquement un port disponible :

```console
azurite --blobPort 0
```

Le port utilisé s’affiche au démarrage d’Azurite.

### <a name="queue-listening-host"></a>Hôte d’écoute de file d'attente

**Facultatif** Par défaut, Azurite écoute le serveur local à l’adresse 127.0.0.1. Utilisez le commutateur `--queueHost` pour définir l’adresse en fonction de vos exigences.

Accepter les requêtes sur la machine locale uniquement :

```console
azurite --queueHost 127.0.0.1
```

Autoriser les requêtes distantes :

```console
azurite --queueHost 0.0.0.0
```

> [!CAUTION]
> L’autorisation des requêtes distantes peut rendre votre système vulnérable aux attaques externes.

### <a name="queue-listening-port-configuration"></a>Configuration du port d’écoute de file d'attente

**Facultatif** Par défaut, Azurite écoute le service de file d’attente sur le port 10001. Utilisez le commutateur `--queuePort` pour spécifier le port d’écoute nécessaire.

> [!NOTE]
> Après avoir utilisé un port personnalisé, vous devez mettre à jour la chaîne de connexion ou la configuration correspondante dans vos outils du service Stockage Azure ou vos kits SDK.

Personnaliser le port d’écoute du service de file d'attente :

```console
azurite --queuePort 8888
```

Laisser le système sélectionner automatiquement un port disponible :

```console
azurite --queuePort 0
```

Le port utilisé s’affiche au démarrage d’Azurite.

### <a name="workspace-path"></a>Chemin de l’espace de travail

**Facultatif** Azurite stocke les données sur le disque local durant l’exécution. Utilisez le commutateur `-l` ou `--location` pour spécifier un chemin en tant qu’emplacement de l’espace de travail. Par défaut, le répertoire de travail du processus actuel est utilisé. Remarquez le « l » minuscule.

```console
azurite -l c:\azurite
azurite --location c:\azurite
```

### <a name="access-log"></a>Journal d’accès

**Facultatif** Le journal des accès s’affiche par défaut dans la fenêtre de console. Désactivez l’affichage du journal des accès à l’aide du commutateur `-s` ou `--silent`.

```console
azurite -s
azurite --silent
```
### <a name="debug-log"></a>Journal de débogage

**Facultatif** Le journal de débogage contient des informations détaillées sur chaque arborescence des appels de procédure relative aux requêtes et aux exceptions. Activez le journal de débogage en fournissant un chemin de fichier local valide au commutateur `-d` ou `--debug`.

```console
azurite -d path/debug.log
azurite --debug path/debug.log
```

### <a name="loose-mode"></a>Mode relâché

**Facultatif** Par défaut, Azurite applique le mode strict pour bloquer les en-têtes et les paramètres de requête non pris en charge. Désactivez le mode strict en utilisant le commutateur `-L` ou `--loose`. Remarquez le « L » majuscule.

```console
azurite -L
azurite --loose
```
### <a name="version"></a>Version

**Facultatif**  : permet d’afficher le numéro de la version d’Azurite installée en utilisant le commutateur `-v` ou `--version`.

```console
azurite -v
azurite --version
```

### <a name="certificate-configuration-https"></a>Configuration de certificat (HTTPS)

**Facultatif**  : par défaut, Azurite utilise le protocole HTTP. Activez le mode HTTPS en fournissant un chemin d’accès à un fichier de certificat Privacy Enhanced Mail (. PEM) ou [Personal Information Exchange (.pfx)](/windows-hardware/drivers/install/personal-information-exchange---pfx--files) au commutateur `--cert`.

Lorsque `--cert` est fourni pour un fichier PEM, vous devez fournir un commutateur `--key` correspondant.

```console
azurite --cert path/server.pem --key path/key.pem
```

Lorsque `--cert` est fourni pour un fichier PFX, vous devez fournir un commutateur `--pwd` correspondant.

```console
azurite --cert path/server.pfx --pwd pfxpassword
```

Pour plus d’informations sur la création de fichiers PEM et PFX, consultez la section [Configuration HTTPS](https://github.com/Azure/Azurite/blob/master/README.md#https-setup).

### <a name="oauth-configuration"></a>Configuration OAuth

**Facultatif**  : activez l’authentification OAuth pour Azurite à l’aide du commutateur `--oauth`.

```console
azurite --oauth basic --cert path/server.pem --key path/key.pem
```

> [!NOTE]
> OAuth requiert un point de terminaison HTTPS. Assurez-vous que le protocole HTTPS est activé en fournissant un commutateur `--cert` avec le commutateur `--oauth`.

Azurite prend en charge l’authentification de base en spécifiant le paramètre `basic` au commutateur `--oauth`. Azurite effectue l’authentification de base, par exemple la validation du jeton de porteur entrant, la vérification de l’émetteur, de l’audience et de l’expiration. Azurite ne vérifie pas la signature du jeton ou les autorisations.

### <a name="skip-api-version-check"></a>Ignorer la vérification de la version de l’API

**Facultatif** : au démarrage, Azurite vérifie que la version de l’API demandée est valide. La commande suivante ignore la vérification de la version de l’API :

```console
azurite --skipApiVersionCheck
```


## <a name="authorization-for-tools-and-sdks"></a>Autorisation pour les outils et les kits SDK

Connectez-vous à Azurite à partir des kits SDK ou des outils du service Stockage Azure, par exemple l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/), à l’aide d’une stratégie d’authentification. L’authentification est obligatoire. Azurite prend en charge l’autorisation avec OAuth, basée sur une clé partagée et les SAP (signatures d’accès partagé). Azurite prend également en charge l’accès anonyme aux conteneurs publics.

Si vous utilisez les kits de développement logiciel (SDK) Azure, démarrez Azurite avec les options `--oauth basic and --cert --key/--pwd`.

### <a name="well-known-storage-account-and-key"></a>Clé et compte de stockage reconnus

Ce compte et cette clé reconnus sont les mêmes que ceux utilisés par l’émulateur de stockage Azure hérité. Azurite les accepte.

- Nom du compte : `devstoreaccount1`
- Clé de compte : `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`

### <a name="custom-storage-accounts-and-keys"></a>Comptes et clés de stockage personnalisés

Azurite prend en charge les noms de compte et les clés de stockage personnalisés via la définition de la variable d’environnement `AZURITE_ACCOUNTS` au format suivant : `account1:key1[:key2];account2:key1[:key2];...`.

Par exemple, utilisez un compte de stockage personnalisé qui a une clé :

```cmd
set AZURITE_ACCOUNTS="account1:key1"
```

```bash
export AZURITE_ACCOUNTS="account1:key1"
```

Vous pouvez aussi utiliser plusieurs comptes de stockage avec deux clés chacun :

```cmd
set AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

```bash
export AZURITE_ACCOUNTS="account1:key1:key2;account2:key1:key2"
```

Par défaut, Azurite actualise les noms de compte et les clés personnalisés à partir de la variable d’environnement toutes les minutes. Avec cette fonctionnalité, vous pouvez effectuer dynamiquement une rotation de la clé de compte ou ajouter de nouveaux comptes de stockage sans redémarrer Azurite.

> [!NOTE]
> Le compte de stockage `devstoreaccount1` par défaut est désactivé quand vous définissez des comptes de stockage personnalisés.

### <a name="connection-strings"></a>Chaînes de connexion

La méthode la plus simple pour vous connecter à Azurite à partir de votre application consiste à configurer une chaîne de connexion dans le fichier config de votre application, qui référence le raccourci *UseDevelopmentStorage=true* . Voici un exemple de chaîne de connexion dans un fichier *app.config*  :

```xml
<appSettings>
  <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
</appSettings>
```

#### <a name="http-connection-strings"></a>Chaînes de connexion HTTP

Vous pouvez transmettre les chaînes de connexion suivantes aux [Kits de développement logiciel Azure (SDK)](https://aka.ms/azsdk) ou aux outils, comme Azure CLI 2.0 ou Explorateur Stockage.

La chaîne de connexion complète est :

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

Pour vous connecter uniquement au service Blob, utilisez la chaîne de connexion suivante :

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;`

Pour vous connecter uniquement au service File d’attente, utilisez la chaîne de connexion suivante :

`DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;`

#### <a name="https-connection-strings"></a>Chaînes de connexion HTTPS

La chaîne de connexion HTTPS complète est :

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Pour utiliser uniquement le service BLOB, utilisez la chaîne de connexion HTTPS :

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;`

Pour utiliser uniquement le service File d’attente, utilisez la chaîne de connexion HTTPS :

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;`

Si vous avez utilisé `dotnet dev-certs` pour générer votre certificat auto-signé, utilisez la chaîne de connexion suivante :

`DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://localhost:10000/devstoreaccount1;QueueEndpoint=https://localhost:10001/devstoreaccount1;`

Mettez à jour la chaîne de connexion lorsque vous utilisez des [clés et comptes de stockage personnalisés](#custom-storage-accounts-and-keys).

Pour plus d’informations, consultez [Configuration des chaînes de connexion Stockage Azure](storage-configure-connection-string.md).

### <a name="azure-sdks"></a>SDK Azure

Pour utiliser Azurite avec les [Kits de développement logiciel Azure (SDK)](https://aka.ms/azsdk), utilisez les options OAuth et HTTPS :

```console
azurite --oauth basic --cert certname.pem --key certname-key.pem
```

#### <a name="azure-blob-storage"></a>Stockage Blob Azure

Vous pouvez ensuite instancier un BlobContainerClient, BlobServiceClient ou BlobClient.

```csharp
// With container URL and DefaultAzureCredential
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new BlobContainerClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "container-name"
  );

// With account name and key
var client = new BlobContainerClient(
    new Uri("https://127.0.0.1:10000/devstoreaccount1/container-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

#### <a name="azure-queue-storage"></a>Stockage File d’attente Azure

Vous pouvez également instancier un QueueClient ou un QueueServiceClient.

```csharp
// With queue URL and DefaultAzureCredential
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"), new DefaultAzureCredential()
  );

// With connection string
var client = new QueueClient(
    "DefaultEndpointsProtocol=https;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=https://127.0.0.1:10000/devstoreaccount1;QueueEndpoint=https://127.0.0.1:10001/devstoreaccount1;", "queue-name"
  );

// With account name and key
var client = new QueueClient(
    new Uri("https://127.0.0.1:10001/devstoreaccount1/queue-name"),
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
```

### <a name="microsoft-azure-storage-explorer"></a>Explorateur Stockage Microsoft Azure

Vous pouvez utiliser Explorateur Stockage pour afficher les données stockées dans Azurite.

#### <a name="connect-to-azurite-using-http"></a>Se connecter à Azurite en utilisant le HTTP

Dans Explorateur Stockage, connectez-vous à Azurite en procédant comme suit :

 1. Sélectionnez l’icône **Manage Accounts** (Gérer les comptes)
 1. Sélectionnez **Ajouter un compte** .
 1. Sélectionnez **Attacher à un émulateur local**
 1. Sélectionnez **Suivant** .
 1. Entrez le nom de votre choix dans le champ **Display name** (Nom complet)
 1. Sélectionnez **Suivant** de nouveau
 1. Sélectionnez **Se connecter** .

#### <a name="connect-to-azurite-using-https"></a>Se connecter à Azurite en utilisant le HTTPS

Par défaut Explorateur Stockage n’ouvre pas de point de terminaison HTTPS qui utilise un certificat auto-signé. Si vous exécutez Azurite avec le HTTPS, il vous faudra probablement utiliser un certificat auto-signé. Dans Explorateur Stockage, importez des certificats SSL via les boîtes de dialogue **Edit** (Modifier)  -> **SSL Certificates** (Certificats SSL)  -> **Import Certificates** (Importer des certificats).

##### <a name="import-certificate-to-storage-explorer"></a>Importer un certificat dans Explorateur Stockage

1. Recherchez le certificat sur votre ordinateur local.
1. Dans Explorateur Stockage, accédez à **Edit** (Modifier)  -> **SSL Certificates** (Certificats SSL)  -> **Import Certificates** (Importer des certificats), puis importez votre certificat.

Si vous n’importez pas de certificat, vous obtiendrez une erreur :

`unable to verify the first certificate` ou `self signed certificate in chain`

##### <a name="add-azurite-via-https-connection-string"></a>Ajouter Azurite via une chaîne de connexion HTTPS

Pour ajouter Azurite HTTPS à Explorateur Stockage, procédez comme suit :

1. Sélectionnez **Activer/désactiver l'Explorateur**
1. Sélectionnez **Local & Attached** (Local et joint)
1. Cliquez avec le bouton droit sur **Storage Accounts** (Comptes de stockage), puis sélectionnez **Connect to Azure Storage** (Se connecter à Stockage Azure).
1. Sélectionnez **Utiliser une chaîne de connexion** .
1. Sélectionnez **Suivant** .
1. Entrez une valeur dans le champ **Nom complet** .
1. Entrez la chaîne de connexion [HTTPS](#https-connection-strings) récupérée dans la section précédente de ce document
1. Sélectionnez **Suivant** .
1. Sélectionnez **Se connecter** .

## <a name="workspace-structure"></a>Structure de l’espace de travail

Les fichiers et dossiers suivants peuvent être créés dans l’emplacement de l’espace de travail lors de l’initialisation de Azurite.

- `__blobstorage__` : répertoire contenant les données binaires persistantes du service Blob Azurite
- `__queuestorage__` : répertoire contenant les données binaires persistantes du service File d’attente Azurite
- `__azurite_db_blob__.json` : fichier de métadonnées du service Blob Azurite
- `__azurite_db_blob_extent__.json` : fichier de métadonnées d’extension du service Blob Azurite
- `__azurite_db_queue__.json` : fichier de métadonnées du service File d’attente Azurite
- `__azurite_db_queue_extent__.json` : fichier de métadonnées d’extension du service File d’attente Azurite

Pour nettoyer Azurite, supprimez les fichiers et dossiers ci-dessus, puis redémarrez l’émulateur.

## <a name="differences-between-azurite-and-azure-storage"></a>Différences entre Azurite et Stockage Azure

Il existe des différences fonctionnelles entre une instance locale d’Azurite et un compte de stockage Azure dans le cloud.

### <a name="endpoint-and-connection-url"></a>Point de terminaison et URL de connexion

Les points de terminaison de service pour Azurite se distinguent des points de terminaison d’un compte de stockage Azure. Dans la mesure où l’ordinateur local n’effectue pas de résolution de noms de domaine, les points de terminaison Azurite doivent être des adresses locales.

Quand vous adressez une ressource dans un compte de stockage Azure, le nom du compte fait partie du nom d’hôte de l’URI. La ressource adressée fait partie du chemin de l’URI :

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

L’URI suivant est une adresse valide pour un objet blob dans un compte de stockage Azure :

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

Dans la mesure où l’ordinateur local n’effectue pas de résolution de noms de domaine, le nom du compte fait partie du chemin de l’URI au lieu du nom d’hôte. Utilisez le format d’URI suivant pour une ressource dans Azurite :

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

Vous pouvez utiliser l’adresse suivante pour accéder à un objet blob dans Azurite :

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

### <a name="scaling-and-performance"></a>Mise à l’échelle et niveau de performance

Azurite ne prend pas en charge la connexion simultanée d’un grand nombre de clients. Il est impossible de garantir le niveau de performance. Azurite est conçu à des fins de développement et de test.

### <a name="error-handling"></a>Gestion des erreurs

Azurite s’aligne sur la logique de gestion des erreurs du Stockage Azure, mais il existe des différences. Par exemple, les messages d’erreur ne sont pas toujours les mêmes, alors que les codes d’état d’erreur sont identiques.

### <a name="ra-grs"></a>RA-GRS

Azurite prend en charge la réplication géoredondante avec accès en lecture (RA-GRS). Pour les ressources de stockage, accédez à l’emplacement secondaire en ajoutant `-secondary` au nom du compte. Par exemple, vous pouvez utiliser l’adresse suivante pour accéder à un objet blob à l’aide de l’emplacement secondaire en lecture seule dans Azurite :

`http://127.0.0.1:10000/devstoreaccount1-secondary/mycontainer/myblob.txt`

### <a name="table-support"></a>Prise en charge des tables

La prise en charge des tables dans Azurite est actuellement en cours de développement et toutes les contributions sont les bienvenues ! Pour obtenir des informations à jour sur la progression, consultez le projet [Azurite V3 Table](https://github.com/Azure/Azurite/wiki/Azurite-V3-Table).

La prise en charge des fonctions durables requiert des tables.

## <a name="azurite-is-open-source"></a>Azurite est open source

Les contributions et suggestions relatives à Azurite sont les bienvenues. Accédez à la page des [projets GitHub](https://github.com/Azure/Azurite/projects) ou des [problèmes GitHub](https://github.com/Azure/Azurite/issues) relatifs à Azurite pour connaître les jalons et les éléments de travail que nous suivons en ce qui concerne les fonctionnalités à venir et les résolutions de bogues. Les éléments de travail détaillés sont également suivis dans GitHub.

## <a name="next-steps"></a>Étapes suivantes

- L’article [Utilisation de l’émulateur de stockage Azure pour le développement et le test](storage-use-emulator.md) documente l’émulateur de stockage Azure hérité, qui est remplacé par Azurite.
- L’article [Configuration des chaînes de connexion Stockage Azure](storage-configure-connection-string.md) explique comment assembler une chaîne de connexion Stockage Azure valide.
