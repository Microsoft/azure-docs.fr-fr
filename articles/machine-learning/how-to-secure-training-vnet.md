---
title: Sécuriser les environnements d’entraînement à l’aide de réseaux virtuels
titleSuffix: Azure Machine Learning
description: Utilisez un réseau virtuel Azure isolé pour sécuriser votre environnement d’entraînement Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 07/16/2020
ms.custom: contperf-fy20q4, tracking-python, contperf-fy21q1
ms.openlocfilehash: 131feaf6ff01659b7d126604a5d081275e64508f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029564"
---
# <a name="secure-an-azure-machine-learning-training-environment-with-virtual-networks"></a>Sécuriser un environnement d’entraînement Azure Machine Learning à l’aide de réseaux virtuels

Dans cet article, vous allez apprendre à sécuriser les environnements d’entraînement à l’aide d’un réseau virtuel dans Azure Machine Learning.

Cet article est le troisième d’une série de cinq qui vous guide à travers le processus de sécurisation d’un workflow Azure Machine Learning. Nous vous recommandons vivement de parcourir tout d’abord la [Première partie : Présentation du réseau virtuel](how-to-network-security-overview.md) pour en savoir plus sur l’architecture globale. 

Consultez les autres articles de cette série :

[1. Présentation du réseau virtuel](how-to-network-security-overview.md) > [Sécurisation de l’espace de travail](how-to-secure-workspace-vnet.md) > **3. Sécurisation de l’environnement d’entraînement** > [4. Sécurisation de l’environnement d’inférence](how-to-secure-inferencing-vnet.md)  > [5. Activation de la fonctionnalité de studio](how-to-enable-studio-virtual-network.md)

Dans cet article, vous découvrirez comment sécuriser les ressources de calcul d’entraînement dans un réseau virtuel :
> [!div class="checklist"]
> - Cluster de calcul Azure Machine Learning
> - Instance de calcul Azure Machine Learning
> - Azure Databricks
> - Machine virtuelle
> - Cluster HDInsight

## <a name="prerequisites"></a>Prérequis

+ Lisez l’article [Vue d’ensemble de la sécurité réseau](how-to-network-security-overview.md) pour comprendre les scénarios courants des réseaux virtuels et l’architecture globale des réseaux virtuels.

+ Un réseau virtuel et un sous-réseau existants à utiliser avec vos ressources de calcul.

