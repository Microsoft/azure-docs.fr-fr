---
title: Gérer les ressources et les quotas
titleSuffix: Azure Purview
description: Découvrez les quotas et les limites de ressources pour Azure Machine Learning et apprenez à demander des augmentations de quota.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/12/2020
ms.openlocfilehash: 57cb1c405e00acb346421d64190a71e9211d21ff
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938832"
---
# <a name="manage-and-increase-quotas-for-resources-with-azure-purview"></a>Gestion et augmentation des quotas de ressources avec Azure Purview
 
Azure Purview est un service cloud destiné aux utilisateurs de données. Il offre la possibilité de gérer de manière centralisée la gouvernance des données sur l’ensemble du patrimoine de données, comprenant à la fois les environnements cloud et les environnements locaux. Il permet aux analystes d’entreprise de rechercher des données pertinentes en utilisant des termes métier significatifs. Pour augmenter les limites jusqu'à la limite maximale pour votre abonnement, contactez le support.
 
## <a name="azure-purview-limits"></a>Limites d’Azure Purview
 
|**Ressource**|  **Limite par défaut**  |**Limite maximale**|
|---|---|---|
|Comptes Purview par région, par locataire (tous abonnements confondus)|3|Contacter le support technique|
|vCore disponibles pour l’analyse, par compte*|160|160|
|Analyses simultanées, par compte à un moment donné (limite établie en fonction du type des sources de données analysées*)|5 | 10 |
|Durée maximale d’exécution d’une analyse|7 jours|7 jours|
|Appels d’API, par compte|10 millions d’API/mois pour la taille de plateforme de 4 unités de capacité <br>40 millions d’API/mois pour la taille de plateforme de 16 unités de capacité |10 millions d’API/mois pour la taille de plateforme de 4 unités de capacité <br>40 millions d’API/mois pour la taille de plateforme de 16 unités de capacité|
|Stockage, par compte|10 Go pour la taille de plateforme de 4 unités de capacité <br>40 Go pour la taille de plateforme de 16 unités de capacité |10 Go pour la taille de plateforme de 4 unités de capacité <br> 40 Go pour la taille de plateforme de 16 unités de capacité |
|Taille des ressources par compte|100 millions de ressources physiques |Contacter le support technique|
|Taille maximale d’une ressource dans un catalogue|2 Mo|2 Mo|
|Longueur maximale d’un nom de ressource et d’un nom de classification|4 Ko|4 Ko|
|Longueur maximale du nom et de la valeur d’une propriété de ressource|32 Ko|32 Ko|
|Longueur maximale du nom et de la valeur d’un attribut de classification|32 Ko|32 Ko|
|Nombre maximal de termes de glossaire, par compte|100K|100K|
 
\* Les scénarios de runtime d’intégration auto-hébergés ne sont pas concernés par les limites définies dans le tableau ci-dessus. 
 
## <a name="next-steps"></a>Étapes suivantes
 
> [!div class="nextstepaction"]
>[Tutoriel : Analyse des données avec Azure Purview](tutorial-scan-data.md)

> [!div class="nextstepaction"]
>[Tutoriel : Parcourir la page d’accueil et rechercher une ressource](tutorial-asset-search.md)

> [!div class="nextstepaction"]
>[Tutoriel : Recherche de ressources et affichage de leur traçabilité](tutorial-browse-and-view-lineage.md)
