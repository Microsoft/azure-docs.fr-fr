---
title: Azure VMware Solution by CloudSimple - Migrer des machines virtuelles de charge de travail vers un cloud privé
description: Explique comment migrer des machines virtuelles d’un vCenter local vers un vCenter de cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 87b8a112a319519dbde977ee30136a884137212d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77019993"
---
# <a name="migrate-workload-vms-from-on-premises-vcenter-to-private-cloud-vcenter-environment"></a>Migrer des machines virtuelles de charge de travail d’un environnement vCenter local vers un environnement vCenter de cloud privé

Pour migrer des machines virtuelles d’un centre de données local vers votre cloud privé CloudSimple, plusieurs options sont disponibles.  Le cloud privé fournit un accès natif à VMware vCenter. De plus, les outils pris en charge par VMware peuvent être utilisés pour la migration de la charge de travail. Cet article décrit certaines des options de migration de vCenter.

## <a name="prerequisites"></a>Conditions préalables requises

La migration des machines virtuelles et des données à partir de votre centre de données local nécessite une connectivité réseau entre le centre de données et votre environnement de cloud privé.  Utilisez l’une des méthodes suivantes pour établir la connectivité réseau :

* [Connexion VPN site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre votre environnement local et votre cloud privé
* Connexion ExpressRoute Global Reach entre votre circuit ExpressRoute local et un circuit ExpressRoute CloudSimple

Le chemin réseau entre votre environnement vCenter local et votre cloud privé doit être disponible pour la migration de machines virtuelles à l’aide de vMotion.  Le réseau vMotion de votre vCenter local doit avoir des fonctionnalités de routage.  Vérifiez que votre pare-feu autorise l’ensemble du trafic vMotion entre votre vCenter local et votre vCenter de cloud privé. (Dans le cloud privé, le routage sur le réseau vMotion est configuré par défaut.)

## <a name="migrate-isos-and-templates"></a>Migrer des fichiers ISO et des modèles

Pour créer des machines virtuelles dans votre cloud privé, utilisez des fichiers ISO et des modèles de machine virtuelle.  Pour charger les fichiers ISO et les modèles dans votre vCenter de cloud privé et les rendre disponibles, utilisez la méthode suivante.

1. Chargez le fichier ISO dans le vCenter de cloud privé à partir de l’IU de vCenter.
2. [Publiez une bibliothèque de contenu](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A0F1C13-7336-45CE-B211-610D39A6E1F4.html) sur votre vCenter de cloud privé :

    1. Publiez votre bibliothèque de contenu locale.
    2. Créez une bibliothèque de contenu dans le vCenter de cloud privé.
    3. Abonnez-vous à la bibliothèque de contenu locale publiée.
    4. Synchronisez la bibliothèque de contenu pour permettre l’accès aux contenus faisant l’objet d’un abonnement.

## <a name="migrate-vms-using-powercli"></a>Migrer des machines virtuelles à l’aide de PowerCLI

Pour migrer des machines virtuelles du vCenter local vers le vCenter de cloud privé, utilisez VMware PowerCLI ou l’outil Cross vCenter Workload Migration Utility disponible à partir de VMware Labs.  L’exemple de script suivant illustre les commandes de migration de PowerCLI.

```
$sourceVC = Connect-VIServer -Server <source-vCenter name> -User <source-vCenter user name> -Password <source-vCenter user password>
$targetVC = Connect-VIServer -Server <target-vCenter name> -User <target-vCenter user name> -Password <target-vCenter user password>
$vmhost = <name of ESXi host on destination>
$vm = Get-VM -Server $sourceVC <name of VM>
Move-VM -VM $vm -VMotionPriority High -Destination (Get-VMhost -Server $targetVC -Name $vmhost) -Datastore (Get-Datastore -Server $targetVC -Name <name of tgt vc datastore>)
```

> [!NOTE]
> Pour utiliser les noms des hôtes ESXi et du serveur vCenter de destination, configurez la redirection DNS entre l’environnement local et votre cloud privé.

## <a name="migrate-vms-using-nsx-layer-2-vpn"></a>Migrer des machines virtuelles à l’aide d’un VPN de couche 2 NSX

Cette option permet d’effectuer la migration en temps réel des charges de travail de votre environnement VMware local vers le cloud privé Azure.  Avec ce réseau de couche 2 étendu, le sous-réseau de l’environnement local est disponible dans le cloud privé.  Une fois la migration effectuée, l’affectation de nouvelles adresses IP n’est pas nécessaire pour les machines virtuelles.

[Migrer des charges de travail à l’aide de réseaux étendus de couche 2](migration-layer-2-vpn.md) explique comment utiliser un VPN de couche 2 pour étendre un réseau de couche 2 entre votre environnement local et votre cloud privé.

## <a name="migrate-vms-using-backup-and-disaster-recovery-tools"></a>Migrer des machines virtuelles à l’aide d’outils de sauvegarde et de reprise d’activité après sinistre

Vous pouvez effectuer la migration de machines virtuelles vers un cloud privé à l’aide d’outils de sauvegarde/restauration et d’outils de reprise d’activité après sinistre.  Utilisez le cloud privé en tant que cible pour la restauration des sauvegardes créées à l’aide d’un outil tiers.  Vous pouvez également utiliser le cloud privé en tant que cible pour la reprise d’activité après sinistre à l’aide de VMware SRM ou d’un outil tiers.

Pour plus d’informations sur l’utilisation de ces outils, consultez les rubriques suivantes :

* [Sauvegarder des machines virtuelles de charge de travail sur un cloud privé CloudSimple avec Veeam B&R](backup-workloads-veeam.md)
* [Configurer un cloud privé CloudSimple en tant que site de reprise d’activité après sinistre pour les charges de travail VMware locales](disaster-recovery-zerto.md)
