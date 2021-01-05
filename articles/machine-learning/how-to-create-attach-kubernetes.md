---
title: Créer et attacher un cluster Azure Kubernetes Service
titleSuffix: Azure Machine Learning
description: Découvrez comment créer un cluster de service Azure Kubernetes via Azure Machine Learning ou comment attacher un cluster AKS existant à votre espace de travail.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 10/02/2020
ms.openlocfilehash: e773c2db9c7849dd9680f8ae0c600405f422d7e1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463184"
---
# <a name="create-and-attach-an-azure-kubernetes-service-cluster"></a>Créer et attacher un cluster Azure Kubernetes Service

Azure Machine Learning peut déployer des modèles Machine Learning entraînés sur Azure Kubernetes Service. Toutefois, vous devez d’abord __créer__ un cluster Azure Kubernetes Service (AKS) à partir de votre espace de travail Azure ML ou __attacher__ un cluster AKS existant. Cet article fournit des informations sur la création et l’attachement d’un cluster.

## <a name="prerequisites"></a>Prérequis

- Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

- L’[extension Azure CLI pour Machine Learning service](reference-azure-machine-learning-cli.md), le [SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’[extension Azure Machine Learning pour Visual Studio Code](tutorial-setup-vscode-extension.md).

- Si vous envisagez d’utiliser un réseau virtuel Azure pour sécuriser la communication entre votre espace de travail Azure ML et le cluster AKS, lisez l’article sur l’[isolement des réseaux lors de l’entraînement et de l’inférence](./how-to-network-security-overview.md).

## <a name="limitations"></a>Limites

- Si vous devez déployer un **équilibreur de charge SLB** (Standard Load Balancer) au lieu d’un équilibreur de charge BLB (Basic Load Balancer) dans votre cluster, créez un cluster sur le portail AKS/l’interface CLI/le kit SDK, puis **attachez-le** à l’espace de travail AML.

- Si vous avez une stratégie Azure Policy qui limite la création d’adresses IP publiques, la création du cluster AKS échouera. AKS a besoin d’une adresse IP publique pour le [trafic de sortie](../aks/limit-egress-traffic.md). L’article relatif au trafic de sortie fournit également des conseils pour verrouiller le trafic sortant du cluster via l’IP publique, sauf pour quelques noms de domaine complets. Il existe deux façons d’activer une adresse IP publique :
    - Le cluster peut utiliser l’adresse IP publique créée par défaut avec l’équilibreur de charge BLB ou SLB, ou
    - Le cluster peut être créé sans adresse IP publique ; une adresse IP publique est alors configurée avec un pare-feu avec une route définie par l’utilisateur. Pour plus d’informations, consultez l’article [Personnaliser la sortie du cluster avec une route définie par l’utilisateur](../aks/egress-outboundtype.md).
    
    Le plan de contrôle AML ne communique pas avec cette adresse IP publique. Il communique avec le plan de contrôle AKS pour les déploiements. 

- Si vous **attachez** un cluster AKS pour lequel une [plage d’adresses IP autorisées a accès au serveur d’API](../aks/api-server-authorized-ip-ranges.md), activez les plages d’adresses IP du plan de contrôle AML pour le cluster AKS. Le plan de contrôle AML est déployé sur les régions jumelées et déploie des pods d’inférence sur le cluster AKS. Le déploiement des pods d’inférence n’est pas possible sans accès au serveur d’API. Utilisez les [plages d’adresses IP](https://www.microsoft.com/download/confirmation.aspx?id=56519) des deux [régions jumelées](../best-practices-availability-paired-regions.md) lors de l’activation des plages d’adresses IP dans un cluster AKS.

    Les plages d’adresses IP autorisées ne fonctionnent qu’avec Standard Load Balancer.

- Lorsque vous **attachez** un cluster AKS, il doit se trouver dans le même abonnement Azure que votre espace de travail Azure Machine Learning.

- Si vous voulez utiliser un cluster AKS privé (avec Azure Private Link), vous devez d’abord créer le cluster, puis l’**attacher** à l’espace de travail. Pour plus d’informations, consultez [Créer un cluster Azure Kubernetes Service privé](../aks/private-clusters.md).

- Le nom de calcul du cluster AKS DOIT être unique au sein de l’espace de travail Azure ML.
    - Le nom est obligatoire et doit comprendre entre 3 et 24 caractères.
    - Les caractères valides sont les lettres majuscules et minuscules, les chiffres et le caractère -.
    - Le nom doit commencer par une lettre.
    - Le nom doit être unique parmi tous les calculs existants au sein d’une région Azure. Si le nom que vous choisissez n’est pas unique, une alerte s’affiche.
   
 - Si vous souhaitez déployer des modèles sur des nœuds **GPU** ou **FPGA** (ou sur une référence SKU spécifique), vous devez créer un cluster de la référence SKU en question. Il n’est pas possible de créer un pool de nœuds secondaire dans un cluster existant et de déployer des modèles dans le pool de nœuds secondaire.
 
- Lors de la création ou de l’attachement d’un cluster, vous pouvez indiquer s’il sera utilisé dans un contexte de __développement/test__ ou de __production__. Si vous souhaitez créer un cluster AKS à des fins de __développement__, de __validation__ et de __test__ plutôt que de production, définissez l’__objet du cluster__ sur __développement/test__. Si vous n’indiquez pas l’objet du cluster, un cluster de __production__ est créé. 

    > [!IMPORTANT]
    > Un cluster de __développement/test__ n’est pas approprié pour le trafic de production et peut augmenter les temps d’inférence. Par ailleurs, les clusters de développement/test ne garantissent pas une tolérance de panne.

- Lors de la création ou de l’attachement d’un cluster, si celui-ci est destiné à la __production__, il doit contenir au moins 12 __processeurs virtuels__. Le nombre de processeurs virtuels doit être calculé en multipliant le __nombre de nœuds__ du cluster par le __nombre de cœurs__ fournis par la taille de machine virtuelle sélectionnée. Par exemple, si vous utilisez une taille de machine virtuelle de « Standard_D3_v2 », qui comporte 4 cœurs virtuels, vous devez définir le nombre de nœuds sur un nombre supérieur ou égal à 3.

    Nous vous recommandons d’utiliser au moins 2 processeurs virtuels pour les clusters de __développement/test__.

- Le kit de développement logiciel (SDK) Azure Machine Learning ne prend pas en charge la mise à l’échelle d'un cluster AKS. Pour mettre à l’échelle les nœuds du cluster, utilisez l’interface utilisateur de votre cluster AKS dans Azure Machine Learning Studio. Vous pouvez modifier le nombre de nœuds, mais pas la taille de machine virtuelle du cluster. Pour plus d’informations sur la mise à l’échelle des nœuds d’un cluster AKS, consultez les articles suivants :

    - [Mettre à l’échelle manuellement le nombre de nœuds dans un cluster AKS](../aks/scale-cluster.md)
    - [Configurer l’utilitaire de mise à l’échelle automatique de cluster dans AKS](../aks/cluster-autoscaler.md)

## <a name="azure-kubernetes-service-version"></a>Version d’Azure Kubernetes Service

Azure Kubernetes Service vous permet de créer un cluster à l’aide de différentes versions de Kubernetes. Pour plus d’informations sur les versions disponibles, consultez [Versions de Kubernetes prises en charge dans Azure Kubernetes Service](../aks/supported-kubernetes-versions.md).

Lorsque vous **créez** un cluster Azure Kubernetes Service à l’aide de l’une des méthodes suivantes, vous *n’avez pas le choix de la version* du cluster créé :

* Studio Azure Machine Learning ou section Azure Machine Learning du portail Azure.
* Extension Machine Learning pour l’interface de ligne de commande Azure.
* Kit de développement logiciel (SDK) Azure Machine Learning.

Ces méthodes de création d’un cluster AKS utilisent la version __par défaut__ du cluster. *La version par défaut change au fil du temps* et de la publication de nouvelles versions de Kubernetes.

Lors de l’**attachement** d’un cluster AKS existant, nous prenons en charge toutes les versions d’AKS actuellement prises en charge.

> [!NOTE]
> Dans de rares cas, il est possible que vous ayez un cluster plus ancien qui n’est plus pris en charge. L’opération d’attachement renvoie alors une erreur et affiche une liste des versions actuellement prises en charge.
>
> Vous pouvez attacher des **préversions**. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. La prise en charge en cas d’utilisation de préversions peut être limitée. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="available-and-default-versions"></a>Versions par défaut et disponibles

Pour trouver les versions AKS par défaut et disponibles, utilisez la commande [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) [az aks get-versions](/cli/azure/aks?view=azure-cli-latest&preserve-view=true#az_aks_get_versions). Par exemple, la commande suivante renvoie les versions disponibles dans la région USA Ouest :

```azurecli-interactive
az aks get-versions -l westus -o table
```

Le résultat de la commande se présente ainsi :

```text
KubernetesVersion    Upgrades
-------------------  ----------------------------------------
1.18.6(preview)      None available
1.18.4(preview)      1.18.6(preview)
1.17.9               1.18.4(preview), 1.18.6(preview)
1.17.7               1.17.9, 1.18.4(preview), 1.18.6(preview)
1.16.13              1.17.7, 1.17.9
1.16.10              1.16.13, 1.17.7, 1.17.9
1.15.12              1.16.10, 1.16.13
1.15.11              1.15.12, 1.16.10, 1.16.13
```

Pour trouver la version par défaut utilisée lors de la **création** d’un cluster via Azure Machine Learning, vous pouvez utiliser le paramètre `--query` pour sélectionner la version par défaut :

```azurecli-interactive
az aks get-versions -l westus --query "orchestrators[?default == `true`].orchestratorVersion" -o table
```

Le résultat de la commande se présente ainsi :

```text
Result
--------
1.16.13
```

Si vous souhaitez **vérifier par programme les versions disponibles**, utilisez l’API REST [Container Service Client - List Orchestrators](/rest/api/container-service/container%20service%20client/listorchestrators). Pour trouver les versions disponibles, repérez les entrées où `orchestratorType` est `Kubernetes`. Les entrées `orchestrationVersion` associées contiennent les versions disponibles qui peuvent être **attachées** à votre espace de travail.

Pour trouver la version par défaut utilisée lors de la **création** d’un cluster via Azure Machine Learning, trouvez l’entrée dans laquelle `orchestratorType` est `Kubernetes` et `default` est `true`. La valeur `orchestratorVersion` associée est la version par défaut. L’extrait de code JSON suivant est un exemple d’entrée :

```json
...
 {
        "orchestratorType": "Kubernetes",
        "orchestratorVersion": "1.16.13",
        "default": true,
        "upgrades": [
          {
            "orchestratorType": "",
            "orchestratorVersion": "1.17.7",
            "isPreview": false
          }
        ]
      },
...
```

## <a name="create-a-new-aks-cluster"></a>Créer un cluster AKS

**Durée estimée** : environ 10 minutes.

La création ou l’attachement d’un cluster AKS est un processus à effectuer une seule fois pour votre espace de travail. Vous pouvez le réutiliser pour vos autres déploiements. Si vous supprimez le cluster ou le groupe de ressources dans lequel il se trouve, vous devrez recréer un cluster lors du prochain déploiement. Vous pouvez avoir plusieurs clusters AKS attachés à votre espace de travail.

L’exemple suivant montre comment créer un cluster AKS à l’aide du Kit de développement logiciel (SDK) et de l’interface CLI :

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

# Example configuration to use an existing virtual network
# prov_config.vnet_name = "mynetwork"
# prov_config.vnet_resourcegroup_name = "mygroup"
# prov_config.subnet_name = "default"
# prov_config.service_cidr = "10.0.0.0/16"
# prov_config.dns_service_ip = "10.0.0.10"
# prov_config.docker_bridge_cidr = "172.17.0.1/16"

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

Pour plus d’informations sur les classes, les méthodes et les paramètres utilisés dans cet exemple, consultez les documents de référence suivants :

* [AksCompute.ClusterPurpose](/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?preserve-view=true&view=azure-ml-py)
* [AksCompute.provisioning_configuration](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py&preserve-view=true#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [ComputeTarget.create](/python/api/azureml-core/azureml.core.compute.computetarget?preserve-view=true&view=azure-ml-py#create-workspace--name--provisioning-configuration-)
* [ComputeTarget.wait_for_completion](/python/api/azureml-core/azureml.core.compute.computetarget?preserve-view=true&view=azure-ml-py#wait-for-completion-show-output-false-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks
```

Pour plus d’informations, consultez la référence [AZ ml computetarget Create AKS](/cli/azure/ext/azure-cli-ml/ml/computetarget/create?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-aks).

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour plus d’informations sur la création d’un cluster AKS dans le portail, consultez [Créer des cibles de calcul dans le studio Azure Machine Learning](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="attach-an-existing-aks-cluster"></a>Attacher un cluster AKS existant

**Durée estimée** : 5 minutes environ.

Si vous avez déjà un cluster AKS dans votre abonnement Azure et qu’il s’agit d’une version 1,17 ou antérieure, vous pouvez l’utiliser pour déployer votre image.

> [!TIP]
> Le cluster AKS existant peut se trouver dans une autre région Azure que celle de votre espace de travail Azure Machine Learning.


> [!WARNING]
> Ne créez pas plusieurs attachements en même temps dans le même cluster AKS depuis votre espace de travail. Par exemple, l’attachement d’un cluster AKS à un espace de travail en utilisant deux noms différents. Chaque nouvel attachement va supprimer le ou les attachements précédents.
>
> Si vous voulez réattacher un cluster AKS, par exemple pour changer le protocole TLS ou un autre paramètre de configuration du cluster, vous devez d’abord supprimer l’attachement existant en utilisant [AksCompute.detach()](/python/api/azureml-core/azureml.core.compute.akscompute?preserve-view=true&view=azure-ml-py#detach--).

Pour plus d’informations sur la création d’un cluster AKS à l’aide de l’interface de ligne de commande Azure ou du portail, consultez les articles suivants :

* [Créer un cluster AKS (CLI)](/cli/azure/aks?bc=%2fazure%2fbread%2ftoc.json&preserve-view=true&toc=%2fazure%2faks%2fTOC.json&view=azure-cli-latest#az-aks-create)
* [Créer un cluster AKS (portail)](../aks/kubernetes-walkthrough-portal.md?preserve-view=true&view=azure-cli-latest)
* [Créer un cluster AKS (modèle ARM sur les modèles de démarrage rapide Azure)](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aks-azml-targetcompute)

L’exemple suivant montre comment attacher un cluster AKS existant à votre espace de travail :

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'myexistingcluster'

# Attach the cluster to your workgroup. If the cluster has less than 12 virtual CPUs, use the following instead:
# attach_config = AksCompute.attach_configuration(resource_group = resource_group,
#                                         cluster_name = cluster_name,
#                                         cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'myaks', attach_config)

# Wait for the attach process to complete
aks_target.wait_for_completion(show_output = True)
```

Pour plus d’informations sur les classes, les méthodes et les paramètres utilisés dans cet exemple, consultez les documents de référence suivants :

* [AksCompute.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py&preserve-view=true#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [AksCompute.ClusterPurpose](/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose?preserve-view=true&view=azure-ml-py)
* [AksCompute.attach](/python/api/azureml-core/azureml.core.compute.computetarget?preserve-view=true&view=azure-ml-py#attach-workspace--name--attach-configuration-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour attacher un cluster existant à l’aide de l’interface de ligne de commande, vous devez récupérer l’ID de ressource du cluster existant. Pour obtenir cette valeur, utilisez la commande suivante. Remplacez `myexistingcluster` par le nom de votre cluster AKS. Remplacez `myresourcegroup` par le groupe de ressources qui contient le cluster :

```azurecli
az aks show -n myexistingcluster -g myresourcegroup --query id
```

Cette commande retourne une valeur semblable au texte suivant :

```text
/subscriptions/{GUID}/resourcegroups/{myresourcegroup}/providers/Microsoft.ContainerService/managedClusters/{myexistingcluster}
```

Pour attacher le cluster existant à votre espace de travail, utilisez la commande suivante. Remplacez `aksresourceid` par la valeur retournée par la commande précédente. Remplacez `myresourcegroup` par le groupe de ressources qui contient votre espace de travail. Remplacez `myworkspace` par le nom de votre espace de travail.

```azurecli
az ml computetarget attach aks -n myaks -i aksresourceid -g myresourcegroup -w myworkspace
```

Pour plus d’informations, consultez la référence [az ml computetarget attach aks](/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks).

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour plus d’informations sur l’attachement d’un cluster AKS dans le portail, consultez [Créer des cibles de calcul dans le studio Azure Machine Learning](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="detach-an-aks-cluster"></a>Détacher un cluster AKS

Pour détacher un cluster de votre espace de travail, utilisez l'une des méthodes suivantes :

> [!WARNING]
> L'utilisation d'Azure Machine Learning studio, du SDK ou de l'extension Azure CLI pour Machine Learning pour détacher un cluster AKS **ne supprime pas le cluster AKS**. Pour supprimer le cluster, consultez [Utiliser Azure CLI avec AKS](../aks/kubernetes-walkthrough.md#delete-the-cluster).

# <a name="python"></a>[Python](#tab/python)

```python
aks_target.detach()
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour détacher le cluster existant de votre espace de travail, utilisez la commande suivante. Remplacez `myaks` par le nom sous lequel le cluster AKS est attaché à votre espace de travail. Remplacez `myresourcegroup` par le groupe de ressources qui contient votre espace de travail. Remplacez `myworkspace` par le nom de votre espace de travail.

```azurecli
az ml computetarget detach -n myaks -g myresourcegroup -w myworkspace
```

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans Azure Machine Learning studio, sélectionnez __Calcul__, __Clusters d'inférence__ et le cluster que vous souhaitez supprimer. Utilisez le lien __Détacher__ pour détacher le cluster.

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser Azure RBAC pour l’autorisation Kubernetes](../aks/manage-azure-rbac.md)
* [Comment et où déployer un modèle ?](how-to-deploy-and-where.md)
* [Déployer un modèle sur un cluster Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)