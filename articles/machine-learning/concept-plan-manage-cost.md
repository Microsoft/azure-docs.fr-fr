---
title: Prévoir et gérer les coûts
titleSuffix: Azure Machine Learning
description: Planifiez et gérez les coûts d’Azure Machine Learning avec l’analyse des coûts dans le portail Azure. Découvrez d’autres conseils permettant de réduire vos coûts lors de la création de modèles ML.
author: sdgilley
ms.author: sgilley
ms.custom: subject-cost-optimization
ms.reviewer: nigup
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: 3eb9df0a0fde5d99bbeb3c2da182b5957fdea1e3
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532922"
---
# <a name="plan-and-manage-costs-for-azure-machine-learning"></a>Planifier et gérer les coûts d’Azure Machine Learning

Cet article explique comment planifier et gérer les coûts d’Azure Machine Learning. Tout d’abord, vous devez utiliser la calculatrice de prix Azure pour planifier les coûts avant d’ajouter des ressources. Ensuite, lorsque vous ajoutez les ressources Azure, passez en revue les coûts estimés. Enfin, utilisez des conseils de réduction des coûts lorsque vous entraînez votre modèle avec des clusters de calcul Azure Machine Learning gérés.

Une fois que vous avez commencé à utiliser des ressources Azure Machine Learning, utilisez les fonctionnalités de gestion des coûts pour définir des budgets et superviser les coûts. Passez également en revue les coûts prévus et déterminez les tendances des dépenses pour identifier les domaines où vous pourriez agir.

Il est important de comprendre que les coûts pour Azure Machine Learning ne représentent qu’une partie des coûts mensuels sur votre facture Azure. Si vous utilisez d’autres services Azure, tous les services et ressources Azure utilisés dans votre abonnement Azure vous sont facturés, y compris les services tiers. Cet article explique comment planifier et gérer les coûts d’Azure Machine Learning. Quand vous connaîtrez bien la gestion des coûts d’Azure Machine Learning, vous pourrez appliquer des méthodes similaires pour gérer les coûts de tous les services Azure utilisés dans votre abonnement.

Lorsque vous entraînez vos modèles Machine Learning, utilisez des clusters de calcul Azure Machine Learning gérés pour tirer parti de conseils de réduction des coûts supplémentaires :

* Configurer vos clusters d’entraînement pour la mise à l’échelle automatique
* Définir des quotas sur votre abonnement et vos espaces de travail
* Définir des stratégies de résiliation sur votre exécution d’entraînement
* Utiliser des machines virtuelles de basse priorité
* Utiliser une instance de machine virtuelle réservée Azure

## <a name="prerequisites"></a>Prérequis

L’analyse des coûts prend en charge différents types de comptes Azure. Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn). Pour afficher les données de coût, vous avez au minimum besoin d’un accès en lecture pour votre compte Azure. 

Pour plus d’informations sur l’attribution de l’accès aux données Azure Cost Management, consultez [Assigner l’accès aux données](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="estimate-costs-before-using-azure-machine-learning"></a>Estimer les coûts avant d’utiliser Azure Machine Learning

Utilisez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) pour estimer les coûts avant de créer les ressources dans un compte Azure Machine Learning. Sur la gauche, sélectionnez **AI + Machine Learning**, puis sélectionnez **Azure Machine Learning** pour commencer.  

La capture d’écran suivante montre l’estimation des coûts effectuée par la calculatrice :

:::image type="content" source="media/concept-plan-manage-cost/capacity-calculator-cost-estimate.png" alt-text="Estimation des coûts dans la calculatrice Azure":::

À mesure que vous ajoutez de nouvelles ressources dans votre espace de travail, revenez à cette calculatrice et ajoutez la même ressource ici pour mettre à jour vos estimations de coûts.

