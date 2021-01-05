---
title: Résolution des problèmes rencontrés avec les graphiques de métriques Azure Monitor
description: Résoudre les problèmes liés à la création, à la personnalisation ou à l’interprétation des graphiques de métriques
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: 7c3af0865282475ded0172d18aecad1dfb61721b
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97814259"
---
# <a name="troubleshooting-metrics-charts"></a>Résolution des problèmes liés aux graphiques de métriques

Utilisez cet article si vous rencontrez des problèmes lors de la création, de la personnalisation ou de l’interprétation de graphiques dans Azure Metrics Explorer. Si vous débutez avec les métriques, découvrez comment [bien démarrer avec Metrics Explorer](metrics-getting-started.md) et les [fonctionnalités avancées de Metrics Explorer](metrics-charts.md). Vous pouvez également consulter des [exemples](metric-chart-samples.md) des graphiques de métriques configurés.

## <a name="chart-shows-no-data"></a>Le graphique n’affiche aucune donnée.

Parfois, vous sélectionnez les métriques et les ressources appropriées, mais les graphiques n’affichent aucune donnée. Ce comportement peut être dû aux raisons suivantes :

### <a name="microsoftinsights-resource-provider-isnt-registered-for-your-subscription"></a>Le fournisseur de ressources Microsoft.Insights n’est pas inscrit pour votre abonnement.

L’exploration de métriques nécessite que le fournisseur de ressources *Microsoft.Insights* soit inscrit dans votre abonnement. Dans de nombreux cas, il est inscrit automatiquement (une fois que vous avez configuré une règle d’alerte, que vous avez personnalisé les paramètres de diagnostic pour une ressource quelconque ou que vous avez configuré une règle de mise à l’échelle automatique). Si le fournisseur de ressources Microsoft.Insights n’est pas inscrit, vous devez l’inscrire manuellement en suivant les étapes décrites dans [Fournisseurs et types de ressources Azure](../../azure-resource-manager/management/resource-providers-and-types.md).

**Solution :** Sous **Abonnements**, ouvrez l’onglet **Fournisseurs de ressources** et vérifiez que *Microsoft.Insights* est inscrit pour votre abonnement.

### <a name="you-dont-have-sufficient-access-rights-to-your-resource"></a>Vous ne disposez pas de droits d’accès à votre ressource suffisants.

