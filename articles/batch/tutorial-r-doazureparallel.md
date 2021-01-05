---
title: Tutoriel - Exécuter une simulation de R parallèle avec Azure Batch
description: Didacticiel - Instructions détaillées pour exécuter une simulation financière de Monte-Carlo dans Azure Batch à l’aide du package doAzureParallel R
ms.devlang: r
ms.topic: tutorial
ms.date: 10/08/2020
ms.custom: mvc
ms.openlocfilehash: f157a3596992fc9d40179653cc1f9a070c6ef803
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106359"
---
# <a name="tutorial-run-a-parallel-r-simulation-with-azure-batch"></a>Tutoriel : Exécuter une simulation de R parallèle avec Azure Batch

Exécutez vos charges de travail R parallèles à l’échelle à l’aide de [doAzureParallel](https://www.github.com/Azure/doAzureParallel), un package léger R qui vous permet d’utiliser Azure Batch directement à partir de votre session R. Le package doAzureParallel s’appuie sur le package R [foreach](https://cran.r-project.org/web/packages/foreach/index.html) bien connu. doAzureParallel prend chaque itération de la boucle foreach et la soumet sous forme de tâche Azure Batch.

Ce didacticiel vous montre comment déployer un pool Batch et exécuter un travail parallèle R dans Azure Batch directement dans RStudio. Vous allez apprendre à effectuer les actions suivantes :


> [!div class="checklist"]
> * Installer doAzureParallel et le configurer pour accéder à vos comptes de stockage et Batch
> * Créer un pool Batch comme serveur principal parallèle pour votre session R
> * Exécuter un exemple de simulation parallèle sur le pool

## <a name="prerequisites"></a>Prérequis

* Une distribution de [R](https://www.r-project.org/) installée, telle que [Microsoft R Open](https://mran.microsoft.com/open). Utilisez la version 3.3.1 ou ultérieure.

* [RStudio](https://www.rstudio.com/), l’édition commerciale ou [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop) open source.

* Un compte Azure Batch et un compte de stockage Azure. Pour créer ces comptes, consultez les démarrages rapides Azure Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]
## <a name="install-doazureparallel"></a>Installer doAzureParallel

Dans la console RStudio, installez le [package doAzureParallel GitHub](https://www.github.com/Azure/doAzureParallel). Les commandes suivantes téléchargent et installent le package et ses dépendances dans votre session R actuelle :

```R
# Install the devtools package
install.packages("devtools")

# Install rAzureBatch package
devtools::install_github("Azure/rAzureBatch")

# Install the doAzureParallel package
devtools::install_github("Azure/doAzureParallel")

# Load the doAzureParallel library
library(doAzureParallel)
```
L’installation peut prendre plusieurs minutes.

Pour configurer doAzureParallel avec les informations d’identification de compte obtenues précédemment, générez un fichier de configuration appelé *credentials.json* dans votre répertoire de travail :

```R
generateCredentialsConfig("credentials.json")
```

Alimentez ce fichier avec les clés et noms du compte de stockage et du compte Batch. Laissez le paramètre `githubAuthenticationToken` inchangé.

Lorsque vous avez terminé, le fichier d’informations d’identification ressemble à ce qui suit :

```json
{
  "batchAccount": {
    "name": "mybatchaccount",
    "key": "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==",
    "url": "https://mybatchaccount.mybatchregion.batch.azure.com"
  },
  "storageAccount": {
    "name": "mystorageaccount",
    "key": "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=="
  },
  "githubAuthenticationToken": ""
}
```

Enregistrez le fichier . Ensuite, exécutez la commande suivante pour définir les informations d’identification de votre session R actuelle :

```R
setCredentials("credentials.json")
```

## <a name="create-a-batch-pool"></a>Créer un pool Batch

doAzureParallel inclut une fonction pour générer un pool Azure Batch (cluster) pour exécuter des travaux R parallèles. Les nœuds exécutent une instance [Data Science Virtual Machine (DSVM) Azure](../machine-learning/data-science-virtual-machine/overview.md) basée sur Ubuntu. Les packages Microsoft R Open et R populaires sont pré-installés sur cette image. Vous pouvez afficher ou personnaliser certains paramètres de cluster, tels que le nombre et la taille des nœuds.

Pour générer un fichier JSON de configuration de cluster dans votre répertoire de travail :

```R
generateClusterConfig("cluster.json")
```

Ouvrez le fichier pour afficher la configuration par défaut, qui inclut 3 nœuds dédiés et 3 nœuds [de faible priorité](batch-low-pri-vms.md). Ces paramètres sont fournis à titre d’exemple. Vous pouvez les utiliser pour faire des essais ou les modifier. Les nœuds dédiés sont réservés à votre pool. Les nœuds de faible priorité sont proposés à prix réduit à partir de la capacité de machine virtuelle excédentaire dans Azure. Les nœuds de faible priorité deviennent indisponibles si la capacité d’Azure est insuffisante.

Pour ce didacticiel, modifiez la configuration comme suit :

* Définissez `taskSlotsPerNode` sur *2*, pour tirer parti des deux cœurs sur chaque nœud
* Définissez `dedicatedNodes` sur *0*, afin de pouvoir essayer les machines virtuelles de faible priorité disponibles pour Batch. Définissez `min` de `lowPriorityNodes` sur *5*. Et `max` sur *10*, ou choisissez des valeurs inférieures si vous le souhaitez.

Conservez les valeurs par défaut pour les autres paramètres, puis enregistrez le fichier. Celui-ci doit se présenter comme suit :

```json
{
  "name": "myPoolName",
  "vmSize": "Standard_D2_v2",
  "taskSlotsPerNode": 2,
  "poolSize": {
    "dedicatedNodes": {
      "min": 0,
      "max": 0
    },
    "lowPriorityNodes": {
      "min": 5,
      "max": 10
    },
    "autoscaleFormula": "QUEUE"
  },
  "containerImage": "rocker/tidyverse:latest",
  "rPackages": {
    "cran": [],
    "github": [],
    "bioconductor": []
  },
  "commandLine": []
}
```

Créez maintenant le cluster. Azure Batch crée le pool immédiatement, mais prend quelques minutes pour allouer et démarrer les nœuds de calcul. Une fois le cluster disponible, enregistrez-le en tant que serveur principal parallèle de votre session R.

```R
# Create your cluster if it does not exist; this takes a few minutes
cluster <- makeCluster("cluster.json")

# Register your parallel backend
registerDoAzureParallel(cluster)

# Check that the nodes are running
getDoParWorkers()
```

La sortie affiche le nombre de « Workers d’exécution » pour doAzureParallel. Ce nombre correspond au nombre de nœuds multiplié par la valeur de `taskSlotsPerNode`. Si vous avez modifié la configuration du cluster comme décrit précédemment, le nombre correspond à *10*.

## <a name="run-a-parallel-simulation"></a>Exécuter une simulation parallèle

Maintenant que votre cluster est créé, vous êtes prêt à exécuter la boucle foreach avec le serveur principal parallèle inscrit (pool Azure Batch). Par exemple, exécutez une simulation financière Monte-Carlo, tout d’abord localement en utilisant une boucle foreach standard, puis en exécutant foreach avec Batch. Cet exemple est une version simplifiée de la prédiction du prix de l’action par simulation d’un grand nombre de résultats différents après 5 ans.

Supposons que le stock de Contoso Corporation gagne en moyenne 1,001 fois son prix d’ouverture chaque jour, mais que sa volatilité (écart type) est de 0,01. Avec un prix de départ de 100 $, utilisez une simulation de prix Monte Carlo pour déterminer le prix de l’action après 5 ans.

Paramètres de la simulation Monte-Carlo :

```R
mean_change = 1.001
volatility = 0.01
opening_price = 100
```

Pour simuler les cours de clôture, définissez la fonction suivante :

```R
getClosingPrice <- function() {
  days <- 1825 # ~ 5 years
  movement <- rnorm(days, mean=mean_change, sd=volatility)
  path <- cumprod(c(opening_price, movement))
  closingPrice <- path[days]
  return(closingPrice)
}
```

Exécutez d’abord 10 000 simulations localement à l’aide d’une boucle foreach standard avec le (mot clé) `%do%` :

```R
start_s <- Sys.time()
# Run 10,000 simulations in series
closingPrices_s <- foreach(i = 1:10, .combine='c') %do% {
  replicate(1000, getClosingPrice())
}
end_s <- Sys.time()
```


Tracez les prix de clôture dans un histogramme pour afficher la distribution des résultats :

```R
hist(closingPrices_s)
```

Le résultat ressemble à ce qui suit :

![Capture d’écran montrant un histogramme des prix de clôture](media/tutorial-r-doazureparallel/closing-prices-local.png)
  
Une simulation locale est effectuée en quelques secondes ou moins :

```R
difftime(end_s, start_s)
```

La durée d’exécution estimée pour 10 millions de résultats localement, à l’aide d’une approximation linéaire, est d’environ 30 minutes :

```R
1000 * difftime(end_s, start_s, unit = "min")
```


Maintenant, exécutez le code avec `foreach` à l’aide du mot-clé `%dopar%` pour comparer le temps nécessaire à l’exécution de 10 millions de simulation dans Azure. Pour mettre en parallèle la simulation avec Batch, exécutez 100 itérations de 100 000 simulations :

```R
# Optimize runtime. Chunking allows running multiple iterations on a single R instance.
opt <- list(chunkSize = 10)
start_p <- Sys.time()
closingPrices_p <- foreach(i = 1:100, .combine='c', .options.azure = opt) %dopar% {
  replicate(100000, getClosingPrice())
}
end_p <- Sys.time()
```

La simulation distribue les tâches aux nœuds dans le pool Batch. Vous pouvez voir l’activité dans la carte thermique pour le pool dans le portail Azure. Accédez aux **comptes Batch** > *myBatchAccount*. Cliquez sur **Pools** > *myPoolName*.

![Carte thermique du pool en cours d’exécution des tâches parallèles R](media/tutorial-r-doazureparallel/pool.png)

La simulation prend quelques minutes. Le package fusionne automatiquement les résultats et les extrait vers le bas à partir des nœuds. Ensuite, vous êtes prêt à utiliser les résultats dans votre session R.

```R
hist(closingPrices_p)
```

Le résultat ressemble à ce qui suit :

![Distribution des prix de clôture](media/tutorial-r-doazureparallel/closing-prices.png)

Combien de temps a duré la simulation parallèle ?

```R
difftime(end_p, start_p, unit = "min")
```

Vous vous apercevrez que le fait d’exécuter la simulation sur le pool Batch vous permet d’augmenter de manière significative les performances sur la durée estimée pour exécuter la simulation localement.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Le travail est automatiquement supprimé une fois terminé. Si vous avez besoin du cluster plus longtemps, appelez la fonction `stopCluster` dans le package doAzureParallel pour le supprimer :

```R
stopCluster(cluster)
```

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> Installer doAzureParallel et le configurer pour accéder à vos comptes de stockage et Batch
> * Créer un pool Batch comme serveur principal parallèle pour votre session R
> * Exécuter un exemple de simulation parallèle sur le pool


Pour plus d’informations sur doAzureParallel, consultez la documentation et les exemples sur GitHub.

> [!div class="nextstepaction"]
> [Package doAzureParallel](https://github.com/Azure/doAzureParallel/)
