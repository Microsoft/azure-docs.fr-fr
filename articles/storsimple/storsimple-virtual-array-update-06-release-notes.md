---
title: Notes de publication de StorSimple Virtual Array Update 0.6 | Microsoft Docs
description: Décrit les problèmes critiques non résolus et les solutions possibles pour StorSimple Virtual Array exécutant Update 0.6.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/24/2017
ms.author: alkohli
ms.openlocfilehash: 24bddb47a32a7b8ad4ea659d52c2d5b2730be9ad
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000910"
---
# <a name="storsimple-virtual-array-update-06-release-notes"></a>Notes de publication de StorSimple Virtual Array Update 0.6

## <a name="overview"></a>Vue d’ensemble

Les notes suivantes identifient les problèmes majeurs existants et les problèmes résolus dans les mises à jour Microsoft Azure StorSimple Virtual Array.

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez les informations contenues dans les notes de publication avant de déployer votre instance StorSimple Virtual Array.

Update 0.6 correspond à la version logicielle **10.0.10293.0**.

> [!IMPORTANT]
> - Les mises à jour entraînent des perturbations et redémarrent votre appareil. Si des E/S sont en cours, l’appareil subit des interruptions de service. Pour obtenir des instructions détaillées sur la façon d’appliquer la mise à jour, consultez la page relative à [l’installation d’Update 0.6](storsimple-virtual-array-install-update-06.md).
>
> - Nous recommandons vivement d’installer Update 0.6 immédiatement, car cette mise à jour contient des correctifs de sécurité critiques.


## <a name="whats-new-in-the-update-06"></a>Nouveautés d’Update 0.6
Update 0.6 est une mise à jour critique et doit être déployée immédiatement. Cette mise à jour contient les correctifs suivants : 

