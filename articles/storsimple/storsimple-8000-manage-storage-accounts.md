---
title: Gérer les informations d’identification d’un compte de stockage, appareil StorSimple série 8000
description: Explique comment utiliser la page Configurer du service StorSimple Device Manager pour ajouter, modifier ou supprimer les clés de sécurité d’un compte de stockage, ou pour effectuer une rotation de ces clés.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: 0df0c12d5942179e2492bfdc932fa25d126a2426
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016866"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-your-storage-account-credentials"></a>Utiliser le service StorSimple Device Manager pour gérer vos informations d’identification de compte de stockage

## <a name="overview"></a>Vue d’ensemble

La section **Configuration** du panneau du service StorSimple Device Manager présente tous les paramètres de service généraux pouvant être créés dans le service StorSimple Device Manager. Ces paramètres peuvent être appliqués à tous les appareils connectés au service et incluent les éléments suivants :

* Informations d’identification du compte de stockage
* Modèles de bande passante 
* Enregistrements de contrôle d’accès 

Ce didacticiel explique comment ajouter, modifier ou supprimer des informations d’identification de compte de stockage, ou effectuer une rotation des clés de sécurité d’un compte de stockage.

 ![Liste d’informations d’identification de compte de stockage 1](./media/storsimple-8000-manage-storage-accounts/createnewstorageacct6.png)  

Les comptes de stockage contiennent les informations d’identification que l’appareil StorSimple utilise pour accéder au compte de stockage que vous procure votre fournisseur de services cloud. Pour les comptes de stockage Microsoft Azure, il s’agit d’informations d’identification telles que le nom du compte et la clé d’accès primaire. 

Sur le panneau **Informations d’identification du compte de stockage**, l’ensemble des comptes de stockage créés pour l’abonnement à la facturation sont affichés sous forme de tableau, et contiennent les données suivantes :

* **Nom** : nom unique attribué au compte lors de sa création.
* **SSL activé** : permet d’indiquer si le protocole TLS est activé et si la communication appareil-cloud s’effectue sur le canal sécurisé.
* **Utilisé par** : nombre de volumes utilisant le compte de stockage.

Les tâches les plus courantes que vous pouvez effectuer sur les comptes de stockage sont les suivantes :

* Ajout d’un compte de stockage 
* Modification d’un compte de stockage 
* Suppression d'un compte de stockage 
* Rotation des clés de comptes de stockage 

## <a name="types-of-storage-accounts"></a>Types de compte de stockage

Vous pouvez utiliser trois types de compte de stockage avec votre appareil StorSimple.

* **Comptes de stockage générés automatiquement** : comme son nom l’indique, ce type de compte de stockage est généré automatiquement à la création du service. Pour plus d'informations sur la création des comptes de stockage, consultez [Étape 1 : Créer un nouveau service](storsimple-8000-deployment-walkthrough-u2.md#step-1-create-a-new-service) dans [Déployer votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md). 
* **Comptes de stockage de l’abonnement au service** : il s’agit des comptes de stockage Azure associés à l’abonnement au service. Pour en savoir plus sur la création de ces comptes de stockage, consultez la rubrique [À propos des comptes de stockage Azure](../storage/common/storage-account-create.md). 
* **Comptes de stockage hors abonnement au service** : il s’agit des comptes de stockage Azure qui ne sont pas associés à votre service et existaient probablement avant sa création.

## <a name="add-a-storage-account"></a>Ajout d’un compte de stockage

Vous pouvez ajouter un compte de stockage en fournissant un nom convivial unique et les informations d’accès liées au compte de stockage (du fournisseur de services cloud spécifié). Vous avez également la possibilité d’activer le mode TLS (Transport Layer Security), auparavant appelé SSL (Secure Sockets Layer), afin de créer un canal sécurisé pour la communication réseau entre l’appareil et le cloud.

Vous pouvez créer plusieurs comptes pour un fournisseur de services cloud donné. Veuillez noter toutefois qu’une fois qu’un compte de stockage a été créé, vous ne pouvez plus modifier le fournisseur de services cloud.

Pendant l’enregistrement du compte de stockage, le service tente de communiquer avec votre fournisseur de services cloud. Les informations d’identification et les données d’accès que vous avez fournies sont alors authentifiées. Un compte de stockage est créé uniquement si l’authentification réussit. Si l’authentification échoue, un message d’erreur s’affiche.

Appliquez les procédures suivantes pour ajouter les informations d’identification de compte de stockage Azure :

* Pour ajouter une information d’identification de compte de stockage bénéficiant de la même inscription Azure que le service Device Manager
* Pour ajouter des informations d’identification de compte de stockage Azure en dehors de l’abonnement au service Device Manager

[!INCLUDE [add-a-storage-account-update2](../../includes/storsimple-8000-configure-new-storage-account-u2.md)]

