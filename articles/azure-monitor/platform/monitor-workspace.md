---
title: Surveiller l’intégrité d’un espace de travail Log Analytics dans Azure Monitor
description: Décrit comment surveiller l’intégrité de votre espace de travail Log Analytics à l’aide des données du tableau Opération.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/20/2020
ms.openlocfilehash: 3cb01a8f1c06bad618ae5c7930920ee0f067038c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184533"
---
# <a name="monitor-health-of-log-analytics-workspace-in-azure-monitor"></a>Surveiller l’intégrité d’un espace de travail Log Analytics dans Azure Monitor
Pour maintenir les performances et la disponibilité de votre espace de travail Log Analytics dans Azure Monitor, vous devez être en mesure de détecter de façon proactive les problèmes qui surviennent. Cet article décrit comment surveiller l’intégrité de votre espace de travail Log Analytics à l’aide des données du tableau [Opération](/azure/azure-monitor/reference/tables/operation). Ce tableau, qui figure dans tous les espaces de travail Log Analytics, présente les erreurs et les avertissements qui surviennent dans votre espace de travail. Nous vous conseillons de consulter ces données régulièrement et de créer des alertes pour être informé en amont des incidents importants survenus dans votre espace de travail.

## <a name="_logoperation-function"></a>Fonction _LogOperation

Azure Monitor Logs envoie des informations relatives aux problèmes au tableau [Opération](/azure/azure-monitor/reference/tables/operation) de l’espace de travail concerné. La fonction système **_LogOperation** est basée sur le tableau **Opération** et fournit un jeu d’informations simplifié pour l’analyse et la création d’alertes.

## <a name="columns"></a>Colonnes

La fonction **_LogOperation** renvoie les colonnes du tableau suivant.

| Colonne | Description |
|:---|:---|
| TimeGenerated | Heure (UTC) à laquelle l’incident est survenu. |
| Category  | Groupe de catégorie de l’opération. Peut être utilisé pour filtrer sur les types d’opérations et créer des audits et des alertes système plus précis. La liste des catégories figure dans la section ci-dessous. |
| Opération  | Description du type d’opération. Cet élément peut indiquer l’une des limites de Log Analytics, le type d’opération ou une partie d’un processus. |
| Level | Niveau de gravité du problème :<br>- Info : Aucune attention particulière n’est requise.<br>- Avertissement : Le processus ne s’est pas terminé comme prévu, votre attention est requise.<br>- Erreur : Le processus a échoué et une attention urgente est requise. 
| Détail | La description détaillée de l’opération contient le message d’erreur spécifique le cas échéant. |
| _ResourceId | ID de ressource de la ressource Azure liée à l’opération.  |
| Computer | Nom de l’ordinateur si l’opération est liée à un agent Azure Monitor. |
| CorrelationId | Utilisé pour regrouper les opérations liées consécutives. |


## <a name="categories"></a>Catégories

Le tableau suivant décrit les catégories de la fonction _LogOperation. 

| Category | Description |
|:---|:---|
| Ingestion           | Opérations faisant partie du processus d’ingestion de données. Vous trouverez plus de détails à la suite. |
| Agent               | Indique un problème lié à l’installation de l’agent. |
| Collecte de données     | Opérations liées aux processus de collecte des données. |
| Ciblage de la solution  | L’opération de type *ConfigurationScope* a été traitée. |
| Solution d’évaluation | Un processus d’évaluation a été exécuté. |


