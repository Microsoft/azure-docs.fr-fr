---
title: Créer des traces de performances côté client
description: Bonnes pratiques pour le profilage des performances côté client à l’aide de WPF
author: florianborn71
ms.author: flborn
ms.date: 12/11/2019
ms.topic: conceptual
ms.openlocfilehash: 1d4ce68bdda5fbc3dfdb7396141289a58dab5bd1
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204093"
---
# <a name="create-client-side-performance-traces"></a>Créer des traces de performances côté client

De nombreuses raisons peuvent expliquer pourquoi les performances d’Azure Remote Rendering ne sont pas aussi bonnes que vous le souhaitez. Au-delà des performances spécifiques au rendu sur le serveur cloud, la qualité de la connexion réseau, en particulier, a une influence significative sur l’expérience. Pour profiler les performances du serveur, reportez-vous au chapitre [Requêtes de performances côté serveur](../overview/features/performance-queries.md).

Ce chapitre porte sur l’identification des goulots d’étranglement côté client potentiels via *:::no-loc text="performance traces":::* .

## <a name="getting-started"></a>Prise en main

Si la fonctionnalité Windows :::no-loc text="performance tracing"::: est nouvelle pour vous, cette section aborde les termes et applications les plus importants pour démarrer.

### <a name="installation"></a>Installation

Les applications utilisées pour effectuer un suivi avec ARR sont des outils à usage général : elles peuvent être utilisées pour l’ensemble du développement sous Windows. Elles sont disponibles dans le [Windows performance Toolkit](/windows-hardware/test/wpt/). Pour y accéder, téléchargez le [kit de déploiement et d’évaluation Windows](/windows-hardware/get-started/adk-install).

### <a name="terminology"></a>Terminologie

Quand vous recherchez des informations sur les traces de performances, vous rencontrez naturellement une multitude de termes spécifiques. Les plus importants sont les suivants :

* `ETW`
* `ETL`
* `WPR`
* `WPA`

**ETW** signifie [**E**vent **T**racing for **W**indows](/windows/win32/etw/about-event-tracing). Il s’agit du suivi d’événements pour Windows. C’est un nom générique désignant la fonctionnalité de suivi au niveau du noyau, intégrée à Windows ; une fonctionnalité particulièrement efficace. Elle est appelée « suivi d’*événements* », car les applications qui prennent en charge ETW émettent des événements pour journaliser des actions qui peuvent aider à identifier les problèmes de performances. Par défaut, le système d’exploitation émet déjà des événements pour les accès au disque, les basculements de tâches, etc. Les applications comme ARR émettent également des événements personnalisés, par exemple concernant les images supprimées, la latence réseau, etc.

**ETL** signifie **E**vent **T**race **L**ogging. Il s’agit de la journalisation des traces d’événements. Ceci signifie simplement qu’une trace a été collectée (journalisée). Elle est alors généralement utilisée comme extension pour les fichiers qui stockent les données de suivi. Ainsi, quand vous enregistrez une trace, vous obtenez généralement un fichier \*.etl.

**WPR** signifie [**W**indows **P**erformance **R**ecorder](/windows-hardware/test/wpt/windows-performance-recorder). Il s’agit de l’Enregistreur de performance Windows, c’est-à-dire l’application qui démarre et arrête l’enregistrement des traces d’événements. Il utilise un fichier de profil (\*.wprp) qui configure les événements exacts à enregistrer. Ce type de fichier `wprp` est fourni avec le SDK ARR. Quand vous enregistrez des traces sur un PC de bureau, vous pouvez lancer directement WPR. Quand vous enregistrez une trace sur un appareil HoloLens, vous passez généralement par l’interface web.

**WPA** signifie [**W**indows **P**erformance **A**nalyzer](/windows-hardware/test/wpt/windows-performance-analyzer). C’est le nom de l’application offrant une interface graphique utilisateur pour ouvrir les fichiers \*.etl et examiner les données afin d’identifier les problèmes de performances. WPA vous permet de trier les données selon différents critères, de les afficher de différentes façons, d’en examiner les détails et de corréler les informations.

