---
title: Diagnostics de transaction Azure Application Insights | Microsoft Docs
description: Diagnostics de transaction de bout en bout Application Insights
ms.topic: conceptual
ms.date: 01/19/2018
ms.reviewer: sdash
ms.openlocfilehash: 7623b7131e6344a67c468d0436884ebfef9b0058
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746095"
---
# <a name="unified-cross-component-transaction-diagnostics"></a>Diagnostics de transaction entre composants unifiés

Les diagnostics unifiés mettent automatiquement en corrélation la télémétrie côté serveur de tous vos composants surveillés Application Insights dans une vue unique. Le fait d’avoir plusieurs ressources avec des clés d’instrumentation distinctes ne pose pas de problème. Application Insights détecte la relation sous-jacente et permet de diagnostiquer facilement le composant d’application, la dépendance ou l’exception à l’origine du ralentissement ou de l’échec de la transaction.

## <a name="what-is-a-component"></a>Qu’est un composant ?

Les composants sont des parties pouvant être déployées de manière indépendante de votre application distribuée/de microservices. Les développeurs et équipes d’opérations disposent d’une visibilité au niveau du code ou d’un accès à la télémétrie générée par ces composants d’application.

* Les composants sont différents des dépendances externes « observées » telles que SQL, EventHub, etc., auxquelles votre équipe/organisation peut ne pas avoir accès (code ou télémétrie).
* Les composants s’exécutent sur un nombre quelconque d’instances de serveur/rôle/conteneur.
* Les composants peuvent être des clés d’instrumentation Application Insights distinctes (même si les abonnements sont différents) ou des rôles différents rapportant à une clé d’instrumentation Application Insights unique. La nouvelle expérience affiche des détails sur tous les composants, quelle que soit leur configuration.

