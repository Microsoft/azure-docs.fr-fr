---
title: Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)
description: Découvrez comment installer et configurer l’extension Diagnostics Windows. Vous trouverez également une description de la façon dont les données sont stockées dans un compte de stockage Azure.
services: azure-monitor
author: bwren
ms.subservice: diagnostic-extension
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: bwren
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: e6ccba27fb599cb26da86e94d3500f4f806ecb76
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91328868"
---
# <a name="install-and-configure-windows-azure-diagnostics-extension-wad"></a>Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)
L’[extension Diagnostics Azure](diagnostics-extension-overview.md) est un agent présent dans Azure Monitor qui collecte des données de supervision dans le système d’exploitation invité, des charges de travail de machines virtuelles Azure et d’autres ressources de calcul. Cet article fournit des détails sur l’installation et la configuration de l’extension de diagnostic Windows et une description de la façon dont les données sont stockées dans un compte Stockage Azure.

L’extension de diagnostic étant implémentée comme [extension de machine virtuelle](../../virtual-machines/extensions/overview.md) dans Azure, elle prend en charge les mêmes options d’installation à l’aide de modèles Resource Manager, de PowerShell et de l’interface CLI. Pour plus d’informations sur l’installation et la maintenance des extensions de machine virtuelle, consultez [Extensions et fonctionnalités de machine virtuelle pour Windows](../../virtual-machines/extensions/features-windows.md).

## <a name="overview"></a>Vue d’ensemble
Quand vous configurez l’extension Diagnostics Azure pour Windows, vous devez spécifier un compte de stockage dans lequel toutes les données spécifiées seront envoyées. Vous pouvez éventuellement en ajouter un pour plusieurs *récepteurs de données* pour envoyer les données vers différents emplacements.

- Récepteur Azure Monitor : envoie les données de performances de l’invité dans les métriques Azure Monitor.
- Récepteur Event Hub : envoie les données de performances et de journal de l’invité aux Event Hubs Azure en vue d’une transmission en dehors d’Azure. Ce récepteur ne peut pas être configuré dans le portail Azure.


## <a name="install-with-azure-portal"></a>Installer avec Portail Azure
Vous pouvez installer et configurer l’extension de diagnostics sur une machine virtuelle individuelle dans le Portail Azure qui vous fournit une interface au lieu de travailler directement avec la configuration. Lorsque vous activez l’extension de diagnostic, elle utilise automatiquement une configuration par défaut avec les compteurs de performance et les événements les plus courants. Vous pouvez modifier cette configuration par défaut en fonction de vos besoins spécifiques.

> [!NOTE]
> Les éléments suivants décrivent les paramètres les plus courants pour l’extension de diagnostic. Pour plus d’informations sur toutes les options de configuration, consultez [Schéma d’extension de diagnostic Windows](diagnostics-extension-schema-windows.md).

1. Ouvrez le menu d’une machine virtuelle dans le Portail Azure.

2. Cliquez sur **Paramètres de diagnostic** dans la section **Surveillance** du menu de la machine virtuelle.

3. Cliquez sur **Activer la supervision d’invités** si l’extension de diagnostic n’a pas déjà été activée.

   ![Activer la supervision](media/diagnostics-extension-windows-install/enable-monitoring.png)

4. Un nouveau compte Stockage Azure sera créé pour la machine virtuelle, dont le nom sera basé sur celui du groupe de ressources de la machine virtuelle, et un ensemble par défaut de compteurs de performances et de journaux de l’invité sera sélectionné.

   ![Paramètres de diagnostic](media/diagnostics-extension-windows-install/diagnostic-settings.png)

5. Dans l’onglet **Compteurs de performances**, sélectionnez les métriques d’invité que vous souhaitez collecter sur cette machine virtuelle. Utilisez le paramètre **Personnaliser** pour une sélection plus avancée.

   ![Compteurs de performance](media/diagnostics-extension-windows-install/performance-counters.png)

6. Dans l’onglet **Journaux**, sélectionnez les journaux à collecter sur la machine virtuelle. Les journaux peuvent être envoyés au stockage ou à des Event Hubs, mais pas à Azure Monitor. Utilisez l’[agent Log Analytics ](log-analytics-agent.md) pour collecter des journaux d’invité et les envoyer à Azure Monitor.

   ![Capture d’écran présentant l’onglet Journaux avec différents journaux sélectionnés pour une machine virtuelle.](media/diagnostics-extension-windows-install/logs.png)

7. Dans l’onglet **Vidages sur incident**, spécifiez les processus pour collecter des images mémoire après un incident. Les données seront écrites dans le compte de stockage pour le paramètre de diagnostic, et vous pouvez éventuellement spécifier un conteneur de blobs.

   ![Vidages sur incident](media/diagnostics-extension-windows-install/crash-dumps.png)

