---
title: Détecter un problème de restauration automatique dans la récupération d’urgence de machines virtuelles VMware avec Azure Site Recovery
description: Cet article explique comment résoudre les problèmes de restauration automatique et de reprotection pendant la reprise d’activité de machines virtuelles VMware sur Azure avec Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: sharrai
ms.openlocfilehash: ed4e52470264441a99c5ccf0a736bb00233510c1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87423114"
---
# <a name="troubleshoot-failback-to-on-premises-from-azure"></a>Résoudre les problèmes de restauration automatique sur l’infrastructure locale à partir d’Azure

Cet article explique comment résoudre les problèmes que vous pouvez rencontrer lorsque vous restaurez automatiquement des machines virtuelles Azure sur votre infrastructure VMware locale, après le basculement vers Azure à l’aide de [Azure Site Recovery](site-recovery-overview.md).

Une restauration automatique implique essentiellement deux étapes principales. Pour la première étape, après le basculement, vous devez reprotéger les machines virtuelles Azure sur des machines locales pour qu’elles démarrent la réplication. La deuxième étape consiste à exécuter un basculement depuis Azure pour effectuer une restauration automatique sur votre site local.

## <a name="common-issues"></a>Problèmes courants

- Si vous effectuez une découverte utilisateur vCenter en mode lecture seule et protégez des machines virtuelles, la protection réussit et le basculement fonctionne. Lors de la reprotection, le basculement échoue parce que les banques de données ne peuvent pas être découvertes. L’un des symptômes est que les banques de données ne sont pas répertoriées lors de la reprotection. Pour résoudre ce problème, vous pouvez mettre à jour les informations d’identification vCenter avec un compte approprié qui dispose des autorisations, puis réessayez le travail.
- Lorsque vous restaurez automatiquement une machine virtuelle Linux et l’exécutez en local, vous pouvez constater que le package du Gestionnaire de réseau est désinstallé de la machine. Cette désinstallation se produit parce que le package du Gestionnaire de réseau est supprimé quand la machine virtuelle est restaurée dans Azure.
- Lorsqu’une machine virtuelle Linux est configurée avec une adresse IP statique puis restaurée automatiquement vers Azure, l’adresse IP est obtenue à partir de DHCP. Lorsque vous basculez en local, la machine virtuelle continue d’utiliser DHCP pour obtenir l’adresse IP. Connectez-vous manuellement à la machine et redéfinissez l’adresse IP avec une adresse statique, si nécessaire. Une machine virtuelle Windows peut reprendre son adresse IP statique.
- Si vous utilisez l’édition gratuite ESXi 5.5 ou l’édition gratuite de l’hyperviseur vSphere 6, le basculement réussit, mais pas la restauration automatique. Vous devrez effectuer une mise à niveau vers une licence d’évaluation d’un des programmes pour activer la restauration automatique.
- Si vous ne pouvez pas atteindre le serveur de configuration depuis le serveur de processus, utilisez Telnet pour vérifier la connectivité au serveur de configuration sur le port 443. Vous pouvez également essayer d’exécuter un test ping sur le serveur de configuration à partir du serveur de processus. Un serveur de processus doit également avoir une pulsation lorsqu’il est connecté au serveur de configuration.
- Un serveur Windows Server 2008 R2 SP1 qui est protégé en tant que serveur physique local ne peut pas être restauré à partir d’Azure sur un site local.
- Vous ne pouvez pas effectuer de restauration automatique dans les circonstances suivantes :
    - Vous avez effectué une migration des machines vers Azure. 
    - Vous avez déplacé une machine virtuelle vers un autre groupe de ressources.
    - Vous avez supprimé la machine virtuelle Azure.
    - Vous avez désactivé la protection de la machine virtuelle.
    - Vous avez créé la machine virtuelle manuellement dans Azure. La machine doit avoir été initialement protégée localement et basculée vers Azure avant la reprotection.
    - Vous pouvez basculer uniquement vers un hôte ESXi. Vous ne pouvez pas restaurer automatiquement des machines virtuelles VMware ou des serveurs physiques vers des hôtes Hyper-V, des ordinateurs physiques ou des stations de travail VMware.


