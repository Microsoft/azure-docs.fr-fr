---
title: Installer Update 1.1 sur StorSimple Virtual Array | Microsoft Docs
description: Décrit comment appliquer des mises à jour à l’aide du portail Azure et de l’interface utilisateur web locale pour StorSimple Virtual Array
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/18/2018
ms.author: alkohli
ms.openlocfilehash: 709794dcccf9b100cbb11381ac34cfb86238b22b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94533989"
---
# <a name="install-update-11-on-your-storsimple-virtual-array"></a>Installer Update 1.1 sur StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Cet article décrit les étapes nécessaires pour installer Update 1.1 sur votre instance StorSimple Virtual Array à l’aide de l’interface utilisateur web locale et du portail Azure.

Pour maintenir votre instance StorSimple Virtual Array à jour, vous appliquez les mises à jour ou les correctifs logiciels. Avant d’appliquer une mise à jour, nous vous recommandons de mettre les volumes ou les partages hors connexion sur l’ordinateur hôte puis sur l’appareil. Vous réduisez ainsi toute possibilité d’altération des données. Une fois que les volumes ou partages sont hors connexion, vous devez également effectuer une sauvegarde manuelle de l’appareil.

> [!IMPORTANT]
> - Update 1.1 correspond à la version logicielle **10.0.10307.0** sur votre appareil. Pour plus d’informations sur les nouveautés de cette mise à jour, consultez [Notes de version pour Update 1.1](storsimple-virtual-array-update-11-release-notes.md).
>
> - N’oubliez pas que l’installation d’une mise à jour ou d’un correctif logiciel nécessite le redémarrage de votre appareil. Étant donné que StorSimple Virtual Array est un appareil à nœud unique, les E/S en cours seront interrompues et votre appareil subira des temps d’arrêt.
>
> - Update 1.1 est disponible dans le portail Azure uniquement si le groupe virtuel exécute la version Update 1. Si votre groupe virtuel exécute une version antérieure à Update 0.6, vous devez installer Update 1.0 avant d’installer Update 1.1.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Si vous utilisez Update 0.2 ou une version ultérieure, nous vous recommandons d’installer les mises à jour par le biais du portail Azure. La procédure du portail oblige l’utilisateur à rechercher et télécharger les mises à jour avant de les installer. En fonction de la version logicielle exécutée par votre groupe virtuel, le processus d’application de la mise à jour via le portail Azure diffère.

 - Si votre groupe virtuel exécute Update 1, le portail Azure installe directement Update 1.1 (10.0.10307.0) sur votre appareil. Cette procédure prend environ dix minutes.
 - Si votre groupe virtuel exécute une version antérieure à Update 0.6, la mise à jour s’effectue en deux étapes. Le portail Azure installe d’abord Update 1.0 (10.0.10296.0) sur votre appareil. Après le redémarrage du groupe virtuel, le portail installe ensuite Update 1.1 (10.0.10307.0) sur votre appareil. Cette procédure prend environ 15 minutes.


[!INCLUDE [storsimple-virtual-array-install-update-via-portal](../../includes/storsimple-virtual-array-install-update-via-portal-11.md)]

Une fois l’installation terminée, accédez au service StorSimple Device Manager. Sélectionnez **Appareils**, puis sélectionnez et cliquez sur l’appareil que vous venez de mettre à jour. Accédez à **Paramètres > Gérer > Mises à jour d’appareil**. La version logicielle doit être **10.0.10307.0**.

![Version logicielle après la mise à jour](./media/storsimple-virtual-array-install-update-11/azupdate17m2.png)

## <a name="use-the-local-web-ui"></a>Utilisation de l’interface utilisateur web locale

La procédure de mise à jour à l’aide de l’interface utilisateur web locale se déroule en deux étapes :

* Téléchargement de la mise à jour ou du correctif
* Installation de la mise à jour ou du correctif

> [!IMPORTANT] 
> **Effectuez cette mise à jour uniquement si vous exécutez la version Update 1 (10.0.10296.0). Si vous exécutez Update 0.6, [installez Update 1](storsimple-virtual-array-install-update-1.md) sur votre appareil avant d’appliquer Update 1.1.**

### <a name="download-the-update-or-the-hotfix"></a>Téléchargement de la mise à jour ou du correctif

Procédez comme suit pour télécharger Update 1.1 à partir du Catalogue Microsoft Update.

#### <a name="to-download-the-update-or-the-hotfix"></a>Pour téléchargement la mise à jour ou le correctif

