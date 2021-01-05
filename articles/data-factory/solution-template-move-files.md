---
title: Déplacer des fichiers entre des stockages basés sur des fichiers
description: Découvrez comment utiliser un modèle de solution pour déplacer des fichiers entre les stockages basés sur fichier à l’aide d’Azure Data Factory.
services: data-factory
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 7/12/2019
ms.openlocfilehash: f6baea73c0c4964bb3937304603a2a92a13d52b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86522718"
---
# <a name="move-files-with-azure-data-factory"></a>Déplacer des fichiers à l’aide de Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

L’activité de copie ADF offre une prise en charge intégrée du scénario « de déplacement » lors de la copie de fichiers binaires entre des magasins de stockage.  La manière de l’activer consiste à définir « deleteFilesAfterCompletion » comme vrai dans l’activité de copie. En procédant ainsi, l’activité de copie supprimera les fichiers du magasin de sources de données une fois la tâche terminée. 

Cet article décrit un modèle de solution comme une autre approche en tirant parti du flux de contrôle flexible ADF, de l’activité de copie et de l’activité de suppression pour atteindre le même scénario. L’un des scénarios courants d’utilisation de ce modèle est le suivant : Les fichiers sont continuellement déposés dans un dossier de destination de votre magasin source. En créant un déclencheur de planification, le pipeline de ADF peut déplacer régulièrement ces fichiers de la source vers le magasin de destination.  La façon dont le pipeline ADF crée des « fichiers mobiles » consiste à récupérer les fichiers du dossier de destination, à copier chacun d’eux dans un autre dossier sur le magasin de destination, puis à supprimer les mêmes fichiers du dossier de destination du magasin source.

> [!NOTE]
> N’oubliez pas que ce modèle est conçu pour déplacer des fichiers au lieu de déplacer des dossiers.  Si vous souhaitez déplacer le dossier en modifiant le jeu de données pour qu’il ne contienne qu’un chemin d’accès au dossier, puis que vous utilisez l’activité de copie et l’activité de suppression pour référencer le même jeu de données qui représente un dossier, vous devez être très prudent. En effet, vous devez veiller à ce qu’aucun nouveau fichier n’arrive dans le dossier entre l’opération de copie et l’opération de suppression. Si de nouveaux fichiers arrivent dans le dossier au moment où votre activité Copy vient juste d’achever la copie mais avant le démarrage de l’activité Delete, il est possible que cette dernière, en supprimant la totalité du dossier, supprime le nouveau fichier qui n’a pas encore été copié dans la destination.

## <a name="about-this-solution-template"></a>À propos de ce modèle de solution

Ce modèle obtient les fichiers de votre stockage source basé sur fichier. Il déplace ensuite chacun d’eux vers le stockage de destination.

Le modèle contient cinq activités :
- **GetMetadata** obtient la liste des objets, y compris les fichiers et sous-dossiers de votre dossier dans le stockage source. Il ne récupère pas les objets de manière récursive. 
- **Filtre** filtrez la liste d’objets à partir de l’activité **GetMetadata** pour sélectionner les fichiers uniquement. 
- **ForEach** obtient la liste des fichiers à partir de l’activité de **Filtre**, puis itère au sein de la liste et transmet chaque fichier à l’activité Copier et Supprimer.
- **Copier** copie un fichier de la source vers le stockage de destination.
- **Supprimer** supprime le même fichier du stockage source.

Le modèle définit quatre paramètres :
- *SourceStore_Location* est le chemin du dossier de votre stockage source à partir duquel vous souhaitez déplacer des fichiers. 
- *SourceStore_Directory* est le chemin du sous-dossier de votre stockage source à partir duquel vous souhaitez déplacer des fichiers.
- *DestinationStore_Location* est le chemin du dossier de votre stockage de destination vers lequel vous souhaitez déplacer des fichiers. 
- *DestinationStore_Directory* est le chemin du sous-dossier de votre stockage de destination vers lequel vous souhaitez déplacer des fichiers.

## <a name="how-to-use-this-solution-template"></a>Utiliser ce modèle de solution

1. Accédez au modèle **Déplacer des fichiers**. Sélectionnez la connexion existante ou créez une **Nouvelle** connexion au stockage de fichiers source à partir duquel vous souhaitez déplacer des fichiers. Sachez que **DataSource_Folder** et **DataSource_File** font référence à la même connexion de votre stockage de fichiers source.

    ![Créer une connexion à la source](media/solution-template-move-files/move-files1.png)

2. Sélectionnez la connexion existante ou créez une **Nouvelle** connexion au stockage de fichiers de destination à partir duquel vous souhaitez déplacer des fichiers.

    ![Créer une connexion à la destination](media/solution-template-move-files/move-files2.png)

3. Sélectionnez l’onglet **Utiliser ce modèle**.
    
4. Vous verrez le pipeline, comme indiqué dans l’exemple suivant :

    ![Afficher le pipeline](media/solution-template-move-files/move-files4.png)

5. Sélectionnez **Déboguer**, entrez les **Paramètres**, puis sélectionnez **Terminer**.   Les paramètres sont le chemin d’accès du dossier à partir duquel vous souhaitez déplacer les fichiers et le chemin d’accès au dossier dans lequel vous souhaitez déplacer les fichiers. 

    ![Exécuter le pipeline](media/solution-template-move-files/move-files5.png)

6. Vérifiez le résultat.

    ![Vérifier le résultat](media/solution-template-move-files/move-files6.png)

## <a name="next-steps"></a>Étapes suivantes

- [Copier les fichiers nouveaux et modifiés par LastModifiedDate avec Azure Data Factory](solution-template-copy-new-files-lastmodifieddate.md)

- [Copier des fichiers issus de plusieurs conteneurs avec Azure Data Factory](solution-template-copy-files-multiple-containers.md)