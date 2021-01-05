---
title: Superviser vos sauvegardes avec l’Explorateur de sauvegarde
description: Cet article explique comment utiliser l'Explorateur de sauvegarde pour superviser les sauvegardes en temps réel dans l'ensemble des coffres, abonnements, régions et locataires.
ms.reviewer: dcurwin
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: a50b9ee05be48113221f2a12f968540bd3a00b3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88824410"
---
# <a name="monitor-your-backups-with-backup-explorer"></a>Superviser vos sauvegardes avec l’Explorateur de sauvegarde

Sachant que les organisations sauvegardent de plus en plus de machines dans le cloud, il est désormais essentiel de superviser efficacement ces sauvegardes. La meilleure façon de commencer est d'avoir recours à un emplacement centralisé pour visualiser les informations opérationnelles d'un parc étendu.

L'Explorateur de sauvegarde est un classeur intégré à Azure Monitor qui permet aux clients du service Sauvegarde Azure de bénéficier de cet emplacement centralisé. L'Explorateur de sauvegarde vous aide à superviser les activités opérationnelles de l'ensemble du parc de sauvegarde Azure, en couvrant les locataires, les emplacements, les abonnements, les groupes de ressources et les coffres. Les principales fonctionnalités de l'Explorateur de sauvegarde sont les suivantes :

* **Perspective à grande échelle** : bénéficiez d'une vue agrégée des éléments, travaux, alertes, stratégies et ressources de sauvegarde qui n'ont pas encore été configurés pour la sauvegarde dans l'ensemble du parc.
* **Analyse approfondie** : consultez des informations détaillées sur vos travaux, alertes, stratégies et éléments de sauvegarde à un emplacement centralisé.
* **Interfaces interactives** : après avoir identifié un problème, vous pouvez le résoudre en accédant facilement à l'élément de sauvegarde ou à la ressource Azure concerné.

Ces fonctionnalités sont fournies prêtes à l'emploi par une intégration native à Azure Resource Graph et aux classeurs Azure Monitor.

> [!NOTE]
>
> * Pour le moment, l'Explorateur de sauvegarde est uniquement disponible pour les données des machines virtuelles Azure.
> * L'Explorateur de sauvegarde a été conçu comme un tableau de bord opérationnel qui permet de visualiser les informations relatives à vos sauvegardes des 7 derniers jours (maximum).
> * L’Explorateur de sauvegarde n’est actuellement pas pris en charge sur les clouds nationaux.
> * Il est actuellement impossible de personnaliser le modèle de l’Explorateur de sauvegarde.
> * Nous vous déconseillons de créer des automatisations personnalisées sur les données Azure Resource Graph.
> * Actuellement, l’Explorateur de sauvegarde vous permet de surveiller les sauvegardes sur un maximum de 1 000 abonnements (sur tous les locataires).

## <a name="get-started"></a>Bien démarrer

Pour ouvrir l'Explorateur de sauvegarde, accédez à l'un de vos coffres Recovery Services, puis sélectionnez le lien **Explorateur de sauvegarde** dans le volet **Vue d'ensemble**.

![Lien rapide vers le coffre](media/backup-azure-monitor-with-backup-explorer/vault-quick-link.png)

Sélectionnez le lien pour ouvrir l'Explorateur de sauvegarde et bénéficier d'une vue agrégée de tous les coffres et abonnements auxquels vous avez accès. Si vous utilisez un compte Azure Lighthouse, vous pouvez voir les données de tous les locataires auxquels vous avez accès. Pour plus d'informations, consultez la section « Vues multilocataires » à la fin de cet article.

![Page d'arrivée de l'Explorateur de sauvegarde](media/backup-azure-monitor-with-backup-explorer/explorer-landing-page.png)

## <a name="backup-explorer-use-cases"></a>Explorateur de sauvegarde - Cas d'usage

L'Explorateur de sauvegarde contient plusieurs onglets, chacun fournissant des informations détaillées sur un artefact de sauvegarde spécifique (par exemple, un élément, un travail ou une stratégie de sauvegarde). Cette section fournit une brève présentation de chacun des onglets. Les vidéos fournissent des exemples de cas d'usage pour chaque artefact de sauvegarde, ainsi que des descriptions des contrôles disponibles.

### <a name="the-summary-tab"></a>L'onglet Résumé

L'onglet **Résumé** offre un aperçu rapide de l'état global de votre parc de sauvegarde. Par exemple, vous pouvez voir le nombre d'éléments protégés, le nombre d'éléments pour lesquels la protection n'a pas été activée ou le nombre de travaux ayant abouti au cours des 24 dernières heures.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQYd]

### <a name="the-backup-items-tab"></a>L'onglet Éléments de sauvegarde

