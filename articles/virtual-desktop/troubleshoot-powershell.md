---
title: Windows Virtual Desktop PowerShell – Azure
description: Comment résoudre des problèmes avec PowerShell quand vous configurez un environnement Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 03b6da1d35247749d8ec2c6459c8ddee69bfccb6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88002277"
---
# <a name="windows-virtual-desktop-powershell"></a>Windows Virtual Desktop PowerShell

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/troubleshoot-powershell-2019.md).

Cet article vous aide à résoudre des problèmes et erreurs rencontrés lors de l’utilisation de PowerShell avec Windows Virtual Desktop. Pour plus d’informations sur les Services Bureau à distance PowerShell, voir [Windows Virtual Desktop PowerShell](/powershell/module/windowsvirtualdesktop/).

## <a name="provide-feedback"></a>Fournir des commentaires

Rendez-vous sur le site [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) pour discuter du service Windows Virtual Desktop avec l’équipe de produit et les membres actifs de la communauté.

## <a name="powershell-commands-used-during-windows-virtual-desktop-setup"></a>Commandes PowerShell utilisées pendant la configuration de Windows Virtual Desktop

Cette section répertorie les commandes PowerShell qui sont généralement utilisées lors de la configuration de Windows Virtual Desktop, et explique comment résoudre des problèmes qui peuvent se produire lors de leur utilisation.

### <a name="error-new-azroleassignment-the-provided-information-does-not-map-to-an-ad-object-id"></a>Erreur : New-AzRoleAssignment: Les informations fournies ne correspondent pas à un ID d’objet AD.

```powershell
New-AzRoleAssignment -SignInName "admins@contoso.com" -RoleDefinitionName "Desktop Virtualization User" -ResourceName "0301HP-DAG" -ResourceGroupName 0301RG -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

**Cause :** L’utilisateur spécifié par le paramètre *-SignInName* est introuvable dans l’Azure Active Directory lié à l’environnement Windows Virtual Desktop.

**Correctif :** Vérifiez les points suivants.

- L’utilisateur doit être synchronisé avec Azure Active Directory.
- L’utilisateur ne doit pas être lié à commerce B2C ou B2B.
- L’environnement Windows Virtual Desktop doit être lié à l’Azure Active Directory correct.

### <a name="error-new-azroleassignment-the-client-with-object-id-does-not-have-authorization-to-perform-action-over-scope-code-authorizationfailed"></a>Erreur : New-AzRoleAssignment: « Le client avec l’ID d’objet n’est pas autorisé à effectuer l’action sur l’étendue (code : AuthorizationFailed) »

**Cause 1 :** Le compte utilisé ne dispose pas d’autorisations de propriétaire sur l’abonnement.

**Correctif 1 :** Un utilisateur disposant d’autorisations de propriétaire doit exécuter l’attribution de rôle. Ou bien, l’utilisateur doit être affecté au rôle Administrateur de l’accès utilisateur pour affecter un utilisateur à un groupe d’applications.

**Cause 2 :** Le compte utilisé dispose d’autorisations de propriétaire, mais ne fait pas partie de l’Azure Active Directory de l’environnement ou n’est pas autorisé à interroger l’Azure Active Directory dans lequel se trouve l’utilisateur.

**Correctif 2 :** Un utilisateur disposant d’autorisations Active Directory doit exécuter l’attribution de rôle.

### <a name="error-new-azwvdhostpool----the-location-is-not-available-for-resource-type"></a>Erreur : New-AzWvdHostPool -- l’emplacement n’est pas disponible pour le type de ressource

```powershell
New-AzWvdHostPool_CreateExpanded: The provided location 'southeastasia' is not available for resource type 'Microsoft.DesktopVirtualization/hostpools'. List of available regions for the resource type is 'eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,centralus'.
```

Cause : Windows Virtual Desktop prend en charge la sélection de l’emplacement des pools d’hôtes, des groupes d’applications et des espaces de travail pour stocker les métadonnées de service dans certains emplacements. Vos options sont limitées en fonction de l’emplacement où cette fonctionnalité est disponible. Cette erreur signifie que la fonctionnalité n’est pas disponible à l’emplacement que vous avez choisi.

Correctif : La liste des régions prises en charge sera publiée dans le message d’erreur. Utilisez plutôt l’une des régions prises en charge.

### <a name="error-new-azwvdapplicationgroup-must-be-in-same-location-as-host-pool"></a>Erreur : New-AzWvdApplicationGroup doit se trouver dans le même emplacement que le pool d’hôtes

```powershell
New-AzWvdApplicationGroup_CreateExpanded: ActivityId: e5fe6c1d-5f2c-4db9-817d-e423b8b7d168 Error: ApplicationGroup must be in same location as associated HostPool
```

**Cause :** Il y a une incompatibilité d’emplacement. Tous les pools d’hôtes, groupes d’applications et espaces de travail ont un emplacement pour stocker les métadonnées du service. Tous les objets que vous créez, qui sont associés entre eux, doivent se trouver dans le même emplacement. Par exemple, si un pool d’hôtes se trouve dans `eastus`, vous devez également créer les groupes d’applications dans `eastus`. Si vous créez un espace de travail auquel inscrire ces groupes d’applications, cet espace de travail doit également être dans `eastus`.

**Correctif :** Récupérez l’emplacement dans lequel le pool d’hôtes a été créé, puis affectez le groupe d’applications que vous créez à ce même emplacement.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir une vue d’ensemble de la résolution des problèmes Windows Virtual Desktop et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview.md).
- Pour résoudre les problèmes de configuration de vos environnement et pools d’hôtes Windows Virtual Desktop, consultez [Création d’un environnement et d’un pool d’hôtes](troubleshoot-set-up-issues.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).
- Pour résoudre les problèmes de connexion au client Windows Virtual Desktop, consultez [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection.md).
- Pour résoudre les problèmes liés aux clients Bureau à distance, consultez [Résoudre des problèmes du client Bureau à distance](troubleshoot-client.md).
- Pour plus d’informations sur le service, consultez [Environnement Windows Virtual Desktop](environment-setup.md).
- Pour en savoir plus sur les actions d’audit, consultez [Opérations d’audit avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Pour en savoir plus sur les actions visant à déterminer les erreurs au cours du déploiement, consultez [Voir les opérations de déploiement](../azure-resource-manager/templates/deployment-history.md).
