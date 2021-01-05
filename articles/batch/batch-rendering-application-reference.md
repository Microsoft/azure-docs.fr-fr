---
title: Utilisation d’applications de rendu
description: Découvrez comment utiliser des applications de rendu avec Azure Batch. Cet article décrit brièvement comment exécuter chaque application de rendu.
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: ace3fe7aee6b9ffc7226448b455bcfea1f931458
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85964870"
---
# <a name="rendering-applications"></a>Applications de rendu

Vous utilisez des applications de rendu en créant des travaux et tâches Batch. La propriété de ligne de commande de la tâche spécifie la ligne de commande et les paramètres appropriés.  Pour créer les tâches de travail, la méthode la plus simple consiste à utiliser les modèles Batch Explorer comme indiqué dans [cet article](./batch-rendering-using.md#using-batch-explorer).  Les modèles peuvent être affichés et des versions modifiées peuvent être créées si nécessaire.

Cet article décrit brièvement comment exécuter chaque application de rendu.

## <a name="rendering-with-autodesk-3ds-max"></a>Rendu avec Autodesk 3ds Max

### <a name="renderer-support"></a>Renderers pris en charge

Outre les renderers intégrés à 3ds Max, les renderers suivants sont disponibles sur les images de machine virtuelle de rendu et peuvent être référencés par le fichier de scène 3ds Max :

* Autodesk Arnold
* V-Ray de Chaos Group

### <a name="task-command-line"></a>Ligne de commande de la tâche

Appelez l’application `3dsmaxcmdio.exe` pour effectuer un rendu par ligne de commande sur un nœud du pool.  Cette application se situe sur le chemin d’accès lorsque la tâche est exécutée. Les paramètres disponibles dans l’application `3dsmaxcmdio.exe` sont les mêmes que ceux disponibles dans l’application `3dsmaxcmd.exe`. Pour en savoir plus à ce sujet, reportez-vous à la [documentation d’aide de 3ds Max](https://help.autodesk.com/view/3DSMAX/2018/ENU/) (Rendering (Rendu) | Section Command-Line Rendering (Rendu par ligne de commande)).

Par exemple :

```
3dsmaxcmdio.exe -v:5 -rfw:0 -start:{0} -end:{0} -bitmapPath:"%AZ_BATCH_JOB_PREP_WORKING_DIR%\sceneassets\images" -outputName:dragon.jpg -w:1280 -h:720 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scenes\dragon.max"
```

Remarques :

* Veillez attentivement à ce que les fichiers d’éléments multimédias soient trouvés.  Assurez-vous que les chemins d’accès sont corrects et qu’il s’agit des chemins relatifs à l’aide de la fenêtre **Asset Tracking** (Suivi des ressources), ou utilisez le paramètre `-bitmapPath` dans la ligne de commande.
* Vérifiez la présence de problèmes éventuels avec le rendu, comme l’impossibilité de localiser les ressources, en vérifiant le fichier `stdout.txt` enregistré par 3ds Max lors de l’exécution de la tâche.

### <a name="batch-explorer-templates"></a>Modèles Batch Explorer

Vous pouvez accéder à des modèles de pool et de travaux dans la **galerie** de Batch Explorer.  Les fichiers sources des modèles sont disponibles dans le [référentiel de données de Batch Explorer dans GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/3dsmax).

## <a name="rendering-with-autodesk-maya"></a>Rendu avec Autodesk Maya

### <a name="renderer-support"></a>Renderers pris en charge

Outre les renderers intégrés à Maya, les renderers suivants sont disponibles sur les images de machine virtuelle de rendu et peuvent être référencés par le fichier de scène 3ds Max :

* Autodesk Arnold
* V-Ray de Chaos Group

### <a name="task-command-line"></a>Ligne de commande de la tâche

Le renderer par ligne de commande `renderer.exe` est utilisé dans la ligne de commande de la tâche. Pour plus d’informations sur le renderer par ligne de commande, reportez-vous à [l’aide de Maya](https://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=GUID-EB558BC0-5C2B-439C-9B00-F97BCB9688E4).

Dans l’exemple suivant, nous utilisons une tâche de préparation du travail pour copier les fichiers de scènes et d’éléments multimédias dans le répertoire de travail de la préparation du travail. Un dossier de sortie est utilisé pour stocker l’image du rendu, et l’image 10 est rendue.

```
render -renderer sw -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

Pour le rendu V-Ray, le fichier de scène Maya spécifie normalement V-Ray comme renderer.  Celui-ci peut également être spécifié dans la ligne de commande :

```
render -renderer vray -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

Pour le rendu Arnold, le fichier de scène Maya spécifie normalement Arnold comme renderer.  Celui-ci peut également être spécifié dans la ligne de commande :

```
render -renderer arnold -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

### <a name="batch-explorer-templates"></a>Modèles Batch Explorer

Vous pouvez accéder à des modèles de pool et de travaux dans la **galerie** de Batch Explorer.  Les fichiers sources des modèles sont disponibles dans le [référentiel de données de Batch Explorer dans GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/maya).

## <a name="next-steps"></a>Étapes suivantes

Utilisez les modèles de pool et de travaux du [référentiel de données dans GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj) à l’aide de Batch Explorer.  Si nécessaire, créez des modèles ou modifiez l’un des modèles fournis.
