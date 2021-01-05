---
title: Bonnes pratiques relatives au Magasin des requêtes dans Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit les bonnes pratiques relatives au Magasin des requêtes dans Azure Database pour PostgreSQL - Serveur unique.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: dd39b7ecd51902f5035b4cd17d59dea964d0c962
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91708830"
---
# <a name="best-practices-for-query-store"></a>Bonnes pratiques relatives au Magasin des requêtes

**S’applique à :** Azure Database pour PostgreSQL - Serveur unique versions 9.6, 10, 11

Cet article décrit les bonnes pratiques relatives à l’utilisation du Magasin des requêtes dans Azure Database pour PostgreSQL.

## <a name="set-the-optimal-query-capture-mode"></a>Définir le mode de capture de requête optimal
Laissez le Magasin des requêtes capturer les données qui vous intéressent. 

|**pg_qs.query_capture_mode** | **Scénario**|
|---|---|
|_Tout_  |Analysez attentivement votre charge de travail concernant toutes les requêtes, leur fréquence d’exécution et d’autres statistiques. Identifiez les nouvelles requêtes dans votre charge de travail. Détectez si des requêtes ad hoc sont utilisées pour identifier les opportunités de paramétrage défini par l’utilisateur ou automatique. _All_ s’accompagne du coût de la consommation accrue des ressources. |
|_Top_  |Concentrez votre attention sur les principales requêtes : celles émises par les clients.
|_Aucun_ |Vous avez déjà capturé un ensemble de requêtes et une fenêtre de temps que vous souhaitez examiner, et vous voulez éliminer les distractions que d’autres requêtes peuvent introduire. _None_ est adapté pour les environnements de test et d’évaluation. _None_ doit être utilisé avec précaution, car vous risquez de laisser passer l’opportunité de suivre et d’optimiser d’importantes nouvelles requêtes. Vous ne pouvez pas récupérer des données sur ces fenêtres de temps. |

Le Magasin des requêtes inclut également un magasin pour les statistiques d’attente. Il existe une requête en mode de capture supplémentaire qui régit les statistiques d’attente : **pgms_wait_sampling.query_capture_mode** peut être définie sur _none_ ou sur _all_. 

> [!NOTE] 
> **pg_qs.query_capture_mode** remplace **pgms_wait_sampling.query_capture_mode**. Si pg_qs.query_capture_mode a la valeur _none_, le paramètre pgms_wait_sampling.query_capture_mode n’a aucun effet. 


## <a name="keep-the-data-you-need"></a>Conserver les données dont vous avez besoin
Le paramètre **pg_qs.retention_period_in_days** spécifie, en jours, la période de conservation des données pour le Magasin des requêtes. Les données des requêtes et des statistiques plus anciennes sont supprimées. Par défaut, le Magasin des requêtes est configuré pour conserver les données pendant 7 jours. Évitez de conserver les données historiques que vous ne prévoyez pas d’utiliser. Augmentez la valeur si vous avez besoin de conserver les données plus longtemps.


## <a name="set-the-frequency-of-wait-stats-sampling"></a>Définir la fréquence d’échantillonnage des statistiques d’attente 
Le paramètre **pgms_wait_sampling.history_period** spécifie la fréquence (en millisecondes) d’échantillonnage des événements d’attente. Plus la période est courte, plus l’échantillonnage est fréquent. Des informations supplémentaires sont récupérées, mais cela s’accompagne d’une hausse du coût de la consommation des ressources. Prolongez cette période si le serveur est en cours de chargement ou si vous n’avez pas besoin d’autant de précision.


## <a name="get-quick-insights-into-query-store"></a>Obtenir des insights rapides dans le Magasin des requêtes
Vous pouvez utiliser [Query Performance Insight](concepts-query-performance-insight.md) dans le portail Azure pour obtenir des insights rapides des données du Magasin des requêtes. Les visualisations exposent les requêtes dont l’exécution est la plus longue et les événements d’attente les plus longs dans le temps.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment obtenir ou définir des paramètres à l’aide du [portail Azure](howto-configure-server-parameters-using-portal.md) ou de l’interface [Azure CLI](howto-configure-server-parameters-using-cli.md).