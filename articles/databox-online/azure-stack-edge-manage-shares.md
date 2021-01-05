---
title: Gestion des partages de fichiers sur Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment utiliser le Portail Azure pour gérer les partages sur votre Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/25/2019
ms.author: alkohli
ms.openlocfilehash: 4d43b70c8f958ad37b6ac2d0ee043d5f07d11da9
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96444586"
---
# <a name="use-the-azure-portal-to-manage-shares-on-azure-stack-edge-pro"></a>Utiliser le portail Azure pour gérer les partages sur Azure Stack Edge Pro

Cet article explique comment gérer les partages sur votre Azure Stack Edge Pro. Vous pouvez gérer Azure Stack Edge Pro par le biais du Portail Azure ou de l’interface utilisateur web locale. Utilisez le Portail Azure pour ajouter, supprimer, actualiser les partages ou synchroniser la clé de stockage du compte de stockage associé aux partages.

## <a name="about-shares"></a>À propos des partages

Pour transférer des données vers Azure, vous devez créer des partages sur votre Azure Stack Edge Pro. Les partages que vous ajoutez sur l’appareil Azure Stack Edge Pro sont des partages locaux ou des partages qui envoient (push) les données vers le cloud.

 - **Partages locaux** : Utilisez ces partages lorsque vous souhaitez que les données soient traitées localement sur l’appareil.
 - **Partages** : Utilisez ces partages lorsque vous souhaitez que les données d’appareil soient automatiquement envoyées à votre compte de stockage dans le cloud. Toutes les fonctions cloud telles que **Actualiser** et **Synchroniser les clés de stockage** s’appliquent à ces partages.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Ajouter un partage
> * Supprimer un partage
> * Actualiser les partages
> * Synchroniser une clé de stockage

## <a name="add-a-share"></a>Ajouter un partage

Pour créer un partage, procédez comme suit dans le Portail Azure.

1. Sur le Portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Passerelle > Partages**. Sélectionnez **+ Ajouter un partage** dans la barre de commandes.

    ![Sélectionner Ajouter un partage](media/azure-stack-edge-manage-shares/add-share-1.png)

2. Dans **Ajouter un partage**, spécifiez les paramètres du partage. Indiquez un nom unique pour votre partage.
    
    Les noms de partage peuvent uniquement contenir des chiffres, des lettres minuscules et des traits d’union. Le nom de partage doit contenir entre 3 et 63 caractères et commencer par une lettre ou un chiffre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret.

3. Sélectionnez un **type** de partage. Le type peut être **SMB** ou **NFS**, SMB étant la valeur par défaut. SMB est la norme pour les clients Windows, tandis que NFS est utilisé pour les clients Linux. Selon que vous choisissez un partage SMB ou NFS, les options proposées diffèrent légèrement.

4. Fournissez un **compte de stockage** dans lequel se trouve le partage. Si le conteneur n’existe pas déjà, un conteneur est créé dans le compte de stockage avec le nom du partage. Si le conteneur existe déjà, le conteneur existant est utilisé.

5. Dans la liste déroulante, choisissez le **Service de stockage** d’objet blob de blocs, d’objet blob de pages ou de fichier. Le type de service choisi varie selon le format dans lequel vous souhaitez que les données résident dans Azure. Par exemple, dans cette instance, nous voulons que les données résident en tant qu’objets blob de blocs dans Azure. C’est pourquoi nous sélectionnons **Objet blob de blocs**. Si vous choisissez un **Objet blob de pages**, vous devez vous assurer que vos données sont de 512 octets alignés. Si vous choisissez un **Objet blob de pages**, vous devez vous assurer que vos données sont de 512 octets alignés.

   > [!IMPORTANT]
   > Vérifiez que le compte de stockage Azure que vous utilisez n’a pas de stratégies d’immuabilité définies si vous l’utilisez avec un appareil Azure Stack Edge Pro ou Data Box Gateway. Pour plus d’informations, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](../storage/blobs/storage-blob-immutability-policies-manage.md).

6. Cette étape varie selon que vous créez un partage SMB ou NFS.
   - **Si vous créez un partage SMB** : dans le champ **All privilege local user** (Utilisateur local avec tous les privilèges), choisissez **Créer** ou **Utiliser l’existant**. Si vous créez un utilisateur local, indiquez un **nom d’utilisateur**, un **mot de passe**, puis confirmez le mot de passe. Cela affecte les autorisations à l’utilisateur local. Après avoir attribué les autorisations à cet emplacement, vous pouvez utiliser l’Explorateur de fichiers par la suite pour modifier ces autorisations.

      ![Ajouter un partage SMB](media/azure-stack-edge-manage-shares/add-smb-share.png)

        Si vous cochez l’option permettant d’autoriser uniquement les opérations de lecture pour ces données de partage, vous avez la possibilité d’indiquer des utilisateurs en lecture seule.
   - **Si vous créez un partage NFS** : vous devez fournir les **adresses IP des clients autorisés** qui peuvent accéder au partage.

      ![Ajouter un partage NFS](media/azure-stack-edge-manage-shares/add-nfs-share.png)

