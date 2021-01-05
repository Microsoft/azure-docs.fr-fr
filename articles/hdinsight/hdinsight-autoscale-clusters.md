---
title: Mise à l’échelle automatique des clusters Azure HDInsight
description: Utilisez la fonction Azure HDInsight de mise à l’échelle automatique pour les clusters Apache Hadoop.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: contperf-fy21q1
ms.date: 09/14/2020
ms.openlocfilehash: 09e4412128a3b13abfa91bf0c128372b30b3e686
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033134"
---
# <a name="autoscale-azure-hdinsight-clusters"></a>Mettre à l’échelle automatiquement des clusters Azure HDInsight

La fonctionnalité de mise à l’échelle automatique gratuite d’Azure HDInsight augmente ou diminue automatiquement le nombre de nœuds Worker dans votre cluster en fonction de critères définis au préalable. Vous définissez un nombre minimal et maximal de nœuds au moment de la création du cluster, vous établissez les critères de mise à l’échelle selon une planification quotidienne ou des métriques de performances spécifiques, et la plateforme HDInsight fait le reste.

## <a name="how-it-works"></a>Fonctionnement

La fonctionnalité de mise à l’échelle automatique utilise deux types de conditions pour déclencher des événements de mise à l’échelle : des seuils pour diverses métriques de performances de cluster (*mise à l’échelle basée sur la charge*) et des déclencheurs temporels (*mise à l’échelle basée sur la planification*). Une mise à l’échelle basée sur la charge modifie le nombre de nœuds dans votre cluster, selon une plage que vous définissez, pour assurer une utilisation optimale de l’UC et réduire les coûts d’exécution. La mise à l’échelle basée sur la planification change le nombre de nœuds dans votre cluster en fonction des opérations que vous associez à des dates et heures spécifiques.

La vidéo suivante fournit une vue d’ensemble des défis que la mise à l’échelle automatique permet de résoudre et comment elle peut vous aider à contrôler les coûts avec HDInsight.


