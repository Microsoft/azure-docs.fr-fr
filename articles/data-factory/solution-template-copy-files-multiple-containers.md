---
title: Copie de fichiers à partir de plusieurs conteneurs
description: Découvrez comment utiliser un modèle de solution pour copier des fichiers provenant de plusieurs conteneurs à l’aide d’Azure Data Factory.
services: data-factory
author: dearandyxu
ms.author: yexu
ms.reviewer: douglasl
manager: anandsub
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/1/2018
ms.openlocfilehash: f78d0b02c9790234a63ef64200dcab72bc64c033
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629423"
---
# <a name="copy-multiple-folders-with-azure-data-factory"></a>Copier plusieurs dossiers avec Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit un modèle de solution que vous pouvez utiliser plusieurs activités de copie pour copier des conteneurs ou des dossiers entre des magasins basés sur des fichiers, où chaque activité de copie est supposée copier un conteneur ou un dossier unique. 

> [!NOTE]
> Si vous souhaitez copier des fichiers provenant d’un même conteneur, il est plus efficace d’utiliser l’[outil Copier des données](copy-data-tool.md) pour créer un pipeline avec une seule activité de copie. Le modèle dans cet article suffit amplement pour ce scénario simple.

## <a name="about-this-solution-template"></a>À propos de ce modèle de solution

Ce modèle énumère les dossiers d’un dossier parent donné sur votre magasin de stockage source. Il copie ensuite chacun des dossiers dans le magasin de destination.

Le modèle comporte trois activités :
- **GetMetadata** analyse votre magasin de stockage source et obtient la liste des sous-dossiers d’un dossier parent donné.
- **ForEach** obtient la liste des sous-dossiers auprès de l’activité **GetMetadata** , puis itère au sein de la liste et transmet chaque dossier à l’activité de copie.
- **Copy** copie chaque dossier du magasin de stockage source dans le magasin de destination.

Le modèle définit les paramètres suivants :
- *SourceFileFolder* fait partie du chemin d’accès au dossier parent de votre magasin de sources de données : *SourceFileFolder/SourceFileDirectory* , où vous pouvez obtenir la liste des sous-dossiers. 
- *SourceFileDirectory* fait partie du chemin d’accès au dossier parent de votre magasin de sources de données : *SourceFileFolder/SourceFileDirectory* , où vous pouvez obtenir la liste des sous-dossiers. 
- *DestinationFileFolder* fait partie du chemin d’accès au dossier parent : *DestinationFileFolder/DestinationFileDirectory* , où les fichiers seront copiés dans votre magasin de destination. 
- *DestinationFileDirectory* fait partie du chemin d’accès au dossier parent : *DestinationFileFolder/DestinationFileDirectory* , où les fichiers seront copiés dans votre magasin de destination. 

Si vous souhaitez copier plusieurs conteneurs sous des dossiers racine entre des magasins de stockage, vous pouvez entrer les quatre paramètres comme */* . En procédant ainsi, vous répliquez tout ce qui se trouve entre les magasins de stockage.

## <a name="how-to-use-this-solution-template"></a>Utiliser ce modèle de solution

1. Accédez au modèle **Copier plusieurs conteneurs de fichiers entre des magasins de fichiers**. Créez une **nouvelle** connexion à votre magasin de stockage source. C’est dans ce magasin que seront copiés les fichiers provenant de plusieurs conteneurs.

    ![Créer une connexion à la source](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image1.png)

2. Créez une **nouvelle** connexion à votre magasin de stockage de destination.

    ![Créer une connexion à la destination](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image2.png)

3. Sélectionnez **Utiliser ce modèle**.

    ![Utiliser ce modèle](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image3.png)
    
4. Vous verrez le pipeline, comme indiqué dans l’exemple suivant :

    ![Afficher le pipeline](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image4.png)

5. Sélectionnez **Déboguer** , entrez les **Paramètres** , puis sélectionnez **Terminer**.

    ![Exécuter le pipeline](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image5.png)

6. Vérifiez le résultat.

    ![Vérifier le résultat](media/solution-template-copy-files-multiple-containers/copy-files-multiple-containers-image6.png)

## <a name="next-steps"></a>Étapes suivantes

- [Copier en bloc à partir d’une base de données à l’aide d’une table de contrôle avec Azure Data Factory](solution-template-bulk-copy-with-control-table.md)

- [Copier des fichiers issus de plusieurs conteneurs avec Azure Data Factory](solution-template-copy-files-multiple-containers.md)
