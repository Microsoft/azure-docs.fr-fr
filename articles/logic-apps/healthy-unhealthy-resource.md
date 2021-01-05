---
title: Configurer la journalisation pour superviser des applications logiques dans Azure Security Center
description: Supervisez l’intégrité de vos ressources Logic Apps dans Azure Security Center en configurant la journalisation des diagnostics.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 12/07/2020
ms.openlocfilehash: 7d43c128f0f587a32cc4b8673727579043f268eb
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629593"
---
# <a name="set-up-logging-to-monitor-logic-apps-in-azure-security-center"></a>Configurer la journalisation pour superviser des applications logiques dans Azure Security Center

Lorsque vous supervisez vos ressources Logic Apps dans [Microsoft Azure Security Center](../security-center/security-center-introduction.md), vous pouvez [vérifier si vos applications logiques suivent les stratégies par défaut](#view-logic-apps-health-status). Azure affiche l’état d’intégrité d’une ressource Logic Apps une fois que vous avez activé la journalisation et configuré correctement la destination des journaux. Cet article explique comment configurer la journalisation des diagnostics et s’assurer que toutes vos applications logiques sont des ressources saines.

> [!TIP]
> Pour connaître l’état actuel du service Logic Apps, consultez la [page d’état Azure](https://status.azure.com/), qui liste l’état de différents produits et services dans chaque région disponible.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Applications logiques existantes avec [journalisation des diagnostics activée](#enable-diagnostic-logging)
* Un espace de travail Log Analytics, nécessaire pour activer la journalisation pour votre application logique. Si vous n’avez pas d’espace de travail, commencez par [créer votre espace de travail](/azure/azure-monitor/learn/quick-create-workspace)

## <a name="enable-diagnostic-logging"></a>Activer la journalisation des diagnostics

Avant de pouvoir afficher l’état d’intégrité des ressources pour vos applications logiques, vous devez [configurer la journalisation des diagnostics](monitor-logic-apps-log-analytics.md). Si vous avez déjà un espace de travail Log Analytics, vous pouvez activer la journalisation lorsque vous créez votre application logique ou sur des applications logiques existantes.

> [!TIP]
> La recommandation par défaut consiste à activer les journaux de diagnostic pour Logic Apps. Toutefois, vous contrôlez ce paramètre pour vos applications logiques. Lorsque vous activez les journaux de diagnostic pour vos applications logiques, vous pouvez utiliser ces informations pour faciliter l’analyse des incidents de sécurité.

### <a name="check-diagnostic-logging-setting"></a>Vérifier le paramètre de journalisation des diagnostics

Si vous ne savez pas si la journalisation des diagnostics est activée pour vos applications logiques, vous pouvez le vérifier dans Security Center :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre de recherche, entrez et sélectionnez **Security Center**.
1. Dans le menu du tableau de bord de Security Center, sous **Général**, sélectionnez **Recommandations**.
1. Dans le tableau des suggestions de sécurité, recherchez et sélectionnez **Activer l’audit et la journalisation** &gt; **Les journaux de diagnostic dans Logic Apps doivent être activés** dans le tableau des contrôles de sécurité.
1. Dans la page de recommandation, développez la section **Étapes de correction** et passez en revue les options. Vous pouvez activer les diagnostics Logic Apps en sélectionnant le bouton **Correction rapide !** ou en suivant les instructions de correction manuelle.

## <a name="view-logic-apps-health-status"></a>Afficher l’état d’intégrité des applications logiques

Une fois que vous avez [activé la journalisation des diagnostics](#enable-diagnostic-logging), vous pouvez voir l’état d’intégrité de vos applications logiques dans Security Center.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre de recherche, entrez et sélectionnez **Security Center**.
1. Dans le menu du tableau de bord de Security Center, sous **Général**, sélectionnez **Inventaire**.
1. Dans la page d’inventaire, filtrez votre liste de ressources pour afficher uniquement les ressources Logic Apps. Dans le menu de la page, sélectionnez **Types de ressources** &gt; **Logic Apps**.

   Le compteur **Ressources non intègres** affiche le nombre d’applications logiques que Security Center considère comme non intègres.
1.  Dans la liste des ressources Logic Apps, examinez la colonne **Recommandations**. Pour consulter les détails d’intégrité d’une application logique spécifique, sélectionnez un nom de ressource ou cliquez sur le bouton de sélection ( **...** ) &gt; **Afficher la ressource**.
1.  Pour résoudre les problèmes potentiels d’intégrité des ressources, suivez les étapes indiquées pour vos applications logiques.

Si la journalisation des diagnostics est déjà activée, il y a peut-être un problème avec la destination de vos journaux. Consultez la section relative à la [résolution des problèmes liés à différentes destinations de journalisation de diagnostic](#fix-diagnostic-logging-for-logic-apps).

## <a name="fix-diagnostic-logging-for-logic-apps"></a>Corriger la journalisation des diagnostics pour les applications logiques

Si vos [applications logiques sont listées comme étant non intègres dans Security Center](#view-logic-apps-health-status), ouvrez votre application logique en mode code dans le portail Azure ou à l’aide d’Azure CLI. Ensuite, vérifiez la configuration de destination de vos journaux de diagnostic : [Azure Log Analytics](#log-analytics-and-event-hubs-destinations), [Azure Event Hubs](#log-analytics-and-event-hubs-destinations) ou [un compte Stockage Azure](#storage-account-destination).

### <a name="log-analytics-and-event-hubs-destinations"></a>Destinations Log Analytics et Event Hubs

Si vous utilisez Log Analytics ou Event Hubs comme destination de vos journaux de diagnostic Logic Apps, vérifiez les paramètres suivants. 

1. Pour confirmer que vous avez activé les journaux de diagnostic, vérifiez que le champ `logs.enabled` des paramètres de diagnostic a la valeur `true`. 
1. Pour confirmer que vous n’avez pas défini un compte de stockage comme destination à la place, vérifiez que le champ `storageAccountId` a la valeur `false`.

Par exemple :

```json
"allOf": [
    {
        "field": "Microsoft.Insights/diagnosticSettings/logs.enabled",
        "equals": "true"
    },
    {
        "anyOf": [
            {
                "field": "Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.enabled",
                "notEquals": "true"
            },
            {
                "field": "Microsoft.Insights/diagnosticSettings/storageAccountId",
                "exists": false
            }
        ]
    }
] 
```

### <a name="storage-account-destination"></a>Destination Compte de stockage

Si vous utilisez un compte de stockage comme destination de vos journaux de diagnostic Logic Apps, vérifiez les paramètres suivants.

1. Pour confirmer que vous avez activé les journaux de diagnostic, vérifiez que le champ `logs.enabled` des paramètres de diagnostic a la valeur `true`.
1. Pour confirmer que vous avez activé une stratégie de conservation pour vos journaux de diagnostic, vérifiez que le champ `retentionPolicy.enabled` a la valeur `true`.
1. Pour confirmer que vous avez défini une durée de conservation comprise entre 0 et 365 jours, vérifiez que le champ `retentionPolicy.days` a comme valeur un nombre compris entre 0 et 365.

```json
"allOf": [
    {
        "field": "Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.enabled",
        "equals": "true"
    },
    {
        "anyOf": [
            {
                "field": "Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.days",
                "equals": "0"
            },
            {
                "field": "Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.days",
                "equals": "[parameters('requiredRetentionDays')]"
            }
          ]
    },
    {
        "field": "Microsoft.Insights/diagnosticSettings/logs.enabled",
        "equals": "true"
    }
]
```