Dans Azure, l’accès aux métriques est contrôlé par le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Vous devez être membre du [lecteur de surveillance](../../role-based-access-control/built-in-roles.md#monitoring-reader), du [contributeur de surveillance](../../role-based-access-control/built-in-roles.md#monitoring-contributor) ou du [contributeur](../../role-based-access-control/built-in-roles.md#contributor) pour explorer les métriques d’une ressource, quelle qu’elle soit.

**Solution :** Assurez-vous que vous disposez des autorisations suffisantes pour la ressource dont vous explorez les métriques.

### <a name="your-resource-didnt-emit-metrics-during-the-selected-time-range"></a>Votre ressource n’a émis aucune métrique sur l’intervalle de temps sélectionné.

Certaines ressources n’émettent pas leurs métriques en permanence. Par exemple, Azure ne collecte pas les métriques des machines virtuelles arrêtées. D’autres ressources peuvent émettre leurs métriques uniquement lorsque certaines conditions se produisent. Par exemple, une métrique indiquant le temps de traitement d’une transaction requiert au moins une transaction. Si aucune transaction n’a eu lieu sur l’intervalle de temps sélectionné, le graphique est naturellement vide. En outre, si la plupart des métriques dans Azure sont collectées toutes les minutes, certaines sont collectées moins fréquemment. Consultez la documentation relative aux métriques pour obtenir plus d’informations sur la métrique que vous tentez d’explorer.

**Solution :** Augmentez l’intervalle de temps sur lequel porte le graphique. Vous pouvez commencer à partir des « 30 derniers jours » en utilisant une granularité temporelle supérieure ou en vous appuyant sur l’option « Automatic time granularity » (Granularité temporelle automatique).

### <a name="you-picked-a-time-range-greater-than-30-days"></a>Vous avez choisi un intervalle de temps supérieur à 30 jours.

[La plupart des métriques dans Azure sont stockées 93 jours](data-platform-metrics.md#retention-of-metrics). Toutefois, vos interrogations peuvent porter sur 30 jours de données au maximum pour un même graphique. Cette limitation ne s’applique pas aux [métriques reposant sur un journal](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics).

**Solution :** Si le graphique est vide ou affiche uniquement une partie des données de métrique, vérifiez que l’intervalle entre les dates de début et de fin dans le sélecteur d’heure ne dépasse pas 30 jours.

### <a name="all-metric-values-were-outside-of-the-locked-y-axis-range"></a>Toutes les valeurs de métrique étaient en dehors de la plage verrouillée de l’axe y.

En [verrouillant les limites de l’axe y du graphique](metrics-charts.md#lock-boundaries-of-chart-y-axis), vous pouvez involontairement empêcher l’affichage de la courbe du graphique sur la zone d’affichage. Par exemple, si l’axe y est verrouillé sur une plage comprise entre 0 et 50 % et que la métrique présente une valeur constante de 100 %, la courbe est toujours rendue en dehors de la zone visible. Le graphique affiché est donc vide.

**Solution :** Vérifiez que les limites de l’axe y du graphique ne sont pas verrouillées en dehors de la plage des valeurs de métrique. Si les limites de l’axe y sont verrouillées, il peut être utile de les réinitialiser temporairement pour vous assurer que les valeurs de métrique ne sont pas en dehors de la plage du graphique. Il n’est pas recommandé de verrouiller la plage de l’axe y avec une granularité automatique pour les graphiques avec une agrégation de type **Somme**, **Min** et **Max**, car leurs valeurs changent avec la granularité si vous redimensionnez la fenêtre du navigateur ou changez de résolution d’écran. Si vous modifiez la granularité, la zone d’affichage de votre graphique peut être vide.

### <a name="you-are-looking-at-a-guest-os-metric-but-didnt-enable-azure-diagnostic-extension"></a>Vous examinez une métrique de SE invité, mais vous n’avez pas activé l’extension Azure Diagnostics.

La collecte des métriques de **SE invité** nécessite la configuration de l’extension Diagnostics Azure ou son activation à l’aide du panneau **Paramètres de diagnostic** de votre ressource.

**Solution :** Si l’extension Diagnostics Azure est activée mais que vos métriques ne s’affichent toujours pas, suivez les étapes décrites dans le [guide de résolution des problèmes liés à l’extension Azure Diagnostics](diagnostics-extension-troubleshooting.md#metric-data-doesnt-appear-in-the-azure-portal). Consultez également les étapes de résolution des problèmes à la section [Impossible de choisir l’espace de noms et les métriques de SE invité](#cannot-pick-guest-os-namespace-and-metrics).

## <a name="error-retrieving-data-message-on-dashboard"></a>Message « Erreur durant la récupération des données » sur le tableau de bord

Ce problème peut se produire quand votre tableau de bord a été créé avec une métrique qui a ensuite été dépréciéeet supprimée dans Azure. Pour vérifier si c’est le cas, ouvrez l’onglet **Métriques** de votre ressource et examinez les métriques disponibles dans le sélecteur de métriques. Si aucune métrique n’est affichée, cela signifie que la métrique a été supprimée d’Azure. En règle générale, quand une métrique est déconseillée, une nouvelle métrique, plus efficace, offre une perspective similaire sur l’intégrité des ressources.

**Solution :** Mettez à jour la vignette défaillante en choisissant une autre métrique pour votre graphique sur le tableau de bord. Vous pouvez [consulter une liste des métriques disponibles pour les services Azure](metrics-supported.md).

## <a name="chart-shows-dashed-line"></a>Le graphique affiche une ligne en pointillés.

Les graphiques de métrique Azure utilisent le style de ligne en pointillés pour indiquer une valeur manquante (ou « valeur Null ») entre deux points de données connus d’un fragment de temps. Par exemple, si vous avez choisi la granularité temporelle « 1 minute » dans le sélecteur d’heure, mais que la métrique a été collectée à 07 h 26, 07 h 27, 07 h 29 et 07 h 30 (notez la minute manquante entre les deuxième et troisième points de données), une ligne en pointillés reliera les points de données à 07 h 27 et 07 h 29, et une ligne pleine reliera tous les autres points de données. La ligne en pointillés descend à zéro lorsque la métrique utilise une agrégation **Nombre** et **Somme**. Pour les agrégations **Moy**, **Min** ou **Max**, la ligne en pointillés relie les deux points de données connus les plus proches. En outre, lorsque des données sont manquantes sur le côté le plus à droite ou le plus à gauche du graphique, la ligne en pointillés s’étend dans la direction du point de données manquant.
  ![Capture d’écran montrant comment, lorsque des données sont manquantes sur le côté le plus à droite ou le plus à gauche du graphique, la ligne en pointillés s’étend dans la direction du point de données manquant.](./media/metrics-troubleshoot/dashed-line.png)

**Solution :** Ce comportement est normal. Il est utile pour identifier les points de données manquants. Le graphique en courbes représente un excellent choix pour visualiser les tendances de métriques à haute densité. Cependant, il peut être difficile à interpréter si les métriques portent sur des valeurs éparses, en particulier lorsque la corrélation de valeurs avec le fragment de temps est importante. La ligne en pointillés facilite la lecture de ces graphiques. Cependant, si votre graphique n’est toujours pas clair, envisagez d’afficher vos métriques à l’aide d’un autre type de graphique. Par exemple, pour la même métrique, un graphique en nuages de points montre clairement chaque fragment de temps en affichant seulement un point quand il existe une valeur et en ignorant l’ensemble du point de données quand la valeur est manquante : ![Capture d’écran mettant en évidence l’option de menu Nuage de points.](./media/metrics-troubleshoot/scatter-plot.png)

   > [!NOTE]
   > Si vous préférez quand même un graphique en courbes pour votre métrique, vous pouvez déplacer la souris sur le graphique pour évaluer plus facilement la granularité temporelle. Le point de données à l’emplacement du pointeur de la souris sera mis en surbrillance.

## <a name="chart-shows-unexpected-drop-in-values"></a>Le graphique présente une baisse des valeurs inattendue.

Dans de nombreux cas, la baisse des valeurs de métrique perçue est due à une mauvaise compréhension des données affichées sur le graphique. Lorsque le graphique affiche les minutes les plus récentes, une baisse des sommes ou des nombres peut prêter à confusion. En effet, les derniers points de données de la métrique n’ont pas encore été reçus ou traités par Azure. Selon le service, la latence de traitement des métriques peut être de plusieurs minutes. Pour les graphiques affichant un intervalle de temps récent avec une précision de une ou cinq minutes, une baisse de la valeur sur les dernières minutes devient plus évidente : ![Capture d’écran montrant une chute de la valeur au cours des dernières minutes.](./media/metrics-troubleshoot/unexpected-dip.png)

**Solution :** Ce comportement est normal. Nous pensons qu’il est avantageux d’afficher les données dès que nous les recevons, même si elles sont *partielles* ou *incomplètes*. Vous pouvez ainsi tirer une conclusion importante plus rapidement et démarrer un examen immédiatement. Par exemple, pour une métrique qui affiche le nombre d’échecs, l’affichage d’une valeur partielle X vous indique qu’au moins X échecs se sont produits sur une minute donnée. Vous pouvez commencer à examiner le problème immédiatement au lieu d’attendre de voir le nombre exact d’échecs qui se sont produits sur cette minute, ce qui n’est peut-être pas si important. Le graphique sera mis à jour une fois que nous aurons reçu l’ensemble du jeu de données. Cependant, à ce moment, de nouveaux points de données correspondant aux minutes les plus récentes seront peut-être également incomplets.

## <a name="cannot-pick-guest-os-namespace-and-metrics"></a>Impossible de choisir l’espace de noms et les métriques de SE invité

Deux catégories de métriques sont disponibles pour les machines virtuelles et groupes de machines virtuelles identiques : Métriques d’**hôte de machine virtuelle** qui sont collectées par l’environnement d’hébergement Azure et métriques de **SE invité (classique)** qui sont collectées par l’[agent de surveillance](agents-overview.md) exécuté sur vos machines virtuelles. Vous installez l’agent de surveillance en activant [l’extension Azure Diagnostics](diagnostics-extension-overview.md).

Par défaut, les métriques de SE invité sont stockées dans le compte de Stockage Azure, que vous sélectionnez dans l’onglet **Paramètres de diagnostic** de votre ressource. Si les métriques de SE invité ne sont pas collectées ou si Metrics Explorer ne peut pas y accéder, vous verrez seulement l’espace de noms de la métrique **Ordinateur hôte de l’ordinateur virtuel** :

![image de métrique](./media/metrics-troubleshoot/vm.png)

**Solution :** Si vous ne voyez pas l’espace de noms et les métriques **SE invité (classique)** dans Metrics Explorer :

1. Vérifiez que [l’extension Azure Diagnostics](diagnostics-extension-overview.md) est activée et configurée pour collecter les métriques.
    > [!WARNING]
    > Vous ne pouvez pas utiliser [l’agent Log Analytics](agents-overview.md#log-analytics-agent) (également appelé Microsoft Monitoring Agent ou « MMA ») pour envoyer le **SE invité** dans un compte de stockage.

1. Assurez-vous que le fournisseur de ressources **Microsoft.Insights** est [inscrit pour votre abonnement](#microsoftinsights-resource-provider-isnt-registered-for-your-subscription).

1. Vérifiez que le compte de stockage n’est pas protégé par le pare-feu. Le Portail Azure a besoin d’accéder au compte de stockage afin de récupérer les données de métriques et de tracer les graphiques.

1. Utilisez [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour vérifier que les métriques sont transmises au compte de stockage. Si les métriques ne sont pas collectées, reportez-vous au [guide de résolution des problèmes liés à l’extension Diagnostics Azure ](diagnostics-extension-troubleshooting.md#metric-data-doesnt-appear-in-the-azure-portal).

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment bien démarrer avec Metrics Explorer](metrics-getting-started.md).
* [Découvrez les fonctionnalités avancées de Metrics Explorer](metrics-charts.md).
* [Consulter la liste des métriques disponibles pour les services Azure](metrics-supported.md)
* [Consulter des exemples de graphiques configurés](metric-chart-samples.md)
