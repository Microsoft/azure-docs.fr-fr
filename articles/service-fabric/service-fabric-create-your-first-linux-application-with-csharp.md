---
title: Créer votre première application Azure Service Fabric sur Linux à l’aide de C#
description: Découvrez comment créer et déployer une application Service Fabric à l’aide de C# et de .NET Core 2.0.
ms.topic: conceptual
ms.date: 04/11/2018
ms.openlocfilehash: 556d8dee4efd492fa98755f1ffd1cdc1c9887856
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82193647"
---
# <a name="create-your-first-azure-service-fabric-application"></a>Créer votre première application Azure Service Fabric
> [!div class="op_single_selector"]
> * [Java - Linux (préversion)](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux (préversion)](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

Service Fabric fournit des Kits de développement logiciel (SDK) pour générer des services Linux dans .NET Core et Java. Dans ce didacticiel, nous apprenons à créer une application pour Linux et à générer un service à l’aide de C# sur .NET Core 2.0.

## <a name="prerequisites"></a>Prérequis
Avant de commencer, assurez-vous que vous avez bien [configuré votre environnement de développement Linux](service-fabric-get-started-linux.md). Si vous utilisez Mac OS X, vous pouvez [configurer un environnement Linux à boîtier unique sur une machine virtuelle à l’aide de Vagrant](service-fabric-get-started-mac.md).

Il vous sera également utile d’installer l’[interface de ligne de commande Service Fabric](service-fabric-cli.md)

### <a name="install-and-set-up-the-generators-for-c"></a>Installer et configurer des générateurs pour C#
Service Fabric fournit des outils de génération de modèles automatique qui vous aideront à créer des applications Service Fabric à partir d’un terminal, à l’aide des générateurs de modèle Yeoman. Suivez ces étapes pour configurer les générateurs de modèle Service Fabric Yeoman pour C# :

1. Installer nodejs et NPM sur votre machine

   ```bash
   curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash 
   nvm install node 
   ```
2. Installer le générateur de modèles [Yeoman](https://yeoman.io/) sur votre machine à partir de NPM

   ```bash
   npm install -g yo
   ```
3. Installer le générateur d’applications Service Fabric Yeoman C# à partir de NPM

   ```bash
   npm install -g generator-azuresfcsharp
   ```

## <a name="create-the-application"></a>Création de l'application
Une application Service Fabric peut contenir un ou plusieurs services, chacun ayant un rôle précis pour la fourniture de la fonctionnalité d’application. Le générateur Service Fabric [Yeoman](https://yeoman.io/) pour C#, que vous avez installé lors de la dernière étape, facilite la création de votre premier service et vous aide à ajouter de nouveaux services ultérieurement. Nous utilisons Yeoman pour créer une application avec un seul service.

1. Dans un terminal, tapez la commande suivante pour commencer la génération de modèles automatique : `yo azuresfcsharp`
2. Donnez un nom à votre application.
3. Choisissez le type de votre premier service et nommez-le. Pour les besoins de ce didacticiel, nous choisissons un service Reliable Actors.

   ![Générateur Yeoman Service Fabric pour C#][sf-yeoman]

> [!NOTE]
> Pour plus d’informations sur les options, voir [Vue d’ensemble des modèles de programmation Service Fabric](service-fabric-choose-framework.md).
>
>

## <a name="build-the-application"></a>Créer l’application
Les modèles Yeoman Service Fabric incluent un script de build que vous pouvez utiliser pour générer l’application à partir du terminal (après avoir accédé au dossier l’application).

  ```sh
 cd myapp
 ./build.sh
  ```

## <a name="deploy-the-application"></a>Déployer l’application

Une fois que l’application est générée, vous pouvez la déployer sur le cluster local.

1. Connectez-vous au cluster Service Fabric local.

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. Exécutez le script d’installation fourni dans le modèle pour copier le package d’application dans le magasin d’images du cluster, inscrire le type d’application et créer une instance de l’application.

    ```bash
    ./install.sh
    ```

L’application générée se déploie de la même manière qu’une autre application Service Fabric. Consultez la documentation sur [la gestion d’une application Service Fabric avec l’interface de ligne de commande Service Fabric](service-fabric-application-lifecycle-sfctl.md) pour obtenir des instructions détaillées.

Vous pourrez retrouver les paramètres de ces commandes dans les manifestes générés au sein du package d’application.

Une fois l’application déployée, ouvrez un navigateur et accédez à [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md), à l’adresse `http://localhost:19080/Explorer`. Ensuite, développez le nœud **Applications** et notez qu’il existe désormais une entrée pour votre type d’application et une autre pour la première instance de ce type.

> [!IMPORTANT]
> Pour déployer l’application sur un cluster Linux sécurisé dans Azure, vous devez configurer un certificat pour valider votre application avec le runtime Service Fabric. Ainsi, vos services Reliable Services peuvent communiquer avec les API d’exécution du runtime Service Fabric sous-jacentes. Pour en savoir plus, consultez [Configurer une application Reliable Services à exécuter sur les clusters Linux](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters).  
>

## <a name="start-the-test-client-and-perform-a-failover"></a>Démarrer le client de test et effectuer un basculement
Les projets d’acteur n’effectuent aucune opération automatiquement. Ils ont besoin d’un autre service ou client pour leur envoyer des messages. Le modèle d’acteur inclut un script de test simple que vous pouvez utiliser pour interagir avec le service d’acteur.

1. Exécutez le script à l’aide de l’utilitaire watch pour afficher la sortie du service d’acteur.

   Sous MAC OS X, vous devez copier le dossier myactorsvcTestClient vers le même emplacement situé dans le conteneur en exécutant les commandes supplémentaires suivantes.
    
    ```bash
    docker cp  [first-four-digits-of-container-ID]:/home
    docker exec -it [first-four-digits-of-container-ID] /bin/bash
    cd /home
    ```
    
    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```
2. Dans Service Fabric Explorer, recherchez le nœud qui héberge le réplica principal pour le service d’acteur. Dans la capture d’écran ci-dessous, il s’agit du nœud 3.

    ![Recherche du réplica principal dans Service Fabric Explorer][sfx-primary]
3. Cliquez sur le nœud trouvé à l’étape précédente, puis sélectionnez **Désactiver (redémarrer)** à partir du menu Actions. Cette action redémarre un nœud de votre cluster local et force un basculement sur un réplica secondaire s’exécutant sur un autre nœud. Dans le même temps, prêtez attention à la sortie du client de test et notez que le compteur continue à être incrémenté malgré le basculement.

## <a name="adding-more-services-to-an-existing-application"></a>Ajout d’autres services à une application existante

Pour ajouter un autre service à une application déjà créée à l’aide de `yo`, procédez comme suit :
1. Accédez au répertoire à la racine de l’application existante.  Par exemple, `cd ~/YeomanSamples/MyApplication`, si `MyApplication` est l’application créée par Yeoman.
2. Exécutez `yo azuresfcsharp:AddService`

## <a name="next-steps"></a>Étapes suivantes

* [Interaction avec les clusters Service Fabric à l’aide de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)
* [Prise en main de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-csharp/yeoman-csharp.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-csharp/sfx-primary.png
