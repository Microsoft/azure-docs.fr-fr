---
title: Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
description: Vue d’ensemble de la fonctionnalité d’intégrité d’Azure Monitor pour machines virtuelles, notamment la façon dont vous pouvez voir l’intégrité de vos machines virtuelles et recevoir des alertes quand une machine virtuelle devient non saine.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/27/2020
ms.openlocfilehash: 96bed9f3b04e54e2e9a5234d78f9a2660126742e
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686455"
---
# <a name="azure-monitor-for-vms-guest-health-preview"></a>Intégrité des invités Azure Monitor pour machines virtuelles (préversion)
L’intégrité des invités Azure Monitor pour machines virtuelles vous permet de voir l’état d’intégrité d’une machine virtuelle tel que défini par un ensemble de mesures de performances échantillonnées à des intervalles réguliers à partir du système d’exploitation invité. Vous pouvez vérifier rapidement l’intégrité de toutes les machines virtuelles d’un abonnement ou d’un groupe de ressources, explorer en détail l’état d’intégrité d’une machine virtuelle spécifique ou être averti de manière proactive quand une machine virtuelle devient non saine. 

## <a name="enable-virtual-machine-health"></a>Activer l’intégrité d’une machine virtuelle
Pour plus d’informations sur l’activation de la fonctionnalité d’intégrité des invités et l’intégration de machines virtuelles, consultez [Activer l’intégrité des invités Azure Monitor pour machines virtuelles (préversion)](vminsights-health-enable.md).

## <a name="pricing"></a>Tarifs
Il n’y a aucun coût direct pour la fonctionnalité d’intégrité des invités, mais il y a un coût pour l’ingestion et le stockage des données d’état d’intégrité dans l’espace de travail Log Analytics. Toutes les données sont stockées dans la table *HealthStateChangeEvent*. Pour plus d’informations sur les modèles tarifaires et les coûts, consultez [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](../platform/manage-cost-storage.md).

## <a name="view-virtual-machine-health"></a>Voir l’intégrité des machines virtuelles
La colonne **Intégrité de la machine virtuelle invitée** de la page **Démarrer** vous donne un aperçu rapide de l’intégrité de chaque machine virtuelle dans un abonnement ou un groupe de ressources particulier. L’intégrité actuelle de chaque machine virtuelle est affichée, des icônes pour chaque groupe montrant le nombre de machines virtuelles de ce groupe ayant ces états.

