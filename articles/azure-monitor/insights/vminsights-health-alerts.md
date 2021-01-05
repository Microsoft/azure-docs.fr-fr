---
title: Alertes d’Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
description: Décrit les alertes créées par Intégrité des invités Azure Monitor pour machines virtuelles, notamment la façon de les activer et de configurer les notifications.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/10/2020
ms.openlocfilehash: 9bbb0be321e78317c52586454de3d49cb2f878c1
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686410"
---
# <a name="azure-monitor-for-vms-guest-health-alerts-preview"></a>Alertes d’Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
La fonctionnalité Intégrité des invités Azure Monitor pour machines virtuelles vous permet de voir l’état d’intégrité d’une machine virtuelle tel que défini par un ensemble de mesures de performances échantillonnées à des intervalles réguliers. Une alerte peut être créée lorsqu’une machine virtuelle ou un moniteur passe à un état défectueux. Vous pouvez afficher et gérer ces alertes avec [celles créées par les règles d’alerte dans Azure Monitor](../platform/alerts-overview.md) et choisir d’être informé de manière proactive lors de la création d’une nouvelle alerte.

## <a name="configure-alerts"></a>Configurer des alertes
Vous ne pouvez pas créer de règle d’alerte explicite pour Intégrité des invités Azure Monitor pour machines virtuelles tant que cette fonctionnalité est en préversion. Par défaut, les alertes sont créées pour chaque machine virtuelle, mais pas pour chaque moniteur.  Cela signifie que si un moniteur passe à un état qui ne modifie pas l’état actuel de la machine virtuelle, aucune alerte n’est créée, car l’état de la machine virtuelle n’a pas changé. 

Vous pouvez désactiver les alertes pour une machine virtuelle particulière ou pour un moniteur particulier sur une machine virtuelle à partir du paramètre **État de l’alerte** dans la configuration de la machine virtuelle sur le portail Azure. Pour plus d’informations sur la configuration des analyses sur le portail Azure, consultez [Configurer la supervision dans Intégrité des invités Azure Monitor pour machines virtuelles (préversion)](vminsights-health-configure.md). Pour plus d’informations sur la configuration des analyses sur un ensemble de machines virtuelles, consultez [Configurer la supervision dans Intégrité des invités Azure Monitor pour machines virtuelles à l’aide de règles de collecte des données (préversion)](vminsights-health-configure-dcr.md).

## <a name="alert-severity"></a>Gravité de l’alerte
La gravité de l’alerte créée par l’intégrité des invités correspond directement à la gravité de la machine virtuelle ou du moniteur déclenchant l’alerte.

| État du moniteur | Gravité de l’alerte |
|:---|:---|
| Critique | Gra1 |
| Avertissement  | Gra2 |
| Healthy  | Sev4 |

## <a name="alert-lifecycle"></a>Cycle de vie des alertes
Une [alerte Azure](../platform/alerts-overview.md) est créée pour chaque machine virtuelle chaque fois qu’elle passe à un état **Avertissement** ou **Critique**. Affichez l’alerte à partir d’**Alertes** dans le menu **Azure Monitor** ou dans le menu de la machine virtuelle sur le portail Azure.

Si une alerte est déjà à l’état **Déclenché** lorsque l’état de la machine virtuelle change, une deuxième alerte n’est pas créée, mais la gravité de la même alerte est modifiée pour correspondre à l’état de la machine virtuelle. Par exemple, si la machine virtuelle passe à l’état **Critique** quand une alerte **Avertissement** était déjà à l’état **Déclenché**, la gravité de cette alerte passera à **SEV1**. Si la machine virtuelle passe à l’état **Avertissement** quand une alerte **Sev1** était déjà à l’état **Déclenché**, la gravité de cette alerte passera à **Sev2**. Si la machine virtuelle revient à un état **Sain**, l’alerte sera résolue et la gravité passera à **Sev4**.