Vous pouvez filtrer et afficher chacun de vos éléments de sauvegarde par abonnement, par coffre et autres caractéristiques. Sélectionnez le nom d'un élément de sauvegarde pour ouvrir le volet Azure correspondant à cet élément. Par exemple, dans le tableau, vous pouvez remarquer que la dernière sauvegarde de l'élément *X* a échoué. En sélectionnant *X*, vous accédez au volet **Sauvegarde** de l'élément, où vous pouvez déclencher une opération de sauvegarde à la demande.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQYc]

### <a name="the-jobs-tab"></a>L'onglet Travaux

Sélectionnez l'onglet **Travaux** pour afficher les détails de tous les travaux qui ont été déclenchés au cours des 7 derniers jours. Vous pouvez ici appliquer les filtres suivants : *Opération de travail*, *État du travail* et *Code d'erreur* (pour les travaux qui ont échoué).

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nOrh]

### <a name="the-alerts-tab"></a>L'onglet Alertes

Sélectionnez l'onglet **Alertes** pour afficher les détails de toutes les alertes générées sur vos coffres au cours des 7 derniers jours. Vous pouvez filtrer les alertes par type (*Échec de la sauvegarde* ou *Échec de la restauration*), par état actuel (*Active* ou *Résolue*) et par gravité (*Critique*, *Avertissement* ou *Information*). Vous pouvez également sélectionner un lien pour accéder à la machine virtuelle Azure et prendre les éventuelles mesures nécessaires.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nTxe]

### <a name="the-policies-tab"></a>L'onglet Stratégies

Sélectionnez l'onglet **Stratégies** pour consulter des informations sur toutes les stratégies de sauvegarde qui ont été créées dans votre parc de sauvegarde. Vous pouvez voir le nombre d'éléments associés à chaque stratégie, ainsi que la durée de conservation et la fréquence de sauvegarde spécifiées par la stratégie.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nLKV]

### <a name="the-backup-not-enabled-tab"></a>L'onglet Sauvegarde non activée

La sauvegarde doit être activée pour toutes les machines qui nécessitent une protection. Grâce à l’Explorateur de sauvegarde, les administrateurs de sauvegarde peuvent rapidement identifier les machines d’une organisation qui ne sont pas encore protégés par une sauvegarde. Pour afficher ces informations, sélectionnez l'onglet **Sauvegarde non activée**.

Le volet **Sauvegarde non activée** affiche la liste de machines non protégées sous forme de tableau. Votre organisation peut attribuer différentes étiquettes aux machines de production et aux machines de test, ou aux machines qui remplissent diverses fonctions. Dans la mesure où chaque classe de machines requiert une stratégie de sauvegarde distincte, le filtrage par étiquette vous permet d'afficher les informations spécifiques à chacune. La sélection du nom d'une machine vous redirige vers le volet **Configurer la sauvegarde** de cette machine, où vous pouvez choisir d'appliquer une stratégie de sauvegarde appropriée.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4nQXZ]

## <a name="export-to-excel"></a>Exporter vers Excel

Vous pouvez exporter le contenu de n'importe quel tableau ou graphique sous forme de feuille de calcul Excel. Le contenu est exporté tel quel, avec les filtres que vous y avez appliqués. Pour exporter des lignes de tableau supplémentaires, vous pouvez augmenter le nombre de lignes à afficher sur la page à l'aide de la liste déroulante **Lignes par page** située en haut de chaque onglet.

## <a name="pin-to-the-dashboard"></a>Épingler sur le tableau de bord

Sélectionnez l'icône « Épingler » disponible en haut de chaque tableau ou graphique pour épingler celui-ci sur le tableau de bord du portail Azure. L'épinglage de ces informations vous permettra de créer un tableau de bord personnalisé contenant les informations qui comptent les plus pour vous.

## <a name="cross-tenant-views"></a>Vues multilocataires

Si vous êtes un utilisateur d'Azure Lighthouse et que vous disposez d'un accès délégué aux abonnements de plusieurs environnements de locataires, vous pouvez utiliser le filtre d'abonnement par défaut. Pour afficher les abonnements dont vous souhaitez consulter les données, sélectionnez l'icône « Filtre » en haut à droite du portail Azure. Lorsque vous utilisez cette fonctionnalité, l'Explorateur de sauvegarde regroupe les informations de tous les coffres répartis sur les différents abonnements que vous avez sélectionnés. Pour en savoir plus, consultez [Présentation d'Azure Lighthouse](../lighthouse/overview.md).

## <a name="next-steps"></a>Étapes suivantes

[Découvrir comment utiliser Azure Monitor pour obtenir des insights sur vos données de sauvegarde](./backup-azure-monitoring-use-azuremonitor.md)