> [!NOTE]
> * **Il vous manque les liens des éléments connexes ?** Toutes les données de télémétrie associées se trouvent dans les sections situées en [haut](#cross-component-transaction-chart) et en [bas](#all-telemetry-with-this-operation-id), du côté gauche. 

## <a name="transaction-diagnostics-experience"></a>Expériences de diagnostics de transaction
Cette vue compte quatre parties principales : une liste de résultats, un graphique de transaction entre composants, une liste de séquence horaire de toutes les données de télémétrie relatives à l’opération, et le volet d’informations de l’élément de télémétrie sélectionné sur la gauche.

![Parties principales](media/transaction-diagnostics/4partsCrossComponent.png)

## <a name="cross-component-transaction-chart"></a>Graphique des transactions entre composants

Ce graphique fournit une chronologie avec des barres horizontales pendant la durée des requêtes et des dépendances entre les composants. Les exceptions collectées sont également marquées sur la chronologie.

* La ligne supérieure de ce graphique représente le point d’entrée, la requête entrante vers le premier composant appelé dans cette transaction. La durée correspond à la durée d’exécution totale de la transaction.
* Les appels vers des dépendances externes sont représentés par des lignes non réductibles simples, avec des icônes représentant le type de dépendance.
* Les appels vers d’autres composants sont représentés par des lignes réductibles. Chaque ligne correspond à une opération spécifique appelée au niveau du composant.
* Par défaut, la requête, la dépendance ou l’exception que vous sélectionnez s’affiche sur la droite.
* Sélectionnez une ligne pour afficher ses [détails à droite](#details-of-the-selected-telemetry). 

> [!NOTE]
> Les appels vers d’autres composants comptent deux lignes : une ligne représente l’appel sortant (dépendance) à partir du composant appelant, et l’autre ligne correspond à la requête entrante au niveau du composant appelé. L’icône de début et le style distinct des barres de durée permettent de les différencier.

## <a name="all-telemetry-with-this-operation-id"></a>Toutes les données de télémétrie avec cet ID d’opération

Cette section montre la liste plate dans une séquence horaire de toutes les données de télémétrie associées à cette transaction. Elle montre également les événements personnalisés, ainsi que les traces qui ne sont pas affichées dans le graphique des transactions. Vous pouvez filtrer cette liste pour n’afficher que les données de télémétrie qui ont été générées par un composant ou un appel donné. Vous pouvez sélectionner un élément de télémétrie de cette liste pour afficher les [détails correspondants à droite](#details-of-the-selected-telemetry).

![Séquence horaire de l’ensemble de la télémétrie](media/transaction-diagnostics/allTelemetryDrawerOpened.png)

## <a name="details-of-the-selected-telemetry"></a>Détails de la télémétrie sélectionnée

Ce volet réductible affiche les détails de l’élément qui a été sélectionné dans le graphique de transactions ou dans la liste. « Afficher tout » répertorie tous les attributs standard qui sont collectés. Les attributs personnalisés sont répertoriés séparément sous le jeu standard. Cliquez sur « ... » sous la fenêtre de la trace de pile pour afficher l’option permettant de copier la trace. Les options « Ouvrir les traces Profiler » et « Ouvrir l’instantané de débogage » montrent les diagnostics au niveau du code dans les volets d’informations correspondants.

![Détail de l’exception](media/transaction-diagnostics/exceptiondetail.png)

## <a name="search-results"></a>Résultats de la recherche

Ce volet réductible affiche les autres résultats qui correspondent aux critères de filtrage. Cliquez sur un résultat pour mettre à jour les détails des trois sections ci-dessus. Nous avons tenté de trouver les exemples qui sont les plus susceptibles d’avoir des informations disponibles pour tous les composants, même si l’échantillonnage est appliqué dans l’un d’eux. Les exemples suivants tiennent lieu de suggestions.

![Résultats de la recherche](media/transaction-diagnostics/searchResults.png)

## <a name="profiler-and-snapshot-debugger"></a>Profileur et débogueur de capture instantanée

Le [profileur Application Insights](./profiler.md) ou le [débogueur de la capture instantanée](snapshot-debugger.md) apporte une aide avec des diagnostics au niveau du code des problèmes de performances et d’échec. Grâce à cette expérience, vous pouvez afficher les traces du profileur ou les instantanés d’un composant d’un simple clic.

Si Profiler ne fonctionne pas, contactez **serviceprofilerhelp\@microsoft.com**

Si le Débogueur de capture instantanée ne fonctionne pas, contactez **snapshothelp\@microsoft.com**

![Intégration de Profiler](media/transaction-diagnostics/profilerTraces.png)

## <a name="faq"></a>Questions fréquentes (FAQ)

*Je vois un composant unique sur le graphique, et les autres n’apparaissent que comme des dépendances externes sans détails sur ce qu’il s’est passé dans ces composants.*

Raisons possibles :

* Les autres composants sont-ils instrumentés avec Application Insights ?
* Utilisent-ils le dernier Kit de développement logiciel (SDK) d’Application Insights stable ?
* Si ces composants sont des ressources Application Insights distinctes, avez-vous l’[accès](resources-roles-access-control.md) requis ? Si vous avez accès et que les composants sont instrumentés avec les derniers Kits de développement logiciel (SDK) Application Insights, faites-le nous savoir via le canal de commentaires en haut à droite.

*Je vois des lignes en double pour les dépendances. Est-ce normal ?*

À ce stade, nous présentons l’appel de dépendance sortant séparément de la requête entrante. En règle générale, les deux appels semblent identiques, seule la valeur de durée est différente en raison de l’aller-retour réseau. L’icône de début et le style distinct des barres de durée permettent de les différencier. Cette présentation des données porte-t-elle à confusion ? Faites-nous part de vos commentaires !

*Qu’en-est-il des variations d’horloges entre différentes instances de composant ?*

Les chronologies sont ajustées pour les variations d’horloges dans le graphique de transaction. Vous pouvez voir les timestamps exacts dans le volet d’informations ou à l’aide d’Analytics.

*Pourquoi la plupart des requêtes d’éléments associés est manquante dans la nouvelle expérience ?*

C'est la procédure normale. Tous les éléments associés, sur tous les composants, sont déjà disponibles sur le côté gauche (sections supérieure et inférieure). La nouvelle expérience comporte deux éléments associés non couverts par le côté gauche : l’ensemble de la télémétrie cinq minutes avant et après cet événement et la chronologie utilisateur.

