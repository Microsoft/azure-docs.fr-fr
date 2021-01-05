---
title: Tutoriel - Effectuer le rendu d’une scène de Blender avec Azure Batch et Batch Explorer
description: 'Didacticiel : comment effectuer le rendu de plusieurs images à partir d’une scène de Blender avec Azure Batch et l’application cliente Batch Explorer'
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: tutorial
ms.openlocfilehash: 5c015e4e83b0505a45690e92c1d732f74b6f6b0e
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106459"
---
# <a name="tutorial-render-a-blender-scene-using-batch-explorer"></a>Didacticiel : Effectuer le rendu d’une scène de Blender avec Batch Explorer

Ce didacticiel montre comment effectuer le rendu de plusieurs images d’une scène de démonstration de Blender. Blender est utilisé pour le didacticiel car il est gratuit pour les machines virtuelles clients et de rendu, mais le processus est très similaire si d’autres applications, telles que Maya ou 3ds Max, sont utilisées.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Charger une scène de Blender sur le Stockage Azure
> * Créer un pool Batch avec plusieurs nœuds pour effectuer le rendu
> * Effectuer le rendu de plusieurs images
> * Afficher et télécharger les fichiers de rendu des images

## <a name="prerequisites"></a>Conditions préalables requises

Vous avez besoin d’un abonnement de paiement à l’utilisation ou autre option d’achat Azure pour utiliser les applications de rendu dans Batch sur une base de paiement à l’utilisation. Les licences de paiement à l’utilisation ne sont pas prises en charge si vous utilisez une offre Azure gratuite qui propose un crédit monétaire.

Un compte Azure Batch avec un compte de stockage associé est requis.  Consultez un des articles de démarrage rapide de Batch, comme [l’article CLI](./quick-create-cli.md) pour créer un compte Batch.

Un quota de cœurs de faible priorité d’au moins 50 cœurs est requis pour la taille de machine virtuelle et le nombre de machines virtuelles spécifiées dans ce didacticiel. Le quota par défaut peut être utilisé, mais une machine virtuelle de plus petite taille devra être utilisée, ce qui signifie que le rendu des images prendra plus de temps. Le processus permettant de demander une augmentation du quota de cœurs accrue est détaillé dans [cet article](./batch-quota-limit.md).

