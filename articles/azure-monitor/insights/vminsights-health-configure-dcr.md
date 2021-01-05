---
title: Configurer la supervision de l’intégrité des invités Azure Monitor pour machines virtuelles en utilisant des règles de collecte des données (préversion)
description: Décrit comment modifier la supervision par défaut de l’intégrité des invités Azure Monitor pour machines virtuelles à grande échelle en utilisant des modèles Resource Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/15/2020
ms.openlocfilehash: 55e5a587a0ad02fa1f8993027b46162a14a58832
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448251"
---
# <a name="configure-monitoring-in-azure-monitor-for-vms-guest-health-using-data-collection-rules-preview"></a>Configurer la supervision de l’intégrité des invités Azure Monitor pour machines virtuelles en utilisant des règles de collecte des données (préversion)
L’[intégrité des invités Azure Monitor pour machines virtuelles](vminsights-health-overview.md) vous permet de voir l’état d’intégrité d’une machine virtuelle tel que défini par un ensemble de mesures de performances échantillonnées à intervalles réguliers. Cet article décrit comment vous pouvez modifier la supervision par défaut sur plusieurs machines virtuelles en utilisant des règles de collecte des données.


## <a name="monitors"></a>Analyses
L’état d’intégrité d’une machine virtuelle est déterminé par le [cumul de l’intégrité](vminsights-health-overview.md#health-rollup-policy) de chacun de ses moniteurs. Il existe deux types de moniteurs dans l’intégrité des invités Azure Monitor pour machines virtuelles, comme indiqué dans le tableau suivant.

| Superviser | Description |
|:---|:---|
| Moniteur d’unités | Mesure certains aspects d’une ressource ou d’une application. Il peut s’agir de la vérification d’un compteur de performances pour déterminer le niveau de performance de la ressource ou sa disponibilité. |
| Moniteur agrégé | Regroupe plusieurs moniteurs pour fournir un seul état d’intégrité agrégé. Un moniteur agrégé peut contenir un ou plusieurs moniteurs d’unités et d’autres moniteurs agrégés. |

L’ensemble des moniteurs utilisés par l’intégrité des invités Azure Monitor pour machines virtuelles ne peut pas être modifié directement. Vous pouvez cependant créer des [remplacements](#overrides) qui modifient le comportement de la configuration par défaut. Les remplacements sont définis dans des règles de collecte des données. Vous pouvez créer plusieurs règles de collecte de données, chacune contenant plusieurs remplacements pour obtenir la configuration de supervision souhaitée.

## <a name="monitor-properties"></a>Propriétés des moniteurs
Le tableau suivant décrit les propriétés que vous pouvez configurer sur chaque moniteur.

| Propriété | Analyses | Description |
|:---|:---|:---|
| activé | Agrégat<br>Unité | Si la valeur est true, le moniteur d’état est calculé et contribue à l’intégrité de la machine virtuelle. Il peut déclencher une alerte si les alertes sont activées. |
| Génération d’alertes | Agrégat<br>Unité | Si la valeur est true, une alerte est déclenchée pour le moniteur quand il passe à un état non sain. Si la valeur est false, l’état du moniteur va contribuer à l’intégrité de la machine virtuelle qui a pu déclencher une alerte. |
| Avertissement | Unité | Critères pour l’état d’avertissement. S’il n’y en a pas, le moniteur n’entre jamais dans un état d’avertissement. |
| Critique | Unité | Critères pour l’état critique. S’il n’y en a pas, le moniteur n’entre jamais dans un état critique. |
| Fréquence d’évaluation | Unité | Fréquence à laquelle l’état d’intégrité est évalué. |
| Recherche arrière | Unité | Taille de la fenêtre de recherche arrière en secondes. Pour obtenir une description détaillée, consultez [Élément monitorConfiguration](#monitorconfiguration-element). |
| Type d’évaluation | Unité | Définit la valeur à utiliser dans l’ensemble d’échantillons. Pour obtenir une description détaillée, consultez [Élément monitorConfiguration](#monitorconfiguration-element). |
| Échantillon minimal | Unité | Nombre minimal de valeurs à utiliser pour calculer la valeur. |
| Échantillon maximal | Unité | Nombre maximal de valeurs à utiliser pour calculer la valeur. |


## <a name="default-configuration"></a>Configuration par défaut
Le tableau suivant liste la configuration par défaut pour chaque moniteur. Cette configuration par défaut ne peut pas être modifiée directement, mais vous pouvez définir des [remplacements](#overrides) qui vont modifier la configuration des moniteurs pour certaines machines virtuelles.


| Superviser | activé | Génération d’alertes | Avertissement | Critique | Fréquence d’évaluation | Recherche arrière | Type d’évaluation | Échantillon minimal | Échantillon maximal |
|:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|
| Utilisation du processeur  | True | False | None | \> 90 %    | 60 secondes | 240 secondes | Min | 2 | 3 |
| Mémoire disponible | True | False | None | \< 100 Mo | 60 secondes | 240 secondes | Max | 2 | 3 |
| Système de fichiers      | True | False | None | \< 100 Mo | 60 secondes | 120 secondes | Max | 1 | 1 |


## <a name="overrides"></a>Remplacements
Un *remplacement* modifie une ou plusieurs propriétés d’un moniteur. Par exemple, un remplacement peut désactiver un moniteur qui est activé par défaut, définir des critères d’avertissement pour le moniteur ou modifier le seuil critique du moniteur. 

Les remplacements sont définis dans une [règles de collecte des données](../platform/data-collection-rule-overview.md). Vous pouvez créer plusieurs règles de collecte des données avec différents ensembles de remplacements et les appliquer à plusieurs machines virtuelles. Vous appliquez une règle de collecte des données à une machine virtuelle en créant une association, comme décrit dans [Configurer la collecte des données pour l’agent Azure Monitor (préversion)](../platform/data-collection-rule-azure-monitor-agent.md#data-collection-rule-associations).


## <a name="multiple-overrides"></a>Remplacements multiples

Un même moniteur peut avoir plusieurs remplacements. Si les remplacements définissent des propriétés différentes, la configuration obtenue est une combinaison de tous les remplacements.

Par exemple, le moniteur `memory|available` ne spécifie pas de seuil d’avertissement ou n’active pas les alertes par défaut. Considérez les remplacements suivants appliqués à ce moniteur :

- Remplacement 1 définit la valeur de la propriété `alertConfiguration.isEnabled` sur `true`
- Remplacement 2 définit `monitorConfiguration.warningCondition` avec une condition de seuil de `< 250`.

La configuration résultante est un moniteur qui passe à un état d’intégrité d’avertissement quand moins de 250 Mo de mémoire sont disponibles, et qui crée une alerte de gravité 2 et passe à un état d’intégrité critique quand moins de 100 Mo de mémoire sont disponibles, et qui crée une alerte de gravité 1 (ou qui remplace l’alerte existante de gravité 2 par une alerte de gravité 1 si elle existait déjà).

Si deux remplacements définissent la même propriété sur le même moniteur, une des valeurs va être prioritaire. Les remplacements sont appliqués en fonction de leur [étendue](#scopes-element), du plus général au plus spécifique. Cela signifie que les remplacements les plus spécifiques auront la chance la plus grande d’être appliqués. L’ordre spécifique est le suivant :

1. Global 
2. Abonnement
3. Resource group
4. Une machine virtuelle. 

Si plusieurs remplacements au même niveau d’étendue définissent la même propriété sur le même moniteur, ils sont appliqués dans l’ordre où ils apparaissent dans la règle de collecte des données. Si les remplacements se trouvent dans des règles de collecte des données différentes, ils sont appliqués dans l’ordre alphabétique des ID de ressource des règles de collecte des données.


## <a name="data-collection-rule-configuration"></a>Configuration des règles de collecte des données
Les éléments JSON de la règle de collecte des données qui définissent les remplacements sont décrits dans les sections suivantes. Un exemple complet est fourni dans [Exemple de règle de collecte des données](#sample-data-collection-rule).

## <a name="extensions-structure"></a>Structure de l’élément extensions
L’intégrité des invités est implémentée en tant qu’extension de l’agent Azure Monitor : les remplacements sont donc définis dans l’élément `extensions` de la règle de collecte des données. 

```json
"extensions": [
    {
        "name": "Microsoft-VMInsights-Health",
        "streams": [
            "Microsoft-HealthStateChange"
        ],
        "extensionName": "HealthExtension",
        "extensionSettings": {   }
    }
]
```

| Élément | Obligatoire | Description |
|:---|:---|:---|
| `name` | Oui | Chaîne définie par l’utilisateur pour l’extension. |
| `streams` | Oui | Liste des flux auxquels les données d’intégrité des invités seront envoyées. Elle doit inclure **Microsoft-HealthStateChange**.  |
| `extensionName` | Oui | Nom de l’extension. Il doit être **HealthExtension**. |
| `extensionSettings` | Oui | Tableau d’éléments `healthRuleOverride` à appliquer à la configuration par défaut. |


## <a name="extensionsettings-element"></a>Élément extensionSettings
Contient les paramètres pour l’extension.

```json
"extensionSettings": {
    "schemaVersion": "1.0",
    "contentVersion": "",
    "healthRuleOverrides": [ ]
}
```

| Élément | Obligatoire | Description |
|:---|:---|:---|
| `schemaVersion` | Oui | Chaîne définie par Microsoft pour représenter le schéma attendu de l’élément. Elle doit actuellement être définie sur 1.0. |
| `contentVersion` | Non | Chaîne définie par l’utilisateur pour suivre les différentes versions de la configuration d’intégrité, si elle est nécessaire. |
| `healthRuleOverrides` | Oui | Tableau d’éléments `healthRuleOverride` à appliquer à la configuration par défaut. |

## <a name="healthrulesoverrides-element"></a>Élément healthRulesOverrides
Contient un ou plusieurs éléments `healthRuleOverride` qui définissent chacun un remplacement.

```json
"healthRuleOverrides": [
    {
        "scopes": [ ],
        "monitors": [ ],
        "monitorConfiguration": { },
        "alertConfiguration": {  },
        "isEnabled": true|false
    }
]
```

| Élément | Obligatoire | Description |
|:---|:---|:---|
| `scopes` | Oui | Liste d’une ou plusieurs étendues qui spécifient les machines virtuelles auxquelles ce remplacement est applicable. Même si la règle de collecte des données est associée à une machine virtuelle, cette machine virtuelle doit être dans une étendue pour que le remplacement soit appliqué. |
| `monitors` | Oui | Liste d’une ou plusieurs chaînes qui définissent les moniteurs qui vont recevoir ce remplacement.  |
| `monitorConfiguration` | Non | Configuration pour le moniteur, notamment les états d’intégrité et comment ils sont calculés. |
| `alertConfiguration` | Non | Configuration de l’alerte pour le moniteur. |
| `isEnabled` | Non | Contrôle si le moniteur est activé ou non. Un moniteur désactivé passe à un état d’intégrité *Désactivé* spécial et reste désactivé sauf s’il est réactivé. Si ce paramètre est omis, le moniteur va hériter son état du moniteur parent dans la hiérarchie. |


## <a name="scopes-element"></a>Élément scopes
Chaque remplacement a une ou plusieurs étendues qui définissent les machines virtuelles auxquelles le remplacement doit être appliqué. L’étendue peut être un abonnement, un groupe de ressources ou une machine virtuelle. Même si le remplacement se trouve dans une règle de collecte des données associée à une machine virtuelle particulière, il est appliqué seulement à cette machine virtuelle si elle se trouve dans une des étendues du remplacement. Ceci vous permet d’associer globalement un plus petit nombre de règles de collecte des données à un ensemble de machines virtuelles, tout en fournissant un contrôle précis sur l’affectation de chaque remplacement au sein de la règle de collecte des données elle-même. Vous pouvez créer un petit ensemble de règles de collecte des données et les associer à un ensemble de machines virtuelles en utilisant une stratégie, tout en spécifiant des remplacements du moniteur d’intégrité pour différents sous-ensembles de ces machines virtuelles en utilisant l’élément scopes.

Le tableau suivant montre des exemples de différentes étendues.

| Étendue |  Exemple |
|:---|:---|
| une seule machine virtuelle | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-name/providers/Microsoft.Compute/virutalMachines/my-vm` |
| Toutes les machines virtuelles d’un groupe de ressources | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-name` |
| Toutes les machines virtuelles dans un abonnement | `/subscriptions/00000000-0000-0000-0000-000000000000/` |
| Toutes les machines virtuelles auxquelles la règle de collecte des données est associée | `*` |


## <a name="monitors-element"></a>Élément monitors
Liste d’une ou plusieurs chaînes qui définissent les moniteurs dans la hiérarchie d’intégrité qui vont recevoir ce remplacement. Chaque élément peut être un nom de moniteur ou un nom de type qui correspond à un ou plusieurs moniteurs qui vont recevoir ce remplacement. 

```json
"monitors": [
    "<monitor name>"
 ],
```



Le tableau suivant liste les noms de moniteur actuellement disponibles.

| Nom de type | Nom | Description |
|:---|:---|:---|
| root | root | Moniteur de plus haut niveau représentant l’intégrité d’une machine virtuelle. | |
| cpu-utilization | cpu-utilization | Moniteur d’utilisation du processeur. | |
| logical-disks | logical-disks | Moniteur agrégé pour l’état d’intégrité de tous les disques supervisés sur une machine virtuelle Windows. | |
| logical-disks\|* | logical-disks\|C:<br>logical-disks\|D: | Moniteur agrégé suivant l’intégrité d’un disque donné sur une machine virtuelle Windows. | 
| logical-disks\|*\|free-space | logical-disks\|C:\|free-space<br>logical-disks\|D:\|free-space | Moniteur d’espace disque libre sur une machine virtuelle Windows. |
| filesystems | filesystems | Moniteur agrégé pour l’intégrité de tous les systèmes de fichiers sur une machine virtuelle Linux. |
| filesystems\|* | filesystems\|/<br>filesystems\|/mnt | Moniteur agrégé suivant l’intégrité d’un système de fichiers sur une machine virtuelle Windows. | filesystems|/var/log |
| filesystems\|*\|free-space | filesystems\|/\|free-space<br>filesystems\|/mnt\|free-space | Moniteur d’espace disque libre sur le système de fichiers d’une machine virtuelle Linux. | 
| mémoire | mémoire | Moniteur agrégé pour l’intégrité de la mémoire d’une machine virtuelle. | |
| memory\|available| memory\|available | Moniteur suivant la mémoire disponible sur la machine virtuelle. | |


## <a name="alertconfiguration-element"></a>Élément alertConfiguration
Spécifie si une alerte doit être créée à partir du moniteur.

```json
"alertConfiguration": {
    "isEnabled": true|false
}
```

| Élément | Obligatoire | Description | 
|:---|:---|:---|
| `isEnabled` | Non | Si la valeur est true, le moniteur va générer une alerte en cas de passage à un état critique ou d’avertissement, et va résoudre l’alerte lors du retour à un état sain. Si la valeur est false ou si elle est omise, aucune alerte n’est générée.  |


## <a name="monitorconfiguration-element"></a>Élément monitorConfiguration
S’applique uniquement aux moniteurs d’unités. Définit la configuration pour le moniteur, notamment les états d’intégrité et comment ils sont calculés.

Les paramètres définissent l’algorithme utilisé pour calculer la valeur de la métrique à comparer à des seuils. Au lieu d’agir sur un échantillon de données de la métrique sous-jacente, le moniteur évalue plusieurs échantillons de métrique reçus dans la fenêtre de temps comprise entre le moment de l’évaluation et une antériorité de `lookbackSec`. Tous les échantillons reçus dans ce laps de temps sont pris en compte et, si le nombre d’échantillons est supérieur à `maxSamples`, les échantillons plus anciens au-delà de `maxSamples` sont ignorés. 

Dans le cas où il y a moins d’échantillons dans l’intervalle d’antériorité que `minSamples`, le moniteur passe à un état d’intégrité *Inconnu*, indiquant qu’il n’y a pas assez de données pour prendre une décision informée sur l’intégrité des métriques sous-jacentes. Si le nombre d’échantillons disponibles est supérieur à `minSamples`, une fonction d’agrégation spécifiée par le paramètre evaluationType est exécutée sur l’ensemble pour calculer une valeur unique.


```json
"monitorConfiguration": {
        "evaluationType" : "<type-of-evaluation>",
        "lookbackSecs": <lookback-number-of-seconds>,
        "evaluationFrequencySecs": <evaluation-frequency-number-of-seconds>,
        "minSamples": <minimum-samples>,
        "maxSamples": <maximum-samples>,
        "warningCondition": {  },
        "criticalCondition": {  }
    }
}
```

| Élément | Obligatoire | Description | 
|:---|:---|:---|
| `evaluationFrequencySecs` | Non | Définit la fréquence d’évaluation de l’état d’intégrité. Chaque moniteur est évalué au moment du démarrage de l’agent et par la suite à un intervalle régulier défini par ce paramètre. |
| `lookbackSecs`   | Non | Taille de la fenêtre de recherche arrière en secondes. |
| `evaluationType` | Non | `min` : prendre la valeur minimale dans l’ensemble des échantillons<br>`max` : prendre la valeur maximale dans l’ensemble des échantillons<br>`avg` : prendre la moyenne des valeurs de l’ensemble des échantillons<br>`all` : comparer chaque valeur individuelle de l’ensemble aux seuils. Le moniteur change d’état si et seulement si tous les échantillons de l’ensemble satisfont à la condition de seuil. |
| `minSamples`     | Non | Nombre minimal de valeurs à utiliser pour calculer la valeur. |
| `maxSamples`     | Non | Nombre maximal de valeurs à utiliser pour calculer la valeur. |
| `warningCondition`  | Non | Seuil et logique de comparaison pour la condition d’avertissement. |
| `criticalCondition` | Non | Seuil et logique de comparaison pour la condition critique. |


## <a name="warningcondition-element"></a>Élément warningCondition
Définit le seuil et la logique de comparaison pour la condition d’avertissement. Si cet élément n’est pas inclus, le moniteur ne passe jamais à l’état d’intégrité d’avertissement.

```json
"warningCondition": {
    "isEnabled": true|false,
    "operator": "<comparison-operator>",
    "threshold": <threshold-value>
},
```

| Propriété | Obligatoire | Description | 
|:---|:---|:---|
| `isEnabled` | Non | Spécifie si la condition est activée. Si la valeur est **false**, la condition est désactivée même si les propriétés de seuil et d’opérateur peuvent être définies. |
| `threshold` | Non | Définit le seuil à comparer à la valeur évaluée. |
| `operator`  | Non | Définit l’opérateur de comparaison à utiliser dans l’expression de seuil. Valeurs possibles : >, <, >=, <=, ==. |


## <a name="criticalcondition-element"></a>Élément criticalCondition
Définit le seuil et la logique de comparaison pour la condition critique. Si cet élément n’est pas inclus, le moniteur ne passe jamais à l’état d’intégrité critique.

```json
"criticalCondition": {
    "isEnabled": true|false,
    "operator": "<comparison-operator>",
    "threshold": <threshold-value>
},
```

| Propriété | Obligatoire | Description | 
|:---|:---|:---|
| `isEnabled` | Non | Spécifie si la condition est activée. Si la valeur est **false**, la condition est désactivée même si les propriétés de seuil et d’opérateur peuvent être définies. |
| `threshold` | Non | Définit le seuil à comparer à la valeur évaluée. |
| `operator`  | Non | Définit l’opérateur de comparaison à utiliser dans l’expression de seuil. Valeurs possibles : >, <, >=, <=, ==. |

## <a name="sample-data-collection-rule"></a>Exemple de règle de collecte de données
Pour obtenir un exemple de règle de collecte de données permettant la surveillance des invités, consultez [Activer une machine virtuelle à l'aide du modèle Resource Manager](vminsights-health-enable.md#enable-a-virtual-machine-using-resource-manager-template).


## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [règles de collecte des données](../platform/data-collection-rule-overview.md).