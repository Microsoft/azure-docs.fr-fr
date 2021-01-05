---
title: Se connecter à une machine virtuelle Windows dans Azure à l’aide d’Azure Active Directory (préversion)
description: Connexion Azure AD sur une machine virtuelle Azure exécutant Windows
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.custom: references_regions, devx-track-azurecli
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef2c9d7e2388d2da7a807fbf6b579360115a8323
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629797"
---
# <a name="sign-in-to-windows-virtual-machine-in-azure-using-azure-active-directory-authentication-preview"></a>Se connecter à une machine virtuelle Windows dans Azure via l’authentification Azure Active Directory (préversion)

Les organisations peuvent désormais utiliser l’authentification Azure Active Directory (AD) pour leurs machines virtuelles Azure exécutées sous **Windows Server 2019 Datacenter Edition** ou **Windows 10 1809** et versions ultérieures. L’utilisation d’Azure AD pour l’authentification sur des machines virtuelles vous offre un moyen de contrôler et d’appliquer des stratégies de manière centralisée. Des outils tels que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et l’accès conditionnel Azure AD vous permettent de contrôler qui peut accéder à une machine virtuelle. Cet article indique comment créer et configurer une machine virtuelle Windows Server 2019 pour utiliser l’authentification Azure AD.

> [!NOTE]
> La connexion Azure AD pour machines virtuelles Windows Azure est une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les avantages liés à l’utilisation de l’authentification Azure AD pour se connecter aux machines virtuelles Windows dans Azure sont nombreux, parmi lesquels :

- Utilisez les mêmes informations d’identification Azure AD fédérées ou managés que vous utilisez normalement.
- Vous n’avez plus besoin de gérer les comptes Administrateur locaux.
- Azure RBAC vous permet d’accorder l’accès approprié aux machines virtuelles en fonction des besoins et de le supprimer lorsque l’accès n’est plus nécessaire.
- Avant d’autoriser l’accès à une machine virtuelle, l’accès conditionnel Azure AD peut imposer des exigences supplémentaires telles que : 
   - Authentification multifacteur
   - Vérification du risque de connexion
- Automatisez et mettez à l’échelle la jointure Azure AD de machines virtuelles Microsoft Azure qui font partie de vos déploiements VDI.

> [!NOTE]
> Une fois cette fonctionnalité activée, vos machines virtuelles Windows dans Azure sont jointes à Azure AD. Vous ne pouvez pas les joindre à un autre domaine, par exemple sur AD ou Azure AD DS en local. Si vous devez le faire, vous devez déconnecter la machine virtuelle de votre locataire Azure AD en désinstallant l’extension.

## <a name="requirements"></a>Spécifications

### <a name="supported-azure-regions-and-windows-distributions"></a>Régions Azure et distributions Windows prises en charge

Les distributions Windows suivantes sont actuellement prises en charge pendant l’évaluation de cette fonctionnalité :

- Windows Server 2019 Datacenter
- Windows 10 1809 et versions ultérieures

> [!IMPORTANT]
> La connexion à distance aux machines virtuelles jointes à Azure AD n’est autorisée qu’à partir des PC Windows 10 qui sont inscrits sur Azure AD (à partir de Windows 10 20H1), joints à Azure AD ou joints à Azure AD par une jointure hybride Azure AD au **même** répertoire que la machine virtuelle. 

Les régions Azure suivantes sont actuellement prises en charge dans la préversion de cette fonctionnalité :

- Toutes les régions globales Azure

> [!IMPORTANT]
> Pour utiliser cette fonctionnalité d’évaluation, déployez uniquement une distribution Windows prise en charge dans une région Azure prise en charge. La fonctionnalité n’est actuellement pas prise en charge ni dans Azure Government ni dans les clouds souverains.

### <a name="network-requirements"></a>Configuration requise pour le réseau

Pour activer l’authentification Azure AD pour vos machines virtuelles Windows dans Azure, vous devez vous assurer que la configuration de votre réseau de machines virtuelles autorise un accès sortant aux points de terminaison suivants sur le port TCP 443 :

