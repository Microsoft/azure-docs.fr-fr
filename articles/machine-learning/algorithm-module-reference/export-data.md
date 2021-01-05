---
title: 'Exporter des données : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Utilisez le module Exporter des données dans le concepteur Azure Machine Learning pour enregistrer les résultats et les données intermédiaires en dehors d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/28/2020
ms.openlocfilehash: c6e3d56958168cd279c98a4ba4c021c2362c2694
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421309"
---
# <a name="export-data-module"></a>Module Exporter les données

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour enregistrer les résultats, les données intermédiaires et les données de travail de vos pipelines dans les destinations de stockage cloud. 

Ce module prend en charge l’exportation de vos données dans les services de données cloud suivants :

- Conteneur d’objets blob Azure
- Partage de fichiers Azure
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2
- Base de données Azure SQL

Avant d’exporter vos données, vous devez d’abord inscrire un magasin de données dans votre espace de travail Azure Machine Learning. Pour plus d'informations, consultez la page [Accéder aux données dans les services de stockage Azure](../how-to-access-data.md).

## <a name="how-to-configure-export-data"></a>Configuration de l’exportation des données

1. Ajoutez le module **Exporter des données** à votre pipeline dans le concepteur. Ce module figure dans la catégorie **Entrée et sortie**.

1. Connectez **Exporter des données** au module qui contient les données à exporter.

1. Sélectionnez **Exporter les données** pour ouvrir le volet **Propriétés**.

1. Pour **Magasin de données**, sélectionnez un magasin de données existant dans la liste déroulante. Vous pouvez également créer un magasin de données. Découvrez comment en consultant [Accéder aux données dans les services de stockage Azure](../how-to-access-data.md).

    > [!NOTE]
    > Il n’est pas possible d’exporter des données d’un certain type vers une colonne de base de données SQL spécifiée d’un autre type. La table cible n’a pas besoin d’exister en premier.

1. La case à cocher **Régénérer la sortie** détermine s’il faut exécuter le module pour régénérer la sortie au moment de l’exécution. 

    Elle est désélectionnée par défaut, ce qui signifie que, si le module a été exécuté avec les mêmes paramètres par le passé, le système réutilisera la sortie de la dernière exécution pour réduire le temps d’exécution. 

    Si elle est sélectionnée, le système exécutera à nouveau le module pour régénérer la sortie.

1. Définissez le chemin d’accès du magasin de données où se trouvent les données. Le chemin d’accès est un chemin d’accès relatif. Les chemins d’accès vides ou les chemins d’accès d’URL ne sont pas autorisés.


1. Pour **Format de fichier**, sélectionnez le format dans lequel les données doivent être stockées.
 
1. Envoyez le pipeline.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
