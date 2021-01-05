---
title: Analyse des performances avec les journaux Azure Monitor
description: Découvrez comment configurer l’agent Log Analytics de façon à effectuer le monitoring des conteneurs et des compteurs de performances de vos clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 04/16/2018
ms.author: srrengar
ms.openlocfilehash: b91a8a8742a5bdc9454ebcbd8894889084a12a79
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258678"
---
# <a name="performance-monitoring-with-azure-monitor-logs"></a>Analyse des performances avec les journaux Azure Monitor

Cet article explique comment ajouter l’agent Log Analytics à un cluster sous la forme d’une extension du groupe de machines virtuelles identiques, puis le connecter à un espace de travail Azure Log Analytics existant. Cela permet de collecter des données de diagnostic sur les conteneurs, les applications et les performances. Si vous l’ajoutez en tant qu’extension à la ressource de groupe de machines virtuelles identiques, Azure Resource Manager veille à l’installer sur tous les nœuds, même lors de la mise à l’échelle du cluster.

> [!NOTE]
> Cet article suppose que vous disposez d’un espace de travail Azure Log Analytics déjà configuré. Dans le cas contraire, voir [Configurer les journaux Azure Monitor](service-fabric-diagnostics-oms-setup.md).

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="add-the-agent-extension-via-azure-cli"></a>Ajouter l’extension d’agent via Azure CLI

La meilleure façon d’ajouter l’agent Log Analytics à un cluster est d’utiliser les API des groupes de machines virtuelles identiques avec Azure CLI. Si vous n’avez pas encore configuré Azure CLI, accédez au portail Azure, puis ouvrez une instance [Cloud Shell](../cloud-shell/overview.md) ou [installez Azure CLI](/cli/azure/install-azure-cli).

1. Lorsque votre instance de Cloud Shell est demandée, veillez à vous trouver dans le même abonnement que votre ressource. Utilisez la commande `az account show` pour vérifier que la valeur « name » correspond à celle de l’abonnement de votre cluster.

2. Dans le Portail, accédez au groupe de ressources où se trouve votre espace de travail Log Analytics. Cliquez sur la ressource Log Analytics (son type sera celui d’un espace de travail Log Analytics). Une fois que vous vous trouvez sur la page de vue d’ensemble des ressources, cliquez sur **Paramètres avancés** dans la section Paramètres du menu de gauche.

    ![Page de propriétés Log Analytics](media/service-fabric-diagnostics-oms-agent/oms-advanced-settings.png)

3. Cliquez sur **Serveurs Windows** si vous créez un cluster Windows, ou sur **Serveurs Linux** si vous créez un cluster Linux. Cette page affiche les éléments `workspace ID` et `workspace key` (répertoriés en tant que clé primaire dans le portail). Vous aurez besoin de ces éléments pour l’étape suivante.

4. Exécutez la commande pour installer l’agent Log Analytics sur votre cluster en utilisant l’API `vmss extension set` :

    Pour un cluster Windows :

    ```azurecli
    az vmss extension set --name MicrosoftMonitoringAgent --publisher Microsoft.EnterpriseCloud.Monitoring --resource-group <nameOfResourceGroup> --vmss-name <nameOfNodeType> --settings "{'workspaceId':'<Log AnalyticsworkspaceId>'}" --protected-settings "{'workspaceKey':'<Log AnalyticsworkspaceKey>'}"
    ```

    Pour un cluster Linux :

    ```azurecli
    az vmss extension set --name OmsAgentForLinux --publisher Microsoft.EnterpriseCloud.Monitoring --resource-group <nameOfResourceGroup> --vmss-name <nameOfNodeType> --settings "{'workspaceId':'<Log AnalyticsworkspaceId>'}" --protected-settings "{'workspaceKey':'<Log AnalyticsworkspaceKey>'}"
    ```

    Voici un exemple illustrant l’ajout de l’agent Log Analytics à un cluster Windows.

    ![Commande CLI de l’agent Log Analytics](media/service-fabric-diagnostics-oms-agent/cli-command.png)

