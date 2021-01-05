---
title: Exception lors de l'exécution de requêtes depuis la vue Apache Ambari Hive dans Azure HDInsight
description: Étapes de dépannage lors de l'exécution de requêtes Apache Hive via la vue Apache Ambari Hive dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 12/23/2019
ms.openlocfilehash: aeedda5c26a2e9dc0fa2b228285cfda45d880d29
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547382"
---
# <a name="exception-when-running-queries-from-apache-ambari-hive-view-in-azure-hdinsight"></a>Exception lors de l'exécution de requêtes depuis la vue Apache Ambari Hive dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Lorsque vous exécutez une requête Apache Hive depuis la vue Apache Ambari Hive, vous recevez par intermittence le message d'erreur suivant :

```error
Cannot create property 'errors' on string '<!DOCTYPE html PUBLIC '-//W3C//DTD XHTML 1.0 Strict//EN' 'http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd'>
<html xmlns='http://www.w3.org/1999/xhtml'>
<head>
<title>IIS 8.5 Detailed Error - 502.3 - Bad Gateway</title>
```

## <a name="cause"></a>Cause :

Un dépassement du délai de la passerelle.

Le délai d’expiration de la passerelle est de 2 minutes. Les requêtes de la vue Ambari Hive sont soumises au point de terminaison `/hive2` via la passerelle. Une fois la requête compilée et acceptée avec succès, HiveServer renvoie un `queryid`. Les clients continuent d’interroger le statut de la requête. Pendant ce processus, si HiveServer ne renvoie pas de réponse HTTP dans les 2 minutes, la passerelle HDI envoie à l'appelant une erreur de dépassement du délai de la passerelle 502.3. Les erreurs peuvent se produire lorsque la requête est soumise pour traitement (plus probable) et aussi pendant l'appel pour obtenir l’état (moins probable). Les utilisateurs peuvent recevoir l'une ou l'autre de ces erreurs.

Le thread du gestionnaire http est censé être rapide : préparer la tâche et renvoyer un `queryid`. Cependant, pour diverses raisons, tous les threads du gestionnaire peuvent être occupés, ce qui entraîne des délais d'expiration pour les nouvelles requêtes et les appels pour obtenir l’état.

### <a name="responsibilities-of-the-http-handler-thread"></a>Responsabilités du thread du gestionnaire HTTP

Lorsque le client soumet une requête à HiveServer, il effectue les tâches suivantes dans le thread de premier plan :

* Analyser la requête, puis effectuer une vérification sémantique
* Acquérir le verrouillage
* Rechercher le metastore si nécessaire
* Compiler la requête (DDL ou DML)
* Préparer un plan de requête
* Effectuer l'autorisation (exécuter toutes les stratégies de ranger applicables dans les clusters sécurisés)

## <a name="resolution"></a>Résolution

Voici quelques recommandations générales pour améliorer la situation :

* Si vous utilisez un Metastore Hive externe, vérifiez les métriques de la base de données et assurez-vous que la base de données n'est pas surchargée. Vous pouvez mettre à l'échelle la couche de la base de données du metastore.

* Assurez-vous que les opérations parallèles sont activées (cela permet aux threads du gestionnaire HTTP de s’exécuter en parallèle). Pour vérifier la valeur, lancez [Apache Ambari](../hdinsight-hadoop-manage-ambari.md) et accédez à **Hive** > **Configs** > **Avanced (Avancé)**  > **Custom hive-site (hive-site personnalisé)** . La valeur de `hive.server2.parallel.ops.in.session` doit être `true`.

* Vérifiez que la référence SKU de la machine virtuelle du cluster n’est pas trop petite pour la charge. Vous pouvez répartir le travail entre plusieurs groupes. Pour plus d’informations, consultez [Choisir un type de cluster](../hdinsight-capacity-planning.md#choose-a-cluster-type).

* Si Ranger est installé sur le cluster, vérifiez qu’il n'y a pas trop de stratégies Ranger à évaluer pour chaque requête. Recherchez les stratégies en double ou inutiles.

* Vérifiez la **taille des segments de mémoire HiveServer2** dans Ambari. Accédez à **Hive** > **Configs** > **Settings (Paramètres)**  > **Optimization (Optimisation)** . Assurez-vous que la valeur est supérieure à 10 Go. Ajustez au besoin cette valeur pour optimiser les performances.

* Assurez-vous que la requête Hive est correctement définie. Pour plus d’informations, consultez [Optimiser les requêtes Apache Hive dans Azure HDInsight](../hdinsight-hadoop-optimize-hive-query.md).

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).