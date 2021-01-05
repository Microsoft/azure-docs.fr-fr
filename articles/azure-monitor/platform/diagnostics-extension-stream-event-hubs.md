---
title: Envoyer des données provenant de l’extension Diagnostics Azure pour Windows à Azure Event Hubs
description: Configurez l’extension de diagnostic dans Azure Monitor pour envoyer des données à Azure Event Hubs afin de pouvoir les transférer vers des emplacements en dehors d’Azure.
ms.subservice: diagnostic-extension
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/18/2020
ms.openlocfilehash: 979535b1f9a237f6975908178fb1e5ed819181b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82233463"
---
# <a name="send-data-from-windows-azure-diagnostics-extension-to-azure-event-hubs"></a>Envoyer des données provenant de l’extension Diagnostics Azure pour Windows à Azure Event Hubs
L’extension Diagnostics Azure est un agent présent dans Azure Monitor qui collecte des données de supervision dans le système d’exploitation invité et des charges de travail de machines virtuelles Azure et autres ressources de calcul. Cet article explique comment envoyer des données de l’extension Diagnostics Azure pour Windows (WAD) à [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) afin de pouvoir les transférer vers des emplacements en dehors d’Azure.

## <a name="supported-data"></a>Données prises en charge

Les données collectées auprès du système d’exploitation invité qui peuvent être envoyées à Event Hubs incluent ce qui suit. Les autres sources de données collectées par WAD, notamment les journaux IIS et les vidages sur incident, ne peuvent pas être envoyées à Event Hubs.

* Suivi d’événements pour les événements Windows (ETW)
* Compteurs de performance
* Journaux des événements Windows, y compris les journaux des applications dans le journal des événements Windows
* Journaux d’activité d’infrastructure de diagnostics Azure

## <a name="prerequisites"></a>Prérequis

* Extension de diagnostic Windows 1.6 ou ultérieure. Consultez [Versions et historique des schémas de configuration de l’extension Diagnostics Azure](diagnostics-extension-versions.md) pour obtenir un historique des versions et [Vue d’ensemble de l’extension Diagnostics Azure](diagnostics-extension-overview.md) pour connaître les ressources prises en charge.
* L’espace de noms Event Hubs doit toujours être approvisionné. Pour plus d’informations, consultez [Prise en main d’Azure Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).


## <a name="configuration-schema"></a>Schéma de configuration
Consultez [Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)](diagnostics-extension-windows-install.md) pour connaître les différentes options d’activation et de configuration de l’extension de diagnostic et [Schéma de configuration d’Azure Diagnostics](diagnostics-extension-schema-windows.md) pour obtenir une référence du schéma de configuration. Le reste de cet article décrit comment utiliser cette configuration pour envoyer des données à un Event Hub. 

Diagnostics Azure envoie toujours les journaux d’activité et les mesures à un compte Stockage Azure. Vous pouvez configurer un ou plusieurs *récepteurs de données* pour envoyer des données à d’autres emplacements. Chaque récepteur est défini dans l’[élément SinksConfig](diagnostics-extension-schema-windows.md#sinksconfig-element) de la configuration publique avec des informations sensibles dans la configuration privée. Cette configuration pour des Event Hubs utilise les valeurs du tableau suivant.

| Propriété | Description |
|:---|:---|
| Nom | Nom descriptif du récepteur. Utilisé dans la configuration pour spécifier les sources de données à envoyer au récepteur. |
| Url  | URL du hub d’événements sous la forme \<event-hubs-namespace\>. servicebus.windows.net/\<event-hub-name\>.          |
| SharedAccessKeyName | Nom d’une stratégie d’accès partagé pour l’Event Hub qui dispose au moins de l’autorisation **Send** (Envoyer). |
| SharedAccessKey     | Clé primaire ou secondaire de la stratégie d’accès partagé pour l’Event Hub. |

Vous trouverez ci-dessous des exemples de configurations privées et publiques. Il s’agit d’une configuration minimale avec un compteur de performances et un journal des événements uniques pour illustrer la configuration et l’utilisation du récepteur de données de l’Event Hub. Pour obtenir un exemple plus complexe, consultez [Schéma de configuration Diagnostics Azure](diagnostics-extension-schema-windows.md).

### <a name="public-configuration"></a>Configuration publique

```JSON
{
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 5120,
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT3M"
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                    "DataSource": [
                    {
                        "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                    }
                ]
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "myEventHub",
                    "EventHub": {
                        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
                        "SharedAccessKeyName": "SendRule"
                    }
                }
            ]
        }
    },
    "StorageAccount": "mystorageaccount",
}
```


### <a name="private-configuration"></a>Configuration privée

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "{base64 encoded key}",
    "storageAccountEndPoint": "https://core.windows.net",
    "EventHub": {
        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    }
}
```



## <a name="configuration-options"></a>Options de configuration
Pour envoyer des données à un récepteur de données, vous spécifiez l’attribut **sinks** sur le nœud de la source de données. Où vous placez l’attribut **sinks** détermine l’étendue de l’assignation. Dans l’exemple suivant, l’attribut **sinks** est défini sur le nœud **PerformanceCounters**, ce qui entraîne l’envoi de tous les compteurs de performances enfants à l’Event Hub.

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "sinks": "MyEventHub",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        }
    ]
}
```


Dans l’exemple suivant, l’attribut **sinks** est appliqué directement à trois compteurs, ce qui entraîne l’envoi de ces seuls compteurs de performances à l’Event Hub. 

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Rejected",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Queued",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        }
    ]
}
```

## <a name="validating-configuration"></a>Validation de la configuration
Vous pouvez utiliser diverses méthodes pour valider l’envoi des données à l’Event Hub. Une méthode simple consiste à utiliser des captures d’Event Hubs comme décrit dans [Capturer des événements avec Azure Event Hubs dans Stockage Blob Azure ou Azure Data Lake Storage](../../event-hubs/event-hubs-capture-overview.md). 


## <a name="troubleshoot-event-hubs-sinks"></a>Résolution des problèmes liés aux récepteurs Event Hubs

- Examinez la table Stockage Azure **WADDiagnosticInfrastructureLogsTable**qui contient les journaux d’activité et les erreurs de Diagnostics Azure. Une option consiste à utiliser un outil tel que l’ [explorateur de stockage Azure](https://www.storageexplorer.com) pour vous connecter à ce compte de stockage, consulter cette table et ajouter une requête pour l’horodatage (TimeStamp) des dernières 24 heures. Vous pouvez utiliser l’outil pour exporter un fichier .csv et l’ouvrir dans une application comme Microsoft Excel. Excel permet de rechercher facilement des chaînes de carte d’appel, telles **qu’EventHubs**afin d’identifier l’erreur signalée.  

- Vérifiez que le hub d’événements a été correctement approvisionné. Toutes les informations de connexion de la section **PrivateConfig** de la connecter doivent correspondre aux valeurs de votre ressource, comme indiqué dans le portail. Assurez-vous qu’une stratégie SAP est définie (*SendRule* dans l’exemple) dans le portail et que l’autorisation *Send* a été octroyée.  

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Event Hubs](../../event-hubs/event-hubs-about.md)
* [Créer un concentrateur d’événements](../../event-hubs/event-hubs-create.md)
* [FAQ sur les hubs d’événements](../../event-hubs/event-hubs-faq.md)

<!-- Images. -->
[0]: ../../event-hubs/media/event-hubs-streaming-azure-diags-data/dashboard.png



