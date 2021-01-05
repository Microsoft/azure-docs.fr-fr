---
title: Préparer des serveurs Windows Server 2003 pour la migration à l’aide d’Azure Migrate
description: Découvrez comment préparer des serveurs Windows Server 2003 pour la migration à l’aide d’Azure Migrate.
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: how-to
ms.date: 05/27/2020
ms.openlocfilehash: f8d3dea970d07d951467a44661e12000ba413f72
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753743"
---
# <a name="prepare-windows-server-2003-machines-for-migration"></a>Préparer des machines Windows Server 2003 pour la migration

Cet article explique comment préparer des machines exécutant Windows Server 2003 pour la migration vers Azure. 


> [!NOTE]
> [Le support étendu de Windows Server 2003](/troubleshoot/azure/virtual-machines/run-win-server-2003#microsoft-windows-server-2003-end-of-support) a pris fin le 14 juillet 2015.  L’équipe du support Azure continue de vous aider à résoudre les problèmes qui concernent l’exécution de Windows Server 2003 sur Azure. Toutefois, ce support est limité aux problèmes qui ne nécessitent pas de dépannage ou de patchs au niveau du système d’exploitation. La migration de vos applications vers des instances Azure exécutant une version plus récente de Windows Server est l’approche recommandée pour vous assurer de tirer parti de la flexibilité et de la fiabilité du cloud Azure. Toutefois, si vous choisissez de migrer votre instance Windows Server 2003 vers Azure, vous pouvez utiliser l’outil Azure Migrate : Outil Server Migration si votre serveur Windows est une machine virtuelle s’exécutant sur VMware ou Hyper-V.


- Vous pouvez utiliser la migration sans agent pour migrer des [machines virtuelles Hyper-V](tutorial-migrate-hyper-v.md) et des [machines virtuelles VMware](tutorial-migrate-vmware.md) vers Azure.
- Pour vous connecter aux machines virtuelles Azure après la migration, Hyper-V Integration Services doit être installé sur la machine virtuelle Azure. Cette option n’est pas installée par défaut sur les ordinateurs Windows Server 2003.
- Il n’existe pas de lien de téléchargement direct pour installer Hyper-V Integration Services, vous devez donc effectuer les opérations suivantes :
    - Pour les machines virtuelles Hyper-V sur lesquelles la suite logicielle n’est pas installée, vous devez extraire les fichiers d’installation d’Integration Services sur une machine exécutant Windows Server 2012 R2/Windows Server 2012 avec le rôle Hyper-V, puis copier le programme d’installation sur l’ordinateur Windows Server 2003. Les fichiers d’installation ne sont pas disponibles sur les ordinateurs exécutant Windows Server 2016.
    - Pour les machines virtuelles VMware, vous créez une tâche de démarrage qui installe Integration Services lors du démarrage de la machine virtuelle Azure après la migration.


## <a name="install-on-hyper-v-vms"></a>Installer sur des machines virtuelles Hyper-V

Avant la migration, vérifiez si Hyper-V Integration Services est installé, et installez-le si nécessaire.

1. Suivez [ces instructions](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services#turn-an-integration-service-on-or-off-using-hyper-v-manager) pour vérifier s’il est installé.
2. S’il n’est pas installé, connectez-vous à un ordinateur exécutant Windows Server 2012 R2/Windows Server 2012 avec le rôle Hyper-V.
3. Accédez au fichier d’installation sous **C:\Windows\System32\vmguest.iso** et montez le fichier.
2. Copiez le dossier d’installation sur l’ordinateur Windows Server 2003, puis installez Integration Services.
4. Après l’installation, vous pouvez conserver les paramètres par défaut dans Integration Services. 

## <a name="install-on-vmware-vms"></a>Installer sur des machines virtuelles VMware

1. Connectez-vous à un ordinateur exécutant Windows Server 2012 R2/Windows Server 2012 avec le rôle Hyper-V.
2. Accédez au fichier d’installation sous **C:\Windows\System32\vmguest.iso** et montez le fichier.
3. Copiez le dossier d’installation sur la machine virtuelle VMware.
4. À partir de la ligne de commande sur la machine virtuelle, exécutez ```gpedit.msc```.
5. Ouvrez **Configuration d’ordinateur** > **Paramètres Windows** > **Scripts (démarrage/arrêt)** .
6. Dans **Démarrage** > **Ajouter** > **Nom du script**, saisissez l’adresse de setup.exe.
7. Après la migration vers Azure, le script s’exécute lors du premier démarrage de la machine virtuelle Azure.
8. Redémarrez manuellement la machine virtuelle Azure. Une fenêtre contextuelle s’affiche dans les diagnostics de démarrage pour indiquer qu’un redémarrage est nécessaire.
9. Une fois que le script est exécuté et que Hyper-V Integration Services est installé sur la machine virtuelle Azure, vous pouvez supprimer le script du démarrage.
10. Après l’installation, vous pouvez conserver les paramètres par défaut dans Integration Services. 

## <a name="next-steps"></a>Étapes suivantes

- Passer en revue la configuration requise pour la migration des machines virtuelles [VMware](migrate-support-matrix-vmware-migration.md) et [Hyper-V](migrate-support-matrix-hyper-v-migration.md).
- Migrer des machines virtuelles [VMware](server-migrate-overview.md) et [Hyper-V](tutorial-migrate-hyper-v.md).
