---
title: Installation d’Update 0.6 sur une instance StorSimple Virtual Array | Microsoft Docs
description: Explique comment utiliser l’interface utilisateur web de StorSimple Virtual Array pour appliquer la mise à jour 0.6 à l’aide du portail Azure et de la méthode des correctifs logiciels.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/18/2017
ms.author: alkohli
ms.openlocfilehash: 4b379bb76eaf7587c53ec55972ccee3f54a55f22
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534243"
---
# <a name="install-update-06-on-your-storsimple-virtual-array"></a>Installation d’Update 0.6 sur StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Cet article décrit les étapes nécessaires pour installer Update 0.6 sur votre instance StorSimple Virtual Array à l’aide de l’interface utilisateur web locale et du portail Azure. Pour maintenir votre instance StorSimple Virtual Array à jour, vous appliquez les mises à jour ou les correctifs logiciels.

Avant d’appliquer une mise à jour, nous vous recommandons de mettre les volumes ou les partages hors connexion sur l’ordinateur hôte puis sur l’appareil. Vous réduisez ainsi toute possibilité d’altération des données. Une fois que les volumes ou partages sont hors connexion, vous devez également effectuer une sauvegarde manuelle de l’appareil.

> [!IMPORTANT]
>
> - Update 0.6 correspond à la version logicielle **10.0.10293.0** sur votre appareil. Pour plus d’informations sur les nouveautés de cette mise à jour, accédez à [Notes de publication pour Update 0.6](storsimple-virtual-array-update-06-release-notes.md).
>
> - Si vous utilisez Update 0.2 ou une version supérieure, nous vous recommandons d’installer les mises à jour au moyen du Portail Azure. Si vous utilisez Update 0.1 ou les versions logicielles GA, vous devez appliquer la méthode de correctif logiciel par le biais de l’interface utilisateur web locale pour installer Update 0.6.
>
> - N’oubliez pas que l’installation d’une mise à jour ou d’un correctif logiciel nécessite le redémarrage de votre appareil. Étant donné que StorSimple Virtual Array est un appareil à nœud unique, les E/S en cours seront interrompues et votre appareil subira des temps d’arrêt.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Si vous utilisez Update 0.2 ou une version ultérieure, nous vous recommandons d’installer les mises à jour par le biais du portail Azure. La procédure du portail oblige l’utilisateur à rechercher et télécharger les mises à jour avant de les installer. Cette procédure prend environ sept minutes. Effectuez les opérations suivantes pour installer la mise à jour ou le correctif logiciel.

[!INCLUDE [storsimple-virtual-array-install-update-via-portal](../../includes/storsimple-virtual-array-install-update-via-portal-04.md)]

Une fois l’installation terminée, accédez au service StorSimple Device Manager. Sélectionnez **Appareils**, puis sélectionnez et cliquez sur l’appareil que vous venez de mettre à jour. Accédez à **Paramètres > Gérer > Mises à jour d’appareil**. La version du logiciel doit être **10.0.10293.0**.

## <a name="use-the-local-web-ui"></a>Utilisation de l’interface utilisateur web locale

La procédure de mise à jour à l’aide de l’interface utilisateur web locale se déroule en deux étapes :

* Téléchargement de la mise à jour ou du correctif
* Installation de la mise à jour ou du correctif

### <a name="download-the-update-or-the-hotfix"></a>Téléchargement de la mise à jour ou du correctif

Procédez comme suit pour télécharger la mise à jour logicielle à partir du Catalogue Microsoft Update.

#### <a name="to-download-the-update-or-the-hotfix"></a>Pour téléchargement la mise à jour ou le correctif

