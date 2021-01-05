---
title: Préparer Azure VMware Solution à une reprise d’activité après sinistre dans Azure Site Recovery
description: Découvrez comment préparer des serveurs Azure VMware Solution à une reprise d’activité sur Azure à l’aide du service Azure Site Recovery.
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/29/2020
ms.author: harshacs
ms.custom: MVC
ms.openlocfilehash: 8e77ede7b04c95bfd6b6b8f660c8d811e7434c0f
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395442"
---
# <a name="prepare-azure-vmware-solution-for-disaster-recovery-to-azure-site-recovery"></a>Préparer Azure VMware Solution à une reprise d’activité après sinistre dans Azure Site Recovery

Cet article explique comment préparer les serveurs Azure VMware Solution à la reprise d’activité sur Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md). 

Il s’agit du deuxième tutoriel d’une série qui vous montre comment configurer la reprise d’activité sur Azure pour les machines virtuelles Azure VMware Solution. Dans le premier tutoriel, nous [avons configuré les composants Azure](avs-tutorial-prepare-azure.md) nécessaires à la reprise d’activité Azure VMware Solution.


Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Préparer un compte sur le serveur vCenter ou sur l’hôte vSphere ESXi pour automatiser la détection de machines virtuelles.
> * Préparer un compte pour l’installation automatique du service Mobilité sur les machines virtuelles VMware
> * Passer en revue les exigences et la prise en charge du serveur et des machines virtuelles VMware
> * Préparer la connexion aux machines virtuelles Azure après le basculement

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour obtenir des instructions détaillées, consultez l’article de la section Procédures dans la table des matières de Site Recovery.

## <a name="before-you-start"></a>Avant de commencer

Vérifiez que vous avez préparé Azure comme indiqué dans le [premier tutoriel de cette série](avs-tutorial-prepare-azure.md).

## <a name="prepare-an-account-for-automatic-discovery"></a>Préparer un compte pour la découverte automatique

Site Recovery doit pouvoir accéder aux serveurs Azure VMware Solution pour :

- Détecter automatiquement les machines virtuelles. Au moins un compte en lecture seule est nécessaire.
- Orchestrer la réplication, le basculement et la restauration automatique. Vous avez besoin d’un compte pouvant exécuter des opérations telles que la création et la suppression de disques ou le démarrage de machines virtuelles.

Créez le compte comme suit :

1. Pour utiliser un compte dédié, créez un rôle au niveau du vCenter. Donnez un nom au rôle, tel que **Azure_Site_Recovery**.
2. Attribuez au rôle les autorisations résumées dans le tableau ci-dessous.
3. Créez un utilisateur sur le serveur vCenter ou sur l’hôte vSphere. Affectez le rôle à l’utilisateur.

### <a name="vmware-account-permissions"></a>Autorisations du compte VMware

**Tâche** | **Rôle/Autorisations** | **Détails**
--- | --- | ---
**Détection des machines virtuelles** | Au moins un utilisateur en lecture seule<br/><br/> Objet de centre de données -> Propager vers l’objet enfant, rôle = lecture seule | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec l’objet **Propager vers enfant** aux objets enfants (hôtes vSphere, banques de données, machines virtuelles et réseaux).
**Réplication complète, basculement, restauration automatique** |  Créez un rôle (Azure_Site_Recovery) avec les autorisations nécessaires, puis attribuez le rôle à un utilisateur ou à un groupe d’utilisateurs VMware<br/><br/> Objet de centre de données -> Propager vers l’objet enfant, rôle = Azure_Site_Recovery<br/><br/> Banque de données -> Allouer de l’espace, parcourir la banque de données, opérations de fichier de bas niveau, supprimer le fichier, mettre à jour les fichiers de machine virtuelle<br/><br/> Réseau -> Attribution de réseau<br/><br/> Ressource -> Affecter les machines virtuelles au pool de ressources, migrer des machines virtuelles hors tension, migrer des machines virtuelles sous tension<br/><br/> Tâches -> Créer une tâche, Mettre à jour une tâche<br/><br/> Machine virtuelle -> Configuration<br/><br/> Machine virtuelle -> Interagir -> répondre à la question, connexion d’appareil, configurer un support de CD, configurer une disquette, mettre hors tension, mettre sous tension, installation des outils VMware<br/><br/> Machine virtuelle -> Stock -> Créer, inscrire, désinscrire<br/><br/> Machine virtuelle -> Approvisionnement -> Autoriser le téléchargement de machines virtuelles, autoriser le chargement de fichiers de machine virtuelle<br/><br/> Machine virtuelle -> Instantanés -> Supprimer les instantanés | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec l’objet **Propager vers enfant** aux objets enfants (hôtes vSphere, banques de données, machines virtuelles et réseaux).

## <a name="prepare-an-account-for-mobility-service-installation"></a>Préparer un compte pour l’installation du service Mobilité

Le service Mobilité doit être installé sur les machines virtuelles que vous souhaitez répliquer. Site Recovery peut effectuer une installation de type push de ce service lorsque vous activez la réplication pour un ordinateur, ou vous pouvez l’installer manuellement, ou à l’aide des outils d’installation.

- Dans ce tutoriel, nous allons installer le service Mobilité avec l’installation de type push.
- Pour cette installation de type push, vous devez préparer un compte dont Site Recovery pourra se servir pour accéder à la machine virtuelle. Vous devez spécifier ce compte quand vous configurez la récupération d’urgence dans la console Azure.

