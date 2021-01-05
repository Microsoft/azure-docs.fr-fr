---
title: Préparer des machines virtuelles VMware pour la reprotection et la restauration automatique avec Azure Site Recovery
description: Préparer la restauration automatique des machines virtuelles VMware après le basculement avec Azure Site Recovery
ms.topic: conceptual
ms.date: 12/24/2019
ms.openlocfilehash: 5a330f8cba31640d0116ca3d5ccab352ce5b3509
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85847744"
---
# <a name="prepare-for-reprotection-and-failback-of-vmware-vms"></a>Préparer la reprotection et la restauration automatique des machines virtuelles VMWare

Après un [basculement](site-recovery-failover.md) de machines virtuelles VMware ou de serveurs physiques locaux vers Azure, vous reprotégez les machines virtuelles Azure créées après le basculement afin qu’elles soient répliquées vers le site local. Une fois la réplication d’Azure vers un emplacement local en place, vous pouvez effectuer une restauration automatique en exécutant un basculement d’Azure vers le site local lorsque vous êtes prêt.

Avant de continuer, regardez la vidéo ci-dessous pour obtenir un aperçu rapide de la restauration automatique à partir d’Azure vers un site local.<br /><br />
> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/VMware-to-Azure-with-ASR-Video5-Failback-from-Azure-to-On-premises/player]

## <a name="reprotectionfailback-components"></a>Composants de reprotection/restauration automatique

Vous avez besoin d’un certain nombre de composants et de paramètres pour pouvoir reprotéger et effectuer une restauration automatique à partir d’Azure.

**Composant**| **Détails**
--- | ---
**Serveur de configuration local** | Le serveur de configuration local doit être en cours d’exécution et connecté à Azure.<br/><br/> La machine virtuelle vers laquelle vous effectuez la restauration automatique doit exister dans la base de données du serveur de configuration. Si un incident affecte le serveur de configuration, restaurez-le avec la même adresse IP pour garantir le bon fonctionnement de la restauration automatique.<br/><br/>  Si les adresses IP des machines répliquées ont été conservées lors du basculement, la connectivité de site à site ou ExpressRoute doit être établie entre les machines virtuelles Azure et la carte réseau de restauration automatique du serveur de configuration. Pour les adresses IP conservées, le serveur de configuration doit disposer de deux cartes réseau : une pour la connectivité des machines sources et une pour la connectivité de la restauration automatique Azure. Cela permet d’éviter le chevauchement des plages d’adresses de sous-réseau des machines virtuelles sources et basculées.
**Serveur de processus dans Azure** | Vous avez besoin d’un serveur de processus dans Azure avant de pouvoir procéder à une restauration automatique sur votre site local.<br/><br/> Le serveur de processus reçoit des données de la machine virtuelle Azure protégée et les envoie au site local.<br/><br/> Comme vous avez besoin d’un réseau à faible latence entre le serveur de processus et la machine virtuelle protégée, nous vous recommandons de déployer le serveur de processus dans Azure pour des performances de réplication supérieures.<br/><br/> Pour preuve de concept, vous pouvez utiliser le serveur de processus local et ExpressRoute avec un peering privé.<br/><br/> Le serveur de processus doit se trouver dans le réseau Azure dans lequel se trouve la machine virtuelle basculée. Le serveur de processus doit également pouvoir communiquer avec le serveur de configuration local et le serveur cible maître.
**Serveur cible maître distinct** | Le serveur cible maître reçoit les données de restauration automatique et, par défaut, un serveur cible maître Windows s’exécute sur le serveur de configuration local.<br/><br/> Un serveur cible maître peut avoir jusqu’à 60 disques attachés. Les machines virtuelles en cours de restauration ont plus d’un total collectif de 60 disques. Si vous effectuez une restauration automatique de volumes importants de trafic, créez un serveur cible maître distinct pour la restauration automatique.<br/><br/> Si des machines sont rassemblées dans un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, les machines virtuelles doivent toutes avoir le système d’exploitation Windows ou Linux. Pourquoi ? Étant donné que toutes les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître, celui-ci doit avoir le même système d’exploitation (avec la même version ou une version ultérieure) que ceux des machines répliquées.<br/><br/> Le serveur cible maître ne doit pas avoir de captures instantanées sur ses disques, sinon la reprotection et la restauration automatique ne fonctionneront pas.<br/><br/> Le serveur cible maître ne peut pas avoir de contrôleur Paravirtual SCSI. Il peut uniquement s’agir d’un contrôleur LSI Logic. Sans contrôleur LSI Logic, la reprotection échoue.
**Stratégie de réplication par restauration automatique** | Pour répliquer vers un site local, vous avez besoin d’une stratégie de restauration automatique. Cette stratégie est automatiquement créée en même temps que la stratégie de réplication vers Azure.<br/><br/> La stratégie est automatiquement associée au serveur de configuration. Elle est définie sur un seuil d’objectif de point de récupération de 15 minutes, une conservation des points de récupération de 24 heures et une fréquence des captures instantanées de cohérence d’application de 60 minutes. La stratégie ne peut pas être modifiée. 
**Peering privé ExpressRoute/VPN de site à site** | La reprotection et la restauration automatique nécessitent une connexion VPN de site à site ou un peering privé ExpressRoute pour répliquer des données. 


