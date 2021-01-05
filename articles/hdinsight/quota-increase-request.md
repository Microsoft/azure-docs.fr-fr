---
title: Requête d’augmentation du quota de cœurs d’UC – Azure HDInsight
description: Découvrez le processus de requête d’augmentation du nombre de cœurs d’UC alloués à votre abonnement.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/07/2020
ms.openlocfilehash: 93c07b47e03bde20d494feb1c86179cd9a24ba26
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539409"
---
# <a name="requesting-quota-increases-for-azure-hdinsight"></a>Requête d’augmentation du quota pour Azure HDInsight

Les quotas de cœurs d’UC permettent de s’assurer que l’utilisation des ressources est répartie de façon équitable entre tous les clients d’une région Azure particulière. Toutefois, dans certains cas, les besoins de votre entreprise peuvent exiger davantage de ressources de cluster que votre quota actuel. Dans ce cas, vous pouvez demander une augmentation du quota de cœurs d’UC afin de pouvoir déployer des clusters qui correspondent à vos besoins en matière de traitement des données.

Quand vous atteignez une limite de quota, vous ne pouvez pas déployer de nouveaux clusters ou effectuer un scale-out des clusters existants en ajoutant des nœuds Worker. La seule limite de quota est le quota de cœurs de processeur qui existe au niveau régional pour chaque abonnement. Par exemple, votre abonnement peut avoir une limite de 30 cœurs d’UC dans la région USA Est, avec 30 cœurs d’UC supplémentaires autorisés dans la région USA Est 2.

## <a name="gather-required-information"></a>Collecte des informations nécessaire

Si vous avez reçu une erreur indiquant que vous avez atteint une limite de quota, utilisez le processus décrit dans cette section pour collecter des informations importantes et envoyer une demande d’augmentation de quota.

1. Déterminez la taille de machine virtuelle, la mise à l’échelle et le type souhaités.
1. Vérifiez les limites de capacité de quota actuelles de votre abonnement. Pour vérifier vos cœurs disponibles, procédez comme suit :

    1. Connectez-vous au [portail Azure](https://portal.azure.com/).
    1. Accédez à la page **Vue d’ensemble** pour le cluster HDInsight.
    1. Dans le menu de gauche, sélectionnez **Limites de quota** . La page affiche le nombre de cœurs en cours d’utilisation, le nombre de cœurs disponibles et le nombre total de cœurs.

Si vous avez besoin d’une augmentation de quota, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez **Aide + support** en bas à gauche de la page.

    ![bouton Aide et support](./media/quota-increase-request/help-support-button.png)

1. Sélectionnez **Nouvelle demande de support** .
1. Dans la page **Nouvelle demande de support** , sous l’onglet **De base** , sélectionnez les options suivantes :

   - **Type de problème**  : **Limites du service et de l’abonnement (quotas)**
   - **Abonnement**  : l’abonnement à modifier
   - **Type de quota**  : **HDInsight**

     ![Créer une demande de support pour augmenter le quota de cœurs d’HDInsight](./media/quota-increase-request/hdinsight-quota-support-request.png)

1. Sélectionnez **Suivant : Solutions >>** .
1. Dans la page **Détails** , entrez une description du problème, sélectionnez la gravité du problème, votre méthode de contact préférée et d’autres champs obligatoires. Utilisez le modèle indiqué ci-dessous pour vous assurer que vous indiquez les informations nécessaires. Les demandes d’augmentation de quota sont évaluées par l’équipe dédiée à la capacité Azure et non par l’équipe du produit HDInsight. Plus vos informations sont exhaustives et plus votre requête aura de chances d’être approuvée.

    ```
    I would like to request [SPECIFY DESIRED AMOUNT] on [DESIRED SKU] for [SUBSCRIPTION ID].
    
    My current quota on this subscription is [CURRENT QUOTA AMOUNT].
    
    I would like to use the extra cores for [DETAIL REASON].
    ```

    ![Détails du problème](./media/quota-increase-request/problem-details.png)

1. Sélectionnez **Suivant : Vérifier + créer >>** .
1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer** .

> [!NOTE]  
> Si vous avez besoin d’augmenter le quota de cœurs d’HDInsight dans une région privée, [envoyez une demande de liste approuvée](https://aka.ms/canaryintwhitelist).

Vous pouvez [contacter le support technique pour demander une augmentation du quota](../azure-portal/supportability/resource-manager-core-quotas-request.md).

Il existe des limites de quota fixes. Par exemple, un seul abonnement Azure peut avoir au maximum 10 000 cœurs. Pour plus d’informations sur ces limites, consultez [Abonnement Azure et limites, quotas et contraintes du service](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="next-steps"></a>Étapes suivantes

* [Configurer des clusters dans HDInsight avec Apache Hadoop, Spark, Kafka et plus encore](hdinsight-hadoop-provision-linux-clusters.md): En savoir plus sur l’installation et la configuration de clusters dans HDInsight.
* [Analyser les performances d’un cluster](hdinsight-key-scenarios-to-monitor.md): En savoir plus sur les principaux scénarios à analyser pour votre cluster HDInsight pouvant affecter sa capacité.