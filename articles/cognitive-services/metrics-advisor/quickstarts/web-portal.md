---
title: 'Démarrage rapide : Portail web de Metrics Advisor'
titleSuffix: Azure Cognitive Services
description: Découvrez comment commencer à utiliser le portail web de Metrics Advisor.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: quickstart
ms.date: 09/30/2020
ms.author: mbullwin
ms.openlocfilehash: 5dbf98d363429e6d22a0b7719cdc669deebd21a0
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348634"
---
# <a name="quickstart-monitor-your-first-metric-using-the-web-portal"></a>Démarrage rapide : Surveiller votre première métrique à l’aide du portail web

Lorsque vous provisionnez une instance Metrics Advisor, vous pouvez utiliser les API et l’espace de travail web pour travailler avec ce service. L’espace de travail web peut être utilisé comme un moyen simple de rapidement prendre en main le service. Il fournit également un moyen visuel de configurer les paramètres, de personnaliser votre modèle et d’effectuer une analyse de la cause racine. 

* Intégrer vos données de métrique
* Afficher vos métriques et visualisations
* Affiner les configurations de détection
* Explorer les insights de diagnostic
* Créer des alertes d’anomalie et s’y abonner

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Une fois que vous disposez de votre abonnement Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="Créer une ressource Metrics Advisor"  target="_blank">créez une ressource Metrics Advisor<span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour déployer votre instance Metrics Advisor.  

    
> [!TIP]
> * 10 à 30 minutes peuvent être nécessaires pour le déploiement de votre ressource Metrics Advisor. Cliquez sur **Accéder à la ressource** une fois le déploiement réussi.
> * Si vous souhaitez utiliser l’API REST pour interagir avec le service, vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez. Vous pouvez les trouver sous l’onglet **Clés et points de terminaison**, dans la ressource créée.

Ce document utilise une base de données SQL comme exemple pour la création de votre premier moniteur.

## <a name="sign-in-to-your-workspace"></a>Vous connecter à votre espace de travail