## <a name="troubleshoot-reprotection-errors"></a>Résoudre les erreurs liées à la reprotection

Cette section explique en détail les erreurs de reprotection courantes et comment les résoudre.

### <a name="error-code-95226"></a>Code d'erreur 95226

**Échec de la reprotection, car la machine virtuelle Azure n’a pas pu contacter le serveur de configuration local.**

Cette erreur se produit quand :

* La machine virtuelle Azure ne peut pas contacter le serveur de configuration local. La machine virtuelle ne peut pas être découverte ni inscrite auprès du serveur de configuration.
* Le service d’application InMage Scout n’est pas exécuté sur la machine virtuelle Azure après le basculement. Ce service est nécessaire pour les communications avec le serveur de configuration local.

Pour résoudre ce problème :

* Vérifiez que le réseau de la machine virtuelle Azure permet à celle-ci de communiquer avec le serveur de configuration local. Vous pouvez soit configurer un VPN de site à site dans votre centre de données local soit configurer une connexion ExpressRoute Azure avec un peering privé sur le réseau virtuel de la machine virtuelle Azure.
* Si la machine virtuelle peut communiquer avec le serveur de configuration local, connectez-vous à la machine virtuelle. Vérifiez ensuite le service d’application InMage Scout. Si vous voyez qu’il ne s’exécute pas, démarrez le service manuellement. Vérifiez que le type de démarrage du service est défini sur **Automatique**.

### <a name="error-code-78052"></a>Code d'erreur 78052

**Impossible de terminer la protection de la machine virtuelle.**

Ce problème peut se produire si une machine virtuelle portant le même nom se trouve déjà sur le serveur cible principal sur lequel vous effectuez la restauration automatique.

Pour résoudre ce problème :

* Sélectionnez un serveur cible principal situé sur un hôte différent. La reprotection va donc créer la machine sur un autre hôte, ce qui évitera les conflits de noms.
* Vous pouvez également utiliser VMotion pour déplacer la cible principale vers un autre hôte où le conflit de noms ne se produira pas. Si la machine virtuelle existante est une machine isolée, renommez-la pour que la nouvelle machine virtuelle puisse être créée sur le même hôte ESXi.


### <a name="error-code-78093"></a>Code d'erreur 78093

**La machine virtuelle n'est pas en cours d'exécution, ne répond pas ou n'est pas accessible.**

Pour résoudre ce problème :

Pour reprotéger une machine virtuelle basculée, la machine virtuelle Azure doit être exécutée de sorte que le service Mobilité s’inscrive auprès du serveur de configuration local et puisse démarrer la réplication en communiquant avec le serveur de processus. Si la machine ne se trouve pas sur le bon réseau ou n’est pas en cours d’exécution (pas de réponse ou arrêt), le serveur de configuration ne peut pas contacter le service Mobilité sur la machine virtuelle pour démarrer la reprotection.

* Redémarrez la machine virtuelle pour qu’elle puisse recommencer à communiquer localement.
* Redémarrez le travail de reprotection après le démarrage de la machine virtuelle Azure.

### <a name="error-code-8061"></a>Code d’erreur 8061

**La banque de données n’est pas accessible à partir de l’hôte ESXi**.

Pour procéder à la restauration automatique, vérifiez les [composants requis cibles maîtres et les banques de données prises en charge](vmware-azure-prepare-failback.md#deploy-a-separate-master-target-server).


## <a name="troubleshoot-failback-errors"></a>Résoudre les erreurs liées à la restauration automatique

Cette section décrit les erreurs courantes que vous pouvez rencontrer lors de la restauration automatique.

### <a name="error-code-8038"></a>Code d'erreur 8038

**Impossible d'afficher la machine virtuelle locale en raison de l'erreur**

Ce problème se produit quand la machine virtuelle locale est exécutée sur un hôte dont le provisionnement en mémoire est insuffisant. 

Pour résoudre ce problème :

* Provisionnez davantage de mémoire sur l’hôte ESXi.
* De plus, vous pouvez utiliser VMotion pour déplacer la machine virtuelle vers un autre hôte ESXi disposant de suffisamment de mémoire pour démarrer la machine virtuelle.
