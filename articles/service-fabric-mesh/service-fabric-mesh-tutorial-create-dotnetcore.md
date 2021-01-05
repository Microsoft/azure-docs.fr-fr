---
title: Créer une application multiservice et la déployer sur Service Fabric Mesh
description: Dans ce didacticiel, vous allez créer une application de maillage multiservice Azure Service Fabric composée d’un site web ASP.NET Core qui communique avec un service web back-end, la déboguer en local et la publier dans Azure.
author: georgewallace
ms.topic: tutorial
ms.date: 09/18/2018
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-csharp
ms.openlocfilehash: fa15432072f6478100a378fdbdb697556fdf1dc9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842544"
---
# <a name="tutorial-create-debug-deploy-and-upgrade-a-multi-service-service-fabric-mesh-app"></a>Tutoriel : Créer, déboguer, déployer et mettre à niveau une application Service Fabric Mesh multiservice

Ce tutoriel est la première partie d’une série d’étapes. Vous allez apprendre à utiliser Visual Studio pour créer une application Azure Service Fabric Mesh comportant un serveur web frontend ASP.NET et un service backend d’API web ASP.NET Core. Vous allez ensuite déboguer l’application dans votre cluster de développement local. Vous allez publier l’application sur Azure, apporter des changements à la configuration et au code, puis mettre à niveau l’application. Enfin, vous allez nettoyer les ressources Azure inutilisées afin d’éviter d’être facturé pour ce que vous n’utilisez pas.

À la fin des opérations, vous aurez effectué la plupart des phases de la gestion du cycle de vie des applications, et généré une application qui illustre un appel de service à service dans une application Service Fabric Mesh.

