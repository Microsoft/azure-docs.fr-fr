---
title: Agrégation d’événements avec Diagnostics Windows Azure
description: Découvrez comment agréger et collecter des événements à l’aide des diagnostics Windows Azure pour la surveillance et le diagnostic de clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 04/03/2018
ms.author: srrengar
ms.openlocfilehash: bcb9ca9e73c0898dc778202eca036a5ae92bebf8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87076135"
---
# <a name="event-aggregation-and-collection-using-windows-azure-diagnostics"></a>Agrégation et collecte d’événements à l’aide de Diagnostics Azure pour Windows
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-event-aggregation-wad.md)
> * [Linux](service-fabric-diagnostics-event-aggregation-lad.md)
>
>

Lorsque vous exécutez un cluster Service Fabric dans Azure, il peut être intéressant de collecter les journaux d’activité de tous les nœuds pour les regrouper dans un emplacement central. La centralisation des journaux d’activité vous permet d’analyser et résoudre les problèmes que vous pourriez rencontrer dans votre cluster ou dans les applications et services exécutés dans ce cluster.

Pour charger et collecter des journaux, vous pouvez utiliser l’extension Diagnostics Azure pour Windows, qui télécharge les journaux dans Stockage Azure, ou envoyer les journaux à Azure Application Insights ou à des hubs d’événements. Vous pouvez également utiliser un processus externe pour lire les événements à partir du stockage et les placer dans une plateforme d'analyse comme les [journaux Azure Monitor](./service-fabric-diagnostics-oms-setup.md) ou une autre solution d'analyse des journaux.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis
Cet article fait référence aux outils suivants :

* [Azure Resource Manager](../azure-resource-manager/management/overview.md)
* [Azure PowerShell](/powershell/azure/)
* [Modèle Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)

