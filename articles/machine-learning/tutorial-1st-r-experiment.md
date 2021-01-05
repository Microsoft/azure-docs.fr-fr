---
title: 'Tutoriel : Utiliser R pour créer un modèle Machine Learning (préversion)'
titleSuffix: Azure Machine Learning
description: Dans ce tutoriel, vous allez utiliser le SDK R Azure Machine Learning pour créer un modèle de régression logistique qui prédit la probabilité de décès au cours d’un accident de voiture.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.reviewer: sgilley
author: revodavid
ms.author: davidsmi
ms.date: 02/07/2020
ms.openlocfilehash: 0e94288b49cd57b59c126c95ca507477f1c56946
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321525"
---
# <a name="tutorial-use-r-to-create-a-machine-learning-model-preview"></a>Tutoriel : Utiliser R pour créer un modèle Machine Learning (préversion)


> [!IMPORTANT]
> Le SDK R Azure Machine Learning est actuellement en préversion publique.
> La préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail en production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Dans ce tutoriel, vous allez utiliser le kit SDK R Azure Machine Learning (préversion) pour créer un modèle de régression logistique qui prédit la probabilité de décès au cours d’un accident de voiture. Vous verrez comment les ressources cloud Azure Machine Learning fonctionnent avec R afin de fournir un environnement scalable pour entraîner et déployer un modèle.  

Dans ce didacticiel, vous allez effectuer les tâches suivantes :
> [!div class="checklist"]
> * Création d’un espace de travail Microsoft Azure Machine Learning
> * Ouvrir RStudio à partir de votre espace de travail
> * Cloner https://github.com/Azure/azureml-sdk-for-r avec les fichiers nécessaires pour exécuter ce tutoriel dans votre espace de travail
> * Charger des données et préparer l’apprentissage
> * Charger des données dans un magasin de données afin qu’elles soient disponibles pour l’entraînement à distance
> * Créer une ressource de calcul pour entraîner le modèle
> * Entraîner un modèle `caret` pour prédire la probabilité de décès
> * Déployer un point de terminaison de prédiction
> * Tester le modèle à partir de R

Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.


## <a name="create-a-workspace"></a>Créer un espace de travail

Un espace de travail Azure Machine Learning est une ressource fondamentale du cloud que vous utilisez pour expérimenter, entraîner et déployer des modèles Machine Learning. Il lie votre abonnement Azure et votre groupe de ressources à un objet facile à consommer dans le service. 

Il existe de nombreuses [façons de créer un espace de travail](how-to-manage-workspace.md). Dans ce tutoriel, vous allez créer un espace de travail par le biais du portail Azure, une console web pour la gestion de vos ressources Azure. 

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

>[!IMPORTANT] 
> Prenez note de votre **espace de travail** et de votre **abonnement**. Vous en aurez besoin pour être sûr de créer votre expérience au bon endroit. 


## <a name="open-rstudio"></a><a name="open"></a>Ouvrir RStudio