Après avoir créé votre ressource, connectez-vous au [portail Metrics Advisor](https://go.microsoft.com/fwlink/?linkid=2143774). Sélectionnez votre espace de travail pour commencer à superviser vos métriques. 
 
Vous pouvez créer une ressource Metrics Advisor à chaque région disponible. Vous pouvez changer d’espace de travail dans le portail Metrics Advisor à tout moment.


## <a name="onboard-time-series-data"></a>Intégrer des données de série chronologique

Metrics Advisor fournit des connecteurs pour différentes sources de données comme SQL Database, Azure Data Explorer ou le stockage Table Azure. Les étapes de connexion des données sont similaires pour les différents connecteurs, même si certains paramètres de configuration peuvent varier. Consultez [Connecter vos données à partir de différentes sources](../data-feeds-from-different-sources.md) pour connaître les paramètres requis pour des sources de données spécifiques.

Ce guide de démarrage rapide utilise une base de données SQL comme exemple. Vous pouvez également ingérer vos propres données en suivant la même procédure.

Pour bien démarrer, connectez-vous à votre espace de travail Metrics Advisor à l’aide de votre compte Active Directory. Dans la page d’accueil, sélectionnez votre **annuaire**, votre **abonnement** et l’**espace de travail** que vous venez de créer, puis cliquez sur **Prise en main**. Après le chargement de la page principale de la charge de travail, sélectionnez **Ajouter un flux de données** dans le menu de gauche.

### <a name="data-schema-requirements-and-configuration"></a>Configuration requise et configuration des schémas de données

[!INCLUDE [data schema requirements](../includes/data-schema-requirements.md)]

### <a name="configure-connection-settings"></a>Configurer les paramètres de connexion

> [!TIP]
> Pour plus d’informations sur les paramètres disponibles, consultez [Comment ajouter des flux de données](../how-tos/onboard-your-data.md).

Ajoutez le flux de données en vous connectant à votre source de données de série chronologique. Commencez par sélectionner les paramètres suivants :

* **Type de source** : Type de source de données dans laquelle vos données de série chronologique sont stockées.
* **Fréquence** : Intervalle entre des points de données consécutifs dans vos données de série chronologique. Par exemple, Annuel, Mensuel, Quotidien. L’intervalle le plus bas pris en charge par la personnalisation est de 60 secondes.
* **Ingérer des données depuis (UTC)**  : Heure de début du premier horodatage à ingérer. 


Ensuite, spécifiez la **chaîne de connexion** avec les informations d’identification de votre source de données, ainsi qu’une **requête** personnalisée. La requête est utilisée pour spécifier les données à ingérer et à convertir dans le schéma requis.

[!INCLUDE [query requirements](../includes/query-requirements.md)]

:::image type="content" source="../media/connection-settings.png" alt-text="Paramètres de connexion" lightbox="../media/connection-settings.png":::


### <a name="verify-the-connection-and-load-the-data-schema"></a>Vérifier la connexion et charger le schéma de données

Une fois la chaîne de connexion et la chaîne de requête créées, sélectionnez **Vérifier et obtenir le schéma** pour vérifier la connexion, et exécutez la requête pour obtenir votre schéma de données à partir de la source de données. En règle générale, cette opération prend quelques secondes selon votre connexion à la source de données. En cas d’erreur à cette étape, vérifiez que :

1. Votre chaîne de connexion et votre requête sont correctes.
2. Votre instance Metrics Advisor est en mesure de se connecter à la source de données s’il existe des paramètres de pare-feu.

### <a name="schema-configuration"></a>Configuration du schéma

Une fois le schéma de données chargé et affiché comme ci-dessous, sélectionnez les champs appropriés.


|Sélection  |Description  |Notes  |
|---------|---------|---------|
|**Timestamp**     | Horodatage d’un point de données. S’il est omis, Metrics Advisor utilise à la place l’horodatage de l’ingestion du point de données. Pour chaque flux de données, vous pouvez spécifier au plus une colonne comme horodatage.        | Optionnel. Doit être spécifié avec au plus une colonne.       |
|**Mesure**     |  Valeurs numériques dans le flux de données. Pour chaque flux de données, vous pouvez spécifier plusieurs mesures, mais au moins une colonne doit être sélectionnée comme mesure.        | Doit être spécifiée avec au moins une colonne.        |
|**Dimension**     | Valeurs catégorielles. Une combinaison de différentes valeurs identifie une série chronologique à une seule dimension, par exemple : pays, langue, locataire. Vous pouvez sélectionner un nombre nul ou arbitraire de colonnes comme dimensions. Remarque : Si vous sélectionnez une colonne qui n’est pas une chaîne comme dimension, soyez vigilant avec l’explosion de dimension. | Optionnel.        |
|**Ignorer**     | Ignorez la colonne sélectionnée.        |         |


:::image type="content" source="../media/schema-configuration.png" alt-text="Configuration du schéma" lightbox="../media/schema-configuration.png":::

### <a name="automatic-roll-up-settings"></a>Paramètres de regroupement automatique

> [!IMPORTANT]
> Si vous souhaitez activer l’**analyse de la cause racine** et d’autres fonctionnalités de diagnostic, les paramètres de regroupement automatique doivent être configurés. Une fois activé, les paramètres de regroupement automatique ne peuvent pas être modifiés.

Metrics Advisor peut effectuer automatiquement une agrégation (SUM/MAX/MIN...) sur chaque dimension lors de l’ingestion, puis crée une hiérarchie qui sera utilisée dans l’analyse de la cause racine et d’autres fonctionnalités de diagnostic. Pour plus d’informations, consultez [Paramètres de regroupement automatique](../how-tos/onboard-your-data.md#automatic-roll-up-settings).

Donnez un nom personnalisé au flux de données, qui s’affichera dans votre espace de travail. Cliquez sur **Envoyer**. 

## <a name="tune-detection-configuration"></a>Ajuster la configuration de détection

Une fois le flux de données ajouté, Metrics Advisor tente d’ingérer les données de métrique à partir de la date de début spécifiée. Il faut un certain temps pour ingérer complètement les données et vous pouvez afficher l’état d’ingestion en cliquant sur **Progression de l’ingestion** en haut de la page de flux de données. Si les données sont ingérées, Metrics Advisor applique la détection et continue à surveiller la source pour détecter de nouvelles données.

Lorsque la détection est appliquée, cliquez sur l’une des métriques listées dans le flux de données pour rechercher la **page de détails des métriques** pour : 
- Afficher les visualisations de toutes les tranches de série chronologique sous cette métrique
- Mettre à jour la configuration de la détection afin de répondre aux résultats attendus
- Configurer une notification pour les anomalies détectées

:::image type="content" source="../media/metric-details.png" alt-text="Détails de la métrique" lightbox="../media/metric-details.png":::

## <a name="view-the-diagnostic-insights"></a>Afficher les insights de diagnostic

Après l’ajustement de la configuration de la détection, les anomalies détectées doivent refléter les anomalies réelles dans vos données. Metrics Advisor effectue une analyse sur les métriques multidimensionnelles, telles que le clustering d’anomalies, la corrélation des incidents et l’analyse de la cause racine. Utilisez ces fonctionnalités pour analyser et diagnostiquer les incidents dans vos données.

Pour afficher les insights de diagnostic, cliquez sur les points rouges dans les visualisations de série chronologique, qui représentent les anomalies détectées. Une fenêtre s’affiche avec un lien vers la page d’analyse des incidents. 

:::image type="content" source="../media/incident-link.png" alt-text="Lien d’incident" lightbox="../media/incident-link.png":::

Après avoir cliqué sur le lien, vous serez orienté vers la page d’analyse des incidents qui se rapporte à l’anomalie correspondante, avec toute une palette d’insights de diagnostic. Dans la partie supérieure, des statistiques sont données sur l’incident, telles que la **gravité**, les **anomalies impliquées**, ainsi que l’**heure de début** et l’**heure de fin** affectées. 

Ensuite, vous verrez l’anomalie ancêtre de l’incident et les conseils de cause racine automatisés. Ces conseils de cause racine automatisés sont générés en analysant l’arborescence d’incidents de toutes les anomalies associées, y compris l’écart, la distribution et la contribution aux anomalies parentes. 

:::image type="content" source="../media/incident-diagnostic.png" alt-text="Diagnostics d’incident" lightbox="../media/incident-diagnostic.png":::

En fonction de ces informations, vous pouvez déjà obtenir une vue directe de ce qui se passe et de l’impact de l’incident, ainsi que de la cause racine la plus probable. De cette façon, il est possible de prendre une mesure immédiate pour résoudre l’incident dès que possible. 

Toutefois, vous pouvez également faire pivoter d’autres informations de diagnostic en tirant parti de fonctionnalités supplémentaires pour descendre dans la hiérarchie des anomalies par dimension, afficher des anomalies similaires et comparer les métriques. Pour plus d’informations, consultez [Comment : diagnostiquer un incident](../how-tos/diagnose-incident.md). 

## <a name="get-notified-when-new-anomalies-are-found"></a>Recevoir une notification quand de nouvelles anomalies sont détectées

Si vous souhaitez être alerté quand une anomalie est détectée dans vos données, vous pouvez créer un abonnement pour une ou plusieurs de vos métriques. Metrics Advisor utilise des hooks pour envoyer des alertes. Trois types de hooks sont pris en charge : les hooks d’e-mail, les webhooks et Azure DevOps. Nous allons utiliser un webhook à titre d’exemple. 

### <a name="create-a-web-hook"></a>Créer un webhook

Un webhook est le point d’entrée permettant de faire remarquer une anomalie par programmation à partir du service Metrics Advisor, qui appelle une API fournie par l’utilisateur lorsqu’une alerte est déclenchée. Pour plus d’informations sur la façon de créer un hook, reportez-vous à la section **Créer un hook** dans [Comment : Configurer des alertes et obtenir des notifications à l’aide d’un hook](../how-tos/alerts.md#create-a-hook). 

### <a name="configure-alert-settings"></a>Configuration des paramètres d'alerte

Après la création d’un hook, un paramètre d’alerte détermine quelles notifications envoyer et de quelle manière. Vous pouvez définir plusieurs paramètres d’alerte pour chaque métrique. Deux paramètres importants sont **Alerte pour** qui spécifie les anomalies à inclure, et **Filtrer les options d’anomalie** qui définit les anomalies à inclure dans l’alerte. Consultez la section **Ajouter ou modifier des paramètres d’alerte** dans [Comment : Configurer des alertes et obtenir des notifications à l’aide d’un hook](../how-tos/alerts.md#add-or-edit-alert-settings) pour plus d’informations.


## <a name="next-steps"></a>Étapes suivantes

- [Intégrer vos flux de données](../how-tos/onboard-your-data.md)
    - [Gérer les flux de données](../how-tos/manage-data-feeds.md)
    - [Configurations pour différentes sources de données](../data-feeds-from-different-sources.md)
- [Utiliser l’API REST ou les bibliothèques clientes](./rest-api-and-client-library.md)
- [Configurer des métriques et affiner la configuration de la détection](../how-tos/configure-metrics.md)