Préparez le compte comme suit :

Préparez un domaine ou un compte local avec les autorisations nécessaires pour l’installation sur la machine virtuelle.

- **Machines virtuelles Windiows** : Pour une installation sur des machines virtuelles Windows, si vous n’utilisez pas un compte de domaine, désactivez le contrôle d’accès des utilisateurs distants sur la machine locale. Pour ce faire, dans le Registre > **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, ajoutez l’entrée DWORD **LocalAccountTokenFilterPolicy** avec une valeur de 1.
- **Machines virtuelles Linux** : Pour une installation sur des machines virtuelles Linux, préparez un compte racine sur le serveur Linux source.


## <a name="check-vmware-requirements"></a>Vérifier les conditions requises VMware

Vérifiez que les machines virtuelles et serveurs VMware respectent les conditions requises.

1. Vérifiez les [versions logicielles](../azure-vmware/concepts-private-clouds-clusters.md#vmware-software-versions) de la solution VMware sur Azure.
2. Vérifiez la [configuration requise des serveurs VMware](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers).
3. Pour les machines virtuelles Linux, [vérifiez](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) les conditions requises en matière de système de fichiers et de stockage. 
4. Vérifiez la prise en charge du [réseau](vmware-physical-azure-support-matrix.md#network) et du [stockage](vmware-physical-azure-support-matrix.md#storage). 
5. Vérifiez ce qui est pris en charge pour la [mise en réseau Azure](vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover), le [stockage](vmware-physical-azure-support-matrix.md#azure-storage) et le [calcul](vmware-physical-azure-support-matrix.md#azure-compute) après le basculement.
6. Les machines virtuelles Azure VMware Solution que vous répliquez sur Azure doivent respecter la [configuration requise des machines virtuelles Azure](vmware-physical-azure-support-matrix.md#azure-vm-requirements).
7. Sur des machines virtuelles Linux, le nom de l’appareil ou le nom du point de montage doit être unique. Vérifiez que deux appareils/points de montage n’ont pas des noms identiques. Notez que les noms ne respectent pas la casse. Par exemple, il est interdit d’attribuer à deux appareils de la même machine virtuelle les noms _appareil1_ et _Appareil1_.


## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>Préparer la connexion aux machines virtuelles Azure après le basculement

Après le basculement, vous pouvez souhaiter vous connecter à des machines virtuelles Azure à partir de votre réseau Azure VMware Solution.

Pour vous connecter à des machines virtuelles Windows à l’aide de RDP après le basculement, effectuez les opérations suivantes :

- **Accès à Internet**. Avant le basculement, activez RDP sur la machine virtuelle Azure VMware Solution. Vérifiez que des règles TCP et UDP sont ajoutées pour le profil **Public** et que RDP est autorisé dans **Pare-feu Windows** > **Applications autorisées** pour tous les profils.
- **Accès VPN de site à site** :
    - Avant le basculement, activez RDP sur la machine virtuelle Azure VMware Solution.
    - RDP doit être autorisé dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** pour les réseaux **Domaine et Privé**.
    - Vérifiez que la stratégie SAN du système d’exploitation est définie sur **OnlineAll**. [Plus d’informations](https://support.microsoft.com/kb/3031135)
- Aucune mise à jour de Windows ne doit être en attente sur la machine virtuelle quand vous déclenchez un basculement. S’il existe des mises à jour, vous ne pouvez pas vous connecter à la machine virtuelle avant la fin de leur exécution.
- Sur la machine virtuelle Azure Windows, après le basculement, vérifiez les **Diagnostics de démarrage** pour afficher une capture d’écran de la machine virtuelle. Si vous ne pouvez pas vous connecter, vérifiez que la machine virtuelle est en cours d’exécution et lisez ces [conseils de résolution des problèmes](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).

Pour vous connecter à des machines virtuelles Linux à l’aide de SSH après le basculement, effectuez les opérations suivantes :

- Sur la machine Azure VMware Solution, avant le basculement, vérifiez que le service Secure Shell est configuré pour démarrer automatiquement au moment où le système s’amorce.
- Vérifiez que les règles de pare-feu autorisent une connexion SSH.
- Sur la machine virtuelle Azure, après le basculement, autorisez les connexions entrantes au port SSH pour les règles des groupes de sécurité réseau sur la machine virtuelle basculée, et pour le sous-réseau Azure auquel elle est connectée.
- [Ajoutez une adresse IP publique](./site-recovery-monitor-and-troubleshoot.md) pour la machine virtuelle.
- Vous pouvez vérifier les **Diagnostics de démarrage** pour afficher une capture d’écran de la machine virtuelle.


## <a name="failback-requirements"></a>Conditions requises pour la restauration automatique
Si vous prévoyez d’effectuer une restauration automatique sur votre cloud Azure VMware Solution, vous devez respecter plusieurs [prérequis pour la restauration automatique](avs-tutorial-reprotect.md#before-you-begin). Vous pouvez vous y préparer dès maintenant, mais vous n’y êtes pas obligé. Vous pouvez vous y préparer après avoir effectué le basculement sur Azure.




## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer la reprise d’activité](avs-tutorial-replication.md)
- Si vous répliquez plusieurs machines virtuelles, [procédez à une planification de la capacité](site-recovery-deployment-planner.md).