[![Page Démarrer avec l’intégrité des machines virtuelles invitées](media/vminsights-health-overview/get-started-page.png)](media/vminsights-health-overview/get-started-page.png#lightbox)


## <a name="monitors"></a>Analyses
Cliquez sur l’état d’intégrité d’une machine virtuelle pour afficher l’état détaillé de chacun de ses moniteurs. Chaque moniteur mesure l’intégrité d’un composant particulier. L’intégrité globale de la machine virtuelle est déterminée par l’intégrité de ses moniteurs individuels. 

![Exemples de moniteurs](media/vminsights-health-overview/monitors.png)

Le tableau suivant liste les moniteurs agrégés et les moniteurs d’unités actuellement disponibles pour chaque machine virtuelle. 

| Superviser | Type | Description |
|:---|:---|:---|
| Utilisation du processeur | Unité | Pourcentage d’utilisation du processeur. |
| Systèmes de fichiers | Agrégat | Intégrité agrégée de tous les systèmes de fichiers sur la machine virtuelle Linux. |
| Système de fichiers  | Agrégat | Intégrité de chaque système de fichiers individuel sur la machine virtuelle Linux. Le nom du moniteur est le nom du système de fichiers. |
| Espace libre | Unité | Pourcentage d’espace libre sur le système de fichiers. |
| Disques logiques | Agrégat | Intégrité agrégée de tous les disques logiques sur la machine virtuelle Windows. |
| Disque logique  | Agrégat | Intégrité de chaque disque logique individuel sur la machine virtuelle Windows. Le nom du moniteur est le nom du disque. |
| Mémoire | Agrégat | Intégrité agrégée de la mémoire sur la machine virtuelle. |
| Mémoire disponible | Unité | Mégaoctets disponibles sur la machine virtuelle. |

## <a name="health-states"></a>États d'intégrité
Chaque moniteur échantillonne des valeurs sur l’agent à chaque minute et compare les valeurs échantillonnées aux critères de chaque état d’intégrité. Si les critères de l’état particulier sont vrais, le moniteur est défini sur cet état. Si aucun des critères n’est vrai, le moniteur est défini sur un état « sain ». Les données sont envoyées de l’agent vers Azure Monitor toutes les trois minutes ou tout de suite après un changement de l’état.

Chaque moniteur a une fenêtre de recherche dans l’antériorité et analyse les échantillons collectés dans cette période. Par exemple, un moniteur peut avoir une fenêtre de recherche dans l’antériorité de 240 secondes avec une recherche de la valeur maximale parmi au moins 2 valeurs échantillonnées, mais pas plus que les 3 dernières. Si les valeurs sont échantillonnées à une fréquence régulière, le nombre d’échantillons dans une fenêtre particulière peut varier légèrement en cas d’interruption dans le fonctionnement de l’agent.

Les moniteurs ont chacun les états d’intégrité potentiels indiqués dans le tableau suivant, et ont un et un seul de ces états à un moment donné. Quand un moniteur est initialisé, il démarre dans un état sain.

| État d’intégrité | Description |
|:---|:---|
| Healthy  | Le moniteur ne dépasse pas le seuil d’avertissement ou le seuil critique. |
| Avertissement  | Le moniteur a dépassé le seuil d’avertissement (s’il est défini). |
| Critique | Le moniteur a dépassé le seuil critique (s’il est défini). |
| Inconnu  | Le nombre de données collectées est insuffisant pour déterminer l’état d’intégrité. |
| Désactivé | Le moniteur est actuellement désactivé. |
| None     | Le moniteur est simplement démarré, mais n’a pas encore été évalué ou l’objet supervisé n’existe plus. |



Il existe deux types de moniteurs, comme le montre le tableau suivant.

| Superviser | Description |
|:---|:---|
| Moniteur d’unités | Mesure certains aspects d’une ressource ou d’une application. Il peut s’agir de la vérification d’un compteur de performances pour déterminer le niveau de performance de la ressource ou sa disponibilité. |
| Moniteur agrégé | Regroupe plusieurs moniteurs pour fournir un seul état d’intégrité agrégé. Un moniteur agrégé peut contenir un ou plusieurs moniteurs d’unités et d’autres moniteurs agrégés. |


  
### <a name="health-rollup-policy"></a>Stratégie de cumul d’intégrité
L’état d’intégrité d’une machine virtuelle est déterminé par le cumul de l’intégrité de chacun de ses moniteurs d’unités et de ses moniteurs agrégés. Chaque moniteur agrégé utilise une stratégie de cumul d’intégrité du pire état dans laquelle l’état du moniteur agrégé correspond à l’état du moniteur enfant qui a le pire état d’intégrité.  

Dans l’exemple suivant, le moniteur **Espace libre** est dans un état critique qui est cumulé aux agrégations pour son instance, puis pour **Systèmes de fichiers**. Même si **Utilisation du processeur** est dans un état d’avertissement, la machine virtuelle est définie sur « critique », car il s’agit du pire état sous-celle-ci. Si l’espace libre devait revenir à un état sain, la machine virtuelle passerait à un état d’avertissement, car l’utilisation du processeur deviendrait alors le moniteur avec le pire état.

![Exemple de rollup d’intégrité](media/vminsights-health-overview/health-rollup-example.png)


## <a name="monitor-details"></a>Détails du moniteur
Sélectionnez un moniteur pour voir ses détails, qui comprennent les onglets suivants.

**Vue d’ensemble** fournit une description du moniteur, la dernière fois qu’il a été évalué et les valeurs échantillonnées pour déterminer l’état d’intégrité actuel. Le nombre d’échantillons peut varier en fonction de la définition du moniteur et de la volatilité des valeurs.

[![Vue d’ensemble des détails du moniteur](media/vminsights-health-overview/monitor-details-overview.png)](media/vminsights-health-overview/monitor-details-overview.png#lightbox)

**Historique** liste l’historique des changements d’état pour le moniteur. Vous pouvez développer un des changements d’état pour voir les valeurs évaluées afin de déterminer l’état d’intégrité. Cliquez sur **Voir la configuration appliquée** pour voir la configuration du moniteur au moment où l’état d’intégrité a changé.



[![Historique des détails du moniteur](media/vminsights-health-overview/monitor-details-history.png)](media/vminsights-health-overview/monitor-details-history.png#lightbox)

### <a name="configuration"></a>Configuration
Affichez et modifiez la configuration du moniteur pour la machine virtuelle sélectionnée. Pour plus d’informations, consultez [Configurer la supervision de l’intégrité des invités Azure Monitor pour machines virtuelles](vminsights-health-enable.md).

[![Configuration des détails du moniteur](media/vminsights-health-overview/monitor-details-configuration.png)](media/vminsights-health-overview/monitor-details-configuration.png#lightbox)




## <a name="next-steps"></a>Étapes suivantes

- [Activer l’intégrité des invités dans Azure Monitor pour machines virtuelles et intégrer des agents](vminsights-health-enable.md)
- [Configurer des moniteurs en utilisant le portail Azure](vminsights-health-configure.md)
- [Configurer des moniteurs en utilisant des règles de collecte des données](vminsights-health-configure-dcr.md)