1. Lancez Internet Explorer et accédez à [https://catalog.update.microsoft.com](https://catalog.update.microsoft.com).

2. Si vous utilisez le catalogue Microsoft Update pour la première fois sur cet ordinateur, cliquez sur **Installer** quand vous êtes invité à installer le module complémentaire Catalogue Microsoft Update.

3. Dans la zone de recherche du Catalogue Microsoft Update, entrez le numéro KB (Base de connaissances) du correctif que vous souhaitez télécharger. Entrez **4337628** pour Update 1.1, puis cliquez sur **Rechercher**.
   
    La liste des correctifs logiciels s’affiche, par exemple **StorSimple Virtual Array Update 1.1**.
   
    ![Rechercher dans le catalogue](./media/storsimple-virtual-array-install-update-11/download1.png)

4. Cliquez sur **Télécharger**.

5. Téléchargez les deux fichiers dans un dossier. Vous pouvez également copier le dossier sur un partage réseau accessible à partir de l’appareil.

6. Ouvrez le dossier dans lequel se trouvent les fichiers.

    ![Fichiers dans le package](./media/storsimple-virtual-array-install-update-11/update01folder.png)

    Le package contient deux fichiers :
    -  Un fichier de package autonome Microsoft `WindowsTH-KB3011067-x64`. Ce fichier est utilisé pour mettre à jour le logiciel de l’appareil.
    - Un fichier contenant les mises à jour cumulatives de juin `Windows8.1-KB4284815-x64`. Pour plus d’informations sur le contenu de ce correctif cumulatif, consultez [Correctif cumulatif mensuel du mois de juin](https://support.microsoft.com/help/4284815/windows-81-update-kb4284815).

### <a name="install-the-update-or-the-hotfix"></a>Installation de la mise à jour ou du correctif

Avant d’installer la mise à jour ou le correctif logiciel, vérifiez les points suivants :

 - La mise à jour ou le correctif logiciel est téléchargé localement sur votre hôte ou accessible via un partage réseau.
 - Votre groupe virtuel exécute Update 1 (10.0.10296.0). S’il exécute Update 0.6, [installez Update 1](storsimple-virtual-array-install-update-1.md) avant d’installer Update 1.1.

Cette procédure prend environ quatre minutes. Effectuez les opérations suivantes pour installer la mise à jour ou le correctif logiciel.

#### <a name="to-install-the-update-or-the-hotfix"></a>Pour installer la mise à jour ou le correctif

1. Dans l’interface utilisateur web locale, accédez à **Maintenance** > **Mise à jour logicielle**. Prenez note de la version du logiciel que vous exécutez. **Effectuez cette mise à jour uniquement si vous exécutez la version Update 1 (10.0.10296.0). Si vous exécutez Update 0.6, [installez Update 1](storsimple-virtual-array-install-update-1.md) sur votre appareil avant d’appliquer Update 1.1.**
   
    ![Capture d’écran montrant les Mises à jour logicielles sélectionnées dans le menu Maintenance.](./media/storsimple-virtual-array-install-update-11/update1m.png)

2. Dans le **chemin d’accès au fichier de mise à jour**, entrez le nom du fichier de mise à jour ou de correctif. Vous pouvez également accéder au fichier d'installation de la mise à jour ou du correctif si celui-ci est placé sur un partage réseau. Cliquez sur **Appliquer**.
   
    ![Capture d’écran montrant la zone de texte Mise à jour du chemin du fichier sur la page Mise à jour logicielle.](./media/storsimple-virtual-array-install-update-11/update2m.png)

3. Un avertissement s’affiche. Puisque la baie virtuelle est un appareil à nœud unique, l’application de la mise à jour entraîne le redémarrage de l’appareil et provoque un temps d’arrêt. Cliquez sur l’icône en forme de coche.
   
   ![Capture d’écran montrant une boîte de dialogue avertissant d’un temps d’arrêt.](./media/storsimple-virtual-array-install-update-11/update3m.png)

4. La mise à jour démarre. Une fois l’appareil correctement mis à jour, il est redémarré. L’interface utilisateur locale n’est pas accessible pendant cet intervalle.
   
    ![Capture d’écran montrant un message confirmant la réussite de la mise à jour.](./media/storsimple-virtual-array-install-update-11/update5m.png)

5. Une fois le redémarrage effectué, vous êtes redirigé vers la page **Se connecter** . Pour vérifier que le logiciel de l’appareil a été mis à jour, accédez à **Maintenance** > **Mise à jour logicielle** dans l’interface utilisateur web locale. La version logicielle doit être **10.0.0.0.0.10307** pour Update 1.1.
   
   > [!NOTE]
   > L’affichage des versions logicielles diffère légèrement entre l’interface utilisateur web locale et le portail Azure. Par exemple, la même version s’affiche comme **10.0.0.0.0.10307** dans l’interface utilisateur web locale et comme **10.0.10307.0** dans le portail Azure.
   
    ![Capture d’écran montrant la page des mises à jour logicielles avec la version actuelle du logiciel.](./media/storsimple-virtual-array-install-update-11/update6m.png)

6. Répétez les étapes 2 à 4 pour installer le correctif de sécurité Windows à l’aide du fichier `Windows8.1-KB4284815-x64`. Le groupe virtuel redémarre après l’installation. Vous devez ensuite vous connecter à l’interface utilisateur web locale.


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [gestion de votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).
