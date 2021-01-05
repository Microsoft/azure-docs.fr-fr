---
title: Notes de version de StorSimple Virtual Array Update 0.2 et 0.1
description: Décrit les problèmes critiques non résolus et les solutions possibles pour StorSimple Virtual Array exécuté sous Update 0.2 et 0.1.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 3993864d-2ddd-4302-a2f1-8d737fba6eab
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/16/2016
ms.author: alkohli
ms.openlocfilehash: 67d3fafb9b4b93ae52bdb09d673bf65cad30b816
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966154"
---
# <a name="storsimple-virtual-array-update-02-and-01-release-notes"></a>Notes de publication de StorSimple Virtual Array Update 0.2 et 0.1
## <a name="overview"></a>Vue d’ensemble
Les notes suivantes identifient les problèmes majeurs existants et les problèmes résolus dans les mises à jour Microsoft Azure StorSimple Virtual Array. Microsoft Azure StorSimple Virtual Array est également appelé appareil virtuel StorSimple local ou appareil virtuel StorSimple. 

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez les informations contenues dans les notes de publication avant de déployer votre appareil virtuel StorSimple.

Update 0.2 correspond à la version logicielle **10.0.10280.0** ; Update 0.1 correspond à la version **10.0.10279.0**. Les sections suivantes répertorient les modifications pour chaque mise à jour. 

> [!NOTE]
> Les mises à jour entraînent des perturbations et redémarrent votre appareil. Si des E/S sont en cours, l’appareil subira des interruptions de service.
> 
> 

## <a name="issues-fixed-in-the-update-02"></a>Problèmes résolus dans Update 0.2
Update 0.2 inclut toutes les modifications de Update 0.1 en plus du correctif décrit dans le tableau suivant :

| Fonctionnalité | Problème |
| --- | --- |
| Mises à jour |Dans la dernière version, les mises à jour n’étaient pas détectées automatiquement dans le portail Azure Classic, donc vous deviez utiliser l’interface utilisateur web locale pour installer les mises à jour. Ce problème a été résolu dans cette version. Après avoir installé Update 0.2, vous pouvez installer les mises à jour ultérieures à l’aide du portail Azure Classic. |

## <a name="whats-new-in-the-update-01"></a>Nouveautés d’Update 0.1
Update 0.1 contient les améliorations et correctifs de bogues suivants. 

* **Résilience améliorée en cas d’indisponibilité du cloud** : cette version corrige plusieurs bogues autour de la récupération d’urgence, de la sauvegarde, de la restauration et de la hiérarchisation en cas d’interruption de la connectivité du cloud. 
* **Performances de restauration améliorées** : cette version comporte des correctifs de bogues qui ont considérablement réduit la durée d’exécution des tâches de restauration.
* **Optimisation de la récupération d’espace automatisée** : quand des données sont supprimées dans des volumes alloués de façon dynamique, les blocs de stockage non utilisés doivent être récupérés. Cette version améliore le processus de récupération d’espace à partir du cloud de manière à accélérer la disponibilité de l’espace inutilisé par rapport aux versions précédentes.
* **Nouvelles images de disque virtuel** : de nouveaux VHD, VHDX et VMDK sont désormais disponibles via le portail Azure Classic. Vous pouvez télécharger ces images pour configurer de nouveaux appareils Update 0.1.
* **Amélioration de la précision de l’état des tâches sur le portail** : dans la version antérieure du logiciel, les rapports d’état des tâches générés sur le portail n’étaient pas précis. Ce problème a été résolu dans cette version.
* **Expérience de jonction de domaine** : correctifs de bogues liés à la jonction de domaine et au changement de nom de l’appareil.

## <a name="issues-fixed-in-the-update-01"></a>Problèmes résolus dans Update 0.1
Le tableau suivant récapitule les problèmes corrigés dans cette version.

| Non. | Fonctionnalité | Problème |
| --- | --- | --- |
| 1 |VMDK |Dans certaines versions de VMware, le disque du système d’exploitation était considéré comme partiellement alloué, ce qui générait des alertes et perturbait les opérations normales. Ce problème a été résolu dans cette version. |
| 2 |Serveur iSCSI |Dans la dernière version, l’utilisateur devait spécifier une passerelle pour chaque interface réseau activée de l’appareil virtuel StorSimple. Ce comportement a été modifié dans cette version afin que l’utilisateur ait à configurer au moins une passerelle pour toutes les interfaces réseau activées. |
| 3 |Package de prise en charge |Dans la version antérieure du logiciel, la collecte du package Support échouait dès lors que la taille du package excédait 1 Go. Ce problème a été résolu dans cette version. |
| 4 |Accès au cloud |Dans la dernière version, si le StorSimple Virtuel Array était redémarré alors qu’il ne disposait pas de connectivité réseau, l’interface utilisateur locale rencontrait des problèmes de connectivité. Ce problème a été résolu dans cette version. |
| 5 |Graphiques de surveillance |Dans la version précédente, après un basculement de l’appareil, les graphiques d’utilisation de la capacité du cloud affichaient des valeurs incorrectes dans le portail Azure Classic. Ce problème a été résolu dans la version actuelle. |

