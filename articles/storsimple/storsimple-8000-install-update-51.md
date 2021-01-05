---
title: Installer Update 5.1 sur un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Explique comment installer Update 5.1 pour la gamme d’appareils StorSimple 8000 sur votre appareil de la gamme StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 03/05/2020
ms.author: alkohli
ms.openlocfilehash: 5b9958f3dd497aa612a92947b8d968439ef9d0e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91575960"
---
# <a name="install-update-51-on-your-storsimple-device"></a>Installer Update 5.1 sur votre appareil StorSimple

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel explique comment installer Update 5.1 sur un appareil StorSimple exécutant une version logicielle antérieure par le biais du portail Azure. <!--The hotfix method is used when you are trying to install Update 5.1 on a device running pre-Update 3 versions. The hotfix method is also used when a gateway is configured on a network interface other than DATA 0 of the StorSimple device and you are trying to update from a pre-Update 1 software version.-->

La mise à jour 5.1 comprend des mises à jour de sécurité sans interruption de service. Les mises à jour sans interruption de service ou régulières peuvent être appliquées via le portail Azure <!--or by the hotfix method-->.

> [!IMPORTANT]
>
> * Update 5.1 est une mise à jour obligatoire à installer immédiatement. Pour plus d’informations, consultez [Notes de version pour Update 5.1](storsimple-update51-release-notes.md).
> * Un ensemble de vérifications préalables manuelles et automatiques sont effectuées avant l’installation pour déterminer l’intégrité de l’appareil sur le plan de l’état du matériel et de la connectivité réseau. Ces vérifications préalables sont effectuées uniquement si vous appliquez les mises à jour à partir du portail Azure.
> * Si vous souhaitez effectuer une installation à l’aide de la méthode du correctif logiciel, contactez le [support Microsoft](mailto:support@microsoft.com).

<!--
> * We strongly recommend that when updating a device running versions prior to Update 3, you install the updates using hotfix method. If you encounter any issues, [log a support ticket](storsimple-8000-contact-microsoft-support.md). 


> * We recommend that you install the software and other regular updates via the Azure portal. You should only go to the Windows PowerShell interface of the device (to install updates) if the pre-update gateway check fails in the portal. Depending upon the version you are updating from, the updates may take 4 hours (or greater) to install. The maintenance mode updates must be installed through the Windows PowerShell interface of the device. As maintenance mode updates are disruptive updates, these result in a down time for your device.


> * If running the optional StorSimple Snapshot Manager, ensure that you have upgraded your Snapshot Manager version to Update 5.1 prior to updating the device.
-->

