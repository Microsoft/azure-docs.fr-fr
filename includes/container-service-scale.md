---
author: dlepow
ms.service: container-service
ms.topic: include
ms.date: 11/09/2018
ms.author: danlep
ms.openlocfilehash: a4af53e035929a44f74a95b8e9897cb1dc0c6d8e
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "66814897"
---
[!INCLUDE [ACS deprecation](container-service-deprecation.md)]

Après le [déploiement d’un cluster Azure Container Service](../articles/container-service/dcos-swarm/container-service-deployment.md), vous devez peut-être modifier le nombre de nœuds de l’agent. Vous risquez par exemple d’avoir besoin de davantage d’agents afin de pouvoir exécuter davantage d’instances ou d’applications de conteneur. 

Vous pouvez changer le nombre de nœuds d’agent dans un cluster DC/OS, Docker Swarm ou Kubernetes à l’aide du portail Azure ou d’Azure CLI. 

## <a name="scale-with-the-azure-portal"></a>Mise à l’échelle avec le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), recherchez les **services de conteneur**, puis cliquez sur le service de conteneur que vous souhaitez modifier.
2. Dans le panneau **Service de conteneur**, cliquez sur **Agents**.
3. Dans **Nombre de machines virtuelles**, entrez le nombre souhaité de nœuds d’agent.

    ![Mise à l’échelle d’un pool dans le portail](./media/container-service-scale/container-service-scale-portal.png)

4. Pour enregistrer la configuration, cliquez sur **Enregistrer**.

## <a name="scale-with-the-azure-cli"></a>Mise à l’échelle avec Azure CLI

[Installez l’interface de ligne de commande Azure](/cli/azure/install-azure-cli) et connectez-vous à un compte Azure avec `az login`.

### <a name="see-the-current-agent-count"></a>Affichage du nombre actuel d’agents
Pour afficher le nombre d’agents actuellement dans le cluster, exécutez la commande `az acs show`. Elle permet d’afficher la configuration du cluster. Par exemple, la commande suivante affiche la configuration du service de conteneur nommé `containerservice-myACSName` dans le groupe de ressources `myResourceGroup` :

```azurecli
az acs show -g myResourceGroup -n containerservice-myACSName
```

La commande renvoie le nombre d’agents dans la valeur `Count` sous `AgentPoolProfiles`.

### <a name="use-the-az-acs-scale-command"></a>Utilisation de la commande az acs scale
Pour modifier le nombre de nœuds de l’agent, exécutez la commande `az acs scale` et indiquez le **groupe de ressources**, le **nom du service de conteneur** et le **nombre de nouveaux agents**. Si vous utilisez un nombre inférieur ou supérieur, vous descendez ou montez en puissance, respectivement.

Par exemple, pour modifier le nombre d’agents dans le précédent cluster en 10, tapez la commande suivante :

```azurecli
az acs scale -g myResourceGroup -n containerservice-myACSName --new-agent-count 10
```

Azure CLI renvoie une chaîne JSON représentant la nouvelle configuration du service conteneur, avec notamment le nouveau nombre d’agents.

Pour plus d’options de commande, exécutez `az acs scale --help`.

## <a name="scaling-considerations"></a>Mise à l'échelle - Éléments à prendre en compte

* Le nombre de nœuds de l’agent doit être compris entre 1 et 100 (inclus). 

* Votre quota de cœurs peut limiter le nombre de nœuds de l’agent dans un cluster.

* Les opérations de mise à l’échelle de nœud de l’agent sont appliquées à un groupe de machines virtuelles Azure identiques qui contient le pool de l’agent. Dans un cluster de contrôleur de domaine/système d’exploitation, seuls les nœuds de l’agent dans le pool privé sont mis à l’échelle par les opérations indiquées dans cet article.

* Selon l’orchestrateur que vous déployez dans votre cluster, vous pouvez mettre à l’échelle séparément le nombre d’instances d’un conteneur en cours d’exécution sur le cluster. Par exemple, dans un cluster de contrôleur de domaine/système d’exploitation, utilisez l’[interface utilisateur Marathon](../articles/container-service/dcos-swarm/container-service-mesos-marathon-ui.md) pour modifier le nombre d’instances d’une application conteneur.


## <a name="next-steps"></a>Étapes suivantes
* Consultez [d’autres exemples](../articles/container-service/dcos-swarm/container-service-create-acs-cluster-cli.md) d’utilisation des commandes Azure CLI avec Azure Container Service.
* En savoir plus sur les [pools d’agents contrôleur de domaine/système d’exploitation](../articles/container-service/dcos-swarm/container-service-dcos-agents.md) d’Azure Container Service.

