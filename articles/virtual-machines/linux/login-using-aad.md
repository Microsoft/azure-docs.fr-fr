---
title: Se connecter à une machine virtuelle Linux avec les informations d’identification d’Azure Active Directory
description: Découvrez comment créer et configurer une machine virtuelle Linux pour utiliser l’authentification Azure Active Directory.
author: SanDeo-MSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 11/17/2020
ms.author: sandeo
ms.openlocfilehash: b4fc6b9facc79db109c5ce5be09576b16a2abdc7
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510887"
---
# <a name="preview-log-in-to-a-linux-virtual-machine-in-azure-using-azure-active-directory-authentication"></a>Aperçu : Se connecter à une machine virtuelle Linux dans Azure via l’authentification Azure Active Directory

Pour améliorer la sécurité des machines virtuelles Linux (VM) dans Azure, vous pouvez intégrer l’authentification Azure Active Directory (AD). Lorsque vous utilisez l’authentification Azure AD pour les machines virtuelles Linux, vous contrôlez et appliquez de façon centralisée des stratégies qui autorisent ou refusent l’accès aux machines virtuelles. Cet article indique comment créer et configurer une machine virtuelle Linux pour utiliser l’authentification Azure AD.


> [!IMPORTANT]
> L’authentification Azure Active Directory est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
> Utilisez cette fonctionnalité sur une machine virtuelle de test que vous prévoyez d’abandonner après le test.
>


Les avantages liés à l’utilisation de l’authentification Azure AD pour se connecter aux machines virtuelles Linux dans Azure sont nombreux, parmi lesquels :

- **Une sécurité améliorée :**
  - Vous pouvez utiliser vos informations d’identification Active Directory d’entreprise pour vous connecter aux machines virtuelles Linux dans Azure. Il est inutile de créer des comptes d’administrateur local et de gérer la durée de vie des informations d’identification.
  - En réduisant la dépendance aux comptes d’administrateur local, vous n’avez pas à vous soucier de la perte/du vol des informations d’identification, des utilisateurs qui configurent des informations d’identification faibles, etc.
  - Les stratégies portant sur la complexité et la durée de vie du mot de passe qui sont configurées pour votre annuaire Azure AD contribuent à sécuriser également les machines virtuelles Linux.
  - Pour sécuriser davantage la connexion aux machines virtuelles Azure, vous pouvez configurer l’authentification multifacteur.
  - La possibilité de se connecter à des machines virtuelles Linux avec Azure Active Directory fonctionne également pour les clients qui utilisent [Federation Services](../../active-directory/hybrid/how-to-connect-fed-whatis.md).

- **Une collaboration parfaite :** le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de spécifier qui peut se connecter à une machine virtuelle donnée en tant qu’utilisateur standard ou avec des privilèges d’administrateur. Lorsque des utilisateurs rejoignent ou quittent votre équipe, vous pouvez mettre à jour la stratégie Azure RBAC pour la machine virtuelle pour accorder les accès appropriés. Cette expérience est beaucoup plus simple que d’avoir à nettoyer les machines virtuelles pour supprimer les clés publiques SSH inutiles. Lorsque des employés quittent votre organisation et que leur compte d’utilisateur est désactivé ou supprimé d’Azure AD, ils n’ont plus accès à vos ressources.

## <a name="supported-azure-regions-and-linux-distributions"></a>Régions Azure et distributions Linux prises en charge

Les distributions Linux suivantes sont actuellement prises en charge dans la préversion de cette fonctionnalité :

| Distribution | Version |
| --- | --- |
| CentOS | CentOS 6, CentOS 7 |
| Debian | Debian 9 |
| OpenSUSE | openSUSE Leap 42.3 |
| RedHat Enterprise Linux | RHEL 6, RHEL 7 | 
| SUSE Linux Enterprise Server | SLES 12 |
| Serveur Ubuntu | Ubuntu 14.04 LTS, Ubuntu Server 16.04 et Ubuntu Server 18.04 |


Les régions Azure suivantes sont actuellement prises en charge dans la préversion de cette fonctionnalité :

- Toutes les régions Azure globales

>[!IMPORTANT]
> Pour utiliser cette fonctionnalité en préversion, déployez uniquement une distribution Linux prise en charge dans une région Azure prise en charge. La fonctionnalité n’est prise en charge ni dans Azure Government ni dans les clouds souverains.
>
> L’utilisation de cette extension n’est pas prise en charge sur les clusters Azure Kubernetes Service (AKS). Pour plus d’informations, consultez [Stratégies de support pour AKS](../../aks/support-policies.md).