1. Lancez Internet Explorer et accédez à [https://catalog.update.microsoft.com](https://catalog.update.microsoft.com).

2. Si vous utilisez le catalogue Microsoft Update pour la première fois sur cet ordinateur, cliquez sur **Installer** quand vous êtes invité à installer le module complémentaire Catalogue Microsoft Update.

3. Dans la zone de recherche du Catalogue Microsoft Update, entrez le numéro KB (Base de connaissances) du correctif que vous souhaitez télécharger. Entrez **4023268** pour Update 0.6, puis cliquez sur **Rechercher**.
   
    La liste des correctifs logiciels s’affiche, par exemple **StorSimple Virtual Array Update 0.6**.
   
    ![Rechercher dans le catalogue](./media/storsimple-virtual-array-install-update-06/download1.png)

4. Cliquez sur **Télécharger**.

5. Vous devez voir cinq fichiers à télécharger. Téléchargez chacun de ces fichiers dans un dossier. Ce dossier peut également être copié sur un partage réseau accessible à partir de l’appareil.

6. Ouvrez le dossier dans lequel se trouvent les fichiers.
    ![Fichiers dans le package](./media/storsimple-virtual-array-install-update-06/update06folder.png)

    Vous voyez :
    -  Un fichier de package autonome Microsoft `WindowsTH-KB3011067-x64`. Ce fichier est utilisé pour mettre à jour le logiciel de l’appareil.
    - Un fichier de package d’agent de surveillance de Genève `GenevaMonitoringAgentPackageInstaller`. Ce fichier est utilisé pour mettre à jour l’agent de surveillance et de diagnostics (MDS) du service. Double-cliquez sur le fichier cab. Un _.msi_ s’affiche. Sélectionnez le fichier avec le bouton droit, puis choisissez **Extraire** le fichier. Vous utiliserez le fichier _.msi_ pour mettre à jour l’agent.

        ![Extraire le fichier de mise à jour de l’agent de MDS](./media/storsimple-virtual-array-install-update-06/extract-geneva-monitoring-agent-installer.png)

        > [!IMPORTANT]
        > Vous n’avez pas besoin de mettre à jour l’agent MDS si vous exécutez StorSimple Update 0.5 (0.0.10293.0).

    - Trois fichiers qui contiennent des mises à jour de sécurité Windows critiques (`windows8.1-kb4012213-x64`,`windows8.1-kb3205400-x64` et `windows8.1-kb4019213-x64`).


### <a name="install-the-update-or-the-hotfix"></a>Installation de la mise à jour ou du correctif

Avant l'installation de la mise à jour ou du correctif, vérifiez que la mise à jour ou le correctif est téléchargé(e) localement sur votre ordinateur hôte ou accessible via un partage réseau.

Cette méthode permet d’installer les mises à jour sur un appareil exécutant les versions logicielles GA ou Update 0.1. Cette procédure prend environ 12 minutes. Effectuez les opérations suivantes pour installer la mise à jour ou le correctif logiciel.

#### <a name="to-install-the-update-or-the-hotfix"></a>Pour installer la mise à jour ou le correctif

1. Dans l’interface utilisateur web locale, accédez à **Maintenance** > **Mise à jour logicielle**. Prenez note de la version du logiciel que vous exécutez. Si vous exécutez la version **10.0.10290.0**, vous n’avez pas besoin de mettre à jour l’agent MDS à l’étape 6.
   
    ![Capture d’écran montrant les Mises à jour logicielles sélectionnées dans le menu Maintenance.](./media/storsimple-virtual-array-install-update-05/update1m.png)

2. Dans le **chemin d’accès au fichier de mise à jour**, entrez le nom du fichier de mise à jour ou de correctif. Vous pouvez également accéder au fichier d'installation de la mise à jour ou du correctif si celui-ci est placé sur un partage réseau. Cliquez sur **Appliquer**.
   
    ![Capture d’écran montrant la zone de texte Mise à jour du chemin du fichier sur la page Mise à jour logicielle.](./media/storsimple-virtual-array-install-update-05/update2m.png)

3. Un avertissement s’affiche. Puisque la baie virtuelle est un appareil à nœud unique, l’application de la mise à jour entraîne le redémarrage de l’appareil et provoque un temps d’arrêt. Cliquez sur l’icône en forme de coche.
   
   ![Capture d’écran montrant une boîte de dialogue avertissant d’un temps d’arrêt.](./media/storsimple-virtual-array-install-update-05/update3m.png)

4. La mise à jour démarre. Une fois l’appareil correctement mis à jour, il est redémarré. L’interface utilisateur locale n’est pas accessible pendant cet intervalle.
   
    ![Capture d’écran montrant un message confirmant la réussite de la mise à jour.](./media/storsimple-virtual-array-install-update-05/update5m.png)

5. Une fois le redémarrage effectué, vous êtes redirigé vers la page **Se connecter** . Pour vérifier que le logiciel de l’appareil a été mis à jour, accédez à **Maintenance** > **Mise à jour logicielle** dans l’interface utilisateur web locale. La version du logiciel doit être **10.0.0.0.0.10293** pour Update 0.6.
   
   > [!NOTE]
   > L’affichage des versions logicielles diffère légèrement entre l’interface utilisateur web locale et le portail Azure. Par exemple, la même version est signalée par **10.0.0.0.0.10293** dans l’interface utilisateur web locale et par **10.0.10293.0** sur le portail Azure.
   
    ![Capture d’écran montrant la page des mises à jour logicielles avec la version actuelle du logiciel.](./media/storsimple-virtual-array-install-update-06/update6m.png)

6. Ignorez cette étape si vous exécutiez StorSimple Virtual Array Update 0.5 (**10.0.10290.0**) avant d’appliquer cette mise à jour. Vous avez noté la version du logiciel à l’étape 1 avant de commencer la mettre à jour. Si vous exécutiez Update 0.5, votre agent MDS est déjà à jour.

    Si vous exécutez une version logicielle antérieure à Update 0.5, l’étape suivante consiste à mettre à jour l’agent MDS. Sur la page **Mise à jour logicielle**, accédez au **Chemin d’accès du fichier de mise à jour** et accédez au fichier `GenevaMonitoringAgentPackageInstaller.msi`. Répétez les étapes 2 à 4. Après le redémarrage de la baie virtuelle, connectez-vous à l’interface utilisateur web locale.

7. Répétez les étapes 2 à 4 pour installer les correctifs de sécurité Windows à l’aide des fichiers `windows8.1-kb4012213-x64`,`windows8.1-kb3205400-x64` et `windows8.1-kb4019213-x64`. La baie virtuelle redémarre après chaque installation, et vous devez vous connecter à l’interface utilisateur web locale.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [gestion de votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).

