---
title: Installation d’Update 4 sur un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Explique comment installer StorSimple série 8000 Update 4 sur votre appareil StorSimple série 8000.
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
ms.date: 08/02/2017
ms.author: alkohli
ms.openlocfilehash: ed6f9d58c5c54c88acf8e3a0e7fda7d2d65b8637
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514317"
---
# <a name="install-update-4-on-your-storsimple-device"></a>Installer Update 4 sur votre appareil StorSimple

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment installer Update 4 sur un appareil StorSimple exécutant une version logicielle antérieure par le biais du portail Azure et à l’aide de la méthode du correctif logiciel. La méthode du correctif logiciel est utilisée quand une passerelle est configurée sur une interface réseau différente de DATA 0 de l’appareil StorSimple et que la mise à jour porte sur une version logicielle antérieure à Update 1.

Update 4 comprend les mises à jour du logiciel de l’appareil, du microprogramme USM, du pilote et du microprogramme LSI, de Storport, de Spaceport, de sécurité du système d’exploitation et beaucoup d’autres mises à jour du système d’exploitation.  Les mises à jour du logiciel de l’appareil, du microprogramme USM, de Spaceport, de Storport et d’autres mises à jour du système d’exploitation sont des mises à jour sans interruption de service. Les mises à jour sans interruption de service ou régulières peuvent être appliquées via le portail Azure ou via la méthode du correctif logiciel. Les mises à jour du microprogramme de disque sont des mises à jour avec interruption de service et peuvent uniquement être appliquées via la méthode du correctif logiciel par le biais de l’interface Windows PowerShell de l’appareil.

> [!IMPORTANT]
> * Un ensemble de vérifications préalables manuelles et automatiques sont effectuées avant l’installation pour déterminer l’intégrité de l’appareil sur le plan de l’état du matériel et de la connectivité réseau. Ces vérifications préalables sont effectuées uniquement si vous appliquez les mises à jour à partir du portail Azure.
> * Nous vous recommandons d’installer les mises à jour du logiciel et d’autres mises à jour régulières au moyen du portail Azure. Vous devez uniquement accéder à l’interface Windows PowerShell de l’appareil (pour installer les mises à jour) en cas d’échec de la vérification de la passerelle avant la mise à jour dans le portail. Selon la version que vous mettez à jour, l’installation des mises à jour peut prendre 4 heures (ou plus). Les mises à jour en mode maintenance doivent également être installées via l’interface Windows PowerShell de l’appareil. Les mises à jour du mode de maintenance étant des mises à jour perturbatrices, elles entraînent un temps d’arrêt pour votre appareil.
> * Si vous exécutez l’outil StorSimple Snapshot Manager facultatif, veillez à le mettre à niveau vers la version Update 4 avant de mettre à jour l’appareil.


