---
title: Configurer la supervision d’Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
description: Décrit comment modifier la supervision par défaut d’Intégrité des invités Azure Monitor pour machines virtuelles (préversion) à l’aide du portail Azure.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/14/2020
ms.openlocfilehash: 427bdec2b5e5ab14d566375d5ad8f9da9dc3e81b
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505594"
---
# <a name="configure-monitoring-in-azure-monitor-for-vms-guest-health-preview"></a>Configurer la supervision d’Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
La fonctionnalité Intégrité des invités Azure Monitor pour machines virtuelles vous permet de voir l’état d’intégrité d’une machine virtuelle tel que défini par un ensemble de mesures de performances échantillonnées à des intervalles réguliers. Cet article décrit la façon dont vous pouvez modifier la supervision par défaut à l’aide du portail Azure. Il décrit également les concepts fondamentaux des moniteurs nécessaires à la [configuration de la supervision à l’aide d’une règle de collecte de données](vminsights-health-configure-dcr.md).

## <a name="open-monitor-configuration"></a>Ouvrir la configuration du moniteur
Ouvrez la configuration du moniteur dans le portail Azure en sélectionnant le moniteur, puis l’onglet **Configuration**.

[![Configuration des détails du moniteur](media/vminsights-health-overview/monitor-details-configuration.png)](media/vminsights-health-overview/monitor-details-configuration.png#lightbox)

## <a name="enable-or-disable-monitors"></a>Activer ou désactiver des moniteurs
Les moniteur d’unités et les moniteurs d’agrégats ont un paramètre **État du moniteur d’intégrité** qui vous permet d’activer ou de désactiver le moniteur. Lorsqu’un moniteur est activé, son état d’intégrité est affiché et utilisé pour définir l’intégrité de la machine virtuelle. Lorsqu’un moniteur est désactivé, son intégrité n’est pas calculée et n’est pas utilisée pour définir l’intégrité de la machine virtuelle.

| Paramètre | Description |
|:---|:---|
| activé | Le moniteur est activé quel que soit le paramètre de son parent. |
| Désactivé | Le moniteur est désactivé quel que soit le paramètre de son parent. |
| Identique au parent | Le moniteur sera activé ou désactivé en fonction du paramètre de son parent. |

Quand un moniteur est désactivé, tous les critères sont affichés comme étant non disponibles, comme dans l’exemple suivant.

![Moniteur désactivé](media/vminsights-health-configure/disabled-monitor.png)


> [!NOTE]
> Si un moniteur parent est désactivé, tous les moniteurs enfants le sont également. Si vous activez explicitement le moniteur enfant, le parent est également activé, mais son état de configuration reste le même. Dans ce cas, vous recevrez le message suivant dans le moniteur parent.
>
> *Il existe une incohérence dans la mesure où l’état configuré du moniteur est « désactivé », alors que l’état d’intégrité ne reflète pas cela. En effet, soit les modifications configurées sont propagées, soit un de ses moniteurs enfants a été activé de manière explicite.*

## <a name="enable-or-disable-virtual-machine"></a>Activer ou désactiver la machine virtuelle
Vous pouvez désactiver la supervision d’une machine virtuelle pour arrêter temporairement tous les moniteurs. Vous pouvez désactiver la supervision d’une machine virtuelle, par exemple, lorsque vous effectuez une opération de maintenance sur celle-ci.

| Paramètre | Description |
|:---|:---|
| activé  | L’état d’intégrité de l’ordinateur s’affiche. |
| Désactivé | L’état d’intégrité de l’ordinateur s’affiche comme étant **désactivé**. Les alertes ne sont pas créées. |

## <a name="health-state-logic"></a>Logique de l’état d’intégrité
La logique de l’état d’intégrité d’un moniteur d’unités détermine les critères de définition de son état d’intégrité. Vous pouvez spécifier le nombre d’états d’intégrité dont dispose un moniteur et le seuil de calcul de chaque état d’intégrité.

![Exemples de critères d’intégrité](media/vminsights-health-configure/sample-health-criteria.png)

Les moniteurs d’agrégat ne spécifient aucune logique d’état d’intégrité. Leur état d’intégrité est défini par les moniteurs d’unités qui leur sont subordonnés en fonction du cumul de leur état d’intégrité.

Les moniteurs d’unités ont chacun deux ou trois états d’intégrité. Chacun aura toujours un état Sain et éventuellement un état Avertissement, un état Critique, ou les deux. Les états Critique et Avertissement requièrent chacun des critères uniques qui définissent quand le moniteur est réglé sur cet état. L’état Sain n’a pas de critère, car cet état est défini lorsque les critères pour les autres états ne sont pas satisfaits.

Dans l’exemple suivant, l’utilisation de l’UC est réglée sur les états d’intégrité suivants :

- Critique si sa valeur est supérieure à 90 %.
- Avertissement si sa valeur est supérieure ou égale à 80 %.
- Sain si sa valeur est inférieure à 80 %. Cela est implicite, car il s’agit de la condition où aucune des autres conditions ne s’applique.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer des moniteurs à grande échelle en utilisant des règles de collecte des données](vminsights-health-configure-dcr.md)