> [!VIDEO https://www.youtube.com/embed/UlZcDGGFlZ0?WT.mc_id=dataexposed-c9-niner]

### <a name="choosing-load-based-or-schedule-based-scaling"></a>Choix de la mise à l’échelle basée sur la planification ou la charge

Choisissez un type de mise à l’échelle en tenant compte des facteurs suivants :

* Variance de la charge : la charge du cluster suit-elle un modèle cohérent à des moments spécifiques, des jours spécifiques ? Si ce n’est pas le cas, nous vous recommandons d’opter pour une mise à l’échelle basée sur la charge.
* Exigences du Contrat de niveau de service : La mise à l’échelle automatique est réactive, et non pas prédictive. Le délai entre le début de l’augmentation de la charge et le moment où le cluster doit atteindre sa taille prévue est-il suffisant ? S’il existe des exigences de contrat SLA strictes et que la charge est un modèle connu fixe, nous vous recommandons d’opter pour une mise à l’échelle basée sur la planification.

### <a name="cluster-metrics"></a>Métriques de cluster

La mise à l’échelle automatique supervise en permanence le cluster et collecte les métriques suivantes :

|Métrique|Description|
|---|---|
|Total Pending CPU|Nombre total de cœurs nécessaires pour commencer l’exécution de tous les conteneurs en attente.|
|Total Pending Memory|Mémoire totale (en Mo) nécessaire pour commencer l’exécution de tous les conteneurs en attente.|
|Total Free CPU|Somme de tous les cœurs inutilisés sur les nœuds Worker actifs.|
|Total Free Memory|Somme de la mémoire inutilisée (en Mo) sur les nœuds Worker actifs.|
|Used Memory per Node|Charge sur un nœud Worker. Un nœud Worker sur lequel 10 Go de mémoire sont utilisés est considéré comme étant plus sollicité qu’un nœud avec 2 Go de mémoire utilisés.|
|Number of Application Masters per Node|Nombre de conteneurs Application Master (AM) en cours d’exécution sur un nœud Worker. Un nœud Worker hébergeant 2 conteneurs AM est considéré comme plus important qu’un nœud Worker hébergeant 0 conteneur AM.|

Les métriques ci-dessus sont contrôlées toutes les 60 secondes. Vous pouvez configurer les opérations de mise à l’échelle de votre cluster en employant l’une de ces métriques.

### <a name="load-based-scale-conditions"></a>Conditions de mise à l’échelle basée sur la charge

Lorsque les conditions suivantes sont détectées, la mise à l’échelle automatique émet une requête de mise à l’échelle :

|Scale-up|Scale-down|
|---|---|
|« Total pending CPU » est supérieur à la valeur de « Total free CPU » pendant plus de 3 minute.|« Total pending CPU » est inférieur à la valeur de « Total free CPU » pendant plus de 10 minutes.|
|« Total pending memory » est supérieur à la valeur de « Total free memory » pendant plus de 3 minute.|« Total pending memory » est inférieur à la valeur de « Total free memory » pendant plus de 10 minutes.|

Pour la montée en puissance, la mise à l’échelle automatique émet une demande de montée en puissance pour ajouter le nombre de nœuds requis. La montée en puissance est basée sur le nombre de nœuds Worker nécessaires pour répondre aux besoins actuels en matière d’UC et de mémoire.

Pour la descente en puissance, la mise à l’échelle automatique émet une demande de suppression d’un certain nombre de nœuds. La descente en puissance est basée sur le nombre de conteneurs AM par nœud. Et la configuration requise en matière d’UC et de mémoire. Le service détecte également les nœuds à supprimer en fonction de l’exécution des travaux en cours. L’opération de descente en puissance désactive tout d’abord les nœuds, puis les supprime du cluster.

### <a name="cluster-compatibility"></a>Compatibilité du cluster

> [!Important]
> La fonctionnalité de mise à l’échelle automatique d’Azure HDInsight a été mise à la disposition générale le 7 novembre 2019 pour les clusters Spark et Hadoop. Elle incluait des améliorations non disponibles dans la préversion de la fonctionnalité. Si vous avez créé un cluster Spark avant le 7 novembre 2019 et que vous souhaitez utiliser la fonctionnalité de mise à l’échelle automatique dessus, l’approche recommandée consiste à créer un nouveau cluster et à activer la mise à l’échelle automatique sur le nouveau cluster.
>
> La mise à l’échelle automatique pour Interactive Query (LLAP) a été publiée en vue d’une mise à disposition générale pour HDI 4.0 le 27 août 2020. Les clusters HBase sont toujours en préversion. La mise à l’échelle automatique est disponible uniquement sur les clusters Spark, Hadoop, Interactive Query et HBase.

Le tableau suivant décrit les types de cluster et les versions qui sont compatibles avec la fonctionnalité de mise à l’échelle automatique.

| Version | Spark | Hive | Interactive Query | hbase | Kafka | Storm | ML |
|---|---|---|---|---|---|---|---|
| HDInsight 3.6 sans ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 4.0 sans ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 3.6 avec ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |
| HDInsight 4.0 avec ESP | Oui | Oui | Oui | Oui* | Non | Non | Non |

\* Les clusters HBase peuvent uniquement être configurés pour une mise à l’échelle basée sur la planification, et non basée sur la charge.

## <a name="get-started"></a>Bien démarrer

### <a name="create-a-cluster-with-load-based-autoscaling"></a>Créer un cluster avec une mise à l’échelle automatique basée sur la charge

Pour activer la fonctionnalité de mise à l’échelle automatique basée sur la charge, procédez comme suit dans le cadre d’un processus normal de création de cluster :

1. Sous l’onglet **Configuration + tarification**, cochez la case **Activer la mise à l’échelle automatique**.
1. Sélectionnez **Basé sur la charge** sous **Type de mise à l’échelle automatique**.
1. Entrez les valeurs prévues pour les propriétés suivantes :  

    * **Nombre de nœuds** initial pour le **Nœud Worker**.
    * Nombre **min** de nœuds Worker.
    * Nombre **max** de nœuds Worker.

    ![Activation de l’option de mise à l’échelle automatique basée sur la charge du nœud Worker](./media/hdinsight-autoscale-clusters/azure-portal-cluster-configuration-pricing-autoscale.png)

Le nombre initial de nœuds Worker doit se situer entre le minimum et le maximum inclus. Cette valeur définit la taille initiale du cluster lors de sa création. Le nombre minimal de nœuds Worker doit être défini sur au moins trois. La mise à l’échelle de votre cluster à moins de trois nœuds peut entraîner le blocage en mode sans échec en raison d’une réplication de fichiers insuffisante.  Pour plus d’informations, consultez [Blocage en mode sans échec](./hdinsight-scaling-best-practices.md#getting-stuck-in-safe-mode).

### <a name="create-a-cluster-with-schedule-based-autoscaling"></a>Créer un cluster avec une mise à l’échelle automatique basée sur la planification

Pour activer la fonctionnalité de mise à l’échelle automatique basée sur la planification, procédez comme suit dans le cadre d’un processus normal de création de cluster :

1. Sous l’onglet **Configuration + tarification**, activez la case à cocher **Activer la mise à l’échelle automatique**.
1. Entrez le **Nombre de nœuds** pour le **Nœud Worker**, qui contrôle la limite de la mise à l’échelle du cluster.
1. Sélectionnez l’option **Basée sur la planification** sous **Type de mise à l’échelle**.
1. Sélectionnez **Configurer** pour ouvrir la fenêtre **Configuration de la mise à l’échelle automatique**.
1. Sélectionnez votre fuseau horaire, puis cliquez sur **+ Ajouter une condition**
1. Sélectionnez les jours de la semaine pour lesquels la nouvelle condition doit s’appliquer.
1. Modifiez l’heure à laquelle la condition doit prendre effet, ainsi que le nombre de nœuds du cluster à mettre à l’échelle.
1. Ajoutez d’autres conditions si nécessaire.

    ![Activation de l’option de création d’un nœud Worker basée sur la planification](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-schedule-creation.png)

Le nombre de nœuds doit être compris entre 3 et le nombre de nœuds Worker maximal que vous avez entré avant d’ajouter des conditions.

### <a name="final-creation-steps"></a>Dernières étapes de la création

Sélectionnez le type de machine virtuelle pour les nœuds Worker en choisissant une machine virtuelle dans la liste déroulante sous **Taille du nœud**. Après avoir sélectionné le type de machine virtuelle pour chaque type de nœud, vous pouvez voir la fourchette de coûts estimée pour l’ensemble du cluster. Ajustez les types de machine virtuelle selon votre budget.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Taille du nœud](./media/hdinsight-autoscale-clusters/azure-portal-cluster-configuration-pricing-vmsize.png)

Votre abonnement a un quota de capacité pour chaque région. Le nombre total de cœurs de vos nœuds principaux et le nombre maximum de nœuds Worker ne peuvent pas dépasser le quota de capacité. Toutefois, ce quota est une limite logicielle ; vous pouvez toujours créer un ticket de support pour l’augmenter aisément.

> [!Note]  
> Si vous dépassez la limite totale de quota de base, vous recevrez un message d’erreur disant indiquant que le nœud maximum a dépassé les noyaux disponibles dans cette région et que vous devez choisir une autre région ou contacter le support pour augmenter le quota (« the maximum node exceeded the available cores in this region, please choose another region or contact the support to increase the quota »).

Pour plus d’informations sur la création de clusters HDInsight à l’aide du portail Azure, consultez [Créer des clusters Linux dans HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md).  

### <a name="create-a-cluster-with-a-resource-manager-template"></a>Création d’un cluster avec un modèle Resource Manager

#### <a name="load-based-autoscaling"></a>Mise à l’échelle automatique basée sur la charge

Vous pouvez créer un cluster HDInsight avec la mise à l’échelle basée sur la charge d’un modèle Azure Resource Manager en ajoutant un nœud `autoscale` à la section `computeProfile` > `workernode` avec les propriétés `minInstanceCount` et `maxInstanceCount`, comme indiqué dans l’extrait de code json indiqué ci-dessous. Pour obtenir un modèle Resource Manager complet, consultez [Modèle de démarrage rapide : Déployer un cluster Spark avec mise à l’échelle automatique basée sur la charge](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-autoscale-loadbased).

```json
{
  "name": "workernode",
  "targetInstanceCount": 4,
  "autoscale": {
      "capacity": {
          "minInstanceCount": 3,
          "maxInstanceCount": 10
      }
  },
  "hardwareProfile": {
      "vmSize": "Standard_D13_V2"
  },
  "osProfile": {
      "linuxOperatingSystemProfile": {
          "username": "[parameters('sshUserName')]",
          "password": "[parameters('sshPassword')]"
      }
  },
  "virtualNetworkProfile": null,
  "scriptActions": []
}
```

#### <a name="schedule-based-autoscaling"></a>Mise à l’échelle automatique basée sur la planification

Vous pouvez créer un cluster HDInsight avec la mise à l’échelle basée sur la planification d’un modèle Azure Resource Manager en ajoutant un nœud `autoscale` à la section `computeProfile` > `workernode`. Le nœud `autoscale` contient un élément `recurrence` qui a un élément `timezone` et un élément `schedule`. Ils vous permettent d’indiquer quand la modification doit avoir lieu. Pour obtenir un modèle Resource Manager complet, consultez [Déployer un cluster Spark avec mise à l’échelle automatique basée sur la planification](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-autoscale-schedulebased).

```json
{
  "autoscale": {
    "recurrence": {
      "timeZone": "Pacific Standard Time",
      "schedule": [
        {
          "days": [
            "Monday",
            "Tuesday",
            "Wednesday",
            "Thursday",
            "Friday"
          ],
          "timeAndCapacity": {
            "time": "11:00",
            "minInstanceCount": 10,
            "maxInstanceCount": 10
          }
        }
      ]
    }
  },
  "name": "workernode",
  "targetInstanceCount": 4
}
```

### <a name="enable-and-disable-autoscale-for-a-running-cluster"></a>Activation et désactivation de la mise à l’échelle automatique d’un cluster en cours d’exécution

#### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Pour activer la mise à l’échelle automatique sur un cluster en cours d’exécution, sélectionnez **Taille du cluster** sous **Paramètres**. Ensuite, sélectionnez **Activer la mise à l’échelle automatique**. Choisissez un type de mise à l’échelle automatique, puis entrez les options de mise à l’échelle basée sur la planification ou la charge. Enfin, sélectionnez **Enregistrer**.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Cluster en cours d’exécution](./media/hdinsight-autoscale-clusters/azure-portal-settings-autoscale.png)

#### <a name="using-the-rest-api"></a>Utilisation de l’API REST

Pour activer ou désactiver la mise à l’échelle automatique sur un cluster en cours d’exécution à l’aide de l’API REST, envoyez une requête POST au point de terminaison de la mise à l’échelle automatique :

```
https://management.azure.com/subscriptions/{subscription Id}/resourceGroups/{resourceGroup Name}/providers/Microsoft.HDInsight/clusters/{CLUSTERNAME}/roles/workernode/autoscale?api-version=2018-06-01-preview
```

Utilisez les paramètres appropriés dans la charge utile de la requête. La charge utile JSON ci-dessous peut être utilisée pour activer la mise à l’échelle automatique. Utilisez la charge utile `{autoscale: null}` pour désactiver la mise à l’échelle automatique.

```json
{ "autoscale": { "capacity": { "minInstanceCount": 3, "maxInstanceCount": 5 } } }
```

Consultez la section précédente sur l’[activation de la mise à l’échelle automatique basée sur la charge](#load-based-autoscaling) pour obtenir une description complète de tous les paramètres de charge utile.

## <a name="monitoring-autoscale-activities"></a>Supervision des activités de mise à l’échelle automatique

### <a name="cluster-status"></a>État du cluster

L’état du cluster répertorié dans le Portail Microsoft Azure peut vous aider à surveiller les activités de mise à l’échelle automatique.

![Activation de l’option de mise à l’échelle automatique basée sur la charge du nœud worker - État du cluster](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-cluster-status.png)

La liste ci-dessous présente les messages d’état de cluster susceptibles de s’afficher.

| État du cluster | Description |
|---|---|
| Exécution en cours | Le cluster fonctionne normalement. Toutes les activités de mise à l’échelle automatique précédentes sont réussies. |
| Mise à jour  | La configuration de la mise à l’échelle automatique du cluster est en cours de mise à jour.  |
| Configuration de HDInsight  | Une opération de montée ou de descente en puissance de cluster est en cours.  |
| Erreur de mise à jour  | HDInsight a rencontré des problèmes pendant la mise à jour de la configuration de la mise à l’échelle automatique. Les clients peuvent choisir de réessayer la mise à jour ou de désactiver la mise à l’échelle automatique.  |
| Error  | Une erreur est survenue dans le cluster. Ce dernier n’est plus utilisable. Supprimez ce cluster et créez-en un autre.  |

Pour afficher le nombre actuel de nœuds dans votre cluster, accédez au graphique **Taille du cluster** sur la page **Vue d’ensemble** de votre cluster. Ou sélectionnez **Taille du cluster** sous **Paramètres**.

### <a name="operation-history"></a>Historique de l’opération

Vous pouvez afficher l’historique de Scale up/Scale down du cluster dans le cadre des métriques de celui-ci. Vous pouvez également dresser la liste de toutes les actions de mise à l’échelle au cours de la journée, de la semaine ou d’une autre période.

Sous **Supervision**, **Métriques**. Ensuite, sélectionnez **Ajouter une métrique** et **Nombre de Workers actifs** dans la zone de liste déroulante **Métrique**. Sélectionnez le bouton en haut à droite pour modifier l’intervalle de temps.

![Activation de l’option de mise à l’échelle d’un nœud worker basée sur la planification - Métrique](./media/hdinsight-autoscale-clusters/hdinsight-autoscale-clusters-chart-metric.png)

## <a name="best-practices"></a>Meilleures pratiques

### <a name="consider-the-latency-of-scale-up-and-scale-down-operations"></a>Prendre en compte la latence de la montée et de la descente en puissance

Une opération de mise à l’échelle peut prendre entre 10 et 20 minutes. Intégrez ce délai lorsque vous configurez une planification personnalisée. Par exemple, si vous avez besoin que la taille du cluster soit de 20 à 09h00, définissez le déclencheur de planification sur une heure antérieure, comme 08h30, pour que l’opération de mise à l’échelle se termine à 09h00.

### <a name="prepare-for-scaling-down"></a>Préparation pour la mise  à l’échelle vers le bas

Au cours de descente en puissance du cluster, la mise à l’échelle automatique désactivera les nœuds pour atteindre à la taille cible. Si des tâches sont en cours d’exécution sur ces nœuds, la mise à l’échelle automatique attendra qu’elles soient terminées pour les clusters Spark et Hadoop. Dans la mesure où chaque nœud Worker joue également un rôle dans HDFS, les données temporaires seront décalées vers les nœuds restants. Par conséquent, nous vous conseillons de vérifier que les nœuds restants disposent d’assez d’espace de stockage pour héberger toutes les données temporaires.

Les travaux en cours d’exécution se poursuivront. Les travaux en attente attendrons d’être planifiés avec moins de nœuds Worker disponibles.

### <a name="be-aware-of-the-minimum-cluster-size"></a>Tenez compte de la taille minimale du cluster

Ne mettez pas à l’échelle votre cluster à moins de trois nœuds. La mise à l’échelle de votre cluster à moins de trois nœuds peut entraîner le blocage en mode sans échec en raison d’une réplication de fichiers insuffisante. Pour plus d’informations, consultez [Blocage en mode sans échec](hdinsight-scaling-best-practices.md#getting-stuck-in-safe-mode).

### <a name="increase-the-number-of-mappers-and-reducers"></a>Augmentez le nombre de mappeurs et de réducteurs

La mise à l’échelle automatique pour les clusters Hadoop surveille également l’utilisation de HDFS. Si le HDFS est occupé, elle suppose que le cluster a toujours besoin des ressources actuelles. Lorsque des données volumineuses sont impliquées dans la requête, vous pouvez augmenter le nombre de mappeurs et de réducteurs pour augmenter le parallélisme et accélérer les opérations HDFS. De cette façon, la mise à l’échelle appropriée est déclenchée lorsque des ressources supplémentaires sont disponibles. 

### <a name="set-the-hive-configuration-maximum-total-concurrent-queries-for-the-peak-usage-scenario"></a>Définissez le nombre maximal de requêtes simultanées dans la configuration Hive pour le scénario d’utilisation maximale

Les événements de mise à l’échelle automatique ne modifient pas la configuration Hive *Nombre total maximum de requêtes simultanées* dans Ambari. Cela signifie que le service interactif Hive Server 2 ne peut traiter qu’un nombre déterminé de requêtes simultanées à un moment donné, même si le nombre de démons Interactive Query est augmenté et réduit en fonction de la charge et de la planification. La recommandation générale consiste à définir cette configuration pour le scénario d’utilisation maximale afin d’éviter toute intervention manuelle.

Toutefois, vous pouvez rencontrer un échec de redémarrage du serveur Hive 2 s’il n’y a qu’un petit nombre de nœuds Worker et que le nombre maximum total de requêtes simultanées est configuré sur une valeur trop élevée. Au minimum, vous avez besoin du nombre minimal de nœuds Worker pouvant prendre en charge le nombre donné d’AM Tez (égal à la configuration du maximum total de requêtes simultanées). 

## <a name="limitations"></a>Limites

### <a name="node-label-file-missing"></a>Fichier d’étiquette de nœud manquant

La mise à l’échelle automatique HDInsight utilise un fichier d’étiquette de nœud pour déterminer si un nœud est prêt à exécuter des tâches. Le fichier d’étiquette de nœud est stocké sur HDFS avec trois réplicas. Si la taille du cluster est considérablement réduite et qu’il y a une grande quantité de données temporaires, les trois réplicas peuvent être supprimés. Dans ce cas, le cluster passe en état d’erreur.

### <a name="interactive-query-daemons-count"></a>Nombre de démons Interactive Query

En cas de clusters Interactive Query avec mise à l’échelle automatique, un événement de scale-up/scale-down automatique augmente ou diminue également le nombre de démons Interactive Query en fonction du nombre de nœuds Worker actifs. La modification du nombre de démons n’est pas conservée dans la configuration `num_llap_nodes` dans Ambari. Si les services Hive sont redémarrés manuellement, le nombre de démons Interactive Query est réinitialisé conformément à la configuration dans Ambari.

Si le service Interactive Query est redémarré manuellement, vous devez modifier manuellement la configuration `num_llap_node` (nombre de nœuds nécessaires pour exécuter le démon Interactive Query Hive) sous *Advanced hive-interactive-env* pour correspondre au nombre de nœuds de Worker actifs.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les instructions de mise à l’échelle manuelle des clusters dans [Instructions de mise à l’échelle](hdinsight-scaling-best-practices.md).
