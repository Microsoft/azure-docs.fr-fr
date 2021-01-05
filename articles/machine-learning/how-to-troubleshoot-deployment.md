---
title: Résoudre les problèmes de déploiement d’un service web distant
titleSuffix: Azure Machine Learning
description: Découvrez comment contourner et résoudre les erreurs courantes de déploiement Docker avec Azure Kubernetes Service et Azure Container Instances.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: gvashishtha
ms.author: gopalv
ms.reviewer: jmartens
ms.date: 11/25/2020
ms.topic: troubleshooting
ms.custom: contperf-fy20q4, devx-track-python, deploy, contperf-fy21q2
ms.openlocfilehash: 92cd70e864ae0490ce3f9e7435d9518241f93c8e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031502"
---
# <a name="troubleshoot-model-deployment"></a>Résoudre les problèmes de déploiement de modèle

Découvrez comment contourner et résoudre les erreurs courantes de déploiement Docker distant avec Azure Container Instances (ACI) et Azure Kubernetes Service (AKS) à l’aide d’Azure Machine Learning.

## <a name="prerequisites"></a>Prérequis

* Un **abonnement Azure**. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).
* Le [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).
* [Interface de ligne de commande Azure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).
* [Extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md).

## <a name="steps-for-docker-deployment-of-machine-learning-models"></a>Étapes pour le déploiement Docker des modèles Machine Learning

Lorsque vous déployez un modèle vers un calcul non local dans Azure Machine Learning, les événements suivants se produisent :

1. Le Dockerfile que vous avez spécifié dans votre objet Environments dans votre InferenceConfig est envoyé au cloud, ainsi que le contenu de votre répertoire source.
1. Si une image précédemment générée n’est pas disponible dans votre registre de conteneurs, une nouvelle image Docker est générée dans le cloud et stockée dans le registre de conteneurs par défaut de votre espace de travail.
1. L’image Docker de votre registre de conteneurs est téléchargée sur votre cible de calcul.
1. Le magasin Blob par défaut de votre espace de travail est monté sur votre cible de calcul, ce qui vous donne accès aux modèles inscrits.
1. Votre serveur web est initialisé en exécutant la fonction `init()` de votre script d’entrée.
1. Lorsque votre modèle déployé reçoit une requête, votre fonction `run()` la gère.

La principale différence lors d’un déploiement local est que l’image conteneur est générée sur votre ordinateur local, ce qui explique pourquoi Docker doit être installé pour un déploiement local.

La compréhension de ces étapes doit vous aider à comprendre où se produisent les erreurs.

## <a name="get-deployment-logs"></a>Obtenir les journaux de déploiement