Si vous choisissez d’installer et d’utiliser l’interface CLI localement, vous devez exécuter Azure CLI version 2.0.31 ou une version ultérieure pour poursuivre la procédure décrite dans ce didacticiel. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="network-requirements"></a>Configuration requise pour le réseau

Pour activer l’authentification Azure AD pour vos machines virtuelles Linux dans Azure, vous devez vérifier que la configuration de votre réseau de machines virtuelles autorise un accès sortant aux points de terminaison suivants sur le port TCP 443 :

* https:\//login.microsoftonline.com
* https:\//login.windows.net
* https:\//device.login.microsoftonline.com
* https:\//pas.windows.net
* https:\//management.azure.com
* https:\//packages.microsoft.com

> [!NOTE]
> Actuellement, les groupes de sécurité réseau Azure ne peuvent pas être configurés pour les machines virtuelles activées avec l’authentification Azure AD.

## <a name="create-a-linux-virtual-machine"></a>Créer une machine virtuelle Linux

Créez un groupe de ressources avec [az group create](/cli/azure/group#az-group-create), puis une machine virtuelle avec [az vm create](/cli/azure/vm#az-vm-create) à l’aide d’une distribution prise en charge dans une région qui l’est également. L’exemple suivant illustre le déploiement d’une machine virtuelle nommée *myVM* qui utilise *Ubuntu 16.04 LTS* dans un groupe de ressources nommé *myResourceGroup* dans la région *southcentralus*. Dans les exemples suivants, vous pouvez indiquer votre propre groupe de ressources et les noms de vos machines virtuelles comme il vous convient.

```azurecli-interactive
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

La création de la machine virtuelle et des ressources de support ne nécessite que quelques minutes.

## <a name="install-the-azure-ad-login-vm-extension"></a>Installer le journal Azure AD dans une extension de machine virtuelle

> [!NOTE]
> Si vous déployez cette extension sur une machine virtuelle précédemment créée, assurez-vous que cette dernière dispose d’au moins 1 Go de mémoire allouée, faute de quoi l’installation de l’extension échouera

Pour vous connecter à une machine virtuelle Linux avec des informations d’identification Azure AD, installez le journal Azure Active Directory dans l’extension de machine virtuelle. Les extensions de machine virtuelle sont de petites applications permettant d’exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. Utilisez la commande [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set) pour installer l’extension *AADLoginForLinux* sur la machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup* :

```azurecli-interactive
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory.LinuxSSH \
    --name AADLoginForLinux \
    --resource-group myResourceGroup \
    --vm-name myVM
```

Le paramètre *provisioningState* défini sur *Succeeded* s’affiche une fois que l’extension est installée sur la machine virtuelle. La machine virtuelle a besoin d’un agent de machine virtuelle en cours d’exécution pour installer l’extension. Pour plus d’informations, consultez [Vue d'ensemble de l'agent de machine virtuelle](../extensions/agent-windows.md).

## <a name="configure-role-assignments-for-the-vm"></a>Configurer des attributions de rôle pour la machine virtuelle

La stratégie de contrôle d’accès en fonction du rôle Azure (Azure RBAC) détermine qui peut se connecter à la machine virtuelle. Deux rôles Azure sont utilisés pour autoriser la connexion aux machines virtuelles :

- **Connexion de l’administrateur aux machines virtuelles** : les utilisateurs auxquels ce rôle est attribué peuvent se connecter à une machine virtuelle Azure avec des privilèges d’administrateur Windows ou d’utilisateur racine Linux.
- **Connexion de l’utilisateur aux machines virtuelles** : les utilisateurs auxquels ce rôle est attribué peuvent se connecter à une machine virtuelle Azure avec des privilèges d’utilisateur standard.

> [!NOTE]
> Pour autoriser un utilisateur à se connecter à la machine virtuelle via le protocole SSH, vous devez attribuer le rôle *Connexion de l’administrateur aux machines virtuelles* ou *Connexion de l’utilisateur aux machines virtuelles*. Les rôles Connexion de l’administrateur aux machines virtuelles et Connexion de l’utilisateur aux machines virtuelles utilisant dataActions, ils ne peuvent pas être attribués à l’étendue du groupe d’administration. Actuellement, ces rôles peuvent être attribués uniquement à l’étendue de l’abonnement, du groupe de ressources ou de la ressource. Un utilisateur Azure auquel le rôle *Propriétaire* ou *Contributeur* est attribué pour une machine virtuelle ne possède pas automatiquement les privilèges pour se connecter à la machine virtuelle via le protocole SSH. 

L’exemple suivant illustre l’utilisation de la commande [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) pour attribuer le rôle *Connexion de l’administrateur aux machines virtuelles* à la machine virtuelle de votre utilisateur Azure actuel. Le nom d’utilisateur de votre compte Azure actif est obtenu à l’aide de la commande [az account show](/cli/azure/account#az-account-show), et *l’étendue* est définie sur la machine virtuelle créée dans une étape précédente avec [az vm show](/cli/azure/vm#az-vm-show). L’étendue peut également être attribuée au niveau d’un groupe de ressources ou d’un abonnement, et les autorisations d’héritage Azure RBAC normales s’appliquent. Pour plus d’informations, consultez [Azure RBAC](../../role-based-access-control/overview.md).

```azurecli-interactive
username=$(az account show --query user.name --output tsv)
vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> Si votre domaine AAD et le domaine du nom d’utilisateur d’ouverture de session ne correspondent pas, vous devez spécifier l’ID d’objet de votre compte d’utilisateur avec *--assignee-object-id*, pas seulement le nom d’utilisateur pour *--assignee*. Vous pouvez obtenir l’ID d’objet de votre compte d’utilisateur avec [az ad user list](/cli/azure/ad/user#az-ad-user-list).

Pour plus d’informations sur l’utilisation du contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer l’accès aux ressources de votre abonnement Azure, consultez les rubriques relatives à l’utilisation d’[Azure CLI](../../role-based-access-control/role-assignments-cli.md), du [portail Azure](../../role-based-access-control/role-assignments-portal.md) ou d’[Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="using-conditional-access"></a>Utilisation d’un accès conditionnel

Vous pouvez appliquer des stratégies d’accès conditionnel, telles qu’une authentification multifacteur ou une vérification du risque de connexion utilisateur, avant d’autoriser l’accès à des machines virtuelles Linux dans Azure qui sont activées avec une connexion à Azure AD. Pour appliquer une stratégie d’accès conditionnel, vous devez sélectionner l’application « Connexion à une machine virtuelle Linux Azure » à partir de l’option d’affectation d’applications ou d’actions cloud, puis utiliser le risque de connexion comme condition et/ou exiger une authentification multifacteur comme contrôle d’octroi d’accès. 

> [!WARNING]
> Le service Azure AD Multi-Factor Authentication activé/appliqué par utilisateur n’est pas pris en charge pour la connexion à une machine virtuelle.

## <a name="log-in-to-the-linux-virtual-machine"></a>Se connecter à la machine virtuelle Linux

Commencez par afficher l’adresse IP publique de votre machine virtuelle à l’aide de la commande [az vm show](/cli/azure/vm#az-vm-show) :

```azurecli-interactive
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

Connectez-vous à la machine virtuelle Azure Linux à l’aide de vos informations d’identification Azure AD. Le paramètre `-l` vous permet de spécifier votre propre adresse de compte Azure AD. Remplacez l’exemple de compte par le vôtre. Les adresses de comptes doivent être entrées en minuscules. Remplacez l’exemple d’adresse IP par l’adresse IP publique de votre machine virtuelle de la commande précédente.

```console
ssh -l azureuser@contoso.onmicrosoft.com 10.11.123.456
```

Vous êtes invité à vous connecter à Azure AD avec un code à usage unique à [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin). Copiez et collez le code à usage unique dans la page de connexion de l’appareil.

Lorsque vous y êtes invité, entrez vos informations d’identification de connexion à Azure AD dans la page de connexion. 

Le message suivant s’affiche dans le navigateur web une fois que vous êtes authentifié : `You have signed in to the Microsoft Azure Linux Virtual Machine Sign-In application on your device.`

Fermez la fenêtre du navigateur, revenez à l’invite SSH, puis appuyez sur la touche **Entrée**. 

Vous êtes à présent connecté à la machine virtuelle Azure Linux avec les autorisations de rôle attribuées, par exemple *Utilisateur de machine virtuelle* ou *Administrateur de machine virtuelle*. Si le rôle *Connexion de l’administrateur de l’ordinateur virtuel* est attribué à votre compte d’utilisateur, vous pouvez utiliser le paramètre `sudo` pour exécuter des commandes qui nécessitent des privilèges racine.

## <a name="sudo-and-aad-login"></a>Connexion sudo et AAD

La première fois que vous exécutez sudo, vous devrez vous authentifier une deuxième fois. Si vous ne souhaitez pas devoir vous authentifier à nouveau pour exécuter sudo, vous pouvez modifier votre fichier sudoers `/etc/sudoers.d/aad_admins` et remplacer cette ligne :

```bash
%aad_admins ALL=(ALL) ALL
```

Avec cette ligne :

```bash
%aad_admins ALL=(ALL) NOPASSWD:ALL
```


## <a name="troubleshoot-sign-in-issues"></a>Résoudre les problèmes de connexion

Certaines erreurs courantes se produisent lorsque vous essayez de vous connecter via le protocole SSH avec des informations d’identification Azure AD, notamment l’absence de l’attribution de rôles Azure et des invites répétées à se connecter. Consultez les sections suivantes pour corriger ces problèmes.

### <a name="access-denied-azure-role-not-assigned"></a>Accès refusé : rôle Azure non attribué

Si vous voyez l’erreur suivante sur l’invite SSH, vérifiez que vous avez configuré des stratégies Azure RBAC pour la machine virtuelle qui accordent à l’utilisateur le rôle *Connexion de l’administrateur aux machines virtuelles* ou *Connexion de l’utilisateur aux machines virtuelles* :

```output
login as: azureuser@contoso.onmicrosoft.com
Using keyboard-interactive authentication.
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FJX327AXD to authenticate. Press ENTER when ready.
Using keyboard-interactive authentication.
Access denied:  to sign-in you be assigned a role with action 'Microsoft.Compute/virtualMachines/login/action', for example 'Virtual Machine User Login'
Access denied
```
> [!NOTE]
> Si vous rencontrez des problèmes avec des attributions de rôles Azure, consultez [Résoudre les problèmes liés à Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/troubleshooting#azure-role-assignments-limit).

### <a name="continued-ssh-sign-in-prompts"></a>Invites de connexion SSH continues

Si vous effectuez correctement l’étape d’authentification dans un navigateur web, vous pouvez immédiatement être invité à vous reconnecter avec un nouveau code. Cette erreur est généralement due à une incompatibilité entre le nom de connexion spécifié à l’invite SSH et le compte avec lequel vous vous êtes connecté à Azure AD. Pour corriger ce problème :

- Vérifiez que le nom de connexion spécifié à l’invite SSH est correct. Une faute de frappe dans le nom de connexion peut provoquer une incompatibilité entre le nom de connexion spécifié à l’invite SSH et le compte avec lequel vous vous êtes connecté à Azure AD. Par exemple, que vous avez tapé *azuresuer\@contoso.onmicrosoft.com* au lieu de *azureuser\@contoso.onmicrosoft.com*.
- Si vous possédez plusieurs comptes d’utilisateurs, veillez à ne pas indiquer un autre compte d’utilisateur dans la fenêtre du navigateur lorsque vous vous connectez à Azure AD.
- Linux est un système d’exploitation qui respecte la casse. La différence entre « Azureuser@contoso.onmicrosoft.com » et « azureuser@contoso.onmicrosoft.com » peut entraîner une incompatibilité. Assurez-vous que vous spécifiez l’UPN en respectant la casse appropriée à l’invite SSH.

### <a name="other-limitations"></a>Autres limitations

Les utilisateurs qui héritent de droits d’accès par le biais de groupes imbriqués ou d’attributions de rôles ne sont pas pris en charge actuellement. L’utilisateur ou le groupe doit recevoir directement les [attributions de rôles requises](#configure-role-assignments-for-the-vm). Par exemple, l’utilisation d’attributions de rôles de groupes imbriqués ou de groupes d’administration n’accorde pas les autorisations appropriées pour permettre à l’utilisateur de se connecter.

## <a name="preview-feedback"></a>Commentaires de la préversion

Partager vos commentaires sur cette version préliminaire fonctionnalité ou signaler des problèmes sur le [forum de commentaires d’Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Active Directory, consultez [Qu’est-ce qu’Azure Active Directory ?](../../active-directory/fundamentals/active-directory-whatis.md)
