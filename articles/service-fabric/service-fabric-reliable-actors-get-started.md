---
title: Créer un service basé sur un acteur sur Azure Service Fabric
description: Découvrez comment créer, déboguer et déployer votre premier service basé sur un acteur en C# à l’aide de Service Fabric Reliable Actors.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 225ccb67153a33ed47af68ebb1549dce37426278
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573459"
---
# <a name="getting-started-with-reliable-actors"></a>Prise en main de Reliable Actors
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-actors-get-started.md)
> * [Java sur Linux](./service-fabric-create-your-first-linux-application-with-java.md)

Cet article décrit la création et le débogage d'une simple application Reliable Actors dans Visual Studio. Pour plus d’informations sur Reliable Actors, consultez l’article [Présentation du modèle Reliable Actors de Service Fabric](service-fabric-reliable-actors-introduction.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous d’avoir configuré l’environnement de développement Service Fabric, y compris Visual Studio, sur votre ordinateur. Pour plus de détails, voir [Configuration de l’environnement de développement](service-fabric-get-started.md).

## <a name="create-a-new-project-in-visual-studio"></a>Création d'un projet dans Visual Studio

Lancez Visual Studio 2019 ou version ultérieure en tant qu’administrateur, puis créez un projet d’**application Service Fabric** :

![Outils Service Fabric pour Visual Studio - nouveau projet][1]

Dans la boîte de dialogue suivante, choisissez **Service d’acteur** sous **.NET Core 2.0** et nommez le service.

![Modèles de projets Service Fabric][5]

Le projet créé affiche la structure suivante :

![Structure d’un projet Service Fabric][2]

## <a name="examine-the-solution"></a>Examiner la solution

La solution inclut trois projets :

* **Le projet d’application (MyApplication)** . Ce projet regroupe tous les services pour le déploiement. Il contient *ApplicationManifest.xml* et les scripts PowerShell pour gérer l’application.

* **Le projet d’interface (HelloWorld.Interfaces)** . Ce projet contient la définition d'interface de l'acteur. Les interfaces d'acteur peuvent être définies dans n'importe quel projet avec n'importe quel nom.  L'interface définit le contrat d'acteur qui est partagé par l'implémentation de l'acteur et les clients appelant l'acteur.  Comme les projets client peuvent en dépendre, il est généralement logique de le définir dans un assembly distinct de l'implémentation de l'acteur.

* **Le projet de service de l’acteur (HelloWorld)** . Ce projet définit le service Service Fabric qui va héberger l'acteur. Il contient l’implémentation de l’acteur *HelloWorld.cs*. Une implémentation de l’acteur est une classe qui dérive d’un type de base `Actor` et implémente les interfaces définies dans le projet *MyActor.Interfaces*. Une classe d’acteur doit également implémenter un constructeur qui accepte une `ActorService` instance et un `ActorId` et les transmet à la classe de base `Actor`.
    
    Ce projet contient également *Program.cs*, qui inscrit les classes d'acteur auprès du runtime Service Fabric en utilisant `ActorRuntime.RegisterActorAsync<T>()`. La classe `HelloWorld` est déjà inscrite. Toute implémentation d'acteur supplémentaire ajoutée au projet doit également être inscrite dans la méthode `Main()`.

## <a name="customize-the-helloworld-actor"></a>Personnaliser l'acteur HelloWorld

Le modèle de projet définit certaines méthodes dans l'interface `IHelloWorld` et les intègre à l'implémentation de l'acteur `HelloWorld`.  Remplacez ces méthodes pour que le service d'acteur renvoie une chaîne « Hello World » simple.

Dans le projet *HelloWorld.Interfaces*, dans le fichier *IHelloWorld.cs*, remplacez la définition de l'interface comme suit :

```csharp
public interface IHelloWorld : IActor
{
    Task<string> GetHelloWorldAsync();
}
```

Dans le projet **HelloWorld**, dans **HelloWorld.cs**, remplacez la totalité de la définition de la classe comme suit :

```csharp
[StatePersistence(StatePersistence.Persisted)]
internal class HelloWorld : Actor, IHelloWorld
{
    public HelloWorld(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<string> GetHelloWorldAsync()
    {
        return Task.FromResult("Hello from my reliable actor!");
    }
}
```