## <a name="service-fabric-platform-events"></a>Événements de la plateforme Service Fabric
Service Fabric inclut quelques [canaux de journalisation prêts à l’emploi](service-fabric-diagnostics-event-generation-infra.md), notamment les canaux ci-après, qui sont préconfigurés avec l’extension pour l’envoi de données de surveillance et de diagnostic à une table de stockage ou à un autre emplacement :
  * [Événements opérationnels](service-fabric-diagnostics-event-generation-operational.md) : opérations de niveau supérieur effectuées par la plateforme Service Fabric. Par exemple : la création d’applications et de services, les modifications d’état des nœuds et les informations de mise à niveau. Ces événements sont émis sous forme de journaux d’activité de suivi d’événements pour Windows (ETW)
  * [Événements du modèle de programmation Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
  * [Événements du modèle de programmation Reliable Services](service-fabric-reliable-services-diagnostics.md)

## <a name="deploy-the-diagnostics-extension-through-the-portal"></a>Déployer l’extension Diagnostics par le biais du portail
La première étape de la collecte de journaux d’activité consiste à déployer l’extension Diagnostics sur les nœuds du groupe de machines virtuelles identiques du cluster Service Fabric. Cette extension collecte les journaux d’activité sur chaque machine virtuelle et les charge dans le compte de stockage que vous spécifiez. Les étapes ci-après vous indiquent comment accomplir cette tâche pour les clusters nouveaux et existants par le biais du Portail Azure et des modèles Azure Resource Manager.

### <a name="deploy-the-diagnostics-extension-as-part-of-cluster-creation-through-azure-portal"></a>Déployer l’extension Diagnostics dans le cadre de la création d’un cluster via le portail Azure
Lorsque vous créez votre cluster, dans le cadre de la configuration de ce dernier, développez les paramètres facultatifs et assurez-vous que Diagnostics est défini sur **Actif** (paramétrage par défaut).

![Paramètres Diagnostics Azure dans le portail pour la création d’un cluster](media/service-fabric-diagnostics-event-aggregation-wad/azure-enable-diagnostics-new.png)

Nous vous recommandons vivement de télécharger le modèle **avant de cliquer sur Créer** à l’étape finale. Pour plus de détails, voir [Configurer un cluster Service Fabric à l’aide d’un modèle Azure Resource Manager](service-fabric-cluster-creation-via-arm.md). Vous devez disposer du modèle pour modifier les canaux (répertoriés ci-dessus) à partir desquels vous souhaitez collecter des données.

![Modèle de cluster](media/service-fabric-diagnostics-event-aggregation-wad/download-cluster-template.png)

Une fois que vous avez entrepris d’agréger des événements dans le service Stockage Azure, [configurez les journaux d’activité Azure Monitor](service-fabric-diagnostics-oms-setup.md) afin d’obtenir des informations détaillées et de les interroger dans le portail des journaux d’activité Azure Monitor.

>[!NOTE]
>Pour l’instant, il n’existe aucun moyen de filtrer ou de nettoyer les événements qui sont envoyés aux tables. Si vous n’implémentez aucun processus de suppression des événements de la table, la table continuera à croître (la limite par défaut est de 50 Go). La procédure de modification de ce comportement est décrite [dans la suite de cet article](service-fabric-diagnostics-event-aggregation-wad.md#update-storage-quota). En outre, un exemple de service de nettoyage de données en cours d’exécution est inclus dans [l’échantillon Watchdog](https://github.com/Azure-Samples/service-fabric-watchdog-service), et il est recommandé d’en écrire un vous-même, sauf si vous avez une bonne raison de stocker les journaux d’activité au-delà de 30 ou 90 jours.



## <a name="deploy-the-diagnostics-extension-through-azure-resource-manager"></a>Déployer l’extension Diagnostics par le biais d’Azure Resource Manager

### <a name="create-a-cluster-with-the-diagnostics-extension"></a>Créer un cluster avec l’extension Diagnostics
Pour créer un cluster avec Resource Manager, vous devez ajouter le fichier de configuration JSON Diagnostics au modèle Resource Manager complet. Nous fournissons un exemple de modèle Resource Manager de cluster à cinq machines virtuelles avec la configuration Diagnostics ajoutée dans le cadre de nos exemples de modèle Resource Manager. Vous pouvez le voir à cet emplacement dans la galerie d’exemples Azure : [Cluster à cinq nœuds avec un exemple de modèle Diagnostics Resource Manager](https://azure.microsoft.com/resources/templates/service-fabric-secure-cluster-5-node-1-nodetype/).

Pour voir les paramètres de diagnostic dans le modèle Resource Manager, ouvrez le fichier azuredeploy.json et recherchez **IaaSDiagnostics**. Pour créer un cluster à l’aide de ce modèle, cliquez sur le bouton **Déployer sur Azure**, disponible via le lien précédent.

Vous pouvez également télécharger l’exemple Resource Manager, y apporter des modifications et créer un cluster à partir du modèle modifié en utilisant la commande `New-AzResourceGroupDeployment` dans une fenêtre Azure PowerShell. Consultez le code suivant pour les paramètres que vous passez à la commande. Pour plus d’informations sur le déploiement d’un groupe de ressources à l’aide de PowerShell, consultez l’article [Déployer un groupe de ressources avec un modèle Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

### <a name="add-the-diagnostics-extension-to-an-existing-cluster"></a>Ajouter l’extension Diagnostics à un cluster existant
Si l’extension Diagnostics n’est pas déployée sur l’un de vos clusters existants, vous pouvez l’ajouter ou la mettre à jour par le biais du modèle de cluster. Modifiez le modèle Resource Manager utilisé pour créer le cluster existant ou téléchargez le modèle sur le portail, comme décrit ci-dessus. Modifiez le fichier template.json en effectuant les opérations suivantes :

Ajouter une ressource de stockage au modèle en l’ajoutant à la section de ressources.

```json
{
    "apiVersion": "2018-07-01",
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
    "name": "[parameters('applicationDiagnosticsStorageAccountType')]"
    "tier": "standard"
  },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
```

 Ensuite, ajoutez-la à la section parameters juste après les définitions de compte de stockage, entre `supportLogStorageAccountName`. Remplacez le texte de l’espace réservé *storage account name goes here* par le nom du compte de stockage de votre choix.

```json
    "applicationDiagnosticsStorageAccountType": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "Replication option for the application diagnostics storage account"
      }
    },
    "applicationDiagnosticsStorageAccountName": {
      "type": "string",
      "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
      "metadata": {
        "description": "Name for the storage account that contains application diagnostics data from the cluster"
      }
    },
```
Mettez ensuite à jour la section `VirtualMachineProfile` du fichier template.json en ajoutant le code suivant dans le tableau extensions. N’oubliez pas d’ajouter une virgule au début ou à la fin, en fonction de l’emplacement d’insertion.

```json
{
    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
    "properties": {
        "type": "IaaSDiagnostics",
        "autoUpgradeMinorVersion": true,
        "protectedSettings": {
        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
        "storageAccountEndPoint": "https://core.windows.net/"
        },
        "publisher": "Microsoft.Azure.Diagnostics",
        "settings": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": "50000",
            "EtwProviders": {
                "EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
                {
                    "provider": "Microsoft-ServiceFabric-Services",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableServiceEventTable"
                    }
                }
                ],
                "EtwManifestProviderConfiguration": [
                {
                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                },
                {
                    "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                }
                ]
            }
            }
        },
        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
        },
        "typeHandlerVersion": "1.5"
    }
}
```

Après avoir modifié le fichier template.json comme décrit, republiez le modèle Resource Manager. Si le modèle a été exporté, exécutez le fichier deploy.ps1 pour republier le modèle. Après le déploiement, assurez-vous que **ProvisioningState** présente la valeur **Succeeded**.

> [!TIP]
> Si vous souhaitez déployer des conteneurs dans votre cluster, autorisez WAD à récupérer les statistiques de docker en ajoutant le code suivant à votre section **WadCfg > DiagnosticMonitorConfiguration**.
>
>```json
>"DockerSources": {
>    "Stats": {
>        "enabled": true,
>        "sampleRate": "PT1M"
>    }
>},
>```

### <a name="update-storage-quota"></a>Mettre à jour le quota de stockage

Étant donné que les tables remplies par l’extension croissent jusqu’à ce que le quota soit atteint, vous pouvez envisager de réduire la taille du quota. La valeur par défaut est de 50 Go et est configurable dans le modèle sous le champ `overallQuotaInMB` figurant sous `DiagnosticMonitorConfiguration`.

```json
"overallQuotaInMB": "50000",
```

## <a name="log-collection-configurations"></a>Configurations de la collecte de journaux
Il est également possible de collecter des journaux d’activité provenant d’autres canaux ; voici quelques configurations courantes que vous pouvez appliquer dans le modèle des clusters qui fonctionnent avec Azure.

* Canal opérationnel - De base : Activé par défaut, opérations de haut niveau effectuées par Service Fabric et le cluster. Cela comprend les événements pour un nœud mis en ligne, une nouvelle application déployée, l’annulation d’une mise à niveau, etc. Pour connaître la liste des événements, consultez la page [Événements du canal opérationnel](./service-fabric-diagnostics-event-generation-operational.md).
  
```json
      scheduledTransferKeywordFilter: "4611686018427387904"
  ```
* Canal opérationnel - Détaillé : Ceci inclut des rapports d’intégrité et des décisions d’équilibrage de charge, ainsi que tous les éléments du canal opérationnel de base. Ces événements sont générés par le système ou bien par votre code à l’aide des API de création de rapports d’intégrité ou de charge, par exemple, [ReportPartitionHealth](/previous-versions/azure/reference/mt645153(v=azure.100)) ou [ReportLoad](/previous-versions/azure/reference/mt161491(v=azure.100)). Pour afficher ces événements dans la visionneuse d’événements de diagnostic de Visual Studio, ajoutez « Microsoft-ServiceFabric:4:0x4000000000000008 » à la liste des fournisseurs ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387912"
  ```

* Canal de données et de messagerie - De base : Événements et journaux d’activité critiques générés dans le chemin des messages (pour le moment, seulement le proxy inverse) et des données, ainsi que les journaux d’activité détaillés du canal opérationnel. Ces événements correspondent à des échecs de traitement des requêtes et à d’autres problèmes critiques dans le proxy inverse, ainsi qu’aux requêtes traitées. **C’est ce que nous recommandons pour une journalisation complète**. Pour afficher ces événements dans l’observateur d’événements de diagnostic de Visual Studio, ajoutez « Microsoft-ServiceFabric:4:0x4000000000000010 » à la liste des fournisseurs ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387928"
  ```

* Canal de données et de messagerie - Détaillé : Canal détaillé qui contient tous les journaux d’activité non critiques des données et des messages du cluster, ainsi que le canal opérationnel détaillé. Pour résoudre les problèmes liés aux différents événements du proxy inverse, consultez le [guide de diagnostic du proxy inverse](service-fabric-reverse-proxy-diagnostics.md).  Pour afficher ces événements dans l’observateur d’événements de diagnostic de Visual Studio, ajoutez « Microsoft-ServiceFabric:4:0x4000000000000020 » à la liste des fournisseurs ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387944"
  ```

>[!NOTE]
>Ce canal détaillé comporte un très gros volume d’événements ; si la collecte d’événements est activée à partir de ce canal, de nombreuses traces sont générées sur un intervalle court, ce qui risque de consommer de la capacité de stockage. Par conséquent, n’activez cette fonctionnalité qu’en cas de nécessité absolue.


Pour activer le **Canal opérationnel de base**, recommandé pour la journalisation complète avec le moins d’informations inutiles, la configuration `EtwManifestProviderConfiguration` de `WadCfg` de votre modèle doit se présenter ainsi :

```json
  "WadCfg": {
        "DiagnosticMonitorConfiguration": {
          "overallQuotaInMB": "50000",
          "EtwProviders": {
            "EtwEventSourceProviderConfiguration": [
              {
                "provider": "Microsoft-ServiceFabric-Actors",
                "scheduledTransferKeywordFilter": "1",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricReliableActorEventTable"
                }
              },
              {
                "provider": "Microsoft-ServiceFabric-Services",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricReliableServiceEventTable"
                }
              }
            ],
            "EtwManifestProviderConfiguration": [
              {
                "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                "scheduledTransferLogLevelFilter": "Information",
                "scheduledTransferKeywordFilter": "4611686018427387904",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricSystemEventTable"
                }
              },
              {
                "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                "scheduledTransferLogLevelFilter": "Information",
                "scheduledTransferKeywordFilter": "4611686018427387904",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                "eventDestination": "ServiceFabricSystemEventTable"
                }
              }
            ]
          }
        }
      },
