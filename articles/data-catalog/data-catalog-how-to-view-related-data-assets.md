---
title: Guide pratique pour afficher les ressources de données associées dans Azure Data Catalog
description: Cet article explique comment afficher les ressources de données associées à la ressource de données sélectionnée dans Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: fb5352b4cc84801296d9e621053672328b02692a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017268"
---
# <a name="how-to-view-related-data-assets-in-azure-data-catalog"></a>Comment afficher les ressources de données associées dans Azure Data Catalog
Azure Data Catalog vous permet d’afficher les ressources de données associées à la ressource de données sélectionnée, ainsi que les relations qui sont établies entre elles. 

## <a name="supported-data-sources"></a>Sources de données prises en charge 
Lorsque vous inscrivez des ressources de données provenant des sources de données suivantes, Azure Data Catalog inscrit automatiquement les métadonnées concernant les relations de jointure qui existent entre les ressources de données sélectionnées. 

- SQL Server
- Azure SQL Database
- MySQL
- Oracle

> [!NOTE]
> Pour que Data Catalog puisse importer la relation entre deux ressources de données, vous devez inscrire les deux ressources en même temps. Si vous les avez déjà ajoutées séparément, ajoutez-les de nouveau ensemble pour importer leur relation.

## <a name="view-related-data-assets"></a>Afficher les ressources de données associées
Pour afficher les ressources de données associées au jeu de données sélectionné, cliquez sur l’onglet **Relations**, comme illustré ci-dessous : 

![Azure Data Catalog - Afficher les ressources de données associées](media/data-catalog-how-to-view-related-data-assets/relationships-tab.png)

Dans cet exemple, la ressource de données **ProductSubcategory** a deux relations : 

- La colonne ProductSubcategoryID de la table Product a une relation de clé étrangère avec la colonne ProductSubcategoryID présente dans la table ProductSubcategory sélectionnée. 
- La colonne ProductCategoryID de la table ProductSubCategory a une relation de clé étrangère avec la colonne ProductCategoryID présente dans la table ProductCategory sélectionnée.

> [!NOTE]
> Remarquez la direction de la flèche dans l’arborescence des relations.  

Pour plus d’informations, telles que le nom complet de la colonne, déplacez la souris sur la colonne pour afficher une info-bulle semblable à celle-ci : 

![Azure Data Catalog - Menu contextuel Relation](media/data-catalog-how-to-view-related-data-assets/relationship-popup.png)

Pour inclure des relations entre ressources qui ont déjà été inscrites, vous devez réinscrire les ressources.

## <a name="next-steps"></a>Étapes suivantes
- [Gestion des ressources de données](data-catalog-how-to-manage.md)