- **Correctifs de sécurité Windows** : cette version Release a des **correctifs de sécurité critiques Windows**. Pour plus d’informations sur les problèmes de sécurité et les correctifs associés, passez en revue les mises à jour de sécurité suivantes :
    - [Mise à jour qualité de la sécurité seule de décembre 2016 pour Windows 8.1 et Windows Server 2012 R2](https://support.microsoft.com/help/3205400/december-2016-security-only-quality-update-for-windows-8.1-and-windows-server-2012-r2)
    - [Mise à jour qualité de la sécurité seule de mars 2017 pour Windows 8.1 et Windows Server 2012 R2](https://support.microsoft.com/help/4012213/march-2017-security-only-quality-update-for-windows-8-1-and-windows-server-2012-r23)
    - [9 mai 2017 : KB4019213 (mise à jour de la sécurité seule)](https://support.microsoft.com/help/4019213/windows-8-update-kb4019213)

- **Correctif pour la restauration** : dans les versions Release antérieures, un bogue empêchait le bon déroulement de la restauration. Ce bogue a été résolu dans cette version.


## <a name="issues-fixed-in-the-update-06"></a>Problèmes résolus dans Update 0.6

Le tableau suivant récapitule les problèmes corrigés dans cette version.

| Non. | Fonctionnalité | Problème |
| --- | --- | --- |
| 1 |Sécurité| Cette version Release contient des mises à jour de sécurité Windows critiques. Nous vous recommandons d’installer cette mise à jour immédiatement.|
| 2 |Restaurer| Pendant une restauration, une condition de concurrence empêchait le bon déroulement du travail de restauration. La résolution de bogue répond à cette condition de concurrence.|


## <a name="known-issues-in-the-update-06"></a>Problèmes connus dans Update 0.6

Le tableau suivant récapitule les problèmes connus pour StorSimple Virtual Array et décrit les problèmes signalés dans les notes de version des versions précédentes.

| Non. | Fonctionnalité | Problème | Solution de contournement/commentaires |
| --- | --- | --- | --- |
| **1.** |Mises à jour |Les appareils virtuels créés dans la version préliminaire ne peuvent pas être mis à jour vers une version à disponibilité générale prise en charge. |Ces appareils virtuels doivent être basculés vers la version à disponibilité générale à l'aide d'un flux de travail de récupération d'urgence (DR). |
| **2.** |Disque de données configuré |Une fois que vous avez configuré un disque de données d'une certaine spécifiée et créé l'appareil virtuel StorSimple correspondant, vous ne devez pas développer, ni réduire le disque de données. Toute tentative de ce type entraîne la perte de toutes les données aux niveaux locaux de l’appareil. | |
| **3.** |Stratégie de groupe |Lorsqu’un appareil est joint à un domaine, le fait d’appliquer une stratégie de groupe peut avoir un impact négatif sur son fonctionnement. |Assurez-vous que votre tableau virtuel est dans sa propre unité organisationnelle (UO) pour Active Directory et qu'aucun objet de stratégie de groupe (GPO) ne lui est appliqué. |
| **4.** |Interface utilisateur web locale |Si les fonctionnalités de sécurité améliorées sont activées dans Internet Explorer (IE ESC), certaines pages de l’interface utilisateur web locale, comme Dépannage ou Maintenance, peuvent ne pas fonctionner correctement. Les boutons sur ces pages peuvent également ne pas fonctionner. |Désactivez les fonctionnalités de sécurité améliorées d'Internet Explorer. |
| **5.** |Interface utilisateur web locale |Sur une machine virtuelle Hyper-V, les interfaces réseau de l'interface utilisateur web sont affichées sous forme d'interfaces 10 Gbits/s. |Ce comportement est le reflet de Hyper-V. Hyper-V affiche toujours 10 Gbits/s pour les cartes de réseau virtuel. |
| **6.** |Partages ou volumes à plusieurs niveaux |Le verrouillage de la plage d'octets pour les applications qui fonctionnent avec les volumes à plusieurs niveaux StorSimple n'est pas pris en charge. Si le verrouillage de la plage d’octets est activé, la hiérarchisation StorSimple ne fonctionne pas. |Mesures recommandées : <br></br>Désactivez le verrouillage de plage d'octets dans la logique de votre application.<br></br>Choisissez de placer les données de cette application dans des volumes épinglés localement par opposition à des volumes à plusieurs niveaux.<br></br>*Inconvénient*: quand le verrouillage de la plage d’octets est activé avec l’utilisation de volumes localement épinglés, le volume localement épinglé peut être en ligne avant même que la restauration ne soit terminée. Dans ce cas, si une restauration est en cours, vous devez attendre que l'opération se termine. |
| **7.** |Partages à plusieurs niveaux |L'utilisation de fichiers volumineux peut entraîner montée en charge de niveau lente. |Lorsque vous utilisez des fichiers volumineux, vérifiez que la taille du plus grand fichier est inférieure à 3 % de la taille du partage. |
| **8.** |Capacité utilisée pour les partages |Vous pouvez constater la consommation du partage en l’absence de données sur le partage. Cette consommation est due au fait que la capacité utilisée pour les partages inclut des métadonnées. | |
| **9.** |Récupération d'urgence |La récupération d'urgence d'un serveur de fichiers ne peut s'effectuer que dans le même domaine que celui de l'appareil source. La récupération d'urgence vers un appareil cible dans un autre domaine n'est pas prise en charge dans cette version. |Ceci est implémenté dans une version ultérieure. Pour plus d’informations, accédez à la page [Basculement d’appareil et récupération d’urgence pour votre StorSimple Virtual Array via le portail Azure](storsimple-virtual-array-failover-dr.md). |
| **10.** |Azure PowerShell |Les appareils virtuels StorSimple ne peuvent pas être gérés via Azure PowerShell dans cette version. |La gestion des appareils virtuels doit être effectuée par le biais du portail Azure et de l’interface utilisateur web locale. |
| **11.** |Modification de mot de passe |La console de l’appareil exécutant l’appareil virtuel accepte uniquement les entrées au format de clavier en-US. | |
| **12.** |CHAP |Il est impossible de supprimer les informations d’identification CHAP une fois qu’elles ont été créées. En outre, si vous modifiez les informations d’identification CHAP, vous devez mettre les volumes hors connexion, puis les remettre en ligne pour que la modification prenne effet. |Ce problème est résolu dans une version ultérieure. |
| **13.** |Serveur iSCSI |Le « stockage utilisé » affiché pour un volume iSCSI peut différer entre le service StorSimple Device Manager et l’hôte iSCSI. |L’hôte iSCSI dispose de la vue du système de fichiers.<br></br>L’appareil voit les blocs alloués lorsque le volume était à sa taille maximale. |
| **14.** |Serveur de fichiers |Si un fichier d’un dossier est associé à un flux de données alternatif (ADS), l’ADS n’est pas sauvegardé ou restauré lors des opérations de récupération d’urgence, de clonage ou de récupération au niveau de l’élément. | |
| **15.** |Serveur de fichiers |Les liens symboliques ne sont pas pris en charge. | |
| **16.** |Serveur de fichiers |Quand des fichiers protégés par le système de fichiers EFS Windows sont copiés ou stockés sur le serveur de fichiers StorSimple Virtual Array, la configuration qui en résulte n’est pas prise en charge.  | |
| **17.** |Mises à jour |Si le code d’erreur 2359302 (hex 0x240006) apparaît quand vous installez un correctif logiciel par le biais de l’interface utilisateur locale, cela implique que le correctif est déjà installé sur votre appareil.   | |

## <a name="next-step"></a>Étape suivante
[Installation d’Update 0.6](storsimple-virtual-array-install-update-06.md) sur StorSimple Virtual Array.

## <a name="references"></a>References
Vous recherchez une note de version antérieure ? Accédez à :

* [Notes de publication de StorSimple Virtual Array Update 0.5](storsimple-virtual-array-update-05-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.4](storsimple-virtual-array-update-04-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.3](storsimple-ova-update-03-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.1 et 0.2](storsimple-ova-update-01-release-notes.md)
* [Notes de publication de StorSimple Virtual Array General Availability]()