---
title: Utiliser un pare-feu
titleSuffix: Azure Machine Learning
description: Contrôler l’accès aux espaces de travail Azure Machine Learning avec les pare-feu Azure. En savoir plus sur les hôtes que vous devez autoriser via le pare-feu.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 11/18/2020
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 8560acd9c5a11004c5144441d395863c8b85edba
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461405"
---
# <a name="use-workspace-behind-a-firewall-for-azure-machine-learning"></a>Utiliser l’espace de travail derrière un Pare-feu pour Azure Machine Learning

Dans cet article, découvrez comment configurer Pare-feu Azure pour contrôler l’accès à votre espace de travail Azure Machine Learning et à l’Internet public. Pour en savoir plus sur la sécurisation d’Azure Machine Learning, consultez [Sécurité de l’entreprise pour Azure Machine Learning](concept-enterprise-security.md)

## <a name="azure-firewall"></a>Pare-feu Azure

Lorsque vous utilisez le pare-feu Azure, utilisez __DNAT (Destination Network Address Translation )__ afin de créer des règles NAT pour le trafic entrant. Pour le trafic sortant, créez des règles de type __réseau__ et/ou __application__. Ces regroupements de règles sont décrits plus en détail dans [Quels sont les concepts de Pare-feu Azure ?](../firewall/firewall-faq.md#what-are-some-azure-firewall-concepts).

### <a name="inbound-configuration"></a>Configuration entrante

Si vous utilisez une __instance de calcul__ Azure Machine Learning ou un __cluster de calcul__, ajoutez des [itinéraires définis par l’utilisateur (UDR)](../virtual-network/virtual-networks-udr-overview.md) pour le sous-réseau contenant les ressources Azure Machine Learning. Cet itinéraire force le trafic __depuis__ les adresses IP des ressources `BatchNodeManagement` et `AzureMachineLearning` vers l’adresse IP publique de votre instance de calcul et de votre cluster de calcul.

Ces UDR autorisent le service Batch à communiquer avec les nœuds de calcul pour la planification des tâches. Ajoutez également l’adresse IP du service Azure Machine Learning dans lequel les ressources existent, car cela est nécessaire pour l’accès aux instances de calcul. Pour obtenir la liste des adresses IP du service Batch et du service Azure Machine Learning, utilisez l’une des méthodes suivantes :

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

Pour plus d’informations, consultez l’article [Créer un pool Azure Batch dans un réseau virtuel](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).

### <a name="outbound-configuration"></a>Configuration sortante

1. Ajoutez des __règles de réseau__, permettant le trafic __vers__ et __depuis__ les étiquettes de service suivantes :

    * AzureActiveDirectory
    * AzureMachineLearning
    * AzureResourceManager
    * Storage.region
    * KeyVault.region
    * ContainerRegistry.region

    Si vous prévoyez d’utiliser les images Docker par défaut fournies par Microsoft et que vous activez les dépendances gérées par l’utilisateur, vous devez également ajouter les étiquettes de service suivantes :

    * MicrosoftContainerRegistry.region
    * AzureFrontDoor.FirstParty

    Pour les entrées contenant `region`, remplacez cette valeur par la région Azure que vous utilisez. Par exemple : `keyvault.westus`.

    Pour le __protocole__, sélectionnez `TCP`. Pour les __ports__ source et de destination, sélectionnez `*`.

1. Ajoutez des __règles d’application__ pour les hôtes suivants :

    > [!NOTE]
    > Il ne s’agit pas d’une liste complète des ordinateurs hôtes requis pour toutes les ressources Python sur Internet, uniquement celles les plus couramment utilisées. Par exemple, si vous avez besoin d’accéder à un référentiel GitHub ou à un autre hôte, vous devez identifier et ajouter les ordinateurs hôtes requis pour ce scénario.

    | **Nom d’hôte** | **Objectif** |
    | ---- | ---- |
    | **anaconda.com**</br>**\*.anaconda.com** | Utilisé pour installer les packages par défaut. |
    | **\*.anaconda.org** | Utilisé pour récupérer les données des dépôts. |
    | **pypi.org** | Utilisé pour lister les dépendances de l’index par défaut, le cas échéant, et si l’index n’a pas été remplacé par les paramètres utilisateur. Si l’index a été remplacé, vous devez également autoriser **\*.pythonhosted.org**. |
    | **cloud.r-project.org** | Utilisé lors de l’installation des packages CRAN pour le développement R. |
    | **\*pytorch.org** | Utilisé par certains exemples basés sur PyTorch. |
    | **\*.tensorflow.org** | Utilisé par certains exemples basés sur Tensorflow. |

    Pour __Protocol:Port__, sélectionnez __http, https__.

    Pour plus d’informations sur la configuration des règles d’application, consultez [Déployer et configurer le pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md#configure-an-application-rule).

1. Pour restreindre l’accès aux modèles déployés sur Azure Kubernetes Service (AKS), consultez [Limiter le trafic de sortie dans Azure Kubernetes Service (AKS)](../aks/limit-egress-traffic.md).

## <a name="other-firewalls"></a>Autres pare-feu

Les instructions de cette section sont génériques, car chaque pare-feu dispose de sa propre terminologie et de configurations spécifiques. Si vous avez des questions sur la façon d’autoriser la communication via votre pare-feu, consultez la documentation du pare-feu que vous utilisez.

S’il n’est pas configuré correctement, le pare-feu peut causer des problèmes lors de l’utilisation de votre espace de travail. Un grand nombre de noms d’hôtes sont utilisés à la fois par l’espace de travail Azure Machine Learning. Les sections suivantes répertorient les hôtes requis pour Azure Machine Learning.

### <a name="microsoft-hosts"></a>Hôtes Microsoft

Les hôtes de cette section sont détenus par Microsoft et fournissent les services nécessaires au bon fonctionnement de votre espace de travail. Les tableaux suivants répertorient les noms d’hôtes des régions Azure (public), Azure Government et Azure China 21Vianet.

**Hôtes Azure généraux**

| **Obligatoire pour** | **Azure public** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Azure Active Directory | login.microsoftonline.com | login.microsoftonline.us | .chinacloudapi.cn |
| Portail Azure | management.azure.com | management.azure.us | management.azure.cn |

**Hôtes Azure Machine Learning**

| **Obligatoire pour** | **Azure public** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Azure Machine Learning Studio | ml.azure.com | ml.azure.us | studio.ml.azure.cn |
| API |\*.azureml.ms | \*.ml.azure.us | \*.ml.azure.cn |
| Expérimentation, historique, Hyperdrive, étiquetage | \*.experiments.azureml.net | \*.ml.azure.us | \*.ml.azure.cn |
| La gestion des modèles | \*.modelmanagement.azureml.net | \*.ml.azure.us | \*.ml.azure.cn |
| Pipeline | \*.aether.ms | \*.ml.azure.us | \*.ml.azure.cn |
| Designer (studio service) | \*.studioservice.azureml.com | \*.ml.azure.us | \*.ml.azure.cn |
| Notebook intégré | \*.notebooks.azure.net | \*.notebooks.usgovcloudapi.net |\*.notebooks.chinacloudapi.cn |
| Notebook intégré | \*.file.core.windows.net | \*.file.core.usgovcloudapi.net | \*.file.core.chinacloudapi.cn |
| Notebook intégré | \*.dfs.core.windows.net | \*.dfs.core.usgovcloudapi.net | \*.dfs.core.chinacloudapi.cn |
| Notebook intégré | \*.blob.core.windows.net | \*.blob.core.usgovcloudapi.net | \*.blob.core.chinacloudapi.cn |
| Notebook intégré | graph.microsoft.com | graph.microsoft.us | graph.chinacloudapi.cn |
| Notebook intégré | \*.aznbcontent.net |  | |

**Hôtes d’instance de calcul et de cluster de calcul Azure Machine Learning**

| **Obligatoire pour** | **Azure public** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Cluster/instance de calcul | \*.batchai.core.windows.net | \*.batchai.core.usgovcloudapi.net |\*.batchai.ml.azure.cn |
| Instance de calcul | \*.instances.azureml.net | \*.instances.azureml.us | \*.instances.azureml.cn |
| Instance de calcul | \*.instances.azureml.ms |  |  |

**Ressources associées utilisées par Azure Machine Learning**

| **Obligatoire pour** | **Azure public** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Compte Stockage Azure | core.windows.net | core.usgovcloudapi.net | core.chinacloudapi.cn |
| Azure Key Vault | vault.azure.net | vault.usgovcloudapi.net | vault.azure.cn |
| Azure Container Registry | azurecr.io | azurecr.us | azurecr.cn |
| Registre de conteneurs Microsoft | mcr.microsoft.com | mcr.microsoft.com | mcr.microsoft.com |


> [!TIP]
> Si vous envisagez d’utiliser une identité fédérée, suivez les instructions de l’article [Bonnes pratiques pour sécuriser les services de fédération Active Directory (AD FS)](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs).

En outre, utilisez les informations du [tunneling forcé](how-to-secure-training-vnet.md#forced-tunneling) afin d’ajouter des adresses IP pour `BatchNodeManagement` et `AzureMachineLearning`.

Pour plus d’informations sur la restriction de l’accès aux modèles déployés sur Azure Kubernetes Service (AKS), consultez [Limiter le trafic de sortie dans Azure Kubernetes Service (AKS)](../aks/limit-egress-traffic.md).

### <a name="python-hosts"></a>Hôtes Python

Les hôtes de cette section sont utilisés pour installer les packages Python. Ils sont requis lors du développement, de l’entraînement et du déploiement. 

> [!NOTE]
> Il ne s’agit pas d’une liste complète des ordinateurs hôtes requis pour toutes les ressources Python sur Internet, uniquement celles les plus couramment utilisées. Par exemple, si vous avez besoin d’accéder à un référentiel GitHub ou à un autre hôte, vous devez identifier et ajouter les ordinateurs hôtes requis pour ce scénario.

| **Nom d’hôte** | **Objectif** |
| ---- | ---- |
| **anaconda.com**</br>**\*.anaconda.com** | Utilisé pour installer les packages par défaut. |
| **\*.anaconda.org** | Utilisé pour récupérer les données des dépôts. |
| **pypi.org** | Utilisé pour lister les dépendances de l’index par défaut, le cas échéant, et si l’index n’a pas été remplacé par les paramètres utilisateur. Si l’index a été remplacé, vous devez également autoriser **\*.pythonhosted.org**. |
| **\*pytorch.org** | Utilisé par certains exemples basés sur PyTorch. |
| **\*.tensorflow.org** | Utilisé par certains exemples basés sur Tensorflow. |

### <a name="r-hosts"></a>Hôtes R

Les hôtes de cette section sont utilisés pour installer les packages R. Ils sont requis lors du développement, de l’entraînement et du déploiement.

> [!NOTE]
> Il ne s’agit pas d’une liste complète des ordinateurs hôtes requis pour toutes les ressources R sur Internet, uniquement celles les plus couramment utilisées. Par exemple, si vous avez besoin d’accéder à un référentiel GitHub ou à un autre hôte, vous devez identifier et ajouter les ordinateurs hôtes requis pour ce scénario.

| **Nom d’hôte** | **Objectif** |
| ---- | ---- |
| **cloud.r-project.org** | Utilisé lors de l’installation des packages CRAN. |

> [!IMPORTANT]
> En interne, le kit SDK R pour Azure Machine Learning utilise des packages Python. Par conséquent, vous devez également autoriser les hôtes Python à traverser le pare-feu.
## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Déployer et configurer un pare-feu Azure à l’aide du portail Azure](../firewall/tutorial-firewall-deploy-portal.md)
* [Sécuriser l’expérimentation Azure Machine Learning et les travaux d’inférence au sein d’un réseau virtuel Azure](how-to-network-security-overview.md)