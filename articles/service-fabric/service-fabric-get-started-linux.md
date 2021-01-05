---
title: Configurer votre environnement de développement sur Linux
description: Installez le runtime et le Kit de développement logiciel (SDK), puis créez un cluster de développement local sur Linux. Une fois la configuration terminée, vous êtes prêt à générer des applications.
ms.topic: conceptual
ms.date: 10/16/2020
ms.custom: devx-track-js
ms.openlocfilehash: 14b8a278605a908b4182c724831b2e42de54a753
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93086888"
---
# <a name="prepare-your-development-environment-on-linux"></a>Préparer votre environnement de développement sur Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)

Pour déployer et exécuter des [applications Azure Service Fabric](service-fabric-application-model.md) sur votre ordinateur de développement Linux, installez le runtime et le Kit de développement logiciel (SDK) courant. Vous pouvez également installer des Kits de développement logiciel (SDK) facultatifs pour le développement de Java et .NET Core. 

La procédure décrite dans cet article suppose que vous effectuez une installation en mode natif sur Linux ou que vous utilisez [l’image conteneur OneBox Service Fabric](https://hub.docker.com/_/microsoft-service-fabric-onebox), c.-à-d. `mcr.microsoft.com/service-fabric/onebox:u18`.

L’interface de ligne de commande (CLI) Azure Service Fabric vous permet de gérer les entités Service Fabric hébergées dans le cloud ou en local. Pour plus d’informations sur l’installation de l’interface de ligne de commande, consultez [Installation de l’interface de ligne de commande Service Fabric](./service-fabric-cli.md).


## <a name="prerequisites"></a>Prérequis

Ces versions de système d’exploitation sont prises en charge pour le développement.

* Ubuntu 16.04 (`Xenial Xerus`), 18.04 (`Bionic Beaver`)

    Assurez-vous que le package `apt-transport-https` est bien installé.
         
    ```bash
    sudo apt-get install apt-transport-https
    ```
* Red Hat Enterprise Linux 7.4 (prise en charge de la préversion de Service Fabric)


## <a name="installation-methods"></a>Méthodes d’installation

<!-- markdownlint-disable MD025 -->
<!-- markdownlint-disable MD024 -->

# <a name="ubuntu"></a>[Ubuntu](#tab/sdksetupubuntu)

## <a name="update-your-apt-sources"></a>Mise à jour des sources APT
Pour installer le Kit de développement logiciel (SDK) et le package runtime associé via l’outil en ligne de commande apt-get, vous devez tout d’abord mettre à jour les sources Advanced Packaging Tool (APT).

## <a name="script-installation"></a>Installation de script

Un script est fourni pour faciliter l’installation du runtime Service Fabric et du Kit de développement logiciel (SDK) Service Fabric courant avec [**l’interface de ligne de commande** sfctl](service-fabric-cli.md). L’exécution du script suppose que vous acceptez les licences de tous les logiciels en cours d’installation. Vous pouvez également effectuer les étapes [d’installation manuelle](#manual-installation) de la section suivante, qui présente les licences associées, ainsi que les composants en cours d’installation.

Une fois que le script s’exécute, vous pouvez passer directement à la section [Configurer un cluster local](#set-up-a-local-cluster).

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

## <a name="manual-installation"></a>Installation manuelle
Suivez les étapes restantes de ce guide pour réaliser une installation manuelle du runtime Service Fabric et du Kit de développement logiciel (SDK) courant.

1. Ouvrez un terminal.

2. Ajoutez le référentiel `dotnet` à votre liste de sources correspondant à votre distribution.

    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

3. Ajoutez la nouvelle clé de protection de la confidentialité MS Open Tech Gnu (GnuPG ou GPG) à votre porte-clés APT.

    ```bash
    sudo curl -fsSL https://packages.microsoft.com/keys/msopentech.asc | sudo apt-key add -
    ```

4. Ajoutez les clés GPG Docker officielles à votre porte-clés APT.

    ```bash
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

5. Configurer le référentiel de Docker.

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

6. Ajouter la clé Azul JDK à votre porte-clés APT et configurer son référentiel.

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9
    sudo apt-add-repository "deb http://repos.azul.com/azure-only/zulu/apt stable main"
    ```

7. Actualisez vos listes de packages selon les référentiels nouvellement ajoutés.

    ```bash
    sudo apt-get update
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster"></a>Installation et configuration du SDK Service Fabric pour un cluster local

Après avoir mis à jour vos sources, vous pouvez installer le SDK. Installez le package SDK Service Fabric, confirmez l’installation, puis acceptez le contrat de licence.

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

> [!TIP]
>   Les commandes suivantes automatisent la validation de la licence pour les packages Service Fabric :
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

# <a name="red-hat-enterprise-linux-74"></a>[Red Hat Enterprise Linux 7.4](#tab/sdksetuprhel74)

## <a name="update-your-yum-repositories"></a>Mettre à jour vos référentiels Yum
Pour installer le Kit de développement logiciel (SDK) et le package runtime associé via l’outil en ligne de commande yum, vous devez tout d’abord mettre à jour les sources de package.

## <a name="manual-installation-rhel"></a>Installation manuelle (RHEL)
Suivez les étapes restantes de ce guide pour réaliser une installation manuelle du runtime Service Fabric et du Kit de développement logiciel (SDK) courant.

1. Ouvrez un terminal.
2. Téléchargez et installez Extra Packages for Enterprise Linux (EPEL).

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. Ajoutez à votre système le référentiel de packages EfficiOS RHEL7.

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. Importez la clé de signature du package EfficiOS dans le porte-clés GPG local.

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. Ajoutez à votre système le référentiel RHEL Microsoft.

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster-rhel"></a>Installation et configuration du SDK Service Fabric pour un cluster local (RHEL)

Après avoir mis à jour vos sources, vous pouvez installer le SDK. Installez le package SDK Service Fabric, confirmez l’installation, puis acceptez le contrat de licence.

```bash
sudo yum install servicefabricsdkcommon
```

---

## <a name="included-packages"></a>Packages inclus
Le runtime Service Fabric qui est fourni avec l’installation du SDK ci-dessus inclut les packages indiqués dans le tableau ci-dessous. 

 | | DotNetCore | Java | Python | NodeJS | 
--- | --- | --- | --- |---
**Ubuntu** | 2.0.7 | AzulJDK 1.8 | Implicite à partir de npm | latest |
**RHEL** | - | OpenJDK 1.8 | Implicite à partir de npm | latest |

## <a name="set-up-a-local-cluster"></a>Configurer un cluster local
1. Démarrez un cluster Service Fabric local pour le développement.

# <a name="container-based-local-cluster"></a>[Cluster local basé sur un conteneur](#tab/localclusteroneboxcontainer)

Démarrez un cluster [Service Fabric Onebox](https://hub.docker.com/_/microsoft-service-fabric-onebox) basé sur un conteneur.

1. Installez Moby pour pouvoir déployer des conteneurs Docker.
    ```bash
    sudo apt-get install moby-engine moby-cli -y
    ```
2. Mettez à jour la configuration du démon Docker sur votre ordinateur hôte avec les paramètres suivants, puis redémarrez le démon Docker. Détails : [Activer la prise en charge d’IPv6](https://docs.docker.com/config/daemon/ipv6/)

    ```json
    {
        "ipv6": true,
        "fixed-cidr-v6": "fd00::/64"
    }
    ```

3. Démarrez le cluster.<br/>
    <b>Ubuntu 18.04 LTS :</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    <b>Ubuntu 16.04 LTS :</b>
    ```bash
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u16
    ```

    >[!TIP]
    > Par défaut, cela extraira l’image avec la dernière version de Service Fabric. Pour des révisions particulières, visitez la page [Docker Hub](https://hub.docker.com/r/microsoft/service-fabric-onebox/).

# <a name="local-cluster"></a>[Cluster local](#tab/localcluster)

Après avoir installé le kit de développement logiciel (SDK) en suivant les étapes ci-dessus, démarrez un cluster local.

1. Exécutez le script de configuration du cluster.

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

---

2. Ouvrez un navigateur web et accédez à **Service Fabric Explorer** (`http://localhost:19080/Explorer`). Au moment où le cluster démarre, vous accédez au tableau de bord Service Fabric Explorer. La configuration complète du cluster peut prendre plusieurs minutes. Si votre navigateur ne parvient pas à ouvrir l’URL, ou si Service Fabric Explorer n’indique pas que le système est prêt, patientez quelques minutes, puis réessayez.

    ![Service Fabric Explorer sur Linux][sfx-linux]

    À ce stade, vous pouvez déployer des packages d’application Service Fabric prédéfinis ou de nouveaux packages basés sur des conteneurs invités ou des exécutables invités. Pour générer de nouveaux services à l’aide des Kits de développement logiciel (SDK) .NET Core ou Java, suivez les étapes de configuration facultatives fournies dans les sections suivantes.


> [!NOTE]
> Les clusters autonomes ne sont pas pris en charge sous Linux.


> [!TIP]
> Si l’un de vos disques SSD est disponible, nous vous recommandons d’utiliser `--clusterdataroot` avec devclustersetup.sh pour transférer un chemin d’accès au dossier du disque SSD et ainsi accroître les performances.

## <a name="set-up-the-service-fabric-cli"></a>Configurer l’interface de ligne de commande Service Fabric

L’[interface de ligne de commande Service Fabric](service-fabric-cli.md) inclut des commandes permettant d’interagir avec des entités Service Fabric, y compris des clusters et des applications. Suivez les instructions de [l’interface de ligne de commande Service Fabric](service-fabric-cli.md) pour installer l’interface de ligne de commande.


## <a name="set-up-yeoman-generators-for-containers-and-guest-executables"></a>Configurer les générateurs Yeoman pour des conteneurs et des exécutables invités
Service Fabric fournit des outils de génération de modèles automatique qui vous aideront à créer des applications Service Fabric à partir d’un terminal, à l’aide des générateurs de modèle Yeoman. Suivez ces étapes pour configurer les générateurs de modèle Yeoman Service Fabric :

1. Installez Node.js et npm sur votre machine.

    ```bash
    sudo add-apt-repository "deb https://deb.nodesource.com/node_8.x $(lsb_release -s -c) main"
    sudo apt-get update
    sudo apt-get install nodejs
    ```
2. Installez le générateur de modèles [Yeoman](https://yeoman.io/) sur votre machine à partir de npm.

    ```bash
    sudo npm install -g yo
    ```
3. Installez le générateur de conteneurs Service Fabric Yeo et le générateur d’exécutables invités à partir de npm.

    ```bash
    sudo npm install -g generator-azuresfcontainer  # for Service Fabric container application
    sudo npm install -g generator-azuresfguest      # for Service Fabric guest executable application
    ```

Après avoir installé ces générateurs, vous devez être en mesure de créer un exécutable invité ou des services de conteneurs en exécutant respectivement `yo azuresfguest` ou `yo azuresfcontainer`.

## <a name="set-up-net-core-31-development"></a>Configurer le développement de .NET Core 3.1

Installez le [Kit de développement logiciel (SDK) .NET Core 3.1 pour Ubuntu](https://www.microsoft.com/net/core#linuxubuntu) pour commencer à [créer des applications Service Fabric C#](service-fabric-create-your-first-linux-application-with-csharp.md). Les packages pour les applications .NET Core Service Fabric sont hébergés sur NuGet.org.

## <a name="set-up-java-development"></a>Configurer le développement Java

Afin de générer des services Service Fabric à l’aide de Java, installez Gradle pour exécuter des tâches de build. Exécutez la commande ci-dessous pour installer Gradle. Les bibliothèques Java Service Fabric sont extraites à partir de Maven.


* Ubuntu

    ```bash
    curl -s https://get.sdkman.io | bash
    sdk install gradle 5.1
    ```

* Red Hat Enterprise Linux 7.4 (prise en charge de la préversion de Service Fabric)

  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```

Vous devez également installer le générateur Service Fabric Yeo pour les exécutables Java. Assurez-vous que [Yeoman est installé](#set-up-yeoman-generators-for-containers-and-guest-executables), puis exécutez la commande suivante :

  ```bash
  npm install -g generator-azuresfjava
  ```
 
## <a name="install-the-eclipse-plug-in-optional"></a>Installer le plug-in Eclipse (facultatif)

Vous pouvez installer le plug-in Eclipse pour Service Fabric à partir de l’environnement IDE Eclipse pour les développeurs Java ou Java EE. Vous pouvez utiliser Eclipse pour créer des applications exécutables invités Service Fabric et des applications de conteneur en plus des applications Service Fabric Java.

> [!IMPORTANT]
> Le plug-in Service Fabric nécessite Eclipse Neon ou une version ultérieure. Consultez les instructions qui suivent cette note pour savoir comment vérifier votre version d’Eclipse. Si vous avez installé une version antérieure d’Eclipse, vous pouvez télécharger des versions les plus récentes à partir du [site Eclipse](https://www.eclipse.org). Nous vous déconseillons de procéder à l’installation par-dessus une installation existante d’Eclipse (par écrasement). Vous pouvez soit la supprimer avant d’exécuter le programme d’installation, soit installer la version la plus récente dans un autre répertoire.
> 
> Sur Ubuntu, nous vous recommandons de procéder à l’installation directement depuis le site Eclipse au lieu d’utiliser le programme d’installation d’un package (`apt` ou `apt-get`). Ceci vous assure d’obtenir la version la plus récente d’Eclipse. Vous pouvez installer l’environnement IDE pour les développeurs Java ou Java EE.

1. Dans Eclipse, assurez-vous d’avoir installé Eclipse Neon ou version ultérieure et Buildship version 2.2.1 ou version ultérieure. Vérifiez les versions des composants installés en sélectionnant **Aide** > **About Eclipse** (À propos d’Eclipse) > **Détails de l’installation**. Vous pouvez mettre à jour Buildship en utilisant les instructions [Eclipse Buildship : plug-ins Eclipse pour Gradle][buildship-update].

2. Pour installer le plug-in Service Fabric, sélectionnez **Aide** > **Installer un nouveau logiciel**.

3. Dans la zone **Work with** (Utiliser avec), entrez **https:\//dl.microsoft.com/eclipse**.

4. Sélectionnez **Ajouter**.

    ![Page des logiciels disponibles][sf-eclipse-plugin]

5. Sélectionnez le plug-in **Service Fabric** , puis cliquez sur **Suivant**.

6. Suivez la procédure d’installation. Acceptez ensuite le contrat de licence de l’utilisateur final.

Si le plug-in Eclipse Service Fabric est déjà installé, assurez-vous de disposer de la dernière version. Pour le vérifier, sélectionnez **Aide** > **About Eclipse** (À propos d’Eclipse) > **Détails de l’installation**. Recherchez ensuite le plug-in Service Fabric dans la liste des plug-ins installés. Sélectionnez **Mettre à jour** si une version plus récente est disponible.

Pour plus d’informations, consultez la section [Plug-in Service Fabric pour le développement d’applications Java sous Eclipse](service-fabric-get-started-eclipse.md).

## <a name="update-the-sdk-and-runtime"></a>Mise à jour du kit de développement logiciel (SDK) et du runtime

Exécutez les commandes suivantes pour mettre à jour vers la dernière version du Kit de développement logiciel (SDK) et du runtime.

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
Pour mettre à jour les fichiers binaires du kit de développement logiciel (SDK) Java à partir de Maven, vous devez mettre à jour les détails de la version du fichier binaire correspondant dans le fichier ``build.gradle`` pour pointer vers la version la plus récente. Pour savoir exactement où mettre à jour la version, vous pouvez vous référer à n’importe quel fichier ``build.gradle`` dans les [exemples de prise en main de Service Fabric](https://github.com/Azure-Samples/service-fabric-java-getting-started).

> [!NOTE]
> La mise à jour des packages peut entraîner l’arrêt de votre cluster de développement local. Après une mise à niveau, suivez les instructions de cet article pour redémarrer le cluster local.

## <a name="remove-the-sdk"></a>Supprimez le Kit de développement logiciel (SDK)
Pour supprimer les Kits de développement logiciel (SDK) Service Fabric, exécutez les commandes suivantes.

* Ubuntu

    ```bash
    sudo apt-get remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    sudo apt-get install -f
    ```


* Red Hat Enterprise Linux 7.4 (prise en charge de la préversion de Service Fabric)

    ```bash
    sudo yum remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Créer et déployer votre première application Java Service Fabric sous Linux à l’aide de Yeoman](service-fabric-create-your-first-linux-application-with-java.md)
* [Créer et déployer votre première application Java Service Fabric sous Linux à l’aide du plug-in Service Fabric pour Eclipse](service-fabric-get-started-eclipse.md)
* [Create your first Java application on Linux (Créer votre première application Java sur Linux)](service-fabric-create-your-first-linux-application-with-csharp.md)
* [Prepare your development environment on OSX (Préparer votre environnement de développement sur OSX)](service-fabric-get-started-mac.md)
* [Préparer un environnement de développement Linux sur Windows](service-fabric-local-linux-cluster-windows.md)
* [Gérer une application Azure Service Fabric à l’aide d’Azure Service Fabric CLI](service-fabric-application-lifecycle-sfctl.md)
* [Différences entre Service Fabric sur Linux et Windows](service-fabric-linux-windows-differences.md)
* [Prise en main de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
