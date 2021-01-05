---
title: Créer un conteneur de profil FSLogix avec Azure Files et Active Directory Domain Services - Azure
description: Cet article explique comment créer un conteneur de profil FSLogix avec Azure Files et Azure Active Directory Domain Services.
author: Heidilohr
ms.topic: how-to
ms.date: 04/10/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 70a56b7efc34ba2fd3c06521c6e4cac6ea28778f
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302471"
---
# <a name="create-a-profile-container-with-azure-files-and-azure-ad-ds"></a>Créer un conteneur de profil avec Azure Files et Azure AD DS

Cet article vous montre comment créer un conteneur de profil FSLogix avec Azure Files et Azure Active Directory Domain Services (AD DS).

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà configuré une instance Azure AD DS. Si vous n’en avez pas encore une, suivez d’abord les instructions indiquées dans [Créer un domaine managé de base](../active-directory-domain-services/tutorial-create-instance.md), puis revenez ici.

## <a name="add-azure-ad-ds-admins"></a>Ajouter des administrateurs Azure AD DS

Pour ajouter des administrateurs, vous devez créer un utilisateur et lui accorder des autorisations.

Pour ajouter un administrateur :

1. Sélectionnez **Azure Active Directory** dans la barre latérale, sélectionnez **Tous les utilisateurs**, puis sélectionnez **Nouvel utilisateur**.

2.  Entrez les détails de l’utilisateur dans les champs.

3. Dans le volet Azure Active Directory sur le côté gauche de l’écran, sélectionnez **Groupes**.

4. Sélectionnez le groupe **Administrateurs AAD DC**.

5. Dans le volet gauche, sélectionnez **Membres**, puis sélectionnez **Ajouter des membres** dans le volet principal. Cette opération affiche la liste de tous les utilisateurs disponibles dans Azure AD. Sélectionnez le nom du profil utilisateur que vous venez de créer.

## <a name="set-up-an-azure-storage-account"></a>Configurer un compte Stockage Azure

À présent, il est temps d’activer l’authentification Azure AD DS sur SMB (Server Message Block).

Pour activer l’authentification :

1. Si ce n’est déjà fait, configurez et déployez un compte Stockage Azure v2 universel en suivant les instructions fournies dans [Créer un compte Stockage Azure](../storage/common/storage-account-create.md).

2. Une fois votre compte configuré, sélectionnez **Accéder à la ressource**.

3. Sélectionnez **Configuration** dans le volet situé à gauche de l’écran, puis activez **Authentification Azure Active Directory pour Azure Files** dans le volet principal. Quand vous avez terminé, sélectionnez **Enregistrer**.

4. Sélectionnez **Vue d’ensemble** dans le volet situé à gauche de l’écran, puis sélectionnez **Fichiers** dans le volet principal.

5. Sélectionnez **Partage de fichiers**, puis entrez le **Nom** et le **Quota** dans les champs qui apparaissent sur le côté droit de l’écran.

## <a name="assign-access-permissions-to-an-identity"></a>Assigner des autorisations d’accès à une identité

D’autres utilisateurs auront besoin d’autorisations d’accès pour accéder à votre partage de fichiers. Pour ce faire, vous devez affecter à chaque utilisateur un rôle disposant des autorisations d’accès appropriées.

Pour attribuer des autorisations d’accès aux utilisateurs :

