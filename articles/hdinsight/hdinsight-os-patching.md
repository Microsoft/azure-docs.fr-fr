---
title: Configurer la planification de la mise à jour corrective du système d’exploitation pour les clusters Azure HDInsight
description: Découvrez comment configurer la planification de la mise à jour corrective du système d’exploitation pour les clusters HDInsight sous Linux.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/21/2020
ms.openlocfilehash: 7b789bf01a043b167d6740f09df935d9b683c48f
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357757"
---
# <a name="configure-the-os-patching-schedule-for-linux-based-hdinsight-clusters"></a>Configurer la planification de la mise à jour corrective du système d’exploitation pour les clusters HDInsight sous Linux

> [!IMPORTANT]
> Les images Ubuntu sont accessibles pour la création d’un nouveau cluster Azure HDInsight dans les trois mois qui suivent leur publication. Plus aucun correctif automatique n'est disponible pour les clusters en cours. Les clients doivent utiliser des actions de script ou d'autres mécanismes pour corriger un cluster en cours d'exécution. Une bonne pratique consiste à exécuter ces actions de script et à appliquer les mises à jour de sécurité juste après la création du cluster.

HDInsight prend en charge l’exécution des tâches courantes sur votre cluster, telles que l’installation des correctifs du système d’exploitation, les mises à jour de sécurité et le redémarrage des nœuds. Ces tâches sont accomplies à l’aide des deux scripts suivants qui peuvent être exécutés en tant qu’[actions de script](hdinsight-hadoop-customize-cluster-linux.md) et configurés avec les paramètres :

- `schedule-reboots.sh` : effectuez un redémarrage immédiat ou planifiez un redémarrage sur les nœuds de cluster.
- `install-updates-schedule-reboots.sh` : installez toutes les mises à jour, les mises à jour du noyau et de la sécurité ou les mises à jour du noyau uniquement.

> [!NOTE]  
> Ces action de script n’appliquent pas automatiquement les mises à jour pour tous les cycles de mise à jour ultérieurs. Exécutez les scripts chaque fois que de nouvelles mises à jour doivent être appliquées pour installer les mises à jour et redémarrez la machine virtuelle.

## <a name="preparation"></a>Préparation

Correctif sur un environnement de non-production représentatif avant le déploiement en production. Élaborez un plan pour tester votre système de manière adéquate avant d’appliquer les correctifs.

De temps en temps, lors d’une session SSH avec votre cluster, vous pouvez recevoir un message indiquant que des mises à jour de sécurité sont disponibles. Le message peut ressembler à ceci :

```
89 packages can be updated.
82 updates are security updates.

*** System restart required ***

Welcome to Spark on HDInsight.

```

L’application de correctifs est facultative et à votre convenance.

## <a name="restart-nodes"></a>Redémarrer des nœuds
  
Le script [schedule-reboots](https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv02/schedule-reboots.sh) définit le type de redémarrage qui sera effectué sur les ordinateurs du cluster. Lors de l’envoi de l’action de script, configurez-la pour qu’elle s’applique aux trois types de nœuds : nœud principal, nœud Worker et Zookeeper. Si le script n’est pas appliqué à un type de nœud, les machines virtuelles de ce type de nœud ne seront pas mises à jour ni redémarrées.

Le `schedule-reboots script` accepte un paramètre numérique :

| Paramètre | Valeurs acceptées | Définition |
| --- | --- | --- |
| Type de redémarrage à effectuer | 1 ou 2 | La valeur 1 active le redémarrage planifié (entre 12 et 24 heures). La valeur 2 active le redémarrage immédiat (dans les 5 minutes). Si aucun paramètre n’est fourni, la valeur par défaut est 1. |  

## <a name="install-updates-and-restart-nodes"></a>Installer des mises à jour et redémarrer des nœuds

Le script [install-updates-schedule-reboots.sh](https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv02/install-updates-schedule-reboots.sh) fournit des options d’installation de différents types de mises à jour et de redémarrage de la machine virtuelle.

Le script `install-updates-schedule-reboots` accepte deux paramètres numériques, comme décrit dans le tableau suivant :

| Paramètre | Valeurs acceptées | Définition |
| --- | --- | --- |
| Type de mises à jour à installer | 0, 1 ou 2 | La valeur 0 installe uniquement les mises à jour du noyau. La valeur 1 installe le noyau et les mises à jour de sécurité, et la valeur 2 installe toutes les mises à jour. Si aucun paramètre n’est fourni, la valeur par défaut est 0. |
| Type de redémarrage à effectuer | 0, 1 ou 2 | La valeur 0 désactive le redémarrage. La valeur 1 active le redémarrage planifié et la valeur 2 active le redémarrage immédiat. Si aucun paramètre n’est fourni, la valeur par défaut est 0. L’utilisateur doit modifier le paramètre 1 pour entrer le paramètre 2. |

> [!NOTE]
> Vous devez marquer un script comme persistant lorsque vous l’appliquez à un cluster existant. Sinon, les nœuds créés lors d’opérations de mise à l’échelle utilisent la planification de mise à jour corrective par défaut. Si vous appliquez le script dans le cadre du processus de création du cluster, il est automatiquement défini comme persistant.

> [!NOTE]
> L’option de redémarrage planifié effectue un redémarrage progressif et automatisé des nœuds de cluster corrigés sur une période de 12 à 24 heures et prend en compte les considérations relatives à la haute disponibilité, au domaine de mise à jour et au domaine d’erreur. Le redémarrage planifié n’interrompt pas l’exécution des charges de travail, mais peut réduire la capacité du cluster provisoirement quand des nœuds ne sont pas disponibles, ce qui entraîne des temps de traitement plus longs. 

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les étapes spécifiques à l’utilisation de l’action de script, consultez les sections suivantes de l’article [Personnalisation de clusters HDInsight basés sur Linux à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md) :

- [Utiliser une action de script lors de la création du cluster](hdinsight-hadoop-customize-cluster-linux.md#script-action-during-cluster-creation)
- [Appliquer une action de script sur un cluster en cours d’exécution](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster)
