---
title: Pool d’hôtes Windows Virtual Desktop (classique) de la Place de marché Azure – Azure
description: Guide pratique pour créer un pool d’hôtes Windows Virtual Desktop (classique) à l’aide de la Place de marché Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 04184a658ae1efd03afd3e920818eb5317bc7553
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88008591"
---
# <a name="tutorial-create-a-host-pool-in-windows-virtual-desktop-classic"></a>Tutoriel : Créer un pool d’hôtes dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../create-host-pools-azure-marketplace.md).

Dans ce tutoriel, vous allez apprendre à créer un pool d’hôtes avec un locataire Windows Virtual Desktop à l’aide d’une offre de la Place de marché Microsoft Azure.

Les pools d'hôtes sont des ensembles d'une ou de plusieurs machines virtuelles identiques dans des environnements de locataires Windows Virtual Desktop. Chaque pool d’hôtes peut contenir un groupe d’applications avec lequel les utilisateurs peuvent interagir comme ils le feraient sur un ordinateur de bureau physique.

Les tâches de ce tutoriel sont les suivantes :

> [!div class="checklist"]
>
> * Créez un pool d’hôtes dans Windows Virtual Desktop.
> * Créez un groupe de ressources avec des machines virtuelles dans un abonnement Azure.
> * Joignez les machines virtuelles au domaine Active Directory.
> * Inscrivez les machines virtuelles avec Windows Virtual Desktop.

## <a name="prerequisites"></a>Prérequis

* Un locataire dans Virtual Desktop. Un précédent [tutoriel](tenant-setup-azure-active-directory.md) crée un locataire.
* [Module PowerShell Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/).

Une fois que vous disposez de ce module, exécutez l’applet de commande suivante pour vous connecter à votre compte :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="run-the-azure-marketplace-offering-to-provision-a-new-host-pool"></a>Exécuter l’offre de la Place de marché Azure pour provisionner un nouveau pool d’hôtes

Pour exécuter l’offre de la Place de marché Azure pour provisionner un nouveau pool d’hôtes :

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.
1. Entrez **Windows Virtual Desktop** dans la fenêtre de recherche de la Place de marché.
1. Sélectionnez **Windows Virtual Desktop - Provisionner un pool d’hôtes**, puis **Créer**.

Ensuite, suivez les instructions indiquées dans la section suivante pour entrer les informations relatives aux onglets appropriés.

### <a name="basics"></a>Concepts de base

Voici la procédure à suivre pour l’onglet **Informations de base** :

1. Sélectionnez un **Abonnement**.
1. Pour **Groupe de ressources**, sélectionnez **Création** et indiquez un nom pour le nouveau groupe de ressources.
1. Sélectionnez une **Région**.
1. Entrez un nom pour le pool d’hôtes. Il doit être unique dans le locataire Windows Virtual Desktop.
1. Sélectionnez **Type de bureau**. Si vous sélectionnez **Personnel**, chaque utilisateur qui se connecte à ce pool d’hôtes est affecté de façon permanente à une machine virtuelle.
1. Entrez les utilisateurs qui peuvent se connecter aux clients Windows Virtual Desktop et accéder à un bureau. Utilisez une liste séparée par des virgules. Par exemple, si vous voulez accorder l’accès à `user1@contoso.com` et `user2@contoso.com`, entrez *`user1@contoso.com,user2@contoso.com`* .
1. Pour **Emplacement des métadonnées de service**, sélectionnez le même emplacement que le réseau virtuel qui dispose d’une connectivité au serveur Active Directory.

   >[!IMPORTANT]
   >Si vous utilisez une solution Azure Active Directory Domain Services (Azure AD DS) et Azure Active Directory (Azure AD) pure, veillez à déployer votre pool d’hôtes dans la même région que votre instance Azure AD DS pour éviter les erreurs de jonction de domaine et d’informations d’identification.

1. Sélectionnez **Suivant : Configurer des machines virtuelles**.

### <a name="configure-virtual-machines"></a>Configurer des machines virtuelles

Pour l’onglet **Configurer des machines virtuelles** :

1. Acceptez les valeurs par défaut ou personnalisez le nombre et la taille des machines virtuelles.

    >[!NOTE]
    >Si la taille de machine virtuelle que vous recherchez n’apparaît pas dans le sélecteur de taille de machine virtuelle, cela signifie que nous ne l’avons pas encore intégrée à l’outil Place de marché Azure. Pour demander une taille, créez une demande ou votez pour une demande existante dans le [forum UserVoice sur Windows Virtual Desktop](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).

1. Entrez un préfixe pour le nom des machines virtuelles. Par exemple, si vous entrez le nom *préfixe*, les machines virtuelles s’appelleront **préfixe-0**, **préfixe-1**, etc.
1. Sélectionnez **Suivant : Paramètres de la machine virtuelle**.

### <a name="virtual-machine-settings"></a>Paramètres de la machine virtuelle

Pour l’onglet **Paramètres de la machine virtuelle** :

1. Pour **Source de l’image**, sélectionnez la source, puis entrez les informations appropriées concernant la façon de la rechercher et de la stocker. Vos options diffèrent pour le stockage d’objets Blob, les images managées et la galerie.

   Si vous choisissez de ne pas utiliser de disques managés, sélectionnez le compte de stockage qui contient le fichier  *.vhd*.
