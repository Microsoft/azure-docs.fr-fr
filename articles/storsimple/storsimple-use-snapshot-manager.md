---
title: Interface utilisateur du gestionnaire d’instantanés StorSimple | Microsoft Docs
description: Décrit l’interface utilisateur de gestionnaire d’instantanés StorSimple et explique comment l’utiliser pour gérer les tâches de sauvegarde et le catalogue de sauvegarde.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: c7d91892-2881-41a2-a7a2-908dc3646493
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.custom: ''
ms.openlocfilehash: 35fa0fc134045585eebd21324352f9b4ff272e87
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954237"
---
# <a name="use-storsimple-snapshot-manager-user-interface-to-manage-backup-jobs-and-backup-catalog"></a>Utiliser l’interface utilisateur StorSimple Snapshot Manager pour gérer les tâches de sauvegarde et le catalogue de sauvegarde

## <a name="overview"></a>Vue d’ensemble
StorSimple Snapshot Manager présente une interface utilisateur intuitive qui vous permet de procéder à des sauvegardes et de les gérer. Ce didacticiel présente l’interface utilisateur, puis explique comment utiliser chacun des composants. Pour obtenir une description détaillée du Gestionnaire d'instantanés StorSimple, consultez [Qu'est-ce que le Gestionnaire d'instantanés StorSimple ?](storsimple-what-is-snapshot-manager.md)

### <a name="console-description"></a>Description de la console
Pour afficher l’interface utilisateur, cliquez sur l’icône Gestionnaire d’instantanés StorSimple de votre bureau. La fenêtre de console s’affiche, comme indiqué dans l’illustration suivante.

![Volets du gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_gui_panes.png)

La fenêtre de console comporte cinq éléments principaux. Cliquez sur le lien approprié pour une description complète de chaque élément.