### <a name="ingestion"></a>Ingestion
Les opérations d’ingestion sont les problèmes qui sont survenus pendant l’ingestion de données, comme les notifications concernant les limites de l’espace de travail Azure Log Analytics atteintes. Les conditions d’erreur de cette catégorie peuvent indiquer une perte de données. Il est particulièrement intéressant de les surveiller. Le tableau suivant fournit des informations détaillées sur ces opérations. Pour connaître les limites de service des espaces de travail Log Analytics, consultez [Limites de services Azure Monitor](../service-limits.md#log-analytics-workspaces).


| Opération | Level | Détail | Article connexe |
|:---|:---|:---|:---|
| Journal personnalisé | Erreur   | Limite de la colonne Champs personnalisés atteinte. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Journal personnalisé | Erreur   | Échec de l’ingestion des journaux personnalisés. | |
| Métadonnées. | Erreur | Erreur de configuration détectée. | |
| Collecte de données | Erreur   | Les données ont été supprimées, car la demande a été créée avant le nombre de jours défini. | [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](manage-cost-storage.md#alert-when-daily-cap-reached)
| Collecte de données | Informations    | Une configuration de l’ordinateur de collecte est détectée.| |
| Collecte de données | Informations    | La collecte de données a commencé en raison du changement de jour. | [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached) |
| Collecte de données | Avertissement | La collecte de données s’est arrêtée, car la limite quotidienne a été atteinte.| [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached) |
| Traitement des données | Erreur   | Format JSON non valide. | [Transmettre des données à Azure Monitor avec l’API Collecteur de données HTTP (préversion publique)](data-collector-api.md#request-body) | 
| Traitement des données | Avertissement | La valeur a été tronquée à la taille maximale autorisée. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Traitement des données | Avertissement | Valeur de champ tronquée en raison de la limite de taille. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) | 
| Taux d’ingestion | Informations | Limite du taux d’ingestion proche de 70 %. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Taux d’ingestion | Avertissement | Le taux d’ingestion est proche de la limite. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Taux d’ingestion | Erreur   | Limite du taux atteinte. | [Limites du service Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Stockage | Erreur   | Impossible d’accéder au compte de stockage, car les informations d’identification utilisées ne sont pas valides.  |



   

## <a name="alert-rules"></a>Règles d'alerte
Utilisez les [alertes de requête de journal](../platform/alerts-log-query.md) dans Azure Monitor pour être informé en amont lorsqu’un problème est détecté dans votre espace de travail Log Analytics. Nous vous recommandons d’utiliser une stratégie qui vous permette de répondre à temps aux problèmes tout en réduisant vos coûts. Votre abonnement est facturé pour chaque règle d’alerte avec un coût qui dépend de la fréquence à laquelle il est évalué.

La stratégie recommandée consiste à commencer par deux règles d’alerte en fonction du niveau du problème. Utilisez une fréquence courte, par exemple toutes les 5 minutes, pour les erreurs et une fréquence plus longue, par exemple 24 heures, pour les avertissements. Dans la mesure où les erreurs indiquent une potentielle perte de données, vous pouvez les traiter rapidement pour réduire les pertes. Les avertissements indiquent généralement un problème qui ne requiert pas une attention immédiate. Vous pouvez donc les examiner de façon quotidienne.

Utilisez le processus décrit dans [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../platform/alerts-log.md) pour créer des alertes de journal. Les sections suivantes décrivent les détails pour chaque règle.


| Requête | Valeur du seuil | Période | Fréquence |
|:---|:---|:---|:---|
| `_LogOperation | where Level == "Error"`   | 0 | 5 | 5 |
| `_LogOperation | where Level == "Warning"` | 0 | 1440 | 1440 |

Ces règles d’alerte répondent de la même manière à toutes les opérations indiquant Erreur ou Avertissement. Une fois familiarisé avec les opérations qui génèrent des alertes, vous pouvez répondre différemment à certaines opérations. Par exemple, vous pouvez envoyer des notifications à différentes personnes pour des opérations spécifiques. 

Pour créer une règle d’alerte pour une opération spécifique, utilisez une requête comportant les colonnes **Catégorie** et **Opération**. 

L’exemple suivant crée une alerte Avertissement lorsque le taux de volume d’ingestion atteint 80 % de la limite.

- Cible : Sélectionnez votre espace de travail Log Analytics.
- Critères :
  - Nom du signal : Recherche personnalisée dans les journaux
  - Requête de recherche : `_LogOperation | where Category == "Ingestion" | where Operation == "Ingestion rate" | where Level == "Warning"`
  - Basé sur : Nombre de résultats
  - Condition : Supérieur à
  - Seuil : 0
  - Période : 5 (minutes)
  - Fréquence : 5 (minutes)
- Nom de la règle d'alerte : limite de données quotidienne atteinte
- Gravité : avertissement (Sev 1)


L’exemple suivant crée une alerte Avertissement lorsque la collecte de données atteint la limite quotidienne. 

- Cible : Sélectionnez votre espace de travail Log Analytics.
- Critères :
  - Nom du signal : Recherche personnalisée dans les journaux
  - Requête de recherche : `_LogOperation | where Category == "Ingestion" | where Operation == "Data Collection" | where Level == "Warning"`
  - Basé sur : Nombre de résultats
  - Condition : Supérieur à
  - Seuil : 0
  - Période : 5 (minutes)
  - Fréquence : 5 (minutes)
- Nom de la règle d'alerte : limite de données quotidienne atteinte
- Gravité : avertissement (Sev 1)



## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [alertes de journal](alerts-log.md).
- [Collectez des données d’audit de requête](../log-query/query-audit.md) pour votre espace de travail.