---
title: Notes de version de Microsoft Azure StorSimple Virtual Array Update 1.2 | Microsoft Docs
description: Décrit les problèmes critiques non résolus de StorSimple Virtual Array Update 1.2 et fournit des solutions.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: article
ms.date: 05/29/2019
ms.author: alkohli
ms.openlocfilehash: 1118dfcec67a49365f1f6e5e522e98b97694d052
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94960188"
---
# <a name="storsimple-virtual-array-update-12-release-notes"></a>Notes de version de StorSimple Virtual Array Update 1.2

Les notes suivantes identifient les problèmes majeurs existants et les problèmes résolus dans les mises à jour Microsoft Azure StorSimple Virtual Array.

Les notes de publication sont constamment mises à jour. Les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez les informations contenues dans les notes de publication avant de déployer votre instance StorSimple Virtual Array.

Update 1.2 correspond à la version logicielle **10.0.10311.0**.

> [!IMPORTANT]
> - Les mises à jour entraînent des perturbations et redémarrent votre appareil. Si des E/S sont en cours, l’appareil subit des interruptions de service. Pour obtenir des instructions détaillées sur les packages qui sont utilisés pour appliquer cette mise à jour, consultez [Télécharger Update 1.2](#download-update-12).
> - Update 1.2 n’est disponible dans le portail Azure que si votre appareil exécute Update 1.0 ou 1.1.

## <a name="whats-new-in-update-12"></a>Nouveautés d'Update 1.2

Cette mise à jour contient les correctifs de bogues suivants :

- Amélioration de la résilience lors du traitement des fichiers supprimés.
- Gestion améliorée des exceptions dans le chemin de démarrage du code permettant une réduction des défaillances lors des sauvegardes, des restaurations, des lectures cloud et des récupérations d’espace automatisées.

## <a name="download-update-12"></a>Télécharger Update 1.2

Pour télécharger cette mise à jour, accédez au [ Catalogue Microsoft Update](https://www.catalog.update.microsoft.com/Home.aspx), puis téléchargez le package KB4502035. Ce package contient les packages suivants :

 - **KB4493446**, qui contient des mises à jour cumulatives pour Windows 2012 R2, jusqu’à avril 2019. Pour plus d’informations sur le contenu de ce correctif cumulatif, consultez [Correctif cumulatif mensuel du mois d’avril](https://support.microsoft.com/help/4493446/windows-8-1-update-kb4493446).
 - **KB3011067**, qui est un fichier de package autonome Microsoft Update (WindowsTH-KB3011067-x64). Ce fichier est utilisé pour mettre à jour le logiciel de l’appareil.

Téléchargez KB4502035 et suivez les instructions pour [appliquer la mise à jour via l’interface utilisateur web locale](storsimple-virtual-array-install-update-11.md#use-the-local-web-ui).

## <a name="issues-fixed-in-update-12"></a>Problèmes résolus dans Update 1.2

Le tableau suivant récapitule les problèmes corrigés dans cette version.

| Non. | Fonctionnalité | Problème |
| --- | --- | --- |
| 1 |Suppression| Dans les versions précédentes du logiciel, le niveau d’utilisation de l’appareil ne changeait pas, même après la suppression de fichiers. Ce problème a été résolu dans cette version. La hiérarchisation du chemin du code est à présent plus résiliente lors du traitement des fichiers supprimés.|
| 2 |Gestion des exceptions| Dans les versions précédentes du logiciel, le redémarrage du système pouvait entraîner des échecs de sauvegarde, de restauration, de lecture à partir du cloud et de récupération d’espace automatisée. Cette version corrige la façon dont les exceptions sont gérées dans le chemin de démarrage.|

## <a name="known-issues-in-update-12"></a>Problèmes connus dans Update 1.2

Aucun problème n’a été signalé concernant Update 1.2. Tous les problèmes mentionnés dans les notes de publication proviennent des versions précédentes. Pour voir le récapitulatif des problèmes connus concernant les versions précédentes, accédez à [Problèmes connus dans Update 1.1](storsimple-virtual-array-update-11-release-notes.md#known-issues-in-update-11).

## <a name="next-steps"></a>Étapes suivantes

Télécharger KB4502035 et [appliquer la mise à jour via l’interface utilisateur web locale](storsimple-virtual-array-install-update-11.md#use-the-local-web-ui)

## <a name="references"></a>References

Vous recherchez une note de version antérieure ? Accédez à :
* [Notes de version de StorSimple Virtual Array Update 1.1](storsimple-virtual-array-update-11-release-notes.md)
* [Notes de version de StorSimple Virtual Array Update 1.0](storsimple-virtual-array-update-1-release-notes.md)
* [Notes de version de StorSimple Virtual Array Update 0.6](storsimple-virtual-array-update-06-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.5](storsimple-virtual-array-update-05-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.4](storsimple-virtual-array-update-04-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.3](storsimple-ova-update-03-release-notes.md)
* [Notes de publication de StorSimple Virtual Array Update 0.1 et 0.2](storsimple-ova-update-01-release-notes.md)
* [Notes de publication de StorSimple Virtual Array General Availability](./storsimple-virtual-array-update-06-release-notes.md)