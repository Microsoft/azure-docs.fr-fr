---
title: Gérer Azure Data Box, Azure Data Box Heavy par le biais du Portail Azure| Microsoft Docs
description: Décrit comment utiliser le Portail Azure pour administrer votre Azure Data Box et Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/20/2020
ms.author: alkohli
ms.openlocfilehash: 7d2734d386f1e49e2227058b148ee6b591d14a42
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94336426"
---
# <a name="use-the-azure-portal-to-administer-your-azure-data-box-and-azure-data-box-heavy"></a>Utiliser le Portail Azure pour administrer votre Azure Data Box et Azure Data Box Heavy

Cet article s’applique à Azure Data Box et à Azure Data Box Heavy. Cet article décrit certains des flux de travail et tâches de gestion complexes qui peuvent être effectués sur l’appareil Azure Data Box. Vous pouvez gérer l’appareil Data Box via le portail Azure ou via l’interface utilisateur web locale.

Cet article se concentre sur les tâches que vous pouvez effectuer à l’aide du portail Azure. Utilisez le Portail Azure pour gérer les commandes et l’appareil Data Box, et suivre l’état des commandes jusqu’à l’étape terminale.

[!INCLUDE [Data Box feature is in preview](../../includes/data-box-feature-is-preview-info.md)]


## <a name="cancel-an-order"></a>Annuler une commande

Il peut vous arriver de devoir annuler, pour diverses raisons, une commande que vous avez passée. 

Pour les commandes d’importation et d’exportation, vous ne pouvez annuler la commande qu’avant son traitement. Une fois la commande traitée et l’appareil Data Box préparé, il n’est plus possible d’annuler la commande.

Pour annuler une commande, procédez comme suit.

1.  Accédez à **Vue d’ensemble > Annuler**.

    ![Commande Annuler sous l’onglet Vue d’ensemble d’une commande](media/data-box-portal-admin/portal-admin-cancel-command.png)

2.  Indiquez un motif d’annulation de la commande.  

    ![Boîte de dialogue Annuler la commande](media/data-box-portal-admin/portal-admin-cancel-order-dbox.png)

3.  Une fois que la commande est annulée, le portail met à jour l’état de la commande et l’affiche en tant que **Annulée**.

## <a name="clone-an-order"></a>Cloner une commande

Le clonage est utile dans certaines situations. Par exemple, un utilisateur a utilisé Data Box pour transférer des données. À mesure qu’il génère de nouvelles données, il a besoin d’un autre appareil Data Box pour transférer ces données vers Azure. Dans ce cas, la commande initiale peut simplement être clonée.

> [!IMPORTANT]
> Le clonage n’est pas disponible pour les commandes d’exportation. Vous ne pouvez cloner que les commandes d’importation.

Effectuez les opérations suivantes pour cloner une commande d’importation.

1.  Accédez à **Vue d’ensemble > Cloner**. 

    ![Commande Cloner sous l’onglet Vue d’ensemble d’une commande](media/data-box-portal-admin/portal-admin-clone-command.png)

2.  Tous les détails de la commande restent inchangés. Le nom de la commande est le nom de la commande d’origine, suivi de *-Clone*. Cochez la case pour confirmer que vous avez lu les informations sur la confidentialité des données. Cliquez sur **Créer**.

Le clone est créé en quelques minutes et le portail est actualisé pour afficher la nouvelle commande.


## <a name="delete-order"></a>Supprimer une commande

Il peut arriver que vous souhaitiez supprimer une commande lorsque celle-ci est terminée. La commande contient vos données personnelles telles que votre nom, votre adresse et vos informations de contact. Ces informations personnelles sont supprimées en même temps que la commande.

Vous pouvez uniquement supprimer des commandes qui sont terminées ou annulées. Pour supprimer une commande, procédez comme suit.

1. Sélectionnez **Toutes les ressources**. Recherchez votre commande.

2. Cliquez sur la commande que vous souhaitez supprimer et accédez à **Vue d’ensemble**. Dans la barre de commandes, cliquez sur **Supprimer**.

    ![Commande Supprimer dans l’onglet Vue d’ensemble d’une commande](media/data-box-portal-admin/portal-admin-delete-command.png)

3. Entrez le nom de la commande lorsque vous êtes invité à confirmer la suppression de la commande. Cliquez sur **Supprimer**.

## <a name="download-shipping-label"></a>Télécharger une étiquette d’expédition

