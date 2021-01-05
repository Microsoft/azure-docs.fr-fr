---
title: Gérer les flux de données dans Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Découvrez comment gérer les flux de données que vous avez ajoutés à Metrics Advisor.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: mbullwin
ms.openlocfilehash: fb6eaf44967732d3a41ea92b0896540a40f694e3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184720"
---
# <a name="how-to-manage-your-data-feeds"></a>Procédure : Gérer vos flux de données

Apprenez à gérer vos flux de données intégrés dans Metrics Advisor. Cet article explique comment gérer les flux de données dans Metrics Advisor.

## <a name="edit-a-data-feed"></a>Modifier un flux de données

> [!NOTE]
> Les détails suivants ne peuvent pas être modifiés après la création d’un flux de données. 
> * ID du flux de données
> * Heure de création
> * Dimension
> * Type de source
> * Granularité

Seul l’administrateur d’un flux de données est autorisé à y apporter des modifications. 

Pour suspendre ou réactiver un flux de données :

1. Dans la liste des flux de données, cliquez sur l’opération que vous souhaitez effectuer sur le flux de données.

2. Sur la page de détails des flux de données, cliquez sur le bouton **État**.

Pour supprimer un flux de données : 

1. Dans la liste des flux de données, cliquez sur **Supprimer** en regard du flux de données.

2. Sur la page de détails des flux de données, cliquez sur **Supprimer**.

Lorsque vous modifiez l’heure de début, vous devez vérifier de nouveau le schéma. Pour la modifier, utilisez l’option **Modifier les paramètres**.

##  <a name="backfill-your-data-feed"></a>Renvoyer votre flux de données

Sélectionnez le bouton **Renvoi** pour déclencher une ingestion immédiate sur un timestamp, résoudre un échec d’ingestion ou remplacer les données existantes.
- L’heure de début est inclusive.
- L’heure de fin est exclusive.
- La détection d’anomalie est de nouveau déclenchée sur la plage sélectionnée uniquement.

:::image type="content" source="../media/datafeeds/backfill-datafeed.png" alt-text="Renvoi d’un flux de données":::

## <a name="manage-permission-of-a-data-feed"></a>Gérer l’autorisation d’un flux de données

L’accès à l’espace de travail est contrôlé par la ressource Metrics Advisor, qui utilise Azure Active Directory pour l’authentification. Une autre couche de contrôle des autorisations est appliquée aux données de métriques.

Metrics Advisor vous permet d’octroyer des autorisations à différents groupes de personnes sur différents flux de données. Il existe deux types de rôles : 

- Administrateur : Dispose d’autorisations totales pour gérer un flux de données, y compris des droits de modification et de suppression.
- Observateur : dispose d’un accès en lecture seule au flux de données.
 

## <a name="advanced-settings"></a>Paramètres avancés

Plusieurs paramètres avancés facultatifs sont disponibles lors de la création d’un flux de données. Vous pouvez les modifier dans la page de détails des flux de données.

### <a name="ingestion-options"></a>Options d’ingestion

* **Décalage du temps d’ingestion** : Par défaut, les données sont ingérées en fonction de la granularité spécifiée. Par exemple, une métrique qui utilise un timestamp *quotidien* est ingérée un jour après son timestamp. Vous pouvez appliquer le décalage en utilisant un nombre *positif* pour retarder le temps d’ingestion ou un nombre *négatif* pour l’avancer.

* **Max concurrency** (Concurrence max) : Définissez ce paramètre si votre source de données prend en charge une concurrence limité. Sinon, conservez le paramètre par défaut.

* **Stop retry after** (Arrêter les nouvelles tentatives après) : En cas d’échec de l’ingestion de données, une nouvelle tentative est automatiquement effectuée au cours d’une période donnée. Le début de la période correspond à l’heure d’occurrence de la première ingestion des données. La durée de cette période est définie en fonction de la granularité. Si vous conservez la valeur par défaut (-1), la valeur sera déterminée en fonction de la granularité, comme indiqué ci-dessous.
    
    | Granularité       | Arrêter les nouvelles tentatives après :           |
    | :------------ | :--------------- |
    | Tous les jours, Personnalisé (>= 1 jour), Toutes les semaines, Tous les mois, Tous les ans     | 7 jours          |
    | Toutes les heures, Personnalisé (< 1 jour)       | 72 heures |

* **Min retry interval** (Intervalle minimal avant nouvelle tentative) : Vous pouvez spécifier l’intervalle minimal lorsque vous tenter de nouveau d’extraire des données de la source. Si vous conservez la valeur par défaut (-1), l’intervalle entre deux nouvelles tentatives sera déterminé en fonction de la granularité, comme indiqué ci-dessous.
    
    | Granularité       | Intervalle minimal avant nouvelle tentative           |
    | :------------ | :--------------- |
    | Tous les jours, Personnalisé (>= 1 jour), Toutes les semaines, Tous les mois     | 30 minutes          |
    | Toutes les heures, Personnalisé (< 1 jour)      | 10 minutes |
    | Annuelle | 1 jour          |
 
### <a name="fill-gap-when-detecting"></a>Fill gap when detecting (Combler les écarts lors de la détection) : 

> [!NOTE]
> Ce paramètre n’affecte pas votre source de données et n’a aucun effet sur les graphiques de données affichés sur le portail. Le remplissage automatique intervient uniquement lors de la détection d’anomalie.