Appuyez sur **Ctrl-Maj-B** pour construire le projet et vérifier que tous les éléments sont correctement compilés.

## <a name="add-a-client"></a>Ajouter un client

Créez une application de console simple pour appeler le service d'acteur.

1. Cliquez avec le bouton droit sur la solution dans l'Explorateur de solutions > **Ajouter** > **Nouveau projet...** .

2. Dans les types de projet **.NET Core**, choisissez **Console App (.NET Core)** .  Nommez le projet *ActorClient*.
    
    ![Boîte de dialogue Ajouter un nouveau projet][6]    
    
    > [!NOTE]
    > Une application de console n’est pas le type d’application que vous utilisez généralement comme client dans Service Fabric, mais elle constitue un exemple pratique pour le débogage et le test à l’aide du cluster Service Fabric local.

3. L'application console doit être une application 64 bits pour maintenir la compatibilité avec le projet d'interface et les autres dépendances.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ActorClient**, puis cliquez sur **Propriétés**.  Dans l'onglet **Build**, définissez **Plateforme cible** sur **x64**.
    
    ![Propriétés de la build][8]

4. Le projet client nécessite le package NuGet des acteurs fiables.  Cliquez sur **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.  Dans la Console du gestionnaire de package, entrez la commande suivante :
    
    ```powershell
    Install-Package Microsoft.ServiceFabric.Actors -IncludePrerelease -ProjectName ActorClient
    ```

    Le package NuGet et toutes ses dépendances sont installés dans le projet ActorClient.

5. Le projet client nécessite également une référence au projet d'interfaces.  Dans le projet ActorClient, cliquez avec le bouton droit sur **Dépendances**, puis cliquez sur **Ajouter une référence de projet...** .  Sélectionnez **Projets > Solution** (le cas échéant), puis cochez la case **HelloWorld.Interfaces**.  Cliquez sur **OK**.
    
    ![Boîte de dialogue Ajouter une référence][7]

6. Dans le projet ActorClient, remplacez la totalité du contenu de *Program.cs* par le code suivant :
    
    ```csharp
    using System;
    using System.Threading.Tasks;
    using Microsoft.ServiceFabric.Actors;
    using Microsoft.ServiceFabric.Actors.Client;
    using HelloWorld.Interfaces;
    
    namespace ActorClient
    {
        class Program
        {
            static void Main(string[] args)
            {
                IHelloWorld actor = ActorProxy.Create<IHelloWorld>(ActorId.CreateRandom(), new Uri("fabric:/MyApplication/HelloWorldActorService"));
                Task<string> retval = actor.GetHelloWorldAsync();
                Console.Write(retval.Result);
                Console.ReadLine();
            }
        }
    }
    ```

## <a name="running-and-debugging"></a>Exécution et débogage

Appuyez sur **F5** pour créer, déployer et exécuter l'application localement dans le cluster de développement Service Fabric.  Pendant le processus de déploiement, vous pouvez afficher la progression dans la fenêtre **Sortie** .

![Fenêtre de sortie de débogage Service Fabric][3]

Si la sortie contient le texte *L'application est prête*, vous pouvez tester le service à l'aide de l'application ActorClient.  Dans l'Explorateur de solutions, cliquez avec le bouton droit sur le projet **ActorClient**, puis cliquez sur **Déboguer** > **Démarrer une nouvelle instance**.  L'application de ligne de commande devrait afficher la sortie du service d'acteur.

![Sortie de l’application][9]

> [!TIP]
> Le runtime Service Fabric Actors émet des [événements et compteurs de performances liés aux méthodes d’acteur](service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters). Ces événements sont utiles dans les diagnostics et la surveillance des performances.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [la façon dont les Reliable Actors utilisent la plateforme Service Fabric](service-fabric-reliable-actors-platform.md).


[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG
[6]: ./media/service-fabric-reliable-actors-get-started/new-console-app.png
[7]: ./media/service-fabric-reliable-actors-get-started/add-reference.png
[8]: ./media/service-fabric-reliable-actors-get-started/build-props.png
[9]: ./media/service-fabric-reliable-actors-get-started/app-output.png
