---
title: Création d’alertes avec des seuils dynamiques dans Azure Monitor
description: Créer des alertes avec des seuils dynamiques basés sur un apprentissage automatique
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 02/16/2020
ms.openlocfilehash: 110d4a3219b4898fa6f138e29f1112d7134f674c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012337"
---
# <a name="metric-alerts-with-dynamic-thresholds-in-azure-monitor"></a>Alertes de métrique avec seuils dynamiques dans Azure Monitor

L’alerte de métrique avec détection des seuils dynamiques s’appuie sur le machine learning avancé pour apprendre le comportement historique des métriques et identifier des modèles et des anomalies indiquant d’éventuels problèmes de service. Elle propose de prendre en charge à la fois une interface utilisateur simple et des opérations à grande échelle en permettant aux utilisateurs de configurer des règles d’alerte par le biais de l’API Azure Resource Manager, de manière totalement automatisée.

Une fois qu’une règle d’alerte est créée, elle se déclenche uniquement quand la métrique supervisée ne se comporte pas comme prévu, en fonction de ses seuils personnalisés.

Nous aimerions beaucoup avoir votre avis, envoyez vos commentaires à <azurealertsfeedback@microsoft.com>.

## <a name="why-and-when-is-using-dynamic-condition-type-recommended"></a>Quand et pourquoi l’utilisation d’un type de condition dynamique est-elle recommandée ?

1. **Génération d’alertes évolutive** : les règles d’alerte de seuil dynamique peuvent créer des seuils adaptés à des centaines de séries de métriques à la fois, tout en offrant quand même la même facilité de définition d’une règle d’alerte sur une seule métrique. Elles vous donnent moins d’alertes à créer et à gérer. Vous pouvez utiliser le portail Azure ou l’API Azure Resource Manager pour les créer. L’approche évolutive s’avère particulièrement utile lorsque vous traitez des dimensions de métrique ou plusieurs ressources, comme toutes les ressources d’abonnement.  [Découvrez comment configurer des alertes de métrique avec des seuils dynamiques à l’aide de modèles](alerts-metric-create-templates.md).

1. **Reconnaissance intelligente des modèles de métriques** : avec notre technologie de machine learning, nous sommes en mesure de détecter automatiquement des modèles de métriques pour nous adapter à leur évolution au fil du temps, ce qui peut souvent inclure leur saisonnalité (toutes les heures/tous les jours/toutes les semaines). L’adaptation au comportement des métriques au fil du temps et les alertes émises selon les écarts par rapport à un modèle permettent de ne pas avoir à connaître le « bon » seuil de chaque métrique. L’algorithme ML utilisé dans les seuils dynamiques est conçu pour empêcher les seuils bruyants (faible précision) ou larges (faible rappel) qui n’ont pas de modèle attendu.

1. **Configuration intuitive** : les seuils dynamiques permettent de configurer des alertes de métrique avec des concepts généraux, ce qui limite le besoin de connaissances spécifiques approfondies sur la métrique.

## <a name="how-to-configure-alerts-rules-with-dynamic-thresholds"></a>Comment configurer des règles d’alerte avec des seuils dynamiques ?

Vous pouvez configurer des alertes avec des seuils dynamiques par le biais des alertes de métrique dans Azure Monitor. [Découvrez comment configurer des alertes de métrique](alerts-metric.md).

## <a name="how-are-the-thresholds-calculated"></a>Comment les seuils sont-ils calculés ?

Les seuils dynamiques apprennent sans discontinuer les données de la série de métriques pour essayer de les modéliser à l’aide d’un ensemble d’algorithmes et de méthodes. Il détecte des modèles dans les données comme une saisonnalité (toutes les heures/tous les jours/toutes les semaines) et est capable de gérer des métriques bruyantes (comme le processeur ou la mémoire de l’ordinateur), ainsi que des métriques à faible dispersion (comme la disponibilité et le taux d’erreur).

Les seuils sont sélectionnés de manière à ce qu’un écart par rapport à ces seuils indique une anomalie dans le comportement de la métrique.

> [!NOTE]
> La détection des modèles saisonniers est définie sur un intervalle d'une heure, d'un jour ou d'une semaine. Cela signifie que d'autres modèles, bihoraires ou bihebdomadaires par exemple, peuvent ne pas être détectés.

## <a name="what-does-sensitivity-setting-in-dynamic-thresholds-mean"></a>Que signifie le paramètre « Sensibilité » dans les seuils dynamiques ?