La première étape du débogage des erreurs consiste à obtenir vos journaux de déploiement. Tout d’abord, suivez les [instructions ici](how-to-deploy-and-where.md#connect-to-your-workspace) pour vous connecter à votre espace de travail.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Pour obtenir les journaux d’un service web déployé, procédez comme suit :

```bash
az ml service get-logs --verbose --workspace-name <my workspace name> --name <service name>
```

# <a name="python"></a>[Python](#tab/python)


En supposant que vous avez un objet de type `azureml.core.Workspace` appelé `ws`, vous pouvez effectuer les opérations suivantes :

```python
print(ws.webservices)

# Choose the webservice you are interested in

from azureml.core import Webservice

service = Webservice(ws, '<insert name of webservice>')
print(service.get_logs())
```

---

## <a name="debug-locally"></a>Déboguer en local

Si vous rencontrez des problèmes lors du déploiement d'un modèle sur ACI ou AKS, déployez-le en tant que service web local. L’utilisation d’un service web local facilite la résolution des problèmes. Pour résoudre les problèmes d’un déploiement localement, consultez [cet article](./how-to-troubleshoot-deployment-local.md).

## <a name="container-cannot-be-scheduled"></a>Impossible de planifier le conteneur

Lorsque vous déployez un service sur une cible de calcul Azure Kubernetes Service, Azure Machine Learning tente de planifier le service avec la quantité de ressources demandée. Si, au bout de cinq minutes, aucun nœud n'est disponible dans le cluster avec suffisamment de ressources, le déploiement échoue. Le message d'échec est le suivant : `Couldn't Schedule because the kubernetes cluster didn't have available resources after trying for 00:05:00`. Vous pouvez corriger cette erreur en ajoutant des nœuds supplémentaires, en modifiant la référence SKU de vos nœuds ou en changeant les besoins en ressources de votre service. 

Le message d’erreur indique généralement la ressource insuffisante. Par exemple, un message d’erreur qui indique `0/3 nodes are available: 3 Insufficient nvidia.com/gpu` signifie que le service réclame des GPU et qu’il y a trois nœuds dans le cluster qui n’ont pas de GPU disponibles. Pour résoudre cette erreur, vous pouvez ajouter d’autres nœuds si vous utilisez une référence SKU de GPU, basculer sur une référence SKU compatible GPU sinon ou modifier votre environnement de sorte qu’il ne réclame pas de GPU.  

## <a name="service-launch-fails"></a>Échec du lancement du service

Une fois l’image correctement générée, le système tente de démarrer un conteneur à l’aide de la configuration du déploiement. Dans le cadre du processus de démarrage du conteneur, la fonction `init()` dans votre script de scoring est appelée par le système. Si la fonction `init()` comprend des exceptions non interceptées, l’erreur **CrashLoopBackOff** apparaît peut-être dans le message d’erreur.

Utilisez les informations de l’article [Examiner le journal Docker](how-to-troubleshoot-deployment-local.md#dockerlog).

## <a name="function-fails-get_model_path"></a>Échec de la fonction : get_model_path()

Souvent, dans la fonction `init()` du script de scoring, la fonction [Model.get_model_path()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-model-path-model-name--version-none---workspace-none-) est appelée pour rechercher un fichier de modèle ou un dossier de fichiers de modèle dans le conteneur. Si le fichier ou le dossier de modèle est introuvable, la fonction échoue. Le moyen le plus simple de déboguer cette erreur consiste à exécuter le code Python ci-dessous dans l’interpréteur de commandes du conteneur :

```python
from azureml.core.model import Model
import logging
logging.basicConfig(level=logging.DEBUG)
print(Model.get_model_path(model_name='my-best-model'))
```

Cet exemple affiche le chemin local (relatif à `/var/azureml-app`) sur le conteneur où votre script de scoring est censé trouver le fichier de modèle ou le dossier. Vous pouvez ensuite vérifier si le fichier ou dossier se trouve effectivement à l’emplacement indiqué.

La définition du niveau de journalisation sur DEBUG peut entraîner l’enregistrement d’informations supplémentaires, qui peuvent être utiles pour identifier l’échec.

## <a name="function-fails-runinput_data"></a>Échec de la fonction : run(input_data)

Si le service est déployé avec succès, mais qu’il plante quand vous publiez des données sur le point de terminaison de scoring, vous pouvez ajouter une instruction d’interception d’erreur dans votre fonction `run(input_data)` afin qu’elle retourne un message d’erreur détaillé. Par exemple :

```python
def run(input_data):
    try:
        data = json.loads(input_data)['data']
        data = np.array(data)
        result = model.predict(data)
        return json.dumps({"result": result.tolist()})
    except Exception as e:
        result = str(e)
        # return error message back to the client
        return json.dumps({"error": result})
```

**Remarque** : Retournez les messages d’erreur à partir de l’appel `run(input_data)` exclusivement à des fins de débogage. Pour des raisons de sécurité, vous ne devez pas renvoyer des messages d’erreur de cette façon dans un environnement de production.

## <a name="http-status-code-502"></a>Code d’état HTTP 502

Un code d’état 502 indique que le service a levé une exception ou s’est planté dans la méthode `run()` du fichier score.py. Utilisez les informations de cet article pour déboguer le fichier.

## <a name="http-status-code-503"></a>Code d’état HTTP 503

Les déploiements Azure Kubernetes Service prennent en charge la mise à l’échelle automatique, ce qui permet d’ajouter des réplicas pour gérer une charge supplémentaire. L'Autoscaler est conçu pour gérer des changements **progressifs** de charge. Si vous recevez des pics importants de demandes par seconde, les clients peuvent recevoir un code d’état HTTP 503. Même si l'Autoscaler réagit rapidement, il faut beaucoup de temps à AKS pour créer des conteneurs supplémentaires.

La décision d’effectuer un scale-up/down est basée sur l’utilisation des réplicas de conteneur actuels. Le nombre de réplicas occupés (qui traitent une demande) divisé par le nombre total de réplicas actuels donne l’utilisation en cours. Si ce nombre dépasse `autoscale_target_utilization`, d’autres réplicas sont créés. S’il est inférieur, des réplicas sont supprimés. La décision d’ajouter des réplicas est hâtive et rapide (environ 1 seconde). La décision de supprimer des réplicas est prudente (environ 1 minute). Par défaut, l'utilisation de la cible de mise à l'échelle automatique est définie sur **70 %** , ce qui signifie que le service peut gérer les pics de demandes par seconde (RPS) **jusqu'à 30 %** .

Il existe deux mesures permettant d’empêcher les codes d’état 503 :

> [!TIP]
> Ces deux approches peuvent être utilisées individuellement ou conjointement.

* Modifiez le niveau d’utilisation auquel la mise à l’échelle automatique crée de nouveaux réplicas. Vous pouvez ajuster la cible de l’utilisation en définissant `autoscale_target_utilization` sur une valeur inférieure.

    > [!IMPORTANT]
    > Cette modification n’entraîne pas la création de réplicas *plus rapidement*. Ils sont créés à un seuil d’utilisation inférieur. Au lieu d’attendre jusqu’à ce que le service soit utilisé à 70 %, la modification de la valeur sur 30 % provoque la création de réplicas lors d’une utilisation à 30 %.
    
    Si le service web utilise déjà les réplicas maximum actuels et que des codes d’état 503 subsistent, augmentez la valeur `autoscale_max_replicas` pour augmenter le nombre maximal de réplicas.

* Modifiez le nombre minimal de réplicas. L’augmentation du nombre minimal de réplicas offre un plus grand pool pour gérer les pics entrants.

    Pour augmenter le nombre minimal de réplicas, définissez `autoscale_min_replicas` sur une valeur plus élevée. Vous pouvez calculer les réplicas requis en utilisant le code suivant, en remplaçant les valeurs par des valeurs spécifiques de votre projet :

    ```python
    from math import ceil
    # target requests per second
    targetRps = 20
    # time to process the request (in seconds)
    reqTime = 10
    # Maximum requests per container
    maxReqPerContainer = 1
    # target_utilization. 70% in this example
    targetUtilization = .7

    concurrentRequests = targetRps * reqTime / targetUtilization

    # Number of container replicas
    replicas = ceil(concurrentRequests / maxReqPerContainer)
    ```

    > [!NOTE]
    > Si vous recevez des pics de demande supérieurs au nouveau nombre minimal pouvant être géré par les réplicas, il se peut que des codes d’état 503 réapparaissent. Par exemple, si le trafic vers votre service augmente, vous devrez peut-être augmenter le nombre minimal de réplicas.

Pour plus d’informations sur la configuration de `autoscale_target_utilization`, `autoscale_max_replicas`, et `autoscale_min_replicas`, consultez les informations de référence sur le module [AksWebservice](/python/api/azureml-core/azureml.core.webservice.akswebservice?preserve-view=true&view=azure-ml-py).

## <a name="http-status-code-504"></a>Code d’état HTTP 504

Un code d’état 504 indique que la requête a expiré. Le délai d’expiration par défaut est de une minute.

Vous pouvez augmenter le délai d’expiration ou essayer d’accélérer le service en modifiant le fichier score.py pour supprimer les appels inutiles. Si ces actions ne corrigent pas le problème, utilisez les informations de cet article pour déboguer le fichier score.py. Le code peut être dans un état sans réponse ou une boucle infinie.

## <a name="advanced-debugging"></a>Débogage avancé

Vous pouvez être amené à déboguer interactivement le code Python contenu dans votre modèle de déploiement. Par exemple, si le script d’entrée échoue pour une raison ne pouvant pas être déterminée par une journalisation supplémentaire. À l’aide de Visual Studio Code et de debugpy, vous pouvez attacher au code qui s’exécute dans le conteneur Docker.

Pour plus d’informations, consultez le [guide de débogage interactif dans VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-deployments).

## <a name="model-deployment-user-forum"></a>[Forum utilisateurs sur le déploiement de modèle](/answers/topics/azure-machine-learning-inference.html)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le déploiement :

* [Où et comment effectuer un déploiement](how-to-deploy-and-where.md)
* [Tutoriel : Effectuer l'apprentissage de modèles et les déployer](tutorial-train-models-with-aml.md)
* [Comment exécuter et déboguer des expériences localement](./how-to-debug-visual-studio-code.md)