Enfin, [Batch Explorer](https://azure.github.io/BatchExplorer/) doit être installé. Il est disponible sur Windows, OSX et Linux. Il est facultatif, mais si [Blender](https://www.blender.org/download/) est installé, l’exemple de fichier modèle peut être affiché.

## <a name="download-the-demo-scene"></a>Télécharger la scène de démonstration

Téléchargez le fichier ZIP de démonstration « Salle de classe » pour Blender à partir de la [page web des fichiers de démonstration Blender](https://www.blender.org/download/demo-files/) et décompressez-le sur un lecteur local.

## <a name="upload-a-scene-to-azure-storage"></a>Charger une scène sur le Stockage Azure

Créer un conteneur de compte de stockage pour les fichiers de scène de démonstration :

* Démarrez Batch Explorer.
* Sélectionnez l’élément de menu « Données » dans le menu principal sur le côté gauche.
* Vérifiez que « groupes de fichiers » est sélectionné dans le menu déroulant.
* Sélectionnez le bouton « + » et créer un nouveau « groupe de fichiers vide » nommé « blender-classroom ».
  * Un groupe de fichiers est simplement un conteneur d’objets blob de stockage Azure avec un préfixe « fgrp- » ; Il s’agit d’une convention utilisée pour filtrer les autres conteneurs dans le compte de stockage

![Groupe de fichiers pour les fichiers de scène](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup.png)

Télécharger les fichiers de scène :

* Sélectionnez le nouveau conteneur et faites glisser-déplacer le contenu du dossier « classe » vers le conteneur dans Batch Explorer.

![Fichiers de scène chargés](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup_uploaded.png)

## <a name="create-azure-storage-container-for-output-images"></a>Créer un conteneur de stockage Azure pour les images de sortie

Créer un conteneur de compte de stockage pour les fichiers de sortie de scène de démonstration :

* Sélectionnez l’élément de menu « Données » dans le menu principal sur le côté gauche.
* Sélectionnez le bouton « + » et créer un nouveau « groupe de fichiers vide » nommé « render-output ».

![Groupe de fichiers pour les fichiers de sortie](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_filegroup.png)

## <a name="create-a-pool-of-vms-for-rendering"></a>Créer un pool de machines virtuelles pour le rendu

Créez un pool Batch à l’aide du rendu de l’image de la machine virtuelle de la place de marché Azure contenant l’application Blender :

* Sélectionnez l’élément de menu « Galerie » dans le menu principal sur le côté gauche.
* Sélectionnez l’élément « Blender » pour obtenir la liste des éléments de l’application.
* Sélectionner les éléments pour le rendu des images sur Windows Server
* L’icône de lien sur le côté droit de l’élément peut éventuellement être sélectionné pour afficher les fichiers de modèle pouvant être utilisés pour créer un pool et un travail.

![Éléments de la galerie Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_gallery_item.png)

* Sélectionnez le bouton « Créer un pool pour une utilisation ultérieure ».
  * Laissez le nom du Pool « blender windows ».
  * Définissez « Dedicated vm count » sur « 0 ».
  * Définissez « Low priority vm count » sur « 3 ».
  * Définissez « Node size » sur « Standard_F16 » : une autre taille de machine virtuelle peut être sélectionnée, mais le temps nécessaire pour effectuer le rendu d’une image dépendra principalement du nombre de cœurs.
* Sélectionnez le bouton vert pour créer le pool.
  * Le pool sera créé presque immédiatement, mais il faudra quelques minutes pour que les machines virtuelles soient allouées et démarrées.

![Modèle de pool pour Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_template.png)

> [!WARNING]
> Notez que lorsque des machines virtuelles sont présentes dans un pool, le coût de ces machines virtuelles est facturé sur votre abonnement Azure. Le pool ou les machines virtuelles doivent être supprimés pour arrêter les frais. Supprimez le pool à la fin de ce didacticiel pour éviter les frais persistants.

L’état du pool et des machines virtuelles peut être surveillé dans la vue « Pools ».L’exemple suivant montre les trois machines virtuelles allouées, deux ont été démarrées et sont inactives, la dernière est toujours en train de démarrer : ![carte thermique du pool](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap.png)

## <a name="create-a-rendering-job"></a>Créer un travail de rendu

Créez un travail de rendu pour effectuer le rendu d’images à l’aide du pool créé :

* Sélectionnez l’élément de menu « Galerie » dans le menu principal sur le côté gauche.
* Sélectionnez l’élément « Blender » pour obtenir la liste des éléments de l’application.
* Sélectionnez les éléments pour le rendu des images sur Windows Server.
* Sélectionnez le bouton « Exécuter le travail avec un pool existant »
* Sélectionnez le pool « blender-windows »
* Définissez le « nom du travail » sur «blender-rendu-tutorial1 »
* Sélectionnez « fgrp-blender-classroom » pour « Données d’entrée »
* Sélectionnez l’icône de fichier pour le « fichier Blend » et sélectionnez « classroom.blend »
* Laisser « Image de début »sur « 1 » et «Image de fin » sur « 5 »
* Définissez « Sorties » sur « fgrp-render-output »
* Sélectionnez le bouton vert pour créer le travail ; un travail sera créé avec cinq tâches, une pour chaque image

![Modèle de travail pour Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_template.png)

Une fois que le travail et toutes les tâches ont été créés, le travail s’affiche avec les tâches de travail : ![liste des tâches de travail](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_task_list.png)

Lorsqu’une tâche commence à s’exécuter pour la première fois sur une machine virtuelle du pool, une tâche de préparation de programme de traitement par lots est exécutée, copiant les fichiers de scène du groupe de fichiers de stockage sur la machine virtuelle pour les rendre accessibles par Blender.
L’état du rendu peut être déterminé en consultant le fichier journal stdout.txt produit par Blender.  Sélectionnez une tâche, les « sorties de tâche » sont affichées par défaut, le fichier « stdout.txt » peut être sélectionné et affiché.
![fichier stdout](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_stdout.png)

Si le pool « blender windows » est sélectionné, les machines virtuelles du pool seront visibles dans un état d’exécution : ![Carte thermique du pool avec les nœuds en cours d’exécution](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap_running.png)

La production des images ayant fait l’objet d’un rendu demandera plusieurs minutes en fonction de la taille de la machine virtuelle sélectionnée.  À l’aide de la machine virtuelle F16 spécifiée précédemment, le rendu des images prend environ 16 minutes.

## <a name="view-the-rendering-output"></a>Afficher la sortie du rendu

Une fois le rendu des images terminé, ces tâches seront affichées comme étant terminées : ![Tâches terminées](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_tasks_complete.png)

L’image rendue est d’abord écrite sur la machine virtuelle et elle peut être affichée en sélectionnant le dossier « wd » : ![Image rendue sur le nœud du pool](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image.png)

Le modèle de travail spécifie également que les fichiers journaux et de l’image de sortie sont réécris dans le groupe de fichiers du compte Stockage Azure spécifié une fois le travail créé.  L’interface utilisateur « Data » peut être utilisée pour afficher les fichiers de sortie et les journaux d’activité. Elle peut également être utilisée pour télécharger les fichiers : ![Image rendue dans le groupe de fichiers de stockage](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image_storage.png)

Une fois toutes les tâches terminées, le travail est marqué comme étant terminé : ![Travail et toutes les tâches terminés](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_alltasks_complete.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

> [!WARNING]
> Le pool doit être supprimé (il peut également être redimensionné jusqu’à zéro nœud) pour arrêter les frais venant des machines virtuelles facturées à l’abonnement Azure.

* Sélectionnez « Pools »
* Sélectionnez le pool « blender-windows »
* Faites un clic droit et cliquez sur « Supprimer » ou bien sélectionnez l’icône Corbeille au-dessus du pool

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Charger une scène de Blender sur le Stockage Azure
> * Créer un pool Batch avec plusieurs nœuds pour effectuer le rendu
> * Effectuer le rendu de plusieurs images
> * Afficher et télécharger les fichiers de rendu des images

Continuez en explorant les applications de rendu disponibles par le biais de Batch Explorer dans la section **Galerie**. Pour chaque application, plusieurs modèles sont disponibles, et d’autres seront disponibles au fil du temps. Par exemple, des modèles Blender permettent de diviser une image en une mosaïque, de sorte que des parties de l’image puissent être rendues en parallèle.

Pour en savoir plus sur le rendu à l’échelle du cloud, consultez les options relatives au service Batch Rendering.

> [!div class="nextstepaction"]
> [Service de rendu Batch](batch-rendering-service.md)
