---
title: Conteneurs de profil FSLogix NetApp dans Windows Virtual Desktop - Azure
description: Créer un conteneur de profils FSLogix à l'aide d'Azure NetApp Files dans Windows Virtual Desktop
author: Heidilohr
ms.topic: how-to
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6a9f2c62d8e7f17f6ea8377982c79fef3dfbb97c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016826"
---
# <a name="create-a-profile-container-with-azure-netapp-files-and-ad-ds"></a>Créer un conteneur de profil avec Azure NetApp Files et Azure AD DS

Nous vous recommandons d’utiliser des conteneurs de profils FSLogix en tant que solution de profil utilisateur pour le [service Windows Virtual Desktop](overview.md). Les conteneurs de profils FSLogix stockent un profil utilisateur complet dans un même conteneur et sont conçus pour l'itinérance des profils dans les environnements informatiques distants non persistants comme Windows Virtual Desktop. Lorsque vous vous connectez, le conteneur se connecte de manière dynamique à l'environnement informatique à l'aide d'un disque dur virtuel (VHD) et d'un disque dur virtuel Hyper-V (VHDX) pris en charge localement. Grâce à ces technologies avancées de pilote de filtre, le profil utilisateur est immédiatement disponible et apparaît dans le système exactement comme un profil utilisateur local. Pour en savoir plus sur les conteneurs de profils FSLogix, consultez [Conteneurs de profils FSLogix et fichiers Azure](fslogix-containers-azure-files.md).

