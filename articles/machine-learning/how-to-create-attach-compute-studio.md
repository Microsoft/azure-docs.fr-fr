---
title: Créer des calculs de formation et de déploiement (studio)
titleSuffix: Azure Machine Learning
description: Utilisez Studio pour créer des ressources de calcul d’entraînement et de déploiement (cibles de calcul) pour le Machine Learning.
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 08/06/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1
ms.openlocfilehash: ab7a74166e85f2ba9fd73e7323cf9cd200cf32e4
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031026"
---
# <a name="create-compute-targets-for-model-training-and-deployment-in-azure-machine-learning-studio"></a>Créer des cibles de calcul pour l’entraînement et le déploiement de modèle dans Azure Machine Learning Studio

Dans cet article, vous allez découvrir comment créer et gérer des cibles de calcul dans Azure Machine Learning Studio.  Vous pouvez également créer et gérer des cibles de calcul avec :

* Le SDK Azure Machine Learning ou l’extension CLI pour Azure Machine Learning
  * [Instance de calcul](how-to-create-manage-compute-instance.md)
  * [Cluster de calcul](how-to-create-attach-compute-cluster.md)
  * [Cluster Azure Kubernetes Service](how-to-create-attach-kubernetes.md)
  * [Autres ressources de calcul](how-to-attach-compute-targets.md)
* L’[extension VS Code](how-to-manage-resources-vscode.md#compute-clusters) pour Azure Machine Learning.


## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui
* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md)

## <a name="whats-a-compute-target"></a>Qu’est-ce qu’une cible de calcul ? 

