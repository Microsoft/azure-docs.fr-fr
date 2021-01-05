---
title: Configurer un mappage de jeu de données dans Azure Data Share
description: Découvrez comment configurer un mappage de jeu de données pour un partage reçu dans Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 08/14/2020
ms.openlocfilehash: eed3e8275400a3e677df53b9d62cf0e0bc70271c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88257843"
---
# <a name="how-to-configure-a-dataset-mapping-for-a-received-share-in-azure-data-share"></a>Comment configurer un mappage de jeu de données pour un partage reçu dans Azure Data Share

Cet article explique comment configurer un mappage de jeu de données pour un partage reçu à l’aide d’Azure Data Share. Vous pouvez configurer un mappage de jeu de données pour spécifier un magasin de données cible dans lequel vous souhaitez recevoir des données, ou si vous devez modifier le magasin de données cible.

## <a name="navigate-to-a-received-data-share"></a>Accéder à un partage de données reçues

Dans le service Azure Data Share, accédez à votre partage reçu et sélectionnez l’onglet **Jeux de données**. 

![Mappage de jeu de données](./media/dataset-mapping.png "Mappage de jeu de données") 

Cochez la case en regard du jeu de données auquel vous souhaitez affecter une destination. Sélectionnez **+ Map to target** (+ Mapper sur cible) afin de choisir un nouveau magasin de destination. Sélectionnez **Démapper** si le jeu de données est déjà mappé et que vous souhaitez modifier le magasin de données cible.

![Mapper sur cible](./media/dataset-map-target.png "Mapper sur cible") 

## <a name="select-a-target-store"></a>Sélectionner un magasin cible

Sélectionnez un type de magasin de données cible dans lequel vous souhaitez que les données arrivent. Pour les partages basés sur une capture instantanée, toutes les données existant déjà dans les comptes de stockage précédemment mappés ne sont pas automatiquement déplacées vers le nouveau magasin cible. Pour les partages sur place, sélectionnez un magasin de données à l’emplacement spécifié. L’emplacement est le centre de données Azure où se trouve le magasin de données source du fournisseur de données.

![Compte de stockage cible](./media/dataset-map-target-sql.png "Stockage cible") 

## <a name="select-a-file-format-sql-sources-only"></a>Sélectionner un format de fichier (sources SQL uniquement)

Si les données sources proviennent d’une source basée sur SQL et que vous voulez les recevoir en tant que fichier, vous pouvez choisir le format dans lequel elles sont reçues. 

![Choisir le format](./media/sql-file-formats.png "Formats de fichiers SQL")

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment commencer à partager des données, passez au tutoriel [Partager vos données](share-your-data.md).