* [Barre de menus](#menu-bar) 
* [Barre d’outils](#tool-bar) 
* [Volet Étendue](#scope-pane) 
* [Volet Résultats](#results-pane) 
* [Volet Actions](#actions-pane) 

En outre, le Gestionnaire d’instantanés StorSimple prend en charge [navigation par clavier et un certain nombre de raccourcis](#keyboard-navigation-and-shortcuts).

### <a name="console-accessibility"></a>Accessibilité de la console
L’interface utilisateur du gestionnaire d’instantanés StorSimple prend en charge les fonctionnalités d’accessibilité fournies par le système d’exploitation Windows et la console Microsoft Management Console (MMC), ainsi que certains raccourcis clavier de gestionnaire d’instantanés StorSimple spécifiques. 

* Pour obtenir une description des fonctionnalités d’accessibilité de Windows, accédez aux [raccourcis clavier pour Windows](https://support.microsoft.com/kb/126449). 
* Pour obtenir une description des fonctionnalités d’accessibilité MMC, consultez la section [Accessibilité à MMC 3.0](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc766075(v=ws.11))
* Pour obtenir une description des fonctionnalités d’accessibilité du Gestionnaire d’instantanés StorSimple, consultez la section [Navigation et raccourcis clavier](#keyboard-navigation-and-shortcuts).

## <a name="menu-bar"></a>Barre de menus
La barre de menus située en haut de la fenêtre de console contient les menus [Fichier](#file-menu), [Action](#action-menu), [Affichage](#view-menu), [Favoris](#favorites-menu), [Fenêtre](#window-menu) et [Aide](#help-menu).

Cliquez sur n’importe quel élément de la barre de menus pour afficher la liste des commandes disponibles dans ce menu. L’exemple suivant montre le menu **Affichage** sélectionné dans la barre de menus.

![Menu Affichage sélectionné](./media/storsimple-use-snapshot-manager/HCS_SSM_View_menu.png)

### <a name="file-menu"></a>Menu Fichier
Le menu **Fichier** contient les commandes standard de Microsoft Management Console (MMC).

#### <a name="menu-access"></a>Accès au menu
Pour afficher le menu **Fichier**, cliquez sur **Fichier** dans la barre de menus. Le menu suivant s’affiche.

![Menu Fichier Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_FileMenu.png) 

#### <a name="menu-description"></a>Description de menu
Le tableau suivant décrit des éléments qui apparaissent dans le menu **Fichier** .

| Élément de menu | Description |
|:--- |:--- |
| Nouveau |Cliquez sur **Nouveau** pour créer une nouvelle console à partir du Gestionnaire d’instantanés StorSimple. |
| Ouvrir |Cliquez sur **Ouvrir** pour ouvrir une console existante. |
| Enregistrer |Cliquez sur **Enregistrer** pour enregistrer la console actuelle. |
| Enregistrer sous |Cliquez sur **Enregistrer sous** pour créer une nouvelle instance de la console actuelle avec un nouveau nom. Utilisez l’option **Enregistrer sous** pour personnaliser un affichage et l’enregistrer pour une récupération ultérieure. Par exemple, vous pouvez créer des composants logiciels enfichables Gestionnaire d’instantanés StorSimple pointant vers des serveurs spécifiques. |
| Ajouter/Supprimer le composant logiciel enfichable |Cliquez sur **Ajouter/Supprimer le composant logiciel enfichable** pour ajouter ou supprimer des composants et organiser les nœuds dans le volet **Étendue**. Pour plus d’informations, consultez la page [Ajouter, supprimer et organiser des composants logiciels enfichables et les extensions dans MMC 3.0](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc722035(v=ws.11)). |
| Options |Cliquez sur **Options** pour modifier l’icône de la console, spécifier des modes d’accès et les autorisations utilisateur, ou supprimer des fichiers de console pour augmenter l’espace disque disponible. |
| Liste des chemins d’accès |Cliquez sur un chemin d’accès dans la liste numérotée pour rouvrir un fichier récemment ouvert. |
| Quitter |Cliquez sur **Quitter** pour fermer le menu **Fichier**. |

### <a name="action-menu"></a>Menu Action
Utilisez le menu **Action** pour effectuer votre choix parmi les actions disponibles. Les éléments disponibles dépendent de la sélection effectuée dans le volet **Étendue** ou **Résultats**.

#### <a name="menu-access"></a>Accès au menu
Pour afficher le menu **Action** , effectuez l’une des opérations suivantes :

* Cliquez avec le bouton droit sur un élément dans le volet **Étendue** ou **Résultats**.
* Sélectionnez un élément dans le volet **Étendue** ou le volet **Résultats**, puis, sur la barre de menus, cliquez sur **Action**. 

Par exemple, si vous sélectionnez le nœud supérieur dans le volet **Étendue**, puis effectuez un clic droit ou, dans la barre de menus, cliquez sur **Action**, le menu suivant s’affiche.

![Menu Action du Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_Action_menu.png)

Le volet **Actions** (à droite de la console) contient la même liste d’actions que le menu **Action**. En outre, le volet **Actions** contient les options du menu **Affichage** qui vous permettent de créer une vue personnalisée du volet **Résultats**.

![Volet Actions avec menu Affichage ouvert](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane_Results.png)

#### <a name="menu-description"></a>Description de menu
Le tableau suivant contient une liste alphabétique des actions du Gestionnaire d’instantanés StorSimple. 

* La colonne **Action** répertorie les actions que vous pouvez effectuer sur les nœuds et les résultats. 
* La colonne **Navigation** explique comment afficher le menu **Action** approprié pour que vous puissiez sélectionner l’action. Certaines actions apparaissent dans plusieurs menus **Action** . Pour ces actions, dans la liste à puces, sélectionnez l’option **Navigation** . 
* La colonne **Description** explique comment utiliser chaque action présente dans le menu **Action** ou le volet Actions et explique ce qu’elle fait.

> [!NOTE]
> Le volet **Actions** et les menus **Action** contiennent des options supplémentaires telles qu’**Affichage**, **Nouvelle fenêtre à partir d’ici**, **Actualiser**, **Exporter la liste** et **Aide**. Ces options sont disponibles dans le cadre de la console MMC et ne sont pas spécifiques au gestionnaire d’instantanés StorSimple. La table comprend des descriptions de ces options.
> 
> 

| Action | Navigation | Description |
|:--- |:--- |:--- |
| Authenticate |Cliquez sur le nœud **Appareils**, puis cliquez avec le bouton droit dans le volet **Résultats**. |Cliquez sur **Authentifier** pour saisir le mot de passe que vous avez configuré pour l’appareil. |
| Clone |Développez **Catalogue de sauvegarde**, développez **Instantanés Cloud**, cliquez sur une sauvegarde datée, puis sélectionnez un volume dans le volet **Résultats**. |Cliquez sur **Clone** pour créer une copie d’instantané cloud et pour stocker celle-ci dans l’emplacement que vous désignez. |
| Configuration d’un appareil |Cliquez avec le bouton droit de la souris sur le nœud **Appareils** . |Cliquez sur **Configurer un appareil** pour configurer un ou plusieurs appareils pour se connecter à l’hôte Windows. |
| Créer la stratégie de sauvegarde |Effectuez l’une des actions suivantes :<ul><li>Cliquez avec le bouton droit sur **Stratégies de sauvegarde**.</li><li>Cliquez sur ou développez **Groupes de volumes**, puis cliquez avec le bouton droit sur un groupe de volumes.</li><li>Cliquez sur ou développez **Catalogue de sauvegarde**, puis cliquez avec le bouton droit sur un groupe de volumes.</li></ul> |Cliquez sur **Créer une stratégie de sauvegarde** pour configurer une sauvegarde planifiée pour un groupe de volumes. |
| Créer un groupe de volumes |Effectuez l’une des actions suivantes :<ul><li>Cliquez sur le nœud **Volumes**, puis cliquez avec le bouton droit sur un volume dans le volet **Résultats**.</li><li>Cliquez avec le bouton droit de la souris sur le nœud **Groupes de volumes** .</li></ul> |Cliquez sur **Créer un groupe de volumes** pour affecter des volumes à un groupe de volumes. |
| DELETE |Cliquez sur un nœud ou sur un résultat (cet élément apparaît dans de nombreux menus **Action** et volets **Actions**). |Cliquez sur **Supprimer** pour supprimer le nœud ou le résultat que vous avez sélectionné. Lorsque la boîte de dialogue de confirmation s’affiche, confirmez ou annulez la suppression. |
| Détails |Cliquez sur le nœud **Appareils**, puis cliquez avec le bouton droit sur un appareil dans le volet **Résultats**. |Cliquez sur **Détails** pour afficher les détails de configuration d’un appareil. |
| Modifier |Cliquez sur **Stratégies de sauvegarde**, puis cliquez avec le bouton droit sur une stratégie dans le volet **Résultats**. |Cliquez sur **Modifier** pour modifier la planification de sauvegarde d’un groupe de volumes. |
| Exporter la liste |Cliquez sur un nœud ou un résultat (cet élément apparaît dans tous les menus **Action** et les volets **Actions**). |Cliquez sur **Exporter la liste** pour enregistrer une liste dans un fichier CSV (fichier de valeurs séparées par des virgules). Vous pouvez ensuite importer ce fichier dans un tableur pour analyse. |
| Aide |Cliquez sur un nœud ou un résultat. (Cet élément apparaît dans tous les menus **Action** et les volets **Actions**.) |Cliquez sur **Aide** pour ouvrir l’aide en ligne dans une fenêtre de navigateur distincte. |
| Nouvelle fenêtre à partir d’ici |Cliquez sur un nœud ou un résultat (cet élément apparaît dans tous les menus **Action** et les volets **Actions**). |Cliquez sur **Nouvelle fenêtre à partir d’ici** pour ouvrir une nouvelle fenêtre du Gestionnaire d’instantanés StorSimple. |
| Actualiser |Cliquez sur un nœud ou un résultat (cet élément apparaît dans tous les menus **Action** et les volets **Actions**). |Cliquez sur **Actualiser** pour mettre à jour la fenêtre Gestionnaire d’instantanés StorSimple actuellement affichée. |
| Actualiser l’appareil |Cliquez sur le nœud **Appareils**, puis cliquez avec le bouton droit dans le volet **Résultats**. |Cliquez sur **Actualiser l’appareil** pour synchroniser un appareil spécifique connecté avec le Gestionnaire d’instantanés StorSimple. |
| Actualiser la liste des appareils |Cliquez avec le bouton droit de la souris sur le nœud **Appareils** . |Cliquez sur **Actualiser la liste des appareils** pour synchroniser la liste des appareils connectés avec le Gestionnaire d’instantanés StorSimple. |
| Relancer l’analyse des volumes |Cliquez avec le bouton droit sur le nœud **Volumes** . |Cliquez sur **Relancer l’analyse des volumes** pour mettre à jour la liste des volumes qui s’affiche dans le volet **Résultats**. |
| Restaurer |Développez **Catalogue de sauvegarde**, développez un groupe de volumes, développez **Instantanés locaux** ou **Instantanés Cloud**, puis cliquez avec le bouton droit sur une sauvegarde. |Cliquez sur **Restaurer** pour remplacer les données du groupe de volumes actuel par les données de la sauvegarde sélectionnée. |
| Effectuer la sauvegarde |Effectuez l’une des actions suivantes :<ul><li>Développez **Groupes de volumes**, puis cliquez avec le bouton droit sur un groupe de volumes.</li><li>Développez **Catalogue de sauvegarde**, puis cliquez avec le bouton droit sur un groupe de volumes.</li></ul> |Cliquez sur **Créer une sauvegarde** pour démarrer immédiatement une opération de sauvegarde. |
| Activer/désactiver l’affichage des importations |Cliquez avec le bouton droit sur le nœud supérieur dans le volet **Étendue** (nœud **Gestionnaire d’instantanés StorSimple** dans les exemples). |Cliquez sur **Activer/Désactiver l’affichage des importations** pour afficher ou masquer les groupes de volumes et les sauvegardes associées importés depuis le tableau de bord du service StorSimple Device Manager. |

### <a name="view-menu"></a>Menu Affichage
Utilisez le menu **Affichage** pour créer une vue personnalisée du contenu du volet **Résultats**. Le menu **Affichage** contient les options **Ajout/Suppression de colonnes** et **Personnaliser**.

#### <a name="menu-access"></a>Accès au menu
Vous pouvez accéder au menu **Affichage** sur la barre de menus et dans le volet **Actions**.

![Menu Affichage du Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_View_menu.png) 

#### <a name="menu-description"></a>Description de menu
Le tableau suivant décrit les éléments qui apparaissent dans le menu **Affichage** .

| Élément de menu | Description |
|:--- |:--- |
| Ajouter/supprimer des colonnes |Cliquez sur **Ajout/Suppression de colonnes** pour ajouter ou supprimer des colonnes dans le volet **Résultats**. |
| Personnaliser |Cliquez sur **Personnaliser** pour afficher ou masquer des éléments dans la fenêtre de console du Gestionnaire d’instantanés StorSimple. |

### <a name="favorites-menu"></a>Menu Favoris
Utilisez le menu **Favoris** pour ajouter, supprimer et organiser des vues de page et des tâches que vous utilisez fréquemment. 

#### <a name="menu-access"></a>Accès au menu
Vous pouvez accéder au menu **Favoris** dans la barre de menus.

![Menu Favoris du Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_FavoritesMenu.png)

#### <a name="menu-description"></a>Description de menu
Le tableau suivant décrit les éléments qui apparaissent dans le menu **Favoris** .

| Élément de menu | Description |
|:--- |:--- |
| Ajouter aux Favoris |Cliquez sur **Ajouter aux Favoris** pour ajouter la vue actuelle à votre liste des Favoris. |
| Organiser les Favoris |Cliquez sur **Organiser les Favoris** pour organiser le contenu de votre dossier Favoris. |

### <a name="window-menu"></a>Menu Fenêtre
Utilisez le menu **Fenêtre** pour ajouter et réorganiser les fenêtres de la console du Gestionnaire d’instantanés StorSimple.

#### <a name="menu-access"></a>Accès au menu
Vous pouvez accéder au menu **Fenêtre** dans la barre de menus.

![Menu de la Fenêtre Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_WindowMenu.png)

La liste numérotée au bas du menu affiche les fenêtres actuellement ouvertes. Cliquez sur n’importe quelle fenêtre de cette liste pour l’afficher au premier plan. 

#### <a name="menu-description"></a>Description de menu
Le tableau suivant décrit les éléments qui apparaissent dans le menu Fenêtre.

| Élément de menu | Description |
|:--- |:--- |
| Nouvelle fenêtre |Cliquez sur **Nouvelle fenêtre** pour ouvrir une nouvelle fenêtre de console (en plus de la fenêtre existante). |
| Cascade |Cliquez sur **Cascade** pour afficher les fenêtres de console ouvertes en cascade. |
| Mosaïque horizontale |Cliquez sur **Mosaïque horizontale** pour afficher les fenêtres de console ouvertes sous forme de mosaïque (ou de grille). |
| Réorganiser les icônes |Si plusieurs fenêtres de console sont ouvertes et éparpillées sur votre bureau, réduisez-les, puis cliquez sur **Réorganiser les icônes** pour les aligner horizontalement en bas de votre écran. |

### <a name="help-menu"></a>Menu aide
Utilisez le menu **Aide** pour afficher l’aide en ligne disponible du Gestionnaire d’instantanés StorSimple et de MMC. Vous pouvez également afficher les informations relatives aux logiciels MMC et Gestionnaire d’instantanés StorSimple actuellement installés sur votre système. 

Vous pouvez accéder au menu **Aide** dans la barre de menus. Vous pouvez également accéder aux Rubriques d’aide du Gestionnaire d’instantanés StorSimple à partir du volet **Actions** .

![Menu Aide du Gestionnaire d’instantanés StorSimple](./media/storsimple-use-snapshot-manager/HCS_SSM_HelpMenu.png)

#### <a name="menu-description"></a>Description de menu
Le tableau suivant décrit les éléments qui apparaissent dans le menu Aide.

| Élément de menu | Description |
|:--- |:--- |
| Aide du Gestionnaire d’instantanés StorSimple |Cliquez sur l’ **Aide sur Gestionnaire d’instantanés StorSimple** pour ouvrir l’aide du gestionnaire d’instantanés StorSimple dans une fenêtre distincte. |
| Rubriques d’aide |Cliquez sur **Rubriques d’aide** pour ouvrir l’aide en ligne de MMC dans une fenêtre distincte. |
| Site web TechCenter |Cliquez sur **Site Web TechCenter** pour ouvrir la page d’accueil Microsoft TechNet TechCenter dans une fenêtre distincte. |
| À propos de Microsoft Management Console |Cliquez sur **À propos de Microsoft Management Console** pour connaître la version de Microsoft Management Console installée sur votre système. |
| À propos du Gestionnaire d’instantanés StorSimple |Cliquez sur **À propos du Gestionnaire d’instantanés StorSimple** pour connaître la version du composant logiciel enfichable installée sur votre système. |

## <a name="tool-bar"></a>Barre d’outils
La barre d’outils située sous la barre de menus contient les icônes de navigation et des tâches. Chaque icône est un raccourci vers une tâche spécifique.

### <a name="icon-descriptions"></a>Description des icônes
Le tableau suivant décrit les icônes qui apparaissent dans la barre d’outils. 

| Icône | Description |
|:--- |:--- |
| ![Flèche gauche](./media/storsimple-use-snapshot-manager/HCS_SSM_LeftArrow.png) |Cliquez sur l’icône flèche gauche pour revenir à la page précédente. |
| ![Flèche droite](./media/storsimple-use-snapshot-manager/HCS_SSM_RightArrow.png) |Cliquez sur la flèche droite pour accéder à la page suivante (si la flèche est grisée, l’action est indisponible). |
| ![Icône Haut](./media/storsimple-use-snapshot-manager/HCS_SSM_Up.png) |Cliquez sur l’icône pour monter d’un niveau dans l’arborescence de la console (volet **Étendue** ). |
| ![Afficher/Masquer l’arborescence de console](./media/storsimple-use-snapshot-manager/HCS_SSM_ShowConsoleTree.png) |Cliquez sur l’icône d’arborescence de la console afficher/masquer pour afficher ou masquer le volet **Étendue** . |
| ![Exporter la liste](./media/storsimple-use-snapshot-manager/HCS_SSM_ExportListIcon.png) |Cliquez sur l’icône Exporter la liste pour exporter une liste vers un fichier CSV que vous indiquez. |
| ![Icône Aide](./media/storsimple-use-snapshot-manager/HCS_SSM_HelpIcon.png) |Cliquez sur l’icône d’aide pour ouvrir une rubrique d’aide en ligne MMC. |
| ![Afficher/masquer le volet Actions](./media/storsimple-use-snapshot-manager/HCS_SSM_ShowAction.png) |Cliquez sur l’icône Afficher/masquer du volet **Actions** pour afficher ou masquer le volet **Actions**. |

## <a name="scope-pane"></a>Volet Étendue
Le volet **Étendue** est le volet le plus à gauche de l’interface utilisateur StorSimple Snapshot Manager. Il contient l’arborescence de la console (ou nœud) et est le mécanisme de navigation principal du Gestionnaire d’instantanés StorSimple. 

### <a name="scope-pane-structure"></a>Structure du volet Étendue
Le volet **Étendue** contient une série d’objets interactifs (nœuds) organisée en arborescence. 

![Volet Étendue](./media/storsimple-use-snapshot-manager/HCS_SSM_Scope_pane.png) 

* Pour développer ou réduire un nœud, cliquez sur l’icône fléchée en regard du nom de nœud.
* Pour afficher l’état ou le contenu d’un nœud, cliquez sur le nom du nœud. Les informations apparaissent dans le volet **Résultats** . 

Le volet **Étendue** contient les nœuds suivants : 

* [Nœud Appareils](#devices-node) 
* [Nœud Volumes](#volumes-node) 
* [Nœud Groupes de volumes](#volume-groups-node) 
* [Nœud Stratégies de sauvegarde](#backup-policies-node) 
* [Nœud Catalogue de sauvegarde](#backup-catalog-node) 
* [Nœud Travaux](#jobs-node) 

### <a name="scope-pane-tasks"></a>Tâches du volet Étendue
Vous pouvez utiliser le volet **Étendue** pour effectuer une action sur un nœud spécifique. Pour sélectionner une tâche, effectuez l’une des opérations suivantes :

* Cliquez sur le nœud, puis sélectionnez la tâche dans le menu qui s’affiche.
* Cliquez sur le nœud, puis sur **Action** sur la barre de menus. Sélectionnez la tâche dans le menu qui s’affiche.
* Cliquez sur le nœud, puis sélectionnez l’action dans le volet **Actions** .

Lorsque vous sélectionnez un nœud et utilisez une des méthodes suivantes pour afficher une liste de tâches, uniquement les actions qui peuvent être effectuées sur ce nœud sont affichées.

### <a name="devices-node"></a>Nœud Appareils
Le nœud **Appareils** représente les appareils et les périphériques virtuels StorSimple connectés au gestionnaire d’instantanés StorSimple. Sélectionnez ce nœud pour vous connecter et configurer un appareil et importer ses volumes associés, les groupes de volumes et les copies de sauvegarde existants. Plusieurs appareils peuvent être connectés au même hôte.

* Pour développer le nœud, cliquez sur l’icône fléchée en regard de **Appareils**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Appareils** ou sur les nœuds qui s’affichent dans la vue étendue.
* Pour afficher la liste des appareils configurés, cliquez sur **Appareils** dans le volet **Étendue**. La liste des appareils, ainsi que des informations sur chaque appareil, s’affichent dans le volet **Résultats** .

### <a name="volumes-node"></a>Nœud Volumes
Le nœud **Volumes** représente les lecteurs correspondant aux volumes montés par l’hôte, y compris ceux détectés via iSCSI et ceux détectés via un appareil. Utilisez ce nœud pour afficher la liste des volumes disponibles et attribuer des volumes individuels aux groupes de volumes.

* Pour développer le nœud, cliquez sur l’icône en forme de flèche en regard de **Volumes**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Volumes** ou sur les nœuds qui s’affichent dans la vue développée.
* Pour afficher une liste des volumes, cliquez sur **Volumes** dans le volet **Étendue**. La liste des volumes ainsi que les informations relatives à chaque volume s’affichent dans le volet **Résultats** .

### <a name="volume-groups-node"></a>Nœud Groupes de volumes
Les groupes de volumes sont également appelés groupes de cohérence. Chaque groupe de volumes est un pool de volumes liés à l’application qui permet de garantir la cohérence des applications lors des opérations de sauvegarde. Utilisez le nœud **Groupes de volumes** pour configurer ces groupes et effectuer des sauvegardes interactives ou créer des planifications de sauvegarde. 

* Pour développer le nœud, cliquez sur l’icône de flèche en regard de **Groupes de volumes**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Groupes de volumes** ou sur les nœuds qui s’affichent dans la vue développée.
* Pour afficher la liste des groupes de volumes, cliquez sur **Groupes de volumes** dans le volet **Étendue**. La liste des groupes de volumes ainsi que des informations sur chaque groupe de volumes s’affichent dans le volet **Résultats** .

### <a name="backup-policies-node"></a>Nœud Stratégies de sauvegarde
Les stratégies de sauvegarde sont des planifications de tâches liées à des instantanés locaux et du cloud. Utilisez le nœud **Stratégies de sauvegarde** pour spécifier la fréquence de création des sauvegardes et la durée pendant laquelle une sauvegarde doit être conservée. 

* Pour développer le nœud, cliquez sur l’icône en forme de flèche en regard de **Stratégies de sauvegarde**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Stratégies de sauvegarde** ou sur les nœuds qui s’affichent dans la vue développée.
* Pour afficher la liste des stratégies de sauvegarde, cliquez sur **Stratégies de sauvegarde** dans le volet **Étendue**. La liste des stratégies de sauvegarde ainsi que des informations sur chaque stratégie s’affichent dans le volet **Résultats** .

> [!NOTE]
> Vous pouvez conserver un maximum de 64 sauvegardes.


### <a name="backup-catalog-node"></a>Nœud Catalogue de sauvegarde
Le nœud **Catalogue de sauvegarde** contient des listes de sauvegardes sur site et hors site des volumes Azure StorSimple. Ce nœud est organisé par groupes de volumes et chaque conteneur de groupe de volumes contient des structures séparées pour les instantanés locaux (nœud **Instantanés locaux**) et les instantanés cloud (nœud **Instantanés Cloud**). Une fois développé, chaque conteneur de groupe de volumes répertorie toutes les sauvegardes réussies effectuées de façon interactive ou via une politique configurée.

* Pour développer le nœud, cliquez sur l’icône de flèche en regard de **Catalogue de sauvegarde**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Catalogue de sauvegarde** ou sur les nœuds qui s’affichent dans la vue développée.
* Pour afficher une liste d’instantanés de sauvegarde, cliquez sur **Catalogue de sauvegarde** dans le volet **Étendue**. La liste d’instantanés ainsi que des informations sur chaque instantané s’affichent dans le volet **Résultats** .

### <a name="local-snapshots-node"></a>Nœud Instantanés locaux
Le nœud **Instantanés locaux** répertorie les instantanés locaux d’un groupe de volumes spécifique. Le nœud est situé sous le nœud **Catalogue de sauvegarde** dans le volet **Étendue**. Les instantanés locaux sont des copies ponctuelles dans le temps de données de volume stockées sur l’appareil Azure StorSimple. En règle générale, ce type de sauvegarde peut être créé et restauré rapidement. Vous pouvez utiliser un instantané local comme vous utiliseriez une copie de sauvegarde locale.

* Pour développer le nœud, cliquez sur l’icône de flèche en regard d’ **Instantanés locaux**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Instantanés locaux** ou sur l’un des nœuds qui s’affichent dans la vue développée.
* Pour afficher une liste des instantanés locaux, cliquez sur **Instantanés locaux** dans le volet **Étendue**. La liste d’instantanés ainsi que des informations sur chaque instantané s’affichent dans le volet **Résultats** .

### <a name="cloud-snapshots-node"></a>Nœud Instantanés Cloud
Le nœud **Instantanés Cloud** répertorie les instantanés cloud pour un groupe de volumes spécifique. Le nœud est situé sous le nœud **Catalogue de sauvegarde** dans le volet **Étendue**. Les instantanés cloud sont des copies ponctuelles de données de volume stockées dans le cloud. Un instantané cloud est équivalent à un instantané répliqué sur un système de stockage hors site distinct. Les instantanés cloud sont particulièrement utiles dans les scénarios de récupération d’urgence.

* Pour développer le nœud, cliquez sur l’icône en forme de flèche en regard d’ **Instantanés Cloud**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Instantanés Cloud** ou sur l’un des nœuds qui s’affichent dans la vue développée.
* Pour afficher une liste des instantanés cloud, cliquez sur **Instantanés Cloud** dans le volet **Étendue**. La liste d’instantanés ainsi que des informations sur chaque instantané s’affichent dans le volet **Résultats** .

### <a name="jobs-node"></a>Nœud Travaux
Le nœud **Travaux** contient des informations sur les travaux de sauvegarde planifiés, en cours d’exécution et récemment terminés. 

* Pour développer le nœud, cliquez sur l’icône en forme de flèche située en regard de **Tâches**.
* Pour afficher un menu d’actions disponibles, cliquez avec le bouton droit sur le nœud **Tâches** ou sur les nœuds qui s’affichent dans la vue développée.
* Pour afficher la liste des tâches planifiées, développez le nœud **Tâches**, puis cliquez sur **Planifiée**. La liste des tâches précédemment configurées et des informations relatives à chaque tâche s’affiche dans le volet **Résultats** . 
* Pour afficher la liste des tâches effectuées récemment, développez le nœud **Tâches**, puis cliquez sur **Ces dernières 24 heures**. Une liste de tâches qui ont été effectuées dans les 24 dernières heures s’affiche dans le volet **Résultats** . Le volet **Résultats** contient également des informations sur chaque tâche terminée.
* Pour afficher une liste des tâches en cours d’exécution, développez le nœud **Tâches**, puis cliquez sur **En cours d’exécution**. La liste des tâches en cours et des informations sur chaque tâche s’affichent dans le volet **Résultats** .

## <a name="results-pane"></a>Results pane
Le volet **Résultats** est le volet central de l’interface utilisateur du Gestionnaire d’instantanés StorSimple. Il contient des listes et des informations d’état détaillées sur le nœud sélectionné dans le volet **Étendue** .

### <a name="example"></a>Exemple
Pour voir l’exemple suivant, cliquez sur le nœud **Groupes de volumes** dans le volet **Étendue**. Le volet **Résultats** affiche une liste des groupes de volumes contenant des détails sur chaque groupe.

![Results pane](./media/storsimple-use-snapshot-manager/HCS_SSM_Results_pane.png) 

Vous pouvez configurer les détails affichés dans le volet **Résultats** : cliquez avec le bouton droit sur un nœud du volet **Étendue**, cliquez sur **Affichage**, puis sur **Ajout/Suppression de colonnes**.

## <a name="actions-pane"></a>Volet Actions
Le volet **Actions** volet est le volet droit de l’interface du Gestionnaire d’instantanés StorSimple. Il contient un menu d’opérations que vous pouvez effectuer sur le nœud ou afficher, ou des données que vous sélectionnez dans le volet **Étendue** ou le volet **Résultats**. Le volet **Actions** contient les mêmes commandes que les menus **Action** disponibles pour les éléments dans le volet **Étendue** et le volet **Résultats**. Pour obtenir une description de chaque action, consultez le tableau dans la section de menu **Action** .

### <a name="examples"></a>Exemples
Pour voir l’exemple suivant, dans le volet **Étendue**, développez le nœud **Tâches**, puis cliquez sur **Planifiée**. Le volet **Actions** affiche les actions disponibles pour le nœud **Planifiée**.

![Exemple de tâches planifiées dans le volet Actions](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane.png) 

Pour afficher davantage d’options, dans le volet **Étendue**, développez le nœud **Tâches**, cliquez sur **Planifiée**, puis cliquez sur une tâche planifiée dans le volet **Résultats**. Le volet **Actions** affiche les actions disponibles pour la tâche planifiée, comme indiqué dans l’exemple suivant.

![Exemple d’actions de tâche dans le volet Actions](./media/storsimple-use-snapshot-manager/HCS_SSM_ActionsPane_Results.png)

## <a name="keyboard-navigation-and-shortcuts"></a>Navigation et raccourcis clavier
Gestionnaire d’instantanés StorSimple active les fonctionnalités d’accessibilité du système d’exploitation Windows et la console Microsoft Management Console (MMC). Il inclut également des fonctionnalités de navigation et les raccourcis claviers spécifiques aux Gestionnaire d’instantanés StorSimple, comme décrit dans les sections suivantes.

* [Touches de déplacement clavier](#keyboard-navigation-keys) 
* [Touches de raccourci de la barre de menus](#menu-bar-shortcut-keys) 
* [Touches de raccourci du volet Étendue](#scope-pane-shortcut-keys) 

### <a name="keyboard-navigation-keys"></a>Touches de déplacement clavier
Le tableau suivant décrit les touches que vous pouvez utiliser pour naviguer dans l’interface utilisateur du gestionnaire d’instantanés StorSimple. 

| Touche de navigation | Action |
|:--- |:--- |
| Touche flèche bas |Utiliser la touche avec flèche vers le bas pour vous déplacer verticalement vers l’élément suivant d’un menu ou d’un volet. |
| Entrez |Appuyez sur la touche entrée pour exécuter une action, puis passez à l’étape suivante. Par exemple, vous pouvez appuyer sur Entrée pour sélectionner **Suivant**, **OK** ou **Créer**, puis passer à l’étape suivante dans l’Assistant. |
| Échap |Appuyez sur la touche ÉCHAP pour fermer un menu ou pour annuler puis fermer une page. |
| F1 |Appuyez sur la touche F1 pour afficher la rubrique d’aide correspondant à la fenêtre active. |
| F5 |Appuyez sur la touche F5 pour actualiser un nœud. |
| F6 |Appuyez sur la touche F6 pour passer du volet **Étendue** au volet **Résultats**. |
| F10 |Appuyez sur la touche F10 pour accéder à la barre de menus. |
| Touche de flèche gauche |Utilisez la touche de direction gauche pour vous déplacer horizontalement depuis une option de la barre de menu vers l’option précédente. Lorsque vous revenez à l’élément précédent de la barre de menus, le menu d’action ou (contextuel) correspondant à l’élément précédent s’affiche. |
| Touche de direction droite |Utilisez la touche de direction droite pour vous déplacer horizontalement d’une option de menu de la barre vers le suivant. Lorsque vous passez à l’élément suivant de la barre de menus, l’action ou le menu (contextuel) correspondant au nouvel élément s’affiche. |
| Touche de tabulation |Utilisez la touche Tab pour passer au volet de console suivant, à la sélection suivante ou à la zone de texte d’une page. |
| Touche de direction Haut |Utilisez la touche de direction Haut pour déplacer verticalement vers l’élément précédent dans un menu ou d’un volet. |

### <a name="menu-bar-shortcut-keys"></a>Touches de raccourci de la barre de menus
Le tableau suivant décrit les combinaisons de touches de raccourci de la barre de menus. Lorsque vous appuyez sur les touches de raccourci et que le menu s’ouvre, vous pouvez utiliser les touches de raccourci du menu (touches soulignées dans le menu). Pour plus d’informations sur la barre de menus, accédez à la [Barre de menus](#menu-bar).

| Raccourci | Résultats | Touche de raccourci du menu | Résultats |
|:--- |:--- |:--- |:--- |
| ALT + F |Ouvre le menu **Fichier** . |N |Ouvre une nouvelle instance de console. |
|  |O |Ouvre la page **Outils d’administration** . | |
|  |S |Enregistre la console du Gestionnaire d’instantanés StorSimple. | |
|  |Un |Ouvre la page **Enregistrer sous** . | |
|  |M |Ouvre la page **Ajouter/Supprimer un composant logiciel enfichable** . | |
|  |P |Ouvre la page **Options** . | |
|  |H |Ouvre l’aide en ligne. | |
| ALT + A |Ouvre le menu **Action** . |I |Active et désactive l’option d’affichage de l’importation. |
|  |W |Ouvre une nouvelle console de gestionnaire d’instantanés StorSimple. | |
|  |F |Met à jour la console du Gestionnaire d’instantanés StorSimple. | |
|  |L |Ouvre la page **Exporter la liste** . | |
|  |H |Ouvre l’aide en ligne. | |
| Alt+V |Ouvre le menu **Affichage** . |Un |Ouvre la page **Ajout/Suppression de colonnes** . |
|  |U |Ouvre la page **Personnalisation de l’affichage** . | |
| ALT + O |Ouvre le menu **Favoris** . |Un |Ouvre la page **Ajouter aux favoris** . |
|  |O |Ouvre la page **Organiser les favoris** . | |
| ALT + W |Ouvre le menu **Fenêtre** . |N |Ouvre une autre fenêtre de Gestionnaire d’instantanés StorSimple. |
|  |C |Affiche toutes les fenêtres de console ouvertes en cascade. | |
|  |T |Affiche toutes les fenêtres de console ouvertes dans une grille. | |
|  |I |Réorganise les icônes sur une rangée horizontale en bas de votre écran. | |
| ALT + H |Ouvre le menu **Aide** . |H |Ouvre l’aide en ligne. |
|  |T |Ouvre la page web Microsoft TechNet Tech Center. | |
|  |Un |Ouvre la page **À propos de Microsoft Management Console** . | |

### <a name="scope-pane-shortcut-keys"></a>Touches de raccourci du volet Étendue
Les tableaux suivants montrent les combinaisons de touches de raccourci de chaque nœud dans le volet **Étendue** . 

* [Touches de raccourci du nœud Périphériques](#devices-node-shortcut-keys)
* [Touches de raccourci du nœud Volumes](#volumes-node-shortcut-keys)
* [Touches de raccourci du nœud Groupes de volumes](#volume-groups-node-shortcut-keys)
* [Touches de raccourci du nœud Stratégies de sauvegarde](#backup-policies-node-shortcut-keys)
* [Touches de raccourci du nœud Catalogue de sauvegarde](#backup-catalog-node-shortcut-keys)
* [Touches de raccourci du nœud Tâches](#jobs-node-shortcut-keys)

#### <a name="devices-node-shortcut-keys"></a>Touches de raccourci du nœud Périphériques
| Raccourci du menu | Résultats |
|:--- |:--- |
| C |Ouvre la page **Configurer un appareil** . |
| D |Actualise la liste des appareils et les détails de l’appareil |
| V |Ouvre le menu **Affichage** . |
| W |Ouvre une nouvelle console du Gestionnaire d’instantanés StorSimple axée sur le nœud **Détails** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| L |Ouvre la page **Exporter la liste** . |
| H |Ouvre l’aide en ligne. |

#### <a name="volumes-node-shortcut-keys"></a>Touches de raccourci du nœud Volumes
| Raccourci du menu | Résultats |
|:--- |:--- |
| V |Met à jour la liste des volumes. |
| V (appuyer deux fois) |Ouvre le menu **Affichage** . |
| W |Ouvre une nouvelle console de Gestionnaire d’instantanés StorSimple axée sur le nœud **Volumes** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| L |Ouvre la page **Exporter la liste** . |
| H |Ouvre l’aide en ligne. |

#### <a name="volume-groups-node-shortcut-keys"></a>Touches de raccourci du nœud Groupes de volumes
| Raccourci du menu | Résultats |
|:--- |:--- |
| G |Ouvre la page **Créer un groupe de volumes** . |
| V |Ouvre le menu **Affichage** . |
| W |Ouvre une nouvelle console de Gestionnaire d’instantanés StorSimple axée sur le nœud **Groupes de volumes** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| L |Ouvre la page **Exporter la liste** . |
| H |Ouvre l’aide en ligne. |

#### <a name="backup-policies-node-shortcut-keys"></a>Touches de raccourci du nœud Stratégies de sauvegarde
| Raccourci du menu | Résultats |
|:--- |:--- |
| B |Ouvre la page **Créer une stratégie** . |
| V |Ouvre le menu **Affichage** . |
| W |Ouvre une nouvelle console de Gestionnaire d’instantanés StorSimple axée sur le nœud **Groupes de volumes** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| L |Ouvre la page **Exporter la liste** . |
| H |Ouvre l’aide en ligne. |

#### <a name="backup-catalog-node-shortcut-keys"></a>Touches de raccourci du nœud Catalogue de sauvegarde
| Raccourci du menu | Résultats |
|:--- |:--- |
| W |Ouvre une nouvelle console de Gestionnaire d’instantanés StorSimple axée sur le nœud **Groupes de volumes** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| H |Ouvre l’aide en ligne. |

#### <a name="jobs-node-shortcut-keys"></a>Touches de raccourci du nœud Tâches
| Raccourci du menu | Résultats |
|:--- |:--- |
| V |Ouvre le menu **Affichage** . |
| W |Ouvre une nouvelle console de Gestionnaire d’instantanés StorSimple axée sur le nœud **Tâches** . |
| F |Met à jour la console du Gestionnaire d’instantanés StorSimple. |
| L |Ouvre la page **Exporter la liste** . |
| H |Ouvre l’Aide en ligne |

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* Découvrez comment [utiliser le Gestionnaire d’instantanés StorSimple pour se connecter à des appareils et les gérer](storsimple-snapshot-manager-manage-devices.md).