La sensibilité d’un seuil d’alerte est un concept général qui contrôle l’ampleur de l’écart par rapport à au comportement de la métrique nécessaire pour déclencher une alerte.
Cette option ne nécessite pas de connaissances spécifiques sur la métrique comme le seuil statique. Voici les options disponibles :

- Élevée : Les seuils sont stricts et proches du modèle de la série de métriques. Une règle d’alerte est déclenchée dès le plus petit écart, ce qui génère un nombre plus important d’alertes.
- Moyenne : Les seuils sont moins stricts et plus équilibrés, les alertes moins nombreuses qu’avec une sensibilité élevée (par défaut).
- Faible : Les seuils sont lâches et plus distants du modèle de la série de métriques. Une règle d’alerte est uniquement déclenchée lors de grands écarts, ce qui génère un nombre moins important d’alertes.

## <a name="what-are-the-operator-setting-options-in-dynamic-thresholds"></a>Quelles sont les options du paramètre « Opérateur » dans les seuils dynamiques ?

La règle d’alerte des seuils dynamiques permet de créer des seuils adaptés au comportement de la métrique dans ses limites supérieures et inférieures avec la même règle d’alerte.
Vous pouvez choisir l’alerte à déclencher dans l’une des trois conditions suivantes :

- Supérieur au seuil supérieur ou inférieur au seuil inférieur (par défaut)
- Supérieur au seuil supérieur
- Inférieur au seuil inférieur

## <a name="what-do-the-advanced-settings-in-dynamic-thresholds-mean"></a>Que signifient les paramètres avancés dans les seuils dynamiques ?

**Périodes d’échec** : les seuils dynamiques vous permettent également de configurer le « nombre de violations avant de déclencher l’alerte », qui correspond à un nombre minimal d’écarts nécessaire pendant une certaine fenêtre de temps pour que le système déclenche une alerte (la fenêtre de temps par défaut est de quatre écarts en 20 minutes). L’utilisateur peut configurer des périodes d’échec et choisir sur quoi être alerté en modifiant les périodes d’échec et la fenêtre de temps. Cette possibilité permet de réduire le nombre d’alertes générées par des pics passagers. Par exemple :

Pour déclencher une alerte quand le problème perdure pendant 20 minutes, 4 fois de suite par groupe de périodes de 5 minutes, utilisez les paramètres suivants :

![Paramètres des périodes d’échec pour un problème qui perdure pendant 20 minutes, 4 fois de suite par groupe de périodes de 5 minutes](media/alerts-dynamic-thresholds/0008.png)

Pour déclencher une alerte en cas de violation d’un seuil dynamique pendant 20 minutes sur les 30 dernières minutes avec un groupe de périodes de 5 minutes, utilisez les paramètres suivants :

![Paramètres des périodes d’échec pour un problème qui perdure pendant 20 minutes sur les 30 dernières minutes par groupe de périodes de 5 minutes](media/alerts-dynamic-thresholds/0009.png)

**Ignorer les données précédentes** : Les utilisateurs peuvent éventuellement définir aussi une date de début à partir de laquelle le système doit commencer à calculer les seuils. Un cas d’usage typique peut se produire quand une ressource était exécutée en mode de test et qu’elle est à présent promue pour répondre à une charge de travail de production. Ainsi, le comportement de toute métrique pendant la phase de test doit être ignoré.

## <a name="how-do-you-find-out-why-a-dynamic-thresholds-alert-was-triggered"></a>Comment connaître la raison pour laquelle une alerte de seuil dynamique a été déclenchée ?

