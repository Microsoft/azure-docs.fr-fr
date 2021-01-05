---
title: Notes de publication de StorSimple 8000 Series Update 5
description: Décrit les nouvelles fonctionnalités, les problèmes et les solutions de contournement de StorSimple 8000 Series Update 5.
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.topic: conceptual
ms.date: 11/13/2017
ms.author: alkohli
ms.openlocfilehash: 4fed65e12a166c046a0c63d7c4849f86805bbe9c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021042"
---
# <a name="storsimple-8000-series-update-5-release-notes"></a>Notes de publication de StorSimple 8000 Series Update 5

## <a name="overview"></a>Vue d’ensemble

Les notes de publication suivantes décrivent les nouvelles fonctionnalités et identifient les problèmes critiques non résolus relatifs à StorSimple 8000 Series Update 5. Elles contiennent également une liste des mises à jour du logiciel StorSimple incluses dans cette version.

Update 5 peut être appliquée à n’importe quel appareil StorSimple exécutant toute version entre Update 0.1 et Update 4. La version d’appareil associée à Update 5 est 6.3.9600.17845.

Lisez les informations contenues dans les notes de publication avant de déployer la mise à jour dans votre solution StorSimple.

> [!IMPORTANT]
> * Update 5 est une mise à jour obligatoire et elle doit être installée immédiatement. Pour plus d’informations, consultez [Comment appliquer Update 5](storsimple-8000-install-update-5.md).
> * Update 5 contient les mises à jour logicielles de l’appareil, celles du microprogramme de disque, de la sécurité du système d’exploitation ainsi que d’autres mises à jour du système d’exploitation. L’installation de cette mise à jour prend environ 4 heures. La mise à jour du microprogramme de disque est une mise à jour perturbatrice qui entraîne une interruption de service pour votre appareil. Nous vous recommandons d’appliquer Update 5 pour maintenir votre appareil à jour.
> * Pour les nouvelles versions, vous ne voyez pas immédiatement les mises à jour, car nous effectuons un déploiement échelonné des mises à jour. Revérifiez les mises à jour dans quelques jours, car celles-ci seront bientôt disponibles.

## <a name="whats-new-in-update-5"></a>Nouveautés d’Update 5

Update 5 comporte les principaux correctifs de bogues et perfectionnements suivants.

* **Utilisation d’AAD (Azure Active Directory) pour l’authentification avec le service StorSimple Device Manager** – À partir d’Update 5, Azure Active Directory est utilisé pour s’authentifier auprès du service StorSimple Device Manager. La dépréciation de l’ancien mécanisme d’authentification est prévue d’ici décembre 2017. Tous les utilisateurs doivent inclure les nouvelles URL d’authentification dans leurs règles de pare-feu. Pour plus d’informations, reportez-vous aux [URL d’authentification répertoriées dans la configuration réseau exigée pour votre appareil StorSimple](storsimple-8000-system-requirements.md#url-patterns-for-azure-portal).

    Si l’URL d’authentification ne figure pas dans les règles de pare-feu, les utilisateurs voient une alerte critique les avertissant que leur appareil StorSimple ne peut pas s’authentifier auprès du service. Si les utilisateurs voient cette alerte, ils doivent intégrer la nouvelle URL d’authentification. Pour plus d’informations, consultez les [Alertes de réseau StorSimple](storsimple-8000-manage-alerts.md#networking-alerts).

* **Nouvelle version de StorSimple Snapshot Manager** : Une nouvelle version de StorSimple Snapshot Manager est publiée avec Update 5, compatible avec tous les appareils StorSimple qui exécutent Update 4 ou version ultérieure. Nous vous recommandons d’effectuer une mise à jour vers cette version. La version précédente de StorSimple Snapshot Manager est utilisée pour les appareils StorSimple qui exécutent Update 3 ou version antérieure. [Téléchargez la version appropriée de StorSimple Snapshot Manager](https://www.microsoft.com/en-us/download/details.aspx?id=44220) et consultez [Déployer StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md).


## <a name="issues-fixed-in-update-5"></a>Problèmes résolus dans Update 5

Le tableau suivant récapitule les problèmes qui ont été résolus dans Update 5.

| Non | Fonctionnalité | Problème | S’applique à un appareil physique | S’applique à un appareil virtuel |
| --- | --- | --- | --- | --- |
| 1 |Accès distant Windows PowerShell |Dans la version précédente, un utilisateur recevait une erreur lorsqu’il tentait d’établir une connexion à distance à StorSimple Cloud Appliance via Windows PowerShell. La cause racine de ce problème a été identifiée et il a été résolu dans cette version. |Non |Oui |
| 2 |Modèles de bande passante |Dans la version précédente, un problème de modèles de bande passante aboutissait à l’utilisation d’une bande passante inférieure à celle pour laquelle l’appareil avait été configuré. Ce problème a été résolu dans cette version. |Oui |Oui |
| 3 |Basculement |Dans la version précédente, quand un appareil doté d’un grand nombre de volumes basculait vers un autre appareil exécutant Update 4, le processus échouait lors de la tentative d’application des enregistrements de contrôle d’accès. Ce problème a été résolu dans cette version. |Oui |Oui |



## <a name="known-issues-in-update-5-from-previous-releases"></a>Problèmes connus dans Update 5 et issus des versions précédentes

Il n’existe aucun nouveau problème connu dans Update 5. Pour obtenir la liste des problèmes subsistant dans Update 5 depuis les versions précédentes, consultez les [Notes de publication d’Update 3](storsimple-update3-release-notes.md#known-issues-in-update-3).

## <a name="serial-attached-scsi-sas-controller-and-firmware-updates-in-update-5"></a>Mises à jour du microprogramme et du contrôleur SAS (Serial-Attached SCSI) dans Update 5

Cette version contient des mises à jour du microprogramme et du pilote LSI et du contrôleur SAS. Pour plus d’informations sur la procédure d’installation de ces mises à jour, consultez [Installer Update 5](storsimple-8000-install-update-5.md) sur votre appareil StorSimple.

## <a name="storsimple-cloud-appliance-updates-in-update-5"></a>Mise à jour de StorSimple Cloud Appliance dans Update 5

Cette mise à jour ne peut pas s’appliquer à l’équipement cloud StorSimple (également connu sous le nom d’appareil virtuel). De nouvelles appliances cloud doivent être créées à l’aide de l’image Update 5. Pour plus d’informations sur la création d’une appliance StorSimple Cloud Appliance, consultez [Déployer et gérer une appliance StorSimple Cloud Appliance](storsimple-8000-cloud-appliance-u2.md).

## <a name="next-step"></a>Étape suivante

Découvrez comment [installer Update 5](storsimple-8000-install-update-5.md) sur votre appareil StorSimple.

