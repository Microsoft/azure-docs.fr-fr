---
title: 'Tutoriel : Installer un appareil physique Azure FXT Edge Filer'
description: Découvrez comment déballer, monter en rack et brancher les câbles du composant physique du cache de stockage hybride Microsoft Azure FXT Edge Filer.
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: 48ee6de28bcd76d4c484b77c981062bad1a3754d
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219749"
---
# <a name="tutorial-install-azure-fxt-edge-filer"></a>Tutoriel : Installer Azure FXT Edge Filer

Ce tutoriel montre comment installer un nœud matériel pour le cache de stockage hybride Azure FXT Edge Filer. Vous devez installer au moins trois nœuds matériels pour créer un cluster Azure FXT Edge Filer.

La procédure d’installation implique le déballage et le montage en rack de l’appareil, ainsi que l’attachement du bras de gestion des câbles (CMA) et du panneau avant. Un autre tutoriel explique comment attacher les câbles réseau et raccorder l’alimentation.

L’installation d’un nœud Azure FXT Edge Filer prend environ une heure.

Ce tutoriel comprend les étapes d’installation suivantes :

> [!div class="checklist"]
>
> * Déballez l’appareil
> * Monter l’appareil dans un rack
> * Installer le panneau avant (facultatif)

## <a name="installation-prerequisites"></a>Prérequis pour l’installation

Avant de commencer, vérifiez que le centre de données et le rack que vous utiliserez disposent des éléments suivants :

