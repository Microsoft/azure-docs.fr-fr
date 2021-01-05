---
title: Gérer les informations d’identification du compte de stockage StorSimple Virtual Array | Microsoft Docs
description: Explique comment utiliser la page Configurer de StorSimple Device Manager pour ajouter, modifier, supprimer ou régénérer les clés de sécurité des informations d’identification du compte de stockage associé à StorSimple Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 234bf8bb-d5fe-40be-9d25-721d7482bc3b
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.openlocfilehash: 70b6231829173059ffee675105bacbe1bd1532f2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023889"
---
# <a name="use-storsimple-device-manager-to-manage-storage-account-credentials-for-storsimple-virtual-array"></a>Utilisation de StorSimple Device Manager pour gérer les informations d’identification du compte de stockage pour StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble
La section **Configuration** du panneau du service StorSimple Device Manager de votre instance StorSimple Virtual Array présente les paramètres de service globaux pouvant être créés dans le service StorSimple Manager. Ces paramètres peuvent être appliqués à tous les appareils connectés au service et incluent les éléments suivants :

* Informations d’identification du compte de stockage
* Enregistrements de contrôle d’accès
  
  ![Tableau de bord du service StorSimple Device Manager](./media/storsimple-virtual-array-manage-storage-accounts/ova-storageaccts-dashboard.png)  

Ce didacticiel vous explique comment ajouter, modifier ou supprimer les informations d’identification du compte de stockage pour votre instance StorSimple Virtual Array. Les informations contenues dans ce didacticiel s’appliquent uniquement à l’instance StorSimple Virtual Array. Pour plus d’informations sur la gestion des comptes de stockage de la gamme 8000, consultez la page [Utiliser le service StorSimple Manager pour gérer votre compte de stockage](./storsimple-8000-manage-storage-accounts.md).

Les informations d’identification des comptes de stockage contiennent les informations d’identification que l’appareil utilise pour accéder au compte de stockage que vous procure votre fournisseur de services cloud. Pour les comptes de stockage Microsoft Azure, il s’agit d’informations d’identification telles que le nom du compte et la clé d’accès primaire.

Sur le panneau **Informations d’identification du compte de stockage**, l’ensemble des informations d’identification de compte de stockage créées pour l’abonnement à la facturation sont affichées sous forme de tableau, et contiennent les données suivantes :

* **Nom** : nom unique attribué au compte lors de sa création.
* **SSL activé** : permet d’indiquer si le protocole TLS est activé et si la communication appareil-cloud s’effectue sur le canal sécurisé.
  
  ![Section de configuration](./media/storsimple-virtual-array-manage-storage-accounts/ova-storageaccountcredentials-blade.png)

Les travaux les plus communes associées aux informations d’identification du compte de stockage pouvant être effectuées sur le panneau **Informations d’identification du compte de stockage** sont les suivantes :

* Ajout des informations d’identification de compte de stockage
* Modification des informations d’identification de compte de stockage
* Suppression des informations d’identification de compte de stockage

## <a name="types-of-storage-account-credentials"></a>Types d’informations d’identification de compte de stockage
Il existe trois types d’informations d’identification de compte de stockage pouvant être utilisées avec votre appareil StorSimple.