Vous pouvez créer des conteneurs de profils FSLogix à l'aide d'[Azure NetApp Files](https://azure.microsoft.com/services/netapp/), un service de plateforme natif d'Azure facile à utiliser qui aide les clients à approvisionner rapidement et en toute fiabilité des volumes SMB de classe entreprise pour leurs environnements Windows Virtual Desktop. Pour en savoir plus sur Azure NetApp Files, consultez [Présentation d'Azure NetApp Files](../azure-netapp-files/azure-netapp-files-introduction.md).

Ce guide vous apprendra à configurer un compte Azure NetApp Files et à créer des conteneurs de profils FSLogix dans Windows Virtual Desktop.

Cet article part du principe que vous avez déjà configuré et regroupé des [pools d'hôtes](create-host-pools-azure-marketplace.md) au sein d'un ou plusieurs locataires de votre environnement Windows Virtual Desktop. Pour en savoir plus sur la configuration de locataires, consultez la section [Créer un locataire dans Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md) ainsi que le [billet de blog de notre communauté technique](https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/Getting-started-with-Windows-Virtual-Desktop/ba-p/391054).

Les instructions fournies dans ce guide sont spécifiquement destinées aux utilisateurs de Windows Virtual Desktop. Si vous recherchez des conseils plus généraux sur la configuration d'Azure NetApp Files et sur la création de conteneurs de profils FSLogix en dehors de Windows Virtual Desktop, consultez la section [Configurer Azure NetApp Files et créer un volume NFS - Démarrage rapide](../azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes.md).

>[!NOTE]
>Cet article ne couvre pas les meilleures pratiques relatives à la sécurisation de l'accès au partage Azure NetApp Files.

>[!NOTE]
>Si vous recherchez des éléments de comparaison sur les différentes options de stockage pour les conteneurs de profil FSLogix sur Azure, consultez [Options de stockage pour les conteneurs de profil FSLogix](store-fslogix-profile.md).

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir créer un conteneur de profils FSLogix pour un pool d'hôtes, vous devez :

- Installer et configurer Windows Virtual Desktop
- Approvisionner un pool d'hôtes Windows Virtual Desktop
- [Activer votre abonnement Azure NetApp Files](../azure-netapp-files/azure-netapp-files-register.md)

## <a name="set-up-your-azure-netapp-files-account"></a>Configurer votre compte Azure NetApp Files

Pour commencer, vous devez configurer un compte Azure NetApp Files.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Assurez-vous que votre compte dispose d'autorisations de contributeur ou d'administrateur.

2. Sélectionnez l'**icône Azure Cloud Shell** à droite de la barre de recherche pour ouvrir Azure Cloud Shell.

3. Une fois Azure Cloud Shell ouvert, sélectionnez **PowerShell**.

4. Si vous utilisez Azure Cloud Shell pour la première fois, créez un compte de stockage avec l'abonnement associé à Azure NetApp Files et Windows Virtual Desktop.

   > [!div class="mx-imgBorder"]
   > ![Fenêtre du compte de stockage avec bouton Créer un stockage encadré en rouge en bas de la fenêtre.](media/create-storage-button.png)

5. Une fois Azure Cloud Shell chargé, exécutez les deux cmdlets suivantes.

   ```azurecli
   az account set --subscription <subscriptionID>
   ```

   ```azurecli
   az provider register --namespace Microsoft.NetApp --wait
   ```

6. Dans la partie gauche de la fenêtre, sélectionnez **Tous les services**. Entrez **Azure NetApp Files** dans la zone de recherche qui s'affiche en haut du menu.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran d’un utilisateur entrant « Azure NetApp Files » dans la zone de recherche Tous les services. Les résultats de la recherche affichent la ressource Azure NetApp Files.](media/azure-netapp-files-search-box.png)


7. Sélectionnez **Azure NetApp Files** dans les résultats de la recherche, puis cliquez sur **Créer**.

8. Sélectionnez le bouton **Ajouter**.
9. Lorsque l’onglet **Nouveau compte NetApp** s’ouvre, entrez les valeurs suivantes :

    - Dans le champ **Nom**, entrez le nom de votre compte NetApp.
    - Dans le menu déroulant **Abonnement**, sélectionnez l'abonnement correspondant au compte de stockage que vous avez configuré à l'étape 4.
    - Dans le menu déroulant **Groupe de ressources**, sélectionnez un groupe de ressources existant, ou créez-en un en sélectionnant **Créer**.
    - Dans le menu déroulant **Emplacement**, sélectionnez la région à laquelle votre compte NetApp est associé. Cette région doit être la même que celle de vos machines virtuelles hôtes de session.

   >[!NOTE]
   >Azure NetApp Files ne prend actuellement pas en charge le montage d'un volume sur plusieurs régions.

10. Lorsque vous avez terminé, sélectionnez **Créer** pour créer votre compte NetApp.

## <a name="create-a-capacity-pool"></a>Créer un pool de capacités

Créez ensuite un pool de capacités :

1. Accédez au menu Azure NetApp Files et sélectionnez votre nouveau compte.
2. Dans le menu de votre compte, sélectionnez **Pools de capacités** sous Service de stockage.
3. Sélectionnez **Ajouter un pool**.
4. Lorsque l’onglet **Nouveau pool de capacités** s’ouvre, entrez les valeurs suivantes :

    - Dans le champ **Nom**, entrez le nom du nouveau pool de capacités.
    - Dans le menu déroulant **Niveau de service**, sélectionnez la valeur de votre choix. Nous recommandons le niveau **Premium** pour la plupart des environnements.
       >[!NOTE]
       >Le paramètre Premium fournit le débit minimum disponible pour un niveau de service Premium, à savoir 256 Mbits/s. Pour un environnement de production, ce débit devra peut-être être ajusté. Le débit final est basé sur la relation décrite dans [Limites de débit](../azure-netapp-files/azure-netapp-files-service-levels.md).
    - Dans le champ **Taille (Tio)** , entrez la taille de pool de capacités qui correspond le mieux à vos besoins. La taille minimum est de 4 Tio.

5. Lorsque vous avez terminé, sélectionnez **OK**.

## <a name="join-an-active-directory-connection"></a>Joindre une connexion Active Directory

Vous devez ensuite joindre une connexion Active Directory.

1. Sélectionnez **Connexions Active Directory** dans le menu situé à gauche de la page, puis sélectionnez le bouton **Joindre** pour ouvrir la page **Joindre Active Directory**.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran du menu Joindre des connexions Active Directory.](media/active-directory-connections-menu.png)

2. Entrez les valeurs suivantes sur la page **Joindre Active Directory** pour joindre une connexion :

    - Dans le champ **DNS principal**, entrez l'adresse IP du serveur DNS de votre environnement qui peut résoudre le nom de domaine.
    - Dans le champ **Domaine**, entrez votre nom de domaine complet (FQDN).
    - Dans le champ **Préfixe de serveur SMB (compte d'ordinateur)** , entrez la chaîne que vous souhaitez ajouter au nom du compte d'ordinateur.
    - Dans le champ **Nom d'utilisateur**, entrez le nom du compte disposant des autorisations nécessaires pour effectuer la jonction de domaine.
    - Dans le champ **Mot de passe**, entrez le mot de passe du compte.

## <a name="create-a-new-volume"></a>Créer un volume

Vous devez ensuite créer un volume.

1. Sélectionnez **Volumes**, puis **Ajouter un volume**.

2. Lorsque l’onglet **Créer un volume** s’ouvre, entrez les valeurs suivantes :

    - Dans le champ **Nom du volume**, entrez le nom du nouveau volume.
    - Dans le menu déroulant **Pool de capacités**, sélectionnez le pool de capacités que vous venez de créer.
    - Dans le champ **Quota (Gio)** , entrez la taille de volume appropriée pour votre environnement.
    - Dans le menu déroulant **Réseau virtuel**, sélectionnez un réseau virtuel existant connecté au contrôleur de domaine.
    - Sous **Sous-réseau**, sélectionnez **Créer**. Gardez à l'esprit que ce sous-réseau sera délégué à Azure NetApp Files.

3.  Sélectionnez **Suivant : Protocole \>\>** pour ouvrir l'onglet Protocole et configurer vos paramètres d'accès au volume.

## <a name="configure-volume-access-parameters"></a>Configurer les paramètres d'accès au volume

Une fois le volume créé, configurez les paramètres d'accès à celui-ci.

1.  Sélectionnez **SMB** comme type de protocole.
2.  Sous Configuration, dans le menu déroulant **Active Directory**, sélectionnez le répertoire que vous avez initialement connecté dans [Joindre une connexion Active Directory](create-fslogix-profile-container.md#join-an-active-directory-connection). N'oubliez pas que la limite est fixée à une seule instance d'Active Directory par abonnement.
3.  Dans la zone de texte **Nom du partage**, entrez le nom du partage utilisé par le pool d'hôtes de session et ses utilisateurs.

4.  Au bas de la page, sélectionnez **Examiner et créer**. La page de validation s'ouvre. Une fois le volume validé, sélectionnez **Créer**.

5.  Le déploiement du nouveau volume commence alors. Au terme du déploiement, vous pouvez utiliser le partage Azure NetApp Files.

6.  Pour afficher le chemin de montage, sélectionnez **Accéder à la ressource** et recherchez-le dans l'onglet Vue d'ensemble.

    > [!div class="mx-imgBorder"]
    > ![Capture de l’écran Vue d’ensemble avec une flèche rouge pointant vers le chemin de montage.](media/overview-mount-path.png)

## <a name="configure-fslogix-on-session-host-virtual-machines-vms"></a>Configurer FSLogix sur des machines virtuelles hôtes de session

Cette section est basée sur [Créer un conteneur de profils pour un pool d'hôtes à l'aide d'un partage de fichiers](create-host-pools-user-profile.md).

1. Tant que vous êtes encore à distance, [téléchargez le fichier .zip de l'agent FSLogix](https://go.microsoft.com/fwlink/?linkid=2084562&clcid=0x409) sur la machine virtuelle hôte de la session.

2. Décompressez le fichier téléchargé.

3. Dans le fichier, accédez à **Versions** > **x64** et exécutez **FSLogixAppsSetup.exe**. Le menu d'installation s'ouvre.

4.  Si vous disposez d'une clé de produit, entrez-la dans la zone de texte Clé de produit.

5. Cochez la case située en regard de **J'accepte les conditions de la licence**.

6. Sélectionnez **Installer**.

7. Accédez à **C:\\Program Files\\FSLogix\\Apps** pour vérifier l'agent installé.

8. Dans le menu Démarrer, exécutez **RegEdit** en tant qu'administrateur.

9. Accédez à **Computer\\HKEY_LOCAL_MACHINE\\software\\FSLogix**.

10. Créez une clé nommée **Profils**.

11.  Créez une valeur nommée **Enabled** avec un type **REG_DWORD** défini sur la valeur de donnée **1**.

12. Créez une valeur nommée **VHDLocations** avec un type **Multi-String** et définissez sa valeur de données sur l'URI du partage Azure NetApp Files.

13. Créez une valeur nommée **DeleteLocalProfileWhenVHDShouldApply** avec une valeur DWORD de 1 pour éviter les problèmes avec les profils locaux existants avant de vous connecter.

     >[!WARNING]
     >Soyez prudent lors de la création de la valeur DeleteLocalProfileWhenVHDShouldApply. Lorsque le système FSLogix Profiles détermine qu’un utilisateur doit avoir un profil FSLogix, mais qu’un profil local existe déjà, le conteneur de profils supprime définitivement le profil local. L’utilisateur est ensuite connecté avec le nouveau profil FSLogix.

## <a name="assign-users-to-session-host"></a>Attribuer des utilisateurs à l'hôte de la session

1. Ouvrez **PowerShell ISE** en tant qu'administrateur et connectez-vous à Windows Virtual Desktop.

2. Exécutez les applets de commande suivantes :

   ```powershell
   Import-Module Microsoft.RdInfra.RdPowershell
   # (Optional) Install-Module Microsoft.RdInfra.RdPowershell
   $brokerurl = "https://rdbroker.wvd.microsoft.com"
   Add-RdsAccount -DeploymentUrl $brokerurl
   ```

3. Lorsque vous êtes invité à entrer des informations d'identification, saisissez les informations d'identification de l'utilisateur doté du rôle Créateur de locataire ou Propriétaire RDS/Contributeur RDS sur le locataire Windows Virtual Desktop.

4. Exécutez les cmdlets suivantes pour attribuer un utilisateur à un groupe Bureau à distance :

   ```powershell
   $wvdTenant = "<your-wvd-tenant>"
   $hostPool = "<wvd-pool>"
   $appGroup = "Desktop Application Group"
   $user = "<user-principal>"
   Add-RdsAppGroupUser $wvdTenant $hostPool $appGroup $user
   ```

## <a name="make-sure-users-can-access-the-azure-netapp-file-share"></a>Vérifier que les utilisateurs ont accès au partage Azure NetApp Files

1. Ouvrez votre navigateur Internet et accédez à <https://rdweb.wvd.microsoft.com/arm/webclient>.

2. Connectez-vous à l'aide des informations d'identification d'un utilisateur attribué au groupe Bureau à distance.

3. Une fois la session utilisateur établie, connectez-vous au portail Azure à l'aide d'un compte administratif.

4. Ouvrez **Azure NetApp Files**, sélectionnez votre compte Azure NetApp Files, puis cliquez sur **Volumes**. Lorsque le menu Volumes s'ouvre, sélectionnez le volume correspondant.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran du compte NetApp que vous avez précédemment configuré sur le portail Azure avec bouton Volumes sélectionné.](media/netapp-account.png)

5. Accédez à l'onglet **Vue d'ensemble** et vérifiez que le conteneur de profils FSLogix utilise de l'espace.

6. Connectez-vous directement à une partie quelconque de la machine virtuelle du pool d'hôtes à l'aide de Bureau à distance et ouvrez l'**Explorateur de fichiers**. Accédez ensuite au **Chemin de montage** (dans l'exemple suivant, le chemin de montage est \\\\anf-SMB-3863.gt1107.onmicrosoft.com\\anf-VOL).

   Ce dossier doit contenir un fichier VHD (ou VHDX) de profils comme celui de l'exemple suivant.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran du contenu du dossier dans le chemin de montage. Celui-ci contient un fichier VHD nommé « Profile_ssbb ».](media/mount-path-folder.png)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez utiliser des conteneurs de profils FSLogix pour configurer un partage de profil utilisateur. Pour en savoir plus sur la création de partages de profils utilisateur avec vos nouveaux conteneurs, consultez [Créer un conteneur de profils pour un pool d'hôtes à l'aide d'un partage de fichiers](create-host-pools-user-profile.md).

Vous pouvez également créer un partage de fichiers Azure Files, dans lequel stocker votre profil FSLogix. Pour en savoir plus, voir [Créer un partage de fichiers Azure Files avec un contrôleur de domaine](create-file-share.md).