---
title: Azure VMware Solution by CloudSimple - Gérer les machines virtuelles du cloud privé dans Azure
description: Décrit comment gérer les machines virtuelles du cloud privé CloudSimple dans le portail Azure, y compris l’ajout de disques, la modification de la capacité des machines virtuelles et l’ajout d’interfaces réseau
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 82aa1ba6ec20e8242568a2725899ddc515a4dc0f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87082067"
---
# <a name="manage-your-cloudsimple-private-cloud-virtual-machines-in-azure"></a>Gérer vos machines virtuelles de cloud privé CloudSimple dans Azure

Pour gérer les machines virtuelles que vous avez [créées pour votre cloud privé CloudSimple](azure-create-vm.md), connectez-vous au [portail Azure](https://portal.azure.com). Recherchez et sélectionnez la machine virtuelle (recherchez sous **Tous les services** ou **Machines virtuelles** dans le menu latéral).

## <a name="control-virtual-machine-operation"></a>Contrôler le fonctionnement de la machine virtuelle

Les contrôles suivants sont disponibles à partir de la page **Vue d’ensemble** pour votre machine virtuelle sélectionnée.

| Control | Description |
| ------------ | ------------- |
| Se connecter | Connectez-vous à la machine virtuelle spécifiée.  |
| Démarrer | Démarre la machine virtuelle spécifiée.  |
| Redémarrer | Arrête puis met hors tension la machine virtuelle spécifiée.  |
| Arrêter | Arrête la machine virtuelle spécifiée.  |
| Capture | Capture une image de la machine virtuelle spécifiée afin de pouvoir l’utiliser en tant qu’image pour créer d’autres machines virtuelles. Consultez [Créer une image managée d’une machine virtuelle généralisée dans Azure](../virtual-machines/windows/capture-image-resource.md).   |
| Déplacer | Déplacez vers la machine virtuelle spécifiée.  |
| DELETE | Supprime la machine virtuelle spécifiée.  |
| Actualiser | Actualise les données dans l’affichage.  |

### <a name="view-performance-information"></a>Afficher les informations de performances

Les graphiques situés dans la partie inférieure de la page **Vue d’ensemble** présentent les données de performances pour l’intervalle sélectionné (de la dernière heure aux 30 derniers jours ; la valeur par défaut est la dernière heure). Dans chaque graphique, vous pouvez afficher les valeurs numériques pour n’importe quel moment dans l’intervalle en déplaçant le curseur sur le graphique.

Les graphiques suivants s’affichent.

| Élément | Description |
| ------------ | ------------- |
| UC (moyenne) | Utilisation moyenne du processeur en pourcentage sur l’intervalle sélectionné.   |
| Réseau | Trafic entrant et sortant du réseau (en Mo) sur l’intervalle sélectionné.  |
| Octets disque | Total des données lues à partir du disque et écrites sur le disque (Mo) au cours de l’intervalle sélectionné.  |
| Opérations disque | Taux moyen d’opérations sur le disque (opérations/seconde) sur l’intervalle sélectionné. |

## <a name="manage-vm-disks"></a>Gérer les disques de machine virtuelle

Pour ajouter un disque de machine virtuelle, ouvrez la page **Disques** de la machine virtuelle sélectionnée. Pour ajouter un disque, cliquez sur **Ajouter un disque**. Configurez chacun des paramètres suivants en entrant ou en sélectionnant une option inline. Cliquez sur **Enregistrer**.

   | Élément | Description |
   | ------------ | ------------- |
   | Nom | Entrez un nom pour identifier le disque.  |
   | Taille | Sélectionnez une des tailles disponibles.  |
   | Contrôleur SCSI | Sélectionnez un contrôleur SCSI. Les contrôleurs disponibles varient en fonction des systèmes d’exploitation pris en charge.  |
   | Mode | Détermine comment le disque participe aux captures instantanées. Choisissez une de ces options : <br> - Indépendant persistant : Toutes les données écrites sur le disque sont écrites de façon permanente.<br> - Indépendant non persistant : Les modifications écrites sur le disque sont abandonnées lors de la mise hors tension ou de la réinitialisation de la machine virtuelle.  Ce mode vous permet de toujours redémarrer la machine virtuelle dans le même état. Pour plus d’informations, consultez la [documentation de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html). |

Pour supprimer un disque, sélectionnez-le, puis cliquez sur **Supprimer**.

## <a name="change-the-capacity-of-the-vm"></a>Modifier la capacité de la machine virtuelle

Pour modifier la capacité de la machine virtuelle, ouvrez la page **Taille** de la machine virtuelle sélectionnée. Spécifiez une des options suivantes, puis cliquez sur **Enregistrer**.

| Élément | Description |
| ------------ | ------------- |
| Nombre de mémoires à tores magnétiques | Nombre de cœurs affectés à la machine virtuelle.  |
| Virtualisation matérielle | Activez la case à cocher pour exposer la virtualisation matérielle au système d’exploitation invité. Consultez l’article VMware [Afficher la virtualisation assistée du matériel VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html). |
| Taille de la mémoire | Sélectionnez la quantité de mémoire à allouer à la machine virtuelle.  

## <a name="manage-network-interfaces"></a>Gérer des interfaces réseau

Pour ajouter une interface, cliquez sur **Ajouter une interface réseau**. Configurez chacun des paramètres suivants en entrant ou en sélectionnant une option inline. Cliquez sur **Enregistrer**.

   | Control | Description |
   | ------------ | ------------- |
   | Nom | Entrez un nom pour identifier l’interface.  |
   | Réseau | Sélectionnez dans la liste des réseaux dans votre cloud privé vSphere.  |
   | Adaptateur | Sélectionnez un adaptateur vSphere dans la liste des types disponibles configurés pour la machine virtuelle. Pour plus d’informations, consultez l’article de la base de connaissances VMware [Choosing a network adapter for your virtual machine](https://kb.vmware.com/s/article/1001805). |
   | Mettre sous tension au démarrage | Choisissez s’il faut activer la carte réseau quand la machine virtuelle est démarrée. La valeur par défaut est **Activer**. |

Pour supprimer une interface réseau, sélectionnez-la, puis cliquez sur **Supprimer**.