Si vous ne souhaitez pas créer l’application de tâche manuellement, vous pouvez [télécharger le code source](https://github.com/azure-samples/service-fabric-mesh) pour obtenir l’application terminée et passer directement au [débogage local de l’application](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md).

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Utilisez Visual Studio pour créer une application Service Fabric Mesh composée d’un service web frontend ASP.NET.
> * Créer un modèle pour représenter des éléments de tâche.
> * Créer un service back-end et récupérer des données à partir de celui-ci.
> * Ajouter un contrôleur et une instance DataContext dans le cadre de ce modèle Model View Controller pour le service back-end.
> * Créer une page web pour afficher les éléments de tâche.
> * Créer des variables d’environnement qui identifient le service back-end.

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer une application Service Fabric Mesh dans Visual Studio
> * [Déboguer une application Service Fabric Mesh qui s’exécute dans votre cluster de développement local](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Déployer une application Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [Mettre à niveau une application Service Fabric Mesh](service-fabric-mesh-tutorial-upgrade.md)
> * [Nettoyer des ressources Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Assurez-vous que vous avez [configuré votre environnement de développement](service-fabric-mesh-howto-setup-developer-environment-sdk.md) qui inclut l’installation du runtime Service Fabric, du Kit de développement logiciel (SDK), de Docker et de Visual Studio 2017.

## <a name="create-a-service-fabric-mesh-project-in-visual-studio"></a>Créer un projet Service Fabric Mesh dans Visual Studio

Exécutez Visual Studio et sélectionnez **Fichier** > **Nouveau** > **Projet...** .

Dans le champ **Recherche** situé en haut de la boîte de dialogue **Nouveau projet**, tapez `mesh`. Sélectionnez le modèle **Application Service Fabric mesh**. Si vous ne voyez pas le modèle, assurez-vous que vous avez installé le kit de développement logiciel (SDK) Mesh et la préversion des outils Visual Studio, comme décrit dans la section [Configurer votre environnement de développement](service-fabric-mesh-howto-setup-developer-environment-sdk.md).  

Dans le champ **Nom**, saisissez `todolistapp` et dans le champ **Emplacement**, indiquez le chemin d’accès du dossier dans lequel vous souhaitez enregistrer les fichiers du projet.

Assurez-vous que l’option **Créer un répertoire pour la solution** est activée, puis cliquez sur **OK** pour créer le projet Service Fabric mesh.

![Capture d’écran montrant comment créer un projet Service Fabric Mesh](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-project.png)

Ensuite, la boîte de dialogue **Nouveau service Service Fabric** s’affiche.

### <a name="create-the-web-front-end-service"></a>Créer le service web frontal

Dans la boîte de dialogue **Nouveau service Service Fabric**, sélectionnez le type de projet **ASP.NET Core** et vérifiez que le paramètre **Système d’exploitation de conteneur** est défini sur **Windows**.

Définissez le champ **Nom du service** sur **WebFrontEnd**. Appuyez sur **OK** pour créer le service ASP.NET Core.

![Boîte de dialogue Nouveau projet Service Fabric mesh dans Visual studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-service-fabric-service.png)

Ensuite, vous verrez la boîte de dialogue Application web ASP.NET Core. Sélectionnez **Application web**, puis cliquez sur **OK**.

![Capture d’écran mettant en évidence le modèle Application web](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-aspnetcore-app.png)

Vous disposez désormais d’une application de maillage Service Fabric. Créez ensuite le modèle pour les informations de tâche.

## <a name="create-the-to-do-items-model"></a>Créer le modèle d’éléments de tâche

Pour plus de simplicité, les éléments de tâche sont stockés dans une liste en mémoire. Créez une bibliothèque de classes pour les éléments de tâche et une liste dans laquelle ils seront répertoriés. Dans l’instance de Visual Studio où est chargée la solution **todolistapp**, sélectionnez **Fichier** > **Ajouter** > **Nouveau projet**.

Dans le champ **Recherche** situé en haut de la boîte de dialogue **Ajouter un nouveau projet**, tapez `C# .net core class`. Sélectionnez le modèle **Bibliothèque de classes (.NET Core)** .

Dans le champ **Nom**, saisissez `Model`. Cliquez sur **OK** pour créer la bibliothèque de classes.

Dans l’Explorateur de solutions, sous **Modèle**, cliquez avec le bouton droit sur **Class1.cs** et choisissez **Renommer**. Renommez la classe **ToDoItem.cs**. Lorsqu’une invite s’affiche vous demandant si vous souhaitez renommer toutes les références, cliquez sur **Oui**.

Remplacez le contenu du champ `class ToDoItem` vide avec :

```csharp
public class ToDoItem
{
    public string Description { get; set; }
    public int Index { get; set; }
    public bool Completed { get; set; }

    public ToDoItem(string description)
    {
        Description = description;
        Index = 0;
    }

    public static ToDoItem Load(string description, int index, bool completed)
    {
        ToDoItem newItem = new ToDoItem(description)
        {
            Index = index,
            Completed = completed
        };

        return newItem;
    }
}
```

Cette classe représente les éléments de tâche.

Dans Visual Studio, cliquez avec le bouton droit sur la bibliothèque de classes **Modèle**, puis sélectionnez **Ajouter** > **Classe...** pour créer une liste qui contiendra les éléments de tâche. La boîte de dialogue **Ajouter un nouvel élément** s’affiche. Définissez le champ **Nom** sur `ToDoList.cs` et cliquez sur **Ajouter**.

Dans **ToDoList.cs**, remplacez le champ `class ToDoList` vide avec :

```csharp
public class ToDoList
{
    private List<ToDoItem> _items;

    public string Name { get; set; }
    public IEnumerable<ToDoItem> Items { get => _items; }

    public ToDoList(string name)
    {
        Name = name;
        _items = new List<ToDoItem>();
    }

    public ToDoItem Add(string description)
    {
        var item = new ToDoItem(description);
        _items.Add(item);
        item.Index = _items.IndexOf(item);
        return item;
    }
    public void Add(ToDoItem item)
    {
        _items.Add(item);
        item.Index = _items.Count - 1;
    }

    public ToDoItem RemoveAt(int index)
    {
        if (index >= 0 && index < _items.Count)
        {
            var result = _items[index];
            _items.RemoveAt(index);

            // Reorder items
            for (int i = index; i < _items.Count; i++)
            {
                _items[i].Index = i;
            }

            return result;
        }
        else
        {
            throw new IndexOutOfRangeException();
        }
    }
}
```

Créez ensuite le service Service Fabric qui effectuera le suivi des éléments de tâche.

## <a name="create-the-back-end-service"></a>Créer le service back-end

Dans la fenêtre **Explorateur de solutions** de Visual Studio, cliquez avec le bouton droit sur **todolistapp** et cliquez sur **Ajouter** > **Nouveau service Service Fabric...** .

La boîte de dialogue **Nouveau service Service Fabric** s’affiche. Sélectionnez le type de projet **ASP.NET Core** et vérifiez que le paramètre **Système d’exploitation de conteneur** est défini sur **Windows**. Définissez le champ **Nom du service** sur **ToDoService**. Cliquez sur **OK** pour créer le service ASP.NET Core.

La boîte de dialogue **Nouvelle application web ASP.NET Core** s’affiche. Dans cette boîte de dialogue, sélectionnez **API**, puis **OK**, et un projet pour le service est ajouté à la solution.

![Nouvelle application de ASP.NET Core dans Visual studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-webapi.png)

Étant donné que le service back-end ne fournit pas d’interface utilisateur, désactivez le lancement du navigateur lors du lancement du service. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur **ToDoService** et sélectionnez **Propriétés**. Dans la fenêtre de propriétés qui s’affiche, sélectionnez l’onglet **Déboguer** sur la gauche, puis décochez l’option **Lancer le navigateur**. Appuyez sur **Ctrl+S** pour enregistrer les modifications.

Étant donné que ce service gère les informations de tâche, ajoutez une référence à la bibliothèque de classes Modèle. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **ToDoService**, puis sélectionnez **Ajouter** > **Référence...** . La boîte de dialogue **Gestionnaire de références** s’affiche.

Dans le **Gestionnaire de références**, cochez la case **Modèle** et cliquez sur **OK**.

### <a name="add-a-data-context"></a>Ajouter un contexte de données

Créez ensuite un contexte de données qui coordonne le traitement des données à partir du modèle de données.

Pour ajouter la classe de contexte de données, dans l’Explorateur de solutions, cliquez avec le bouton droit sur **ToDoService**, puis sélectionnez **Ajouter** > **Classe**.
Dans la boîte de dialogue **Ajouter un nouvel élément** qui s’affiche, assurez-vous que l’option **Classe** est sélectionné, définissez le champ **Nom** sur `DataContext.cs` et cliquez sur **Ajouter**.

Dans **DataContext.cs**, remplacez le contenu du champ `class DataContext` vide avec :

```csharp
public static class DataContext
{
    public static Model.ToDoList ToDoList { get; } = new Model.ToDoList("Azure learning List");

    static DataContext()
    {
        // Seed to-do list
        ToDoList.Add(Model.ToDoItem.Load("Learn about microservices", 0, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric", 1, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric Mesh", 2, false));
    }
}
```

Ce contexte de données minimal renseigne certains exemples d’éléments de tâche et permet d’y accéder.

### <a name="add-a-controller"></a>Ajout d'un contrôleur

Un contrôleur par défaut, qui gère les demandes HTPP et crée la réponse HTTP, a été fourni par le modèle lors de la création du projet **ToDoService**. Dans **l’Explorateur de solutions**, sous **ToDoService**, ouvrez le dossier **Contrôleurs** pour afficher le fichier **ValuesController.cs**. 

Cliquez avec le bouton droit sur **ValuesController.cs**, puis sélectionnez **Renommer**. Renommez le fichier `ToDoController.cs`. Si une invite s’affiche vous demandant si vous souhaitez renommer toutes les références, cliquez sur **Oui**.

Ouvrez le fichier **ToDoController.cs** et remplacez le contenu du champ `class ToDoController` avec :

```csharp
[Route("api/[controller]")]
public class ToDoController : Controller
{
    // GET api/todo
    [HttpGet]
    public IEnumerable<Model.ToDoItem> Get()
    {
        return DataContext.ToDoList.Items;
    }

    // GET api/todo/5
    [HttpGet("{index}")]
    public Model.ToDoItem Get(int index)
    {
        return DataContext.ToDoList.Items.ElementAt(index);
    }

    //// POST api/values
    //[HttpPost]
    //public void Post([FromBody]string value)
    //{
    //}

    //// PUT api/values/5
    //[HttpPut("{id}")]
    //public void Put(int id, [FromBody]string value)
    //{
    //}

    // DELETE api/values/5
    [HttpDelete("{index}")]
    public void Delete(int index)
    {
    }
}
```

Ce didacticiel ne couvre pas les opérations d’implémentation, d’ajout, de suppression, etc., afin de se concentrer sur la communication avec un autre service.

## <a name="create-the-web-page-that-displays-to-do-items"></a>Créer la page web qui affiche les éléments de tâche

Une fois le service back-end implémenté, codez le site web qui affichera les éléments de tâche fournis par ce service. Les étapes suivantes s’effectuent au sein du projet **WebFrontEnd**.

La page web qui affiche les éléments de tâche a besoin d’accéder à la liste et à la classe **ToDoItem**.
Dans **l’Explorateur de solutions**, ajoutez une référence au projet Modèle en effectuant un clic droit sur **WebFrontEnd** et en sélectionnant **Ajouter** > **Référence...** . La boîte de dialogue **Gestionnaire de références** s’affiche.

Dans le **Gestionnaire de références**, cochez la case **Modèle** et cliquez sur **OK**.

Dans **l’Explorateur de solutions**, ouvrez la page Index.cshtml en accédant à **WebFrontEnd** > **Pages** > **Index.cshtml** . Ouvrez la page **Index.cshtml**.

Remplacez le contenu de l’intégralité du fichier par le code HTML suivant qui définit une table simple pour afficher des éléments de tâche :

```HTML
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Dans l’**Explorateur de solutions**, cliquez sur l’icône de liste déroulante du fichier **Index.cshtml**, puis ouvrez **Index.cshtml.cs**.

En haut du fichier **Index.cshtml.cs**, ajoutez `using System.Net.Http;`.

Remplacez le contenu du champ `public class IndexModel` avec :

```csharp
public class IndexModel : PageModel
{
    public Model.ToDoItem[] Items = new Model.ToDoItem[] { };

    public void OnGet()
    {
        HttpClient client = new HttpClient();

        using (HttpResponseMessage response = client.GetAsync(backendUrl).GetAwaiter().GetResult())
        {
            if (response.StatusCode == System.Net.HttpStatusCode.OK)
            {
                Items = Newtonsoft.Json.JsonConvert.DeserializeObject<Model.ToDoItem[]>(response.Content.ReadAsStringAsync().Result);
            }
        }
    }

    private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
    private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
}
```

### <a name="create-environment-variables"></a>Créer des variables d’environnement

L’URL pour le service back-end est nécessaire pour communiquer avec ce service. Pour les besoins de ce didacticiel, l’exemple de code suivant (qui est défini ci-dessus dans le cadre du modèle IndexModel) lit les variables d’environnement pour composer l’URL :

```csharp
private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
```

L’URL se compose du nom du service et du port. Toutes ces informations se trouvent dans le fichier service.yaml disponible dans le projet **ToDoService**.

> [!IMPORTANT]
> Dans les étapes suivantes, les fichiers YAML seront modifiés.
> Des caractères d’espacement, et non des tabulations, doivent être utilisés pour mettre en retrait les variables dans le fichier service.yaml. Dans le cas contraire, les variables ne seront pas compilées. Visual Studio peut insérer des tabulations lorsque vous créez les variables d’environnement. Remplacez toutes les tabulations par des espaces. Si vous voyez des erreurs dans la sortie de débogage **build**, vous pouvez quand même lancer l’application, mais pas avant d’avoir converti les tabulations en espaces et regénéré l’application. Pour vérifier que le fichier service.yaml ne contient aucune tabulation, vous pouvez afficher les espaces blancs dans l’éditeur Visual Studio sous **Modifier**  > **Avancé**  > **Afficher les espaces blancs**.
> Notez que les fichiers service.yaml sont traités à l’aide des paramètres régionaux anglais. Par exemple, si vous devez utiliser un séparateur décimal, utilisez un point au lieu d’une virgule.

Dans **l’Explorateur de solutions**, accédez au projet **ToDoService** et sélectionnez **Ressources du service** > **service.yaml**.

![Figure 1 : fichier service.yaml du projet ToDoService](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-port.png)

 Le nom du service (`ToDoService`) se trouve sous `services:`. Voir (1) dans la figure ci-dessus.

* Le port (`80`) se trouve sous `endpoints:`. Voir (2) dans la figure ci-dessus. Le numéro de port de votre projet sera probablement différent.

Ensuite, nous devons définir les variables d’environnement qui représentent le nom du service et le numéro de port dans le projet WebFrontEnd, pour lui permettre d’appeler le service back-end.

Dans **l’Explorateur de solutions**, accédez à **WebFrontEnd** > **Ressources du service** > **service.yaml** pour définir les variables qui spécifient l’adresse du service back-end.

Dans le fichier service.yaml, ajoutez les variables suivantes sous `environmentVariables:` (vous devez d’abord supprimer `#` pour décommenter `environmentVariables:`). L’espacement est important, donc alignez les variables que vous ajoutez sur les autres variables sous `environmentVariables:`. Il est très important que la valeur de ApiHostPort corresponde à la valeur du port de ToDoServiceListener, qui a été vue précédemment dans le fichier service.yaml de ToDoService.

```yaml
- name: ApiHostPort
  value: 
- name: ToDoServiceName
  value: ToDoService
```

> [!Tip]
> Il existe deux façons de spécifier la valeur de `ToDoServiceName` : 
> - Uniquement le nom du service, qui est résolu à la fois dans un scénario de débogage sur Windows 10, et lors du déploiement du service dans Azure Service Fabric Mesh
> - En tant que nom complet (servicename.appname). Cela fonctionne uniquement lors d’un débogage sur Windows 10.
> Il est conseillé d’utiliser le nom du service uniquement pour la résolution du service.

Le fichier **service.yaml** de votre projet **WebFrontEnd** doit ressembler à ce qui suit, même si votre valeur `ApiHostPort` sera probablement différente :

![Fichier service.yaml dans le projet WebFrontEnd](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-envvars.png)


Vous êtes maintenant prêt à créer et déployer l’image de l’application de maillage Service Fabric, avec le service web back-end, dans votre cluster local.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créez une application Service Fabric Mesh composée d’un service web frontend ASP.NET.
> * Créer un modèle pour représenter des éléments de tâche.
> * Créer un service back-end et récupérer des données à partir de celui-ci.
> * Ajouter un contrôleur et une instance DataContext dans le cadre de ce modèle Model View Controller pour le service back-end.
> * Créer une page web pour afficher les éléments de tâche.
> * Créer des variables d’environnement qui identifient le service back-end.

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Déboguer une application Service Fabric Mesh s’exécutant dans votre cluster de développement local](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)