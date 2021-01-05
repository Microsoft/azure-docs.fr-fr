---
title: Vue d’ensemble de l’isolement et de la sécurité des réseaux virtuels
titleSuffix: Azure Machine Learning
description: Utiliser un réseau virtuel Microsoft Azure avec Azure Machine Learning pour sécuriser les ressources d’espace de travail et les environnements de calcul.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 10/06/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, references_regions, contperf-fy21q1
ms.openlocfilehash: dd24c4f20fa3a56d25ef142947268b2d2b75474a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029598"
---
# <a name="virtual-network-isolation-and-privacy-overview"></a>Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels

Dans cet article, vous allez découvrir comment utiliser des réseaux virtuels (VNets) pour sécuriser la communication réseau dans Azure Machine Learning. Cet article utilise un exemple de scénario pour vous montrer comment configurer un réseau virtuel complet.

Cet article fait partie d’une série en cinq parties qui vous guide à travers le processus de sécurisation d’un workflow Azure Machine Learning. Nous vous recommandons vivement de commencer par lire cet article pour comprendre les concepts. 

Voici les autres articles de cette série :

**1. Présentation du réseau virtuel** > [2. Sécurisation de l’espace de travail](how-to-secure-workspace-vnet.md) > [3. Sécurisation de l’environnement d’entraînement](how-to-secure-training-vnet.md) > [4. Sécurisation de l’environnement d’inférence](how-to-secure-inferencing-vnet.md) > [5. Activation de la fonctionnalité studio](how-to-enable-studio-virtual-network.md)

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous connaissez déjà les sujets suivants :
+ [Réseaux virtuels Azure](../virtual-network/virtual-networks-overview.md)
+ [Réseaux IP](../virtual-network/public-ip-addresses.md)
+ [Azure Private Link](how-to-configure-private-link.md)
+ [Groupes de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md)
+ [Pare-feu de réseau](../firewall/overview.md)

## <a name="example-scenario"></a>Exemple de scénario

Dans cette section, vous allez découvrir comment on peut configurer un scénario de réseau courant pour sécuriser la communication Azure Machine Learning avec des adresses IP privées.

Le tableau suivant compare l’accès des services aux différentes parties d’un réseau Azure Machine Learning avec et sans VNet.

| Scénario | Espace de travail | Ressources associées | Environnement de calcul pour l’entraînement | Environnement de calcul pour l’inférence |
|-|-|-|-|-|-|
|**Aucun réseau virtuel**| Adresse IP publique | Adresse IP publique | Adresse IP publique | Adresse IP publique |
|**Protéger les ressources dans un réseau virtuel**| Adresse IP privée (point de terminaison privé) | Adresse IP publique (point de terminaison de service) <br> **- ou -** <br> Adresse IP privée (point de terminaison privé) | IP privée | IP privée  | 

* **Espace de travail** - Créez un point de terminaison privé à partir de votre réseau virtuel pour vous connecter à Private Link sur l’espace de travail. Le point de terminaison privé connecte l’espace de travail au réseau virtuel par le biais de plusieurs adresses IP privées.
* **Ressource associée** - Utilisez les points de terminaison du service ou des points de terminaison privés pour vous connecter à des ressources d’espace de travail telles que Stockage Azure, Azure Key Vault et Azure Container Services.
    * Les **points de terminaison de service** fournissent l’identité de votre réseau virtuel au service Azure. Une fois que vous avez activé les points de terminaison de service dans votre réseau virtuel, vous pouvez ajouter une règle de réseau virtuel afin de sécuriser les ressources du service Azure pour votre réseau virtuel. Les points de terminaison de service utilisent des adresses IP publiques.
    * Les **points de terminaison privés** sont des interfaces réseau qui vous connectent de manière sécurisée à un service alimenté par Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel.
* **Accès au calcul d’entraînement** - Accédez à des cibles de calcul d’entraînement comme une instance de calcul Azure Machine Learning et des clusters de calcul Azure Machine Learning de manière sécurisée avec des adresses IP privées. 
* **Accès au calcul d’inférence** - Accédez à des clusters de calcul Azure Kubernetes Services (AKS) avec des adresses IP privées.


Les cinq sections suivantes vous montrent comment sécuriser le scénario réseau décrit ci-dessus. Pour sécuriser votre réseau, vous devez :