Vous devrez peut-être télécharger l’étiquette d’expédition si l’écran E-ink de votre Data Box ne fonctionne pas et n’affiche pas l’étiquette de retour. Ce flux de travail ne s’applique pas aux Data Box Heavy, car ces appareils ne possèdent pas d’écran E-ink.

Procédez comme suit pour télécharger une étiquette d’expédition.

1.  Accédez à **Vue d’ensemble > Télécharger une étiquette d’expédition**. Cette option est disponible uniquement une fois que l’appareil a été expédié. 

    ![Télécharger une étiquette d’expédition](media/data-box-portal-admin/portal-admin-download-shipping-label.png)

2.  L’étiquette de retour suivante est alors téléchargée. Enregistrez puis imprimez l’étiquette. Pliez et insérez l’étiquette dans la pochette transparente accolée à l’appareil. Vérifiez que l’étiquette est visible. Retirez les éventuels autocollants qui ont été accolés à l’appareil lors de la précédente expédition.

    ![Exemple d’étiquette d’expédition](media/data-box-portal-admin/portal-admin-example-shipping-label.png)

## <a name="edit-shipping-address"></a>Modifier l’adresse de livraison

Il peut arriver que vous deviez modifier l’adresse de livraison une fois la commande passée. Cela est possible uniquement avant l’envoi de l’appareil. Une fois l’appareil expédié, vous ne pouvez plus modifier l’adresse de livraison.

Pour modifier la commande, procédez comme suit.

1. Accédez à **Détails de la commande > Modifier l’adresse de livraison**.

    ![Commande Modifier l’adresse d’expédition dans Détails de la commande](media/data-box-portal-admin/portal-admin-edit-shipping-address-command.png)

2. Modifiez et validez l’adresse de livraison, puis enregistrez les modifications.

    ![Boîte de dialogue Modifier l’adresse d’expédition](media/data-box-portal-admin/portal-admin-edit-shipping-address-dbox.png)

## <a name="edit-notification-details"></a>Modifier les détails de notification

Vous devrez peut-être modifier les utilisateurs qui recevront les e-mails indiquant l’état de la commande. Par exemple, un utilisateur doit être informé dès que l’appareil est livré ou récupéré. Un autre utilisateur doit quant à lui être informé lorsque la copie des données est terminée afin qu’il puisse vérifier que les données se trouvent bien dans le compte de stockage Azure avant de les supprimer de la source. Dans ce cas, vous pouvez modifier les détails de notification.

Pour modifier les détails de modification, procédez comme suit.

1. Accédez à **Détails de la commande > Modifier les détails de notification**.

    ![Commande Modifier les détails de notification dans Détails de la commande](media/data-box-portal-admin/portal-admin-edit-notification-details-command.png)

2. Vous pouvez maintenant modifier les détails de notification et enregistrer les modifications.
 
    ![Boîte de dialogue Modifier les détails de notification](media/data-box-portal-admin/portal-admin-edit-notification-details-dbox.png)


## <a name="download-order-history"></a>Télécharger l’historique des commandes

Une fois que la commande Data Box est terminée, les données sur les disques de l’appareil sont effacées. Lorsque le nettoyage de l’appareil est terminé, vous pouvez télécharger l’historique des commandes dans le portail Azure.

Procédez comme suit pour télécharger l’historique des commandes.

1. Dans votre commande Data Box, accédez à **Vue d’ensemble**. Vérifiez que la commande est terminée. Si la commande est terminée ainsi que le nettoyage de l’appareil, accédez à **Order details** (Détails de la commande). L’option **Download order history** (Télécharger l’historique des commandes) est disponible.

    ![Télécharger l’historique des commandes](media/data-box-portal-admin/portal-admin-download-order-history.png)