## <a name="viewing-alerts"></a>Affichage des alertes
Affichez les alertes créées par Intégrité des invités Azure Monitor pour machines virtuelles avec d’autres [alertes dans le portail Azure](../platform/alerts-overview.md#alerts-experience). Vous pouvez sélectionner **Alertes** dans le menu **Azure Monitor** pour afficher les alertes de toutes les ressources surveillées ou sélectionner **Alertes** dans le menu d’une machine virtuelle pour afficher les alertes de cette machine virtuelle uniquement.

## <a name="alert-properties"></a>Propriétés de l'alerte

### <a name="properties-in-the-azure-portal"></a>Propriétés dans le portail Azure
Les propriétés de l’alerte lorsque vous la visualisez dans le portail Azure sont décrites dans le tableau suivant.

| Propriété | Description |
|:---|:---|
| État du moniteur avant la création de l’alerte | État du moniteur ou de la machine virtuelle avant que cette alerte soit déclenchée la première fois. |
| État du moniteur lors de la création de l’alerte | État du moniteur ou de la machine virtuelle lorsque l’alerte a été déclenchée pour la première fois. Il s’agit de l’état qui a provoqué le déclenchement de l’alerte. |
| Pour en savoir plus sur la transition d’état lors de la création d’une alerte | Lien vers la page Intégrité de la machine virtuelle où vous pouvez voir la transition d’état exacte. Cette transition d’état représente l’instance lors du premier passage du moniteur de l’état **Sain** à un état de non sain. |

Par exemple, un moniteur passe de l’état **Sain** à **Critique** à l’heure t0, et une nouvelle alerte est déclenchée avec **Sev1**. Il passe ensuite de l’état **Critique** à **Avertissement** à l’heure t1, et la gravité de l’alerte est mise à jour à **Sev2**. Il devient **Sain** à l’heure t2, et l’alerte est résolue.

Les propriétés de l’alerte comporteront ces valeurs pendant la totalité de la séquence.

- État du moniteur avant la création de l’alerte : Healthy
- État du moniteur lors de la création de l’alerte : Critique
- Pour en savoir plus sur la transition d’état lors de la création d’une alerte : Le lien de navigation vers la transition d’état s’est produit à l’heure t0.


### <a name="properties-in-notifications"></a>Propriétés dans les notifications
Les propriétés de l’alerte incluses dans les notifications sont décrites dans le tableau suivant.

| Propriété | Description |
|:---|:---|
| État précédent du moniteur | État du moniteur ou de la machine virtuelle avant la modification de l’état. Si la mise à jour de la gravité d’alerte déclenche cette notification, cette propriété représente l’état juste avant la mise à jour de la gravité. |
| État actuel du moniteur | État du moniteur ou de la machine virtuelle après la modification de l’état. Si la mise à jour de la gravité d’alerte déclenche cette notification, cette propriété représente le nouvel état. |
| Pour en savoir plus sur cette transition d’état | Lien vers la page Intégrité de la machine virtuelle où vous pouvez voir la transition d’état exacte. Cette transition d’état représente l’instance lorsque l’état du moniteur a changé, ce qui a déclenché cette notification. |

En utilisant l’exemple ci-dessus, les notifications ont les propriétés suivantes à chaque fois.

Notification reçue à l’heure t0
- État précédent du moniteur : Healthy
- État actuel du moniteur : Critique
- Pour en savoir plus sur cette transition d’état : Le lien de navigation vers la transition d’état s’est produit à l’heure t0.

Notification reçue à l’heure t1
- État précédent du moniteur : Critique
- État actuel du moniteur : Avertissement
- Pour en savoir plus sur cette transition d’état : Le lien de navigation vers la transition d’état s’est produit à l’heure t1.

Notification reçue à l’heure t2
- État précédent du moniteur : Avertissement
- État actuel du moniteur : Healthy
- Pour en savoir plus sur cette transition d’état : Le lien de navigation vers la transition d’état s’est produit à l’heure t2.

## <a name="configure-notifications"></a>Configurer les notifications
Pour être averti de manière proactive d’une alerte déclenchée par l’intégrité des invités, créez un [groupe d’actions](../platform/action-groups.md) pour définir les différentes actions à effectuer, telles que l’envoi d’un SMS ou le démarrage d’une application logique. Créez ensuite une [règle d’action](../platform/alerts-action-rules.md) qui spécifie l’étendue des moniteurs et des machines virtuelles et qui utilise ce groupe d’actions.

Dans le menu **Superviser** du portail Azure, sélectionnez **Alertes**.  Sélectionnez **Gérer les actions**, puis **Règles d’action (préversion)** . 

![Nouvelle règle d’action](media/vminsights-health-alerts/action-rule-new.png)

Cliquez sur **Nouvelle règle d’action** pour créer une nouvelle règle. Cliquez sur **Sélectionner** à côté d’Étendue et sélectionnez un abonnement, un groupe de ressources ou une ou plusieurs machines virtuelles spécifiques. La notification sera déclenchée uniquement pour les machines virtuelles qui se trouvent dans l’étendue.

![Étendue de la règle action](media/vminsights-health-alerts/action-rule-scope.png)

Cliquez sur **Ajouter** à côté de **Filtre**. Créez un filtre dans lequel **le service du moniteur est égal à VM Insights – Intégrité**. Ajoutez d’autres filtres pour spécifier les alertes spécifiques qui doivent déclencher la notification. Par exemple, vous pouvez utiliser **Gravité** pour faire correspondre les alertes de tous les moniteurs qui correspondent à une gravité particulière.

![Filtre de règle d’action](media/vminsights-health-alerts/action-rule-filter.png)

Dans **Définir sur cette étendue**, sélectionnez **Groupe d’actions**, puis sélectionnez le groupe d’actions à associer au moniteur. Donnez un nom à la règle et sélectionnez le groupe de ressources dans lequel elle doit être enregistrée. Cliquez sur **Créer** pour créer la règle.

![Règle d’action](media/vminsights-health-alerts/action-rule.png)


## <a name="next-steps"></a>Étapes suivantes

- [Activer l’intégrité des invités dans Azure Monitor pour machines virtuelles et intégrer des agents](vminsights-health-enable.md)
- [Configurer des moniteurs en utilisant le portail Azure](vminsights-health-configure.md)
- [Configurer des moniteurs en utilisant des règles de collecte des données](vminsights-health-configure-dcr.md)