* **Informations d’identification de compte de stockage générées automatiquement** : comme son nom l’indique, de type de données de compte de stockage est automatiquement généré lors de la création du service. Pour plus d’informations sur la création de ce type d’informations d’identification de compte de stockage, consultez la section [Création d’un nouveau service](storsimple-virtual-array-manage-service.md#create-a-service).
* **Informations d’identification de compte de stockage dans l’abonnement au service** : ces données sont celles qui sont associées à l’abonnement au service. Pour plus d’informations sur la création de ce type d’informations d’identification, consultez la section [À propos des comptes de stockage Azure](../storage/common/storage-account-create.md).
* **Informations d’identification de compte de stockage en dehors de l’abonnement au service** : il s’agit ici des données qui ne sont pas associées à votre service et existaient probablement avant sa création.

## <a name="add-a-storage-account-credential"></a>Ajout des informations d’identification de compte de stockage
Pour ajouter les informations d’identification du compte de stockage à votre configuration de service StorSimple Device Manager, vous pouvez fournir un nom convivial et des informations d’identification d’accès uniques liées au compte de stockage. Vous avez également la possibilité d’activer le mode TLS (Transport Layer Security), auparavant appelé SSL (Secure Sockets Layer), afin de créer un canal sécurisé pour la communication réseau entre l’appareil et le cloud.

Vous pouvez créer plusieurs comptes pour un fournisseur de services cloud donné. Pendant l’enregistrement des informations d’identification du compte de stockage, le service tente de communiquer avec votre fournisseur de services cloud. Les informations d’identification et les données d’accès fournies sont authentifiées à ce moment. Les informations d’identification du compte de stockage sont créées uniquement en cas de succès de l’authentification. Si l’authentification échoue, un message d’erreur approprié s’affiche.

Appliquez les procédures suivantes pour ajouter les informations d’identification de compte de stockage Azure :

* Pour ajouter une information d’identification de compte de stockage bénéficiant de la même inscription Azure que le service Device Manager
* Pour ajouter des informations d’identification de compte de stockage Azure en dehors de l’abonnement au service Device Manager

#### <a name="to-add-a-storage-account-credential-that-has-the-same-azure-subscription-as-the-device-manager-service"></a>Pour ajouter une information d’identification de compte de stockage bénéficiant de la même inscription Azure que le service Device Manager

1. Accédez au service Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**.
3. Cliquez sur **Add**.
4. Dans le panneau **Ajouter un compte de stockage**, procédez comme suit :
   
    1. Pour **Abonnement**, sélectionnez **Actuel**.
    2. Fournissez le nom de votre compte de stockage Azure.
    3. Sélectionnez **Activer** afin de créer un canal sécurisé pour la communication réseau entre votre appareil StorSimple et le cloud. Sélectionnez **Désactiver** uniquement si vous évoluez au sein d’un cloud privé.
    4. Cliquez sur **Add**. Un message s’affiche pour vous informer une fois le compte de stockage correctement créé.<br></br>
   
        ![Ajouter une information d’identification de compte de stockage existant](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-storageacct.png)

#### <a name="to-add-an-azure-storage-account-credential-that-is-outside-of-the-device-manager-service-subscription"></a>Pour ajouter des informations d’identification de compte de stockage Azure en dehors de l’abonnement au service Device Manager

1. Accédez au service Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**. Ici sont répertoriées les informations d’identification de compte de stockage existantes associées au service StorSimple Device Manager.
3. Cliquez sur **Add**.
4. Dans le panneau **Ajouter un compte de stockage**, procédez comme suit :
   
    1. Pour **Abonnement**, sélectionnez **Autre**.
   
    2. Sélectionnez le nom associé à vos informations d’identification de compte de stockage Azure.
   
    3. Dans la zone textuelle **Clé d’accès au compte de stockage**, fournissez la clé primaire d’accès pour vos informations d’identification de compte de stockage Azure. Pour obtenir cette clé, accédez au service du stockage Azure, sélectionnez vos informations d’identification du compte de stockage, puis cliquez sur **Gérer les clés d’accès**. Vous pouvez maintenant copier la clé d’accès primaire.
   
    4. Pour activer le mode TLS, cliquez sur le bouton **Activer** afin de créer un canal sécurisé pour la communication réseau entre votre service StorSimple Device Manager et le cloud. Cliquez sur le bouton **Désactiver** uniquement si vous évoluez au sein d’un cloud privé.
   
    5. Cliquez sur **Add**. Un message s’affiche pour vous informer de la création des informations d’identification du compte de stockage.

5. Les informations d’identification de compte de stockage nouvellement créées s’affiche sur le panneau du service Device Manager, sous **Informations d’identification du compte de stockage**.
   
    ![Ajouter des informations d’identification en dehors de l’abonnement au service Device Manager](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-outside-storageacct.png)

## <a name="edit-a-storage-account-credential"></a>Modification des informations d’identification de compte de stockage
Vous pouvez modifier des informations d’identification du compte de stockage utilisées par votre appareil. Si vous modifiez des informations d’identification de compte de stockage actuellement utilisées, les champs modifiables seront celui de la clé d’accès et celui du mode TLS de ces données. Vous pouvez saisir la nouvelle clé d’accès de stockage ou modifier la sélection **Activer le mode SSL** , puis enregistrer les modifications.

#### <a name="to-edit-a-storage-account-credential"></a>Pour modifier des informations d’identification de compte de stockage
1. Accédez au service Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**. Ici sont répertoriées les informations d’identification de compte de stockage existantes associées au service StorSimple Device Manager.
3. Dans la liste des informations d’identification de compte de stockage sous forme de tableau, sélectionnez le compte que vous souhaitez modifier, puis double-cliquez dessus.
4. Dans le panneau **Propriétés** des informations d’identification du compte de stockage, procédez comme suit :
   
   1. Si nécessaire, vous pouvez modifier le réglage de l’option **Activer le mode SSL**.
   2. Vous pouvez choisir de régénérer les clés d’accès de vos informations d’identification de compte de stockage. Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../storage/common/storage-account-keys-manage.md). Fournissez la nouvelle clé des informations d’identification de compte de stockage. Pour un compte de stockage Azure, il s’agit de la clé d’accès primaire.
   3. Pour enregistrer les paramètres, cliquez sur **Enregistrer** en haut du panneau **Propriétés**. Les paramètres sont mis à jour sur le panneau **Informations d’identification du compte de stockage**.
      
      ![Modification des informations d’identification de compte de stockage](./media/storsimple-virtual-array-manage-storage-accounts/ova-edit-storageacct.png)