#### <a name="to-add-an-azure-storage-account-credential-outside-of-the-storsimple-device-manager-service-subscription"></a>Pour ajouter des informations d’identification de compte de stockage Azure en dehors de l’abonnement au service StorSimple Device Manager

1. Accédez au service StorSimple Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**. Ici sont répertoriées les informations d’identification de compte de stockage existantes associées au service StorSimple Device Manager.
3. Cliquez sur **Add**.
4. Dans le panneau **Ajouter les informations d’identification d’un compte de stockage**, procédez comme suit :
   
    1. Pour **Abonnement**, sélectionnez **Autre**.
   
    2. Sélectionnez le nom associé à vos informations d’identification de compte de stockage Azure.
   
    3. Dans la zone textuelle **Clé d’accès au compte de stockage**, fournissez la clé primaire d’accès pour vos informations d’identification de compte de stockage Azure. Pour obtenir cette clé, accédez au service du stockage Azure, sélectionnez vos informations d’identification du compte de stockage, puis cliquez sur **Gérer les clés d’accès**. Vous pouvez maintenant copier la clé d’accès primaire.
   
    4. Pour activer le mode TLS, cliquez sur le bouton **Activer** afin de créer un canal sécurisé pour la communication réseau entre votre service StorSimple Device Manager et le cloud. Cliquez sur le bouton **Désactiver** uniquement si vous évoluez au sein d’un cloud privé.
   
    5. Cliquez sur **Add**. Un message s’affiche pour vous informer de la création des informations d’identification du compte de stockage.

5. Les informations d’identification de compte de stockage nouvellement créées s’affiche sur le panneau du service Device Manager, sous **Informations d’identification du compte de stockage**.
   


## <a name="edit-a-storage-account"></a>Modification d’un compte de stockage

Vous pouvez modifier un compte de stockage utilisé par un conteneur de volumes. Si vous modifiez un compte de stockage actuellement utilisé, le seul champ modifiable est celui de la clé d’accès du compte de stockage. Vous pouvez saisir la nouvelle clé d’accès de stockage et enregistrer les paramètres mis à jour.

#### <a name="to-edit-a-storage-account"></a>Pour modifier un compte de stockage

1. Accédez à votre service StorSimple Device Manager. Dans la section **Configuration**, cliquez sur **Informations d’identification du compte de stockage**.

    ![Informations d’identification du compte de stockage 2](./media/storsimple-8000-manage-storage-accounts/editstorageacct1.png)

2. Dans la liste des informations d’identification du compte de stockage du panneau **Informations d’identification du compte de stockage**, sélectionnez et cliquez sur les informations que vous souhaitez modifier. 