1. À partir du portail Azure, ouvrez le partage de fichiers que vous avez créé dans la section [Configurer un compte Stockage Azure](#set-up-an-azure-storage-account).

2. Sélectionnez **Contrôle d’accès (IAM)** .

3. Sélectionnez **Ajouter une attribution de rôle**.

4. Sous l’onglet **Ajouter une attribution de rôle**, sélectionnez le rôle intégré approprié dans la liste des rôles. Pour que le compte obtienne les autorisations appropriées, vous devez au moins sélectionner **Collaborateur de partage SMB des données du fichier de stockage**.

5. Dans **Attribuer l’accès à**, sélectionnez **Utilisateur Azure Active Directory, groupe ou principal de service**.

6. Sélectionnez un nom ou une adresse e-mail pour l’identité Azure Active Directory cible.

7. Sélectionnez **Enregistrer**.

## <a name="get-the-storage-account-access-key"></a>Obtenir la clé d’accès du compte de stockage

Ensuite, vous devez obtenir la clé d’accès de votre compte de stockage.

Pour obtenir la clé d’accès du compte de stockage :

1. Dans la barre latérale du portail Azure, sélectionnez **Comptes de stockage**.

2. Dans la liste des comptes de stockage, sélectionnez le compte pour lequel vous avez activé Azure AD DS et créé les rôles personnalisés au cours des étapes ci-dessus.

3. Sous **Paramètres**, sélectionnez **Clés d’accès** et copiez la clé à partir de **key1**.

4. Accédez à l’onglet **Machines virtuelles** et recherchez les machines virtuelles qui feront partie de votre pool hôte.

5. Sélectionnez le nom de la machine virtuelle sous **Machines virtuelles (adVM)** et sélectionnez **Se connecter**.

    Cette action télécharge un fichier RDP qui vous permettra de vous connecter à la machine virtuelle avec ses propres informations d’identification.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’onglet RDP de la fenêtre de connexion à la machine virtuelle.](media/rdp-tab.png)

6. Une fois connecté à la machine virtuelle, exécutez une invite de commandes en tant qu’administrateur.

7. Exécutez la commande suivante :

     ```cmd
     net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> /user:Azure\<storage-account-name> <storage-account-key>
     ```

    - Remplacez `<desired-drive-letter>` par la lettre de lecteur de votre choix (par exemple, `y:`).
    - Remplacez toutes les instances de `<storage-account-name>` par le nom du compte de stockage que vous avez spécifié.
    - Remplacez `<share-name>` par le nom du partage que vous avez créé.
    - Remplacez `<storage-account-key>` par la clé de compte de stockage récupérée d’Azure.

    Par exemple :

     ```cmd
     net use y: \\fsprofile.file.core.windows.net\share HDZQRoFP2BBmoYQ=(truncated)= /user:Azure\fsprofile)
     ```

8. Exécutez les commandes suivantes pour permettre à vos utilisateurs Windows Virtual Desktop de créer leur propre conteneur de profils tout en bloquant l’accès aux conteneurs de profils par d’autres utilisateurs.

     ```cmd
     icacls <mounted-drive-letter>: /grant <user-email>:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

    - Remplacez `<mounted-drive-letter>` par la lettre du lecteur que vous avez utilisé pour mapper le lecteur.
    - Remplacez `<user-email>` par l’UPN de l’utilisateur ou du groupe Active Directory qui contient les utilisateurs qui auront besoin d’accéder au partage.

    Par exemple :

     ```cmd
     icacls <mounted-drive-letter>: /grant john.doe@contoso.com:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

## <a name="create-a-profile-container"></a>Créer un conteneur de profils

Vos profils étant prêts, nous allons créer un conteneur de profil FSLogix.

Pour configurer un conteneur de profil FSLogix :

1. Connectez-vous à la machine virtuelle hôte de session que vous avez configurée au début de cet article, puis [téléchargez et installez l’agent FSLogix](/fslogix/install-ht/).

2. Décompressez le fichier de l’agent FSLogix que vous avez téléchargé et accédez à **x64** > **Releases**, puis ouvrez **FSLogixAppsSetup.exe**.

3. Une fois le programme d’installation lancé, sélectionnez **J’accepte les conditions générales de la licence**. Le cas échéant, fournissez une nouvelle clé.

4. Sélectionnez **Installer**.

5. Ouvrez le **lecteur C**, puis accédez à **Fichiers programmes** > **FSLogix** > **Applications** pour vous assurer que l’agent FSLogix a été correctement installé.

     >[!NOTE]
     > S’il y a plusieurs machines virtuelles dans le pool hôte, vous devez répéter les étapes 1 à 5 pour chaque machine virtuelle.

6. Exécutez l’**Éditeur du Registre** (RegEdit) en tant qu’administrateur.

7. Accédez à **Ordinateur** > **HKEY_LOCAL_MACHINE** > **software** > **FSLogix**, cliquez avec le bouton droit sur **FSLogix**, sélectionnez **Nouveau**, puis sélectionnez **Clé**.

8. Créez une clé nommée **Profiles**.

