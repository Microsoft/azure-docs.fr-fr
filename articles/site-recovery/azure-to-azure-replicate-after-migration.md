---
title: Configurer la récupération d’urgence après une migration vers Azure à l’aide d’Azure Site Recovery
description: Cet article décrit comment préparer des machines pour configurer la récupération d’urgence entre des régions Azure après une migration vers Azure à l’aide d’Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: a71e476a214c44514c7d57c54a09a38218ad6d2a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86135669"
---
# <a name="set-up-disaster-recovery-for-azure-vms-after-migration-to-azure"></a>Configurer la récupération d’urgence pour des machines virtuelles Azure après la migration vers Azure 


Consultez cet article si vous avez [migré des machines locales vers des machines virtuelles Azure](./migrate-tutorial-on-premises-azure.md) à l’aide du service [Site Recovery](site-recovery-overview.md) et souhaitez à présent configurer ces machines virtuelles à des fins de récupération d'urgence dans une région Azure secondaire. Cet article explique comment s'assurer que l'agent de machine virtuelle Azure est installé sur les machines virtuelles migrées et comment supprimer le service Site Recovery Mobility devenu inutile au terme de la migration.



## <a name="verify-migration"></a>Vérifier la migration

Avant de configurer la récupération d’urgence, assurez-vous que la migration s’est effectuée correctement. Pour réussir une migration, après le basculement, sélectionnez l’option **Terminer la migration** pour chaque machine à migrer. 

## <a name="verify-the-azure-vm-agent"></a>Vérifier l’agent de machine virtuelle Azure

L'[agent de machine virtuelle Azure](../virtual-machines/extensions/agent-windows.md) doit être installé sur chaque machine virtuelle Azure. Pour répliquer des machines virtuelles Azure, Site Recovery installe une extension sur l’agent.

- Si la machine exécute la version 9.7.0.0 ou ultérieure du service Site Recovery Mobility, ce dernier installe automatiquement l’agent de machine virtuelle Azure sur les machines virtuelles Windows. Pour les versions antérieures, vous installerez l’agent manuellement.
- Pour les machines virtuelles Linux, vous devez installer l’agent de machine virtuelle Azure manuellement. Installez l’agent de machine virtuelle Azure uniquement si la version 9.6 ou antérieure du service Mobilité est installée sur la machine migrée.


### <a name="install-the-agent-on-windows-vms"></a>Installer l'agent sur les machines virtuelles Windows

Si vous exécutez une version du service Site Recovery Mobility antérieure à la version 9.7.0.0, ou s'il vous faut, pour toute autre raison; installer l'agent manuellement, procédez comme suit :  

1. Vérifiez que vous disposez des autorisations d'administrateur sur la machine virtuelle.
2. Téléchargez le [programme d’installation de l’agent de machine virtuelle Azure](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409).
3. Exécutez le fichier du programme d’installation.

#### <a name="validate-the-installation"></a>validation de l'installation
Pour vérifier que l'agent est installé :

1. Sur la machine virtuelle Azure, dans le dossier C:\WindowsAzure\Packages, vous devez voir le fichier WaAppAgent.exe.
2. Cliquez avec le bouton droit sur ce fichier et, dans **Propriétés**, sélectionnez l’onglet **Détails**.
3. Vérifiez que le champ **Version du produit** indique 2.6.1198.718 ou une version ultérieure.

[En savoir plus](../virtual-machines/extensions/agent-windows.md) sur l'installation de l'agent pour Windows.

### <a name="install-the-agent-on-linux-vms"></a>Installer l'agent sur les machines virtuelles Linux

Installez manuellement l'agent de [machine virtuelle Linux Azure](../virtual-machines/extensions/agent-linux.md) comme suit :

1. Vérifiez que vous disposez d'autorisations d’administrateur sur la machine.
2. Nous vous recommandons vivement d'installer l'agent de machine virtuelle Linux à l'aide d'un package RPM ou DEB à partir du référentiel de packages de la distribution. Tous les [fournisseurs de distribution approuvés](../virtual-machines/linux/endorsed-distros.md) intègrent l’agent Azure Linux dans leurs images et référentiels.
    - Nous recommandons vivement la mise à jour de l’agent uniquement par le biais de référentiel de distribution.
    - Nous vous déconseillons d’installer l’agent de machine virtuelle Linux directement à partir de GitHub et de le mettre à jour.
    -  Si la dernière version de l’agent n’est pas disponible pour la distribution, contactez le support de distribution pour savoir comment l’installer. 

#### <a name="validate-the-installation"></a>validation de l'installation 

1. Exécutez la commande suivante : **ps -e** pour vérifier que l’agent Azure s’exécute sur la machine virtuelle Linux.
2. Si le processus ne s’exécute pas, redémarrez-le à l’aide des commandes suivantes :
    - Pour Ubuntu : **service walinuxagent start**
    - Pour les autres distributions : **service waagent start**


## <a name="uninstall-the-mobility-service"></a>Désinstaller le service Mobilité

1. Désinstallez manuellement le service Mobilité de la machine virtuelle Azure à l’aide d’une des méthodes suivantes. 
    - Pour Windows, dans le Panneau de configuration > **Ajout/Suppression de programmes**, désinstallez **Service Mobilité/Serveur cible maître Microsoft Azure Site Recovery**. À partir d’une invite de commandes avec élévation de privilèges, exécutez :
        ```
        MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
        ```
    - Pour Linux, connectez-vous en tant qu’utilisateur racine. Sur un terminal, accédez à **/user/local/ASR** et exécutez la commande suivante :
        ```
        ./uninstall.sh -Y
        ```
2. Redémarrez la machine virtuelle avant de configurer la réplication.

## <a name="next-steps"></a>Étapes suivantes

[Consultez la résolution des problèmes](site-recovery-extension-troubleshoot.md) liés à l'extension Site Recovery sur l’agent de machine virtuelle Azure.
[Répliquer rapidement](azure-to-azure-quickstart.md) une machine virtuelle Azure dans une région secondaire.
