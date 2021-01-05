---
title: Créer une application Azure Service Fabric Reliable Actors Java sur Linux
description: Découvrez comment créer et déployer une application Java Service Fabric Reliable Actors en cinq minutes.
ms.topic: conceptual
ms.date: 06/18/2018
ms.custom: devx-track-java
ms.openlocfilehash: 7d87b72437f86d7dc1ca4e3cf9f3d67609691c70
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655949"
---
# <a name="create-your-first-java-service-fabric-reliable-actors-application-on-linux"></a>Création de votre première application Java Service Fabric Reliable Actors
> [!div class="op_single_selector"]
> * [Java - Linux](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

Ce guide de démarrage rapide vous aide à créer votre première application Azure Service Fabric Java dans un environnement de développement Linux en quelques minutes.  Lorsque vous aurez terminé, vous disposerez d’une simple application de service unique Java exécutée sur le cluster de développement local.  

## <a name="prerequisites"></a>Prérequis
Avant de commencer, installez le Kit de développement logiciel (SDK), l’interface de ligne de commande Service Fabric, Yeoman, configurez l’environnement de développement Java, puis configurez un cluster de développement dans votre [environnement de développement Linux](service-fabric-get-started-linux.md). Si vous utilisez Mac OS X, vous pouvez [configurer un environnement de développement sur Mac à l’aide de Docker](service-fabric-get-started-mac.md).

Installez aussi l’[interface de ligne de commande (CLI) de Service Fabric](service-fabric-cli.md).

### <a name="install-and-set-up-the-generators-for-java"></a>Installer et configurer les générateurs de Java
Service Fabric fournit des outils de génération de modèles automatique qui vous aideront à créer une application Java Service Fabric à partir d’un terminal à l’aide du générateur de modèle Yeoman.  Si Yeoman n’est pas déjà installé, consultez [Prise en main de Service Fabric avec Linux](service-fabric-get-started-linux.md#set-up-yeoman-generators-for-containers-and-guest-executables) pour obtenir des instructions sur la configuration de Yeoman. Exécutez la commande suivante pour installer le générateur de modèles Yeoman Service Fabric pour Java.

  ```bash
  npm install -g generator-azuresfjava
  ```

## <a name="basic-concepts"></a>Concepts de base
Pour prendre en main Reliable Actors, il vous suffit de comprendre quelques concepts de base :

* **Service d’acteur**. Les entités Reliable Actors sont empaquetées dans des Reliable Services qui peuvent être déployés dans l’infrastructure Service Fabric. Les instances d’acteur sont activées dans une instance de service nommée.
* **Enregistrement d’acteur**. Comme avec les Reliable Services, un service Reliable Actor doit être enregistré avec le runtime Service Fabric. En outre, le type d’acteur doit être enregistré auprès du runtime de l’acteur.
* **Interface d’acteur**. L’interface d’acteur est utilisée pour définir une interface publique fortement typée d’un acteur. Dans la terminologie du modèle Reliable Actor, l’interface d’acteur définit les types de messages que l’acteur peut comprendre et traiter. L’interface d’acteur est utilisée par d’autres acteurs et applications clientes pour « envoyer » (de façon asynchrone) des messages à l’acteur. Reliable Actors peut implémenter plusieurs interfaces.
* **Classe ActorProxy**. La classe ActorProxy est utilisée par des applications clientes pour appeler les méthodes exposées par le biais de l’interface d’acteur. La classe ActorProxy fournit deux fonctionnalités importantes :

  * Résolution de noms : Elle est en mesure de localiser l’acteur dans le cluster (rechercher le nœud du cluster dans lequel il est hébergé).
  * Gestion des échecs : Elle peut retenter les appels de méthode et résoudre de nouveau l’emplacement de l’acteur après une défaillance qui nécessite le déplacement de l’acteur vers un autre nœud du cluster, par exemple.

Les règles suivantes, qui se rapportent aux interfaces d’acteur, sont importantes :

* Les méthodes d'interface d'acteur ne peuvent pas être surchargées.
* Les méthodes d’interface d'acteur ne doivent pas avoir de paramètres de sortie, de paramètres de référence ou de paramètres facultatifs.
* Les interfaces génériques ne sont pas prises en charge.

## <a name="create-the-application"></a>Création de l'application
Une application Service Fabric contient un ou plusieurs services, chacun ayant un rôle précis pour la fourniture de la fonctionnalité d’application. Le générateur que vous avez installé lors de la dernière section facilite la création de votre premier service et vous aide à ajouter de nouveaux services ultérieurement.  Vous pouvez également créer, générer et déployer des applications Java Service Fabric à l’aide d’un plug-in d’Eclipse. Consultez les instructions de [création et déploiement de votre première application Java à l’aide d’Eclipse](service-fabric-get-started-eclipse.md). Pour ce démarrage rapide, utilisez Yeoman pour créer une application avec un service unique qui stocke et obtient une valeur de compteur.

1. Saisissez ``yo azuresfjava`` dans un terminal.
2. Donnez un nom à votre application.
3. Choisissez le type de votre premier service et nommez-le. Pour ce didacticiel, choisissez un service Reliable Actors. Pour plus d’informations sur les autres types de services, voir [Vue d’ensemble des modèles de programmation Service Fabric](service-fabric-choose-framework.md).
   ![Générateur Yeoman Service Fabric pour Java][sf-yeoman]

Si vous nommez l’application « HelloWorldActorApplication » et l’acteur « HelloWorldActor », les modèles automatiques suivants sont générés :

```bash
HelloWorldActorApplication/
├── build.gradle
├── HelloWorldActor
│   ├── build.gradle
│   ├── settings.gradle
│   └── src
│       └── reliableactor
│           ├── HelloWorldActorHost.java
│           └── HelloWorldActorImpl.java
├── HelloWorldActorApplication
│   ├── ApplicationManifest.xml
│   └── HelloWorldActorPkg
│       ├── Code
│       │   ├── entryPoint.sh
│       │   └── _readme.txt
│       ├── Config
│       │   ├── _readme.txt
│       │   └── Settings.xml
│       ├── Data
│       │   └── _readme.txt
│       └── ServiceManifest.xml
├── HelloWorldActorInterface
│   ├── build.gradle
│   └── src
│       └── reliableactor
│           └── HelloWorldActor.java
├── HelloWorldActorTestClient
│   ├── build.gradle
│   ├── settings.gradle
│   ├── src
│   │   └── reliableactor
│   │       └── test
│   │           └── HelloWorldActorTestClient.java
│   └── testclient.sh
├── install.sh
├── settings.gradle
└── uninstall.sh
```
## <a name="reliable-actors-basic-building-blocks"></a>Blocs de construction de base des acteurs fiables
Les concepts de base décrits précédemment se traduisent par les blocs de construction de base d’un service Reliable Actor.

### <a name="actor-interface"></a>Interface d’acteur
Elle contient la définition d'interface de l'acteur. Cette interface définit le contrat de l’acteur qui est partagé par l’implémentation de l’acteur et les clients appelant l’acteur, donc il est généralement justifié de le définir dans un endroit distinct de l’implémentation de l’acteur et qui peut être partagé par plusieurs autres services ou application clientes.

`HelloWorldActorInterface/src/reliableactor/HelloWorldActor.java`:

```java
public interface HelloWorldActor extends Actor {
    @Readonly   
    CompletableFuture<Integer> getCountAsync();

    CompletableFuture<?> setCountAsync(int count);
}
```

### <a name="actor-service"></a>Service d’acteur
Il contient votre implémentation d’acteur et le code d’inscription de l’acteur. La classe d’acteur implémente l’interface d’acteur. Il s’agit de l’endroit dans quelquel votre acteur effectue son travail.

`HelloWorldActor/src/reliableactor/HelloWorldActorImpl`:

```java
@ActorServiceAttribute(name = "HelloWorldActorService")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class HelloWorldActorImpl extends FabricActor implements HelloWorldActor {
    private Logger logger = Logger.getLogger(this.getClass().getName());

    public HelloWorldActorImpl(FabricActorService actorService, ActorId actorId){
        super(actorService, actorId);
    }

    @Override
    protected CompletableFuture<?> onActivateAsync() {
        logger.log(Level.INFO, "onActivateAsync");

        return this.stateManager().tryAddStateAsync("count", 0);
    }

    @Override
    public CompletableFuture<Integer> getCountAsync() {
        logger.log(Level.INFO, "Getting current count value");
        return this.stateManager().getStateAsync("count");
    }

    @Override
    public CompletableFuture<?> setCountAsync(int count) {
        logger.log(Level.INFO, "Setting current count value {0}", count);
        return this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value);
    }
}
```

### <a name="actor-registration"></a>Enregistrement d’acteur
Le service de l’acteur doit être enregistré avec un type de service dans le runtime Service Fabric. Afin que le service de l’acteur exécute vos instances d’acteur, le type d’acteur doit également être enregistré auprès du Service de l’acteur. La méthode d’inscription `ActorRuntime` effectue ce travail pour les acteurs.

`HelloWorldActor/src/reliableactor/HelloWorldActorHost`:

```java
public class HelloWorldActorHost {

private static final Logger logger = Logger.getLogger(HelloWorldActorHost.class.getName());

public static void main(String[] args) throws Exception {

        try {

            ActorRuntime.registerActorAsync(HelloWorldActorImpl.class, (context, actorType) -> new FabricActorService(context, actorType, (a,b)-> new HelloWorldActorImpl(a,b)), Duration.ofSeconds(10));
            Thread.sleep(Long.MAX_VALUE);
        } catch (Exception e) {
            logger.log(Level.SEVERE, "Exception occurred", e);
            throw e;
        }
    }
}
```

## <a name="build-the-application"></a>Créer l’application
Les modèles Yeoman Service Fabric incluent un script de build pour [Gradle](https://gradle.org/), que vous pouvez utiliser pour générer l’application à partir du terminal.
Les dépendances Java Service Fabric ont été extraites de Maven. Pour générer et travailler sur les applications Java Service Fabric, assurez-vous d’avoir installé JDK et Gradle. S’ils ne sont pas installés, consultez [Prise en main de Service Fabric avec Linux](service-fabric-get-started-linux.md#set-up-java-development) pour obtenir des instructions sur l’installation de JDK et Gradle.

Pour générer et placer l’application dans un package, exécutez ce qui suit :

  ```bash
  cd HelloWorldActorApplication
  gradle
  ```

## <a name="deploy-the-application"></a>Déployer l’application
Une fois que l’application est générée, vous pouvez la déployer sur le cluster local.

1. Connectez-vous au cluster Service Fabric local (le cluster doit être [configuré et en cours d’exécution](service-fabric-get-started-linux.md#set-up-a-local-cluster)).

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. Exécutez le script d’installation fourni dans le modèle pour copier le package d’application dans le magasin d’images du cluster, inscrire le type d’application et créer une instance de l’application.

    ```bash
    ./install.sh
    ```

L’application générée se déploie de la même manière qu’une autre application Service Fabric. Consultez la documentation sur [la gestion d’une application Service Fabric avec l’interface de ligne de commande Service Fabric](service-fabric-application-lifecycle-sfctl.md) pour obtenir des instructions détaillées.

Vous pourrez retrouver les paramètres de ces commandes dans les manifestes générés au sein du package d’application.

Une fois l’application déployée, ouvrez un navigateur et accédez à [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md), à l’adresse `http://localhost:19080/Explorer`.
Ensuite, développez le nœud **Applications** et notez qu’il existe désormais une entrée pour votre type d’application et une autre pour la première instance de ce type.

> [!IMPORTANT]
> Pour déployer l’application sur un cluster Linux sécurisé dans Azure, vous devez configurer un certificat pour valider votre application avec le runtime Service Fabric. Ainsi, vos services Reliable Actors peuvent communiquer avec les API d’exécution du runtime Service Fabric sous-jacentes. Pour en savoir plus, consultez [Configurer une application Reliable Services à exécuter sur les clusters Linux](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters).  
>

## <a name="start-the-test-client-and-perform-a-failover"></a>Démarrer le client de test et effectuer un basculement
Les acteurs ne font rien seuls, ils ont besoin d’un autre service ou client pour leur envoyer des messages. Le modèle d’acteur inclut un script de test simple que vous pouvez utiliser pour interagir avec le service d’acteur.

> [!Note]
> Le client de test utilise la classe ActorProxy pour communiquer avec des acteurs, qui doivent s’exécuter dans le même cluster que le service d’acteur ou partager le même espace d’adressage IP.  Vous pouvez exécuter le client de test sur le même ordinateur que le cluster de développement local.  Toutefois, pour communiquer avec des acteurs dans un cluster distant, vous devez déployer une passerelle sur le cluster qui gère la communication externe avec les acteurs.

1. Exécutez le script à l’aide de l’utilitaire watch pour afficher la sortie du service d’acteur.  Le script de test appelle la méthode `setCountAsync()` sur l’acteur pour incrémenter un compteur, appelle la méthode `getCountAsync()` sur l’acteur pour obtenir la nouvelle valeur du compteur et affiche cette valeur dans la console.

   Sous MAC OS X, vous devez copier le dossier HelloWorldTestClient vers certains emplacements situés dans le conteneur en exécutant les commandes supplémentaires suivantes.    

    ```bash
     docker cp HelloWorldTestClient [first-four-digits-of-container-ID]:/home
     docker exec -it [first-four-digits-of-container-ID] /bin/bash
     cd /home
     ```

    ```bash
    cd HelloWorldActorTestClient
    watch -n 1 ./testclient.sh
    ```

2. Dans Service Fabric Explorer, recherchez le nœud qui héberge le réplica principal pour le service d’acteur. Dans la capture d’écran ci-dessous, il s’agit du nœud 3. Le réplica principal de service gère les opérations de lecture et d’écriture.  Les modifications d'état de service sont ensuite propagées aux réplicas secondaires, en cours d'exécution sur les nœuds 0 et 1 dans la capture d'écran ci-dessous.

    ![Recherche du réplica principal dans Service Fabric Explorer][sfx-primary]

3. Dans **Nœuds**, cliquez sur le nœud trouvé à l’étape précédente, puis sélectionnez **Désactiver (redémarrer)** dans le menu Actions. Cette action permet de redémarrer le nœud qui exécute le réplica de service principal et de forcer un basculement sur l’un des réplicas secondaires s’exécutant sur un autre nœud.  Ce réplica secondaire est promu au rang de réplica principal, un autre réplica secondaire est créé sur un autre nœud, et le réplica principal commence à effectuer des opérations de lecture/écriture. Pendant le redémarrage du nœud, prêtez attention à la sortie du client de test et notez que le compteur continue à être incrémenté malgré le basculement.

## <a name="remove-the-application"></a>Supprimer l’application
Utilisez le script de désinstallation fourni dans le modèle pour supprimer l’instance d’application, désinscrire le package d’application et le supprimer de magasin d’images du cluster.

```bash
./uninstall.sh
```

Dans Service Fabric Explorer, vous voyez que l’application et le type d’application n’apparaissent plus dans le nœud **Applications**.

## <a name="service-fabric-java-libraries-on-maven"></a>Bibliothèques Java Service Fabric sur Maven
Les bibliothèques Java Service Fabric ont été hébergées dans Maven. Vous pouvez ajouter les dépendances dans les éléments ``pom.xml`` ou ``build.gradle`` de vos projets pour utiliser les bibliothèques Java Service Fabric à partir du référentiel **mavenCentral**. 

### <a name="actors"></a>Acteurs

Assistance Reliable Actor de Service Fabric pour votre application.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-actors</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-actors:1.0.0'
  }
  ```

### <a name="services"></a>Services

Assistance Reliable Services de Service Fabric pour votre application.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-services</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-services:1.0.0'
  }
  ```

### <a name="others"></a>Autres
#### <a name="transport"></a>Transport

Assistance de la couche transport pour application Java Service Fabric. Il est inutile d’ajouter explicitement cette dépendance à votre service Reliable Actor ou à vos Applications de service, à moins d’effectuer la programmation au niveau de la couche transport.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-transport</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-transport:1.0.0'
  }
  ```

#### <a name="fabric-support"></a>Assista,ce Fabric

Assistance du niveau système pour Service Fabric, qui communique avec le runtime Service Fabric. Vous n’avez pas besoin d’ajouter explicitement cette dépendance à vos applications Reliable Actor ou Service Fabric. Elle est extraite automatiquement depuis Maven, lorsque vous incluez les autres dépendances ci-dessus.

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf:1.0.0'
  }
  ```

## <a name="next-steps"></a>Étapes suivantes

* [Création de votre première application Java Service Fabric sur Linux à l’aide d’Eclipse](service-fabric-get-started-eclipse.md)
* [Présentation des Acteurs fiables Service Fabric](service-fabric-reliable-actors-introduction.md)
* [Interagir avec les clusters Service Fabric à l’aide de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)
* [Prise en main de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png
