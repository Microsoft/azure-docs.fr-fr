---
title: 'Démarrage rapide : Votre première requête Go'
description: Dans ce démarrage rapide, vous suivez les étapes pour activer le package Resource Graph pour Go et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: e09bbb63b56fd30eb3aa9e7e527acdd8691d3ee6
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655796"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-go"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de Go

La première étape pour utiliser Azure Resource Graph consiste à vérifier que les packages nécessaires pour Go sont installés. Ce démarrage rapide décrit le processus d’ajout des packages à votre installation Go.

Au terme de ce processus, vous aurez ajouté les packages à votre installation Go et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="add-the-resource-graph-package"></a>Ajouter le package Resource Graph

Pour permettre à Go d’interroger Azure Resource Graph, le package doit être ajouté. Ce package fonctionne partout où Go peut être utilisé, notamment avec [bash sur Windows 10](/windows/wsl/install-win10) ou par le biais d’une installation locale.

1. Vérifiez que la version la plus récente de Go est installée (**1.14** au minimum). Si ce n’est pas le cas, téléchargez-la à l’adresse [Golang.org](https://golang.org/dl/).

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.5.1** au minimum). Si ce n’est pas le cas, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI est nécessaire pour permettre à Go d’utiliser la méthode `auth.NewAuthorizerFromCLI()` dans l’exemple suivant. Pour plus d’informations sur les autres options, consultez [Azure SDK pour Go - Autres détails d’authentification](https://github.com/Azure/azure-sdk-for-go#more-authentication-details).

1. S’authentifier par le biais d’Azure CLI.

   ```azurecli
   az login
   ```

1. Dans votre environnement Go, installez les packages nécessaires pour Azure Resource Graph :

   ```bash
   # Add the Resource Graph package for Go
   go get -u github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph

   # Add the Azure auth package for Go
   go get -u github.com/Azure/go-autorest/autorest/azure/auth
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Une fois les packages Go ajoutés à l’environnement de votre choix, vous pouvez exécuter une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

1. Créez l’application Go et enregistrez la source suivante sous `argQuery.go` :

   ```Go
   package main
   
   import (
      "fmt"
      "os"
      "context"
      "strconv"
      arg "github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph"
      "github.com/Azure/go-autorest/autorest/azure/auth"
   )
   
   func main() {
       // Get variables from command line arguments
       var query = os.Args[1]
       var subList = os.Args[2:]
   
       // Create and authorize a ResourceGraph client
       argClient := arg.New()
       authorizer, err := auth.NewAuthorizerFromCLI()
       if err == nil {
           argClient.Authorizer = authorizer
       } else {
           fmt.Printf(err.Error())
       }
     
       // Set options
       RequestOptions := arg.QueryRequestOptions {
           ResultFormat: "objectArray",
       }
     
       // Create the query request
       Request := arg.QueryRequest {
           Subscriptions: &subList,
           Query: &query,
           Options: &RequestOptions,
       }
     
       // Run the query and get the results
       var results, queryErr = argClient.Resources(context.Background(), Request)
       if queryErr == nil {
           fmt.Printf("Resources found: " + strconv.FormatInt(*results.TotalRecords, 10) + "\n")
           fmt.Printf("Results: " + fmt.Sprint(results.Data) + "\n")
       } else {
           fmt.Printf(queryErr.Error())
       }
   }
   ```

1. Générez l’application Go :

   ```bash
   go build argQuery.go
   ```

1. Exécutez votre première requête Azure Resource Graph à l’aide de l’application Go compilée. Remplacez `<SubID>` par votre ID d’abonnement :

   ```bash
   argQuery "Resources | project name, type | limit 5" "<SubID>"
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Remplacez le premier paramètre par `argQuery` et la requête par `order by` suivi de la propriété **Name**. Remplacez `<SubID>` par votre ID d’abonnement :

   ```bash
   argQuery "Resources | project name, type | limit 5 | order by name asc" "<SubID>"
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Remplacez le premier paramètre par `argQuery` et changez la requête en `order by` pour classer d’abord les résultats selon la propriété **Name**, puis limitez (`limit`) les résultats aux cinq premiers. Remplacez `<SubID>` par votre ID d’abonnement :

   ```bash
   argQuery "Resources | project name, type | order by name asc | limit 5" "<SubID>"
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer les packages installés de votre environnement Python, vous pouvez utiliser la commande suivante :

```bash
# Remove the installed packages from the Go environment
go clean -i github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph
go clean -i github.com/Azure/go-autorest/autorest/azure/auth
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez ajouté les packages Resource Graph à votre environnement Go et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)