Si les traces ETL peuvent être créées sur n’importe quel appareil Windows (PC local, HoloLens, serveur cloud, etc.), elles sont généralement enregistrées sur le disque et analysées avec WPA sur un PC de bureau. Les fichiers ETL peuvent être envoyés à d’autres développeurs pour consultation. Notez cependant que certaines informations sensibles comme les chemins de fichiers et les adresses IP peuvent être capturées dans les traces ETL. Vous pouvez utiliser ETW de deux façons : pour enregistrer les traces ou pour les analyser. L’enregistrement des traces est simple et nécessite une configuration minimale. En revanche, l’analyse des traces nécessite une bonne compréhension de l’outil WPA et du problème que vous examinez. Nous fournissons plus bas des références vers une documentation générale pour l’apprentissage de WPA et des instructions sur l’interprétation des traces spécifiques à ARR.

## <a name="recording-a-trace-on-a-local-pc"></a>Enregistrement d’une trace sur un PC local

Pour identifier les problèmes de performances d’ARR, il est préférable d’enregistrer une trace directement sur un appareil HoloLens. C’est en effet la seule façon d’obtenir un instantané des véritables caractéristiques de performances. Toutefois, si vous souhaitez enregistrer spécifiquement une trace sans restrictions relatives aux performances HoloLens ou si vous souhaitez simplement apprendre à utiliser WPA et que vous n’avez pas besoin d’une trace réelle, effectuez les étapes suivantes.

### <a name="wpr-configuration"></a>Configuration de WPR

1. Lancez le [:::no-loc text="Windows Performance Recorder":::](/windows-hardware/test/wpt/windows-performance-recorder) à partir du *menu Démarrer*.
1. Développez **Plus d’options**.
1. Cliquez sur **Ajouter des profils...** .
1. Sélectionnez le fichier *AzureRemoteRenderingNetworkProfiling.wprp*, qui se trouve dans le SDK ARR, sous *Tools/ETLProfiles*.
   Le profil est à présent listé dans WPR sous *Mesures personnalisées*. Assurez-vous qu’il s’agit du seul profil activé.
1. Développez *Triage de premier niveau* :
    * Si vous voulez uniquement capturer une trace rapide des événements de réseau d’ARR, **désactivez** cette option.
    * Si vous devez mettre en corrélation des événements de réseau d’ARR avec d’autres caractéristiques du système (utilisation du processeur ou de la mémoire, par exemple), **activez** cette option.
    * Si vous activez cette option, la taille de la trace atteindra probablement plusieurs gigaoctets. Son enregistrement et son ouverture dans WPA seront alors particulièrement longs.

Votre configuration de WPR doit désormais ressembler à ceci :

![Configuration de WPR](./media/wpr.png)

### <a name="recording"></a>Enregistrement

Cliquez sur **Démarrer** pour démarrer l’enregistrement d’une trace. Vous pouvez démarrer et arrêter l’enregistrement à tout moment. Vous n’avez pas besoin de fermer votre application préalablement. Comme vous pouvez le voir, il est inutile de spécifier l’application à suivre, car ETW enregistre toujours une trace pour l’ensemble du système. Le fichier `wprp` spécifie les types d’événements à enregistrer.

Cliquez sur **Enregistrer** pour arrêter l’enregistrement et spécifiez l’emplacement de stockage du fichier ETL.

Vous disposez maintenant d’un fichier ETL que vous pouvez ouvrir directement dans WPA ou envoyer à quelqu’un d’autre.

## <a name="recording-a-trace-on-a-hololens"></a>Enregistrement d’une trace sur un appareil HoloLens

Pour enregistrer une trace sur un appareil HoloLens, démarrez votre appareil et entrez son adresse IP dans un navigateur pour ouvrir le *portail d’appareil*.

![Portail d’appareil](./media/wpr-hl.png)

1. Sur la gauche, accédez à *Performance > Suivi des performances*.
1. Sélectionnez **Profils personnalisés**.
1. Cliquez sur **:::no-loc text="Browse...":::**
1. Sélectionnez le fichier *AzureRemoteRenderingNetworkProfiling.wprp*, qui se trouve dans le SDK ARR, sous *Tools/ETLProfiles*.
1. Cliquez sur **Lancer le suivi**.
1. L’appareil HoloLens enregistre à présent une trace. Veillez à déclencher les problèmes de performances que vous souhaitez examiner. Cliquez ensuite sur **Arrêter le suivi**.
1. La trace est listée en bas de la page web. Cliquez sur l’icône de disque sur le côté droit pour télécharger le fichier ETL.

Vous disposez maintenant d’un fichier ETL que vous pouvez ouvrir directement dans WPA ou envoyer à quelqu’un d’autre.

## <a name="analyzing-traces-with-wpa"></a>Analyse des traces avec WPA

