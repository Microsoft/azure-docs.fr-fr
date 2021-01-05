---
title: Surveiller les événements de cluster Linux dans Azure Service Fabric
description: Découvrez comment surveiller des événements de cluster Service Fabric Linux en écrivant des événements de plateforme Service Fabric dans syslog.
author: srrengar
ms.topic: conceptual
ms.date: 10/23/2018
ms.author: srrengar
ms.openlocfilehash: fe31c6fdca3651bfe56e798b30d50c9f047c680b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258642"
---
# <a name="service-fabric-linux-cluster-events-in-syslog"></a>événements de cluster Linux Service Fabric dans Syslog

Service Fabric expose un ensemble d’événements de plateforme pour vous informer des activités importantes concernant votre cluster. Vous trouverez la liste complète des événements exposés [ici](service-fabric-diagnostics-event-generation-operational.md). Ces événements peuvent être utilisés de plusieurs façons. Dans cet article, nous allons vous montrer comment configurer Service Fabric pour écrire ces événements dans Syslog.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="introduction"></a>Introduction

Dans la version 6.4, SyslogConsumer a été introduit pour envoyer les événements de la plateforme Service Fabric au Syslog pour les clusters Linux. Une fois activés, les événements sont automatiquement dirigés vers Syslog, qui peut être collecté et envoyé par l’agent Log Analytics.

Chaque événement Syslog comprend 4 composants
* Facility
* Identité
* Message
* severity

Le SyslogConsumer écrit tous les événements de plateforme à l’aide de la Facility `Local0`. Vous pouvez choisir une autre Facility valide en modifiant la configuration. L’identité utilisée est `ServiceFabric`. Le champ Message contient l’événement entier sérialisé au format JSON afin qu’il puisse être interrogé et consommé par une variété d’outils. 

## <a name="enable-syslogconsumer"></a>Activer SyslogConsumer

Pour activer SyslogConsumer, vous devez effectuer une mise à niveau de votre cluster. La section `fabricSettings` doit être mise à jour avec le code suivant. Notez que ce code inclut uniquement les sections relatives à SyslogConsumer

```json
    "fabricSettings": [
        {
            "name": "Diagnostics",
            "parameters": [
            {
                "name": "ConsumerInstances",
                "value": "AzureWinFabCsv, AzureWinFabCrashDump, AzureTableWinFabEtwQueryable, SyslogConsumer"
            }
            ]
        },
        {
            "name": "SyslogConsumer",
            "parameters": [
            {
                "name": "ProducerInstance",
                "value": "WinFabLttProducer"
            },
            {
            "name": "ConsumerType",
            "value": "SyslogConsumer"
            },
            {
                "name": "IsEnabled",
                "value": "true"
            }
            ]
        },
        {
            "name": "Common",
            "parameters": [
            {
                "name": "LinuxStructuredTracesEnabled",
                "value": "true"
            }
            ]
        }
    ],
```

Voici les modifications apportées
1. Dans la section commune, il existe un nouveau paramètre nommé `LinuxStructuredTracesEnabled`. **Il est nécessaire pour que les événements Linux soient structurés et sérialisés lors de leur envoi à Syslog.**
2. Dans la section Diagnostics, un nouveau paramètre ConsumerInstance: SyslogConsumer a été ajouté. Il indique à la plateforme qu’il existe un autre consommateur des événements. 
3. La nouvelle section SyslogConsumer doit avoir `IsEnabled` comme `true`. Elle est configurée pour utiliser automatiquement la Facility Local0. Vous pouvez la remplacer en ajoutant un autre paramètre.

```json
    {
        "name": "New LogFacility",
        "value": "<Valid Syslog Facility>"
    }
```

## <a name="azure-monitor-logs-integration"></a>Intégration de Journaux Azure Monitor
Vous pouvez lire ces événements Syslog dans un outil de supervision, tel que Journaux Azure Monitor. Vous pouvez créer un espace de travail Log Analytics à l’aide de la place de marché Azure en utilisant ces [instructions].(../azure-monitor/learn/quick-create-workspace.md). Vous devez également ajouter l’agent Log Analytics à votre cluster pour collecter et envoyer ces données à l’espace de travail. Il s’agit du même agent utilisé pour collecter les compteurs de performances. 

1. Accédez au panneau `Advanced Settings`

    ![Paramètres de l’espace de travail](media/service-fabric-diagnostics-oms-syslog/workspace-settings.png)

2. Cliquez sur `Data`
3. Cliquez sur `Syslog`
4. Configurez Local0 comme Facility pour le suivi. Vous pouvez ajouter une autre Facility si vous l’avez modifiée dans fabricSettings

    ![Configurer les messages Syslog](media/service-fabric-diagnostics-oms-syslog/syslog-configure.png)
5. Accédez à l’Explorateur des requêtes en cliquant sur `Logs` dans le menu de la ressource d’espace de travail pour commencer à interroger

    ![Journaux d’activité de l’espace de travail](media/service-fabric-diagnostics-oms-syslog/workspace-logs.png)
6. Vous pouvez interroger la table `Syslog` en recherchant `ServiceFabric` comme ProcessName. La requête ci-dessous est un exemple montrant comment analyser le JSON dans l’événement et afficher son contenu

```kusto
    Syslog | where ProcessName == "ServiceFabric" | extend $payload = parse_json(SyslogMessage) | project $payload
```

![Requête Syslog](media/service-fabric-diagnostics-oms-syslog/syslog-query.png)

L’exemple ci-dessus vient d’un événement NodeDown. Vous pouvez consulter la liste complète des événements [ici](service-fabric-diagnostics-event-generation-operational.md).

## <a name="next-steps"></a>Étapes suivantes
* [Déployez l’agent Log Analytics](service-fabric-diagnostics-oms-agent.md) sur vos nœuds pour collecter les compteurs de performances, ainsi que les statistiques et les journaux d’activité Docker de vos conteneurs
* Familiarisez-vous avec les fonctionnalités de [requête et de recherche dans les journaux](../azure-monitor/log-query/log-query-overview.md) proposées par Journaux Azure Monitor
* [Utilisez le Concepteur de vues pour créer des vues personnalisées dans Journaux Azure Monitor](../azure-monitor/platform/view-designer.md)
* Informations de référence pour en savoir plus sur [l’intégration de Journaux Azure Monitor à Syslog](../azure-monitor/platform/data-sources-syslog.md).
