---
title: Se préparer à la défaillance d’un appareil Azure Stack Edge Pro
description: Décrit comment obtenir le remplacement d’un appareil Azure Stack Edge Pro défectueux.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 12/11/2020
ms.author: alkohli
ms.openlocfilehash: b437ce7b6894ebefe38b32f27d370d9f8c4bfe80
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97369019"
---
# <a name="prepare-for-an-azure-stack-edge-pro-gpu-device-failure"></a>Se préparer à la défaillance d’un appareil Azure Stack Edge Pro avec GPU

Cet article vous aide à vous préparer à la défaillance d’un appareil en détaillant comment enregistrer et sauvegarder la configuration et les données de l’appareil sur votre appareil Azure Stack Edge Pro avec GPU. 

Cet article n’inclut pas les procédures visant à sauvegarder des conteneurs Kubernetes et IoT déployés sur votre appareil Azure Stack Edge Pro avec GPU. 

## <a name="understand-device-failures"></a>Comprendre les défaillances d’appareils

Votre appareil Azure Stack Edge Pro avec GPU peut rencontrer deux types de défaillances matérielles.

- Les défaillances tolérables qui exigent que vous remplaciez un composant matériel. Ces défaillances vous permettent d’utiliser l’appareil dont l’état est détérioré. Parmi ces défaillances, citons une unité d’alimentation défaillante unique ou un disque défaillant unique sur l’appareil. Dans chacun de ces cas, l’appareil peut continuer à fonctionner. Contactez Support Microsoft dès que possible pour remplacer les composants défaillants.

- Défaillances non tolérables qui nécessitent le remplacement de l’ensemble de l’appareil (par exemple, lorsque deux disques ont échoué sur votre appareil). Dans ces cas, contactez immédiatement Support Microsoft. Après avoir déterminé que le remplacement d’un appareil est nécessaire, Support vous aidera à remplacer votre appareil Azure Stack Edge.

Pour vous préparer à des défaillances non tolérables, vous devez sauvegarder les éléments suivants sur votre appareil :

- Informations concernant la configuration de l’appareil
- Données des partages locaux Edge et des partages cloud Edge
- Fichiers et dossiers associés aux machines virtuelles exécutées sur votre appareil


## <a name="back-up-device-configuration"></a>Sauvegarder la configuration de l’appareil

Lors de la configuration initiale de l’appareil, il est important de conserver une copie des informations de configuration de l’appareil, comme indiqué dans la [liste de vérification du déploiement](azure-stack-edge-gpu-deploy-checklist.md). Pendant la récupération, ces informations de configuration sont appliquées au nouvel appareil de remplacement. 

## <a name="protect-device-data"></a>Protéger les données des appareils

Les données de l’appareil peuvent être de l’un des types suivants :

- Données dans des partages cloud Edge
- Données dans des partages locaux
- Fichiers et dossiers sur des machines virtuelles

Les sections suivantes décrivent les étapes et les recommandations pour protéger chacun de ces types de données sur votre appareil.

## <a name="protect-data-in-edge-cloud-shares"></a>Protéger des données dans des partages cloud Edge

Vous pouvez créer des partages cloud Edge qui hiérarchisent les données de votre appareil vers Azure. En fonction de la bande passante réseau disponible, configurez des modèles de bande passante sur votre appareil pour réduire les pertes de données en cas de défaillance non tolérable.

> [!IMPORTANT]
> Si l’appareil présente une défaillance non tolérable, les données locales qui ne sont pas hiérarchisées depuis l’appareil vers Azure peuvent être perdues. 

## <a name="protect-data-in-edge-local-shares"></a>Protéger des données dans des partages locaux Edge

Si vous déployez Kubernetes ou IoT Edge, procédez à une configuration pour enregistrer les données d’application localement sur l’appareil et ne les synchronisez pas avec Stockage Azure. Les données sont stockées sur un partage sur l’appareil. Il pourrait être utile de sauvegarder les données de ces partages.

Les solutions tierces de protection des données suivantes peuvent être utilisées pour sauvegarder les données dans les partages SMB ou NFS locaux. 

| Logiciels tiers           | Référence à la solution                               |
|--------------------------------|---------------------------------------------------------|
| Cohesity                       | [https://www.cohesity.com/solution/cloud/azure/](https://www.cohesity.com/solution/cloud/azure/) <br> Pour plus d’informations, contactez Cohesity.          |
| Commvault                      | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br> Pour plus d’informations, contactez Commvault.          |
| Veritas                        | [http://veritas.com/azure](http://veritas.com/azure) <br> Pour plus d’informations, contactez Veritas.   |
| Veeam                          | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Pour plus d’informations, contactez Veeam. |


## <a name="protect-files-and-folders-on-vms"></a>Protéger les fichiers et dossiers sur les machines virtuelles

Azure Stack Edge fonctionne avec Sauvegarde Azure et d’autres solutions tierces de protection des données pour offrir une solution de sauvegarde permettant de protéger les données contenues dans les machines virtuelles déployées sur l’appareil. Le tableau suivant répertorie les références aux solutions disponibles que vous pouvez choisir.


| Solutions de sauvegarde        | Systèmes d’exploitation pris en charge   | Référence                                                                |
|-------------------------|----------------|--------------------------------------------------------------------------|
| Agent Microsoft Azure Recovery Services (MARS) pour Sauvegarde Azure | Windows        | [À propos de l’agent MARS](../backup/backup-azure-about-mars.md)    |
| Cohesity                | Windows, Linux | [Présentation de la solution d’intégration, de sauvegarde et de récupération de Microsoft Azure](https://www.cohesity.com/solution/cloud/azure) <br>Pour plus d’informations, contactez Cohesity.                          |
| Commvault               | Windows, Linux | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br>Pour plus d’informations, contactez Commvault.                          |
| Veritas                 | Windows, Linux | [https://vox.veritas.com/t5/Protection/Protecting-Azure-Stack-Edge-with-NetBackup/ba-p/883370](https://vox.veritas.com/t5/Protection/Protecting-Azure-Stack-Edge-with-NetBackup/ba-p/883370) <br> Pour plus d’informations, contactez Veritas.                    |
| Veeam                   | Windows, Linux | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Pour plus d’informations, contactez Veeam. |


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [récupérer après la défaillance d’un appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-recover-device-failure.md).