## <a name="known-issues-in-the-update-01"></a>Problèmes connus dans Update 0.1
Le tableau suivant récapitule les problèmes connus pour StorSimple Virtual Array et décrit les problèmes signalés dans les notes de version des versions précédentes. **Les problèmes signalés dans la note de version sont repérés par un astérisque. Presque tous les problèmes de cette liste portent sur la version en disponibilité générale de StorSimple Virtual Array.**

| Non. | Fonctionnalité | Problème | Solution de contournement/commentaires |
| --- | --- | --- | --- |
| **1.** |Mises à jour |Les appareils virtuels créés dans la version préliminaire ne peuvent pas être mis à jour vers une version à disponibilité générale prise en charge. |Ces appareils virtuels doivent être basculés vers la version à disponibilité générale à l'aide d'un flux de travail de récupération d'urgence (DR). |
| **2.** |Disque de données configuré |Une fois que vous avez configuré un disque de données d'une certaine spécifiée et créé l'appareil virtuel StorSimple correspondant, vous ne devez pas développer, ni réduire le disque de données. Toute tentative de ce type entraînera une perte de toutes les données au niveau local de l’appareil. | |
| **3.** |Stratégie de groupe |Lorsqu’un appareil est joint à un domaine, le fait d’appliquer une stratégie de groupe peut avoir un impact négatif sur son fonctionnement. |Assurez-vous que votre tableau virtuel est dans sa propre unité organisationnelle (UO) pour Active Directory et qu'aucun objet de stratégie de groupe (GPO) ne lui est appliqué. |
| **4.** |Interface utilisateur web locale |Si les fonctionnalités de sécurité améliorées sont activées dans Internet Explorer (IE ESC), certaines pages de l’interface utilisateur web locale, comme Dépannage ou Maintenance, peuvent ne pas fonctionner correctement. Les boutons sur ces pages peuvent également ne pas fonctionner. |Désactivez les fonctionnalités de sécurité améliorées d'Internet Explorer. |
| **5.** |Interface utilisateur web locale |Sur une machine virtuelle Hyper-V, les interfaces réseau de l'interface utilisateur web sont affichées sous forme d'interfaces 10 Gbits/s. |Ce comportement est le reflet de Hyper-V. Hyper-V affiche toujours 10 Gbits/s pour les cartes de réseau virtuel. |
| **6.** |Partages ou volumes à plusieurs niveaux |Le verrouillage de la plage d'octets pour les applications qui fonctionnent avec les volumes à plusieurs niveaux StorSimple n'est pas pris en charge. Si le verrouillage de la plage d'octets est activé, la hiérarchisation StorSimple ne fonctionnera pas. |Mesures recommandées : <br></br>Désactivez le verrouillage de plage d'octets dans la logique de votre application.<br></br>Choisissez de placer les données de cette application dans des volumes épinglés localement par opposition à des volumes à plusieurs niveaux.<br></br>*Inconvénient* : si le verrouillage de la plage d’octets est activé avec l’utilisation de volumes localement épinglés, sachez que le volume épinglé localement peut être en ligne même avant la fin de la restauration. Dans ce cas, si une restauration est en cours, vous devez attendre que l'opération se termine. |
| **7.** |Partages à plusieurs niveaux |L'utilisation de fichiers volumineux peut entraîner montée en charge de niveau lente. |Lorsque vous utilisez des fichiers volumineux, vérifiez que la taille du plus grand fichier est inférieure à 3 % de la taille du partage. |
| **8.** |Capacité utilisée pour les partages |Vous pouvez constater la consommation du partage en l'absence de données sur le partage. Ceci est dû au fait que la capacité utilisée pour les partages inclut des métadonnées. | |
| **9.** |Récupération d'urgence |La récupération d'urgence d'un serveur de fichiers ne peut s'effectuer que dans le même domaine que celui de l'appareil source. La récupération d'urgence vers un appareil cible dans un autre domaine n'est pas prise en charge dans cette version. |Ceci sera implémenté dans une version ultérieure. |
| **10.** |Azure PowerShell |Les appareils virtuels StorSimple ne peuvent pas être gérés via Azure PowerShell dans cette version. |La gestion des appareils virtuels doit être effectuée par le biais du portail Azure Classic et l'interface utilisateur web locale. |
| **11.** |Modification de mot de passe |La console de l’appareil exécutant le tableau virtuel accepte uniquement les entrées au format de clavier en-US. | |
| **12.** |CHAP |Il est impossible de supprimer les informations d’identification CHAP une fois qu’elles ont été créées. En outre, si vous modifiez les informations d’identification CHAP, vous devrez mettre les volumes hors connexion, puis les remettre en ligne pour que la modification prenne effet. |Ceci sera résolu dans une version ultérieure. |
| **13.** |Serveur iSCSI |Le « stockage utilisé » affiché pour un volume iSCSI peut différer entre le service StorSimple Manager et l’hôte iSCSI. |L’hôte iSCSI dispose de la vue du système de fichiers.<br></br>L’appareil voit les blocs alloués lorsque le volume était à sa taille maximale. |
| **14.** |Serveur de fichiers* |Si un fichier d’un dossier est associé à un flux de données alternatif (ADS), l’ADS n’est pas sauvegardé ou restauré lors des opérations de récupération d’urgence, de clonage ou de récupération au niveau de l’élément. | |

## <a name="next-step"></a>Étape suivante
[Installation de mises à jour](./storsimple-virtual-array-install-update-06.md) sur votre instance StorSimple Virtual Array.