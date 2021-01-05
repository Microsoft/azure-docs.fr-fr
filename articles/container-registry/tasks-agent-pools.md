---
title: Utiliser un pool dédié pour exécuter des tâches - Tâches
description: Configurez un pool de calcul dédié (pool d’agents) dans votre registre pour exécuter une tâche Azure Container Registry.
ms.topic: article
ms.date: 10/12/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 94956af14aad2b62e6455f443329bcd3232095c0
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844912"
---
# <a name="run-an-acr-task-on-a-dedicated-agent-pool"></a>Exécuter une tâche ACR sur un pool d’agents dédié

Configurez un pool de machines virtuelles gérées par Azure (*pool d’agents*) pour permettre l’exécution de vos [tâches Azure Container Registry][acr-tasks] dans un environnement de calcul dédié. Une fois que vous avez configuré un ou plusieurs pools dans votre registre, vous pouvez choisir un pool pour exécuter une tâche à la place de l’environnement de calcul par défaut du service.

Un pool d’agents fournit les fonctionnalités suivantes :

- **Prise en charge des réseaux virtuels** : affectez un pool d’agents à un réseau virtuel Azure, ce qui permet d’accéder aux ressources du réseau virtuel, comme un registre de conteneurs, un coffre de clés ou un stockage.
- **Mise à l’échelle en fonction des besoins** : augmentez le nombre d’instances dans un pool d’agents pour les tâches nécessitant beaucoup de ressources, ou mettez à l’échelle sur zéro. La facturation est basée sur l’allocation de pool. Pour plus d’informations, consultez les [tarifs](https://azure.microsoft.com/pricing/details/container-registry/).
- **Options flexibles** : choisissez parmi différents [niveaux de pool](#pool-tiers) et mettez à l’échelle des options pour répondre aux besoins de la charge de travail de votre tâche.
- **Gestion Azure** : les pools de tâches sont corrigés et gérés par Azure, ce qui permet une allocation réservée sans avoir à gérer les machines virtuelles individuelles.

Cette fonctionnalité est disponible uniquement au niveau de service **Premium** de registre de conteneurs. Pour plus d’informations sur les niveaux de service et les limites de registre, consultez [Références SKU Azure Container Registry][acr-tiers].

> [!IMPORTANT]
> Cette fonctionnalité est actuellement en préversion et certaines [limitations s’appliquent](#preview-limitations). Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires][terms-of-use]. Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.
>

## <a name="preview-limitations"></a>Limitations de la version préliminaire

- Les pools d’agents de tâche prennent actuellement en charge les nœuds Linux. Les nœuds Windows ne sont actuellement pas pris en charge.
- Les pools d'agents de tâche sont disponibles en préversion dans les régions suivantes : USA Ouest 2, USA Centre Sud, USA Est 2, USA Est, USA Centre, USGov Arizona, USGov Texas et USGov Virginie.
- Pour chaque registre, le quota total par défaut de processeurs virtuels (Core) est de 16 pour tous les pools d’agents standard et de 0 pour les pools d’agents isolés. Ouvrez une [demande de support ][open-support-ticket] pour une allocation supplémentaire.
- À l’heure actuelle, vous ne pouvez pas annuler une tâche exécutée sur un pool d’agents.

## <a name="prerequisites"></a>Prérequis

* Pour utiliser les étapes Azure CLI décrites dans cet article, vous devez disposer d’Azure CLI version 2.3.1 ou ultérieure. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli]. Ou exécutez cela dans [Azure Cloud Shell](../cloud-shell/quickstart.md).
* Si vous ne disposez pas d’un registre de conteneurs, [créez-en un][create-reg-cli] (niveau Premium requis) dans une région de préversion.

## <a name="pool-tiers"></a>Niveaux de pool

Les niveaux de pool d’agents fournissent les ressources suivantes par instance dans le pool.

|Niveau    | Type  |  UC  |Mémoire (Go)  |
|---------|---------|---------|---------|
|S1     |  standard    | 2       |    3     |
|S2     |  standard    | 4       |    8     |
|S3     |  standard    | 8       |   16     |
|I6     |  isolés    | 64     |   216     |


## <a name="create-and-manage-a-task-agent-pool"></a>Créer et gérer un pool d’agents de tâche

### <a name="set-default-registry-optional"></a>Définir le registre par défaut (facultatif)

Pour simplifier les commandes Azure CLI qui suivent, définissez le registre par défaut en exécutant la commande [az configure][az-configure] :

```azurecli
az configure --defaults acr=<registryName>
```

Les exemples suivants supposent que vous avez défini le registre par défaut. Si ce n’est pas le cas, transmettez un paramètre `--registry <registryName>` dans chaque commande `az acr`.

### <a name="create-agent-pool"></a>Créer un pool d’agents

Créez un pool d’agents à l’aide de la commande [az acr agentpool create][az-acr-agentpool-create]. L’exemple suivant crée un pool de niveau S2 (4 UC/instance). Par défaut, le pool contient 1 instance.

```azurecli
az acr agentpool create \
    --name myagentpool \
    --tier S2
```

> [!NOTE]
> La création d’un pool d’agents et d’autres opérations de gestion de pool prend quelques minutes.

### <a name="scale-pool"></a>Pool de mise à l’échelle

Mettez à l’échelle la taille du pool vers le haut ou vers le bas avec la commande [az acr agentpool update][az-acr-agentpool-update]. L’exemple suivant met à l’échelle le pool à 2 instances. Vous pouvez mettre à l’échelle jusqu’à 0 instance.

```azurecli
az acr agentpool update \
    --name myagentpool \
    --count 2
```

## <a name="create-pool-in-a-virtual-network"></a>Créer un pool dans un réseau virtuel

### <a name="add-firewall-rules"></a>Ajouter des règles de pare-feu

Les pools d’agents de tâche requièrent un accès aux services Azure suivants. Les règles de pare-feu suivantes doivent être ajoutées à tous les groupes de sécurité réseau existants ou itinéraires définis par l’utilisateur.

| Sens | Protocol | Source         | Port source | Destination          | Port de destination | Utilisé    |
|-----------|----------|----------------|-------------|----------------------|-----------|---------|
| Règle de trafic sortant  | TCP      | VirtualNetwork | Quelconque         | AzureKeyVault        | 443       | Default |
| Règle de trafic sortant  | TCP      | VirtualNetwork | Quelconque         | Stockage              | 443       | Default |
| Règle de trafic sortant  | TCP      | VirtualNetwork | Quelconque         | Event Hub             | 443       | Default |
| Règle de trafic sortant  | TCP      | VirtualNetwork | Quelconque         | AzureActiveDirectory | 443       | Default |
| Règle de trafic sortant  | TCP      | VirtualNetwork | Quelconque         | AzureMonitor         | 443       | Default |

> [!NOTE]
> Si vos tâches requièrent des ressources supplémentaires provenant de l’Internet public, ajoutez les règles correspondantes. Par exemple, des règles supplémentaires sont nécessaires pour exécuter une tâche de génération de Docker qui extrait les images de base de Docker Hub ou restaure un package NuGet.

### <a name="create-pool-in-vnet"></a>Créer un pool dans un réseau virtuel

L’exemple suivant crée un pool d’agents dans le sous-réseau *mysubnet* du réseau *myvnet* :

```azurecli
# Get the subnet ID
subnetId=$(az network vnet subnet show \
        --resource-grop myresourcegroup \
        --vnet-name myvnet \
        --name mysubnetname \
        --query id --output tsv)

az acr agentpool create \
    --name myagentpool \
    --tier S2 \
    --subnet-id $subnetId
```

## <a name="run-task-on-agent-pool"></a>Exécuter la tâche sur le pool d’agents

Les exemples suivants montrent comment spécifier un pool d’agents lors de la mise en file d’attente d’une tâche.

> [!NOTE]
> Pour utiliser un pool d’agents dans une tâche ACR, assurez-vous que le pool contienne au moins 1 instance.
>

### <a name="quick-task"></a>Tâche rapide

Mettez en file d’attente une tâche rapide sur le pool d’agents à l’aide de la commande [az acr build][az-acr-build] et transmettez le paramètre `--agent-pool` :

```azurecli
az acr build \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --file Dockerfile \
    https://github.com/Azure-Samples/acr-build-helloworld-node.git
```

### <a name="automatically-triggered-task"></a>Tâche déclenchée automatiquement

Par exemple, créez une tâche planifiée sur le pool d’agents avec [az acr task create][az-acr-task-create], en transmettant le paramètre `--agent-pool`.

```azurecli
az acr task create \
    --name mytask \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --schedule "0 21 * * *" \
    --file Dockerfile \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git \
    --commit-trigger-enabled false
```

Pour vérifier la configuration des tâches, exécutez [az acr task run][az-acr-task-run] :

```azurecli
az acr task run \
    --name mytask
```

### <a name="query-pool-status"></a>État du pool de requêtes

Pour trouver le nombre d’exécutions actuellement planifiées sur le pool d’agents, exécutez [az acr agentpool show][az-acr-agentpool-show].

```azurecli
az acr agentpool show \
    --name myagentpool \
    --queue-count
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir plus d’exemples de builds et de maintenance d’image de conteneur dans le cloud, consultez la [série de tutoriels sur ACR Tasks](container-registry-tutorial-quick-task.md).



[acr-tasks]:           container-registry-tasks-overview.md
[acr-tiers]:           container-registry-skus.md
[azure-cli]:           /cli/azure/install-azure-cli
[open-support-ticket]: https://aka.ms/acr/support/create-ticket
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[az-configure]: /cli/azure#az-configure
[az-acr-agentpool-create]: /cli/azure/acr/agentpool#az-acr-agentpool-create
[az-acr-agentpool-update]: /cli/azure/acr/agentpool#az-acr-agentpool-update
[az-acr-agentpool-show]: /cli/azure/acr/agentpool#az-acr-agentpool-show
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[create-reg-cli]: container-registry-get-started-azure-cli.md
