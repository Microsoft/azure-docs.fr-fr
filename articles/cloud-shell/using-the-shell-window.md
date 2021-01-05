---
title: Utilisation de la fenêtre Azure Cloud Shell | Microsoft Docs
description: Vue d’ensemble de l’utilisation de la fenêtre Azure Cloud Shell.
services: azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 04/15/2019
ms.author: damaerte
ms.openlocfilehash: 3435d958049012750e775e2a17f43f463a0cc358
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70860326"
---
# <a name="using-the-azure-cloud-shell-window"></a>Utilisation de la fenêtre Azure Cloud Shell

Ce document explique comment utiliser la fenêtre Cloud Shell.

## <a name="swap-between-bash-and-powershell-environments"></a>Basculer entre les environnements Bash et PowerShell

Utilisez le sélecteur d’environnement de la barre d’outils Cloud Shell pour basculer de Bash à PowerShell.  
![Sélectionner l’environnement](media/using-the-shell-window/env-selector.png)

## <a name="restart-cloud-shell"></a>Redémarrer Cloud Shell
Cliquez sur l’icône de redémarrage de la barre d’outils Cloud Shell pour réinitialiser l’état de la machine.  
![Redémarrer Cloud Shell](media/using-the-shell-window/restart.png)
> [!WARNING]
> Le redémarrage de Cloud Shell réinitialise l’état de la machine et tous les fichiers non conservés par votre partage de fichiers Azure sont perdus.

## <a name="change-the-text-size"></a>Modifier la taille du texte
Cliquez sur l’icône des paramètres en haut à gauche de la fenêtre, puis pointez sur l’option « Taille du texte » et sélectionnez la taille de texte voulue. Votre sélection sera conservée d’une session à l’autre.
![Taille du texte](media/using-the-shell-window/text-size.png)

## <a name="change-the-font"></a>Modifier la police
Cliquez sur l’icône des paramètres en haut à gauche de la fenêtre, puis pointez sur l’option « Police » et sélectionnez la police qui vous convient.  Votre sélection sera conservée d’une session à l’autre.
![Police](media/using-the-shell-window/text-font.png)

## <a name="upload-and-download-files"></a>Charger et télécharger des fichiers
Dans le coin supérieur gauche de la fenêtre, cliquez sur l’icône Charger/Télécharger des fichiers, puis sélectionnez Charger ou Télécharger.  
![Charger/Télécharger des fichiers](media/using-the-shell-window/uploaddownload.png)
* Pour télécharger des fichiers, utilisez la fenêtre contextuelle pour accéder au fichier sur votre ordinateur local, sélectionnez le fichier souhaité et cliquez sur le bouton « Ouvrir ».  Le fichier sera téléchargé dans le répertoire `/home/user`.
* Pour télécharger le fichier, entrez le chemin complet du fichier dans la fenêtre contextuelle (c’est-à-dire en fait un chemin sous le répertoire `/home/user` qui s’affiche par défaut), puis sélectionnez le bouton « Télécharger ».  
> [!NOTE] 
> Les noms de fichiers et les chemins d’accès complets respectent la casse dans Cloud Shell. Vérifiez soigneusement votre emploi des minuscules/majuscules lorsque vous tapez le chemin d’accès d’un fichier.

## <a name="open-another-cloud-shell-window"></a>Ouvrir une autre fenêtre Cloud Shell
Cloud Shell permet plusieurs sessions simultanées dans les onglets du navigateur en permettant à chaque session d’exister sous la forme d’un processus distinct.
Lorsque vous quittez une session, veillez à fermer chaque fenêtre de session dans la mesure où chaque processus s’exécute indépendamment sur la même machine.  
Dans le coin supérieur gauche de la fenêtre, cliquez sur l’icône Ouvrir une nouvelle session. Un nouvel onglet s’ouvre. Il affiche une autre session connectée au conteneur existant.
![Ouvrir une nouvelle session](media/using-the-shell-window/newsession.png)

## <a name="cloud-shell-editor"></a>Éditeur Cloud Shell
* Reportez-vous à la page [Utilisation de l’éditeur Azure Cloud Shell](using-cloud-shell-editor.md).

## <a name="web-preview"></a>Aperçu web
Dans le coin supérieur gauche de la fenêtre, cliquez sur l’icône d’aperçu web, sélectionnez « Configurer », puis indiquez le port que vous souhaitez ouvrir.  Sélectionnez soit « Ouvrir le port » pour ouvrir uniquement le port, ou « Ouvrir et parcourir » pour ouvrir le port et afficher un aperçu du port dans un nouvel onglet.  
![Aperçu web](media/using-the-shell-window/preview.png)  
<br>
![Configurer le port](media/using-the-shell-window/preview-configure.png)  
Dans le coin supérieur gauche de la fenêtre, sélectionnez « Aperçu du port... » pour afficher un aperçu d’un port ouvert dans un nouvel onglet. Dans le coin supérieur gauche de la fenêtre, cliquez sur l’icône d’aperçu web, sélectionnez « Fermer le port... » pour fermer le port ouvert.  
![Aperçu du port/Fermer le port](media/using-the-shell-window/preview-options.png)

## <a name="minimize--maximize-cloud-shell-window"></a>Réduire et agrandir la fenêtre Cloud Shell
Cliquez sur l’icône de réduction sur la partie supérieure droite de la fenêtre pour la masquer. Cliquez de nouveau sur l’icône Cloud Shell pour la réafficher.
Cliquez sur l’icône d’agrandissement pour agrandir la fenêtre à sa hauteur maximale. Pour revenir à la taille de fenêtre précédente, cliquez sur Restaurer.  
![Réduire ou agrandir la fenêtre](media/using-the-shell-window/minmax.png)

## <a name="copy-and-paste"></a>Copier et coller
[!INCLUDE [copy-paste](../../includes/cloud-shell-copy-paste.md)]

## <a name="resize-cloud-shell-window"></a>Redimensionner la fenêtre Cloud Shell
Cliquez et faites glisser le bord supérieur de la barre d’outils vers haut ou le bas pour redimensionner la fenêtre Cloud Shell.

## <a name="scrolling-text-display"></a>Défilement du texte
Faites défiler le texte du terminal avec la souris ou le pavé tactile.

## <a name="exit-command"></a>Commande Quitter
Cliquez sur `exit` pour mettre un terme à la session active. Ce comportement se produit par défaut au bout de 20 minutes en l’absence d’interaction.

## <a name="next-steps"></a>Étapes suivantes

[Démarrage rapide de Bash dans Cloud Shell](quickstart.md) <br>
[Démarrage rapide de PowerShell dans Cloud Shell](quickstart-powershell.md)