* Un emplacement 1U disponible sur le rack où vous avez l’intention de monter l’appareil
* Bloc d’alimentation AC et systèmes de refroidissement qui répondent aux besoins de l’Azure FXT Edge Filer. (Pour obtenir de l’aide lors de la planification et du dimensionnement de l’installation, consultez [Caractéristiques du module d’alimentation et thermiques](fxt-specs.md#power-and-thermal-specifications).)  

  > [!NOTE]
  > Pour tirer pleinement parti des deux unités alimentation redondantes, utilisez des unités de distribution d’alimentation sur deux circuits différents lors de l’attachement de l’alimentation secteur. Pour plus d’informations, consultez [Raccorder les câbles d’alimentation](fxt-network-power.md#connect-power-cables).  

## <a name="unpack-the-hardware-node"></a>Déballer le nœud matériel

Chaque nœud Azure FXT Edge Filer est livré dans une seule boîte. Effectuez ces étapes pour déballer un appareil.

1. Placez le carton sur une surface plane et droite.

2. Vérifiez que le carton et le polystyrène ne comportent pas de trace d’impacts, de coupures, d’infiltrations d’eau ou tout autre type de dégâts. Si le carton ou le reste de l’emballage vous semble trop endommagé, ne l’ouvrez pas. Contactez le Support Microsoft pour savoir si l’appareil est en état de marche.

3. Déballez le carton. Vérifiez qu’il contient les éléments suivants :
   * 1 appareil FXT dans son boîtier
   * 2 câbles d’alimentation
   * 1 panneau avant et une clé
   * 1 assemblage du kit de rails
   * 1 bras de gestion des câbles (CMA)
   * Livret d’instructions d’installation du CMA
   * Livret d’instructions d’installation en rack
   * Livret relatif à la sécurité, à l’environnement et aux réglementations

Si vous n’avez pas reçu tous les éléments énumérés ici, contactez le revendeur de l’appareil afin d’obtenir une assistance technique.

Assurez-vous que l’appareil a eu suffisamment de temps pour atteindre la température ambiante avant de l’installer ou de le mettre sous tension. Si vous remarquez de la condensation sur une partie quelconque de l’appareil, attendez au moins 24 heures avant de l’installer.

L’étape suivante consiste à monter votre appareil en rack.

## <a name="rack-the-device"></a>Montage de l’appareil en rack

L’appareil Azure FXT Edge Filer doit être installé dans un rack standard de 19 pouces.

Le cache de stockage hybride Azure FXT Edge Filer est constitué d’au moins trois appareils Azure FXT Edge Filer. Répétez les étapes d’installation en rack pour chaque appareil qui fait partie de votre système.

### <a name="rack-install-prerequisites"></a>Prérequis pour l’installation en rack

* Avant de commencer, lisez les instructions du livret relatif à la sécurité, à l’environnement et aux réglementations fourni avec votre appareil.

  > [!NOTE]
  > Veillez à toujours être deux pour soulever le nœud, y compris quand vous l’installez en rack ou quand vous le retirez du rack.

* Identifiez le type d’installation sur rail utilisé avec le rack.
  * Pour les racks enfichables à trous carrés ou ronds, suivez les instructions relatives aux rails sans outil.
  * Pour les racks à trous filetés, suivez les instructions relatives aux rails avec outils.
  
    Pour l’installation avec outils, vous devez fournir huit vis : type 10-32, 12-24, M5 ou M6. Le diamètre des têtes de vis doit être inférieur à 10 mm (0,4 po).

### <a name="identify-the-rail-kit-contents"></a>Identifier le contenu du kit de rails

Identifiez les éléments d’installation de l’assemblage du kit de rails :

1. Deux assemblages de rails coulissants A7 Dell ReadyRails II (1)
1. Deux sangles à boucle et crochet (2)

![Dessin numéroté du contenu du kit de rails](media/fxt-install/identify-rail-kit-contents-400.png)

### <a name="rail-assembly---tool-less-rails-square-hole-or-round-hole-racks"></a>Assemblages de rails : rails sans outil (racks à trous ronds ou carrés)

Pour les racks avec trous ronds ou carrés enfichables, appliquez cette procédure pour assembler et installer les rails.

1. Positionnez les extrémités des rails gauche et droit étiquetés **FRONT** face à l’intérieur. Positionnez chaque extrémité afin qu’elle siège dans les trous sur la face avant des brides verticales. (1)

2. Alignez chaque extrémité dans les trous haut et bas du rack, dans l’espace où vous souhaitez effectuer le montage. (2)

3. Engagez l’arrière du rail jusqu’à son enclenchement complet dans la collerette verticale du rack et jusqu’à l’enclenchement du loquet. Répétez ces étapes pour positionner et enclencher l’extrémité avant sur la bride verticale du rack. (3)

> [!TIP]
> Pour retirer les rails, tirez sur le bouton de dégagement du loquet situé au milieu de l’extrémité (4) et dégagez chaque rail.

![Diagramme de l’installation et de l’enlèvement des rails sans outil, avec étapes numérotées](media/fxt-install/installing-removing-tool-less-rails-400.png)

### <a name="rail-assembly---tooled-rails-threaded-hole-racks"></a>Assemblage de rails : rails avec outils (racks à trous filetés)

Pour les racks avec trous filetés, appliquez cette procédure pour assembler et installer les rails.

1. Retirez les broches des supports de fixation avant et arrière avec un tournevis plat. (1)
1. Tirez et faites pivoter les sous-ensembles de loquets des rails afin de les retirer des supports de fixation. (2)
1. Fixez les rails droit et gauche aux brides verticales avant du rail à l’aide de deux paires de vis. (3)
1. Faites glisser les montants arrière gauche et droit vers l’avant jusqu’à ce qu’ils soient placés contre les brides verticales arrière du rail et fixez-les avec deux paires de vis. (4)

![Diagramme de l’installation et de l’enlèvement des rails avec outils, avec étapes numérotées](media/fxt-install/installing-removing-tooled-rails-400.png)

### <a name="install-the-system-in-the-rack"></a>Installer le système dans le rack

Suivez ces étapes pour monter l’appareil Azure FXT Edge Filer dans le rack.

1. Dégagez les rails coulissants internes en les faisant glisser jusqu’à ce qu’ils s’enclenchent. (1)
1. Repérez les picots du rail arrière de chaque côté de l’appareil et abaissez-les pour les emboîter dans les fentes en J des rails coulissants. (2)
1. Faites pivoter l’appareil vers le bas jusqu’à ce que tous les picots du rail se trouvent dans les fentes en J. (3)
1. Repoussez l’appareil vers l’intérieur jusqu’à ce que les leviers de verrouillage s’enclenchent.
1. Appuyez sur les deux boutons de dégagement des deux rails (4) et poussez l’appareil dans le rack.

![Diagramme d’installation du système dans un rack, avec étapes numérotées](media/fxt-install/installing-system-rack-400.png)

### <a name="remove-the-system-from-the-rack"></a>Retirer le système installé dans le rack

Pour retirer l’appareil du rack, appliquez la procédure suivante.

1. Localisez les leviers de verrouillage situés sur les côtés des rails internes (1).
2. Déverrouillez chaque levier en le faisant pivoter jusqu’à ce qu’il soit en position de dégagement (2).
3. Tenez le système par ses côtés et tirez-le vers l’avant jusqu’à aligner les picots du rail avec les fentes en J. Soulevez le système, dégagez-le et posez-le sur une surface plane (3).

![Illustration de l’enlèvement d’un système du rack, avec étapes numérotées](media/fxt-install/removing-system-rack-400.png)

### <a name="engage-the-slam-latch"></a>Engager le rabat

1. À partir de l’avant, identifiez le rabat de chaque côté du système (1).
2. Les rabats s’enclenchent automatiquement quand vous poussez le système dans le rack.

Pour libérer les rabats lors de l’enlèvement du système, tirez-les vers le haut (2).

Des vis de montage facultatives sont fournies pour fixer le système au rack lors de l’expédition ou dans d’autres environnements instables. Recherchez la vis sous chaque rabat et serrez-la avec un tournevis cruciforme n°2 (3).

![Illustration avec numéros de l’engagement et de la libération du rabat](media/fxt-install/engaging-releasing-slam-latch-400.png)

### <a name="install-the-cable-management-arm"></a>Installer le bras de gestion des câbles

Un bras de gestion des câbles (CMA) facultatif est fourni avec le FXT Edge Filer. Vous trouverez des instructions imprimées pour son installation dans l’emballage.

1. Déballez et identifiez les composants du kit de gestion des câbles :
   * Bac de CMA (1)
   * CMA (2)
   * Colliers de serrage de câbles en nylon (3)
   * Crochets d’attachement de CMA (4)
   * Câble du voyant d’état (5)

   > [!TIP]
   > Pour sécuriser le CMA lors de l’expédition dans le rack, enroulez les colliers autour du panier et du bac, et serrez-les fermement. Le fait de fixer le CMA de cette manière aidera également à sécuriser votre système dans les environnements instables.

   ![Illustration des composants du CMA](media/fxt-install/cma-kit-400.png)

2. Installez le bac du CMA.

   Le bac du CMA offre un support et agit comme un dispositif de retenue pour le CMA.

   1. Alignez et engagez chaque côté du bac avec les crochets sur les bords intérieurs des rails.
   1. Poussez le bac vers l’avant jusqu’à ce qu’il s’enclenche. (1)
   1. Pour retirer le bac, appuyez sur les boutons de la libération du loquet vers le centre et retirez le bac des crochets (2).

   ![Illustration de l’installation du bac de CMA](media/fxt-install/cma-tray-install-400.png)

3. Installez les crochets d’attachement de CMA.

   > [!NOTE]
   >
   > * Vous pouvez attacher le CMA au rail de montage gauche ou droit, selon la façon dont vous prévoyez de router les câbles à partir du système.
   > * Pour plus de commodité, montez le CMA du côté opposé à l’alimentation électrique (côté A). En cas de montage du côté B, le CMA doit être déconnecté afin de pouvoir retirer le bloc d’alimentation externe.
   > * Veillez à toujours retirer le bac avant de retirer les blocs d’alimentation.

   ![Illustration de l’installation des crochets de CMA](media/fxt-install/cma-bracket-l-r-install-400.png)

   1. Sélectionnez le crochet d’attachement de CMA correspondant au côté où vous voulez monter le CMA (côté B ou côté A).
   1. Installez le crochet d’attachement de CMA avec le marquage côté A ou côté B correspondant à l’arrière du rail coulissant.
   1. Alignez les trous sur le crochet avec les broches du rail coulissant. Poussez le crochet jusqu’à ce qu’il se mette en place.

4. Installez le CMA.

   1. À l’arrière du système, fixez le loquet situé à l’avant du CMA sur le crochet intérieur de l’assemblage coulissant jusqu’à ce que le loquet s’engage (1).
   1. Fixez l’autre loquet à l’extrémité du crochet extérieur jusqu’à ce que le loquet s’engage (2).
   1. Pour retirer le CMA, désengagez les deux loquets en appuyant sur les boutons de retrait du CMA en haut des logements de loquets interne et externe (3).

   ![Illustration de l’installation principale de CMA](media/fxt-install/cma-install-400.png)

   Vous pouvez faire pivoter le CMA hors du système afin de pouvoir y accéder et effectuer des opérations de maintenance. À l’extrémité articulée, soulevez le CMA hors du bac pour le désinstaller (1). Une fois le CMA retiré du bac, faites-le tourner pour l’enlever du système (2).

   ![Illustration montrant le CMA ouvert par rotation à des fins de maintenance](media/fxt-install/cma-swing-over-tray-400.png)

## <a name="install-the-front-bezel-optional"></a>Installer le panneau avant (facultatif)

Cette section explique comment installer et retirer le panneau avant (façade) pour le matériel Azure FXT Edge Filer.

Pour installer le panneau avant

1. Recherchez et retirez la clé du panneau, qui se trouve dans l’emballage du panneau.
1. Alignez le panneau avec l’avant du châssis, et insérez les broches situées du côté droit du panneau dans les trous situés dans la bride de montage du rack de droite du nœud.
1. Fixez l’extrémité gauche du panneau sur le châssis. Appuyez sur le panneau jusqu’à ce que le bouton situé à gauche s’enclenche.
1. Verrouillez le panneau avec la clé.

Pour retirer le panneau avant

1. Déverrouillez le panneau à l’aide de la clé.
1. Appuyez sur le bouton de libération du côté gauche et retirez l’extrémité gauche du panneau du châssis.
1. Dégagez la partie droite et retirez le panneau.

   ![Image montrant le bouton de libération à gauche du panneau, et comment l’enlever en tirant vers l’extérieur à partir du côté gauche](media/fxt-install/remove-bezel-edited-600.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez déballé et monté en rack l’appareil, continuez l’installation en attachant les câbles réseau et en raccordant l’alimentation secteur à l’appareil Azure FXT Edge Filer.

> [!div class="nextstepaction"]
> [Câbler les ports réseau et l’alimentation](fxt-network-power.md)
