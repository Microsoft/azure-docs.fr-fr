---
title: Tutoriel - Mettre à niveau une application Azure Service Fabric Mesh
description: Ce tutoriel est la quatrième partie d’une série et vous montre comment mettre à niveau une application Azure Service Fabric Mesh directement depuis Visual Studio.
author: georgewallace
ms.topic: conceptual
ms.date: 11/29/2018
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-csharp
ms.openlocfilehash: 2d74841e1873e5bd84d839122e0e662ef65ccd11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842000"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>Tutoriel : Découvrez comment mettre à niveau une application Service Fabric à l'aide de Visual Studio

Ce tutoriel est la quatrième partie d’une série et vous montre comment mettre à niveau une application Azure Service Fabric Mesh directement depuis Visual Studio. La mise à niveau inclut une mise à jour du code et une mise à jour de la configuration. Vous verrez que les étapes de la mise à niveau et de la publication dans Visual Studio sont les mêmes.

Ce didacticiel vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Mettre à niveau un service Service Fabric Mesh à l’aide de Visual Studio

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Créer une application Service Fabric Mesh dans Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Déboguer une application Service Fabric Mesh qui s’exécute dans votre cluster de développement local](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Déployer une application Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * Mettre à niveau une application Service Fabric Mesh
> * [Nettoyer des ressources Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas déployé l’application To-Do, suivez les instructions fournies dans [Publier une application web Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md).

## <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>Mettre à niveau un service Service Fabric Mesh à l’aide de Visual Studio

Cet article explique comment mettre à niveau un microservice à l’intérieur d’une application. Dans cet exemple, nous allons modifier le service `WebFrontEnd` pour afficher une catégorie de tâche et augmenter la quantité d’UC qui lui est attribuée. Ensuite, nous allons mettre à niveau le service déployé.

## <a name="modify-the-config"></a>Modifier la configuration

Lorsque vous créez une application Service Fabric Mesh, Visual studio ajoute un fichier **parameters.yaml** pour chaque environnement de déploiement (cloud et local). Dans ces fichiers, vous pouvez définir des paramètres et leurs valeurs, qui peuvent ensuite être référencés à partir de vos fichiers *.yaml Mesh, tels que service.yaml ou network.yaml.  Visual Studio fournit certaines variables, telles que la quantité d’UC utilisable par le service.

Nous allons mettre à jour le paramètre `WebFrontEnd_cpu` pour attribuer la valeur de `1.5` aux ressources de processeur, en prévision d’une plus forte utilisation du service **WebFrontEnd**.

1. Dans le projet **todolistapp**, sous **Environnements** > **Cloud**, ouvrez le fichier **parameters.yaml**. Remplacez la valeur de `WebFrontEnd_cpu` par `1.5`. En tant que bonne pratique, le nom du paramètre commence par le nom du service `WebFrontEnd_`, afin de le différencier des paramètres du même nom qui s’appliquent à d’autres services.

    ```xml
    WebFrontEnd_cpu: 1.5
    ```

2. Ouvrez le fichier **service.yaml** du projet **WebFrontEnd** sous **WebFrontEnd** > **Ressources du service**.

    Remarquez que dans la section `resources:`, le paramètre `cpu:` est défini sur `"[parameters('WebFrontEnd_cpu')]"`. Si le projet est créé pour le cloud, la valeur de `'WebFrontEnd_cpu` est prise dans le fichier **Environnements** > **Cloud** > **parameters.yaml** et est `1.5`. Si le projet est créé pour s’exécuter localement, la valeur est prise dans le fichier **Environnements** > **Local** > **parameters.yaml** et est « 0,5 ».

> [!Tip]
> Par défaut, le fichier de paramètres, qui est un homologue du fichier profile.yaml, est utilisé pour fournir les valeurs de ce fichier profile.yaml.
> Par exemple, Environnements > Cloud > parameters.yaml fournit les valeurs des paramètres pour Environnements > Cloud > profile.yaml.
>
> Vous pouvez remplacer cela en ajoutant le code suivant au fichier profile.yaml :`parametersFilePath=”relative or full path to the parameters file”` Par exemple, `parametersFilePath=”C:\MeshParms\CustomParameters.yaml”` ou `parametersFilePath=”..\CommonParameters.yaml”`.

## <a name="modify-the-model"></a>Modifier le modèle

Pour introduire une modification du code, ajoutez une propriété `Category` à la classe `ToDoItem` dans le fichier `ToDoItem.cs`.

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

Ensuite, mettez à jour la méthode `Load()`, dans le même fichier, pour définir la catégorie sur une chaîne par défaut :

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>Modifier le service

Le projet `WebFrontEnd` est une application ASP.NET Core avec une page web affichant des élément de liste de tâches. Dans le projet `WebFrontEnd`, ouvrez `Index.cshtml` et ajoutez les deux lignes suivantes, indiquées ci-dessous, pour afficher la catégorie de la tâche :

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Générez et exécutez l’application pour vérifier que vous voyez une nouvelle colonne de catégorie dans la page web qui répertorie les tâches.

## <a name="upgrade-the-app-from-visual-studio"></a>Mettre à niveau l’application à partir de Visual Studio

Que vous effectuiez une mise à niveau du code ou une mise à niveau de la configuration (dans ce cas nous effectuons les deux), mettez à niveau votre application Service Fabric Mesh sur Azure en cliquant avec le bouton droit sur **todolistapp** dans Visual Studio, puis en sélectionnant **Publier...**

Ensuite, une boîte de dialogue **Publier une application Service Fabric** s’affiche.

Utilisez la liste déroulante **Profil cible** pour sélectionner le fichier profile.yaml à utiliser pour ce déploiement. Comme nous mettons à niveau l’application dans le cloud, nous sélectionnons le fichier **cloud.yaml** dans la liste déroulante ; celui-ci utilisera la valeur « 1,0 » de `WebFrontEnd_cpu`, qui est définie dans ce fichier.

![Boîte de dialogue Publier Service Fabric mesh dans Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Sélectionnez votre compte et votre abonnement Azure. Définissez **Emplacement** sur le lieu que vous avez utilisé quand vous avez initialement publié l’application de tâches dans Azure. Dans cet article, nous utilisons la région **USA Est**.

Définissez **Groupe de ressources** sur le groupe de ressources que vous avez utilisé quand vous avez initialement publié l’application de tâches dans Azure.

Définissez **Azure Container Registry** sur le nom du registre de conteneurs Azure que vous avez créé quand vous avez initialement publié l’application de tâches dans Azure.

Dans la boîte de dialogue de publication, appuyez sur le bouton **Publier** pour mettre à niveau l’application de tâches sur Azure.

Supervisez la progression de la mise à niveau en sélectionnant le volet **Service Fabric Tools** dans la fenêtre de **Sortie** de Visual Studio. 

Une fois l’image générée et envoyée au registre de conteneurs Azure, un lien **Pour l’état** s’affiche dans la sortie, sur lequel vous pouvez cliquer pour superviser le déploiement dans le portail Azure.

Une fois la mise à niveau terminée, la sortie de **Service Fabric Tools** affiche l’adresse IP et le port de votre application sous la forme d’une URL.

```json
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Ouvrez un navigateur web et accédez à l’URL pour voir le site web en cours d’exécution dans Azure. Vous devez maintenant voir une page web qui contient une colonne de catégorie.

## <a name="next-steps"></a>Étapes suivantes

Cette partie du didacticiel vous a permis de répondre aux questions suivantes :
> [!div class="checklist"]
> * Comment mettre à niveau une application Service Fabric Mesh à l’aide de Visual Studio ?

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Nettoyer des ressources Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)