7. Pour accéder aisément aux partages à partir des modules de computing en périphérie, utilisez le point de montage local. Sélectionnez **Utiliser le partage avec le computing en périphérie** pour que le partage soit monté automatiquement après sa création. Lorsque cette option est sélectionnée, le module Edge peut également utiliser le computing avec le point de montage local.

8. Cliquez sur **Créer** pour créer le partage. Vous êtes averti que le partage est en cours de création. Une fois le partage créé avec les paramètres spécifiés, le panneau **Partages** s’actualise pour refléter le nouveau partage.

## <a name="add-a-local-share"></a>Ajouter un partage local

1. Sur le Portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Passerelle > Partages**. Sélectionnez **+ Ajouter un partage** dans la barre de commandes.

   ![Sélectionner Ajouter un partage 2](media/azure-stack-edge-manage-shares/add-local-share-1.png)

2. Dans **Ajouter un partage**, spécifiez les paramètres du partage. Indiquez un nom unique pour votre partage.
    
    Les noms de partage peuvent uniquement contenir des chiffres, des lettres minuscules et des traits d’union. Le nom de partage doit contenir entre 3 et 63 caractères et commencer par une lettre ou un chiffre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret.

3. Sélectionnez un **type** de partage. Le type peut être **SMB** ou **NFS**, SMB étant la valeur par défaut. SMB est la norme pour les clients Windows, tandis que NFS est utilisé pour les clients Linux. Selon que vous choisissez un partage SMB ou NFS, les options proposées diffèrent légèrement.

4. Pour accéder aisément aux partages à partir des modules de computing en périphérie, utilisez le point de montage local. Sélectionnez **Utiliser le partage avec le computing en périphérie** pour que le module en périphérie puisse utiliser le computing avec le point de montage local.

5. Sélectionnez **Configurer en tant que partages locaux Edge**. Les données des partages locaux restent locales sur l’appareil. Vous pouvez les traiter localement.

6. Dans le champ **All privilege local user** (Utilisateur local avec tous les privilèges), choisissez **Créer** ou **Utiliser l’existant**.

7. Sélectionnez **Create** (Créer). 

   ![Créer un partage local](media/azure-stack-edge-manage-shares/add-local-share-2.png)

    Un message de notification indique que le partage est en cours de création. Une fois le partage créé avec les paramètres spécifiés, le panneau **Partages** s’actualise pour refléter le nouveau partage.

   ![Afficher les mises à jour dans le panneau Partages](media/azure-stack-edge-manage-shares/add-local-share-3.png)
    
    Sélectionnez le partage pour afficher le point de montage local des modules de computing en périphérie associés.

   ![Afficher les détails de partage local](media/azure-stack-edge-manage-shares/add-local-share-4.png)

## <a name="mount-a-share"></a>Monter un partage

Si vous avez créé un partage avant de configurer le calcul sur votre appareil Azure Stack Edge Pro, vous devez monter le partage. Effectuez les étapes suivantes pour monter un partage.

1. Sur le Portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Passerelle > Partages**. Dans la liste des partages, sélectionnez le partage que vous souhaitez monter. La colonne **Utilisé pour le computing** affiche l’état **Désactivé** pour le partage sélectionné.

   ![Sélectionner un partage 3](media/azure-stack-edge-manage-shares/select-share-mount.png)

2. Sélectionnez **Monter**.

   ![Sélectionner l’option de montage](media/azure-stack-edge-manage-shares/select-mount.png)

3. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui**. Le partage est alors monté.

   ![Confirmer le montage](media/azure-stack-edge-manage-shares/confirm-mount.png)

4. Une fois le partage monté, accédez à la liste des partages. Vous pouvez constater que la colonne **Utilisé pour le computing** indique l’état de partage **Activé**.

   ![Partage monté](media/azure-stack-edge-manage-shares/share-mounted.png)

5. Resélectionnez le partage pour voir le point de montage local du partage. Le module computing en périphérie utilise ce point de montage local pour le partage.

   ![Point de montage local pour le partage](media/azure-stack-edge-manage-shares/share-mountpoint.png)

## <a name="unmount-a-share"></a>Démonter un partage

Effectuez les étapes suivantes dans le portail Azure pour démonter un partage.

1. Sur le Portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Passerelle > Partages**.

   ![Sélectionner un partage 4](media/azure-stack-edge-manage-shares/select-share-unmount.png)

2. Dans la liste des partages, sélectionnez le partage que vous souhaitez démonter. Vous devez vous assurer que le partage à démonter n’est pas utilisé par un module. Si le partage est utilisé par un module, vous verrez apparaître des problèmes avec le module correspondant. Sélectionnez **Démonter**.

   ![Sélectionner l’option de démontage](media/azure-stack-edge-manage-shares/select-unmount.png)

3. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui**. Le partage est alors démonté.

   ![Confirmer le démontage](media/azure-stack-edge-manage-shares/confirm-unmount.png)

4. Une fois le partage démonté, accédez à la liste des partages. Vous voyez que la colonne **Utilisé pour le computing** indique l’état de partage **Désactivé**.

   ![Partage démonté](media/azure-stack-edge-manage-shares/share-unmounted.png)

## <a name="delete-a-share"></a>Supprimer un partage

Pour supprimer un partage, procédez comme suit dans le Portail Azure.

1. Dans la liste des partages, sélectionnez et cliquez sur le partage que vous souhaitez supprimer.

   ![Sélectionner un partage 5](media/azure-stack-edge-manage-shares/delete-share-1.png)

2. Cliquez sur **Supprimer**.

   ![Clic sur Supprimer](media/azure-stack-edge-manage-shares/delete-share-2.png)

3. Cliquez sur **Oui** lorsque vous êtes invité à confirmer l’opération.

   ![Confirmation de suppression](media/azure-stack-edge-manage-shares/delete-share-3.png)

La liste des partages est mise à jour afin de tenir compte de la suppression.

## <a name="refresh-shares"></a>Actualiser les partages

La fonctionnalité d’actualisation vous permet d’actualiser le contenu d’un partage. Lorsque vous actualisez un partage, une recherche est lancée pour rechercher tous les objets Azure, y compris les objets blob et les fichiers qui ont été ajoutés dans le cloud depuis la dernière actualisation. Ces fichiers supplémentaires sont ensuite téléchargés pour actualiser le contenu du partage sur l’appareil.

> [!IMPORTANT]
>
> - Vous ne pouvez pas actualiser des partages locaux.
> - Les autorisations et les listes de contrôle d’accès (ACL) ne sont pas conservées lors d’une opération d’actualisation. 

Pour actualiser un partage, procédez comme suit dans le Portail Azure.

1. Accédez à **Partages** dans le Portail Azure. Sélectionnez et cliquez sur le partage que vous souhaitez actualiser.

   ![Sélectionner un partage 6](media/azure-stack-edge-manage-shares/refresh-share-1.png)

2. Cliquez sur **Actualiser**.

   ![Cliquer sur Actualiser](media/azure-stack-edge-manage-shares/refresh-share-2.png)
 
3. Cliquez sur **Oui** lorsque vous êtes invité à confirmer l’opération. Un travail démarre pour actualiser le contenu du partage local.

   ![Confirmer l’actualisation](media/azure-stack-edge-manage-shares/refresh-share-3.png)

4. Pendant l’actualisation, l’option d’actualisation du menu contextuel est grisée. Cliquez sur la notification de travail pour afficher l’état du travail d’actualisation.

5. La durée nécessaire pour l’actualisation varie en fonction du nombre de fichiers dans le conteneur Azure, ainsi que des fichiers sur l’appareil. Une fois l’actualisation terminée, le timestamp du partage est mis à jour. Même si l’actualisation présente des échecs partiels, l’opération est considérée comme réussie et le timestamp est mis à jour. Les journaux d’erreurs d’actualisation sont également mis à jour.

   ![Timestamp mis à jour](media/azure-stack-edge-manage-shares/refresh-share-4.png)
 
En cas d’échec, une alerte est déclenchée. L’alerte décrit en détail la cause du problème et la recommandation à suivre pour le résoudre. L’alerte contient également un lien vers un fichier présentant le résumé des échecs, notamment les fichiers qui n’ont pas pu être mis à jour ou supprimés.

## <a name="sync-storage-keys"></a>Synchroniser des clés de stockage

Si la rotation des clés de votre compte de stockage n’a pas été effectuée, vous devez synchroniser les clés d’accès de stockage. La synchronisation permet à l’appareil d’obtenir les dernières clés pour votre compte de stockage.

Procédez comme suit dans le Portail Azure pour synchroniser votre clé d’accès de stockage.

1. Accédez à **Vue d’ensemble** dans votre ressource. Dans la liste des partages, choisissez un partage associé au compte de stockage que vous devez synchroniser et cliquez dessus.

    ![Sélectionner le partage avec le compte de stockage adéquat](media/azure-stack-edge-manage-shares/sync-storage-key-1.png)

2. Cliquez sur **Sync storage key** (Synchroniser une clé de stockage). Cliquez sur **Oui** lorsque vous êtes invité à confirmer l’opération.

     ![Sélectionner Sync storage key (Synchroniser une clé de stockage)](media/azure-stack-edge-manage-shares/sync-storage-key-2.png)

3. Fermez la boîte de dialogue une fois la synchronisation terminée.

>[!NOTE]
> Cette opération est à faire une seule fois pour un compte de stockage donné. Vous n’avez pas besoin de répéter cette action pour tous les partages associés au même compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [gérer des utilisateurs via le Portail Azure](azure-stack-edge-manage-users.md).