2. Cliquez sur **Télécharger l’historique des commandes**. Dans l’historique téléchargé, vous verrez un enregistrement des journaux d’activité de suivi de l’opérateur. Il existe deux ensembles de journaux correspondant aux deux nœuds présents sur un Data Box Heavy. Si vous faites défiler jusqu’en bas de ce journal, vous pouvez voir des liens vers :
    
   - **Copier les journaux d’activité** - affiche la liste des fichiers en erreur pendant la copie des données entre la zone de données et votre compte de stockage Azure (commande d’importation) ou *vice-versa* (commande d’exportation).
   - **Journaux d’audit** - contiennent des informations sur la façon de mettre sous tension et d’accéder aux partages sur la Data Box lorsqu’elle est hors du centre de données Azure.
   - **Fichiers de nomenclature dans une commande d’importation** - obtenez la liste des fichiers (également connue sous le nom de manifeste de fichier) que vous pouvez télécharger pendant **Préparer l’expédition** et qui comporte les noms, les tailles et les sommes de contrôle des fichiers.
   - **Journaux détaillés dans une commande d’exportation**- obtenez la liste des fichiers avec le nom et la taille des tailles des fichiers ainsi qu’un calcul de somme de contrôle lorsque les données ont été copiées à partir des comptes de stockage Azure vers le Data Box.

   Voici un exemple d’historique de commande à partir d’une commande d’importation.

    ```output
    -------------------------------
    Microsoft Data Box Order Report
    -------------------------------
    Name                                               : DataBoxTestOrder                              
    StartTime(UTC)                                     : 10/31/2018 8:49:23 AM +00:00                       
    DeviceType                                         : DataBox                                           
    -------------------
    Data Box Activities
    -------------------
    Time(UTC)                 | Activity                       | Status          | Description  
    
    10/31/2018 8:49:26 AM     | OrderCreated                   | Completed       |                                                   
    11/2/2018 7:32:53 AM      | DevicePrepared                 | Completed       |                                                   
    11/3/2018 1:36:43 PM      | ShippingToCustomer             | InProgress      | Shipment picked up. Local Time : 11/3/2018 1:36:43        PM at AMSTERDAM-NLD                                                                                
    11/4/2018 8:23:30 PM      | ShippingToCustomer             | InProgress      | Processed at AMSTERDAM-NLD. Local Time : 11/4/2018        8:23:30 PM at AMSTERDAM-NLD                                                                        
    11/4/2018 11:43:34 PM     | ShippingToCustomer             | InProgress      | Departed Facility in AMSTERDAM-NLD. Local Time :          11/4/2018 11:43:34 PM at AMSTERDAM-NLD                                                               
    11/5/2018 1:38:20 AM      | ShippingToCustomer             | InProgress      | Arrived at Sort Facility LEIPZIG-DEU. Local Time :        11/5/2018 1:38:20 AM at LEIPZIG-DEU                                                                
    11/5/2018 2:31:07 AM      | ShippingToCustomer             | InProgress      | Processed at LEIPZIG-DEU. Local Time : 11/5/2018          2:31:07 AM at LEIPZIG-DEU                                                                            
    11/5/2018 4:05:58 AM      | ShippingToCustomer             | InProgress      | Departed Facility in LEIPZIG-DEU. Local Time :            11/5/2018 4:05:58 AM at LEIPZIG-DEU                                                                    
    11/5/2018 4:35:43 AM      | ShippingToCustomer             | InProgress      | Transferred through LUTON-GBR. Local Time :              11/5/2018 4:35:43 AM at LUTON-GBR                                                                         
    11/5/2018 4:52:15 AM      | ShippingToCustomer             | InProgress      | Departed Facility in LUTON-GBR. Local Time :              11/5/2018 4:52:15 AM at LUTON-GBR                                                                        
    11/5/2018 5:47:58 AM      | ShippingToCustomer             | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR.            Local Time : 10/5/2018 5:47:58 AM at LONDON-HEATHROW-GBR                                                
    11/5/2018 6:27:37 AM      | ShippingToCustomer             | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time :            11/5/2018 6:27:37 AM at LONDON-HEATHROW-GBR                                                            
    11/5/2018 6:39:40 AM      | ShippingToCustomer             | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local          Time : 11/5/2018 6:39:40 AM at LONDON-HEATHROW-GBR                                                    
    11/5/2018 8:13:49 AM      | ShippingToCustomer             | InProgress      | Arrived at Delivery Facility in LAMBETH-GBR. Local        Time : 11/5/2018 8:13:49 AM at LAMBETH-GBR                                                         
    11/5/2018 9:13:24 AM      | ShippingToCustomer             | InProgress      | With delivery courier. Local Time : 11/5/2018            9:13:24 AM at LAMBETH-GBR                                                                               
    11/5/2018 12:03:04 PM     | ShippingToCustomer             | Completed       | Delivered - Signed for by. Local Time : 11/5/2018        12:03:04 PM at LAMBETH-GBR                                                                          
    1/25/2019 3:19:25 PM      | ShippingToDataCenter           | InProgress      | Shipment picked up. Local Time : 1/25/2019 3:19:25        PM at LAMBETH-GBR                                                                                       
    1/25/2019 8:03:55 PM      | ShippingToDataCenter           | InProgress      | Processed at LAMBETH-GBR. Local Time : 1/25/2019          8:03:55 PM at LAMBETH-GBR                                                                            
    1/25/2019 8:04:58 PM      | ShippingToDataCenter           | InProgress      | Departed Facility in LAMBETH-GBR. Local Time :            1/25/2019 8:04:58 PM at LAMBETH-GBR                                                                    
    1/25/2019 9:06:09 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR.            Local Time : 1/25/2019 9:06:09 PM at LONDON-HEATHROW-GBR                                                
    1/25/2019 9:48:54 PM      | ShippingToDataCenter           | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time :            1/25/2019 9:48:54 PM at LONDON-HEATHROW-GBR                                                            
    1/25/2019 10:30:20 PM     | ShippingToDataCenter           | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local          Time : 1/25/2019 10:30:20 PM at LONDON-HEATHROW-GBR                                                   
    1/26/2019 2:17:10 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility BRUSSELS-BEL. Local Time        : 1/26/2019 2:17:10 PM at BRUSSELS-BEL                                                              
    1/26/2019 2:31:57 PM      | ShippingToDataCenter           | InProgress      | Processed at BRUSSELS-BEL. Local Time : 1/26/2019        2:31:57 PM at BRUSSELS-BEL                                                                          
    1/26/2019 3:37:53 PM      | ShippingToDataCenter           | InProgress      | Processed at BRUSSELS-BEL. Local Time : 1/26/2019        3:37:53 PM at BRUSSELS-BEL                                                                          
    1/27/2019 11:01:45 AM     | ShippingToDataCenter           | InProgress      | Departed Facility in BRUSSELS-BEL. Local Time :          1/27/2019 11:01:45 AM at BRUSSELS-BEL                                                                 
    1/28/2019 7:11:35 AM      | ShippingToDataCenter           | InProgress      | Arrived at Delivery Facility in AMSTERDAM-NLD.            Local Time : 1/28/2019 7:11:35 AM at AMSTERDAM-NLD                                                     
    1/28/2019 9:07:57 AM      | ShippingToDataCenter           | InProgress      | With delivery courier. Local Time : 1/28/2019            9:07:57 AM at AMSTERDAM-NLD                                                                             
    1/28/2019 1:35:56 PM      | ShippingToDataCenter           | InProgress      | Scheduled for delivery. Local Time : 1/28/2019            1:35:56 PM at AMSTERDAM-NLD                                                                            
    1/28/2019 2:57:48 PM      | ShippingToDataCenter           | Completed       | Delivered - Signed for by. Local Time : 1/28/2019        2:57:48 PM at AMSTERDAM-NLD                                                                         
    1/29/2019 2:18:43 PM      | PhysicalVerification           | Completed       |                                              
    1/29/2019 3:49:50 PM      | DeviceBoot                     | Completed       | Appliance booted up successfully                  
    1/29/2019 3:49:51 PM      | AnomalyDetection               | Completed       | No anomaly detected.                               
    1/29/2019 4:55:00 PM      | DataCopy                       | Started         |                                                 
    2/2/2019 7:07:34 PM       | DataCopy                       | Completed       | Copy Completed.                                   
    2/4/2019 7:47:32 PM       | SecureErase                    | Started         |                                                  
    2/4/2019 8:01:10 PM       | SecureErase                    | Completed       | Azure Data Box:DEVICESERIALNO has been sanitized          according to NIST 800-88 Rev 1.                                                                       

    ------------------
    Data Box Log Links
    ------------------

    Account Name         : Gus                                                       
    Copy Logs Path       : databoxcopylog/DataBoxTestOrder_CHC533180024_CopyLog_73a81b2d613547a28ecb7b1612fe93ca.xml
    Audit Logs Path      : azuredatabox-chainofcustodylogs\7fc6cac9-9cd6-4dd8-ae22-1ce479666282\chc533180024
    BOM Files Path       : azuredatabox-chainofcustodylogs\7fc6cac9-9cd6-4dd8-ae22-1ce479666282\chc533180024      
    ```

    Vous pouvez ensuite accéder à votre compte de stockage, puis afficher les journaux d’activité de copie.

   ![Journaux de copie pour un compte de stockage](media/data-box-portal-admin/portal-admin-storage-account-copy-logs.png)

   Vous pouvez également afficher les journaux d’activité de chaîne de responsabilité qui incluent les journaux d’audit et les fichiers de nomenclature.

   ![Journaux d’activité de chaîne de responsabilité pour un compte de stockage](media/data-box-portal-admin/portal-admin-storage-account-chain-of-custody-logs.png)