Cet exemple utilise une instance de calcul dans votre espace de travail pour une expérience préconfigurée sans installation. Utilisez [votre propre environnement](https://azure.github.io/azureml-sdk-for-r/articles/installation.html) si vous préférez contrôler votre environnement, vos packages et vos dépendances sur votre propre machine.

Utilisez RStudio sur une instance de calcul ML Azure pour exécuter ce tutoriel.  

1. Sélectionnez **Compute** à gauche.

1. Ajoutez une ressource de calcul s’il n’en existe pas déjà.

1. Une fois le calcul en cours d’exécution, utilisez le lien **RStudio** pour ouvrir RStudio.


## <a name="clone-the-sample-vignettes"></a><a name="azure"></a>Cloner les exemples de vignettes 

Clonez le référentiel GitHub https://github.com/Azure/azureml-sdk-for-r pour obtenir une copie des fichiers de vignettes que vous allez exécuter dans ce tutoriel.

1. Dans RStudio, accédez à l’onglet « Terminal » et exécutez une commande cd dans le répertoire où vous souhaitez cloner le référentiel.

1. Exécutez `git clone https://github.com/Azure/azureml-sdk-for-r` dans le terminal pour cloner le dépôt.

1. Dans RStudio, accédez au dossier *vignettes* du dossier *azureml-sdk-for-r* cloné.  Sous *vignettes* , sélectionnez le fichier *train-and-deploy-first-model.Rmd* pour trouver la vignette utilisée dans ce tutoriel. Les fichiers supplémentaires utilisés pour la vignette se trouvent dans le sous-dossier *train-and-deploy-first-model*. Une fois que vous avez ouvert la vignette, définissez le répertoire de travail sur l’emplacement du fichier via **Session > Définir le répertoire de travail > Sur l’emplacement du fichier source**. 

> [!Important]
> Le reste de cet article contient le même contenu que ce que vous voyez dans le fichier *train-and-deploy-first-model.Rmd*. Si vous savez utiliser RMarkdown, n’hésitez pas à utiliser le code de ce fichier.  Vous pouvez aussi copier/coller les extraits de code à partir de ce fichier ou à partir de cet article dans un script R ou la ligne de commande. 


## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement
La configuration de votre travail de développement dans ce tutoriel inclut les actions suivantes :

* Installer les packages nécessaires
* Vous connecter à un espace de travail, afin que votre instance de calcul puisse communiquer avec des ressources distantes
* Créer une expérience pour effectuer le suivi de vos exécutions
* Créer une cible de calcul distante à utiliser pour l’apprentissage

### <a name="install-required-packages"></a>Installer les packages nécessaires
La version la plus récente du Kit de développement logiciel (SDK) R de CRAN est déjà installée sur l’instance de calcul. Si vous souhaitez installer la version de développement à partir de GitHub au lieu de récupérer les derniers correctifs de bogues, exécutez la commande suivante :
    
```R
remotes::install_github('https://github.com/Azure/azureml-sdk-for-r')
azuremlsdk::install_azureml()
```

> [!WARNING]
> Pendant le processus d’installation, si vous recevez l’invite « `Would you like to install Miniconda? [Y/n]:` », répondez avec « `n` », car Anaconda est déjà installé sur l’instance de calcul et une installation de Miniconda n’est pas nécessaire.

À présent, importez le package **azuremlsdk**.

```R
library(azuremlsdk)
```

Les scripts d’apprentissage et de scoring (`accidents.R` et `accident_predict.R`) ont des dépendances supplémentaires. Si vous prévoyez d’exécuter ces scripts localement, assurez-vous que vous disposez également de ces packages.

### <a name="load-your-workspace"></a>Charger votre espace de travail
Instanciez un objet espace de travail à partir de votre espace de travail existant. Le code suivant charge les détails sur l’espace de travail à partir du fichier **config.json**. Vous pouvez également récupérer un espace de travail à l’aide de [`get_workspace()`](https://azure.github.io/azureml-sdk-for-r/reference/get_workspace.html).

```R
ws <- load_workspace_from_config()
```

### <a name="create-an-experiment"></a>Créer une expérience
Une expérience Azure ML effectue le suivi d’un regroupement d’exécutions, en général à partir du même script d’apprentissage. Créez une expérience pour faire le suivi des exécutions afin d’effectuer l’apprentissage du modèle caret sur les données relatives aux accidents.

```R
experiment_name <- "accident-logreg"
exp <- experiment(ws, experiment_name)
```

### <a name="create-a-compute-target"></a>Créer une cible de calcul
En utilisant Capacité de calcul Azure Machine Learning (AmlCompute), un service managé, les scientifiques des données peuvent entraîner des modèles Machine Learning sur des clusters de machines virtuelles Azure. Il peut s’agir notamment de machines virtuelles prenant en charge les GPU. Dans ce tutoriel, vous allez créer un cluster AmlCompute à un seul nœud qui vous servira d’environnement d’entraînement. Le code ci-dessous crée automatiquement le cluster de calcul s’il n’existe pas encore dans votre espace de travail.

Vous devrez peut-être patienter quelques minutes pour que votre cluster de calcul soit provisionné s’il n’existe pas encore.

```R
cluster_name <- "rcluster"
compute_target <- get_compute(ws, cluster_name = cluster_name)
if (is.null(compute_target)) {
  vm_size <- "STANDARD_D2_V2" 
  compute_target <- create_aml_compute(workspace = ws,
                                       cluster_name = cluster_name,
                                       vm_size = vm_size,
                                       max_nodes = 1)
}

wait_for_provisioning_completion(compute_target)
```

## <a name="prepare-data-for-training"></a>Préparer les données pour l’apprentissage
Ce tutoriel utilise des données provenant de l’agence américaine [National Highway Traffic Safety Administration](https://cdan.nhtsa.gov/tsftables/tsfar.htm) (remerciements à [Mary C. Meyer et Tremika Finney](https://www.stat.colostate.edu/~meyer/airbags.htm)).
Ce jeu de données comprend des données provenant de plus de 25 000 accidents automobiles aux États-Unis, avec des variables que vous pouvez utiliser pour prédire la probabilité de décès. Tout d’abord, importez les données dans R et transformez-les en un nouveau dataframe `accidents` pour l’analyse, puis exportez-les vers un fichier `Rdata`.

```R
nassCDS <- read.csv("nassCDS.csv", 
                     colClasses=c("factor","numeric","factor",
                                  "factor","factor","numeric",
                                  "factor","numeric","numeric",
                                  "numeric","character","character",
                                  "numeric","numeric","character"))
accidents <- na.omit(nassCDS[,c("dead","dvcat","seatbelt","frontal","sex","ageOFocc","yearVeh","airbag","occRole")])
accidents$frontal <- factor(accidents$frontal, labels=c("notfrontal","frontal"))
accidents$occRole <- factor(accidents$occRole)
accidents$dvcat <- ordered(accidents$dvcat, 
                          levels=c("1-9km/h","10-24","25-39","40-54","55+"))

saveRDS(accidents, file="accidents.Rd")
```

### <a name="upload-data-to-the-datastore"></a>Charger les données dans le magasin de données
Chargez des données dans le cloud afin qu’elles puissent être accessibles à votre environnement d’apprentissage à distance. Chaque espace de travail Azure Machine Learning est fourni avec un magasin de données par défaut qui stocke les informations de connexion dans le conteneur Blob Azure provisionné dans le compte de stockage associé à l’espace de travail. Le code suivant permet de charger dans ce magasin de données les données relatives aux accidents que vous avez créées précédemment.

```R
ds <- get_default_datastore(ws)

target_path <- "accidentdata"
upload_files_to_datastore(ds,
                          list("./accidents.Rd"),
                          target_path = target_path,
                          overwrite = TRUE)
```


## <a name="train-a-model"></a>Effectuer l’apprentissage d’un modèle

Pour ce tutoriel, placez un modèle de régression logistique sur vos données chargées à l’aide de votre cluster de calcul distant. Pour envoyer un travail, vous devez :

* Préparer le script d’apprentissage
* Créer un estimateur
* Envoi du travail

### <a name="prepare-the-training-script"></a>Préparer le script d’apprentissage
Un script d’apprentissage appelé `accidents.R` vous est fourni dans le répertoire *train-and-deploy-first-model*. Notez les détails suivants **dans le script d’entraînement** qui permettent de tirer parti du service Azure Machine Learning pour l’entraînement :

* Le script d’apprentissage prend un argument `-d` pour rechercher le répertoire qui contient les données d’apprentissage. Quand vous définissez et soumettez votre travail ultérieurement, vous pointez vers le magasin de données pour cet argument. Azure ML monte le dossier de stockage sur le cluster distant pour le travail d’apprentissage.
* Le script d’apprentissage journalise la précision finale sous forme de métrique dans l’enregistrement d’exécution dans Azure ML à l’aide de `log_metric_to_run()`. Le SDK Azure ML fournit un ensemble d’API de journalisation pour la journalisation de diverses métriques pendant les exécutions d’apprentissage. Ces métriques sont enregistrées et conservées dans l’enregistrement d’exécution de l’expérience. Les métriques sont ensuite accessibles à tout moment ou peuvent être affichées dans la page des détails de l’exécution de [Studio](https://ml.azure.com). Consultez la [référence](https://azure.github.io/azureml-sdk-for-r/reference/index.html#section-training-experimentation) pour obtenir l’ensemble complet des méthodes de journalisation `log_*()`.
* Le script d’entraînement enregistre votre modèle dans un répertoire nommé **outputs**. Le dossier `./outputs` bénéficie d’un traitement spécial de la part d’Azure ML. Au cours de l’apprentissage, les fichiers écrits dans `./outputs` sont automatiquement chargés dans votre enregistrement d’exécution par Azure ML et conservés en tant qu’artefacts. En enregistrant le modèle entraîné dans `./outputs`, vous pouvez accéder à votre fichier de modèle et le récupérer même lorsque l’exécution est terminée et que vous n’avez plus accès à votre environnement de formation à distance.

### <a name="create-an-estimator"></a>Créer un estimateur

Un estimateur Azure ML encapsule les informations de configuration d’exécution nécessaires à l’exécution d’un script d’apprentissage sur la cible de calcul. Les exécutions Azure ML sont exécutées en tant que tâches en conteneur sur la cible de calcul spécifiée. Par défaut, l’image Docker générée pour votre travail de formation inclut R, le SDK Azure ML et un ensemble de packages R couramment utilisés. Consultez la liste complète des packages par défaut inclus ici.

Pour créer l’estimateur, définissez :

* Le répertoire qui contient vos scripts nécessaires à l’apprentissage (`source_directory`). Tous les fichiers dans ce répertoire sont chargés dans le(s) nœud(s) de cluster pour l’exécution. Le répertoire doit contenir votre script d’apprentissage et tous les scripts supplémentaires requis.
* Le script d’apprentissage qui sera exécuté (`entry_script`).
* La cible de calcul (`compute_target`), dans le cas présent, le cluster AmlCompute que vous avez créé précédemment.
* Les paramètres requis à partir du script d’apprentissage (`script_params`). Azure ML exécutera votre script d’apprentissage en tant que script de ligne de commande avec `Rscript`. Dans ce tutoriel, vous spécifiez un argument pour le script, le point de montage du répertoire de données auquel vous pouvez accéder avec `ds$path(target_path)`.
* Les dépendances d’environnement requises pour l’apprentissage. L’image Docker par défaut créée pour l’apprentissage contient déjà les trois packages (`caret`, `e1071` et `optparse`) nécessaires dans le script d’apprentissage.  Vous n’avez donc pas besoin de spécifier d’informations supplémentaires. Si vous utilisez des packages R qui ne sont pas inclus par défaut, utilisez le paramètre `cran_packages` de l’estimateur pour ajouter des packages CRAN supplémentaires. Consultez la référence [`estimator()`](https://azure.github.io/azureml-sdk-for-r/reference/estimator.html) pour découvrir l’ensemble complet des options configurables.

```R
est <- estimator(source_directory = "train-and-deploy-first-model",
                 entry_script = "accidents.R",
                 script_params = list("--data_folder" = ds$path(target_path)),
                 compute_target = compute_target
                 )
```

### <a name="submit-the-job-on-the-remote-cluster"></a>Envoyer le travail sur le cluster distant

Enfin, envoyez le travail à exécuter sur votre cluster. `submit_experiment()` retourne un objet Exécution que vous utilisez ensuite pour interagir avec l’exécution. Au total, la première exécution prend **environ 10 minutes**. Mais pour les exécutions suivantes, tant que les dépendances de script ne changent pas, la même image Docker est réutilisée.  Dans ce cas, l’image est mise en cache et la durée de démarrage du conteneur est beaucoup plus rapide.

```R
run <- submit_experiment(exp, est)
```

Vous pouvez afficher les détails de l’exécution dans RStudio Viewer. Cliquez sur le lien « Affichage web » pour accéder à Azure Machine Learning Studio, où vous pouvez surveiller l’exécution dans l’interface utilisateur.

```R
view_run_details(run)
```

L’apprentissage du modèle se produit en arrière-plan. Attendez que l’entraînement du modèle soit terminé avant d’exécuter davantage de code.

```R
wait_for_run_completion(run, show_output = TRUE)
```

Vous (et vos collègues ayant accès à l’espace de travail) pouvez soumettre plusieurs expériences en parallèle et Azure ML se charge de la planification des tâches sur le cluster de calcul. Vous pouvez même configurer le cluster pour le Scale up automatique jusqu’à plusieurs nœuds et le Scale back lorsqu’il n’y a plus de tâches de calcul dans la file d’attente. Cette configuration est un moyen économique pour les équipes de partager des ressources de calcul.

## <a name="retrieve-training-results"></a>Récupérer les résultats de l’apprentissage
Une fois l’apprentissage du modèle terminé, vous pouvez accéder aux artefacts de votre travail qui ont été conservés dans l’enregistrement d’exécution, notamment les métriques journalisées et le modèle entraîné final.

### <a name="get-the-logged-metrics"></a>Obtenir les métriques journalisées
Dans le script d’apprentissage `accidents.R`, vous avez journalisé une métrique à partir de votre modèle : la précision des prédictions dans les données d’apprentissage. Vous pouvez voir les métriques dans [Studio](https://ml.azure.com) ou les extraire dans la session locale sous forme de liste R comme suit :

```R
metrics <- get_run_metrics(run)
metrics
```

Si vous avez exécuté plusieurs expériences (par exemple, en utilisant des variables, des algorithmes ou des hyperparamètres différents), vous pouvez utiliser les métriques de chaque exécution pour comparer et choisir le modèle à utiliser en production.

### <a name="get-the-trained-model"></a>Obtenir le modèle entraîné
Vous pouvez récupérer le modèle entraîné et examiner les résultats dans votre session R locale. Le code suivant permet de télécharger le contenu du répertoire `./outputs`, qui comprend le fichier de modèle.

```R
download_files_from_run(run, prefix="outputs/")
accident_model <- readRDS("outputs/model.rds")
summary(accident_model)
```

Vous voyez des facteurs qui contribuent à une augmentation de la probabilité estimée de décès :

* Vitesse d’impact supérieure 
* Conducteur mâle
* Occupant plus ancien
* Passager

Vous voyez des probabilités de décès inférieures avec :

* Présence d’airbags
* Présence de ceintures de sécurité
* Collision frontale 

L’année de fabrication du véhicule n’a pas d’effet significatif.

Vous pouvez utiliser ce modèle pour faire de nouvelles prédictions :

```R
newdata <- data.frame( # valid values shown below
 dvcat="10-24",        # "1-9km/h" "10-24"   "25-39"   "40-54"   "55+"  
 seatbelt="none",      # "none"   "belted"  
 frontal="frontal",    # "notfrontal" "frontal"
 sex="f",              # "f" "m"
 ageOFocc=16,          # age in years, 16-97
 yearVeh=2002,         # year of vehicle, 1955-2003
 airbag="none",        # "none"   "airbag"   
 occRole="pass"        # "driver" "pass"
 )

## predicted probability of death for these variables, as a percentage
as.numeric(predict(accident_model,newdata, type="response")*100)
```

## <a name="deploy-as-a-web-service"></a>Déployer en tant que service web

Avec votre modèle, vous pouvez prédire le risque de décès lors d’une collision. Utilisez Azure ML pour déployer votre modèle en tant que service de prédiction. Dans ce tutoriel, vous allez déployer le service web dans [Azure Container Instances](../container-instances/index.yml) (ACI).

### <a name="register-the-model"></a>Inscrire le modèle

Tout d’abord, inscrivez le modèle que vous avez téléchargé dans votre espace de travail avec [`register_model()`](https://azure.github.io/azureml-sdk-for-r/reference/register_model.html). Un modèle inscrit peut être n’importe quelle collection de fichiers, mais dans ce cas, l’objet modèle R suffit. Azure ML utilise le modèle inscrit pour le déploiement.

```R
model <- register_model(ws, 
                        model_path = "outputs/model.rds", 
                        model_name = "accidents_model",
                        description = "Predict probablity of auto accident")
```

### <a name="define-the-inference-dependencies"></a>Définir les dépendances d’inférence
Pour créer un service web pour votre modèle, vous devez d’abord créer un script de scoring (`entry_script`), un script R qui prendra en entrée des valeurs de variable (au format JSON) et donnera en sortie une prédiction à partir de votre modèle. Pour ce tutoriel, utilisez le fichier de scoring fourni `accident_predict.R`. Le script de scoring doit contenir une méthode `init()` qui charge votre modèle et retourne une fonction qui utilise le modèle pour effectuer une prédiction basée sur les données d’entrée. Pour plus d’informations, consultez la [documentation](https://azure.github.io/azureml-sdk-for-r/reference/inference_config.html#details).

Ensuite, définissez un **environnement** Azure ML pour les dépendances du package de votre script. Avec un environnement, vous spécifiez des packages R (à partir de CRAN ou autre) qui sont nécessaires à l’exécution de votre script. Vous pouvez également fournir les valeurs des variables d’environnement que votre script peut référencer pour modifier son comportement. Par défaut, Azure ML génère la même image Docker par défaut que celle utilisée avec l’estimateur pour l’apprentissage. Étant donné que le tutoriel n’a pas d’exigences particulières, créez un environnement sans attributs spéciaux.

```R
r_env <- r_environment(name = "basic_env")
```

Si vous souhaitez utiliser votre propre image Docker pour le déploiement, spécifiez le paramètre `custom_docker_image`. Consultez la référence [`r_environment()`](https://azure.github.io/azureml-sdk-for-r/reference/r_environment.html) pour découvrir l’ensemble complet des options configurables permettant de définir un environnement.

À présent, vous avez tout ce dont vous avez besoin pour créer une **configuration d’inférence** pour encapsuler vos dépendances d’environnement et le script de scoring.

```R
inference_config <- inference_config(
  entry_script = "accident_predict.R",
  source_directory = "train-and-deploy-first-model",
  environment = r_env)
```

### <a name="deploy-to-aci"></a>Déployer sur ACI
Dans ce tutoriel, vous allez déployer votre service dans ACI. Ce code provisionne un conteneur unique pour répondre aux demandes entrantes, ce qui convient pour les tests et les charges légères. Pour plus d’options configurables, consultez [`aci_webservice_deployment_config()`](https://azure.github.io/azureml-sdk-for-r/reference/aci_webservice_deployment_config.html). (Pour les déploiements à l’échelle de la production, vous pouvez également [déployer dans Azure Kubernetes Service](https://azure.github.io/azureml-sdk-for-r/articles/deploy-to-aks.html).)

``` R
aci_config <- aci_webservice_deployment_config(cpu_cores = 1, memory_gb = 0.5)
```

Maintenant, vous déployez votre modèle en tant que service web. Le déploiement **peut prendre plusieurs minutes**. 

```R
aci_service <- deploy_model(ws, 
                            'accident-pred', 
                            list(model), 
                            inference_config, 
                            aci_config)

wait_for_deployment(aci_service, show_output = TRUE)
```

## <a name="test-the-deployed-service"></a>Tester le service déployé

Maintenant que votre modèle est déployé en tant que service, vous pouvez tester le service à partir de R à l’aide de [`invoke_webservice()`](https://azure.github.io/azureml-sdk-for-r/reference/invoke_webservice.html).  Fournissez un nouvel ensemble de données à partir desquelles effectuer une prédiction, convertissez-le en JSON et envoyez-le au service.

```R
library(jsonlite)

newdata <- data.frame( # valid values shown below
 dvcat="10-24",        # "1-9km/h" "10-24"   "25-39"   "40-54"   "55+"  
 seatbelt="none",      # "none"   "belted"  
 frontal="frontal",    # "notfrontal" "frontal"
 sex="f",              # "f" "m"
 ageOFocc=22,          # age in years, 16-97
 yearVeh=2002,         # year of vehicle, 1955-2003
 airbag="none",        # "none"   "airbag"   
 occRole="pass"        # "driver" "pass"
 )

prob <- invoke_webservice(aci_service, toJSON(newdata))
prob
```

Vous pouvez aussi obtenir le point de terminaison HTTP du service web, qui accepte les appels du client REST. Vous pouvez partager ce point de terminaison avec toute personne souhaitant tester le service web ou l’intégrer dans une application.

```R
aci_service$scoring_uri
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez les ressources une fois que vous n’en avez plus besoin. Ne supprimez pas les ressources que vous envisagez de réutiliser. 

Supprimez le service web :
```R
delete_webservice(aci_service)
```

Supprimez le modèle inscrit :
```R
delete_model(model)
```

Supprimez le cluster de calcul :
```R
delete_compute(compute)
```

### <a name="delete-everything"></a>Tout supprimer

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

Vous pouvez également conserver le groupe de ressources mais supprimer un espace de travail unique. Affichez les propriétés de l’espace de travail, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que vous avez terminé votre première expérience Azure Machine Learning dans R, apprenez-en plus sur le [SDK Azure Machine Learning pour R](https://azure.github.io/azureml-sdk-for-r/index.html).

* Consultez les exemples des autres dossiers *vignettes* pour en savoir plus sur Azure Machine Learning avec R.