[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-4-via-the-azure-portal"></a>Installer Update 4 avec le portail Azure
Suivez la procédure ci-dessous pour mettre à jour votre appareil vers [Update 4](storsimple-update4-release-notes.md).

> [!NOTE]
> Microsoft extrait des informations de diagnostic supplémentaires de l’appareil. Par conséquent, lorsque notre équipe en charge des opérations identifie des appareils qui ont un problème, nous sommes plus à même de collecter des informations à partir de ces appareils et de diagnostiquer les problèmes. 

[!INCLUDE [storsimple-8000-install-update4-via-portal](../../includes/storsimple-8000-install-update4-via-portal.md)]

Vérifiez que votre appareil exécute **StorSimple 8000 Series Update 4 (6.3.9600.17820)** . Le paramètre **Dernière date de mise à jour** doit également être modifié.

* Vous verrez maintenant que les mises à jour en mode maintenance sont disponibles (ce message peut continuer à afficher jusqu’à 24 heures après l’installation des mises à jour). Les mises à jour en mode maintenance entraînent des temps d’arrêt de l’appareil et ne peuvent être appliquées que par le biais de l’interface Windows PowerShell de votre appareil.

* Téléchargez les mises à jour en mode maintenance à l’aide de la procédure indiquée dans [Pour télécharger des correctifs logiciels](#to-download-hotfixes) pour rechercher et télécharger KB4011837, qui installe les mises à jour du microprogramme de disque (les autres mises à jour doivent déjà être installées à ce stade). Suivez les étapes répertoriées dans [Installer et vérifier le correctif logiciel en mode Maintenance](#to-install-and-verify-maintenance-mode-hotfixes) pour installer ces mises à jour en mode maintenance.

## <a name="install-update-4-as-a-hotfix"></a>Installer Update 4 en tant que correctif logiciel
La méthode recommandée pour installer Update 4 est d’utiliser le portail Azure.

Utilisez cette procédure en cas d’échec de la vérification de la passerelle lors de la tentative d’installation des mises à jour par le biais du portail Azure. La vérification échoue car vous avez une passerelle affectée à une interface réseau différente de DATA 0 et votre appareil exécute une version logicielle antérieure à Update 1.

Les versions logicielles qui peuvent être mises à niveau à l’aide de la méthode du correctif logiciel sont les suivantes :

* Update 0.1, 0.2, 0.3
* Update 1, 1.1, 1.2
* Update 2, 2.1, 2.2
* Update 3, 3.1


La méthode du correctif logiciel implique les trois étapes suivantes :

1. Télécharger les correctifs logiciels à partir du catalogue Microsoft Update
2. Installer et vérifier les correctifs logiciels en mode Normal
3. Installer et vérifier les correctifs logiciels en mode Maintenance

#### <a name="download-updates-for-your-device"></a>Télécharger des mises à jour pour votre appareil

Vous devez télécharger et installer les correctifs logiciels suivants dans l’ordre indiqué et les dossiers suggérés :

| JSON | Ko | Description | Type de mise à jour | Durée d’installation |Installer dans le dossier|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4011839 |Mise à jour logicielle |Normal <br></br>sans interruption de service |~ 25 minutes |FirstOrderUpdate|
| 2A. |KB4011841 <br> KB4011842 |Mises à jour du pilote et du microprogramme LSI <br> Mise à jour du microprogramme USM (version 3.38) |Normal <br></br>sans interruption de service |~ 3 heures <br> (inclut 2A. + 2B. + 2C.)|SecondOrderUpdate|
| 2B. |KB3139398, KB3108381 <br> KB3205400, KB3142030 <br> KB3197873, KB3197873 <br> KB3192392, KB3153704 <br> KB3174644, KB3139914  |Package de mises à jour de sécurité du système d’exploitation <br> Télécharger Windows Server 2012 R2 |Normal <br></br>sans interruption de service |- |SecondOrderUpdate|
| 2C. |KB3210083, KB3103616 <br> KB3146621, KB3121261 <br> KB3123538 |Package de mises à jour du système d’exploitation <br> Télécharger Windows Server 2012 R2 |Normal <br></br>sans interruption de service |- |SecondOrderUpdate|

En outre, vous devez peut-être installer les mises à jour du microprogramme de disque en plus de toutes les mises à jour indiquées dans les tableaux précédents. Pour vérifier si vous avez besoin des mises à jour du microprogramme de disque, vous pouvez exécuter l’applet de commande `Get-HcsFirmwareVersion` . Si vous utilisez la version `XMGJ`, `XGEG`, `KZ50`, `F6C2`, `VR08`, `N002`, `0106` du microprogramme, vous n’avez pas besoin d’installer ces mises à jour.

| JSON | Ko | Description | Type de mise à jour | Durée d’installation | Installer dans le dossier|
| --- | --- | --- | --- | --- | --- |
| 3. |KB3121899 |Microprogramme de disque |Maintenance <br></br>Interruption de service |~ 30 minutes | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * Cette procédure ne doit être effectuée qu’une seule fois pour appliquer Update 4. Pour appliquer les mises à jour ultérieures, vous pouvez utiliser le portail Azure.
> * Si vous effectuez la mise à jour à partir d’Update 3 ou 3.1, le temps total d’installation sera d’environ 4 heures.
> * Avant d’utiliser cette procédure pour appliquer la mise à jour, vérifiez que les deux contrôleurs d’appareil sont en ligne et que tous les composants matériels sont sains.

Procédez comme suit pour télécharger et importer les correctifs logiciels.

[!INCLUDE [storsimple-install-update4-hotfix](../../includes/storsimple-install-update4-hotfix.md)]

[!INCLUDE [storsimple-install-troubleshooting](../../includes/storsimple-install-troubleshooting.md)]

## <a name="next-steps"></a>Étapes suivantes
Découvrez plus en détail la [version Update 4](storsimple-update4-release-notes.md).

