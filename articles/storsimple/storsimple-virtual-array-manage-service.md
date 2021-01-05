---
title: Déployer le service StorSimple Device Manager | Microsoft Docs
description: Cet article décrit comment créer et supprimer le service StorSimple Device Manager dans le Portail Azure, ainsi que la procédure de gestion de la clé d’inscription du service.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 28499494-8c4d-4a7f-9d44-94b2b8a93c93
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 92041be4d3d5ef4d980c17fa0a5870b740c1784a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995003"
---
# <a name="deploy-the-storsimple-device-manager-service-for-storsimple-virtual-array"></a>Déployer le service StorSimple Device Manager pour StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs appareils StorSimple. Après avoir créé le service, vous pouvez l’utiliser pour gérer les appareils à partir du portail Microsoft Azure s’exécutant dans un navigateur. Vous pouvez ainsi surveiller tous les appareils qui sont connectés au service StorSimple Device Manager à partir d’un emplacement central unique, ce qui réduit la charge administrative.

Les tâches courantes associées à un service StorSimple Device Manager sont les suivantes :

* Créer un service
* Supprimer un service
* Obtenir la clé d’inscription du service
* Régénérer la clé d’inscription du service

Ce didacticiel explique comment exécuter chacune des tâches ci-dessus. Les informations contenues dans cet article s’appliquent uniquement aux tableaux virtuels StorSimple. Pour plus d’informations sur la gamme StorSimple 8000, consultez la page [déployer un service StorSimple Manager](./storsimple-8000-manage-service.md).

## <a name="create-a-service"></a>Créer un service

Pour créer un service, vous avez besoin des éléments suivants :

* Un abonnement avec un contrat Entreprise
* Un compte de stockage Microsoft Azure actif
* Les informations de facturation utilisées pour la gestion des accès

Vous pouvez également choisir de générer un compte de stockage lorsque vous créez le service.

Un seul service peut gérer plusieurs appareils. Cependant, un appareil ne peut pas couvrir plusieurs services. Une grande entreprise peut avoir plusieurs instances de service pour utiliser différents abonnements, organisations ou même emplacements de déploiement.

> [!NOTE]
> Vous devez créer des instances distinctes du service StorSimple Device Manager pour gérer les appareils de la gamme StorSimple 8000 et les baies StorSimple Virtual Array.


Procédez comme suit pour créer un service.

[!INCLUDE [storsimple-virtual-array-create-new-service](../../includes/storsimple-virtual-array-create-new-service.md)]

## <a name="delete-a-service"></a>Supprimer un service

Avant de supprimer un service, assurez-vous qu’aucun appareil connecté ne l’utilise. Si le service est en cours d’utilisation, désactivez les appareils connectés. L’opération de désactivation rompt la connexion entre l’appareil et le service, mais conserve les données de l’appareil dans le cloud.

> [!IMPORTANT]
> Après qu’un service a été supprimé, l’opération ne peut pas être annulée. Un appareil qui utilisait le service doit être réinitialisé aux paramètres d’usine avant de pouvoir être utilisé avec un autre service. Dans ce scénario, les données locales de l’appareil, ainsi que la configuration, seront perdues.
 

Pour supprimer un service, procédez comme suit.

#### <a name="to-delete-a-service"></a>Pour supprimer un service

1. Sélectionnez **Toutes les ressources**. Recherchez votre service StorSimple Device Manager. Sélectionnez le service à supprimer.
   
    ![Sélectionner le service à supprimer](./media/storsimple-virtual-array-manage-service/deleteservice2.png)
2. Accédez à votre tableau de bord des services pour vous assurer qu’aucun appareil n’est connecté au service. Si aucun appareil n’est inscrit auprès de ce service, vous en êtes également averti par un message de bannière. Cliquez sur **Supprimer**.
   
    ![Suppression du service](./media/storsimple-virtual-array-manage-service/deleteservice3.png)

3. Lorsque vous êtes invité à confirmer l’opération, cliquez sur **Oui** dans la notification de confirmation. 
   
    ![Confirmer la suppression du service](./media/storsimple-virtual-array-manage-service/deleteservice4.png)
