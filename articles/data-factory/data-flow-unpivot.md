---
title: Annulation de la transformation de tableau croisé dynamique dans le flux de données de mappage
description: Transformation de suppression de tableau croisé dynamique de flux de données de mappage Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/14/2020
ms.openlocfilehash: ef861cdf394716a70d85e43ce9c60f46af2cc2e4
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040209"
---
# <a name="unpivot-transformation-in-mapping-data-flow"></a>Annulation de la transformation de tableau croisé dynamique dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la suppression de tableau croisé dynamique dans le flux de données de mappage ADF pour transformer un jeu de données non normalisé en version plus normalisée en développant les valeurs de colonnes multiples d’un seul enregistrement dans plusieurs enregistrements avec les mêmes valeurs dans une colonne unique.

![Capture d’écran montrant l’option Supprimer le tableau croisé dynamique sélectionnée dans le menu.](media/data-flow/unpivot1.png "Options de suppression de tableau croisé dynamique 1")

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4B1RR]

## <a name="ungroup-by"></a>Dissocier par

![Capture d’écran montrant les paramètres Supprimer le tableau croisé dynamique avec l’onglet Dissocier par sélectionné.](media/data-flow/unpivot5.png "Options de suppression de tableau croisé dynamique 2")

Commencez par définir les colonnes de dissociation souhaitées pour l'agrégation de votre suppression de tableau croisé dynamique. Définissez une ou plusieurs colonnes à dissocier avec le signe + en regard de la liste des colonnes.

## <a name="unpivot-key"></a>Clé de suppression du tableau croisé dynamique

![Capture d’écran montrant les paramètres Supprimer le tableau croisé dynamique avec l’onglet Clé de suppression du tableau croisé dynamique sélectionné.](media/data-flow/unpivot6.png "Options de suppression de tableau croisé dynamique 3")

La Clé de suppression de tableau croisé dynamique est la colonne à partir de laquelle ADF ajoutera un tableau croisé dynamique à une ligne. Par défaut, chaque valeur unique comprise dans le jeu de données de ce champ ajoutera un tableau croisé dynamique à une ligne. Toutefois, si vous le souhaitez, vous pouvez entrer les valeurs du jeu de données que vous souhaitez ajouter en tant que tableau croisé dynamique aux valeurs de la ligne.

## <a name="unpivoted-columns"></a>Tableau croisé dynamique des colonnes supprimé

![Capture d’écran montrant les paramètres Supprimer le tableau croisé dynamique avec l’onglet Aperçu des données sélectionné.](media/data-flow//unpivot7.png "Options de suppression de tableau croisé dynamique 4")

Enfin, choisissez le nom de la colonne de stockage des valeurs des colonnes faisant l'objet de la suppression de tableau croisé dynamique qui sont transformées en lignes.

(Facultatif) Vous pouvez supprimer des lignes contenant des valeurs Null.

Par exemple, SumCost est le nom de colonne choisi dans l'exemple présenté ci-dessus.

![Image représentant les colonnes PO, Vendor et Fruit avant et après une transformation UnPivot utilisant la colonne Fruit comme clé de suppression du tableau croisé dynamique.](media/data-flow/unpivot3.png)

La disposition des colonnes sur « Normal » regroupera toutes les nouvelles colonnes concernées par la suppression de tableau croisé dynamique à partir d'une seule valeur. La disposition des colonnes sur « Latéral » regroupera les nouvelles colonnes concernées par la suppression de tableau croisé dynamique générées à partir d'une colonne existante.

![Capture d’écran montrant le résultat de la transformation.](media/data-flow//unpivot7.png "Options de suppression de tableau croisé dynamique 5")

Le tableau croisé dynamique supprimé du jeu de résultats final affiche maintenant les totaux de colonnes sous forme de tableau croisé dynamique supprimé dans des valeurs de ligne distinctes.

## <a name="next-steps"></a>Étapes suivantes

Utilisez la [transformation Tableau croisé dynamique](data-flow-pivot.md) pour convertir des lignes en colonnes.
