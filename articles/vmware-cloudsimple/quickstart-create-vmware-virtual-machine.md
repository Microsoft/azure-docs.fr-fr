---
title: 'Démarrage rapide : Utiliser des machines virtuelles VMware sur Azure'
titleSuffix: Azure VMware Solution by CloudSimple
description: Découvrez comment configurer et utiliser des machines virtuelles VMware à partir du portail Azure en utilisant Azure VMware Solution by CloudSimple
author: sharaths-cs
ms.author: dikamath
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4ab613c251bc43a025e0381046805ec998a04227
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77019551"
---
# <a name="quickstart---consume-vmware-vms-on-azure"></a>Démarrage rapide – Utiliser des machines virtuelles VMware sur Azure

Pour créer une machine virtuelle sur le portail Azure, utilisez les modèles d’ordinateurs virtuels que votre administrateur CloudSimple a activé pour votre abonnement. Les modèles de machine virtuelle se trouvent dans l’infrastructure VMware.

## <a name="cloudsimple-vm-creation-on-azure-requires-a-vm-template"></a>La création d’une machine virtuelle CloudSimple sur Azure nécessite un modèle d’ordinateur virtuel

Créez une machine virtuelle sur votre cloud privé à partir de l’interface utilisateur de vCenter. Pour créer un modèle, suivez les instructions de la page [Cloner une machine virtuelle sur un modèle dans le client web vSphere](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-FE6DE4DF-FAD0-4BB0-A1FD-AFE9A40F4BFE.html). Stockez le modèle de machine virtuelle sur votre vCenter de cloud privé.

## <a name="create-a-virtual-machine-in-the-azure-portal"></a>Créer une machine virtuelle dans le portail Azure

1. Sélectionnez **Tous les services**.

2. Recherchez **Machines virtuelles CloudSimple**.

3. Cliquez sur **Add**.

    ![Créer une machine virtuelle CloudSimple](media/create-cloudsimple-virtual-machine.png)

4. Entrez les informations de base, puis cliquez sur **Suivant : taille**.

    ![Créer une machine virtuelle CloudSimple – Principes de base](media/create-cloudsimple-virtual-machine-basic-info.png)

    | Champ | Description |
    | ------------ | ------------- |
    | Abonnement | Abonnement Azure associé à votre cloud privé.  |
    | Groupe de ressources | Groupe de ressources auquel la machine virtuelle sera attribuée. Vous pouvez sélectionner un groupe existant ou en créer un. |
    | Nom | Nom destiné à identifier la machine virtuelle.  |
    | Emplacement | Région Azure dans laquelle la machine virtuelle est hébergée.  |
    | Cloud privé | Cloud privé CloudSimple où vous voulez créer la machine virtuelle. |
    | Liste de ressources partagées | Liste de ressources partagées mappée pour la machine virtuelle. Choisissez parmi les pools de ressources disponibles. |
    | Modèle vSphere | Modèle vSphere pour la machine virtuelle.  |
    | Nom d'utilisateur | Nom d’utilisateur de l’administrateur de la machine virtuelle (pour les modèles Windows).|
    | Mot de passe |  Mot de passe de l’administrateur de la machine virtuelle (pour les modèles Windows). |
    | Confirmer le mot de passe | Confirmez le mot de passe. |

5. Sélectionnez le nombre de cœurs et la capacité mémoire pour la machine virtuelle et cliquez sur **Suivant :Configurations**. Sélectionnez la case à cocher si vous voulez exposer la virtualisation complète de l’UC au système d’exploitation invité. Les applications nécessitant la virtualisation matérielle peuvent fonctionner sur des machines virtuelles sans traduction binaire ou paravirtualisation. Pour plus d’informations, consultez l’article VMware <a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html" target="_blank">Afficher la virtualisation assistée du matériel VMware</a>.

    ![Créer une machine virtuelle CloudSimple – taille](media/create-cloudsimple-virtual-machine-size.png)

6. Configurez les interfaces réseau et les disques comme décrit dans les tableaux suivants, puis cliquez sur **Vérifier + créer**.

    ![Créer une machine virtuelle CloudSimple – configurations](media/create-cloudsimple-virtual-machine-configurations.png)

    Pour les interfaces réseau, cliquez sur **Ajouter une interface réseau**, puis configurez les paramètres suivants.

    | Control | Description |
    | ------------ | ------------- |
    | Nom | Entrez un nom pour identifier l’interface.  |
    | Réseau | Sélectionnez dans la liste des groupes de ports distribués configurés dans votre cloud privé vSphere.  |
    | Adaptateur | Sélectionnez un adaptateur vSphere dans la liste des types disponibles configurés pour la machine virtuelle. Pour plus d’informations, consultez l’article de la base de connaissances VMware <a href="https://kb.vmware.com/s/article/1001805" target="_blank">Choosing a network adapter for your virtual machine</a>. |
    | Mettre sous tension au démarrage | Choisissez s’il faut activer la carte réseau quand la machine virtuelle est démarrée. La valeur par défaut est **Activer**. |

    Pour les disques, cliquez sur **Ajouter un disque**, puis configurez les paramètres suivants.

    | Élément | Description |
    | ------------ | ------------- |
    | Nom | Entrez un nom pour identifier le disque.  |
    | Taille | Sélectionnez une des tailles disponibles.  |
    | Contrôleur SCSI | Sélectionnez un contrôleur SCSI pour le disque.  |
    | Mode | Détermine comment le disque participe aux captures instantanées. Choisissez une de ces options : <br> - Indépendant persistant : Toutes les données écrites sur le disque sont écrites de façon permanente.<br> - Indépendant non persistant : Les modifications écrites sur le disque sont abandonnées lors de la mise hors tension ou de la réinitialisation de la machine virtuelle.  Le mode indépendant non persistant vous permet de toujours redémarrer la machine virtuelle dans le même état. Pour plus d’informations, consultez la <a href="https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html" target="_blank">documentation de VMware</a>.

7. Une fois la validation terminée, passez en revue les paramètres et cliquez sur **Créer**. Pour apporter des modifications, cliquez sur les onglets situés en haut.

    ![Créer une machine virtuelle CloudSimple – vérifier](media/create-cloudsimple-virtual-machine-review.png)

## <a name="next-steps"></a>Étapes suivantes

* [Afficher une liste des machines virtuelles CloudSimple](azure-create-vm.md#view-list-of-cloudsimple-virtual-machines)
* [Gérer des machines virtuelles CloudSimple à partir d’Azure](azure-manage-vm.md)
