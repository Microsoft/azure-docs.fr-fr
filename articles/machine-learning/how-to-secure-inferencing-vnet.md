---
title: Sécuriser les environnements d’inférence à l’aide de réseaux virtuels
titleSuffix: Azure Machine Learning
description: Utilisez un réseau virtuel Azure isolé pour sécuriser votre environnement d’inférence Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 10/23/2020
ms.custom: contperf-fy20q4, tracking-python, contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: ba5dd0ccc06a443378f87cfb92da76616ad67263
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029513"
---
# <a name="secure-an-azure-machine-learning-inferencing-environment-with-virtual-networks"></a>Sécuriser un environnement d’inférence Azure Machine Learning à l’aide de réseaux virtuels

Dans cet article, vous allez apprendre à sécuriser les environnements d’inférence à l’aide d’un réseau virtuel dans Azure Machine Learning.

Cet article est le quatrième d’une série de cinq de qui vous guide à travers le processus de sécurisation d’un workflow Azure Machine Learning. Nous vous recommandons vivement de parcourir tout d’abord la [Première partie : Présentation du réseau virtuel](how-to-network-security-overview.md) pour en savoir plus sur l’architecture globale. 

Consultez les autres articles de cette série :

[1. Présentation du réseau virtuel](how-to-network-security-overview.md) > [Sécurisation de l’espace de travail](how-to-secure-workspace-vnet.md) > [3. Sécurisation de l’environnement de formation](how-to-secure-training-vnet.md) > **4. Sécurisation de l’environnement d’inférence** > [5. Activation de la fonctionnalité de studio](how-to-enable-studio-virtual-network.md)

Dans cet article, vous découvrirez comment sécuriser les ressources de calcul d’inférence dans un réseau virtuel :
> [!div class="checklist"]
> - Un cluster Azure Kubernetes Service (AKS) par défaut
> - Cluster AKS privé
> - Cluster AKS avec liaison privée
> - Azure Container Instances (ACI)


## <a name="prerequisites"></a>Prérequis

+ Lisez l’article [Vue d’ensemble de la sécurité réseau](how-to-network-security-overview.md) pour comprendre les scénarios courants des réseaux virtuels et l’architecture globale des réseaux virtuels.

+ Un réseau virtuel et un sous-réseau existants à utiliser avec vos ressources de calcul.