### <a name="wpa-basics"></a>Concepts de base de WPA

Windows Performance Analyzer est l’outil standard permettant d’ouvrir des fichiers ETL et d’inspecter les traces. Le fonctionnement de WPA n’est pas expliqué dans cet article. Pour commencer, consultez les ressources suivantes :

* Regardez les [vidéos d’introduction](/windows-hardware/test/wpt/windows-performance-analyzer) pour une première vue d’ensemble.
* WPA possède lui-même un onglet *Getting Started*, qui décrit les étapes courantes. Consultez les rubriques disponibles. La rubrique « View Data », en particulier, offre une brève présentation de la création de graphes pour des données spécifiques.
* Ce [site web](https://randomascii.wordpress.com/2015/09/24/etw-central/) fournit également d’excellentes informations, qui ne sont cependant pas toutes destinées aux débutants.

### <a name="graphing-data"></a>Présentation des données sous forme de graphe

Nous vous recommandons de prendre connaissance des points suivants pour commencer à utiliser le suivi ARR.

![Graphe de performance](./media/wpa-graph.png)

L’image ci-dessus montre un tableau de données de suivi et une représentation de ces données sous forme de graphe.

Dans le tableau du bas, vous remarquez une barre jaune (or) et une barre bleue. Vous pouvez faire glisser ces barres et les déplacer comme vous le souhaitez.

Toutes les **colonnes à gauche de la barre jaune** sont interprétées comme **clés**. Les clés sont utilisées pour structurer l’arborescence dans la fenêtre située en haut à gauche. Ici, nous avons deux colonnes *clés* : « Provider Name » et « Task Name ». L’arborescence dans la fenêtre en haut à gauche comporte donc deux niveaux. Si vous réorganisez les colonnes ou ajoutez ou supprimez des colonnes dans la zone des clés, la structure de l’arborescence change.

Les **colonnes à droite de la barre bleue** sont utilisées pour l’**affichage du graphe** dans la fenêtre située en haut à droite. Généralement, seule la première colonne est utilisée. Cependant, certains modes de graphe nécessitent plusieurs colonnes de données. Pour pouvoir utiliser des graphes linéaires, vous devez définir le *mode d’agrégation* sur cette colonne. Utilisez « Avg » ou « Max ». Le mode d’agrégation permet de déterminer la valeur du graphe pour un pixel donné, quand un pixel couvre une plage incluant plusieurs événements. Vous pouvez l’observer en définissant l’agrégation sur « Sum », puis en faisant un zoom avant et arrière.

Les colonnes du milieu n’ont pas de signification particulière.

![Vue des événements](./media/wpa-event-view.png)

Dans la fenêtre *Generic Events View Editor*, vous pouvez configurer toutes les colonnes à afficher, le mode d’agrégation, le tri et les colonnes utilisées comme clés ou pour les graphes. Dans l’exemple ci-dessus, le champ **Field 2** est activé et les champs Field 3 à 6 sont désactivés. Le champ Field 2 est généralement le premier champ de *données personnalisées* d’un événement ETW, donc pour les événements « FrameStatistics » d’ARR, qui représentent une valeur de latence du réseau. Activez d’autres colonnes « Field » pour voir d’autres valeurs relatives à cet événement.

### <a name="presets"></a>Présélections

Pour analyser correctement une trace, vous devez déterminer votre propre workflow ainsi que l’affichage de données que vous privilégiez. Toutefois, pour pouvoir obtenir rapidement une vue d’ensemble des événements spécifiques à ARR, nous incluons un profil de la Plateforme de protection de licence logicielle Windows et des fichiers prédéfinis dans le dossier *Tools/ETLProfiles*. Pour charger un profil complet, sélectionnez *Profiles > Apply...* dans la barre de menus de WPA, ou ouvrez le panneau *My Presets* (*Window > My Presets*) et sélectionnez *Import*. La première méthode établit une configuration complète de WPA comme illustré dans l’image ci-dessous. La seconde effectue uniquement des présélections pour les différentes configurations de vues et vous permet d’ouvrir rapidement une vue pour examiner une donnée d’événement ARR spécifique.

![Présélections](./media/wpa-arr-trace.png)

L’image ci-dessus montre des vues de différents événements spécifiques à ARR ainsi qu’une vue de l’utilisation globale du processeur.

## <a name="next-steps"></a>Étapes suivantes

* [Requêtes de performances côté serveur](../overview/features/performance-queries.md)