Vous pouvez explorer les instances d’alertes déclenchées dans l’affichage des alertes en cliquant sur le lien de l’e-mail ou du SMS, ou dans un navigateur pour afficher les alertes dans le portail Azure. [Découvrez-en plus sur l'affichage des alertes](alerts-overview.md#alerts-experience).

L’affichage des alertes présente :

- Tous les détails relatifs aux métriques au moment où l'alerte de seuil dynamique a été déclenchée.
- Un graphique de la période au cours de laquelle l’alerte a été déclenchée, incluant les seuils dynamiques utilisés à ce moment précis.
- La possibilité de fournir des commentaires sur l’alerte de seuil dynamique et l'expérience d'affichage des alertes afin d'améliorer les futures détections.

## <a name="will-slow-behavior-changes-in-the-metric-trigger-an-alert"></a>Une modification de comportement lente dans la métrique déclenche-t-elle une alerte ?

Probablement pas. Les seuils dynamiques excellent à détecter des écarts significatifs plutôt que des problèmes à évolution lente.

## <a name="how-much-data-is-used-to-preview-and-then-calculate-thresholds"></a>Quelle quantité de données est utilisée pour prévoir, puis calculer les seuils ?

Quand une règle d’alerte est créée pour la première fois, les seuils qui apparaissent dans le graphique reposent sur un nombre suffisant de données d’historique pour calculer des modèles saisonniers horaires ou journaliers (10 jours). Une fois la règle d’alerte créée, les seuils dynamiques utilisent toutes les données d’historique nécessaires disponibles et s’adaptent aux nouvelles données afin de proposer des seuils plus précis. Dès lors, après ce calcul, le graphique affiche aussi des modèles hebdomadaires.

## <a name="how-much-data-is-needed-to-trigger-an-alert"></a>Quelle est la quantité de données requises pour déclencher une alerte ?

En présence d’une nouvelle ressource ou si des données métriques sont manquantes, les seuils dynamiques ne déclenchent pas d’alertes avant trois jours ou avant qu’au moins 30 échantillons de données métriques soient disponibles et ce, afin de proposer des seuils précis.
Pour les ressources existantes avec des données métriques suffisantes, les seuils dynamiques peuvent déclencher immédiatement des alertes.

## <a name="dynamic-thresholds-best-practices"></a>Bonnes pratiques concernant les seuils dynamiques

Les seuils dynamiques peuvent être appliqués à la plupart des plateformes ou métriques personnalisées dans Azure Monitor, tout comme ils convenaient pour les métriques d’infrastructure et d’application communes.
Voici une série de bonnes pratiques sur la façon de configurer des alertes sur certaines de ces métriques à l’aide de seuils dynamiques.

### <a name="dynamic-thresholds-on-virtual-machine-cpu-percentage-metrics"></a>Seuils dynamiques sur les métriques de pourcentage processeur de machine virtuelle

1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Moniteur**. L’affichage Moniteur consolide tous vos paramètres et données de supervision dans une même vue.

2. Cliquez sur **Alertes**, puis sur **+ Nouvelle règle d’alerte**.

    > [!TIP]
    > La plupart des panneaux de ressources incluent également l’option **Alertes** dans leur menu de ressources, sous **Supervision**. Vous pouvez créer des alertes à partir de là également.

3. Cliquez sur **Sélectionner une cible** dans le volet contextuel qui se charge, puis sélectionnez une ressource cible sur laquelle définir une alerte. Utilisez les listes déroulantes **Abonnement** et **Type de ressource « Machines virtuelles »** pour rechercher la ressource à superviser. Vous pouvez également utiliser la barre de recherche pour rechercher votre ressource.

4. Une fois que vous avez sélectionné une ressource cible, cliquez sur **Ajouter une condition**.

5. Sélectionnez **Pourcentage UC**.

6. Si vous le souhaitez, affinez la métrique en ajustant la **Période** et **l’Agrégation**. Nous vous déconseillons d’utiliser le type d’agrégation « Maximum » pour ce type de métrique, car il est moins représentatif du comportement. Pour le type d’agrégation « Maximum », le seuil statique est peut-être plus approprié.

7. Vous verrez un graphique de la métrique pendant les 6 dernières heures. Définissez les paramètres d’alerte :
    1. **Type de condition** : choisissez l’option « Dynamique ».
    1. **Sensibilité** : choisissez la sensibilité Moyenne ou Faible pour réduire le bruit des alertes.
    1. **Opérateur** : choisissez « Supérieur à », sauf si le comportement représente l’utilisation de l’application.
    1. **Fréquence** : envisagez de la réduire suivant l’impact commercial de l’alerte.
    1. **Périodes d’échec** (option avancée) : la fenêtre de vérification doit être d’au moins 15 minutes. Par exemple, si la période est définie sur cinq minutes, il doit y avoir au moins trois périodes d’échec.

8. Le graphique de métriques affiche les seuils calculés en fonction des données récentes.

9. Cliquez sur **Done**.

10. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).

11. Ajoutez un groupe d’actions à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.

12. Cliquez sur **Terminé** pour enregistrer la règle d’alerte de métrique.