Pour plus d’informations, consultez [Tarifs Azure Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="understand-the-full-billing-model-for-azure-machine-learning"></a>Comprendre le modèle de facturation complet pour Azure Machine Learning

Azure Machine Learning s’exécute sur l’infrastructure Azure qui accumule les coûts avec Azure Machine Learning lorsque vous déployez la nouvelle ressource. Il est important de comprendre qu’une infrastructure supplémentaire peut accumuler des frais. Vous devez gérer ce coût lorsque vous apportez des modifications aux ressources déployées. 

### <a name="costs-that-typically-accrue-with-azure-machine-learning"></a>Coûts qui s’accumulent généralement avec Azure Machine Learning

Lorsque vous créez des ressources pour un espace de travail Azure Machine Learning, des ressources pour d’autres services Azure sont également créées. Il s'agit des éléments suivants :

* Compte de base [Azure Container Registry](https://azure.microsoft.com/pricing/details/container-registry?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Stockage d'objets blob de blocs Azure](https://azure.microsoft.com/pricing/details/storage/blobs?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Usage général v1)
* [Key Vault](https://azure.microsoft.com/pricing/details/key-vault?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Application Insights](https://azure.microsoft.com/en-us/pricing/details/monitor?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
 
### <a name="costs-might-accrue-after-resource-deletion"></a>Coûts qui peuvent s’accumuler après la suppression de la ressource

Lorsque vous supprimez un espace de travail Azure Machine Learning dans le portail Azure ou avec l’interface de ligne de commande Azure, les ressources suivantes continuent d’exister. Les coûts continueront de s’accumuler jusqu’à ce que vous supprimiez ces ressources.

* Azure Container Registry
* Stockage d'objets blob de blocs Azure
* Key Vault
* Application Insights

Pour supprimer l’espace de travail ainsi que ces ressources dépendantes, utilisez le Kit de développement logiciel (SDK) :

```python
ws.delete(delete_dependent_resources=True)
```

Si vous créez le service Azure Kubernetes (AKS) dans votre espace de travail, ou si vous attachez des ressources de calcul à votre espace de travail, vous devez les supprimer séparément dans le [portail Azure](https://portal.azure.com).

### <a name="using-monetary-credit-with-azure-machine-learning"></a>Utilisation de Monetary Credit avec Azure Machine Learning

Vous pouvez payer les frais Azure Machine Learning avec votre crédit d’engagement financier EA. Vous ne pouvez cependant pas utiliser le crédit d’engagement financier EA pour payer des frais pour des produits et services tiers, y compris ceux de la Place de marché Azure.


## <a name="create-budgets"></a>Créer des budgets

Vous pouvez créer des [budgets](https://docs.microsoft.com/azure/cost-management/tutorial-acm-create-budgets?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) pour gérer les coûts et des [alertes](https://docs.microsoft.com/azure/cost-management/cost-mgt-alerts-monitor-usage-spending?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) permettant d’avertir automatiquement des parties prenantes en cas d’anomalies de dépenses et de risques de dépenses excessives. Les alertes sont basées sur les dépenses par rapport aux seuils de budget et de coût. Les budgets et les alertes sont créés pour les abonnements et les groupes de ressources Azure : ils sont donc utiles dans le cadre d’une stratégie globale de supervision des coûts. 

Vous pouvez créer des budgets avec des filtres pour des ressources ou des services spécifiques dans Azure si vous souhaitez disposer d’une plus grande granularité dans votre analyse. Les filtres vous permettent de vous assurer que vous ne créez pas accidentellement de nouvelles ressources entraînant un surcoût. Pour plus d’informations sur les options de filtre lorsque vous créez un budget, consultez [Options de regroupement et de filtre](https://docs.microsoft.com/azure/cost-management-billing/costs/group-filter?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Exporter des données de coûts

Vous pouvez également [exporter vos données de coûts](https://docs.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) vers un compte de stockage. C’est utile quand vous ou d’autres personnes avez besoin d’effectuer des analyses supplémentaires des données concernant les coûts. Par exemple, une équipe Finance peut analyser les données avec Excel ou Power BI. Vous pouvez exporter vos coûts selon une planification quotidienne, hebdomadaire ou mensuelle, et définir une plage de dates personnalisée. L’exportation des données des coûts est la méthode recommandée pour récupérer les jeux de données des coûts.

## <a name="other-ways-to-manage-and-reduce-costs-for-azure-machine-learning"></a>Autres façons de gérer et de réduire les coûts d’Azure Machine Learning

Utilisez ces conseils pour limiter les coûts de vos ressources de calcul Machine Learning.

### <a name="use-azure-machine-learning-compute-cluster-amlcompute"></a>Utiliser le cluster de calcul Azure Machine Learning (AmlCompute)

Avec des données constamment modifiées, vous avez besoin d’un entraînement de modèle rapide et rationalisé et d’un réentraînement pour maintenir des modèles précis. Toutefois, l’entraînement continu est coûteux, en particulier pour les modèles de deep learning sur les GPU. 

Les utilisateurs Azure Machine Learning peuvent utiliser le cluster de calcul Azure Machine Learning géré, également appelé AmlCompute. AmlCompute prend en charge un large éventail d’options de GPU et d’UC. AmlCompute est hébergé en interne pour le compte de votre abonnement par Azure Machine Learning. Il offre les mêmes sécurité, conformité et gouvernance de qualité professionnelle qu’à l’échelle du cloud Azure IaaS.

Ces pools de calcul étant à l’intérieur de l’infrastructure IaaS d’Azure, vous pouvez déployer, mettre à l’échelle et gérer votre entraînement avec les mêmes exigences de sécurité et de conformité que le reste de votre infrastructure.  Ces déploiements se produisent dans votre abonnement et obéissent à vos règles de gouvernance. Découvrez plus en détail la [capacité de calcul Azure Machine Learning](how-to-create-attach-compute-cluster.md).

### <a name="configure-training-clusters-for-autoscaling"></a>Configurer des clusters d’entraînement pour la mise à l’échelle automatique

La mise à l’échelle automatique des clusters selon les besoins de votre charge de travail vous permet de réduire les coûts et d’utiliser uniquement ce dont vous avez besoin.

Les clusters AmlCompute sont conçus pour effectuer dynamiquement la mise à l’échelle en fonction de votre charge de travail. Le cluster peut être mis à l’échelle jusqu’au nombre maximal de nœuds que vous configurez. À la fin de chaque exécution, le cluster publie des nœuds et effectue une mise à l’échelle jusqu’au nombre de nœuds minimal que vous avez configuré.

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

Vous pouvez également configurer la durée d’inactivité du nœud avant l’exécution du scale-down. Par défaut, le délai d’inactivité avant le scale-down est défini sur 120 secondes.

+ Si vous effectuez une expérimentation moins itérative, réduisez ce délai pour réduire les coûts.
+ Si vous effectuez une expérimentation de développement/test hautement itérative, vous devrez peut-être augmenter la durée pour ne pas avoir à payer des mises à l’échelle constantes après chaque modification apportée à votre environnement ou script d’entraînement.

Les clusters AmlCompute peuvent être configurés pour répondre aux besoins fluctuants de votre charge de travail dans le portail Azure, à l’aide de la [classe SDK AmlCompute](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?preserve-view=true&view=azure-ml-py) et de l’interface [CLI AmlCompute](/cli/azure/ext/azure-cli-ml/ml/computetarget/create?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute) avec les [API REST](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/machinelearningservices/resource-manager/Microsoft.MachineLearningServices/stable).

```azurecli
az ml computetarget create amlcompute --name testcluster --vm-size Standard_NC6 --min-nodes 0 --max-nodes 5 --idle-seconds-before-scaledown 300
```

### <a name="set-quotas-on-resources"></a>Définir des quotas sur les ressources

AmlCompute est fourni avec une [configuration de quota (ou limite)](how-to-manage-quotas.md#azure-machine-learning-compute). Ce quota est défini par famille de machines virtuelles (par exemple, la série Dv2, la série NCv3) et varie selon la région pour chaque abonnement. Les abonnements commencent par des valeurs par défaut réduites pour vous aider, mais utilisez ce paramètre pour contrôler la quantité de ressources AmlCompute pouvant être mises en service dans votre abonnement. 

Configurez également un [quota au niveau de l’espace de travail par famille de machines virtuelles](how-to-manage-quotas.md#workspace-level-quotas) pour chaque espace de travail d’un abonnement. Cela vous permet d’avoir un contrôle plus précis sur les coûts que chaque espace de travail peut potentiellement générer, et de restreindre certaines familles de machines virtuelles. 

Pour définir des quotas au niveau de l’espace de travail, commencez dans le [portail Azure](https://portal.azure.com).  Sélectionnez un espace de travail quelconque dans votre abonnement, puis sélectionnez **Utilisations + quotas** dans le volet gauche. Sélectionnez ensuite l’onglet **Configurer les quotas** pour afficher les quotas. Vous avez besoin de privilèges dans l’étendue de l’abonnement pour définir ce quota, car il s’agit d’un paramètre qui affecte plusieurs espaces de travail.

### <a name="set-run-autotermination-policies"></a>Définir des stratégies de résiliation automatique d’exécution 

Dans certains cas, vous devez configurer vos exécutions d’apprentissage pour limiter leur durée ou y mettre fin prématurément. Par exemple, lorsque vous utilisez le réglage d’hyperparamètre intégré d’Azure Machine Learning ou Machine Learning automatisé.

Voici quelques options dont vous disposez :
* Définissez un paramètre appelé `max_run_duration_seconds` dans votre RunConfiguration pour contrôler la durée maximale pendant laquelle une exécution peut s’étendre sur le calcul que vous choisissez (calcul cloud local ou distant).
* Pour le [réglage d’hyperparamètre](how-to-tune-hyperparameters.md#early-termination), définissez une stratégie de résiliation anticipée à partir d’une stratégie Bandit, d’une stratégie d’arrêt médiane ou d’une stratégie de sélection de troncation. Pour contrôler davantage les balayages d’hyperparamètres, utilisez des paramètres tels que `max_total_runs` ou `max_duration_minutes`.
* Pour [Machine Learning automatisé](how-to-configure-auto-train.md#exit), définissez des stratégies de résiliation similaires à l’aide de l’indicateur `enable_early_stopping`. Utilisez également des propriétés telles que `iteration_timeout_minutes` et `experiment_timeout_minutes` pour contrôler la durée maximale d’une exécution ou de l’ensemble de l’expérience.

### <a name="use-low-priority-vms"></a><a id="low-pri-vm"></a> Utiliser des machines virtuelles de faible priorité

Azure vous permet d’utiliser une capacité inutilisée excessive en tant que machines virtuelles de basse priorité sur des groupes identiques de machines virtuelles (VMSS), Batch et le service Machine Learning. Ces allocations sont préventives mais disponibles à prix réduit par rapport aux machines virtuelles dédiées. En général, nous vous recommandons d’utiliser des machines virtuelles de faible priorité pour les charges de travail Batch. Vous devez également les utiliser lorsque des interruptions sont récupérables par le biais de renvois (pour l’inférence par lots) ou de redémarrages (pour un entraînement Deep Learning avec points de contrôle).

Les machines virtuelles de basse priorité ont un quota unique distinct de la valeur de quota dédiée, qui est défini par famille de machines virtuelles. Apprenez-en davantage sur les [quotas AmlCompute](how-to-manage-quotas.md).

 Les machines virtuelles de basse priorité ne fonctionnent pas pour les instances de calcul, car elles doivent prendre en charge les expériences de notebook interactives.

### <a name="use-reserved-instances"></a>Utiliser des instances réservées

Une autre façon d’économiser de l’argent sur les ressources de calcul est l’instance de machine virtuelle réservée Azure. Avec cette offre, vous vous engagez à des termes de 1 an ou 3 ans. Ces remises vont jusqu’à 72 % des prix de paiement à l’utilisation et s’appliquent directement sur votre facture Azure mensuelle.

La capacité de calcul Azure Machine Learning prend en charge les instances réservées par nature. Si vous achetez une instance réservée d’un an ou de trois ans, nous appliquons automatiquement la remise sur la base de votre calcul managé Azure Machine Learning.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment optimiser votre investissement cloud avec Azure Cost Management](https://docs.microsoft.com/azure/cost-management-billing/costs/cost-mgt-best-practices?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Apprenez-en davantage sur la gestion des coûts avec l’[analyse du coût](https://docs.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Découvrez comment [éviter des coûts imprévus](https://docs.microsoft.com/azure/cost-management-billing/manage/getting-started?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Suivez le cours d’apprentissage guidé [Cost Management](https://docs.microsoft.com/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).