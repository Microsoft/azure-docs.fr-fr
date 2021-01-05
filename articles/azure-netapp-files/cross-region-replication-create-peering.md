---
title: Créer une réplication de volume pour Azure NetApp Files | Microsoft Docs
description: Décrit comment créer un peering de réplication de volume pour Azure NetApp Files afin de configurer la réplication inter-région.
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
ms.date: 09/16/2020
ms.author: b-juche
ms.openlocfilehash: 412724a072bfc03a67bf1005173702f757c6fdf4
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95249935"
---
# <a name="create-volume-replication-for-azure-netapp-files"></a>Créer une réplication de volume pour Azure NetApp Files

> [!IMPORTANT]
> La fonctionnalité de réplication inter-région est actuellement disponible en préversion publique. Vous devez soumettre une demande d’inscription à la liste d’attente pour accéder à la fonctionnalité via la [page de soumission de demande d’inscription à la liste d’attente de réplication inter-région Azure NetApp Files](https://aka.ms/anfcrrpreviewsignup). Attendez de recevoir un e-mail de confirmation officiel de l’équipe Azure NetApp Files avant d’utiliser la fonctionnalité de réplication inter-région.

Cet article explique comment configurer la réplication inter-région en créant un peering de réplication. 

La configuration d’un peering de réplication vous permet de répliquer de façon asynchrone des données à partir d’un volume Azure NetApp Files (source) vers un autre volume Azure NetApp Files (destination). Le volume source et le volume de destination doivent être déployés dans des régions distinctes. Le niveau de service du pool de capacité de destination peut correspondre à celui du pool de capacité source, ou vous pouvez sélectionner un niveau de service différent.   

La réplication Azure NetApp Files ne prend actuellement pas en charge plusieurs abonnements. Toutes les réplications doivent être effectuées sous un même abonnement.

Avant de commencer, veillez à passer en revue les [exigences et considérations relatives à l’utilisation de la réplication inter-région](cross-region-replication-requirements-considerations.md).  

## <a name="locate-the-source-volume-resource-id"></a>Rechercher l’ID de ressource du volume source  

Vous devez obtenir l’ID de ressource du volume source que vous souhaitez répliquer. 

1. Accédez au volume source, puis sélectionnez **Propriétés** sous Paramètres pour afficher l’ID de ressource du volume source.   
    ![Rechercher l’ID de ressource du volume source](../media/azure-netapp-files/cross-region-replication-source-volume-resource-id.png)
 
2. Copiez l’ID de ressource dans le Presse-papiers.  Vous en aurez besoin ultérieurement.

## <a name="create-the-data-replication-volume-the-destination-volume"></a>Créer le volume de réplication de données (volume de destination)

Vous devez créer un volume de destination sur lequel vous souhaitez répliquer les données du volume source.  Avant de pouvoir créer un volume de destination, vous devez disposer d’un compte NetApp et d’un pool de capacité dans la région de destination. 

1. Le compte de destination doit se trouver dans une région différente du volume source. Si nécessaire, créez un compte NetApp dans la région Azure à utiliser pour la réplication en effectuant les étapes décrites dans [Créer un compte NetApp](azure-netapp-files-create-netapp-account.md).   
Vous pouvez également sélectionner un compte NetApp existant dans une autre région.  

2. Si nécessaire, créez un pool de capacité dans le nouveau compte NetApp en effectuant les étapes décrites dans [Configurer un pool de capacité](azure-netapp-files-set-up-capacity-pool.md).   

    Vous pouvez également sélectionner un pool de capacité existant pour héberger le volume de destination de réplication.  

    Le niveau de service du pool de capacité de destination peut correspondre à celui du pool de capacité source, ou vous pouvez sélectionner un niveau de service différent.

3. Déléguez un sous-réseau dans la région à utiliser pour la réplication en effectuant les étapes décrites dans [Déléguer un sous-réseau à Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

4. Créez le volume de réplication de données en sélectionnant **Volumes** sous Service de stockage dans le compte NetApp de destination. Cliquez ensuite sur le bouton **+ Ajouter la réplication des données**.  

    ![Ajouter la réplication des données](../media/azure-netapp-files/cross-region-replication-add-data-replication.png)
 
5. Dans la page Créer un volume qui s’affiche, renseignez les champs suivants sous l’onglet **Informations de base** :
    * Nom du volume
    * Pool de capacités
    * Quota du volume
        > [!NOTE] 
        > Nous vous recommandons de faire en sorte que la taille du quota de volume reflète celle du volume source.
    * Réseau virtuel 
    * Subnet

    Pour plus d’informations sur les champs, consultez [Créer un volume NFS](azure-netapp-files-create-volumes.md#create-an-nfs-volume). 

6. Sous l’onglet **Protocole**, sélectionnez le même protocole que le volume source.  
Pour le protocole NFS, vérifiez que les règles de stratégie d’exportation répondent aux exigences des hôtes du réseau distant qui accèderont à l’exportation.  

7. Sous l’onglet **Étiquettes**, créez des paires clé/valeur si nécessaire.  

8. Sous l’onglet **Réplication**, collez l’ID de ressource du volume source que vous avez obtenu dans [Rechercher l’ID de ressource du volume source](#locate-the-source-volume-resource-id), puis sélectionnez la planification de réplication souhaitée. Les options de planification de réplication sont les suivantes : toutes les 10 minutes, toutes les heures, tous les jours, toutes les semaines ou tous les mois.  

    ![Créer une réplication de volume](../media/azure-netapp-files/cross-region-replication-create-volume-replication.png)

9. Cliquez sur **Vérifier + créer**, puis sur **Créer** pour créer le volume de réplication de données.   

    ![Vérifier et créer une réplication](../media/azure-netapp-files/cross-region-replication-review-create-replication.png)

## <a name="authorize-replication-from-the-source-volume"></a>Autoriser la réplication à partir du volume source  

Pour autoriser la réplication, vous devez obtenir l’ID de ressource du volume de destination de réplication et le coller dans le champ Autoriser du volume source de réplication. 

1. Dans le portail Azure, accédez à Azure NetApp Files.

2. Accédez au compte NetApp de destination et au pool de capacité de destination où se trouve le volume de destination de réplication.

3. Sélectionnez le volume de destination de réplication, accédez à **Propriétés** sous Paramètres, puis recherchez l’**ID de ressource** du volume de destination. Copiez l’ID de ressource du volume de destination dans le Presse-papiers.

    ![Propriétés : ID de ressource](../media/azure-netapp-files/cross-region-replication-properties-resource-id.png) 
 
4. Dans Azure NetApp Files, accédez au compte source de réplication et au pool de capacité source. 

5. Recherchez le volume source de réplication et sélectionnez-le. Accédez à **Réplication** sous Service de stockage, puis cliquez sur **Autoriser**.

    ![Autoriser la réplication](../media/azure-netapp-files/cross-region-replication-authorize.png) 

6. Dans le champ Autoriser, collez l’ID de ressource du volume de réplication de destination obtenu à l’étape 3, puis cliquez sur **OK**.

## <a name="next-steps"></a>Étapes suivantes  

* [Réplication entre régions](cross-region-replication-introduction.md)
* [Configuration requise et considérations pour la réplication entre régions](cross-region-replication-requirements-considerations.md)
* [Afficher l’état d’intégrité de la relation de réplication](cross-region-replication-display-health-status.md)
* [Métriques de réplication de volume](azure-netapp-files-metrics.md#replication)
* [Gérer la reprise d’activité après sinistre](cross-region-replication-manage-disaster-recovery.md)
* [Supprimer des volumes ou des réplications de volume](cross-region-replication-delete.md)
* [Résoudre les problèmes de réplication entre régions](troubleshoot-cross-region-replication.md)