5. Normalement, l’ajout de l’agent aux nœuds prend moins de 15 minutes. Vous pouvez vérifier que les agents ont bien été ajoutés à l’aide de l’API `az vmss extension list` :

    ```azurecli
    az vmss extension list --resource-group <nameOfResourceGroup> --vmss-name <nameOfNodeType>
    ```

## <a name="add-the-agent-via-the-resource-manager-template"></a>Ajouter l’agent en utilisant un modèle Resource Manager

Des exemples de modèles Gestionnaire des ressources qui déploient un espace de travail Azure Log Analytics et ajoutent un agent à chacun de vos nœuds sont disponibles pour [Windows](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure) ou [Linux](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeType-Secure-OMS).

Vous pouvez télécharger et modifier ces modèles pour déployer un cluster qui correspond mieux à vos besoins.

## <a name="view-performance-counters"></a>Afficher les compteurs de performances

Maintenant que vous avez ajouté l’agent Log Analytics, accédez au portail Log Analytics pour choisir les compteurs de performances à collecter.

1. Dans le portail Azure, accédez au groupe de ressources dans lequel vous avez créé la solution Service Fabric Analytics. Sélectionnez **ServiceFabric\<nameOfLog AnalyticsWorkspace\>** .

2. Cliquez sur **Log Analytics**.

3. Cliquez sur **Paramètres avancés**.

4. Cliquez sur **Données**, puis sur **Compteurs de performances Windows ou Linux**. Une liste des compteurs par défaut que vous pouvez choisir d’activer s’affiche, et vous pouvez également définir l’intervalle de collecte. En outre, vous pouvez ajouter [d’autres compteurs de performances](service-fabric-diagnostics-event-generation-perf.md) à collecter. Le format approprié est référencé dans cet [article](/windows/win32/perfctrs/specifying-a-counter-path).

5. Cliquez sur **Enregistrer**, puis sur **OK**.

6. Fermez le panneau Paramètres avancés.

7. Sous le titre Général, cliquez sur **Récapitulatif de l’espace de travail**.

8. Des vignettes représentant un graphique s’affichent pour chacune des solutions activées, y compris pour Service Fabric. Cliquez sur le graphique **Service Fabric** pour accéder à la solution Service Fabric Analytics.

9. Vous pouvez voir quelques vignettes avec des graphiques sur le canal opérationnel et les événements de services fiables. La représentation graphique du flux de données correspondant aux compteurs que vous avez sélectionnés s’affichent dans Métriques de nœud.

10. Cliquez sur un graphique de métrique de conteneur pour afficher les détails supplémentaires. Vous pouvez également interroger les données du compteur de performances de la même façon que les événements de cluster et appliquer des filtres sur les nœuds, le nom du compteur de performances et les valeurs à l’aide du langage de requête Kusto.

![Interroger le compteur de performances Log Analytics](media/service-fabric-diagnostics-event-analysis-oms/oms_node_metrics_table.PNG)

## <a name="next-steps"></a>Étapes suivantes

* Collectez les [compteurs de performances](service-fabric-diagnostics-event-generation-perf.md) dont vous avez besoin. Pour configurer l’agent Log Analytics de façon à collecter certains compteurs de performances, voir [Configurer les sources de données](../azure-monitor/platform/agent-data-sources.md#configuring-data-sources).
* Configurez les journaux Azure Monitor de façon à paramétrer [l’alerte automatisée](../azure-monitor/platform/alerts-overview.md) pour faciliter la détection et les diagnostics.
* En guise d’alternative, vous pouvez collecter des compteurs de performances via [l’extension Diagnostics Azure et les envoyer à Application Insights](service-fabric-diagnostics-event-aggregation-wad.md#add-the-application-insights-sink-to-the-resource-manager-template)