- https:\//enterpriseregistration.windows.net
- https:\//login.microsoftonline.com
- https:\//device.login.microsoftonline.com
- https:\//pas.windows.net

## <a name="enabling-azure-ad-login-in-for-windows-vm-in-azure"></a>Activation de la connexion Azure AD dans pour les machines virtuelles Windows dans Azure

Pour utiliser la connexion Azure AD pour une machine virtuelle Windows dans Azure, vous devez d’abord activer l’option de connexion Azure AD pour votre machine virtuelle Windows, puis vous devez configurer les attributions de rôle Azure pour les utilisateurs autorisés à se connecter à la machine virtuelle.
Vous pouvez activer la connexion Azure AD pour votre machine virtuelle Windows de plusieurs façons :

- À l’aide de l’expérience Portail Azure lors de la création d’une machine virtuelle Windows
- À l’aide de l’expérience Azure Cloud Shell lors de la création d’une machine virtuelle Windows **ou pour une machine virtuelle Windows existante**

### <a name="using-azure-portal-create-vm-experience-to-enable-azure-ad-login"></a>À l’aide du Portail Azure, créer une expérience de machine virtuelle pour activer la connexion Azure AD

Vous pouvez activer la connexion Azure AD pour les images de machine virtuelle Windows Server 2019 Datacenter ou Windows 10 1809 et versions ultérieures. 

Pour créer une machine virtuelle Windows Server 2019 Datacenter dans Azure avec l’ouverture de session Azure AD : 