```

## <a name="collect-from-new-eventsource-channels"></a>Collecter à partir de nouveaux canaux EventSource

Pour mettre à jour Diagnostics afin de collecter les journaux d’activité de nouveaux canaux EventSource représentant une nouvelle application que vous êtes sur le point de déployer, effectuez les mêmes étapes que précédemment pour le programme d’installation de Diagnostics d’un cluster existant.

Mettez à jour la section `EtwEventSourceProviderConfiguration` dans le fichier template.json pour ajouter des entrées pour les nouveaux canaux EventSource avant d’appliquer la mise à jour de la configuration à l’aide de la commande PowerShell `New-AzResourceGroupDeployment`. Le nom de la source de l’événement est défini dans le cadre de votre code dans le fichier ServiceEventSource.cs généré par Visual Studio.

Par exemple, si votre source d’événements est nommée My-Eventsource, ajoutez le code suivant pour placer les événements de My-Eventsource dans une table nommée MyDestinationTableName.

```json
        {
            "provider": "My-Eventsource",
            "scheduledTransferPeriod": "PT5M",
            "DefaultEvents": {
            "eventDestination": "MyDestinationTableName"
            }
        }
```

Pour collecter des compteurs de performances ou des journaux d’événements, modifiez le modèle Resource Manager en utilisant les exemples fournis dans [Créer une machine virtuelle Windows avec des fonctionnalités de supervision et de diagnostics à l’aide d’un modèle Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json). Republiez ensuite le modèle Resource Manager.

## <a name="collect-performance-counters"></a>Collecter des compteurs de performances

Pour collecter les métriques de performances de votre cluster, ajoutez les compteurs de performances à « WadCfg > DiagnosticMonitorConfiguration » dans le modèle Resource Manager de votre cluster. Pour obtenir des instructions sur la modification de votre `WadCfg` en vue de collecter les données de certains compteurs de performances, consultez [Analyse des performances avec WAD](service-fabric-diagnostics-perf-wad.md). Pour obtenir la liste des compteurs de performances que nous vous recommandons de collecter, consultez [Compteurs de performances de Service Fabric](service-fabric-diagnostics-event-generation-perf.md).
  
Si vous utilisez un récepteur Application Insights, comme décrit dans la section ci-dessous, et souhaitez afficher ces mesures dans Application Insights, assurez-vous d’ajouter le nom du récepteur dans la section « récepteurs » comme indiqué ci-dessus. Cela envoie automatiquement les compteurs de performances configurés à votre ressource Application Insights.


## <a name="send-logs-to-application-insights"></a>Envoyer les journaux d’activité à Application Insights

### <a name="configuring-application-insights-with-wad"></a>Configuration d'Application Insights avec WAD

>[!NOTE]
>Cette configuration s’applique uniquement aux clusters Windows pour le moment.

Il existe deux méthodes principales permettant d’envoyer des données à Azure Application Insights à partir de WAD, ce qui nécessite d’ajouter un récepteur Application Insights à la configuration de WAD, via le portail Azure ou via un modèle Azure Resource Manager.

#### <a name="add-an-application-insights-instrumentation-key-when-creating-a-cluster-in-azure-portal"></a>Ajouter une clé d’instrumentation Application Insights lors de la création d’un cluster dans le portail Azure

![Ajout d’une clé AIKey](media/service-fabric-diagnostics-event-analysis-appinsights/azure-enable-diagnostics.png)

Lors de la création d’un cluster, si le diagnostic est « Activé » », un champ facultatif permettant d’entrer une clé d’instrumentation Application Insights s’affiche. Si vous collez votre clé Application Insights ici, le récepteur Application Insights est automatiquement configuré à votre intention dans le modèle Resource Manager utilisé pour le déploiement de votre cluster.

#### <a name="add-the-application-insights-sink-to-the-resource-manager-template"></a>Ajouter le récepteur Application Insights au modèle Resource Manager

Dans l’élément « WadCfg » du modèle Resource Manager, ajoutez un récepteur (« Sink ») en incluant les deux modifications suivantes :

1. Ajoutez la configuration du récepteur directement une fois que la déclaration de la `DiagnosticMonitorConfiguration` est terminée :

    ```json
    "SinksConfig": {
        "Sink": [
            {
                "name": "applicationInsights",
                "ApplicationInsights": "***ADD INSTRUMENTATION KEY HERE***"
            }
        ]
    }

    ```

2. Incluez le récepteur dans la `DiagnosticMonitorConfiguration` en ajoutant la ligne suivante dans la `DiagnosticMonitorConfiguration` de `WadCfg` (juste avant de déclarer les `EtwProviders`) :

    ```json
    "sinks": "applicationInsights"
    ```

Dans les deux extraits de code précédents, le nom « applicationInsights » a été utilisé pour décrire le récepteur. Cela n’est pas obligatoire, et tant que le nom du récepteur est inclus dans l’élément « sinks », vous pouvez définir n’importe quelle chaîne comme nom.

Actuellement, les journaux d’activité du cluster s’affichent sous forme de **traces** dans la visionneuse des journaux d’activité d’Application Insights. Étant donné que la plupart des traces provenant de la plateforme sont de type « Informations », vous pouvez également envisager de modifier la configuration du récepteur pour que seuls les journaux d’activité de type « Avertissement » ou « Erreur » soient envoyés. Pour ce faire, ajoutez « Channels » à votre récepteur, comme illustré dans [cet article](../azure-monitor/platform/diagnostics-extension-to-application-insights.md).

>[!NOTE]
>Si vous utilisez une clé Application Insights incorrecte dans le portail ou dans votre modèle Resource Manager, vous devrez modifier manuellement la clé et mettre à jour/redéployer le cluster.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez correctement configuré les diagnostics Azure, vous verrez les données de vos tables de stockage dans les journaux d’activité ETW et EventSource. Si vous choisissez d’utiliser les journaux d’activité Azure Monitor, Kibana ou une autre plateforme d’analytique données et de visualisation des données qui n’est pas directement configurée dans le modèle Resource Manager, veillez à configurer la plateforme de votre choix pour lire les données à partir de ces tables de stockage. Cette opération relativement simple pour les journaux d’activité Azure Monitor est expliquée dans [Analyse d’événements et de journaux](service-fabric-diagnostics-event-analysis-oms.md). L’utilisation d’Application Insights est un peu plus compliquée ici, car il peut être configuré en tant que partie de la configuration de l’extension Diagnostics. Reportez-vous par conséquent à l[’article approprié](service-fabric-diagnostics-event-analysis-appinsights.md) si vous choisissez d’utiliser AI.

>[!NOTE]
>Il n’existe actuellement aucun moyen de filtrer ou de nettoyer les événements qui sont envoyés à la table. Si vous n’implémentez aucun processus de suppression des événements de la table, la table continuera à croître. Un exemple de service de nettoyage de données en cours d’exécution est inclus avec l[’échantillon Watchdog](https://github.com/Azure-Samples/service-fabric-watchdog-service), et il est recommandé d’en écrire un vous-même, sauf s’il existe une bonne raison de stocker les journaux d’activité au-delà de 30 ou 90 jours.

* [Découvrez comment collecter des compteurs de performances ou des journaux d’activité à l’aide de l’extension Diagnostics](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)
* [Analyse et visualisation d’événements avec Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Analyse et visualisation d’événements avec les journaux d’activité Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Analyse et visualisation d’événements avec Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Analyse et visualisation d’événements avec les journaux d’activité Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