1. Sécuriser l’[**espace de travail et les ressources associées**](#secure-the-workspace-and-associated-resources).
1. Sécuriser l’[**environnement d’entraînement**](#secure-the-training-environment).
1. Sécuriser l’[**environnement d’inférence**](#secure-the-inferencing-environment).
1. Facultatif : [**activer la fonctionnalité studio**](#optional-enable-studio-functionality).
1. Configurer les [**paramètres de pare-feu**](#configure-firewall-settings)

## <a name="secure-the-workspace-and-associated-resources"></a>Sécuriser l’espace de travail et les ressources associées

Utilisez la procédure ci-dessous pour sécuriser votre espace de travail et les ressources associées. Ces étapes permettent à vos services de communiquer dans le réseau virtuel.

1. Créez un [espace de travail avec Private Link activé](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint) pour activer la communication entre votre réseau virtuel et votre espace de travail.
1. Ajoutez Azure Key Vault au réseau virtuel avec un [point de terminaison de service](../key-vault/general/overview-vnet-service-endpoints.md) ou un [point de terminaison privé](../key-vault/general/private-link-service.md). Définissez le Key Vault sur [« Autoriser les services Microsoft approuvés à contourner ce pare-feu »](how-to-secure-workspace-vnet.md#secure-azure-key-vault).
1. Ajoutez votre compte de stockage Azure au réseau virtuel avec un [point de terminaison de service](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints) ou un [point de terminaison privé](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-private-endpoints).
1. [Configurez Azure Container Registry de manière à utiliser un point de terminaison privé](how-to-secure-workspace-vnet.md#enable-azure-container-registry-acr) et [activez la délégation de sous-réseau dans Azure Container Instances](how-to-secure-inferencing-vnet.md#enable-azure-container-instances-aci).

![Diagramme d’architecture représentant la manière dont l’espace de travail et les ressources associées communiquent entre eux par le biais des points de terminaison de service ou privés au sein d’un réseau virtuel](./media/how-to-network-security-overview/secure-workspace-resources.png)

Retrouvez les instructions détaillées relatives à cette procédure dans l’article [Sécuriser un espace de travail Azure Machine Learning](how-to-secure-workspace-vnet.md). 

### <a name="limitations"></a>Limites

La sécurisation de votre espace de travail et des ressources associées dans un réseau virtuel présente les limitations suivantes :
- L’utilisation d’un espace de travail Azure Machine Learning avec un lien privé n’est pas disponible dans les régions Azure Government ou Azure China 21Vianet.
- Toutes les ressources doivent se trouver derrière le même réseau virtuel. Toutefois, des sous-réseaux peuvent être utilisés au sein d’un même réseau virtuel.

## <a name="secure-the-training-environment"></a>Sécuriser l’environnement d’entraînement

Dans cette section, vous allez voir comment sécuriser l’environnement d’entraînement dans Azure Machine Learning. Vous découvrirez également comment Azure Machine Learning effectue un travail d’entraînement pour comprendre le fonctionnement des configurations réseau.

Pour sécuriser l’environnement d’entraînement, procédez comme suit :

1. Créez une [instance de calcul et un cluster de calcul Azure Machine Learning dans le réseau virtuel](how-to-secure-training-vnet.md#compute-instance) pour exécuter le travail d’entraînement.
1. [Autorisez la communication entrante à partir d’Azure Batch Service](how-to-secure-training-vnet.md#mlcports) de sorte que Batch Service puisse envoyer des travaux à vos ressources de calcul. 

![Diagramme d’architecture représentant la sécurisation des clusters et instances de calcul managés](./media/how-to-network-security-overview/secure-training-environment.png)

Retrouvez les instructions détaillées relatives à cette procédure dans l’article [Sécuriser un environnement d’entraînement](how-to-secure-training-vnet.md). 

### <a name="example-training-job-submission"></a>Envoi d’un exemple de travail d’entraînement 

Dans cette section, vous allez voir comment Azure Machine Learning communique de manière sécurisée entre les services pour soumettre un travail d’entraînement. Vous découvrirez comment toutes vos configurations s’associent pour sécuriser la communication.

1. Le client charge des scripts et des données d’entraînement vers des comptes de stockage sécurisés au moyen d’un point de terminaison de service ou privé.

1. Le client envoie un travail d’entraînement à l’espace de travail Azure Machine Learning via le point de terminaison privé.

1. Azure Batch Services reçoit le travail de la part de l’espace de travail et l’envoie à l’environnement de calcul via l’équilibreur de charge public configuré avec la ressource de calcul. 

1. La ressource de calcul reçoit le travail et commence l’entraînement. La ressource de calcul accède aux comptes de stockage sécurisés pour télécharger les fichiers d’entraînement et charger la sortie. 

![Diagramme d’architecture représentant l’envoi d’un travail d’entraînement Azure Machine Learning lors de l’utilisation d’un réseau virtuel](./media/how-to-network-security-overview/secure-training-job-submission.png)


### <a name="limitations"></a>Limites

- L’instance et les clusters Azure Compute doivent se trouver dans le même réseau virtuel, la même région et le même abonnement que l’espace de travail et les ressources associées. 

## <a name="secure-the-inferencing-environment"></a>Sécuriser l’environnement d’inférence

Dans cette section, vous allez découvrir les options disponibles pour sécuriser un environnement d’inférence. Nous vous recommandons d’utiliser des clusters Azure Kubernetes Services (AKS) pour les déploiements de production à grande échelle.

Deux options sont disponibles pour les clusters AKS dans un réseau virtuel :

- Déployer ou attacher un cluster AKS par défaut dans votre réseau virtuel.
- Attacher un cluster AKS privé à votre réseau virtuel.

Les **clusters AKS par défaut** ont un plan de contrôle avec des adresses IP publiques. Vous pouvez ajouter un cluster AKS par défaut à votre réseau virtuel pendant le déploiement ou attacher un cluster une fois celui-ci créé.

Les **clusters AKS privés** ont un plan de contrôle qui n’est accessible que par adresse IP privée. Les clusters AKS privés doivent être attachés une fois le cluster créé.

Retrouvez les instructions détaillées concernant l’ajout de clusters par défaut et privés dans l’article [Sécuriser un environnement d’inférence](how-to-secure-inferencing-vnet.md). 

Le diagramme de réseau suivant représente un espace de travail Azure Machine Learning sécurisé avec un cluster AKS privé attaché au réseau virtuel.

![Diagramme d’architecture représentant l’attachement d’un cluster AKS privé au réseau virtuel. Le plan de contrôle AKS est placé en dehors du réseau virtuel client](./media/how-to-network-security-overview/secure-inferencing-environment.png)

### <a name="limitations"></a>Limites
- Les clusters AKS doivent appartenir au même réseau virtuel que l’espace de travail et ses ressources associées. 

## <a name="optional-enable-studio-functionality"></a>Facultatif : activer la fonctionnalité studio

[Sécuriser l’espace de travail](#secure-the-workspace-and-associated-resources) > [Sécuriser l’environnement d’entraînement](#secure-the-training-environment) > [Sécuriser l’environnement d’inférence](#secure-the-inferencing-environment) > **Activer la fonctionnalité studio** > [Configurer les paramètres du pare-feu](#configure-firewall-settings)

Si votre stockage se trouve dans un réseau virtuel, vous devez d’abord effectuer des étapes de configuration supplémentaires pour activer toutes les fonctionnalités dans [le studio](overview-what-is-machine-learning-studio.md). Par défaut, les fonctionnalités suivantes sont désactivées :

* Aperçu des données dans Studio
* Visualisation des données dans le concepteur
* Déploiement d’un modèle dans le concepteur
* Envoi d’une expérience AutoML
* Démarrage d’un projet d’étiquetage

Pour activer toute la fonctionnalité studio dans un réseau virtuel, consultez [Utiliser le studio Azure Machine Learning dans un réseau virtuel](how-to-enable-studio-virtual-network.md#configure-data-access-in-the-studio). Le studio prend en charge les comptes de stockage à l’aide de points de terminaison de service ou de points de terminaison privés.

### <a name="limitations"></a>Limites
- [L’étiquetage des données assisté par ML](how-to-create-labeling-projects.md#use-ml-assisted-labeling) ne prend pas en charge les comptes de stockage par défaut sécurisés derrière un réseau virtuel. Vous devez utiliser un compte de stockage autre que celui par défaut pour l’étiquetage des données assisté par ML. Notez que le compte de stockage autre que celui par défaut peut être sécurisé derrière le réseau virtuel. 

## <a name="configure-firewall-settings"></a>Configurer les paramètres de pare-feu

Configurez votre pare-feu pour contrôler l’accès aux ressources de votre espace de travail Azure Machine Learning et à l’Internet public. Bien que le Pare-feu Azure soit recommandé, vous devriez pouvoir utiliser d’autres produits de pare-feu pour sécuriser votre réseau. Si vous avez des questions sur la façon d’autoriser la communication via votre pare-feu, consultez la documentation du pare-feu que vous utilisez.

Pour plus d’informations sur les paramètres de pare-feu, consultez [Utiliser un espace de travail derrière un pare-feu](how-to-access-azureml-behind-firewall.md).

## <a name="custom-dns"></a>Système DNS personnalisé

Si vous devez utiliser une solution DNS personnalisée pour votre réseau virtuel, vous devez ajouter des enregistrements d’hôte pour votre espace de travail.

Pour plus d’informations sur les noms de domaine et les adresses IP requis, consultez [Comment utiliser un espace de travail avec un serveur DNS personnalisé](how-to-custom-dns.md).

## <a name="next-steps"></a>Étapes suivantes

Cet article fait partie d’une série en quatre parties sur les réseaux virtuels. Consultez les autres articles pour découvrir comment sécuriser un réseau virtuel :

* [Partie 2 : Vue d’ensemble des réseaux virtuels](how-to-secure-workspace-vnet.md)
* [Partie 3 : Sécuriser l’environnement d’entraînement](how-to-secure-training-vnet.md)
* [Partie 4 : Sécuriser l’environnement d’inférence](how-to-secure-inferencing-vnet.md)
* [Partie 5 : Activer la fonctionnalité Studio](how-to-enable-studio-virtual-network.md)