1. Connectez-vous au [Portail Azure](https://portal.azure.com) à l’aide d’un compte disposant d’un accès pour créer des machines virtuelles, puis sélectionnez **+ Créer une ressource**.
1. Saisissez **Windows Serveur** dans la barre de recherche Rechercher dans la Place de marché.
   1. Cliquez sur **Windows Server** et choisissez **Windows Server 2019 Datacenter** dans la liste déroulante Sélectionner un plan logiciel.
   1. Cliquez sur **Créer**.
1. Sous l’onglet « Gestion », activez l’option permettant de **Se connecter avec les informations d’identification AAD (préversion)** sous la section Azure Active Directory en la passant à l’état **Activé**.
1. Assurez-vous que **Identité managée affectée par le système** sous la section Identité est définie sur **Activé**. Cette action doit se produire automatiquement une fois que vous avez activé la connexion avec les informations d’identification Azure AD.
1. Parcourez le reste de l’expérience de création d’une machine virtuelle. Dans cette préversion, vous devrez créer un nom d’utilisateur et un mot de passe Administrateur pour la machine virtuelle.

![Se connecter avec des informations d’identification Azure AD créer une machine virtuelle](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-login-with-azure-ad.png)

> [!NOTE]
> Pour vous connecter à la machine virtuelle à l’aide de vos informations d’identification Azure AD, vous devez d’abord configurer les attributions de rôles pour la machine virtuelle, comme décrit dans l’une des sections ci-dessous.

### <a name="using-the-azure-cloud-shell-experience-to-enable-azure-ad-login"></a>Utilisation de l’expérience Azure Cloud Shell pour activer la connexion Azure AD

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Cliquez simplement sur le bouton Copier pour copier le code, collez-le dans Cloud Shell, puis appuyez sur Entrée pour l’exécuter. Cloud Shell peut être ouvert de plusieurs façons :

Sélectionnez Essayer dans le coin supérieur droit d’un bloc de code.
Ouvrez Cloud Shell dans votre navigateur.
Sélectionnez le bouton Cloud Shell dans le menu en haut à droite du [Portail Azure](https://portal.azure.com).

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, cet article vous demande d’exécuter Azure CLI version 2.0.31 ou ultérieure. Pour déterminer la version, exécutez la commande az--version. Si vous devez l’installer ou la mettre à niveau, consultez l’article [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). 
1. Créez une machine virtuelle avec [az vm create](/cli/azure/vm#az-vm-create) à l’aide d’une distribution prise en charge dans une région prise en charge. 
1. Installez l’extension de machine virtuelle de connexion Azure AD. 

L’exemple suivant illustre le déploiement d’une machine virtuelle nommée myVM qui utilise Win2019Datacenter dans un groupe de ressources nommé myResourceGroup dans la région southcentralus. Dans les exemples suivants, vous pouvez indiquer votre propre groupe de ressources et les noms de vos machines virtuelles comme il vous convient.

```AzureCLI
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Win2019Datacenter \
    --assign-identity \
    --admin-username azureuser \
    --admin-password yourpassword
```

> [!NOTE]
> Vous devez activer l’option Identité managée affectée par le système sur votre machine virtuelle avant d’installer l’extension de machine virtuelle de connexion Azure AD.

La création de la machine virtuelle et des ressources de support ne nécessite que quelques minutes.

Enfin, installez l’extension de machine virtuelle de connexion Azure AD pour activer la connexion Azure AD pour la machine virtuelle Windows. Les extensions de machine virtuelle sont de petites applications permettant d’exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. Utilisez l’ensemble [az vm extension](/cli/azure/vm/extension#az-vm-extension-set) pour installer l’extension AADLoginForWindows sur la machine virtuelle nommée myVM dans le groupe de ressources myResourceGroup :

> [!NOTE]
> Vous pouvez installer l’extension AADLoginForWindows sur une machine virtuelle existante Windows Server 2019 ou Windows 10 1809 et versions ultérieures existante l’activer pour l’authentification Azure AD. Vous trouverez ci-dessous un exemple d’interface CLI AZ.

```AzureCLI
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADLoginForWindows \
    --resource-group myResourceGroup \
    --vm-name myVM
```

Le `provisioningState` de `Succeeded` s’affiche, une fois que l’extension est installée sur la machine virtuelle.

## <a name="configure-role-assignments-for-the-vm"></a>Configurer des attributions de rôle pour la machine virtuelle

Maintenant que vous avez créé la machine virtuelle, vous devez configurer la stratégie RBAC Azure pour déterminer qui peut se connecter à la machine virtuelle. Deux rôles Azure sont utilisés pour autoriser la connexion aux machines virtuelles :

- **Connexion de l’administrateur aux machines virtuelles** : les utilisateurs auxquels ce rôle est attribué peuvent se connecter à une machine virtuelle Azure avec des privilèges Administrateur.
- **Connexion de l’utilisateur aux machines virtuelles** : les utilisateurs auxquels ce rôle est attribué peuvent se connecter à une machine virtuelle Azure avec des privilèges d’utilisateur standard.

> [!NOTE]
> Pour autoriser un utilisateur à se connecter à la machine virtuelle via le protocole RDP (Remote Desktop Protocol), vous devez attribuer le rôle Connexion de l’administrateur aux machines virtuelles ou Connexion de l’utilisateur aux machines virtuelles. Un utilisateur Azure auquel le rôle Propriétaire ou Contributeur est attribué pour une machine virtuelle ne possède pas automatiquement les privilèges pour se connecter à la machine virtuelle via RDP. Cela permet de fournir une séparation auditée entre l’ensemble des personnes qui contrôlent les machines virtuelles et l’ensemble des personnes qui peuvent accéder aux machines virtuelles.

Il existe plusieurs façons de configurer des attributions de rôles pour une machine virtuelle :

- À l’aide de l’expérience du portail Azure AD
- À l’aide de l’expérience Azure Cloud Shell

> [!NOTE]
> Les rôles Connexion de l’administrateur aux machines virtuelles et Connexion de l’utilisateur aux machines virtuelles utilisent dataActions et ne peuvent donc pas être attribués au niveau du groupe d’administration. Actuellement, ces rôles peuvent être attribués uniquement au niveau de l’abonnement, du groupe de ressources ou de la ressource.

### <a name="using-azure-ad-portal-experience"></a>À l’aide de l’expérience du portail Azure AD

Pour configurer les attributions de rôles pour vos machines virtuelles Windows Server 2019 Datacenter activées pour Azure AD :

1. Accédez à la page de vue d’ensemble de la machine virtuelle spécifique
1. Sélectionnez **Contrôle d’accès (IAM)** à partir des options de menu
1. Sélectionnez **Ajouter**, **Ajouter une attribution de rôle** pour ouvrir le volet Ajouter une attribution de rôle.
1. Dans la liste déroulante **Rôle**, sélectionnez un rôle, par exemple **Connexion de l’administrateur aux machines virtuelles** ou **Connexion de l’utilisateur aux machines virtuelles**.
1. Dans le champ **Sélectionner**, sélectionnez un utilisateur, un groupe, un principal de service ou une identité managée. Si vous ne voyez pas le principal de sécurité dans la liste, vous pouvez saisir du texte dans la zone **Sélectionner** pour rechercher des noms d’affichage, des adresses de messagerie et des identificateurs d’objet dans le répertoire.
1. Sélectionnez **Enregistrer** pour attribuer le rôle.

Après quelques instants, le principal de sécurité est attribué au rôle dans l’étendue sélectionnée.

![Attribuer des rôles aux utilisateurs qui accèdent à la machine virtuelle](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-access-control-assign-role.png)

### <a name="using-the-azure-cloud-shell-experience"></a>À l’aide de l’expérience Azure Cloud Shell

L’exemple suivant illustre l’utilisation de la commande [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) pour attribuer le rôle Connexion de l’administrateur aux machines virtuelles à la machine virtuelle de votre utilisateur Azure actuel. Le nom d’utilisateur de votre compte Azure actif est obtenu à l’aide de la commande [az account show](/cli/azure/account#az-account-show), et l’étendue est définie sur la machine virtuelle créée lors d’une étape précédente avec [az vm show](/cli/azure/vm#az-vm-show). L’étendue peut également être attribuée au niveau d’un groupe de ressources ou d’un abonnement, et les autorisations d’héritage Azure RBAC normales s’appliquent. Pour plus d’informations, consultez [Se connecter à une machine virtuelle Linux dans Azure via l’authentification Azure Active Directory](../../virtual-machines/linux/login-using-aad.md).

```   AzureCLI
$username=$(az account show --query user.name --output tsv)
$vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> Si votre domaine AAD et le domaine du nom d’utilisateur d’ouverture de session ne correspondent pas, vous devez spécifier l’ID d’objet de votre compte d’utilisateur avec `--assignee-object-id`, pas seulement le nom d’utilisateur pour `--assignee`. Vous pouvez obtenir l’ID d’objet de votre compte d’utilisateur avec [az ad user list](/cli/azure/ad/user#az-ad-user-list).

Pour plus d’informations sur l’utilisation du contrôle d’accès en fonction du rôle (RBAC) d’Azure pour gérer l’accès aux ressources de votre abonnement Azure, consultez les articles suivants :

- [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../../role-based-access-control/role-assignments-cli.md)
- [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](../../role-based-access-control/role-assignments-portal.md)
- [Ajoutez ou supprimez des attributions de rôles Azure à l’aide d’Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="using-conditional-access"></a>Utilisation d’un accès conditionnel

Vous pouvez appliquer des stratégies d’accès conditionnel, telles qu’une authentification multifacteur ou une vérification du risque de connexion utilisateur, avant d’autoriser l’accès à des machines virtuelles Windows dans Azure qui sont activées avec une connexion à Azure AD. Pour appliquer une stratégie d’accès conditionnel, vous devez sélectionner l’application « Connexion à une machine virtuelle Microsoft Azure » à partir de l’option d’affectation d’applications ou actions cloud, puis utiliser le risque de connexion comme condition et/ou exiger une authentification multifacteur comme contrôle pour l’octroi d’accès. 

> [!NOTE]
> Si vous utilisez « Exiger l’authentification multifacteur » comme contrôle pour l’octroi d’accès pour demander l’accès à l’application « Connexion à une machine virtuelle Microsoft Azure », vous devez fournir une revendication d’authentification multifacteur avec le client qui lance la session Bureau à distance (RDP) sur la machine virtuelle Windows cible dans Azure. La seule façon d’y parvenir sur un client Windows 10 est d’utiliser le code PIN Windows Hello Entreprise ou une authentification biométrique avec le client RDP. La prise en charge de l’authentification biométrique a été ajoutée au client RDP dans Windows 10 version 1809. Le Bureau à distance utilisant l’authentification Windows Hello Entreprise est disponible uniquement pour les déploiements qui utilisent le modèle approuvé de certificat et qui ne sont actuellement pas disponibles pour le modèle approuvé de clé.

> [!WARNING]
> Le service Azure AD Multi-Factor Authentication activé/appliqué par utilisateur n’est pas pris en charge pour la connexion à une machine virtuelle.

## <a name="log-in-using-azure-ad-credentials-to-a-windows-vm"></a>Se connecter à l’aide des informations d’identification Azure AD sur une machine virtuelle Windows

> [!IMPORTANT]
> La connexion à distance aux machines virtuelles jointes à Azure AD n’est autorisée qu’à partir des PC Windows 10 qui sont soit inscrits sur Azure AD (build minimale 20H1 requise), soit joints à Azure AD ou joints à Azure AD par une jointure hybride Azure AD au **même** annuaire que la machine virtuelle. En outre, pour se connecter via RDP à l’aide d’informations d’identification Azure AD, l’utilisateur doit appartenir à l’un des deux rôles Azure, Connexion de l’administrateur aux machines virtuelles ou Connexion de l’utilisateur aux machines virtuelles. Si vous utilisez un PC Windows 10 inscrit dans Azure AD, vous devez entrer les informations d’identification au format AzureAD\UPN (par exemple, AzureAD\john@contoso.com). À ce stade, vous ne pouvez pas utiliser le service Azure Bastion pour vous connecter à l’aide de l’authentification Azure Active Directory avec l’extension AADLoginForWindows. Seul le protocole RDP direct est pris en charge.

Pour vous connecter à votre machine virtuelle Windows Server 2019 à l’aide d’Azure AD : 

1. Accédez à la page de vue d’ensemble de la machine virtuelle qui a été activée avec l’ouverture de session Azure AD.
1. Sélectionnez **Se connecter** pour ouvrir le panneau Se connecter à la machine virtuelle.
1. Sélectionnez **Télécharger le fichier RDP**.
1. Sélectionnez **Ouvrir** pour lancer le client Connexion Bureau à distance.
1. Sélectionnez **Se connecter** pour lancer la boîte de dialogue d’ouverture de session Windows.
1. Ouvrez une session à l’aide de vos informations d’identification Azure AD.

Vous êtes à présent connecté à la machine virtuelle Azure Windows Server 2019 avec les autorisations de rôle attribuées, par exemple Utilisateur de machine virtuelle ou Administrateur de machine virtuelle. 

> [!NOTE]
> Vous pouvez enregistrer le fichier .RDP localement sur votre ordinateur pour lancer les futures connexions de bureau à distance sur votre machine virtuelle au lieu d’accéder à la page de vue d’ensemble de la machine virtuelle dans le Portail Azure et à l’aide de l’option de connexion.

## <a name="troubleshoot"></a>Dépanner

### <a name="troubleshoot-deployment-issues"></a>Résoudre les problèmes de déploiement

L’extension AADLoginForWindows doit être installée correctement pour que la machine virtuelle termine le processus de jonction Azure AD. Procédez comme suit si l’extension de machine virtuelle ne s’installe pas correctement.

1. Connectez-vous via RDP à la machine virtuelle à l’aide du compte Administrateur local et examinez le fichier CommandExecution.log sous  
   
   C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.ActiveDirectory.AADLoginForWindows\0.3.1.0. 

   > [!NOTE]
   > Si l’extension redémarre après l’échec initial, le journal contenant l’erreur de déploiement est enregistré sous CommandExecution_YYYYMMDDHHMMSSSSS.log. "
1. Ouvrez une invite de commandes PowerShell sur la machine virtuelle et vérifiez que ces requêtes par rapport au point de terminaison Instance Metadata Service (IMDS) en cours d’exécution sur l’hôte Azure sont renvoyées :

   | Commande à exécuter | Sortie attendue |
   | --- | --- |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/instance?api-version=2017-08-01"` | Informations correctes sur la machine virtuelle Azure |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/info?api-version=2018-02-01"` | ID de locataire valide associé à l’abonnement Azure |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/oauth2/token?resource=urn:ms-drs:enterpriseregistration.windows.net&api-version=2018-02-01"` | Jeton d’accès valide émis par Azure Active Directory pour l’identité managée qui est attribuée à cette machine virtuelle |

   > [!NOTE]
   > Le jeton d’accès peut être décodé à l’aide d’un outil tel que [http://calebb.net/](http://calebb.net/). Vérifiez que l’« appid » dans le jeton d’accès correspond à l’identité managée attribuée à la machine virtuelle.

1. Vérifiez que les points de terminaison requis sont accessibles à partir de la machine virtuelle à l’aide de la ligne de commande :
   
   - curl https:\//login.microsoftonline.com/ -D –
   - curl https:\//login.microsoftonline.com/`<TenantID>`/ -D –

   > [!NOTE]
   > Remplacez `<TenantID>` par l’ID de locataire Azure AD associé à l’abonnement Azure.

   - curl https:\//enterpriseregistration.windows.net/ -D -
   - curl https:\//device.login.microsoftonline.com/ -D -
   - curl https:\//pas.windows.net/ -D -

1. L’état de l’appareil peut être affiché en exécutant `dsregcmd /status`. L’objectif est que l’état de l’appareil s’affiche comme `AzureAdJoined : YES`.

   > [!NOTE]
   > L’activité de jointure Azure AD est capturée dans la visionneuse d’événements sous le journal Registration\Admin de l’appareil de l’utilisateur.

Si l’extension AADLoginForWindows échoue avec un code d’erreur spécifique, vous pouvez effectuer les étapes suivantes :

#### <a name="issue-1-aadloginforwindows-extension-fails-to-install-with-terminal-error-code-1007-and-exit-code--2145648574"></a>Problème 1 : Échec de l’installation de l’extension AADLoginForWindows avec le code d’erreur de terminal « 1007 » et le code de sortie :-2145648574.

Ce code de sortie se traduit par DSREG_E_MSI_TENANTID_UNAVAILABLE, car l’extension est incapable d’interroger les informations du locataire Azure AD.

1. Vérifiez que la machine virtuelle Azure peut récupérer le TenantID à partir d’Instance Metadata Service.

   - Connectez-vous via RDP à la machine virtuelle en tant qu’administrateur local et vérifiez que le point de terminaison retourne un ID de locataire valide en exécutant cette commande à partir d’une ligne de commande élevée sur la machine virtuelle :
      
      - curl -H Metadata:true http://169.254.169.254/metadata/identity/info?api-version=2018-02-01

1. L’administrateur de machines virtuelles tente d’installer l’extension AADLoginForWindows, mais une identité managée affectée par le système n’a pas d’abord activé la machine virtuelle. Accédez au panneau Identité de la machine virtuelle. Dans l’onglet Affectée par le système, vérifiez qu’État est activé.

#### <a name="issue-2-aadloginforwindows-extension-fails-to-install-with-exit-code--2145648607"></a>Problème2 : Échec de l’installation de l’extension AADLoginForWindows avec le code de sortie :-2145648607.

Ce code de sortie se traduit par DSREG_AUTOJOIN_DISC_FAILED, car l’extension n’est pas en mesure d’atteindre le point de terminaison `https://enterpriseregistration.windows.net`.

1. Vérifiez que les points de terminaison requis sont accessibles à partir de la machine virtuelle à l’aide de la ligne de commande :

   - curl https:\//login.microsoftonline.com/ -D –
   - curl https:\//login.microsoftonline.com/`<TenantID>`/ -D –
   
   > [!NOTE]
   > Remplacez `<TenantID>` par l’ID de locataire Azure AD associé à l’abonnement Azure. Si vous devez trouver l’ID de locataire, vous pouvez pointer sur le nom de votre compte pour obtenir l’ID de locataire/répertoire, ou sélectionnez Azure Active Directory > Propriétés > ID de répertoire dans le Portail Azure.

   - curl https:\//enterpriseregistration.windows.net/ -D -
   - curl https:\//device.login.microsoftonline.com/ -D -
   - curl https:\//pas.windows.net/ -D -

1. Si l’une des commandes échoue avec le message « Impossible de résoudre `<URL>` hôte », essayez d’exécuter cette commande pour déterminer le serveur DNS utilisé par la machine virtuelle.
   
   `nslookup <URL>`

   > [!NOTE] 
   > Remplacez `<URL>` par les noms de domaine complets utilisés par les points de terminaison, tels que « login.microsoftonline.com ».

1. Ensuite, vérifiez si la spécification d’un serveur DNS public permet à la commande de fonctionner correctement :

   `nslookup <URL> 208.67.222.222`

1. Si nécessaire, modifiez le serveur DNS affecté au groupe de sécurité réseau auquel la machine virtuelle Azure appartient.

#### <a name="issue-3-aadloginforwindows-extension-fails-to-install-with-exit-code-51"></a>Problème 3 : Échec de l’installation de l’extension AADLoginForWindows avec le code de sortie : 51

Le code de sortie 51 se traduit par « Cette extension n’est pas prise en charge sur le système d’exploitation de la machine virtuelle ».

En Préversion publique, l’extension AADLoginForWindows est uniquement destinée à être installée sur Windows Server 2019 ou Windows 10 (version 1809 ou ultérieure). Vérifiez que la version de Windows est prise en charge. Si la version de Windows n’est pas prise en charge, désinstallez l’extension de machine virtuelle.

### <a name="troubleshoot-sign-in-issues"></a>Résoudre les problèmes de connexion

Certaines erreurs courantes se produisent lorsque vous essayez de vous connecter via RDP avec des informations d’identification Azure AD, notamment l’absence d’attribution de rôles Azure, un client non autorisé ou la méthode de connexion 2FA requise. Utilisez les informations suivantes pour corriger ces problèmes.

L’état de l’appareil et de la SSO peut être affiché en exécutant `dsregcmd /status`. L’objectif est que l’état de l’appareil s’affiche comme `AzureAdJoined : YES` et `SSO State` comme `AzureAdPrt : YES`.

En outre, la connexion RDP à l’aide de comptes Azure AD est capturée dans la visionneuse d’événements sous les journaux d’événements AAD\Operational.

#### <a name="azure-role-not-assigned"></a>rôle Azure non attribué

Si le message d’erreur suivant s’affiche lorsque vous établissez une connexion Bureau à distance à votre machine virtuelle : 

- Votre compte est configuré de façon à vous empêcher d’utiliser cet appareil. Pour plus d’informations, contactez votre administrateur système.

![Votre compte est configuré de façon à vous empêcher d’utiliser cet appareil.](./media/howto-vm-sign-in-azure-ad-windows/rbac-role-not-assigned.png)

Vérifiez que vous disposez de [stratégies Azure RBAC configurées](../../virtual-machines/linux/login-using-aad.md) pour la machine virtuelle qui accorde à l’utilisateur le rôle Connexion de l’administrateur aux machines virtuelles ou Connexion de l’utilisateur aux machines virtuelles :

> [!NOTE]
> Si vous rencontrez des problèmes avec des attributions de rôles Azure, consultez [Résoudre les problèmes liés à Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/troubleshooting#azure-role-assignments-limit).
 
#### <a name="unauthorized-client"></a>Client non autorisé

Si le message d’erreur suivant s’affiche lorsque vous établissez une connexion Bureau à distance à votre machine virtuelle : 

- Vos informations d’identification n’ont pas fonctionné

![Vos informations d’identification n’ont pas fonctionné](./media/howto-vm-sign-in-azure-ad-windows/your-credentials-did-not-work.png)

Vérifiez que le PC Windows 10 que vous utilisez pour établir la connexion Bureau à distance est un ordinateur qui est soit joint à Azure AD, soit hybride joint à Azure AD au même répertoire Azure AD auquel votre machine virtuelle est jointe. Pour plus d’informations sur l’identité d’appareil, consultez l’article [Présentation de l’identité d’appareil](./overview.md).

> [!NOTE]
> La build 20H1 de Windows 10 a ajouté la prise en charge d’un PC inscrit dans Azure AD pour initier une connexion RDP à votre machine virtuelle. Lorsque vous utilisez un PC inscrit dans Azure AD (pas joint à Azure AD ou joint à Azure AD par une jointure hybride) comme client RDP pour initier des connexions à votre machine virtuelle, vous devez entrer des informations d’identification au format AzureAD\UPn (par exemple, AzureAD\john@contoso.com).

Vérifiez également que l’extension AADLoginForWindows n’a pas été désinstallée une fois la jointure Azure AD terminée.
 
#### <a name="mfa-sign-in-method-required"></a>Méthode de connexion MFA requise

Si le message d’erreur suivant s’affiche lorsque vous établissez une connexion Bureau à distance à votre machine virtuelle : 

- La méthode de connexion que vous essayez d’utiliser n’est pas autorisée. Essayez une autre méthode de connexion ou contactez votre administrateur système.

![La méthode de connexion que vous essayez d’utiliser n’est pas autorisée.](./media/howto-vm-sign-in-azure-ad-windows/mfa-sign-in-method-required.png)

Si vous avez configuré une stratégie d’accès conditionnel qui requiert un authentification multifacteur (MFA) pour vous permettre d’accéder à la ressource, vous devez vous assurer que le PC Windows 10 qui établit la connexion Bureau à distance à votre machine virtuelle se connecte à l’aide d’une méthode d’authentification forte, telle que Windows Hello. Si vous n’utilisez pas une méthode d’authentification forte pour votre connexion Bureau à distance, l’erreur précédente s’affiche.

Si vous n’avez pas déployé Windows Hello Entreprise et si ce n’est pas possible pour l’instant, vous pouvez exclure l’exigence d’authentification multifacteur en configurant la stratégie d’accès conditionnel qui exclut l’application « Connexion à une machine virtuelle Microsoft Azure » de la liste des applications cloud qui exigent une authentification MFA. Pour en savoir plus sur Windows Hello Entreprise, consultez [vue d’ensemble de Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification).

> [!NOTE]
> L’authentification par code confidentiel Windows Hello Entreprise avec le Bureau à distance a été prise en charge par Windows 10 pour plusieurs versions, mais la prise en charge de l’authentification biométrique avec le Bureau à distance a été ajoutée dans Windows 10 version 1809. L’authentification Windows Hello Entreprise lors de l’utilisation du Bureau à distance n’est disponible que pour les déploiements qui utilisent le modèle approuvé de certificat et qui ne sont actuellement pas disponibles pour le modèle approuvé de clé.
 
## <a name="preview-feedback"></a>Commentaires de la préversion

Partagez vos commentaires sur cette fonctionnalité d’évaluation ou signalez des problèmes lors de son utilisation sur le [forum de commentaires d’Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Active Directory, consultez [Qu’est-ce qu’Azure Active Directory ?](../fundamentals/active-directory-whatis.md)
