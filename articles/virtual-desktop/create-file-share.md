---
title: Créer un partage de fichiers Azure Files avec un contrôleur de domaine - Azure
description: Configurez un conteneur de profil FSLogix sur un partage de fichiers Azure dans un pool hôte Windows Virtual Desktop existant avec votre domaine Active Directory.
author: Heidilohr
ms.topic: how-to
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 8f8086aced26fc46fb1430df074082e8c3365baa
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746809"
---
# <a name="create-a-profile-container-with-azure-files-and-ad-ds"></a>Créer un conteneur de profil avec Azure Files et AD DS

Dans cet article, vous allez apprendre à créer un partage de fichiers Azure authentifié par un contrôleur de domaine sur un pool hôte Windows Virtual Desktop existant. Vous pouvez utiliser ce partage de fichiers pour stocker les profils de stockage.

Ce processus utilise Active Directory Domain Services (AD DS), qui est un service d’annuaire local. Si vous recherchez des informations sur la création d’un conteneur de profil FSLogix avec Azure AD DS, consultez [Créer un conteneur de profil FSLogix avec Azure Files](create-profile-container-adds.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que votre contrôleur de domaine est synchronisé avec Azure et qu’il peut être résolu à partir du réseau virtuel Azure (VNET) auquel vos hôtes de session sont connectés.

## <a name="set-up-a-storage-account"></a>Installer un compte de stockage

Tout d’abord, vous devez configurer un compte de stockage Azure Files.

Pour installer un compte de stockage :

1. Connectez-vous au portail Azure.

2. Recherchez **compte de stockage** dans la barre de recherche.

3. Sélectionnez **+Ajouter**.

4. Entrez les informations suivantes dans la page **Créer un compte de stockage** :

    - Créez un groupe de ressources.
    - Entrez un nom unique pour votre compte de stockage.
    - Pour **Emplacement** , nous vous recommandons de choisir le même emplacement que le pool d’hôtes Windows Virtual Desktop.
    - Pour **Performances** , sélectionnez **Standard**. (En fonction de vos besoins en E/S par seconde. Pour davantage d’informations, consultez [Options de stockage pour conteneurs de profil FSLogix dans Windows Virtual Desktop](store-fslogix-profile.md).)
    - Pour **Type de compte** , sélectionnez **StorageV2** ou **FileStorage** (disponible uniquement si le niveau de performance est Premium).
    - Pour **Réplication** , sélectionnez **Stockage localement redondant (LRS)** .

5. Lorsque vous avez terminé, sélectionnez **Vérifier + créer** , puis sélectionnez **Créer**.

Si vous avez besoin d’instructions de configuration plus détaillées, consultez [Disponibilité régionale](../storage/files/storage-files-identity-auth-active-directory-enable.md#regional-availability).

## <a name="create-an-azure-file-share"></a>Crée un partage de fichiers Azure

Ensuite, vous devez créer un partage de fichiers Azure.

Pour créer un partage de fichiers :

1. Sélectionnez **Accéder à la ressource**.

2. Sur la page Vue d’ensemble, sélectionnez **Partages de fichiers**.

3. Sélectionnez **+Partages de fichiers** , créez un nouveau partage de fichiers nommé **Profils** , puis entrez un quota approprié ou laissez le champ vide si vous préférez ne pas mettre de quota.

4. Sélectionnez **Create** (Créer).

## <a name="enable-active-directory-authentication"></a>Activer l’authentification Active Directory

Ensuite, vous devez activer l’authentification Active Directory (AD). Pour activer cette stratégie, vous devez suivre les instructions de cette section sur un ordinateur qui est déjà joint à un domaine. Pour activer l’authentification, suivez ces instructions sur la machine virtuelle exécutant le contrôleur de domaine :

1. Protocole RDP (Remote Desktop Protocol) à la machine virtuelle jointe au domaine.

2. Suivez les instructions dans [Activer l’authentification AD DS pour vos partages de fichiers Azure](../storage/files/storage-files-identity-ad-ds-enable.md) pour installer le module AzFilesHybrid et activer l’authentification.

3.  Ouvrez le Portail Azure, ouvrez votre compte de stockage, sélectionnez **Configuration** , puis confirmez qu’ **Active Directory (AD)** est défini sur **Activé**.

     > [!div class="mx-imgBorder"]
     > ![Une capture d’écran de la page Configuration avec Azure Active Directory (AD) activé.](media/active-directory-enabled.png)

## <a name="assign-azure-rbac-permissions-to-windows-virtual-desktop-users"></a>Attribuer des autorisations Azure RBAC aux utilisateurs Windows Virtual Desktop

Tous les utilisateurs qui doivent disposer de profils FSLogix stockés sur le compte de stockage doivent se voir attribuer le rôle Contributeur de partage SMB des données des fichiers de stockage.

Les utilisateurs qui se connectent aux hôtes de session Windows Virtual Desktop ont besoin d’autorisations d’accès pour accéder à votre partage de fichiers. L’octroi de l’accès à un partage de fichiers Azure implique la configuration des autorisations au niveau du partage et au niveau NTFS, comme pour un partage Windows traditionnel.

Pour configurer les autorisations au niveau du partage, attribuez à chaque utilisateur un rôle disposant des autorisations d’accès appropriées. Les autorisations peuvent être attribuées à des utilisateurs individuels ou à un groupe Azure AD. Pour en savoir plus, consultez [Assigner des autorisations d’accès à une identité](../storage/files/storage-files-identity-ad-ds-assign-permissions.md).

>[!NOTE]
>Les comptes ou groupes auxquels vous affectez des autorisations doivent avoir été créés dans le domaine et synchronisés avec Azure AD. Les comptes créés dans Azure AD ne fonctionnent pas.

Pour attribuer des autorisations de contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

1. Ouvrez le portail Azure.

2. Ouvrez le compte de stockage que vous avez créé dans [Installer un compte de stockage](#set-up-a-storage-account).

3. Sélectionnez **Partages de fichiers** , puis sélectionnez le nom du partage de fichiers que vous prévoyez d’utiliser.

4. Sélectionnez **Contrôle d’accès (IAM)** .

5. Sélectionnez **Ajouter une attribution de rôle**.

6. Dans l’onglet **Ajouter une attribution de rôle** , sélectionnez **Contributeur élevé de partage SMB de données de fichier de stockage** pour le compte administrateur.

     Pour attribuer des autorisations aux utilisateurs pour leurs profils FSLogix, suivez les mêmes instructions. Toutefois, lorsque vous accédez à l’étape 5, sélectionnez **Contributeur de partage SMB de données de fichier de stockage** à la place.

7. Sélectionnez **Enregistrer**.

## <a name="assign-users-permissions-on-the-azure-file-share"></a>Attribuer des autorisations aux utilisateurs sur le partage de fichiers Azure

Une fois que vous avez attribué des autorisations Azure RBAC à vos utilisateurs, vous devez configurer les autorisations NTFS.

Vous devez connaître deux choses concernant le Portail Azure avant commencer :

- Le chemin d’accès UNC.
- La clé du compte de stockage.

### <a name="get-the-unc-path"></a>Obtenir le chemin d’accès UNC

Voici comment procéder :

1. Ouvrez le portail Azure.

2. Ouvrez le compte de stockage que vous avez créé dans [Installer un compte de stockage](#set-up-a-storage-account).

3. Sélectionnez **Paramètres** , puis **Propriétés**.

4. Copiez le **point de terminaison de service de fichier principal** URI dans l’éditeur de texte de votre choix.

5. Après avoir copié l’URI, effectuez les opérations suivantes pour le modifier en UNC :

    - Supprimez `https://` et remplacez par `\\`
    - Remplacez la barre oblique `/` par une barre oblique inverse `\`.
    - Ajoutez le nom du partage de fichiers que vous avez créé dans [Crée un partage de fichiers Azure](#create-an-azure-file-share) à la fin de l’UNC.

        Par exemple : `\\customdomain.file.core.windows.net\<fileshare-name>`

### <a name="get-the-storage-account-key"></a>Obtenir la clé du compte de stockage

Pour obtenir la clé du compte de stockage :

1. Ouvrez le portail Azure.

2. Ouvrez le compte de stockage que vous avez créé dans [Installer un compte de stockage](#set-up-a-storage-account).

3. Sous l’onglet du **Compte de stockage** , sélectionnez **Clés d’accès**.

4. Copiez **key1** ou **key2** dans un fichier sur votre ordinateur local.

### <a name="configure-ntfs-permissions"></a>Configurer les autorisations NTFS

Pour configurer vos autorisations NTFS :

1. Ouvrez une invite de commandes sur une machine virtuelle jointe au domaine.

2. Exécutez la commande suivante pour monter le partage de fichiers Azure et lui attribuer une lettre de lecteur :

     ```cmd
     net use <desired-drive-letter>: <UNC-path> <SA-key> /user:Azure\<SA-name>
     ```

3. Exécutez la commande suivante pour vérifier les autorisations d’accès au partage de fichiers Azure :

    ```cmd
    icacls <mounted-drive-letter>:
    ```

    Remplacez `<mounted-drive-letter>` par la lettre du lecteur auquel vous êtes mappé.

    Les *Utilisateurs Autorité NT\Utilisateurs authentifiés* et *BUILTIN\Users* disposent de certaines autorisations par défaut. Ces autorisations par défaut permettent à ces utilisateurs de lire les conteneurs de profils d’autres utilisateurs. Toutefois, les autorisations décrites dans [Configurer les autorisations de stockage à utiliser avec les conteneurs de profils et les conteneurs Office](/fslogix/fslogix-storage-config-ht) ne permettent pas aux utilisateurs de lire les conteneurs de profils d’autres utilisateurs.

4. Exécutez les commandes suivantes pour permettre à vos utilisateurs Windows Virtual Desktop de créer leur propre conteneur de profils tout en bloquant l’accès à leurs conteneurs de profils par d’autres utilisateurs.

     ```cmd
     icacls <mounted-drive-letter>: /grant <user-email>:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

     - Remplacez <mounted-drive-letter> par la lettre du lecteur que vous avez utilisé pour mapper le lecteur.
     - Remplacez <user-email> par l’UPN de l’utilisateur ou du groupe de Active Directory qui contient les utilisateurs qui auront besoin d’accéder au partage.

     Par exemple :

     ```cmd
     icacls <mounted-drive-letter>: /grant john.doe@contoso.com:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

## <a name="configure-fslogix-on-session-host-vms"></a>Configurer FSLogix sur des machines virtuelles hôtes de session

Cette section vous indique comment configurer une machine virtuelle avec FSLogix. Vous devez suivre ces instructions chaque fois que vous configurez un hôte de session. Avant de commencer la configuration, suivez les instructions de [Télécharger et installer FSLogix](/fslogix/install-ht). Plusieurs options sont disponibles pour s’assurer que les clés de Registre sont définies sur tous les hôtes de session. Vous pouvez définir ces options dans une image ou configurer une stratégie de groupe.

Pour configurer FSLogix sur vos machines virtuelles hôtes de session :

1. RDP vers la machine virtuelle hôte de session du pool hôte Windows Virtual Desktop.

2. [Téléchargez et installez FSLogix](/fslogix/install-ht).

5. Suivez les instructions de [Configurer les paramètres du registre de conteneurs de profils](/fslogix/configure-profile-container-tutorial#configure-profile-container-registry-settings) :

    - Accédez à **Computer** > **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **FSLogix**.

    - Créez une clé **Profils**.

    - Créez **Enabled, DWORD** avec la valeur 1.

    - Créez **VHDLocations, MULTI_SZ**.

    - Définissez la valeur de **VHDLocations** sur le chemin d’accès UNC que vous avez généré dans [Obtenir le chemin d’accès UNC](#get-the-unc-path).

6. Redémarrez la machine virtuelle.

## <a name="testing"></a>Test

Une fois que vous avez installé et configuré FSLogix, vous pouvez tester votre déploiement en vous connectant avec un compte d’utilisateur auquel un groupe d’applications ou un ordinateur de bureau a été affecté sur le pool hôte. Assurez-vous que le compte d’utilisateur avec lequel vous vous connectez dispose de l’autorisation sur le partage de fichiers.

Si l’utilisateur s’est connecté précédemment, il dispose d’un profil local qui sera utilisé lors de cette session. Pour éviter de créer un profil local, créez un nouveau compte d’utilisateur à utiliser pour les tests ou utilisez les méthodes de configuration décrites dans [Didacticiel : configurer le conteneur de profil pour rediriger les profils utilisateurs](/fslogix/configure-profile-container-tutorial/).

Pour vérifier vos autorisations sur votre session :

1. Démarrez une session sur Windows Virtual Desktop.

2. Ouvrez le portail Azure.

3. Ouvrez le compte de stockage que vous avez créé dans [Installer un compte de stockage](#set-up-a-storage-account).

4. Sélectionnez **Créer un partage** sur la page Créer un partage de fichiers Azure.

5. Assurez-vous qu’un dossier contenant le profil utilisateur existe désormais dans vos fichiers.

Pour des tests supplémentaires, suivez les instructions de [Vérifier que votre profil fonctionne](create-profile-container-adds.md#make-sure-your-profile-works).

## <a name="next-steps"></a>Étapes suivantes

Pour résoudre les problèmes liés à FSLogix, consultez [ce guide de dépannage](/fslogix/fslogix-trouble-shooting-ht).
