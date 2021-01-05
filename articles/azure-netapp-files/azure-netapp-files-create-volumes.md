---
title: Créer un volume NFS pour Azure NetApp Files | Microsoft Docs
description: Cet article explique comment créer un volume NFS dans Azure NetApp Files. En savoir plus sur les éléments à prendre en compte, notamment la version à utiliser, et les meilleures pratiques.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/24/2020
ms.author: b-juche
ms.openlocfilehash: 2cc9d3e0fb711a0662852ce4f2c5a08dc626f246
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854731"
---
# <a name="create-an-nfs-volume-for-azure-netapp-files"></a>Créer un volume NFS pour Azure NetApp Files

Azure NetApp Files prend en charge la création de volumes avec NFS (NFSv3 et NFSv4.1), SMB3 ou le double protocole (NFSv3 et SMB). La consommation de capacité d’un volume est comptée par rapport à la capacité configurée de son pool. Cet article explique comment créer un volume NFS. 

## <a name="before-you-begin"></a>Avant de commencer 
* Vous devez déjà avoir configuré un pool de capacité.  
    Voir [Configurer un pool de capacité](azure-netapp-files-set-up-capacity-pool.md).   
* Un sous-réseau doit être délégué à Azure NetApp Files.  
    Voir [Déléguer un sous-réseau à Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="considerations"></a>Considérations 

* Choix de la version de NFS à utiliser  
  NFSv3 peut gérer de nombreux cas d’usage et est couramment déployé dans la plupart des applications d’entreprise. Vous devez valider la version (NFSv3 ou NFSv 4.1) requise par votre application et créer votre volume à l’aide de la version appropriée. Par exemple, si vous utilisez [Apache ActiveMQ](https://activemq.apache.org/shared-file-system-master-slave), le verrouillage de fichiers avec NFSv 4.1 est recommandé par rapport à NFSv3. 

* Sécurité  
  La prise en charge des bits en mode UNIX (lecture, écriture et exécution) est disponible pour NFSv3 et NFSv 4.1. Un accès de niveau racine est requis sur le client NFS pour monter des volumes NFS.

* Prise en charge des utilisateurs/groupes locaux et de LDAP pour NFSv 4.1  
  Actuellement, NFSv 4.1 prend en charge l’accès racine aux volumes uniquement. Consultez [Configurer le domaine par défaut NFSv4.1 pour Azure NetApp Files](azure-netapp-files-configure-nfsv41-domain.md). 

## <a name="best-practice"></a>Bonne pratique

* Assurez-vous d’utiliser les instructions de montage appropriées pour le volume.  Voir [Monter ou démonter un volume pour des machines virtuelles Windows ou Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md).

* Le client NFS doit se trouver sur le même réseau virtuel ou réseau virtuel avec peering que le volume Azure NetApp Files. La connexion depuis l’extérieur du réseau virtuel est prise en charge ; toutefois, cela introduira une latence supplémentaire et réduira les performances globales.

* Assurez-vous que le client NFS est à jour et qu’il exécute les mises à jour les plus récentes du système d’exploitation.

## <a name="create-an-nfs-volume"></a>Créer un volume NFS

1.  Cliquez sur le panneau **Volumes** à partir du panneau Pools de capacités. Cliquez sur **+ Ajouter un volume** pour créer un volume. 

    ![Accédez à Volumes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png) 

2.  Dans la fenêtre Créer un volume, cliquez sur **Créer** et fournissez les informations pour les champs suivants sous l’onglet De base :   
    * **Nom du volume**      
        Spécifiez le nom du volume que vous créez.   

        Un nom de volume doit être unique au sein de chaque pool de capacité. Il doit comprendre au moins trois caractères. Vous pouvez utiliser tous les caractères alphanumériques.   

        Vous ne pouvez pas utiliser `default` ni `bin` comme nom de volume.

    * **Pool de capacités**  
        Spécifiez le pool de capacité dans lequel vous souhaitez que le volume soit créé.

    * **Quota**  
        Spécifiez la quantité de stockage logique allouée au volume.  

        Le champ **Quota disponible** indique la quantité d’espace inutilisé dans le pool de capacités choisi, que vous pouvez utiliser pour créer un volume. La taille du nouveau volume ne doit pas dépasser le quota disponible.  

    * **Débit (Mio/s)**    
        Si le volume est créé dans un pool de capacité avec Qualité de service manuelle, spécifiez le débit souhaité pour le volume.   

        Si le volume est créé dans un pool de capacité avec Qualité de service automatique, la valeur affichée dans ce champ est (quota x débit du niveau de service).   

    * **Réseau virtuel**  
        Spécifiez le réseau virtuel Azure (VNet) à partir duquel vous voulez accéder au volume.  

        Le réseau virtuel que vous spécifiez doit avoir un sous-réseau délégué à Azure NetApp Files. Le service Azure NetApp Files est accessible seulement à partir du même réseau virtuel ou d’un sous-réseau qui se trouve dans la même région que le volume via le peering de réseau virtuel. Vous pouvez également accéder au volume à partir de votre réseau local via Express Route.   

    * **Sous-réseau**  
        Spécifiez le sous-réseau que vous souhaitez utiliser pour le volume.  
        Le sous-réseau que vous spécifiez doit être délégué à Azure NetApp Files. 
        
        Si vous n’avez pas délégué un sous-réseau, vous pouvez cliquer sur **Créer** sur la page Créer un volume. Ensuite, dans la page Créer un sous-réseau, fournissez les informations sur le sous-réseau, puis sélectionnez **Microsoft.NetApp/volumes** pour déléguer le sous-réseau à Azure NetApp Files. Dans chaque réseau virtuel, un seul sous-réseau peut être délégué à Azure NetApp Files.   
 
        ![Créer un volume](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Créer un sous-réseau](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si vous souhaitez appliquer une stratégie d’instantané existante au volume, cliquez sur **Afficher la section avancée** pour la développer, indiquez si vous souhaitez masquer le chemin d'accès de l’instantané, puis sélectionnez une stratégie d’instantané dans le menu déroulant. 

        Pour plus d’informations sur la création d’une stratégie d’instantané, consultez [Gérer les stratégies d’instantané](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Afficher la sélection avancée](../media/azure-netapp-files/volume-create-advanced-selection.png)

3. Cliquez sur **Protocole**, puis effectuez les actions suivantes :  
    * Sélectionnez **NFS** comme type de protocole pour le volume.   
    * Spécifiez le **chemin du fichier** à utiliser pour créer le chemin d’exportation du nouveau volume. Le chemin d’exportation est utilisé pour monter le volume et y accéder.

        Le nom du chemin du fichier peut contenir seulement des lettres, des chiffres et des traits d’union (« - »). Il doit comprendre entre 16 et 40 caractères. 

        Le chemin d’accès du fichier doit être unique au sein de chaque abonnement et chaque région. 

    * Sélectionnez la version de NFS (**NFSv3** ou **NFSv4.1**) du volume.  

    * Si vous utilisez NFSv4.1, indiquez si vous souhaitez activer le chiffrement **Kerberos** pour le volume.  

        Des configurations supplémentaires sont requises si vous utilisez Kerberos avec NFSv4.1. Suivez les instructions fournies dans [Configurer le chiffrement Kerberos NFSv4.1](configure-kerberos-encryption.md).

    * Le cas échéant, [configurez une stratégie d’exportation pour le volume NFS](azure-netapp-files-configure-export-policy.md).

    ![Spécifier le protocole NFS](../media/azure-netapp-files/azure-netapp-files-protocol-nfs.png)

4. Cliquez sur **Vérifier et créer** pour passer en revue les informations du volume.  Cliquez ensuite sur **Créer** pour créer le volume.

    Le volume créé s’affiche dans la page Volumes. 
 
    Un volume hérite de l’abonnement, du groupe de ressources et des attributs d’emplacement de son pool de capacité. Vous pouvez suivre l’état du déploiement du volume dans le volet des notifications.


## <a name="next-steps"></a>Étapes suivantes  

* [Configurer le domaine par défaut NFSv4.1 pour Azure NetApp Files](azure-netapp-files-configure-nfsv41-domain.md)
* [Configurer le chiffrement Kerberos NFSv4.1](configure-kerberos-encryption.md)
* [Monter ou démonter un volume pour des machines virtuelles Windows ou Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Configurer une stratégie d’exportation pour un volume NFS](azure-netapp-files-configure-export-policy.md)
* [Limites des ressources pour Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [En savoir plus sur l’intégration d’un réseau virtuel pour les services Azure](../virtual-network/virtual-network-for-azure-services.md)