4. La suppression du service peut nécessiter quelques minutes. Une fois le service supprimé, vous en êtes informé par un message.
   
    ![Suppression du service réussie](./media/storsimple-virtual-array-manage-service/deleteservice6.png)

La liste des services est actualisée.

 ![Liste des services mise à jour](./media/storsimple-virtual-array-manage-service/deleteservice7.png)

## <a name="get-the-service-registration-key"></a>Obtenir la clé d’inscription du service
Une fois que vous avez créé un service, vous devez inscrire votre appareil StorSimple auprès du service. Pour inscrire votre premier appareil StorSimple, vous avez besoin de la clé d’inscription du service. Pour inscrire des appareils supplémentaires avec un service StorSimple existant, vous avez besoin de la clé d’inscription et de la clé de chiffrement des données du service (générée sur le premier appareil lors de l’inscription). Pour plus d’informations sur la clé de chiffrement des données du service, consultez la rubrique [Sécurité StorSimple](./storsimple-8000-security.md). Vous pouvez obtenir la clé d’inscription en accédant au panneau **Clés** relatif à votre service.

Procédez comme suit pour obtenir la clé d’inscription du service.

#### <a name="to-get-the-service-registration-key"></a>Pour obtenir la clé d’inscription du service
1. Dans le panneau **StorSimple Device Manager**, accédez à **Gestion &gt;** **Clés**.
   
   ![Panneau Clés](./media/storsimple-virtual-array-manage-service/getregkey2.png)
2. Une clé d’inscription du service apparaît dans le panneau **Clés**. Copiez la clé d’inscription à l’aide de l’icône de copie. 

Conservez la clé d’inscription du service dans un emplacement sûr. Vous aurez besoin de cette clé, ainsi que de la clé de chiffrement des données du service, pour enregistrer des appareils supplémentaires auprès du service. Après avoir obtenu la clé d’inscription du service, vous devez configurer votre appareil via l’interface Windows PowerShell pour StorSimple.

## <a name="regenerate-the-service-registration-key"></a>Régénérer la clé d’inscription du service
Vous devez régénérer une clé d’inscription du service si vous êtes amené à effectuer une rotation des clés ou si la liste des administrateurs du service a changé. Lorsque vous régénérez la clé, la nouvelle clé est utilisée uniquement pour l’enregistrement des appareils suivants. Les appareils déjà enregistrés ne sont pas affectés par ce processus.

Procédez comme suit pour régénérer une clé d’inscription du service.

#### <a name="to-regenerate-the-service-registration-key"></a>Pour régénérer la clé d’inscription du service
1. Dans le panneau **StorSimple Device Manager**, accédez à **Gestion &gt;** **Clés**.
   
   ![Panneau Clés de nouveau](./media/storsimple-virtual-array-manage-service/getregkey2.png)
2. Dans le panneau **Clés**, cliquez sur **Régénérer**.
   
   ![Cliquer sur Régénérer](./media/storsimple-virtual-array-manage-service/getregkey5.png)
3. Dans le panneau **Régénérer la clé d’inscription du service**, vérifiez l’action requise lors de la régénération des clés. Tous les appareils qui seront inscrits auprès de ce service par la suite utiliseront la nouvelle clé d’inscription. Cliquez sur **Régénérer** pour confirmer l’opération. Une fois l’inscription terminée, vous en êtes informé par un message.
   
   ![Confirmer la régénération de la clé](./media/storsimple-virtual-array-manage-service/getregkey3.png)
4. Une nouvelle clé d’inscription du service s’affiche.
   
    ![Confirmation supplémentaire de la regénération de la clé](./media/storsimple-virtual-array-manage-service/getregkey4.png)
   
   Copiez cette clé et sauvegardez-la pour enregistrer tout nouvel appareil auprès de ce service.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [prendre en main](storsimple-virtual-array-deploy1-portal-prep.md) une solution StorSimple Virtual Array.
* Découvrez comment [gérer votre appareil StorSimple](storsimple-ova-web-ui-admin.md).