## <a name="view-order-status"></a>Afficher l’état de la commande

Vous êtes informé par courrier électronique dès que l’état de l’appareil change dans le portail.

### <a name="statuses-for-import-order"></a>États des commandes d’importation

Voici les états d’une commande d’importation.

|État de la commande |Description |
|---------|---------|
|Ordered (Validée)     | La commande a été validée. <br>Si l’appareil est disponible, Microsoft identifie l’appareil à expédier et le prépare. <br> Si l’appareil n’est pas disponible immédiatement, la commande sera traitée lorsque l’appareil devient disponible. Le traitement de la commande peut prendre quelques jours à deux mois. Si elle ne peut pas être traitée sous 90 jours, la commande est annulée et vous en êtes informé.         |
|Processed (Traitée)     | La commande a été traitée. Conformément à votre commande, l’appareil est prêt pour expédition dans le centre de données.         |
|Dispatched (Distribuée)     | La commande a été expédiée. Utilisez l’ID de suivi affiché sur votre commande dans le portail pour effectuer le suivi de l’expédition.        |
|Delivered (Livrée)     | Le colis a été livré à l’adresse spécifiée dans la commande.        |
|Picked up (Récupérée)     |Le colis de retour a été récupéré et scanné par le transporteur.         |
|Reçu     | Votre appareil a été reçu et scanné au centre de données Azure. <br> Une fois le colis inspecté, le téléchargement de l’appareil commencera.      |
|Copie de données     | La copie des données est en cours. Suivez la progression de la copie de votre commande sur le portail Azure. <br> Attendez que la copie des données se termine. |
|Completed       |La commande a été terminée avec succès.<br> Vérifiez que vos données se trouvent bien dans Azure avant de supprimer les données locales des serveurs.         |
|Completed with errors (Terminée avec des erreurs)| La copie des données a été effectuée mais des erreurs se sont produites pendant l’opération. <br> Consultez les journaux d’activité de copie en utilisant le chemin d’accès spécifié dans le portail Azure. Consultez [des exemples de journaux de copie lorsque le téléchargement s’est terminé avec des erreurs](./data-box-logs.md#upload-completed-with-errors).   |
|Terminé avec des avertissements| La copie des données a été effectuée, mais vos données ont été modifiées. Les données comportaient des erreurs non critiques d’objet BLOB ou de nom de fichier qui ont été corrigées en modifiant les noms des fichiers ou des objets BLOB. <br> Consultez les journaux d’activité de copie en utilisant le chemin d’accès spécifié dans le portail Azure. Prenez note des modifications apportées à vos données. Consultez [des exemples de journaux de copie lorsque le téléchargement s’est terminé avec des avertissements](./data-box-logs.md#upload-completed-with-warnings).   |
|Opération annulée            |La commande a été annulée. <br> Soit vous avez annulé la commande, soit une erreur s’est produite et le service a annulé la commande. Si elle ne peut pas être traitée sous 90 jours, la commande est également annulée et vous en êtes informé.     |
|Nettoyer | Les données sur les disques de l’appareil sont effacées. Le nettoyage de l’appareil est considéré comme terminé lorsque l’historique de la commande est disponible au téléchargement dans le portail Azure.|

### <a name="statuses-for-export-order"></a>États des commandes d’exportation

Voici les états d’une commande d’exportation.

|État de la commande |Description |
|---------|---------|
|Ordered (Validée)     | La commande d’exportation a été validée. <br>Si l’appareil est disponible, Microsoft identifie l’appareil à expédier et le prépare. <br> Si l’appareil n’est pas disponible immédiatement, la commande sera traitée lorsque l’appareil devient disponible. Le traitement de la commande peut prendre quelques jours à deux mois. Si elle ne peut pas être traitée sous 90 jours, la commande est annulée et vous en êtes informé.         |
|Opération annulée            |La commande a été annulée. <br> Soit vous avez annulé la commande (vous ne pouvez annuler qu’avant le traitement de la commande) soit une erreur s’est produite et le service a annulé la commande. Si elle ne peut pas être traitée sous 90 jours, la commande est également annulée et vous en êtes informé.     |
|Processed (Traitée)     | La commande a été traitée. Conformément à votre commande, l’appareil est prêt pour la copie des données dans le centre de données. Des partages d’appareils sont créés.         |
|Copie de données en cours     | La copie des données à partir des comptes de Stockage Azure spécifiés sur l’appareil est en cours. Suivez la progression de la copie de votre commande sur le portail Azure. <br> Attendez que la copie des données se termine. |
|Copie effectuée     | La copie des données à partir des comptes de Stockage Azure spécifiés sur l’appareil est en cours. Un fichier journal détaillé (si l’option a été activée dans la commande) et un journal de copie sont créés dans votre compte de stockage. Le journal détaillé contient les informations sur tous les fichiers (nom, chemin d’accès, somme de contrôle) qui sont copiés sur l’appareil. Le journal de copie contient le résumé du processus de copie, notamment une liste des fichiers qui n’ont pas pu être copiés en raison d’erreurs.<br> Les données du compte de stockage restent telles quelles. |
|Copie terminée avec des erreurs| La copie des données a été effectuée mais des erreurs se sont produites pendant l’opération. <br> Consultez les journaux d’activité de copie dans le compte de Stockage Azure en utilisant le chemin d’accès spécifié dans le Portail Azure. Consultez [des exemples de journaux de copie lorsque le téléchargement s’est terminé avec des erreurs](./data-box-logs.md#upload-completed-with-errors).   |
|Copie terminée avec des avertissements| La copie des données à partir du compte de stockage Azure a été effectuée, mais les données comportaient des erreurs non critiques. <br> Consultez les journaux d’activité de copie en utilisant le chemin d’accès spécifié dans le portail Azure. Notez les erreurs non critiques. Consultez [des exemples de journaux de copie lorsque le téléchargement s’est terminé avec des avertissements](./data-box-logs.md#upload-completed-with-warnings).   |
|La copie a échoué avec des erreurs| La copie de données à partir du compte de stockage Azure a échoué et la commande est annulée. Aucun appareil ne sera expédié.<br> Consultez les journaux d’activité de copie dans le compte de Stockage Azure en utilisant le chemin d’accès spécifié dans le Portail Azure. Consultez [des exemples de journaux de copie lorsque le téléchargement a échoué avec des erreurs](./data-box-logs.md#upload-completed-with-errors).   |
|Dispatched (Distribuée)     |La commande a été expédiée. Utilisez l’ID de suivi affiché sur votre commande dans le portail pour effectuer le suivi de l’expédition.        |
|Delivered (Livrée)     |Le colis a été livré à l’adresse spécifiée dans la commande.        |
|Picked up (Récupérée)     |Le colis de retour a été récupéré et scanné par le transporteur.         |
|Reçu     | Votre appareil a été reçu et scanné au centre de données Azure. <br> L’expédition est examinée.      |
|Completed           |La commande est terminée.     |
|Nettoyer | Les données sur les disques de l’appareil sont effacées. Le nettoyage de l’appareil est considéré comme terminé lorsque l’historique de la commande est disponible au téléchargement dans le portail Azure.|

> [!NOTE]
> Si la tâche de copie pour exporter des données à partir de comptes de Stockage Azure vers Data Box se termine avec des erreurs ou des avertissements, l’appareil est expédié quand même. Ce n’est qu’en cas d’échec de copie que la commande est annulée et que l’appareil n’est pas expédié.


Si vous utilisez l’expédition à gestion automatique, lorsque la copie est terminée et avant que vous ne receviez l’appareil, les états suivants s’affichent (au lieu de ceux mentionnés dans le tableau précédent) :

|État de la commande |Description |
|---------|---------|
|Prêt pour le retrait au centre de données Azure      |L’appareil est prêt à être récupéré dans le centre de données Azure.        |
|Picked up (Récupérée)    |Vous avez récupéré l’appareil.         |
|Prêt pour la réception au centre de données Azure     |L’appareil est prêt à être reçu dans le centre de données Azure.        |
|Reçu     |L’appareil est prêt est reçu dans le centre de données Azure.      |





## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [résoudre les problèmes de votre Data Box et de votre Data Box Heavy](data-box-troubleshoot.md).