+ Pour déployer des ressources dans un réseau virtuel ou un sous-réseau, votre compte d’utilisateur doit disposer d’autorisations pour les actions suivantes dans le contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

    - « Microsoft.Network/virtualNetworks/join/action » sur la ressource de réseau virtuel.
    - « Microsoft.Network/virtualNetworks/subnet/join/action » sur la ressource de sous-réseau virtuel.

    Pour plus d’informations sur Azure RBAC avec la mise en réseau, consultez [Rôles intégrés pour la mise en réseau](../role-based-access-control/built-in-roles.md#networking)


## <a name="compute-clusters--instances"></a><a name="compute-instance"></a>Clusters et instances de calcul 

Pour utiliser une [__cible de calcul__ Azure Machine Learning gérée](concept-compute-target.md#azure-machine-learning-compute-managed) ou une [__instance de calcul__ Azure Machine Learning](concept-compute-instance.md) dans un réseau virtuel, le réseau doit réunir les conditions suivantes :

> [!div class="checklist"]
> * Le réseau virtuel doit être dans les mêmes abonnement et région que l’espace de travail Azure Machine Learning.
> * Le sous-réseau spécifié pour l’instance ou le cluster de calcul doit avoir suffisamment d’adresses IP non attribuées pour toutes les machines virtuelles ciblées. Si le sous-réseau n’a pas suffisamment d’adresses IP non attribuées, le cluster de calcul est alloué partiellement.
> * Vérifiez si vos stratégies ou verrous de sécurité sur l’abonnement ou le groupe de ressources du réseau virtuel restreignent les autorisations pour gérer le réseau virtuel. Si vous souhaitez sécuriser le réseau virtuel en limitant le trafic, laissez certains ports ouverts pour le service Capacité de calcul. Pour plus d’informations, voir la section [Ports requis](#mlcports).
> * Si vous vous apprêtez à placer plusieurs instances ou clusters de calcul sur un réseau virtuel, vous devrez peut-être demander une augmentation du quota pour une ou plusieurs de vos ressources.
> * Si le ou les comptes de stockage Azure pour l’espace de travail sont également sécurisés dans un réseau virtuel, ils doivent se trouver dans le même réseau virtuel que l’instance ou le cluster de capacité de calcul Azure Machine Learning. 
> * Pour que la fonctionnalité d’instance de calcul Jupyter fonctionne, vérifiez que la communication avec le socket web n’est pas désactivée. Vérifiez que votre réseau autorise les connexions WebSocket à *.instances.azureml.net et *.instances.azureml.ms. 
> * Lorsque l’instance de calcul est déployée dans un espace de travail privé, elle n’est accessible qu’à partir d’un réseau virtuel. Si vous utilisez un DNS ou un fichier d’hôtes personnalisé, ajoutez une entrée pour `<instance-name>.<region>.instances.azureml.ms` avec l’adresse IP privée du point de terminaison privé de l’espace de travail. Pour plus d’informations, consultez l’article [DNS personnalisé](./how-to-custom-dns.md).
    
> [!TIP]
> La capacité de calcul ou le cluster Machine Learning alloue automatiquement des ressources réseau supplémentaires au __groupe de ressources qui contient le réseau virtuel__. Pour chaque instance ou cluster de calcul, le service alloue les ressources suivantes :
> 
> * Un seul groupe de sécurité réseau
> * Une seule adresse IP publique
> * Un seul équilibreur de charge
> 
> Dans le cas de clusters, ces ressources sont supprimées (et recréées) chaque fois que le cluster n’a plus aucun nœud. Toutefois, pour une instance, les ressources sont conservées jusqu’à ce que l’instance soit complètement supprimée (l’arrêt ne supprime pas les ressources). 
> Ces ressources sont limitées par les [quotas de ressources](../azure-resource-manager/management/azure-subscription-service-limits.md) de l’abonnement.


### <a name="required-ports"></a><a id="mlcports"></a> Ports requis

Si vous envisagez de sécuriser le réseau virtuel en restreignant le trafic réseau vers/à partir de l’Internet public, vous devez autoriser les communications entrantes à partir du service Azure Batch.

Le service Batch ajoute des groupes de sécurité réseau (NSG) au niveau des interfaces réseau (NIC) qui sont attachées aux machines virtuelles. Ces groupes de sécurité réseau configurent automatiquement des règles de trafic entrant et sortant pour autoriser le trafic suivant :

- Trafic TCP entrant sur les ports 29876 et 29877 à partir d’une __balise de service__ de __BatchNodeManagement__.

    ![Règle de trafic entrant qui utilise la balise de service BatchNodeManagement](./media/how-to-enable-virtual-network/batchnodemanagement-service-tag.png)

- (facultatif) Trafic TCP entrant sur le port 22 pour autoriser l’accès à distance. Utilisez ce port uniquement si vous souhaitez vous connecter à l’aide du protocole SSH sur l’adresse IP publique.

- Le trafic sortant sur n’importe quel port vers le réseau virtuel.

- Le trafic sortant sur n’importe quel port vers internet.

- Le trafic TCP entrant de l’instance de calcul sur le port 44224 à partir d’une __Balise de service__ __AzureMachineLearning__.

> [!IMPORTANT]
> Soyez prudent si vous modifiez ou ajoutez des règles de trafic entrant ou sortant dans des groupes de sécurité réseau configurés par Batch. Si un groupe de sécurité réseau bloque la communication vers les nœuds de calcul, le service Capacité de calcul définit l’état de ces nœuds sur « inutilisable ».
>
> Vous n’avez pas besoin de spécifier des groupes de sécurité réseau au niveau du sous-réseau, car le service Azure Batch configure ses propres groupes de sécurité réseau. Toutefois, si le sous-réseau qui contient la capacité de calcul Azure Machine Learning est associé à des groupes de sécurité réseau ou à un pare-feu, vous devez également autoriser le trafic listé plus haut.

La configuration de la règle de groupe de sécurité réseau dans le Portail Azure est illustrée dans les images suivantes :

:::image type="content" source="./media/how-to-enable-virtual-network/amlcompute-virtual-network-inbound.png" alt-text="Les règles de groupe de sécurité réseau de trafic entrant pour Capacité de calcul Machine Learning" border="true":::



![Règles de groupe de sécurité réseau de trafic entrant pour Capacité de calcul Machine Learning](./media/how-to-enable-virtual-network/experimentation-virtual-network-outbound.png)

### <a name="limit-outbound-connectivity-from-the-virtual-network"></a><a id="limiting-outbound-from-vnet"></a> Limitez la connectivité sortante à partir du réseau virtuel

Si vous ne souhaitez pas utiliser les règles de trafic sortant par défaut et souhaitez limiter l’accès sortant de votre réseau virtuel, effectuez les étapes suivantes :

- Refusez la connexion internet sortante à l’aide des règles NSG.

- Pour une __instance de calcul__ ou un __cluster de calcul__, limitez le trafic sortant aux éléments suivants :
   - Stockage Azure, en utilisant la __balise de service__ __Storage.RegionName__. Où `{RegionName}` est le nom d’une région Azure.
   - Azure Container Registry, en utilisant la __balise de service__ __AzureContainerRegistry.RegionName__. Où `{RegionName}` est le nom d’une région Azure.
   - Azure Machine Learning, à l’aide de la __balise du service__ de __AzureMachineLearning__
   - Azure Resource Manager, à l’aide de la __balise de service__ de __AzureResourceManager__
   - Azure Active Directory, à l’aide de la __balise de service__ de __AzureActiveDirectory__

La configuration de la règle de groupe de sécurité réseau dans le Portail Azure est illustrée dans l’image suivante :

[![Les règles des groupes de sécurité réseau de trafic sortant pour Capacité de calcul](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png)](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png#lightbox)

> [!NOTE]
> Si vous prévoyez d’utiliser des images Docker par défaut fournies par Microsoft et que vous activez les dépendances gérées par l’utilisateur, vous devez également utiliser les __étiquettes de service__ suivantes :
>
> * __MicrosoftContainerRegistry__
> * __AzureFrontDoor.FirstParty__
>
> Cette configuration est nécessaire si les scripts d’apprentissage contiennent du code de ce type :
>
> __Entraînement RunConfig__
> ```python
> # create a new runconfig object
> run_config = RunConfiguration()
> 
> # configure Docker 
> run_config.environment.docker.enabled = True
> # For GPU, use DEFAULT_GPU_IMAGE
> run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE 
> run_config.environment.python.user_managed_dependencies = True
> ```
>
> __Entraînement Estimator__
> ```python
> est = Estimator(source_directory='.',
>                 script_params=script_params,
>                 compute_target='local',
>                 entry_script='dummy_train.py',
>                 user_managed=True)
> run = exp.submit(est)
> ```

### <a name="forced-tunneling"></a>Tunneling forcé

Si vous utilisez le [tunneling forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md) avec le calcul Azure Machine Learning, vous devez autoriser la communication avec l’Internet public depuis le sous-réseau contenant la ressource de calcul. Cette communication permet de planifier les tâches et d’accéder au Stockage Azure.

Pour ce faire, il y a deux manières de procéder :

* Utilisez un [NAT de réseau virtuel](../virtual-network/nat-overview.md). Une passerelle NAT fournit une connectivité Internet sortante pour un ou plusieurs sous-réseaux dans votre réseau virtuel. Pour en savoir plus, consultez [Conception de réseaux virtuels avec des ressources de passerelle NAT](../virtual-network/nat-gateway-resource.md).

* Ajoutez des [itinéraires définis par l’utilisateur (UDR)](../virtual-network/virtual-networks-udr-overview.md) au sous-réseau qui contient la ressource de calcul. Établissez un UDR pour chaque adresse IP utilisée par le service Azure Batch dans la région où se trouvent vos ressources. Ces UDR autorisent le service Batch à communiquer avec les nœuds de calcul pour la planification des tâches. Ajoutez également l’adresse IP du service Azure Machine Learning dans lequel les ressources existent, car cela est nécessaire pour l’accès aux instances de calcul. Pour obtenir la liste des adresses IP du service Batch et du service Azure Machine Learning, utilisez l’une des méthodes suivantes :

    * Téléchargez les [plages d’adresses IP Azure et les balises de service](https://www.microsoft.com/download/details.aspx?id=56519), et recherchez `BatchNodeManagement.<region>` et `AzureMachineLearning.<region>` dans le fichier, où `<region>` est votre région Azure.

    * Utilisez [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) pour télécharger les informations. L’exemple suivant télécharge les informations d’adresse IP et filtre les informations pour la région USA Est 2 :

        ```azurecli-interactive
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'Batch')] | [?properties.region=='eastus2']"
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='eastus2']"
        ```

        > [!TIP]
        > Si vous utilisez les régions USA Virginie, USA Arizona ou les régions Chine Est 2, ces commandes ne retournent aucune adresse IP. Utilisez plutôt un des liens suivants pour télécharger une liste d’adresses IP :
        >
        > * [Plages d’adresses IP et étiquettes de service pour Azure Government](https://www.microsoft.com/download/details.aspx?id=57063)
        > * [Plages d’adresses IP et étiquettes de service pour Azure Chine](https://www.microsoft.com//download/details.aspx?id=57062)
    
    Lorsque vous ajoutez les UDR, définissez l’itinéraire pour chaque préfixe d’adresse IP Batch connexe et définissez __Type de tronçon suivant__ sur __Internet__. L’illustration suivante propose un exemple de cet UDR dans le portail Azure :

    ![Exemple de UDR pour un préfixe d’adresse](./media/how-to-enable-virtual-network/user-defined-route.png)

    > [!IMPORTANT]
    > Les adresses IP peuvent changer au fil du temps.

    Outre les UDR que vous définissez, le trafic sortant vers Stockage Azure doit être autorisé via votre appliance de réseau local. Plus précisément, les URL pour ce trafic présentent les formes suivantes : `<account>.table.core.windows.net`, `<account>.queue.core.windows.net`, et `<account>.blob.core.windows.net`. 

    Pour plus d’informations, consultez l’article [Créer un pool Azure Batch dans un réseau virtuel](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).


### <a name="create-a-compute-cluster-in-a-virtual-network"></a>Créer un cluster de calcul dans un réseau virtuel

Pour créer un cluster Capacité de calcul Machine Learning, effectuez les étapes suivantes :

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com/), puis sélectionnez votre abonnement et votre espace de travail.

1. Sélectionnez __Compute__ à gauche.

1. Sélectionnez __Clusters d'entraînement__ à partir du centre, puis sélectionnez __+__ .

1. Dans la boîte de dialogue __Nouveau cluster d'entraînement__, développez la section __Paramètres avancés__.

1. Pour configurer cette ressource de calcul afin d'utiliser un réseau virtuel, effectuez les actions suivantes dans la section __Configurer le réseau virtuel__ :

    1. Dans la liste déroulante __Groupe de ressources__, sélectionnez le groupe de ressources qui contient le réseau virtuel.
    1. Dans la liste déroulante __Réseau virtuel__, sélectionnez le réseau virtuel qui contient le sous-réseau.
    1. Dans la liste déroulante __Sous-réseau__, sélectionnez le sous-réseau à utiliser.

   ![Les paramètres de réseau virtuel Capacité de calcul Machine Learning](./media/how-to-enable-virtual-network/amlcompute-virtual-network-screen.png)

Vous pouvez également créer un cluster Capacité de calcul Machine Learning à l’aide du SDK Azure Machine Learning. Le code suivant crée un cluster Capacité de calcul dans le sous-réseau `default` d’un réseau virtuel nommé `mynetwork` :

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# The Azure virtual network name, subnet, and resource group
vnet_name = 'mynetwork'
subnet_name = 'default'
vnet_resourcegroup_name = 'mygroup'

# Choose a name for your CPU cluster
cpu_cluster_name = "cpucluster"

# Verify that cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print("Found existing cpucluster")
except ComputeTargetException:
    print("Creating new cpucluster")

    # Specify the configuration for the new cluster
    compute_config = AmlCompute.provisioning_configuration(vm_size="STANDARD_D2_V2",
                                                           min_nodes=0,
                                                           max_nodes=4,
                                                           vnet_resourcegroup_name=vnet_resourcegroup_name,
                                                           vnet_name=vnet_name,
                                                           subnet_name=subnet_name)

    # Create the cluster with the specified name and configuration
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

    # Wait for the cluster to be completed, show the output log
    cpu_cluster.wait_for_completion(show_output=True)
```

Une fois le processus de création terminé, vous pouvez entraîner votre modèle en utilisant le cluster dans le cadre d’une expérience. Pour plus d’informations, consultez [Sélectionner et utiliser une cible de calcul pour l’entraînement](how-to-set-up-training-targets.md).

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="access-data-in-a-compute-instance-notebook"></a>Accéder aux données d’un notebook d’instance de capacité de calcul

Si vous utilisez des notebooks sur une instance de capacité de calcul Azure, vous devez vérifier que votre notebook s’exécute sur une ressource de calcul derrière le même réseau virtuel et le même sous-réseau que vos données. 

Vous devez configurer votre instance de capacité de calcul pour qu’elle se trouve sur le même réseau virtuel lors de la création sous **Paramètres avancés** > **Configurer le réseau virtuel**. Vous ne pouvez pas ajouter une instance de capacité de calcul existante à un réseau virtuel.

## <a name="azure-databricks"></a>Azure Databricks

Vous devez remplir les conditions suivantes pour pouvoir utiliser Azure Databricks dans un réseau virtuel avec votre espace de travail :

> [!div class="checklist"]
> * Le réseau virtuel doit être dans les mêmes abonnement et région que l’espace de travail Azure Machine Learning.
> * Si le ou les comptes de stockage Azure pour l’espace de travail sont également sécurisés dans un réseau virtuel, ils doivent se trouver dans le même réseau virtuel que le cluster Azure Databricks.
> * Outre les sous-réseaux __databricks-private__ et __databricks-public__ utilisés par Azure Databricks, le sous-réseau __default__ créé pour le réseau virtuel est également requis.

Pour obtenir des informations spécifiques sur l’utilisation d’Azure Databricks avec un réseau virtuel, consultez [Déployer des Azure Databricks dans votre réseau virtuel Azure](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-inject.html).

<a id="vmorhdi"></a>

## <a name="virtual-machine-or-hdinsight-cluster"></a>Machine virtuelle ou cluster HDInsight

> [!IMPORTANT]
> Azure Machine Learning prend uniquement en charge les machines virtuelles exécutant Ubuntu.

Dans cette section, vous allez apprendre à utiliser une machine virtuelle ou un cluster Azure HDInsight dans un réseau virtuel avec votre espace de travail.

### <a name="create-the-vm-or-hdinsight-cluster"></a>Créer la machine virtuelle ou le cluster HDInsight

Créez une machine virtuelle ou un cluster HDInsight à l’aide du portail Azure ou de l’interface de ligne de commande Azure, et placez le cluster dans un réseau virtuel Azure. Pour plus d’informations, consultez les articles suivants :
* [Créer et gérer des réseaux virtuels Azure pour des machines virtuelles Linux](../virtual-machines/linux/tutorial-virtual-network.md)

* [Étendre HDInsight à l’aide d’un réseau virtuel Azure](../hdinsight/hdinsight-plan-virtual-network-deployment.md)

### <a name="configure-network-ports"></a>Configurer des ports réseau 

Autorisez Azure Machine Learning à communiquer avec le port SSH sur la machine virtuelle ou le cluster, configurez une entrée source pour le groupe de sécurité réseau. Le port SSH est généralement le port 22. Pour autoriser le trafic provenant de cette source, effectuez les actions suivantes :

1. Dans la liste déroulante __Source__, sélectionnez __Balise de service__.

1. Dans la liste déroulante __Balise de service source__, sélectionnez __AzureMachineLearning__.

    ![Règles de trafic entrant pour effectuer des expériences sur une machine virtuelle ou un cluster HDInsight à l’intérieur d’un réseau virtuel](./media/how-to-enable-virtual-network/experimentation-virtual-network-inbound.png)

1. Dans la liste déroulante __Plages de port source__, sélectionnez __*__ .

1. Dans la liste déroulante __Destination__, sélectionnez __Tous__.

1. Dans la liste déroulante __Plages de port de destination__, sélectionnez __22__.

1. Sous __Protocole__, sélectionnez __Tous__.

1. Sous __Action__, sélectionnez __Autoriser__.

Conservez les règles de trafic sortant par défaut pour le groupe de sécurité réseau. Pour plus d’informations, consultez les règles de sécurité par défaut dans [Groupes de sécurité](../virtual-network/network-security-groups-overview.md#default-security-rules).

Si vous ne souhaitez pas utiliser les règles de trafic sortant par défaut et souhaitez limiter l’accès sortant de votre réseau virtuel, consultez la section [Limiter la connectivité sortante à partir du réseau virtuel](#limiting-outbound-from-vnet).

### <a name="attach-the-vm-or-hdinsight-cluster"></a>Joindre la machine virtuelle ou le cluster HDInsight

Attachez la machine virtuelle ou le cluster HDInsight à votre espace de travail Azure Machine Learning. Pour plus d’informations, consultez [Configurer des cibles de calcul pour l’entraînement des modèles](how-to-set-up-training-targets.md).

## <a name="next-steps"></a>Étapes suivantes

Cet article est le troisième d’une série en quatre parties sur les réseaux virtuels. Consultez les autres articles pour découvrir comment sécuriser un réseau virtuel :

* [Partie 1 : Vue d’ensemble des réseaux virtuels](how-to-network-security-overview.md)
* [Partie 2 : Sécuriser les ressources d’espace de travail](how-to-secure-workspace-vnet.md)
* [Partie 4 : Sécuriser l’environnement d’inférence](how-to-secure-inferencing-vnet.md)
* [Partie 5 : Activer la fonctionnalité Studio](how-to-enable-studio-virtual-network.md)