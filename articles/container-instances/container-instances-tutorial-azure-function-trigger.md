---
title: Tutoriel - Déclencher un groupe de conteneurs par fonction Azure
description: Créer une fonction PowerShell serverless déclenchée par HTTP pour automatiser la création d’instances de conteneur Azure
ms.topic: tutorial
ms.date: 06/10/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 7b94d583ca26b88d093810528d3193f20d765f1d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349246"
---
# <a name="tutorial-use-an-http-triggered-azure-function-to-create-a-container-group"></a>Tutoriel : Utiliser une fonction Azure déclenchée par HTTP pour créer un groupe de conteneurs

[Azure Functions](../azure-functions/functions-overview.md) est un service de calcul serverless pouvant exécuter des scripts ou du code en réponse à divers événements, notamment une requête HTTP, un minuteur ou un message dans une file d’attente Stockage Azure.

Dans ce tutoriel, vous allez créer une fonction Azure qui prend une requête HTTP et déclenche le déploiement d’un [groupe de conteneurs](container-instances-container-groups.md). Cet exemple présente les bases de l’utilisation d’Azure Functions pour créer automatiquement des ressources dans Azure Container Instances. Vous pouvez modifier ou étendre cet exemple pour des scénarios plus complexes ou d’autres déclencheurs d’événements. 

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Utiliser Visual Studio Code avec l’[extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour créer une fonction PowerShell de base déclenchée par HTTP.
> * Activer une identité dans l’application de fonction et lui accorder les autorisations nécessaires pour créer des ressources Azure.
> * Modifier et republier la fonction PowerShell pour automatiser le déploiement d’un groupe de conteneurs monoconteneur.
> * Vérifier le déploiement déclenché par HTTP du conteneur.

## <a name="prerequisites"></a>Prérequis

Consultez [Créer votre première fonction dans Azure à l’aide de Visual Studio Code](../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-powershell#configure-your-environment) pour obtenir les prérequis à l’installation et à l’utilisation de Visual Studio Code avec l’extension Azure Functions sur votre système d’exploitation.

Les étapes supplémentaires de cet article utilisent Azure PowerShell. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure PowerShell][azure-powershell-install] et [Se connecter à Azure](/powershell/azure/get-started-azureps#sign-in-to-azure).

## <a name="create-a-basic-powershell-function"></a>Créer une fonction PowerShell de base

Suivez les étapes de la section [Créer votre première fonction PowerShell dans Azure](../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-powershell) pour créer une fonction PowerShell à l’aide du modèle Déclencheur HTTP. Utilisez le nom de fonction Azure par défaut **HttpTrigger**. Comme indiqué dans le guide de démarrage rapide, testez la fonction localement, puis publiez le projet sur une application de fonction dans Azure. Cet exemple est une fonction de base déclenchée par HTTP qui retourne une chaîne de texte. Dans les étapes ultérieures de cet article, vous modifierez la fonction pour créer un groupe de conteneurs.

Cet article part du principe que vous publiez le projet avec le nom *myfunctionapp* dans un groupe de ressources Azure nommé automatiquement en fonction du nom de l’application de fonction (également *myfunctionapp*). Dans les étapes ultérieures, utilisez à la place le nom de votre application de fonction et le nom de votre groupe de ressources.

## <a name="enable-an-azure-managed-identity-in-the-function-app"></a>Activer une identité managée par Azure dans l’application de fonction

Les commandes suivantes activent une [identité managée](../app-service/overview-managed-identity.md?toc=/azure/azure-functions/toc.json#add-a-system-assigned-identity) affectée par le système dans votre application de fonction. L’hôte PowerShell qui exécute l’application peut s’authentifier automatiquement auprès d’Azure à l’aide de cette identité, ce qui permet aux fonctions d’effectuer des actions sur les services Azure dont l’identité a obtenu une autorisation d’accès. Dans ce tutoriel, vous allez accorder à l’identité managée les autorisations nécessaires pour créer des ressources dans le groupe de ressources de l’application de fonction. 

[Ajoutez une identité](../app-service/overview-managed-identity.md?tabs=dotnet#using-azure-powershell-1) à l’application de fonction :

```powershell
Update-AzFunctionApp -Name myfunctionapp `
    -ResourceGroupName myfunctionapp `
    -IdentityType SystemAssigned
```

Attribuez à l’identité le rôle de contributeur étendu au groupe de ressources :

```powershell
$SP=(Get-AzADServicePrincipal -DisplayName myfunctionapp).Id
$RG=(Get-AzResourceGroup -Name myfunctionapp).ResourceId
New-AzRoleAssignment -ObjectId $SP -RoleDefinitionName "Contributor" -Scope $RG
```

## <a name="modify-httptrigger-function"></a>Modifier la fonction HttpTrigger

Modifiez le code PowerShell pour que la fonction **HttpTrigger** crée un groupe de conteneurs. Dans le fichier `run.ps1` de la fonction, recherchez le bloc de code suivant. Ce code affiche une valeur de nom si elle est passée en tant que chaîne de requête dans l’URL de la fonction :

```powershell
[...]
if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}
[...]
```

Remplacez ce code par le bloc d’exemple suivant. Ici, si une valeur de nom est passée dans la chaîne de requête, elle est utilisée pour nommer et créer un groupe de conteneurs à l’aide de l’applet de commande [New-AzContainerGroup][new-azcontainergroup]. Veillez à remplacer le nom du groupe de ressources *myfunctionapp* par le nom du groupe de ressources de votre application de fonction :

```powershell
[...]
if ($name) {
    New-AzContainerGroup -ResourceGroupName myfunctionapp -Name $name `
        -Image alpine -OsType Linux `
        -Command "echo 'Hello from an Azure container instance triggered by an Azure function'" `
        -RestartPolicy Never
    if ($?) {
        $body = "This HTTP triggered function executed successfully. Started container group $name"
    }
    else  {
        $body = "There was a problem starting the container group."
    }
[...]
```

Cet exemple crée un groupe de conteneurs constitué d’une instance monoconteneur exécutant l’image `alpine`. Le conteneur exécute une seule commande `echo`, puis s’arrête. Dans un exemple concret, vous pouvez déclencher la création d’un ou plusieurs groupes de conteneurs pour exécuter un traitement par lots.
 
## <a name="test-function-app-locally"></a>Tester l’application de fonction localement

Vérifiez que la fonction s’exécute localement avant de republier le projet d’application de fonction sur Azure. Quand elle est exécutée localement, la fonction ne crée pas de ressources Azure. Toutefois, vous pouvez tester le flux de la fonction en passant et en ne passant pas de valeur de nom dans une chaîne de requête. Pour déboguer la fonction, consultez [Déboguer localement des fonctions Azure PowerShell](../azure-functions/functions-debug-powershell-local.md).

## <a name="republish-azure-function-app"></a>Republier l’application de fonction Azure

Après avoir vérifié que la fonction s’exécute localement, republiez le projet sur l’application de fonction existante dans Azure.

1. Dans Visual Studio Code, ouvrez la palette de commandes. Recherchez et sélectionnez `Azure Functions: Deploy to Function App...`.
1. Sélectionnez le dossier de travail actif à compresser et à déployer.
1. Sélectionnez l’abonnement, puis le nom de l’application de fonction existante (*myfunctionapp*). Confirmez que vous souhaitez remplacer le déploiement précédent.

Une notification s’affiche après que votre application de fonction a été créée et que le package de déploiement a été appliqué. Sélectionnez **Afficher la sortie** dans cette notification pour afficher les résultats de la création et du déploiement, y compris les ressources Azure que vous avez mises à jour.

## <a name="run-the-function-in-azure"></a>Exécuter la fonction dans Azure

Une fois le déploiement terminé, obtenez l’URL de la fonction. Par exemple, utilisez la zone **Azure : Fonctions** dans Visual Studio Code pour copier l’URL de fonction **HttpTrigger** ou pour obtenir l’URL de fonction dans le [portail Azure](../azure-functions/functions-create-first-azure-function.md#test-the-function).

L’URL de la fonction est au format :

```
https://myfunctionapp.azurewebsites.net/api/HttpTrigger
```

### <a name="run-function-without-passing-a-name"></a>Exécuter la fonction sans passer de nom

En guise de premier test, exécutez la commande `curl` et passez l’URL de la fonction sans ajouter de chaîne de requête `name`. 

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger"
```

La fonction retourne le code d’état 200 et le texte `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response` :

```
[...]
> GET /api/HttpTrigger? HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/1.1 200 OK
< Content-Length: 135
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:50:27 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.* Closing connection 0
```

### <a name="run-function-and-pass-the-name-of-a-container-group"></a>Exécuter la fonction et passer le nom d’un groupe de conteneurs

Exécutez à présent la commande `curl` et ajoutez le nom d’un groupe de conteneurs (*mycontainergroup*) sous la forme d’une chaîne de requête `?name=mycontainergroup` :

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger?name=mycontainergroup"
```

La fonction retourne le code d’état 200 et déclenche la création du groupe de conteneurs :

```
[...]
> GET /api/HttpTrigger1?name=mycontainergroup HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 92
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:54:31 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Started container group mycontainergroup* Closing connection 0
```

Vérifiez que le conteneur a été exécuté avec la commande [Get-AzContainerInstanceLog][get-azcontainerinstancelog] :

```azurecli
Get-AzContainerInstanceLog -ResourceGroupName myfunctionapp `
  -ContainerGroupName mycontainergroup 
```

Exemple de sortie :

```console
Hello from an Azure container instance triggered by an Azure function
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources que vous avez créées dans ce tutoriel, vous pouvez exécuter la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Cette commande supprime l’application de fonction que vous avez créée, ainsi que le conteneur en cours d’exécution et toutes les ressources associées.

```azurecli-interactive
az group delete --name myfunctionapp
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une fonction Azure qui prend une requête HTTP et déclenche le déploiement d’un groupe de conteneurs. Vous avez appris à :

> [!div class="checklist"]
> * Utiliser Visual Studio Code avec l’extension Azure Functions pour créer une fonction PowerShell de base déclenchée par HTTP.
> * Activer une identité dans l’application de fonction et lui accorder les autorisations nécessaires pour créer des ressources Azure.
> * Modifier le code de la fonction PowerShell pour automatiser le déploiement d’un groupe de conteneurs monoconteneur.
> * Vérifier le déploiement déclenché par HTTP du conteneur.

Pour obtenir un exemple détaillé de lancement et de supervision d’un travail conteneurisé, consultez le billet de blog [Event-Driven Serverless Containers with PowerShell Azure Functions and Azure Container Instances](https://dev.to/azure/event-driven-serverless-containers-with-powershell-azure-functions-and-azure-container-instances-e9b) et l’[exemple de code](https://github.com/anthonychu/functions-powershell-run-aci) associé.

Consultez la [documentation sur Azure Functions](../azure-functions/index.yml) pour obtenir des instructions détaillées sur la création de fonctions Azure et la publication d’un projet de fonctions. 

<!-- IMAGES -->


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-powershell-install]: /powershell/azure/install-az-ps
[new-azcontainergroup]: /powershell/module/az.containerinstance/new-azcontainergroup
[get-azcontainerinstancelog]: /powershell/module/az.containerinstance/get-azcontainerinstancelog