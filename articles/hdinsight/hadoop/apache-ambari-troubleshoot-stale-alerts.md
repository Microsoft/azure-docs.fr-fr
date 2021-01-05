---
title: Alertes obsolètes Apache Ambari dans Azure HDInsight
description: Discussion et analyse des causes et solutions possibles pour les alertes obsolètes Apache Ambari dans HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 01/22/2020
ms.openlocfilehash: 9564e149055dcabf205694b3cf6c3ddc03163037
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533833"
---
# <a name="scenario-apache-ambari-stale-alerts-in-azure-hdinsight"></a>Scénario : Alertes obsolètes Apache Ambari dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Dans l’interface utilisateur Apache Ambari, vous pouvez voir une alerte semblable à celle-ci :

![Exemple d’alerte obsolète Apache Ambari](./media/apache-ambari-troubleshoot-stale-alerts/ambari-stale-alerts-example.png)

## <a name="cause"></a>Cause

Les agents Ambari surveillent en permanence l’intégrité d’un grand nombre de ressources. Les *alertes* peuvent être configurées pour vous signaler si des propriétés spécifiques du cluster respectent ou non des seuils prédéterminés. Après chaque vérification de ressource, si la condition d’alerte est remplie, les agents Ambari signalent l’état au serveur Ambari et déclenchent une alerte. Si une alerte n’est pas vérifiée dans l’intervalle défini dans son profil d’alerte, le serveur déclenche une alerte *Alertes obsolètes du serveur Ambari* .

Il existe plusieurs raisons pour lesquelles une vérification d’intégrité peut ne pas s’exécuter à l’intervalle défini :

* Les hôtes sont très sollicités (utilisation élevée de l’UC), de sorte que l’agent Ambari ne dispose pas de suffisamment de ressources système pour exécuter les alertes à temps.

* Le cluster est occupé à exécuter de nombreux travaux ou services pendant une période de charge importante.

* Un petit nombre d’hôtes dans le cluster hébergent un grand nombre de composants et sont donc requis pour exécuter un grand nombre d’alertes. Si le nombre de composants est important, les tâches d’alerte risquent de ne pas respecter les intervalles planifiés.

## <a name="resolution"></a>Résolution

Essayez les méthodes suivantes pour résoudre les problèmes liés aux alertes obsolètes Ambari.

### <a name="increase-the-alert-interval-time"></a>Augmenter la durée d’intervalle d’une alerte

Vous pouvez augmenter la valeur d’intervalle d’une alerte individuelle en fonction du temps de réponse de votre cluster et de sa charge :

1. Dans l’interface utilisateur Apache Ambari, sélectionnez l’onglet **Alertes** .
1. Sélectionnez le nom de la définition d’alerte de votre choix.
1. Dans la définition, sélectionnez **Modifier** .
1. Augmentez la valeur **Intervalle de vérification** , puis sélectionnez **Enregistrer** .

### <a name="increase-the-alert-interval-time-for-ambari-server-alerts"></a>Augmenter la durée d’intervalle d’une alerte pour les alertes du serveur Ambari

1. Dans l’interface utilisateur Apache Ambari, sélectionnez l’onglet **Alertes** .
1. Dans la liste déroulante **Groupes** , sélectionnez **AMBARI par défaut** .
1. Sélectionnez l’alerte **Alertes du serveur Ambari** .
1. Dans la définition, sélectionnez **Modifier** .
1. Augmentez la valeur **Intervalle de vérification** .
1. Augmentez la valeur **Multiplicateur d’intervalle** , puis sélectionnez **Enregistrer** .

### <a name="disable-and-reenable-the-alert"></a>Désactiver et réactiver l’alerte

Pour ignorer une alerte obsolète, désactivez-la, puis réactivez-la :

1. Dans l’interface utilisateur Apache Ambari, sélectionnez l’onglet **Alertes** .
1. Sélectionnez le nom de la définition d’alerte de votre choix.
1. Dans la définition, sélectionnez l’option **Activée** dans la partie la plus à droite de l’interface utilisateur.
1. Dans la fenêtre contextuelle **Confirmation** , sélectionnez **Confirmer la désactivation** .
1. Patientez quelques secondes pour que toutes les « instances » d’alerte affichées sur la page soient effacées.
1. Dans la définition, sélectionnez l’option **Désactivée** dans la partie la plus à droite de l’interface utilisateur.
1. Dans la fenêtre contextuelle **Confirmation** , sélectionnez **Confirmer l’activation** .

### <a name="increase-the-alert-grace-period"></a>Augmenter la période de grâce des alertes

Avant que l’agent Ambari ne signale qu’une alerte configurée a manqué sa planification, une période de grâce est appliquée. Si l’alerte n’a pas eu lieu à l’heure prévue, mais qu’elle a été exécutée pendant la période de grâce, l’alerte obsolète n’est pas générée.

La valeur `alert_grace_period` par défaut est 5 secondes. Vous pouvez configurer ce paramètre dans /etc/ambari-agent/conf/ambari-agent.ini. Pour les hôtes sur lesquels des alertes obsolètes se produisent à intervalles réguliers, augmentez la valeur à 10 pour essayer. Ensuite, redémarrer l’agent Ambari.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, rendez-vous sur l’un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Rendez-vous sur la page [@AzureSupport](https://twitter.com/azuresupport) sur Twitter. Il s’agit du compte Microsoft Azure officiel pour améliorer l’expérience client. Il fournit à la communauté Azure les ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, envoyez une demande de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Pour y accéder, sélectionnez l’aide ( **?** ) dans le menu du portail ou ouvrez le volet **Aide + support** . Pour plus d’informations, consultez la page [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). 

  Le support pour la gestion et la facturation des abonnements est inclus dans votre abonnement Microsoft Azure. Le support technique est disponible via les [plans de support Azure](https://azure.microsoft.com/support/plans/).