Azure Machine Learning vous permet de faire l’apprentissage de votre modèle sur une variété de ressources ou d’environnements, appelés collectivement [__cibles de calcul__](concept-azure-machine-learning-architecture.md#compute-targets). Une cible de calcul peut être un ordinateur local ou une ressource cloud, comme une capacité de calcul Azure Machine Learning, Azure HDInsight ou une machine virtuelle distante.  Vous pouvez également créer des cibles de calcul pour le déploiement de modèle, comme décrit dans [« Déployer des modèles avec le service Azure Machine Learning »](how-to-deploy-and-where.md).

## <a name="view-compute-targets"></a><a id="portal-view"></a>Afficher les cibles de calcul

Pour afficher toutes les cibles de calcul de votre espace de travail, effectuez les étapes suivantes :

1. Accédez à [Azure Machine Learning Studio](https://ml.azure.com).
 
1. Sous __Gérer__, sélectionnez __Calcul__.

1. Sélectionnez des onglets en haut pour afficher chaque type de cible de calcul.

    :::image type="content" source="media/how-to-create-attach-studio/view-compute-targets.png" alt-text="Affichez la liste des cibles de calcul":::

## <a name="create-compute-target"></a><a id="portal-create"></a>Créer une cible de calcul

Suivez la procédure ci-dessus pour afficher la liste des cibles de calcul. Puis procédez comme suit pour créer une cible de calcul :

1. Sélectionnez l’onglet en haut correspondant au type de calcul que vous allez créer.

1. Si vous n’avez aucune cible de calcul, sélectionnez  **Créer** au milieu de la page.
  
    :::image type="content" source="media/how-to-create-attach-studio/create-compute-target.png" alt-text="Créer une cible de calcul":::

1. Si vous voyez une liste de ressources de calcul, sélectionnez **+ Nouveau** au-dessus de la liste.

    :::image type="content" source="media/how-to-create-attach-studio/select-new.png" alt-text="Sélectionnez Nouveau":::


1. Remplissez le formulaire correspondant à votre type de calcul :

  * [Instance de calcul](#compute-instance)
  * [Clusters de calcul](#amlcompute)
  * [Clusters d’inférence](#inference-clusters)
  * [Calcul attaché](#attached-compute)

1. Sélectionnez __Create__ (Créer).

1. Pour afficher l’état de l’opération de création, sélectionnez la cible de calcul dans la liste :

    :::image type="content" source="media/how-to-create-attach-studio/view-list.png" alt-text="Affichez l’état du calcul dans une liste":::


### <a name="compute-instance"></a>Instance de calcul

Utilisez les [étapes ci-dessus](#portal-create) pour créer l’instance de calcul.  Remplissez ensuite le formulaire comme suit :

:::image type="content" source="media/concept-compute-instance/create-compute-instance.png" alt-text="Créer une instance de calcul":::


|Champ  |Description  |
|---------|---------|
|Nom du calcul     |  <li>Le nom est obligatoire et doit comprendre entre 3 et 24 caractères.</li><li>Les caractères valides sont les lettres majuscules et minuscules, les chiffres et le caractère **-** .</li><li>Le nom doit commencer par une lettre.</li><li>Le nom doit être unique parmi tous les calculs existants au sein d’une région Azure. Si le nom que vous choisissez n’est pas unique, une alerte s’affiche.</li><li>Si le caractère **-** est utilisé, il doit être suivi d’au moins une lettre plus loin dans le nom.</li>     |
|Type de machine virtuelle |  Choisissez UC ou GPU. Vous ne pouvez pas modifier ce type après la création.     |
|Taille de la machine virtuelle     |  Il se peut que les tailles de machines virtuelles prises en charge soient restreintes dans votre région. Vérifier la [liste de disponibilité](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Activer/désactiver l’accès SSH     |   L’accès SSH est désactivé par défaut.  Il n’est pas possible de modifier l’accès SSH après sa création. Veillez à activer l’accès si vous envisagez de déboguer de manière interactive avec [VS Code Remote](how-to-set-up-vs-code-remote.md)   |
|Paramètres avancés     |  facultatif. Configurez un réseau virtuel. Pour créer l’instance de calcul à l’intérieur d’un réseau virtuel Azure (vnet), spécifiez le **Groupe de ressources**, le **Réseau virtuel** et le **Sous-réseau**. Pour plus d’informations, consultez la [configuration requise](./how-to-secure-training-vnet.md) pour le réseau virtuel.  |

### <a name="compute-clusters"></a><a name="amlcompute"></a> Clusters de calcul

Créez un cluster de calcul à un seul ou plusieurs nœuds pour vos charges de travail d’entraînement, d’inférence de lot ou d’apprentissage par renforcement. Utilisez les [étapes ci-dessus](#portal-create) pour créer le cluster de calcul.  Remplissez ensuite le formulaire comme suit :


|Champ  |Description  |
|---------|---------|
|Nom du calcul     |  <li>Le nom est obligatoire et doit comprendre entre 3 et 24 caractères.</li><li>Les caractères valides sont les lettres majuscules et minuscules, les chiffres et le caractère **-** .</li><li>Le nom doit commencer par une lettre.</li><li>Le nom doit être unique parmi tous les calculs existants au sein d’une région Azure. Si le nom que vous choisissez n’est pas unique, une alerte s’affiche.</li><li>Si le caractère **-** est utilisé, il doit être suivi d’au moins une lettre plus loin dans le nom.</li>     |
|Type de machine virtuelle |  Choisissez UC ou GPU. Vous ne pouvez pas modifier ce type après la création.     |
|Priorité de machine virtuelle | Choisissez **Dédié** ou **Basse priorité**.  Les machines virtuelles basse priorité sont moins chères, mais ne garantissent pas les nœuds de calcul. Votre travail peut être anticipé.
|Taille de la machine virtuelle     |  Il se peut que les tailles de machines virtuelles prises en charge soient restreintes dans votre région. Vérifier la [liste de disponibilité](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Nombre minimal de nœuds | Nombre minimal de nœuds que vous souhaitez provisionner. Si vous souhaitez un nombre dédié de nœuds, spécifiez-le ici. Faites des économies en affectant la valeur 0 comme minimum. Ainsi, vous ne paierez aucun nœud lorsque le cluster sera inactif. |
|Nombre maximal de nœuds | Nombre maximal de nœuds que vous souhaitez provisionner. Le calcul sera automatiquement mis à l’échelle jusqu’à ce nombre maximal de nœuds lors de l’envoi d’un travail. |
|Paramètres avancés     |  facultatif. Configurez un réseau virtuel. Pour créer l’instance de calcul à l’intérieur d’un réseau virtuel Azure (vnet), spécifiez le **Groupe de ressources**, le **Réseau virtuel** et le **Sous-réseau**. Pour plus d’informations, consultez la [configuration requise](./how-to-secure-training-vnet.md) pour le réseau virtuel.   Attachez également des [identités managées](#managed-identity) pour accorder l’accès aux ressources.     |

#### <a name="set-up-managed-identity"></a><a name="managed-identity"></a>Configurer une identité managée

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-intro.md)]

Lors de la création du cluster de calcul ou de la modification de ses détails, dans les **Paramètres avancés**, activez **Attribuer une identité managée** et spécifiez une identité affectée par le système ou une identité affectée par l’utilisateur.

#### <a name="managed-identity-usage"></a>Utilisation de l’identité managée

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-default.md)]

### <a name="inference-clusters"></a>Clusters d’inférence

> [!IMPORTANT]
> L’utilisation du service Azure Kubernetes avec Azure Machine Learning offre plusieurs options de configuration. Certains scénarios, tels que la mise en réseau, nécessitent une installation et une configuration supplémentaires. Pour plus d’informations sur l’utilisation d’AKS avec Azure ML, consultez [Créer et attacher un cluster Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

Créez ou attachez un cluster AKS (Azure Kubernetes service) pour l’inférence à grande échelle. Utilisez les [étapes ci-dessus](#portal-create) pour créer le cluster AKS.  Remplissez ensuite le formulaire comme suit :


|Champ  |Description  |
|---------|---------|
|Nom du calcul     |  <li>Le nom est obligatoire. Le nom doit faire entre 2 et 16 caractères. </li><li>Les caractères valides sont les lettres majuscules et minuscules, les chiffres et le caractère **-** .</li><li>Le nom doit commencer par une lettre.</li><li>Le nom doit être unique parmi tous les calculs existants au sein d’une région Azure. Si le nom que vous choisissez n’est pas unique, une alerte s’affiche.</li><li>Si le caractère **-** est utilisé, il doit être suivi d’au moins une lettre plus loin dans le nom.</li>     |
|Kubernetes Service | Sélectionnez **Créer** et remplissez le reste du formulaire.  Ou sélectionnez **Utiliser l’existant**, puis sélectionnez un cluster AKS existant dans votre abonnement.
|Région |  Sélectionnez l’emplacement où sera créé le cluster. |
|Taille de la machine virtuelle     |  Il se peut que les tailles de machines virtuelles prises en charge soient restreintes dans votre région. Vérifier la [liste de disponibilité](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Objectif du cluster  | Sélectionnez **Production** ou **Dev-test**. |
|Nombre de nœuds | Le nombre de nœuds multiplié par le nombre de cœurs (processeurs virtuels) de la machine virtuelle doit être supérieur ou égal à 12. |
| Configuration réseau | Sélectionnez **Avancé** pour créer le calcul au sein d’un réseau virtuel existant. Pour plus d’informations sur AKS dans un réseau virtuel, consultez [Isolement réseau pendant l’entraînement et l’inférence avec les points de terminaison privés et les réseaux virtuels privés](./how-to-secure-inferencing-vnet.md). |
| Activer la configuration SSL | Utilisez ceci pour configurer le certificat SSL sur le calcul. |

### <a name="attached-compute"></a>Calcul attaché

Pour utiliser des cibles de calcul créées en dehors de l’espace de travail Azure Machine Learning, vous devez les joindre. Une fois la cible de calcul jointe, elle sera disponible dans votre espace de travail.  Utilisez **Calcul attaché** afin d’attacher une cible de calcul pour l’**entraînement**.  Utilisez **Clusters d’inférence** afin d’attacher un cluster AKS pour l’**inférence**.

Utilisez les [étapes ci-dessus](#portal-create) pour attacher un calcul.  Remplissez ensuite le formulaire comme suit :

1. Entrez un nom pour la cible de calcul. 
1. Sélectionnez le type de calcul à attacher. Il n’est pas possible d’attacher tous les types de capacités de calcul à partir d’Azure Machine Learning Studio. Actuellement, les types qui peuvent être attachés pour l’entraînement sont les suivants :
    * Machine virtuelle distante
    * Azure Databricks (pour une utilisation dans des pipelines de Machine Learning)
    * Azure Data Lake Analytics (pour une utilisation dans des pipelines de Machine Learning)
    * Azure HDInsight

1. Remplissez le formulaire et indiquez une valeur pour les propriétés requises.

    > [!NOTE]
    > Microsoft recommande d’utiliser des clés SSH, car elles sont plus sûres que les mots de passe. Les mots de passe sont vulnérables aux attaques en force brute. Les clés SSH utilisent des signatures de chiffrement. Pour plus d’informations sur la création de clés SSH pour une utilisation avec Machines virtuelles Azure, consultez les documents suivants :
    >
    > * [Créer et utiliser des clés SSH sur Linux ou macOS](../virtual-machines/linux/mac-create-ssh-keys.md)
    > * [Créer et utiliser des clés SSH sur Windows](../virtual-machines/linux/ssh-from-windows.md)

1. Sélectionnez __Attacher__. 


## <a name="next-steps"></a>Étapes suivantes

Après avoir créé une cible et l’avoir attachée à votre espace de travail, vous l’utilisez dans votre [configuration d’exécution](how-to-set-up-training-targets.md) avec un objet `ComputeTarget` :

```python
from azureml.core.compute import ComputeTarget
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

* Utilisez la ressource de calcul pour [soumettre une exécution d’entraînement](how-to-set-up-training-targets.md).
* [Tutoriel : Entraîner un modèle](tutorial-train-models-with-aml.md) utilise une cible de calcul managée pour entraîner un modèle.
* Découvrez comment [optimiser efficacement les hyperparamètres](how-to-tune-hyperparameters.md) afin de générer des modèles plus efficaces.
* Une fois le modèle formé, découvrez [comment et où déployer les modèles](how-to-deploy-and-where.md).
* [Utiliser Azure Machine Learning avec des réseaux virtuels Azure](./how-to-network-security-overview.md)