1. Entrez le nom d’utilisateur principal et le mot de passe. Ce compte doit être le compte de domaine destiné à joindre les machines virtuelles au domaine Active Directory. Ces mêmes nom d’utilisateur et mot de passe sont créés sur les machines virtuelles sous la forme d’un compte local. Vous pouvez réinitialiser ces comptes locaux ultérieurement.

   >[!NOTE]
   > Si vous joignez vos machines virtuelles à un environnement Azure AD DS, vérifiez que votre utilisateur de jonction de domaine est membre du [groupe Administrateurs AAD DC](../../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group).
   >
   > Le compte doit également faire partie du domaine géré par Azure AD DS ou du locataire Azure AD. Les comptes de répertoires externes associés à votre Client Azure AD ne peuvent pas s’authentifier correctement au cours du processus de jonction de domaine.

1. Sélectionnez le **Réseau virtuel** qui dispose d’une connectivité au serveur Active Directory, puis choisissez un sous-réseau pour héberger les machines virtuelles.
1. Sélectionnez **Suivant : Informations sur Windows Virtual Desktop**.

### <a name="windows-virtual-desktop-tenant-information"></a>Informations sur le locataire Windows Virtual Desktop

Pour l’onglet **Informations sur le locataire Windows Virtual Desktop** :

1. Dans **Nom du groupe de locataires Windows Virtual Desktop**, entrez le nom du groupe de locataires qui contient votre locataire. Conservez la valeur par défaut, sauf si un nom de groupe de locataires spécifique vous a été fourni.
1. Dans **Nom du locataire Windows Virtual Desktop**, entrez le nom du locataire que vous allez créer dans ce pool d’hôtes.
1. Spécifiez le type d’informations d’identification que vous voulez utiliser pour vous authentifier comme propriétaire des services Bureau à distance (RDS) du locataire Windows Virtual Desktop. Entrez l’UPN ou le principal de service et un mot de passe.

   Si vous avez suivi le tutoriel [Créer des principaux de service et des attributions de rôles avec PowerShell](create-service-principal-role-powershell.md), sélectionnez **Principal de service**.

1. Pour **Principal de service**, pour **ID de locataire Azure AD**, entrez le compte d’administrateur de locataire correspondant à l’instance d’Azure AD qui contient le principal de service. Seuls les principaux de service avec des informations d’identification de mot de passe sont pris en charge.
1. Sélectionnez **Suivant : Vérifier + créer**.

## <a name="complete-setup-and-create-the-virtual-machine"></a>Terminer la configuration et la création de la machine virtuelle

Dans **Vérifier et créer**, passez en revue les informations de configuration. Si vous avez besoin de changer quelque chose, revenez en arrière et apportez les modifications. Quand vous êtes prêt, sélectionnez **Créer** pour déployer votre pool d’hôtes.

En fonction du nombre de machines virtuelles que vous créez, ce processus peut prendre 30 minutes, voire plus.

>[!IMPORTANT]
> Pour contribuer à sécuriser votre environnement Windows Virtual Desktop dans Azure, nous vous recommandons de ne pas ouvrir le port entrant 3389 sur vos machines virtuelles. Windows Virtual Desktop ne nécessite pas l’ouverture du port entrant 3389 pour permettre aux utilisateurs d’accéder aux machines virtuelles du pool d’hôtes.
>
> Si vous devez ouvrir le port 3389 pour résoudre des problèmes, nous vous recommandons d’utiliser un accès à la machine virtuelle juste-à-temps. Pour plus d’informations, consultez [Sécuriser vos ports de gestion avec un accès juste-à-temps](../../security-center/security-center-just-in-time.md).

## <a name="optional-assign-additional-users-to-the-desktop-application-group"></a>(Facultatif) Affecter des utilisateurs supplémentaires au groupe d’applications de bureau

Une fois que la Place de marché Azure a terminé la création du pool, vous pouvez affecter davantage d’utilisateurs au groupe d’applications de bureau. Si vous ne souhaitez pas en ajouter, ignorez cette section.

Pour affecter des utilisateurs au groupe d’applications de bureau :

1. Ouvrez une fenêtre PowerShell.

1. Exécutez la commande suivante pour vous connecter à l’environnement Windows Virtual Desktop :

   ```powershell
   Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
   ```

1. Ajoutez des utilisateurs au groupe d’applications de bureau en exécutant cette commande :

   ```powershell
   Add-RdsAppGroupUser <tenantname> <hostpoolname> "Desktop Application Group" -UserPrincipalName <userupn>
   ```

   L’UPN (nom d’utilisateur principal) de l’utilisateur doit correspondre à l’identité de l’utilisateur dans Azure AD, par exemple, *user1@contoso.com* . Si vous voulez ajouter plusieurs utilisateurs, exécutez la commande pour chacun d’eux.

Les utilisateurs que vous ajoutez au groupe d’applications de bureau peuvent se connecter à Windows Virtual Desktop avec les clients Bureau à distance pris en charge et voir une ressource pour le bureau d’une session.

Voici les clients actuellement pris en charge :

* [Client Bureau à distance pour Windows 7 et Windows 10](connect-windows-7-10-2019.md)
* [Client web Windows Virtual Desktop](connect-web-2019.md)

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un pool d’hôtes et affecté des utilisateurs pour qu’ils puissent accéder à son bureau. Vous pouvez remplir votre pool d’hôtes avec des programmes RemoteApp. Pour en savoir plus sur la façon de gérer des applications dans Windows Virtual Desktop, consultez ce tutoriel :

> [!div class="nextstepaction"]
> [Gérer les groupes d’applications](manage-app-groups-2019.md)
