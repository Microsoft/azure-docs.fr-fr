---
title: Notes de publication de StorSimple Virtual Array Update 0.5 | Microsoft Docs
description: Décrit les problèmes critiques non résolus et les solutions possibles pour StorSimple Virtual Array exécutant Update 0.5.
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
ms.date: 05/08/2017
ms.author: alkohli
ms.openlocfilehash: f70cf96baed274eaf260832bd83ab0e2c91f3683
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967038"
---
# <a name="storsimple-virtual-array-update-05-release-notes"></a>Notes de publication de StorSimple Virtual Array Update 0.5

## <a name="overview"></a>Vue d’ensemble

Les notes suivantes identifient les problèmes majeurs existants et les problèmes résolus dans les mises à jour Microsoft Azure StorSimple Virtual Array.

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez les informations contenues dans les notes de publication avant de déployer votre instance StorSimple Virtual Array.

Update 0.5 correspond à la version logicielle **10.0.10290.0**.

> [!NOTE]
> Les mises à jour entraînent des perturbations et redémarrent votre appareil. Si des E/S sont en cours, l’appareil subit des interruptions de service. Pour obtenir des instructions détaillées sur la façon d’appliquer la mise à jour, consultez la page relative à [l’installation d’Update 0.5](storsimple-virtual-array-install-update-05.md).


## <a name="whats-new-in-the-update-05"></a>Nouveautés d’Update 0.5
Update 0.5 est essentiellement une version de correctif de bogue. Les principales améliorations et les principaux correctifs de bogues sont les suivantes :

- **Améliorations de la résilience de sauvegarde** : cette version comporte des correctifs qui améliorent la résilience de sauvegarde. Dans les versions antérieures, les nouvelles tentatives de sauvegarde étaient limitées à certaines exceptions. Dans cette version, toutes les exceptions de sauvegarde font l’objet d’une nouvelle tentative, ce qui améliore la résilience des sauvegardes.

- **Mises à jour pour la surveillance de l’utilisation du stockage** : la surveillance de l’utilisation du stockage sera interrompue à compter du 30 juin 2017 pour la série d’appareils virtuels StorSimple. Ce changement s’applique aux graphiques d’analyse de tous les appareils virtuels qui exécutent des versions Update 0.4 ou antérieures. Cette mise à jour contient les modifications dont vous avez besoin pour pouvoir continuer à utiliser la surveillance de l’utilisation du stockage dans le portail Azure. Installez cette mise à jour critique avant le 30 juin 2017 pour continuer d’utiliser cette fonctionnalité.


## <a name="issues-fixed-in-the-update-05"></a>Problèmes résolus dans Update 0.5

Le tableau suivant récapitule les problèmes corrigés dans cette version.

| Non. | Fonctionnalité | Problème |
| --- | --- | --- |
| 1 |Résilience de sauvegarde| Dans les versions antérieures, les nouvelles tentatives de sauvegarde étaient limitées à certaines exceptions. Cette version contient un correctif destiné à améliorer la résilience des sauvegardes en étendant les nouvelles tentatives à toutes les exceptions de sauvegarde.|
| 2 |Surveillance| La surveillance de l’utilisation du stockage sera interrompue le 30 juin 2017 pour la série d’appareils virtuels StorSimple. Cette action a un impact sur les graphiques de surveillance du service StorSimple Device Manager exécuté sur les appareils virtuels StorSimple (modèle 1200). Cette version comporte des mises à jour qui permettent aux utilisateurs de continuer d’utiliser la fonction de surveillance de l’utilisation du stockage sur les appareils virtuels après le 30 juin 2017.|
| 3 |Serveur de fichiers| Dans les versions antérieures, un utilisateur pouvait copier par erreur des fichiers chiffrés sur l’appareil virtuel. Cette version contient un correctif qui n’autorise pas la copie de fichiers chiffrés sur l’appareil virtuel. Si votre appareil comporte déjà des fichiers chiffrés avant la mise à jour, les sauvegardes continueront d’échouer tant que l’ensemble des fichiers chiffrés n’auront pas été supprimés du système. |


## <a name="known-issues-in-the-update-05"></a>Problèmes connus dans Update 0.5

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

## <a name="next-step"></a>Étape suivante
[Installation d’Update 0.5](storsimple-virtual-array-install-update-05.md) sur StorSimple Virtual Array.

## <a name="references"></a>References
Vous recherchez une note de version antérieure ? Accédez à :

* [Notes de publication de StorSimple Virtual Array Update 0.4](storsimple-virtual-array-update-04-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.3](storsimple-ova-update-03-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.1 et 0.2](storsimple-ova-update-01-release-notes.md)
* [Notes de publication de StorSimple Virtual Array General Availability](./storsimple-virtual-array-update-06-release-notes.md)