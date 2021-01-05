---
title: Actions de menu MMC du Gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Explique comment utiliser les actions de menu standard de la console Microsoft Management Console (MMC) du gestionnaire d’instantanés StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 78ef81af-0d3a-4802-be54-ad192f9ac8a6
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 01064c3668b673baea7aedd9a65c92b03c48dc10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90056001"
---
# <a name="use-the-mmc-menu-actions-in-storsimple-snapshot-manager"></a>Utilisez les actions du menu MMC dans Gestionnaire d’instantanés StorSimple

## <a name="overview"></a>Vue d’ensemble
Dans le Gestionnaire d’instantanés StorSimple, les actions suivantes figureront sur tous les menus d’action et toutes les variations du volet **Actions** .

* Affichage
* Nouvelle fenêtre à partir d’ici 
* Actualiser 
* Exporter la liste 
* Aide 

Ces actions font partie de la Microsoft Management Console (MMC) et ne sont pas spécifiques au gestionnaire d’instantanés StorSimple. Ce didacticiel décrit ces actions et explique comment utiliser chacune d’elles dans le Gestionnaire d’instantanés StorSimple.

## <a name="view"></a>Affichage
Vous pouvez utiliser l’option **Affichage** pour modifier la vue du volet **Résultats** et l’affichage de la fenêtre de console. 

#### <a name="to-change-the-results-pane-view"></a>Pour modifier l’affichage du volet Résultats
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit de la souris sur n’importe quel nœud ou développez le nœud et cliquez avec le bouton droit de la souris sur un élément du volet **Résultats**, puis cliquez sur l’option **Affichage**. 
3. Pour ajouter ou supprimer les colonnes qui apparaissent dans le volet **Résultats**, cliquez sur **Ajout/Suppression de colonnes**. La boîte de dialogue **Ajout/Suppression de colonnes** .
   
    ![Ajouter ou supprimer des colonnes dans le volet Résultats](./media/storsimple-snapshot-manager-mmc-menu/HCS_SSM_Add_remove_columns.png) 
4. Remplissez le formulaire comme suit :
   
   * Sélectionnez des éléments dans la liste de colonnes **Disponible** et cliquez sur **Ajouter** pour les ajouter à la liste **Colonnes affichées**. 
   * Cliquez sur les éléments dans la liste **Colonnes affichées**, puis cliquez sur **Supprimer** pour les supprimer de la liste. 
   * Sélectionnez un élément dans la liste de colonnes **Affichées** et cliquez sur **Monter** ou **Descendre** pour déplacer l’élément vers le haut ou vers le bas dans la liste. 
   * Cliquez sur **Paramètres par défaut** pour revenir à la configuration par défaut du volet **Résultats**. 
5. Une fois que vous avez terminé vos sélections, cliquez sur **OK**. 

#### <a name="to-change-the-console-window-view"></a>Pour modifier l’affichage de la fenêtre de console
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur n’importe quel nœud, sur **Affichage**, puis cliquez sur **Personnaliser**. La boîte de dialogue **Personnaliser** s’affiche.
   
    ![Personnaliser la fenêtre de console](./media/storsimple-snapshot-manager-mmc-menu/HCS_SSM_Customize.png) 
3. Activez ou désactivez les cases à cocher pour afficher ou masquer des éléments dans la fenêtre de console. Une fois que vous avez terminé vos sélections, cliquez sur **OK**.

## <a name="new-window-from-here"></a>Nouvelle fenêtre à partir d’ici
Vous pouvez utiliser l’option **Nouvelle fenêtre à partir d’ici** pour ouvrir une nouvelle fenêtre de console.

#### <a name="to-open-a-new-console-window"></a>Ouvrir une nouvelle fenêtre de console
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez sur n’importe quel nœud avec le bouton droit de la souris, puis cliquez sur **Nouvelle fenêtre**. 
   
    Une nouvelle fenêtre indiquant uniquement l’étendue que vous avez sélectionnée s’affiche. Par exemple, si vous cliquez avec le bouton droit de la souris sur le nœud **Stratégies de sauvegarde**, la nouvelle fenêtre affiche uniquement le nœud **Stratégies de sauvegarde** dans le volet **Étendue** et une liste de stratégies de sauvegarde dans le volet **Résultats**. Consultez l’exemple qui suit.
   
    ![Nouvelle fenêtre à partir d’ici](./media/storsimple-snapshot-manager-mmc-menu/HCS_SSM_NewWindow.png) 

## <a name="refresh"></a>Actualiser
Vous pouvez utiliser l’action **Actualiser** pour mettre à jour la fenêtre de console.

#### <a name="to-update-the-console-window"></a>Pour mettre à jour la fenêtre de console
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit de la souris sur n’importe quel nœud ou développez le nœud et cliquez avec le bouton droit de la souris sur un élément du volet **Résultats**, puis cliquez sur l’option **Actualiser**. 

## <a name="export-list"></a>Exporter la liste
Vous pouvez utiliser l’action **Exporter la liste** pour enregistrer une liste dans un fichier de valeurs séparées par des virgules (CSV). Vous pouvez par exemple exporter la liste des stratégies de sauvegarde ou le catalogue de sauvegarde. Vous pouvez ensuite importer le fichier CSV dans une application de tableur pour analyse.

#### <a name="to-save-a-list-in-a-comma-separated-value-csv-file"></a>Pour enregistrer une liste dans un fichier CSV (fichier de valeurs séparées par des virgules)
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple. 
2. Dans le volet **Étendue**, cliquez avec le bouton droit de la souris sur n’importe quel nœud ou développez le nœud et cliquez avec le bouton droit de la souris sur un élément du volet **Résultats**, puis cliquez sur l’option **Exporter la liste**. 
3. La boîte de dialogue **Exporter la liste** s’affiche. Remplissez le formulaire comme suit : 
   
   1. Dans la zone **Nom de fichier** , tapez un nom pour le fichier CSV ou cliquez sur la flèche pour effectuer une sélection dans la liste déroulante.
   2. Dans la zone **Type de fichier** , cliquez sur la flèche et sélectionnez un type de fichier dans la liste déroulante.
   3. Pour enregistrer uniquement les éléments sélectionnés, sélectionnez les lignes puis cliquez sur la case à cocher **Enregistrer uniquement les lignes sélectionnées** . Pour enregistrer toutes les listes exportées, désactivez la case à cocher **Enregistrer uniquement les lignes sélectionnées** .
   4. Cliquez sur **Enregistrer**.
      
      ![Exporter la liste dans un fichier de valeurs séparées par des virgules](./media/storsimple-snapshot-manager-mmc-menu/HCS_SSM_Export_List.png) 

## <a name="help"></a>Aide
Utilisez le menu **Aide** pour consulter l’aide en ligne disponible du Gestionnaire d’instantanés StorSimple et de MMC .

#### <a name="to-view-available-online-help"></a>Pour afficher l’aide en ligne disponible
1. Cliquez sur l’icône de bureau pour démarrer le Gestionnaire d’instantanés StorSimple.
2. Dans le volet **Étendue**, cliquez avec le bouton droit de la souris sur n’importe quel nœud ou développez le nœud et cliquez avec le bouton droit de la souris sur un élément du volet **Résultats**, puis cliquez sur l’option **Aide**. 

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [l’interface utilisateur du Gestionnaire d’instantanés StorSimple](storsimple-use-snapshot-manager.md).
* En savoir plus sur [l’utilisation du Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).

