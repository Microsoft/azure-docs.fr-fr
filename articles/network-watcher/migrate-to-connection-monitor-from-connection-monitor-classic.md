---
title: Migrer vers le Moniteur de connexion à partir du Moniteur de connexion
titleSuffix: Azure Network Watcher
description: Découvrez comment migrer vers le Moniteur de connexion à partir du Moniteur de connexion.
services: network-watcher
documentationcenter: na
author: vinynigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: vinigam
ms.openlocfilehash: a97f3bf3a479d956747755a421d7946f7aafd4f1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494237"
---
# <a name="migrate-to-connection-monitor-from-connection-monitor-classic"></a>Migrer vers le Moniteur de connexion à partir du Moniteur de connexion (classique)

En quelques clics et sans temps d’arrêt, vous pouvez migrer des moniteurs de connexion vers la nouvelle fonctionnalité Moniteur de connexion améliorée. Pour en savoir plus sur les avantages, consultez [Moniteur de connexion](./connection-monitor-overview.md).

## <a name="key-points-to-note"></a>Points clés à noter

La migration produit les résultats suivants :

* Les agents et les paramètres de pare-feu fonctionnent tels quels. Aucune modification n’est nécessaire. 
* Les moniteurs de connexion existants seront mappés vers Moniteur de connexion > Groupe de test > Format de test. En sélectionnant **Modifier**, vous pouvez afficher et modifier les propriétés de la nouvelle fonctionnalité Moniteur de connexion, télécharger un modèle pour apporter des modifications à Moniteur de connexion et le soumettre via Azure Resource Manager. 
* Les machines virtuelles Azure avec l’extension Network Watcher envoient des données à l’espace de travail et aux métriques. Le Moniteur de connexion met les données à disposition par le biais des nouvelles métriques (ChecksFailedPercent et RoundTripTimeMs) à la place des anciennes métriques (ProbesFailedPercent et AverageRoundtripMs). 
* Analyse des données :
   * **Alertes** : Migrées automatiquement vers les nouvelles métriques.
   * **Tableaux de bord et intégrations** : Nécessite la modification manuelle de l’ensemble de métriques. 
    
## <a name="prerequisites"></a>Prérequis

Si vous utilisez un espace de travail personnalisé, assurez-vous que Network Watcher est activé dans votre abonnement et dans la région de votre espace de travail Log Analytics. 

## <a name="migrate-the-connection-monitors"></a>Migrer les moniteurs de connexion

1. Pour migrer les anciens moniteurs de connexion vers la version plus récente, sélectionnez **Moniteur de connexion**, puis sélectionnez **Migrer les moniteurs de connexion**.

    ![Capture d’écran montrant la migration des moniteurs de connexion vers Moniteur de connexion.](./media/connection-monitor-2-preview/migrate-cm-to-cm-preview.png)
    
1. Sélectionnez votre abonnement et les moniteurs de connexion que vous souhaitez migrer, puis sélectionnez **Migrer les éléments sélectionnés**. 

En quelques clics seulement, vous avez migré les moniteurs de connexion existants vers Moniteur de connexion. 

Vous pouvez désormais personnaliser les propriétés de Moniteur de connexion, modifier l’espace de travail par défaut, télécharger des modèles et vérifier l’état de la migration. 

Après le début de la migration, les modifications suivantes ont lieu : 
* La ressource Azure Resource Manager passe au moniteur de connecter plus récent.
    * Le nom, la région et l’abonnement du moniteur de connexion restent inchangés. L’ID de ressource n’est pas concerné.
    * Sauf si le moniteur de connexion est personnalisé, un espace de travail Log Analytics par défaut est créé dans l’abonnement et dans la région du moniteur de connexion. Cet espace de travail est l’emplacement où sont stockées les données d’analyse. Les données du résultat de test sont également stockées dans les métriques.
    * Chaque test est migré vers un groupe de test appelé *defaultTestGroup*.
    * Les points de terminaison source et de destination sont créés et utilisés dans le groupe de test. Les noms par défaut sont *defaultSourceEndpoint* et *defaultDestinationEndpoint*.
    * Le port de destination et l’intervalle de détection sont déplacés vers la configuration de test appelée *defaultTestConfiguration*. Le protocole est défini sur la base des valeurs de port. Les seuils de réussite et autres propriétés facultatives sont laissés en blanc.
* Les alertes de métriques sont migrées vers les alertes de métriques de Moniteur de connexion. Les métriques sont différentes, d’où la modification. Pour plus d’informations, consultez [Surveillance de la connectivité réseau à l’aide de Moniteur de connexion](./connection-monitor-overview.md#metrics-in-azure-monitor).
* Les moniteurs de connexion migrés ne sont plus affichés comme l’ancienne solution de moniteur de connexion. Ils sont désormais disponibles uniquement dans Moniteur de connexion.
* Toutes les intégrations externes, telles que les tableaux de bord dans Power BI et Grafana, et les intégrations aux systèmes Security Information and Event Management (SIEM) doivent être migrées manuellement. Il s’agit de la seule étape manuelle que vous devez effectuer pour migrer votre installation.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le Moniteur de connexion, consultez :
* [Migration de Network Performance Monitor vers Moniteur de connexion](./migrate-to-connection-monitor-from-network-performance-monitor.md)
* [Créer un Moniteur de connexion à l’aide du portail Azure](./connection-monitor-create-using-portal.md)