Certaines séries chronologiques ne sont pas continues. Pour plus de précision, lorsqu’il manque des points de données, Metrics Advisor utilise la valeur spécifiée pour les remplir avant d’effectuer une détection d’anomalie.
Les options sont : 

* Utilisation de la valeur dérivée du précédent point de données réel. Il s’agit de l’option par défaut.
* Utilisation d’une valeur spécifique.

### <a name="action-link-template"></a>Modèle de lien d’action : 

Les modèles de lien d’action permettent de prédéfinir des URL HTTP actionnables, qui se composent des espaces réservés `%datafeed`, `%metric`, `%timestamp`, `%detect_config`et `%tagset`. Vous pouvez utiliser le modèle pour rediriger une anomalie ou un incident vers une URL spécifique à des fins d’exploration au niveau du détail.

:::image type="content" source="../media/action-link-template.png" alt-text="Modèle de lien d’action" lightbox="../media/action-link-template.png":::

Une fois que vous avez renseigné le lien d’action, cliquez sur **Go to action link** (Accéder au lien d’action) sous l’option Action de la liste d’incidents et dans le menu contextuel de l’arborescence des incidents. Remplacez les espaces réservés du modèle de lien d’action par les valeurs correspondantes de l’anomalie ou de l’incident.

| Espace réservé | Exemples | Commentaire |
| ---------- | -------- | ------- |
| `%datafeed` | - | ID du flux de données |
| `%metric` | - | ID de la métrique |
| `%detect_config` | - | ID de configuration de la détection |
| `%timestamp` | - | Timestamp d’une anomalie ou heure de fin d’un incident persistant |
| `%tagset` | `%tagset`, <br> `[%tagset.get("Dim1")]`, <br> `[ %tagset.get("Dim1", "filterVal")]` | Valeurs dimensionnelles d’une anomalie ou d’une anomalie majeure d’un incident.   <br> `filterVal` est utilisé pour filtrer les valeurs correspondantes entre crochets.   |

Exemples :

* Si le modèle de lien d’action est `https://action-link/metric/%metric?detectConfigId=%detect_config`,
  * Le lien d’action `https://action-link/metric/1234?detectConfigId=2345` accède aux anomalies ou aux incidents associés à la métrique `1234` et à la configuration de détection `2345`.

* Si le modèle de lien d’action est `https://action-link?[Dim1=%tagset.get('Dim1','')&][Dim2=%tagset.get('Dim2','')]`, 
    * Le lien d’action serait `https://action-link?Dim1=Val1&Dim2=Val2` lorsque l’anomalie est `{ "Dim1": "Val1", "Dim2": "Val2" }`. 
    * Le lien d’action serait `https://action-link?Dim2=Val2` lorsque l’anomalie est `{ "Dim1": "", "Dim2": "Val2" } `, car `[Dim1=***&]` est ignoré pour la chaîne vide de la valeur dimensionnelle. 

* Si le modèle de lien d’action est `https://action-link?filter=[Name/Dim1 eq '%tagset.get('Dim1','')' and ][Name/Dim2 eq '%tagset.get('Dim2','')']`, 
    * Le lien d’action serait `https://action-link?filter=Name/Dim1 eq 'Val1' and Name/Dim2 eq 'Val2'` lorsque l’anomalie est `{ "Dim1": "Val1", "Dim2": "Val2" }`. 
    * Le lien d’action serait `https://action-link?filter=Name/Dim2 eq 'Val2'` lorsque l’anomalie est `{ "Dim1": "", "Dim2": "Val2" }`, car `[Name/Dim1 eq '***' and ]` est ignoré pour la chaîne vide de la valeur dimensionnelle. 
   
### <a name="data-feed-not-available-alert-settings"></a>Paramètres d’alerte « Data feed not available » (Flux de données non disponible)

Un flux de données est considéré comme non disponible si aucune donnée n’est ingérée depuis la source au cours de la période de grâce spécifiée à compter du début de l’ingestion du flux de données. Dans ce cas, une alerte est déclenchée.

Pour configurer une alerte, vous devez d’abord [créer un hook](alerts.md#create-a-hook). Les alertes seront envoyées via le hook configuré.

* **Grace period** (Période de grâce) : ce paramètre est utilisé pour déterminer à quel moment envoyer une alerte si aucun point de données n’est ingéré. Le point de référence est l’heure de la première ingestion. En cas d’échec d’une ingestion, Metrics Advisor continue de s’exécuter à un intervalle régulier spécifié par la granularité. Si l’échec persiste après la période de grâce, une alerte est envoyée.

* **Auto snooze** (Répétition automatique) : Lorsque cette option a la valeur zéro, chaque timestamp à l’état *Not Available* (Non disponible) déclenche une alerte. Si vous spécifiez un paramètre différent de zéro, les timestamps continus qui suivent le premier timestamp *non disponible* ne se déclenchent en fonction du paramètre spécifié.

## <a name="next-steps"></a>Étapes suivantes
- [Configurer des métriques et affiner la configuration de la détection](configure-metrics.md)
- [Ajuster la détection des anomalies à l’aide de commentaires](anomaly-feedback.md)
- [Diagnostiquer un incident](diagnose-incident.md).