8. Dans l’onglet **Récepteurs**, indiquez si vous souhaitez envoyer les données vers des emplacements autres que le stockage Azure. Si vous sélectionnez **Azure Monitor**, les données de performances de l’invité seront envoyées aux métriques Azure Monitor. Vous ne pouvez pas configurer le récepteur d’Event Hubs dans le portail Azure.

   ![Capture d’écran présentant l’onglet Récepteurs dont l’option Envoyer des données de diagnostic à Azure Monitor est activée.](media/diagnostics-extension-windows-install/sinks.png)
   
   Si vous n’avez pas activé une identité affectée par le système configurée pour votre machine virtuelle, l’avertissement ci-dessous peut s’afficher lorsque vous enregistrez une configuration avec le récepteur Azure Monitor. Cliquez sur la bannière pour activer l’identité affectée par le système.
   
   ![Entité gérée](media/diagnostics-extension-windows-install/managed-entity.png)

9. Dans l’onglet **Agent**, vous pouvez modifier le compte de stockage, définir le quota de disque et spécifier s’il faut collecter les journaux de l’infrastructure de diagnostic.  

   ![Capture d’écran présentant l’onglet Agent avec l’option qui permet de définir le compte de stockage.](media/diagnostics-extension-windows-install/agent.png)

10. Pour enregistrer la configuration, cliquez sur **Enregistrer**. 

> [!NOTE]
> Bien que la configuration de l’extension de diagnostic puisse être au format JSON ou XML, toute configuration effectuée dans le Portail Azure est toujours stockée au format JSON. Si vous utilisez XML avec une autre méthode de configuration, puis modifiez votre configuration avec le Portail Azure, les paramètres seront modifiés en format JSON.

## <a name="resource-manager-template"></a>Modèle Resource Manager
Pour plus d’informations sur le déploiement de l’extension de diagnostic à l’aide de vos modèles Azure Resource Manager, consultez [Utiliser la surveillance et les diagnostics avec une machine virtuelle Windows et des modèles Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md). 

## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure
Vous pouvez utiliser Azure CLI pour déployer l’extension Diagnostics Azure sur une machine virtuelle existante à l’aide de la commande [az vm extension set](/cli/azure/vm/extension?view=azure-cli-latest#az-vm-extension-set), comme indiqué dans l’exemple suivant. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name IaaSDiagnostics \
  --publisher Microsoft.Azure.Diagnostics \
  --protected-settings protected-settings.json \
  --settings public-settings.json 
```

Les paramètres protégés sont définis dans l’[élément PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element) du schéma de configuration. Voici l’exemple minimal d’un fichier de paramètres protégés qui définit le compte de stockage. Pour plus d’informations sur les paramètres privés, consultez [Exemple de configuration](diagnostics-extension-schema-windows.md#privateconfig-element).

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
}
```

Les paramètres publics sont définis dans l’[élément PublicConfig](diagnostics-extension-schema-windows.md#publicconfig-element) du schéma de configuration. Voici l’exemple minimal d’un fichier de paramètres publics qui permet la collecte de journaux d’infrastructure de diagnostic, d’un seul compteur de performances et d’un seul journal des événements. Pour plus d’informations sur les paramètres publics, consultez [Exemple de configuration](diagnostics-extension-schema-windows.md#publicconfig-element).

```JSON
{
  "StorageAccount": "mystorageaccount",
  "WadCfg": {
    "DiagnosticMonitorConfiguration": {
      "overallQuotaInMB": 5120,
      "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": [
          {
            "counterSpecifier": "\\Processor Information(_Total)\\% Processor Time",
            "unit": "Percent",
            "sampleRate": "PT60S"
          }
        ]
      },
      "WindowsEventLog": {
        "scheduledTransferPeriod": "PT1M",
        "DataSource": [
          {
            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
          }
        ]
      }
    }
  }
}
```



## <a name="powershell-deployment"></a>Déploiement PowerShell
Vous pouvez utiliser PowerShell pour déployer l’extension Diagnostics Azure sur une machine virtuelle existante à l’aide de la commande [Set-AzVMDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azurevmdiagnosticsextension), comme indiqué dans l’exemple suivant. 

```powershell
Set-AzVMDiagnosticsExtension -ResourceGroupName "myvmresourcegroup" `
  -VMName "myvm" `
  -DiagnosticsConfigurationPath "DiagnosticsConfiguration.json"
```

Les paramètres privés sont définis dans l’[élément PrivateConfig](diagnostics-extension-schema-windows.md#privateconfig-element), tandis que les paramètres publics sont définis dans l’[élément PublicConfig](diagnostics-extension-schema-windows.md#publicconfig-element) du schéma de configuration. Vous pouvez également choisir de spécifier les détails du compte de stockage en tant que paramètres de la cmdlet Set-AzVMDiagnosticsExtension au lieu de les inclure dans les paramètres privés.

Voici l’exemple minimal d’un fichier config qui permet la collecte de journaux d’infrastructure de diagnostic, d’un seul compteur de performances et d’un seul journal des événements. Pour plus d’informations sur les paramètres privés et publics, consultez [Exemple de configuration](diagnostics-extension-schema-windows.md#publicconfig-element). 

```JSON
{
    "PublicConfig": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
                "overallQuotaInMB": 10000,
                "DiagnosticInfrastructureLogs": {
                    "scheduledTransferLogLevelFilter": "Error"
                },
                "PerformanceCounters": {
                    "scheduledTransferPeriod": "PT1M",
                    "PerformanceCounterConfiguration": [
                        {
                            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                            "sampleRate": "PT3M",
                            "unit": "percent"
                        }
                    ]
                },
                "WindowsEventLog": {
                    "scheduledTransferPeriod": "PT1M",
                        "DataSource": [
                        {
                            "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                        }
                    ]
                }
            }
        },
        "StorageAccount": "mystorageaccount",
        "StorageType": "TableAndBlob"
    },
    "PrivateConfig": {
        "storageAccountName": "mystorageaccount",
        "storageAccountKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "storageAccountEndPoint": "https://mystorageaccount.blob.core.windows.net"
    }
}
```

Voir aussi [Utiliser PowerShell pour activer Diagnostics Azure sur une machine virtuelle exécutant Windows](../../virtual-machines/extensions/diagnostics-windows.md).

## <a name="data-storage"></a>Stockage des données
Le tableau suivant répertorie les différents types de données collectées à partir de l’extension de diagnostic et indique si elles sont stockées sous la forme d’une table ou d’un BLOB. Les données stockées dans des tableaux peuvent également être stockées dans des blobs en fonction du [paramètre StorageType](diagnostics-extension-schema-windows.md#publicconfig-element) de votre configuration publique.


| Données | Type de stockage | Description |
|:---|:---|:---|
| WADDiagnosticInfrastructureLogsTable | Table de charge de travail | Moniteur de diagnostic et modifications de la configuration. |
| WADDirectoriesTable | Table de charge de travail | Répertoires que le moniteur de diagnostic surveille.  Cela inclut les journaux d’activité IIS, les journaux d’activité de requêtes ayant échoué et les répertoires personnalisés IIS.  L’emplacement du fichier journal blob est spécifié dans le champ de conteneur et le nom de l’objet blob se trouve dans le champ RelativePath.  Le champ AbsolutePath indique l’emplacement et le nom du fichier tel qu’il existait sur la machine virtuelle Azure. |
| WadLogsTable | Table de charge de travail | Journaux d’activité rédigés dans le code à l’aide de l’écouteur de suivi. |
| WADPerformanceCountersTable | Table de charge de travail | Compteurs de performance. |
| WADWindowsEventLogsTable | Table de charge de travail | Journaux des événements Windows. |
| wad-iis-failedreqlogfiles | Objet blob | Contient des informations issues des journaux d’activité de requêtes IIS ayant échoué. |
| wad-iis-logfiles | Objet blob | Contient des informations sur les journaux IIS. |
| « personnalisé » | Objet blob | Conteneur personnalisé basé sur la configuration des répertoires analysés par le moniteur de diagnostic.  Le nom de ce conteneur d’objets blobs est spécifié dans WADDirectoriesTable. |

## <a name="tools-to-view-diagnostic-data"></a>Outils permettant d’afficher les données de diagnostic
Plusieurs outils sont disponibles pour afficher les données après leur transfert vers le stockage. Par exemple :

* Explorateur de serveurs dans Visual Studio : si vous avez installé Microsoft Azure Tools pour Microsoft Visual Studio, vous pouvez utiliser le nœud de stockage Azure dans l’Explorateur de serveurs pour afficher des objets blobs en lecture seule et les données du tableau depuis vos comptes de stockage Azure. Vous pouvez afficher des données à partir de votre compte d’émulateur de stockage local et de comptes de stockage que vous avez créés pour Azure. Pour plus d’informations, consultez [Consultation et gestion des ressources de stockage avec l’Explorateur de serveurs](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage).
* [L’explorateur de stockage Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome qui vous permet d’utiliser facilement les données Azure Storage sur Windows, OSX et Linux.
* [Azure Management Studio](https://www.cerebrata.com/products/azure-management-studio/introduction) inclut Diagnostics Azure Manager qui vous permet d’afficher, de télécharger et de gérer les données de diagnostic collectées par les applications s’exécutant dans Azure.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur le transfert des données d’analyse à Azure Event Hubs, consultez la page [Envoyer les données de l’extension Diagnostics Azure pour Windows à Event Hubs](diagnostics-extension-stream-event-hubs.md).
