---
title: Configuration des pilotes GPU AMD de série N Azure pour Windows
description: Procédure de configuration des pilotes GPU AMD pour les machines virtuelles série N exécutant Windows Serveur ou Windows dans Azure
author: vikancha-MSFT
manager: jkabat
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/4/2019
ms.author: vikancha
ms.openlocfilehash: b62095a3dc48480d8b1d33328d2d0dc25470d763
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461088"
---
# <a name="install-amd-gpu-drivers-on-n-series-vms-running-windows"></a>Installer les pilotes GPU AMD sur les machines virtuelles de série N exécutant Windows

Pour tirer parti des fonctionnalités GPU des nouvelles machines virtuelles de la série NVv4 Azure exécutant Windows, installez des pilotes GPU AMD. L’[extension du pilote GPU AMD](../extensions/hpccompute-amd-gpu-windows.md) installe les pilotes GPU AMD sur une machine virtuelle de série NVv4. Installez ou gérez l’extension à l’aide du portail Azure ou d’outils tels qu’Azure PowerShell ou les modèles Azure Resource Manager. Pour connaître les systèmes d’exploitation pris en charge et les étapes de déploiement, consultez lq [documentation sur l’extension du pilote GPU AMD](../extensions/hpccompute-amd-gpu-windows.md).

Si vous choisissez d’installer manuellement les pilotes GPU AMD, cet article indique les systèmes d’exploitation, les pilotes et les étapes d’installation et de vérification pris en charge.

Seuls les pilotes GPU publiés par Microsoft sont pris en charge sur les machines virtuelles de série NVv4. N’installez PAS de pilotes GPU à partir d’une autre source.

Pour obtenir les spécifications de base, les capacités de stockage et les informations relatives aux disques, consultez [GPU Windows VM sizes](../sizes-gpu.md?toc=/azure/virtual-machines/windows/toc.json) (Tailles de machine virtuelle Windows GPU).



## <a name="supported-operating-systems-and-drivers"></a>Systèmes d’exploitation et pilotes pris en charge

| Système d''exploitation | Pilote |
| -------- |------------- |
| Windows 10 Entreprise multisession - Build 1903 <br/><br/>Windows 10 - Build 1809<br/><br/>Windows Server 2016<br/><br/>Windows Server 2019 | [20.Q1.17](https://download.microsoft.com/download/d/e/f/def0fb44-15ab-4b83-959a-8094eb9d0dfe/AMD-Azure-NVv4-Driver-20Q1-Hotfix3.exe) (.exe) |


## <a name="driver-installation"></a>Installation du pilote

1. Connectez-vous à chaque machine virtuelle série NVv4 à l’aide du Bureau à distance.

2. Si vous devez désinstaller la version précédente du pilote, téléchargez l’utilitaire de nettoyage AMD [ici](https://download.microsoft.com/download/4/f/1/4f19b714-9304-410f-9c64-826404e07857/AMDCleanupUtilityni.exe) Veuillez ne pas utiliser l’utilitaire fourni avec la version précédente du pilote.

3. Téléchargez et installez la dernière version du pilote.

4. Redémarrez la machine virtuelle.

## <a name="verify-driver-installation"></a>Vérification de l’installation du pilote

Vous pouvez vérifier l’installation du pilote dans le Gestionnaire de périphériques. L’exemple suivant illustre une configuration réussie de la carte Radeon Instinct MI25 sur une machine virtuelle NVv4 Azure.
<br />

![Capture d’écran montrant une configuration réussie de la carte Radeon Instinct MI25 sur une machine virtuelle NVv4 Azure.](./media/n-series-amd-driver-setup/device-manager.png)

Vous pouvez utiliser dxdiag pour vérifier les propriétés d'affichage du GPU, y compris la RAM vidéo. L'exemple suivant illustre une partition de 1/2 de la carte Radeon Instinct MI25 sur une machine virtuelle NVv4 Azure.
<br />
![Capture d’écran montrant une partition de 1/2 de la carte Radeon Instinct MI25 sur une machine virtuelle NVv4 Azure.](./media/n-series-amd-driver-setup/dxdiag-output-new.png)

Si vous exécutez Windows 10 build 1903 ou version ultérieure, dxdiag n'affichera aucune information dans l'onglet « Affichage ». Utilisez l'option « Enregistrer toutes les informations » située en bas et le fichier de sortie affichera les informations relatives au GPU AMD MI25.

![Propriétés du pilote GPU](./media/n-series-amd-driver-setup/dxdiag-details.png)
