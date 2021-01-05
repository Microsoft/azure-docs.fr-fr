---
title: Vue d’ensemble du provisionnement Linux
description: Vue d’ensemble de la façon d’intégrer vos images de machine virtuelle Linux ou de créer des images à utiliser dans Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: overview
ms.workload: infrastructure
ms.date: 06/22/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: a7d9aa7de8bb75a22acc85c77924765eaa1b6b3b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87080146"
---
# <a name="azure-linux-vm-provisioning"></a>Provisionnement de machines virtuelles Linux Azure
Lorsque vous créez une machine virtuelle à partir d’une image généralisée (Shared Image Gallery ou image managée), le plan de contrôle vous permet de créer une machine virtuelle et de lui transmettre des paramètres. Ce processus porte le nom de *provisionnement* de machines virtuelles. Pendant le provisionnement, la plateforme met les valeurs de paramètres de création de machine virtuelle nécessaires (nom d’hôte, nom d’utilisateur, mot de passe, clés SSH, customData) à la disposition de la machine virtuelle lors de son démarrage. 

Un agent de provisionnement intégré à l’image interagit avec la plateforme, en se connectant à plusieurs interfaces de provisionnement indépendantes, définit les propriétés et indique à la plateforme qu’il a terminé. 

Les agents de provisionnement peuvent être l’[agent Linux Azure](../extensions/agent-linux.md) ou [cloud-init](./using-cloud-init.md). Ce sont des [prérequis](create-upload-generic.md) pour la création d’images généralisées.

Les agents de provisionnement prennent en charge toutes les [distributions Linux Azure](./endorsed-distros.md) approuvées, et vous trouverez dans de nombreux cas les images des distributions approuvées fournies avec cloud-init et l’agent Linux. Vous pouvez ainsi faire en sorte que cloud-init gère le provisionnement avant que l’agent Linux ne prenne en charge la gestion des [extensions Azure](../extensions/features-windows.md). La prise en charge des extensions signifie que la machine virtuelle sera alors éligible pour prendre en charge d’autres services Azure, tels que la réinitialisation de mot de passe de machine virtuelle, Supervision Azure, Sauvegarde Azure, Azure Disk Encryption, etc.

Une fois le provisionnement terminé, cloud-init s’exécute à chaque démarrage. Cloud-init supervise les modifications apportées à la machine virtuelle, telles que les changements de mise en réseau, le montage et le formatage du disque éphémère, et le démarrage de l’agent Linux. L’agent Linux s’exécute continuellement sur le serveur, en recherchant un « état de l’objectif » (nouvelle configuration) à partir de la plateforme Azure. Par conséquent, chaque fois que vous installez des extensions, l’agent est en mesure de les traiter.

Alors qu’il existe actuellement deux agents de provisionnement, cloud-init doit être l’agent de provisionnement que vous choisissez et l’agent Linux doit être installé pour la prise en charge des extensions. Cela vous permet de tirer parti des optimisations de la plateforme et de désactiver ou de supprimer l’agent Linux. Pour plus d’informations sur la création d’images sans l’agent et sur sa suppression, consultez cette [documentation](disable-provisioning.md).

Si vous avez un noyau Linux qui ne peut pas prendre en charge l’exécution de l’un des agents, mais que vous souhaitez définir certaines propriétés de création de machine virtuelle (nom d’hôte, customData, nom d’utilisateur, mot de passe, clés SSH), ce document explique comment vous pouvez [créer des images généralisées sans un agent](no-agent.md) et répondre aux exigences de la plateforme.


## <a name="provisioning-agent-responsibilities"></a>Responsabilités de l’agent de provisionnement

**Provisionnement d’image**
  
- Création d'un compte d'utilisateur
- Configuration des types d'authentification SSH
- Déploiement des clés publiques et des paires de clés SSH
- Définition du nom d'hôte
- Publication du nom d'hôte sur la plateforme DNS
- Génération de rapports sur l'empreinte digitale de la clé d'hôte SSH destinés à la plateforme
- Gestion du disque de ressources
- Formatage et montage du disque de ressources
- Consommation et traitement de `customData`
 
**Mise en réseau**
  
- Gestion des itinéraires afin d'améliorer la compatibilité avec les serveurs DHCP de plateforme
- Garantie de la stabilité du nom de l'interface réseau

**Noyau**
  
- Configuration de l’architecture NUMA virtuelle (désactivée pour le noyau <`2.6.37`)
- Consommation de l’entropie Hyper-V pour `/dev/random`
- Configuration des délais d'expiration SCSI de l'appareil racine (qui peut être distant)

**Diagnostics**
  
- Redirection de la console vers le port série

## <a name="communication"></a>Communication
Le flux d'informations de la plateforme à l'agent se produit via deux canaux :

- Un DVD attaché au moment du démarrage pour les déploiements IaaS. Le DVD comprend un fichier de configuration compatible OVF qui inclut toutes les informations de provisionnement différentes des paires de clés SSH réelles.
- Un point de terminaison TCP qui expose une API REST utilisée pour obtenir la configuration du déploiement et de la topologie.


## <a name="azure-provisioning-agent-requirements"></a>Exigences de l’agent de provisionnement Azure
L’agent Linux et cloud-init reposent sur certains packages système pour fonctionner correctement :
- Python 2.6+
- OpenSSL 1.0+
- OpenSSH 5.3+
- Utilitaires de système de fichiers : `sfdisk`, `fdisk`, `mkfs`, `parted`
- Outils de mot de passe : chpasswd, sudo
- Outils de traitement de texte : sed, grep
- Outils réseau : ip-route
- Prise en charge du noyau pour le montage de systèmes de fichiers UDF.

## <a name="next-steps"></a>Étapes suivantes

Si nécessaire, vous pouvez [désactiver le provisionnement et supprimer l’agent Linux](disable-provisioning.md).