9.  Cliquez avec le bouton droit sur **Profiles**, sélectionnez **Nouveau**, puis sélectionnez **Valeur DWORD (32 bits)** . Nommez la valeur **Enabled** et définissez la valeur **Données** sur **1**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la clé Profiles. Le fichier REG_DWORD est mis en surbrillance et sa valeur Données est définie sur 1.](media/dword-value.png)

10. Cliquez avec le bouton droit sur **Profiles**, sélectionnez **Nouveau**, puis sélectionnez **Valeur de chaînes multiples**. Nommez la valeur **VHDLocations** et entrez l’URI du partage Azure Files `\\fsprofile.file.core.windows.net\share` comme valeur de Données.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la clé Profiles montrant le fichier VHDLocations. Sa valeur Données indique l’URI du partage Azure Files.](media/multi-string-value.png)

## <a name="assign-users-to-a-session-host"></a>Attribuer des utilisateurs à un hôte de la session

À présent, vous devez attribuer des utilisateurs à votre hôte de session.

Pour attribuer des utilisateurs :

1. Exécutez Windows PowerShell en tant qu’administrateur, puis exécutez l’applet de commande suivante pour vous connecter à Windows Virtual Desktop avec PowerShell :

   ```powershell
   Import-Module Microsoft.RdInfra.RdPowershell

   #Optional
   Install-Module Microsoft.RdInfra.RdPowershell

   $brokerurl = "https://rdbroker.wvd.microsoft.com"

   Add-RdsAccount -DeploymentUrl $brokerurl
   ```

   Quand vous êtes invité à entrer des informations d’identification, indiquez l’utilisateur auquel a été accordé le rôle TenantCreator, Propriétaire RDS ou Contributeur RDS sur le locataire Windows Virtual Desktop.

2. Exécutez les applets de commande suivantes pour attribuer l’utilisateur au groupe Bureau à distance :

     ```powershell
     $tenant = "<your-wvd-tenant>"

     $pool1 = "<wvd-pool>"

     $appgroup = "Desktop Application Group"

     $user1 = "<user-principal>"

     Add-RdsAppGroupUser $tenant $pool1 $appgroup $user1
     ```

    Comme pour les applets de commande précédentes, veillez à remplacer `<your-wvd-tenant>`, `<wvd-pool>` et `<user-principal>` par les valeurs appropriées.

    Par exemple :

     ```powershell
     $pool1 = "contoso"

     $tenant = "contoso"

     $appgroup = "Desktop Application Group"

     $user1 = "jane.doe@contoso.com"

     Add-RdsAppGroupUser $tenant $pool1 $appgroup $user1
     ```

## <a name="make-sure-your-profile-works"></a>Vérifier que votre profil fonctionne

Maintenant, il ne vous reste plus qu’à vérifier que le profil que vous avez créé existe et fonctionne comme prévu.

Pour vérifier votre profil :

1. Ouvrez un navigateur et accédez au [client web Windows Virtual Desktop](https://rdweb.wvd.microsoft.com/arm/webclient).

2. Connectez-vous avec le compte d’utilisateur attribué au groupe Bureau à distance.

3. Une fois la session utilisateur établie, ouvrez le portail Azure et connectez-vous avec un compte d’administrateur.

4. Dans la barre latérale, sélectionnez **Comptes de stockage**.

5. Sélectionnez le compte de stockage que vous avez configuré comme partage de fichiers pour votre pool hôte de session et activé avec Azure AD DS.

6. Sélectionnez l’icône **Fichiers**, puis développez votre partage.

    Si tout est correctement configuré, vous devriez voir un **annuaire** dont le nom présente la mise en forme suivante : `<user SID>-<username>`.

## <a name="next-steps"></a>Étapes suivantes

Si vous recherchez d’autres façons de créer des conteneurs de profil FSLogix, consultez les articles suivants :

- [Créer un conteneur de profils pour un pool hôte à l’aide d’un partage de fichiers](create-host-pools-user-profile.md)
- [Créer un conteneur de profils FSLogix pour un pool d’hôtes à l'aide d’Azure NetApp Files](create-fslogix-profile-container.md)

Vous trouverez des informations plus détaillées sur les concepts liés aux conteneurs FSlogix pour les fichiers Azure dans [Conteneurs de profil FSLogix et fichiers Azure](fslogix-containers-azure-files.md).