3. Vous pouvez modifier la sélection **Activer SSL**. Vous pouvez également cliquer sur **Plus**, puis sélectionner **Synchroniser la clé d’accès** pour effectuer la rotation de vos clés d’accès de compte de stockage. Consultez [Rotation des clés de comptes de stockage](#key-rotation-of-storage-accounts) pour plus d’informations sur la rotation des clés. Une fois que vous avez modifié les paramètres, cliquez sur **Enregistrer**. 

    ![Enregistrer les modifications apportées aux informations d’identification de compte de stockage](./media/storsimple-8000-manage-storage-accounts/editstorageacct3.png)

4. Cliquez sur **Oui** lorsque vous êtes invité à confirmer l’opération. 

    ![Confirmer les modifications](./media/storsimple-8000-manage-storage-accounts/editstorageacct4.png)

Les paramètres sont mis à jour et enregistrés pour votre compte de stockage. 

## <a name="delete-a-storage-account"></a>Suppression d'un compte de stockage

> [!IMPORTANT]
> Vous pouvez supprimer un compte de stockage uniquement s’il n’est pas utilisé par un conteneur de volumes. Si un compte de stockage est utilisé par un conteneur de volumes, supprimez d’abord ce dernier, puis le compte de stockage associé.

#### <a name="to-delete-a-storage-account"></a>Pour supprimer un compte de stockage

1. Accédez à votre service StorSimple Device Manager. Dans la section **Configuration**, cliquez sur **Informations d’identification du compte de stockage**.

2. Dans le tableau de comptes de stockage, placez le pointeur sur le compte à supprimer. Cliquez avec le bouton droit pour ouvrir le menu contextuel, puis cliquez sur **Supprimer**.

    ![Supprimer les informations d’identification de compte de stockage](./media/storsimple-8000-manage-storage-accounts/deletestorageacct1.png)

3. Lorsque vous êtes invité à confirmer la suppression, cliquez sur **Oui** . La liste du tableau est mise à jour pour refléter les modifications.

    ![Confirmation de suppression](./media/storsimple-8000-manage-storage-accounts/deletestorageacct2.png)

## <a name="key-rotation-of-storage-accounts"></a>Rotation des clés de comptes de stockage

Pour des raisons de sécurité, les centres de données exigent souvent une rotation des clés. Chaque abonnement Microsoft Azure peut être associé à un ou plusieurs comptes de stockage. L’accès à ces comptes est contrôlé par l’abonnement et les clés d’accès de chaque compte de stockage. 

Lorsque vous créez un compte de stockage, Microsoft Azure génère deux clés d’accès de stockage de 512 bits, qui servent à l’authentification lors de l’accès au compte de stockage. Grâce aux deux clés d’accès de stockage, vous pouvez régénérer les clés sans interrompre votre service de stockage ni l’accès à ce service. La clé actuellement utilisée est la clé *primaire* et la clé de sauvegarde est appelée clé *secondaire*. Vous devez fournir une de ces deux clés lorsque votre appareil Microsoft Azure StorSimple accède à votre fournisseur de services de stockage cloud.

## <a name="what-is-key-rotation"></a>La rotation des clés, de quoi s’agit-il ?

En règle générale, les applications utilisent une seule clé pour accéder à vos données. Après un certain temps, vous pouvez faire en sorte que vos applications utilisent la deuxième clé. Une fois que vos applications utilisent la clé secondaire, vous pouvez mettre la première clé hors service et générer une nouvelle clé. En utilisant les deux clés de cette manière, vous permettez à vos applications d’accéder aux données sans provoquer de temps d’arrêt.

Les clés de compte de stockage sont toujours stockées dans le service sous forme chiffrée. Toutefois, elles peuvent être réinitialisées par le biais du service StorSimple Device Manager. Ce service peut obtenir la clé primaire et la clé secondaire de tous les comptes de stockage d’un abonnement, y compris les comptes créés dans le service de stockage, ainsi que les comptes de stockage par défaut générés lors de la création du service StorSimple Device Manager. Le service StorSimple Device Manager obtient toujours ces clés à partir du portail Azure Classic et les stocke sous forme chiffrée.

## <a name="rotation-workflow"></a>Workflow de rotation

Un administrateur Microsoft Azure peut régénérer ou modifier la clé primaire ou secondaire en accédant directement au compte de stockage (via le service Microsoft Azure Storage). Le service StorSimple Device Manager ne voit pas automatiquement cette modification.

Pour informer le service StorSimple Device Manager de la modification, vous devez accéder au service StorSimple Device Manager et au compte de stockage, puis synchroniser la clé primaire ou secondaire (selon celle qui a été modifiée). Le service obtient alors la clé la plus récente, chiffre les clés, puis envoie la clé chiffrée à l’appareil.

#### <a name="to-synchronize-keys-for-storage-accounts-in-the-same-subscription-as-the-service"></a>Pour synchroniser les clés des comptes de stockage associés au même abonnement que le service 
1. Accédez à votre service StorSimple Device Manager. Dans la section **Configuration**, cliquez sur **Informations d’identification du compte de stockage**.
2. Dans la liste tabulaire des comptes de stockage, cliquez sur celui que vous souhaitez modifier. 

    ![synchroniser les clés](./media/storsimple-8000-manage-storage-accounts/syncaccesskey1.png)

3. Cliquez sur **Plus**, puis sélectionnez **Synchroniser la clé d’accès** pour effectuer la rotation.   

    ![synchroniser les clés 2](./media/storsimple-8000-manage-storage-accounts/syncaccesskey2.png)

4. Dans le service StorSimple Device Manager, vous devez mettre à jour la clé que vous avez précédemment modifiée dans le service de stockage Microsoft Azure. Si vous avez modifié (régénéré) la clé primaire, sélectionnez la clé **primaire**. Si vous avez modifié la clé secondaire, sélectionnez la clé **secondaire**. Cliquez sur **Synchroniser la clé**.
      
      ![synchroniser les clés 3](./media/storsimple-8000-manage-storage-accounts/syncaccesskey3.png)

Un message vous informe que la clé est synchronisée.

#### <a name="to-synchronize-keys-for-storage-accounts-outside-of-the-service-subscription"></a>Pour synchroniser les clés des comptes de stockage hors abonnement au service
1. Dans la page **Services**, cliquez sur l’onglet **Configurer**.
2. Cliquez sur **Ajouter/modifier des comptes de stockage**.
3. Dans la boîte de dialogue qui s’affiche, procédez comme suit :
   
   1. Sélectionnez le compte de stockage associé à la clé que vous souhaitez mettre à jour.
   2. Vous devrez mettre à jour la clé d’accès de stockage dans le service StorSimple Device Manager. Dans ce cas, vous pouvez voir la clé d’accès de stockage. Entrez la nouvelle clé dans le champ **Clé d’accès au compte de stockage**. 
   3. Enregistrez vos modifications. La clé d’accès de votre compte de stockage doit maintenant être à jour.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [sécurité StorSimple](storsimple-8000-security.md).
* En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).