> [!NOTE]
> Les règles d’alerte de métrique créées via le portail sont créées dans le même groupe de ressources que la ressource cible.

### <a name="dynamic-thresholds-on-application-insights-http-request-execution-time"></a>Seuils dynamiques sur la durée d’exécution de la requête HTTP Application Insights

1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Moniteur**. L’affichage Moniteur consolide tous vos paramètres et données de supervision dans une même vue.

2. Cliquez sur **Alertes**, puis sur **+ Nouvelle règle d’alerte**.

    > [!TIP]
    > La plupart des panneaux de ressources incluent également l’option **Alertes** dans leur menu de ressources, sous **Supervision**. Vous pouvez créer des alertes à partir de là également.

3. Cliquez sur **Sélectionner une cible** dans le volet contextuel qui se charge, puis sélectionnez une ressource cible sur laquelle définir une alerte. Utilisez les listes déroulantes **Abonnement** et **Type de ressource « Application Insights »** pour rechercher la ressource à superviser. Vous pouvez également utiliser la barre de recherche pour rechercher votre ressource.

4. Une fois que vous avez sélectionné une ressource cible, cliquez sur **Ajouter une condition**.

5. Sélectionnez la **Durée d’exécution de la requête HTTP**.

6. Si vous le souhaitez, affinez la métrique en ajustant la **Période** et **l’Agrégation**. Nous vous déconseillons d’utiliser le type d’agrégation « Maximum » pour ce type de métrique, car il est moins représentatif du comportement. Pour le type d’agrégation « Maximum », le seuil statique est peut-être plus approprié.

7. Vous verrez un graphique de la métrique pendant les 6 dernières heures. Définissez les paramètres d’alerte :
    1. **Type de condition** : choisissez l’option « Dynamique ».
    1. **Opérateur** : choisissez « Supérieur à » pour réduire les alertes déclenchées sur l’amélioration de la durée.
    1. **Fréquence** : envisagez de la réduire suivant l’impact commercial de l’alerte.

8. Le graphique de métriques affiche les seuils calculés en fonction des données récentes.

9. Cliquez sur **Done**.

10. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).

11. Ajoutez un groupe d’actions à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.

12. Cliquez sur **Terminé** pour enregistrer la règle d’alerte de métrique.

> [!NOTE]
> Les règles d’alerte de métrique créées via le portail sont créées dans le même groupe de ressources que la ressource cible.

## <a name="interpreting-dynamic-threshold-charts"></a>Interprétation des graphiques de seuil dynamique

Vous trouverez ci-dessous un graphique présentant une métrique, ses limites de seuil dynamique et certaines alertes déclenchées quand la valeur se situe en-deçà des seuils autorisés.

![Découvrir comment configurer des alertes de métrique](media/alerts-dynamic-thresholds/threshold-picture-8bit.png)

Utilisez les informations suivantes pour interpréter le graphique précédent.

- **Ligne bleue** : métrique réelle mesurée au fil du temps.
- **Zone colorée en bleu** : montre la plage autorisée pour la métrique. Tant que les valeurs de la métrique restent dans cette plage, aucune alerte n’est générée.
- **Points bleus** : si vous cliquez sur une partie du graphique, puis pointez sur la ligne bleue, un point bleu apparaît sous le curseur pour indiquer une valeur de métrique agrégée individuelle.
- **Fenêtre avec un point bleu** : montre la valeur de la métrique mesurée (le point bleu) et les valeurs supérieure et inférieure de la plage autorisée.  
- **Point rouge avec un cercle noir** : montre la première valeur de la métrique située en dehors de la plage autorisée. Il s’agit de la valeur qui déclenche une alerte de métrique et la met dans un état actif.
- **Points rouges** : indique d’autres valeurs mesurées en dehors de la plage autorisée. Ces valeurs ne déclenchent pas d’autres alertes de métrique, mais l’alerte reste active.
- **Zone rouge** : indique l’heure à laquelle la valeur de la métrique est sortie de la plage autorisée. L’alerte reste à l’état actif tant que les valeurs mesurées suivantes se trouvent en dehors de la plage autorisée, mais aucune nouvelle alerte n’est déclenchée.
- **Fin de la zone rouge** : lorsque la ligne bleue revient à l’intérieur des valeurs autorisées, la zone rouge s’arrête et la ligne des valeurs mesurées redevient bleue. L’état de l’alerte de métrique déclenchée au moment du point rouge avec le contour noir est défini sur résolu. 
