---
title: Créer un pool d’hôtes Windows Virtual Desktop via PowerShell – Azure
description: Découvrez comment créer un pool d’hôtes dans Windows Virtual Desktop avec des cmdlets PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 10/02/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: a47126a48ea63efd4e49097428679b85b7a95a61
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91667161"
---
# <a name="create-a-windows-virtual-desktop-host-pool-with-powershell"></a>Créer un pool d’hôtes Windows Virtual Desktop avec PowerShell

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/create-host-pools-powershell-2019.md).

Les pools d'hôtes sont des ensembles d'une ou de plusieurs machines virtuelles identiques dans des environnements de locataires Windows Virtual Desktop. Chaque pool d’hôtes peut être associé à plusieurs groupes RemoteApp, à un groupe d’applications de bureau et à plusieurs hôtes de session.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà suivi les instructions indiquées dans [Configurer le module PowerShell](powershell-module.md).

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>Utilisez votre client PowerShell pour créer un pool d’hôtes

Exécutez l’applet de commande suivante pour vous connecter à l’environnement Windows Virtual Desktop :

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -HostPoolType <Pooled|Personal> -LoadBalancerType <BreadthFirst|DepthFirst|Persistent> -Location <region> -DesktopAppGroupName <appgroupname>
```

Cette applet de commande crée le pool d’hôtes, l’espace de travail et le groupe d’applications de bureau. En outre, elle inscrit le groupe d’applications de bureau dans l’espace de travail. Vous pouvez créer un espace de travail avec cette cmdlet ou utiliser un espace de travail existant.

Exécutez la cmdlet suivante pour créer un jeton d’inscription et permettre à un hôte de session de rejoindre le pool d'hôtes et de le sauvegarder dans un nouveau fichier sur votre ordinateur local. Vous pouvez spécifier la durée de validité du jeton d’inscription à l'aide du paramètre -ExpirationHours.

>[!NOTE]
>La date d’expiration du jeton ne peut pas être antérieure à une heure ni postérieure à un mois. Si vous définissez *-ExpirationTime* en dehors de cette plage, l’applet de commande ne crée pas le jeton.

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

Par exemple, si vous souhaitez créer un jeton qui expire dans deux heures, exécutez cette applet de commande :

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddHours(2).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

Exécutez la commande suivante pour ajouter des utilisateurs Azure Active Directory au groupe d’applications de bureau par défaut du pool d’hôtes.

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Exécutez la commande suivante pour ajouter des utilisateurs Azure Active Directory au groupe d’applications de bureau par défaut du pool d’hôtes :

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Exécutez la cmdlet suivante pour exporter le jeton d’inscription vers une variable, que vous utiliserez ultérieurement dans [Inscrire les machines virtuelles dans le pool d'hôtes Windows Virtual Desktop](#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).

```powershell
$token = Get-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname>
```

## <a name="create-virtual-machines-for-the-host-pool"></a>Créer les machines virtuelles pour le pool d'hôtes

Vous pouvez maintenant créer une machine virtuelle Azure à joindre à votre pool d’hôtes Windows Virtual Desktop.

Vous pouvez créer une machine virtuelle de plusieurs façons :

- [Créer une machine virtuelle à partir d’une image Azure Gallery](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [Créer une machine virtuelle à partir d’une image managée](../virtual-machines/windows/create-vm-generalized-managed.md)
- [Créer une machine virtuelle à partir d’une image non managée](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-user-image-data-disks)

>[!NOTE]
>Si vous déployez une machine virtuelle à l’aide de Windows 7 en tant que système d’exploitation hôte, le processus de création et de déploiement sera un peu différent. Pour plus d'informations, consultez [Déployer une machine virtuelle Windows 7 sur Windows Virtual Desktop](./virtual-desktop-fall-2019/deploy-windows-7-virtual-machine.md).

Une fois que vous avez créé vos machines virtuelles hôtes de session, [appliquez une licence Windows à une machine virtuelle hôte de session](./apply-windows-license.md#apply-a-windows-license-to-a-session-host-vm) pour exécuter vos machines virtuelles Windows ou Windows Server sans payer une autre licence.

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-agent-installations"></a>Préparer les machines virtuelles pour l'installation des agents Windows Virtual Desktop

Avant d'installer les agents Windows Virtual Desktop et d'inscrire les machines virtuelles dans votre pool d’hôtes Windows Virtual Desktop, les opérations suivantes sont requises :

- Vous devez joindre le domaine à la machine. Ainsi, le compte Azure Active Directory des utilisateurs Windows Virtual Desktop entrants est mappé à leur compte Active Directory pour leur permettre d'accéder à la machine virtuelle.
- Vous devez installer le rôle Hôte de session Bureau à distance (RDSH) si la machine virtuelle exécute un système d’exploitation Windows Server. Le rôle RDSH permet la bonne installation des agents Windows Virtual Desktop.

À des fins de jonction de domaine, procédez comme suit sur chaque machine virtuelle :

1. [Connectez-vous à la machine virtuelle](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) avec les informations d’identification que vous avez indiquées lors de la création de la machine virtuelle.
2. Sur la machine virtuelle, lancez le **Panneau de configuration** et sélectionnez **Système**.
3. Sélectionnez successivement **Nom de l’ordinateur**, **Modifier les paramètres** et **Modifier…**
4. Sélectionnez **Domaine**, puis entrez le domaine Active Directory sur le réseau virtuel.
5. Authentifiez-vous avec un compte de domaine qui dispose de privilèges d’accès sur les machines de jonction de domaine.

    >[!NOTE]
    > Si vous joignez vos machines virtuelles à un environnement Azure Active Directory Domain Services (Azure AD DS), vérifiez que votre utilisateur de jonction de domaine est également membre du [groupe Administrateurs AAD DC](../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group).

>[!IMPORTANT]
>Nous vous recommandons de ne pas activer de stratégies ou de configurations qui désactivant Windows Installer. Si vous désactivez Windows Installer, le service ne pourra pas installer les mises à jour de l'agent sur vos hôtes de session, et ces derniers ne fonctionneront pas correctement.

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool"></a>Inscrire les machines virtuelles dans le pool d'hôtes Windows Virtual Desktop

L'inscription des machines virtuelles dans un pool d’hôtes Windows Virtual Desktop est aussi simple que l'installation des agents Windows Virtual Desktop.

Pour inscrire les agents Windows Virtual Desktop, procédez comme suit sur chaque machine virtuelle :

1. [Connectez-vous à la machine virtuelle](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) avec les informations d’identification que vous avez indiquées lors de la création de la machine virtuelle.
2. Téléchargez et installez l'agent Windows Virtual Desktop.
   - Téléchargez l'[agent Windows Virtual Desktop](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv).
   - Exécutez le programme d’installation. Lorsque le programme d’installation vous demande le jeton d’inscription, entrez la valeur obtenue à partir de la cmdlet **Get-AzWvdRegistrationInfo**.
3. Téléchargez et installez le chargeur de démarrage de l'agent Windows Virtual Desktop.
   - Téléchargez le [chargeur de démarrage de l'agent Windows Virtual Desktop](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH).
   - Exécutez le programme d’installation.

>[!IMPORTANT]
>Pour contribuer à sécuriser votre environnement Windows Virtual Desktop dans Azure, nous vous recommandons de ne pas ouvrir le port entrant 3389 sur vos machines virtuelles. Windows Virtual Desktop ne nécessite pas l’ouverture du port entrant 3389 pour permettre aux utilisateurs d’accéder aux machines virtuelles du pool hôte. Si vous devez ouvrir le port 3389 pour résoudre des problèmes, nous vous recommandons d’utiliser un [accès à la machine virtuelle juste-à-temps](../security-center/security-center-just-in-time.md). Nous vous recommandons également de ne pas attribuer vos machines virtuelles à une adresse IP publique.

## <a name="update-the-agent"></a>Mettre à jour l’agent

Vous devrez mettre à jour l’agent si vous vous trouvez dans l’une des situations suivantes :

- Vous souhaitez migrer une session précédemment inscrite vers un nouveau pool d’hôtes.
- L’hôte de la session n’apparaît pas dans votre pool d’hôtes après une mise à jour.

Pour mettre à jour l’agent :

1. Connectez-vous à la machine virtuelle en tant qu’administrateur.
2. Accédez à **Services**, puis arrêtez les processus **Rdagent** et **Chargeur d’agent Bureau à distance**.
3. Ensuite, recherchez les MSI de l’agent et du chargeur de démarrage. Ils se trouvent soit dans le dossier **C:\DeployAgent**, soit dans l’emplacement où vous les avez enregistrés lors de l’installation.
4. Recherchez les fichiers suivants et désinstallez-les :
     
     - Microsoft.RDInfra.RDAgent.Installer-x64-verx.x.x
     - Microsoft.RDInfra.RDAgentBootLoader.Installer-x64

   Pour désinstaller ces fichiers, cliquez avec le bouton droit sur chaque nom de fichier, puis sélectionnez **Désinstaller**.
5. Si vous le souhaitez, vous pouvez également supprimer les paramètres de registre suivants :
     
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDInfraAgent
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDAgentBootLoader

6. Une fois que vous avez désinstallé ces éléments, vous devez supprimer toutes les associations avec l’ancien pool d’hôtes. Si vous souhaitez réinscrire cet hôte auprès du service, suivez les instructions figurant dans [Inscrire les machines virtuelles dans le pool d'hôtes Windows Virtual Desktop](create-host-pools-powershell.md#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).


## <a name="next-steps"></a>Étapes suivantes

Après avoir créé un pool d'hôtes, vous pouvez le remplir avec RemoteApps. Pour en savoir plus sur la façon de gérer des applications dans Windows Virtual Desktop, consultez le tutoriel Gérer les groupes d’applications.

> [!div class="nextstepaction"]
> [Gérer les groupes d’applications](./manage-app-groups.md)