[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-51-through-the-azure-portal"></a>Installer Update 5.1 à l’aide du portail Azure

Suivez la procédure ci-dessous pour mettre à jour votre appareil vers [Update 5.1](storsimple-update51-release-notes.md).

> [!NOTE]
> Microsoft extrait des informations de diagnostic supplémentaires de l’appareil. Par conséquent, lorsque notre équipe en charge des opérations identifie des appareils qui ont un problème, nous sommes plus à même de collecter des informations à partir de ces appareils et de diagnostiquer les problèmes.

#### <a name="to-install-an-update-from-the-azure-portal"></a>Pour installer une mise à jour à partir du portail Azure

1. Dans la page du service StorSimple, sélectionnez votre appareil.

    ![Sélectionnez l’appareil](./media/storsimple-8000-install-update-51/update1.png)

2. Accédez à **Paramètres de l’appareil** > **Mises à jour de l’appareil**.

    ![Capture d’écran du panneau Paramètres avec l’option Mises à jour de l’appareil en évidence.](./media/storsimple-8000-install-update-51/update2.png)

3. Une notification s’affiche si de nouvelles mises à jour sont disponibles. Dans le panneau **Mises à jour de l’appareil**, vous pouvez également cliquer sur **Rechercher les mises à jour**. Une tâche est créée pour rechercher les mises à jour disponibles. Un message s’affiche lorsque la tâche est terminée.

    ![Capture d’écran du panneau Paramètres avec l’option Mises à jour de l’appareil en évidence et le panneau Mises à jour de l’appareil avec le message De nouvelles mises à jour régulières sont disponibles en évidence.](./media/storsimple-8000-install-update-51/update3.png)

4. Nous vous recommandons de consulter les notes de publication avant d’appliquer une mise à jour sur votre appareil. Pour appliquer les mises à jour, cliquez sur **Installer les mises à jour**. Dans le panneau **Confirmer les mises à jour standard**, passez en revue les conditions préalables à remplir avant d’appliquer les mises à jour. Cochez la case pour indiquer que vous êtes prêt à mettre à jour l’appareil, puis cliquez sur **Installer**.

    ![Capture d’écran du panneau Mises à jour de l’appareil avec l’option Installer les mises à jour en évidence et l’option Confirmer les mises à jour standard avec les options Accepter et Installer en évidence.](./media/storsimple-8000-install-update-51/update4.png)

5. Un ensemble de vérifications préalables démarre. Ces vérifications incluent :
   
   * **Contrôles d’intégrité des contrôleurs** , destinés à vérifier que les contrôleurs d’appareil sont en bon état et en ligne.
   * **Contrôles d’intégrité des composants matériels** , utilisés pour vérifier l’état des composants matériels de votre appareil StorSimple.
   * **Contrôles DATA 0** , afin de s’assurer de l’activation de DATA 0 sur votre appareil. Si cette interface n’est pas activée, vous devez l’activer, puis réessayer.

     La mise à jour est téléchargée et installée uniquement si toutes les vérifications sont effectuées avec succès. Un message s’affiche lorsque les vérifications sont en cours. Si les pré-vérifications échouent, les causes de cet échec vous sont communiquées. Résolvez ces problèmes, puis retentez l’opération. Vous devrez peut-être contacter le support technique Microsoft si vous ne pouvez pas résoudre ces problèmes vous-même.

7. Une fois les vérifications préalables terminées, une tâche de mise à jour est créée. Un message s’affiche une fois la tâche de mise à jour créée.
   
    ![Capture d’écran de la notification indiquant « Démarrage du travail de mise à jour logicielle ».](./media/storsimple-8000-install-update-51/update6.png)
   
    La mise à jour est ensuite appliquée à votre appareil.

9. La mise à jour prend quelques heures. Sélectionnez la tâche de mise à jour et cliquez sur **Détails** pour afficher les détails de la tâche à tout moment.

    ![Capture d’écran du panneau Mises à jour de l’appareil avec l’option Télécharger et installer les mises à jour logicielles en cours en évidence et le panneau Installer les mises à jour.](./media/storsimple-8000-install-update-51/update8.png)

     Vous pouvez également surveiller la progression de la tâche de mise à jour à partir de **Paramètres de l’appareil > Travaux**. Le panneau **Travaux** indique la progression de la mise à jour.

     ![Capture d’écran du panneau Paramètres avec Travaux en évidence et le panneau Travaux montrant la progression de la mise à jour.](./media/storsimple-8000-install-update-51/update7.png)

10. Une fois la tâche terminée, accédez à **Paramètres de l’appareil > Mises à jour de l’appareil**. La version du logiciel doit maintenant être mise à jour.


Vérifiez que votre appareil exécute **Update 5.1 (6.3.9600.17885) pour la gamme d’appareils StorSimple 8000**. Le paramètre **Dernière date de mise à jour** doit être modifié.
<!-- 5.1 - KB 4542887-->

<!--You will now see that the Maintenance mode updates are available (this message might continue to be displayed for up to 24 hours after you install the updates). The steps to install maintenance mode update are detailed in the next section.

[!INCLUDE [storsimple-8000-install-maintenance-mode-updates](../../includes/storsimple-8000-install-maintenance-mode-updates.md)]

## Install Update 5.1 as a hotfix

The software versions that can be upgraded using the hotfix method are:

* Update 0.1, 0.2, 0.3
* Update 1, 1.1, 1.2
* Update 2, 2.1, 2.2
* Update 3, 3.1
* Update 4
* Update 5

> [!NOTE]
> The recommended method to install Update 5.1 is through the Azure portal when trying to update from Update 3 and later version. When updating a device running versions prior to Update 3, use this procedure. You can also use this procedure if you fail the gateway check when trying to install the updates through the Azure portal. The check fails when you have a gateway assigned to a non-DATA 0 network interface and your device is running a software version earlier than Update 1.

The hotfix method involves the following three steps:

1. Download the hotfixes from the Microsoft Update Catalog.
2. Install and verify the regular mode hotfixes.
3. Install and verify the maintenance mode hotfix.

#### Download updates for your device

You must download and install the following hotfixes in the prescribed order and the suggested folders:

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4037264 |Software update<br> Download both _HcsSoftwareUpdate.exe_ and _CisMSDAgent.exe_ |Regular <br></br>Non-disruptive |~ 25 mins |FirstOrderUpdate|

If updating from a device running Update 4, you only need to install the OS cumulative updates as second order updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2A. |KB4025336 |OS cumulative updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|

If installing from a device running Update 3 or earlier, install the following in addition to the cumulative updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2B. |KB4011841 <br> KB4011842 |LSI driver and firmware updates <br> USM firmware update (version 3.38) |Regular <br></br>Non-disruptive |~ 3 hrs <br> (includes 2A. + 2B. + 2C.)|SecondOrderUpdate|
| 2C. |KB3139398 <br> KB3142030 <br> KB3108381 <br> KB3153704 <br> KB3174644 <br> KB3139914   |OS security updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|
| 2D. |KB3146621 <br> KB3103616 <br> KB3121261 <br> KB3123538 |OS updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|



You may also need to install disk firmware updates on top of all the updates shown in the preceding tables. You can verify whether you need the disk firmware updates by running the `Get-HcsFirmwareVersion` cmdlet. If you are running these firmware versions: `XMGJ`, `XGEG`, `KZ50`, `F6C2`, `VR08`, `N003`, `0107`, then you do not need to install these updates.

| Order | KB | Description | Update type | Install time | Install in folder|
| --- | --- | --- | --- | --- | --- |
| 3. |KB4037263 |Disk firmware |Maintenance <br></br>Disruptive |~ 30 mins | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * If updating from Update 4, the total install time is close to 4 hours.
> * Before using this procedure to apply the update, make sure that both the device controllers are online and all the hardware components are healthy.

Perform the following steps to download and install the hotfixes.

[!INCLUDE [storsimple-install-update5-hotfix](../../includes/storsimple-install-update5-hotfix.md)]
-->
<!--
[!INCLUDE [storsimple-8000-install-troubleshooting](../../includes/storsimple-8000-install-troubleshooting.md)]
-->

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail la [version Update 5.1](storsimple-update51-release-notes.md).