## <a name="delete-a-storage-account-credential"></a>Suppression des informations d’identification de compte de stockage
> [!IMPORTANT]
> Vous ne pouvez pas supprimer des informations d’identification d’un compte de stockage en cours d’utilisation. Si les informations d’identification du compte de stockage sont en cours d’utilisation, vous en êtes informé.
> 
> 

#### <a name="to-delete-a-storage-account-credential"></a>Pour supprimer des informations d’identification de compte de stockage
1. Accédez au service Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**. Ici sont répertoriées les informations d’identification de compte de stockage existantes associées au service StorSimple Device Manager.
3. Dans la liste des informations d’identification de compte de stockage sous forme de tableau, sélectionnez le compte que vous souhaitez supprimer, puis double-cliquez dessus.
4. Dans le panneau **Propriétés** des informations d’identification du compte de stockage, procédez comme suit :
   
   1. Pour supprimer les informations d’identification, cliquez sur **Supprimer**.
   2. Lorsque vous êtes invité à confirmer la suppression, cliquez sur **Oui** . La liste du tableau est mise à jour suivant les modifications.
      
      ![Suppression des informations d’identification de compte de stockage](./media/storsimple-virtual-array-manage-storage-accounts/ova-del-storageacct.png)

## <a name="synchronizing-storage-account-credential-keys"></a>Synchronisation des clés des informations d’identification du compte de stockage
Pour des raisons de sécurité, les centres de données exigent souvent une rotation des clés. Un administrateur Microsoft Azure peut régénérer ou modifier la clé primaire ou secondaire en accédant directement aux informations d’identification du compte de stockage (via le service de stockage Microsoft Azure). Le service StorSimple Device Manager ne voit pas automatiquement cette modification.

Pour informer le service StorSimple Device Manager de la modification, vous devez accéder au service StorSimple Device Manager et aux informations d’identification du compte de stockage, puis synchronisez la clé primaire ou secondaire (selon celle qui a été modifiée). Le service obtient alors la clé la plus récente, chiffre les clés, puis envoie la clé chiffrée à l’appareil.

#### <a name="to-synchronize-keys-for-storage-account-credentials-in-the-same-subscription-as-the-service-azure-only"></a>Pour synchroniser les clés des informations d’identification des comptes de stockage associés au même abonnement que le service (Azure uniquement)
1. Sur le panneau d’accueil du service, sélectionnez votre service, double-cliquez sur le nom du service, puis dans la section **Configuration**, cliquez sur **Informations d’identification du compte de stockage**.
2. Sur le panneau **Informations d’identification du compte de stockage**, dans la liste des informations d’identification du compte de stockage, sélectionnez les informations dont vous voulez synchroniser les clés.
3. Dans le panneau **Propriétés** des informations d’identification sélectionnées de compte de stockage, procédez comme suit :
   
    1. Cliquez sur **Plus**, puis sur **Synchroniser la clé d’accès**.
   
    2. Lorsque vous êtes invité à confirmer l’opération, cliquez sur **Synchroniser la clé** afin de terminer la synchronisation.
    
4. Dans le service StorSimple Device Manager, vous devez mettre à jour la clé que vous avez précédemment modifiée dans le service de stockage Microsoft Azure. Dans le panneau **Synchroniser la clé de compte de stockage**, si la clé primaire d’accès a été modifiée (régénérée), cliquez sur Primaire, puis sur **Synchroniser la clé**. Si la clé secondaire a été modifiée, cliquez sur **Secondaire**, puis sur **Synchroniser la clé**.
   
    ![Synchroniser la clé d’accès](./media/storsimple-virtual-array-manage-storage-accounts/ova-sync-access-key.png)

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).