+ Pour déployer des ressources dans un réseau virtuel ou un sous-réseau, votre compte d’utilisateur doit disposer d’autorisations pour les actions suivantes dans le contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

    - « Microsoft.Network/virtualNetworks/join/action » sur la ressource de réseau virtuel.
    - « Microsoft.Network/virtualNetworks/subnet/join/action » sur la ressource de sous-réseau virtuel.

    Pour plus d’informations sur Azure RBAC avec la mise en réseau, consultez [Rôles intégrés pour la mise en réseau](../role-based-access-control/built-in-roles.md#networking).

<a id="aksvnet"></a>

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Pour utiliser un cluster AKS dans un réseau virtuel, les exigences réseau suivantes doivent être satisfaites :

> [!div class="checklist"]
> * Remplissez les conditions requises indiquées sur la page [Configurer un réseau Azure CNI dans AKS (Azure Kubernetes Service)](../aks/configure-azure-cni.md#prerequisites).
> * L’instance AKS et le réseau virtuel doivent être dans la même région. Si vous sécurisez le ou les comptes Stockage Azure utilisés par l’espace de travail dans un réseau virtuel, ils doivent se trouver aussi dans le même réseau virtuel que l’instance AKS.


Pour ajouter AKS sur un réseau virtuel à votre espace de travail, effectuez les étapes suivantes :

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com/), puis sélectionnez votre abonnement et votre espace de travail.

1. Sélectionnez __Compute__ à gauche.

1. Sélectionnez __Clusters d'inférence__ à partir du centre, puis sélectionnez __+__ .

1. Dans la boîte de dialogue __Nouveau cluster d'inférence__, sélectionnez __Avancé__ sous __Configuration réseau__.

1. Pour configurer cette ressource de calcul afin d'utiliser un réseau virtuel, procédez comme suit :

    1. Dans la liste déroulante __Groupe de ressources__, sélectionnez le groupe de ressources qui contient le réseau virtuel.
    1. Dans la liste déroulante __Réseau virtuel__, sélectionnez le réseau virtuel qui contient le sous-réseau.
    1. Dans la liste déroulante __Sous-réseau__, sélectionnez le sous-réseau.
    1. Dans la __plage d’adresses de service Kubernetes__, entrez la plage d’adresses du service Kubernetes. Cette plage d’adresses utilise une plage d’adresses IP de notation CIDR (Classless Inter-Domain Routing) pour définir les adresses IP disponibles pour le cluster. Elle ne doit empiéter sur aucune plage d’adresses IP de sous-réseau (par exemple, 10.0.0.0/16).
    1. Dans la zone __plage d’adresses IP du service DNS Kubernetes__, entrez l’adresse IP du service DNS Kubernetes. Cette adresse IP est affectée au service DNS Kubernetes. Elle doit se situer dans la plage d’adresses du service Kubernetes (par exemple, 10.0.0.10).
    1. Dans la zone __adresse du pont Docker__, entrez l’adresse du pont Docker. Cette adresse IP est affectée au pont Docker. Elle ne doit appartenir à aucune plage d’adresses IP de sous-réseau, ni à la plage d’adresses du service Kubernetes (par exemple, 172.17.0.1/16).

   ![Azure Machine Learning : Paramètres de réseau virtuel Capacité de calcul Machine Learning](./media/how-to-enable-virtual-network/aks-virtual-network-screen.png)

1. Quand vous déployez un modèle en tant que service web sur AKS, un point de terminaison de scoring est créé pour gérer les demandes d’inférence. Vérifiez que le groupe de sécurité réseau qui contrôle le réseau virtuel dispose d’une règle de sécurité du trafic entrant activée pour l’adresse IP du point de terminaison de scoring si vous souhaitez l’appeler en dehors du réseau virtuel.

    Pour trouver l’adresse IP du point de terminaison de scoring, examinez l’URI de scoring pour le service déployé. Pour plus d’informations sur l’affichage de l’URI de scoring, consultez [Consommer un modèle déployé en tant que service web](how-to-consume-web-service.md#connection-information).

   > [!IMPORTANT]
   > Conservez les règles de trafic sortant par défaut pour le groupe de sécurité réseau. Pour plus d’informations, consultez les règles de sécurité par défaut dans [Groupes de sécurité](../virtual-network/network-security-groups-overview.md#default-security-rules).

   [![Règle de sécurité de trafic entrant](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png)](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png#lightbox)

    > [!IMPORTANT]
    > L’adresse IP indiquée dans l’image pour le point de terminaison de scoring est différente pour vos déploiements. Bien que la même adresse IP soit partagée par tous les déploiements sur un cluster AKS, chaque cluster AKS a une adresse IP différente.

Vous pouvez également utiliser le SDK Azure Machine Learning pour ajouter Azure Kubernetes Service dans un réseau virtuel. Si vous avez déjà un cluster AKS dans un réseau virtuel, attachez-le à l’espace de travail comme décrit dans [Comment déployer dans AKS](how-to-deploy-and-where.md). Le code suivant crée une instance Azure Kubernetes Service dans le sous-réseau `default` d’un réseau virtuel nommé `mynetwork` :

```python
from azureml.core.compute import ComputeTarget, AksCompute

# Create the compute configuration and set virtual network information
config = AksCompute.provisioning_configuration(location="eastus2")
config.vnet_resourcegroup_name = "mygroup"
config.vnet_name = "mynetwork"
config.subnet_name = "default"
config.service_cidr = "10.0.0.0/16"
config.dns_service_ip = "10.0.0.10"
config.docker_bridge_cidr = "172.17.0.1/16"

# Create the compute target
aks_target = ComputeTarget.create(workspace=ws,
                                  name="myaks",
                                  provisioning_configuration=config)
```

Une fois le processus de création terminé, vous pouvez effectuer une inférence, ou un scoring de modèle, sur un cluster AKS derrière un réseau virtuel. Pour plus d’informations, consultez [Guide pratique pour déployer sur AKS](how-to-deploy-and-where.md).

Pour plus d'informations sur l'utilisation du contrôle d'accès en fonction du rôle avec Kubernetes, consultez [Utiliser Azure RBAC pour l'autorisation Kubernetes](../aks/manage-azure-rbac.md).

## <a name="network-contributor-role"></a>Rôle du contributeur de réseau

> [!IMPORTANT]
> Si vous créez ou attachez un cluster AKS en fournissant un réseau virtuel que vous avez précédemment créé, vous devez accorder au principal de service ou à l’identité managée de votre cluster AKS le rôle de _contributeur de réseau_ sur le groupe de ressources contenant le réseau virtuel.
>
> Pour ajouter l’identité en tant que contributeur de réseau, procédez comme suit :

1. Pour rechercher le principal de service ou l’ID d’identité managée pour AKS, utilisez les commandes Azure CLI suivantes. Remplacez `<aks-cluster-name>` par le nom du cluster. Remplacez `<resource-group-name>` par le nom du groupe de ressources _contenant le cluster AKS_ :

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query servicePrincipalProfile.clientId
    ``` 

    Si cette commande renvoie la valeur `msi`, utilisez la commande suivante pour identifier l’ID du principal de service pour l’identité managée :

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query identity.principalId
    ```

1. Pour rechercher l’ID du groupe de ressources contenant votre réseau virtuel, utilisez la commande suivante. Remplacez `<resource-group-name>` par le nom du groupe de ressources _contenant le réseau virtuel_ :

    ```azurecli-interactive
    az group show -n <resource-group-name> --query id
    ```

1. Pour ajouter le principal de service ou l’identité managée en tant que contributeur de réseau, utilisez la commande suivante. Remplacez `<SP-or-managed-identity>` par l’ID renvoyé pour le principal de service ou l’identité managée. Remplacez `<resource-group-id>` par l’ID renvoyé pour le groupe de ressources contenant le réseau virtuel.

    ```azurecli-interactive
    az role assignment create --assignee <SP-or-managed-identity> --role 'Network Contributor' --scope <resource-group-id>
    ```
Pour plus d’informations sur l’utilisation de l’équilibreur de charge interne avec AKS, consultez [Utiliser un équilibreur de charge interne avec Azure Kubernetes Service (AKS)](../aks/internal-lb.md).

## <a name="secure-vnet-traffic"></a>Sécuriser le trafic de réseau virtuel

Il existe deux approches pour isoler le trafic vers et depuis le cluster AKS en direction du réseau virtuel :

* __Cluster AKS privé__ : Cette approche utilise Azure Private Link pour sécuriser les communications avec le cluster pour les opérations de déploiement et de gestion.
* __Équilibreur de charge AKS interne__ : Cette approche configure le point de terminaison de vos déploiements sur AKS pour utiliser une adresse IP privée dans le réseau virtuel.

> [!WARNING]
> L’équilibreur de charge interne ne fonctionne pas avec un cluster AKS qui utilise kubenet. Si vous souhaitez utiliser un équilibreur de charge interne et un cluster AKS privé en même temps, configurez votre cluster AKS privé avec Azure Container Networking Interface (CNI). Pour plus d’informations, consultez [Configurer la mise en réseau d’Azure CNI dans Azure Kubernetes Service](../aks/configure-azure-cni.md).

### <a name="private-aks-cluster"></a>Cluster AKS privé

Les clusters AKS par défaut ont un plan de contrôle (ou un serveur d’API) avec des adresses IP publiques. Vous pouvez configurer AKS pour utiliser un plan de contrôle privé en créant un cluster AKS privé. Pour plus d’informations, consultez [Créer un cluster Azure Kubernetes Service privé](../aks/private-clusters.md).

Après avoir créé le cluster AKS privé, [attachez le cluster au réseau virtuel](how-to-create-attach-kubernetes.md) à utiliser avec Azure Machine Learning.

> [!IMPORTANT]
> Avant d’utiliser un cluster AKS accessible par liaison privée avec Azure Machine Learning, vous devez ouvrir un incident de support pour activer cette fonctionnalité. Pour plus d’informations, consultez [Gérer et augmenter les quotas](how-to-manage-quotas.md#private-endpoint-and-private-dns-quota-increases).

### <a name="internal-aks-load-balancer"></a>Équilibreur de charge AKS interne

Par défaut, les déploiements AKS utilisent un [équilibreur de charge public](../aks/load-balancer-standard.md). Dans cette section, vous allez apprendre à configurer AKS pour utiliser un équilibreur de charge interne. Un équilibreur de charge interne (ou privé) est utilisé lorsque des adresses IP privées sont autorisées uniquement au niveau du serveur front-end. Les équilibreurs de charge internes sont utilisés pour équilibrer la charge du trafic au sein d’un réseau virtuel

Un équilibreur de charge privé est activé en configurant AKS pour utiliser un _équilibreur de charge interne_. 

#### <a name="enable-private-load-balancer"></a>Activer l’équilibreur de charge privé

> [!IMPORTANT]
> Vous ne pouvez pas activer une adresse IP privée lors de la création du cluster Azure Kubernetes Service dans le studio Azure Machine Learning. Vous pouvez en créer un avec un équilibreur de charge interne lors de l’utilisation du SDK Python ou de l’extension d’Azure CLI pour le Machine Learning.

Les exemples suivants montrent comment __créer un cluster AKS avec une adresse IP privée/un équilibreur de charge interne__ à l’aide du SDK et de l’interface de ligne de commande :

# <a name="python"></a>[Python](#tab/python)

```python
import azureml.core
from azureml.core.compute import AksCompute, ComputeTarget

# Verify that cluster does not exist already
try:
    aks_target = AksCompute(workspace=ws, name=aks_cluster_name)
    print("Found existing aks cluster")

except:
    print("Creating new aks cluster")

    # Subnet to use for AKS
    subnet_name = "default"
    # Create AKS configuration
    prov_config=AksCompute.provisioning_configuration(load_balancer_type="InternalLoadBalancer")
    # Set info for existing virtual network to create the cluster in
    prov_config.vnet_resourcegroup_name = "myvnetresourcegroup"
    prov_config.vnet_name = "myvnetname"
    prov_config.service_cidr = "10.0.0.0/16"
    prov_config.dns_service_ip = "10.0.0.10"
    prov_config.subnet_name = subnet_name
    prov_config.docker_bridge_cidr = "172.17.0.1/16"

    # Create compute target
    aks_target = ComputeTarget.create(workspace = ws, name = "myaks", provisioning_configuration = prov_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks --load-balancer-type InternalLoadBalancer
```

> [!IMPORTANT]
> L’interface CLI vous permet uniquement de créer un cluster AKS avec un équilibreur de charge interne. Il n’existe aucune commande AZ ML pour mettre à niveau un cluster existant afin d’utiliser un équilibreur de charge interne.

Pour plus d’informations, consultez la référence [AZ ml computetarget Create AKS](/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-create-aks).

---

Lorsque vous __attachez un cluster existant__ à votre espace de travail, vous devez attendre la fin de l’opération d’attachement pour configurer l’équilibreur de charge. Pour plus d’informations sur l’attachement d’un cluster, consultez [Attacher un cluster AKS existant](how-to-create-attach-kubernetes.md).

Une fois le cluster existant attaché, vous pouvez le mettre à jour pour utiliser un équilibreur de charge interne/une adresse IP privée :

```python
import azureml.core
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute import AksCompute

# ws = workspace object. Creation not shown in this snippet
aks_target = AksCompute(ws,"myaks")

# Change to the name of the subnet that contains AKS
subnet_name = "default"
# Update AKS configuration to use an internal load balancer
update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
aks_target.update(update_config)
# Wait for the operation to complete
aks_target.wait_for_completion(show_output = True)
```

## <a name="enable-azure-container-instances-aci"></a>Activer Azure Container Instances (ACI)

Les instances de conteneur Azure (ACI) sont créées dynamiquement lors du déploiement d’un modèle. Pour permettre à Azure Machine Learning de créer un réseau ACI à l’intérieur du réseau virtuel, vous devez activer la __délégation de sous-réseau__ pour le sous-réseau utilisé par le déploiement.

> [!WARNING]
> Lorsque vous utilisez Azure Container Instances dans un réseau virtuel, le réseau virtuel doit être :
> * Dans le même groupe de ressources que votre espace de travail Azure Machine Learning.
> * Si votre espace de travail comporte un __point de terminaison privé__, le réseau virtuel utilisé pour Azure Container Instances doit être identique à celui utilisé par le point de terminaison privé de l’espace de travail.
>
> Lorsque vous utilisez Azure Container Instances à l’intérieur du réseau virtuel, l’instance Azure Container Registry (ACR) de votre espace de travail ne doit pas se trouver dans le réseau virtuel.

Pour utiliser ACI sur un réseau virtuel à votre espace de travail, effectuez les étapes suivantes :

1. Pour activer la délégation de sous-réseau sur votre réseau virtuel, utilisez les informations de l’article [Ajouter ou supprimer une délégation de sous-réseau](../virtual-network/manage-subnet-delegation.md). Vous pouvez activer la délégation lors de la création d’un réseau virtuel ou l’ajouter à un réseau existant.

    > [!IMPORTANT]
    > Lorsque vous activez la délégation, utilisez `Microsoft.ContainerInstance/containerGroups` comme valeur __Déléguer le sous-réseau à un service__.

2. Déployez le modèle à l’aide de [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?preserve-view=true&view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none--vnet-name-none--subnet-name-none-&preserve-view=true), en utilisant les paramètres `vnet_name` et `subnet_name`. Appliquez ces paramètres au nom du réseau virtuel et au sous-réseau où vous avez activé la délégation.

## <a name="limit-outbound-connectivity-from-the-virtual-network"></a> Limitez la connectivité sortante à partir du réseau virtuel

Si vous ne souhaitez pas utiliser les règles de trafic sortant par défaut et souhaitez limiter l’accès sortant de votre réseau virtuel, vous devez autoriser l’accès à Azure Container Registry. Par exemple, assurez-vous que vos groupes de sécurité réseau (NSG) contiennent une règle qui autorise l’accès à l’étiquette de service __AzureContainerRegistry.RegionName__, où {RegionName} correspond au nom d’une région Azure.

## <a name="next-steps"></a>Étapes suivantes

Cet article est le troisième d’une série en quatre parties sur les réseaux virtuels. Consultez les autres articles pour découvrir comment sécuriser un réseau virtuel :

* [Partie 1 : Vue d’ensemble des réseaux virtuels](how-to-network-security-overview.md)
* [Partie 2 : Sécuriser les ressources d’espace de travail](how-to-secure-workspace-vnet.md)
* [Partie 3 : Sécuriser l’environnement de formation](how-to-secure-training-vnet.md)
* [Partie 5 : Activer la fonctionnalité Studio](how-to-enable-studio-virtual-network.md)