## <a name="ports-for-reprotectionfailback"></a>Ports pour la reprotection/restauration automatique

Un certain nombre de ports doivent être ouverts pour la reprotection/restauration automatique. Le graphique suivant illustre les ports et le déroulement de la reprotection/restauration automatique.

![Ports pour le basculement et la restauration automatique](./media/vmware-azure-reprotect/failover-failback.png)


## <a name="deploy-a-process-server-in-azure"></a>Déployer un serveur de processus dans Azure

1. [Configurez un serveur de processus](vmware-azure-set-up-process-server-azure.md) dans Azure à des fins de restauration automatique.
2. Vérifiez que les machines virtuelles Azure peuvent atteindre le serveur de processus. 
3. Vérifiez que la connexion VPN de site à site ou le réseau de peering privé ExpressRoute dispose d’une bande passante suffisante pour envoyer des données du serveur de processus au site local.

## <a name="deploy-a-separate-master-target-server"></a>Déployer un serveur cible maître distinct

1. Notez les [exigences et limitations](#reprotectionfailback-components) du serveur cible maître.
2. Créez un serveur cible maître [Windows](site-recovery-plan-capacity-vmware.md#deploy-additional-master-target-servers) ou [Linux](vmware-azure-install-linux-master-target.md) pour qu’il corresponde au système d’exploitation des machines virtuelles que vous souhaitez reprotéger et restaurer.
3. Veillez à ne pas utiliser Storage vMotion pour le serveur cible maître, sinon la restauration automatique peut échouer. La machine virtuelle ne peut pas démarrer, car les disques ne sont pas disponibles.
    - Pour éviter ce problème, excluez le serveur cible maître de votre liste vMotion.
    - Si une cible maître est l’objet d’une tâche Storage vMotion après la reprotection, les disques de machine virtuelle protégés qui sont attachés au serveur cible maître migrent vers la cible de la tâche vMotion. Si vous essayez d’effectuer une restauration automatique après cela, le détachement des disques échoue, car les disques sont introuvables. Il est alors difficile de trouver les disques dans vos comptes de stockage. Si cela se produit, recherchez-les manuellement et attachez-les à la machine virtuelle. Après quoi, vous pouvez démarrer la machine virtuelle locale.

4. Ajoutez un lecteur de rétention au serveur cible maître Windows existant. Ajoutez un disque et formatez le lecteur. Le lecteur de rétention est utilisé pour arrêter les points dans le temps où la machine virtuelle est répliquée vers le site local. Notez ces critères. S’ils ne sont pas remplis, le lecteur n’est pas listé pour le serveur cible maître :
    - Le volume n’est pas utilisé à d’autres fins, par exemple une cible de réplication, et il n’est pas en mode verrouillage.
    - Le volume n’est pas un volume de cache. Le volume d’une installation personnalisée du serveur de processus et du serveur maître cible n’est pas éligible pour un volume de rétention. Lorsque le serveur de processus et le serveur cible maître sont installés sur un volume, ce volume est un volume de cache du serveur cible maître.
    - Le type de système de fichiers du volume n’est pas FAT ni FAT32.
    - La capacité du volume est différente de zéro.
    - Le volume de rétention par défaut pour Windows est le volume R.
    - Le volume de rétention par défaut pour Linux est /mnt/retention.

5. Ajoutez un lecteur si vous utilisez un serveur de processus existant. Le nouveau lecteur doit répondre aux exigences de la dernière étape. Si le lecteur de rétention n’est pas présent, il n’apparaît pas dans la liste déroulante de sélection du portail. Une fois que vous avez ajouté un lecteur au serveur cible maître local, 15 minutes sont nécessaires pour que le lecteur apparaisse dans la sélection sur le portail. Vous pouvez actualiser le serveur de configuration si le lecteur n’apparaît pas après 15 minutes.
6. Installez les outils VMware ou open-vm-tools sur le serveur maître cible. Sans les outils, les banques de données sur l’hôte ESXi du serveur cible maître ne peuvent pas être détectées.
7. Définissez le paramètre disk.EnableUUID=true dans les paramètres de configuration de la machine virtuelle cible maître dans VMware. Si cette ligne n’existe pas, ajoutez-la. Ce paramètre est obligatoire pour fournir un UUID cohérent au disque VMDK afin que celui-ci monte correctement.
8. Vérifiez les conditions requises pour l’accès au serveur vCenter :
    - Si la machine virtuelle vers laquelle vous effectuez la restauration automatique se trouve sur un hôte ESXi géré par le serveur VMware vCenter, le serveur cible maître doit avoir accès au fichier VMDK (Virtual Machine Disk) de la machine virtuelle locale afin d’écrire les données répliquées sur les disques de la machine virtuelle. Vérifiez que le magasin de données de la machine virtuelle locale est monté sur l’hôte du serveur cible maître avec accès en lecture/écriture.
    - Si la machine virtuelle ne se trouve pas sur un hôte ESXi géré par un serveur VMware vCenter, le service Site Recovery crée une machine virtuelle lors de la reprotection. Cette machine virtuelle est créée sur l’hôte ESXi où vous créez la machine virtuelle du serveur cible maître. Choisissez l’hôte ESXi avec soin pour y créer la machine virtuelle. Le disque dur de la machine virtuelle doit être dans une banque de données accessible par l’hôte sur lequel le serveur cible maître s’exécute.
    - Une autre option, si la machine virtuelle locale existe pour la restauration automatique, consiste à la supprimer avant de procéder à une restauration automatique. L’opération de restauration automatique crée ensuite une machine virtuelle sur le même hôte que l’hôte ESXi cible maître. Si vous effectuez la restauration automatique vers un autre emplacement, les données sont récupérées dans le même magasin de données et sur le même hôte ESXi que ceux qui sont utilisés par le serveur cible maître local.
9. Pour les machines physiques restaurées automatiquement vers des machines virtuelles VMware, vous devez effectuer la détection de l’hôte sur lequel le serveur cible maître s’exécute avant de pouvoir reprotéger la machine.
10. Vérifiez que l’hôte ESXi sur lequel figure la machine virtuelle cible maître possède au moins un magasin de données VMFS (Virtual Machine File System) attaché. Si aucun magasin de données VMFS n’est attaché, l’entrée du magasin de données dans les paramètres de reprotection est vide et vous ne pouvez pas continuer.


## <a name="next-steps"></a>Étapes suivantes

[Reprotéger](vmware-azure-reprotect.md) une machine virtuelle.
