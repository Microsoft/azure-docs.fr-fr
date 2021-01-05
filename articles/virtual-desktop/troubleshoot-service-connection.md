---
title: Détecter un problème de connexion au service dans Windows Virtual Desktop – Azure
description: Comment résoudre des problèmes lorsque vous configurez des connexions de service dans un environnement de locataire Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 10/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6a9eb99ae7af88e77fa597fa92ff8e6278c307e6
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108947"
---
# <a name="windows-virtual-desktop-service-connections"></a>Connexions au service Windows Virtual Desktop

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/troubleshoot-service-connection-2019.md).

Appuyez-vous sur cet article pour résoudre les problèmes liés aux connexions à un client Windows Virtual Desktop.

## <a name="provide-feedback"></a>Fournir des commentaires

Vous pouvez nous fournir des commentaires et discuter du service Windows Virtual Desktop avec l’équipe de produit et d’autres membres actifs de la communauté dans [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

## <a name="user-connects-but-nothing-is-displayed-no-feed"></a>Lorsqu’un utilisateur se connecte, rien ne s’affiche (aucun flux)

Si l’utilisateur peut démarrer les clients Bureau à distance et s’authentifier, il ne voit cependant aucune icône dans le flux de découverte web.

1. Vérifiez que l’utilisateur signalant les problèmes a été affecté aux groupes d’applications à l’aide de cette ligne de commande :

     ```powershell
     Get-AzRoleAssignment -SignInName <userupn>
     ```

2. Vérifiez que l’utilisateur se connecte avec les informations d’identification correctes.

3. Si le client web est utilisé, vérifiez qu’il n’y a aucun problème d’informations d’identification mises en cache.

4. Si l’utilisateur fait partie d’un groupe d’utilisateurs Azure Active Directory (AD), assurez-vous que le groupe d’utilisateurs est un groupe de sécurité au lieu d’un groupe de distribution. Le bureau virtuel Windows ne prend pas en charge les groupes de distribution Azure AD.

## <a name="user-loses-existing-feed-and-no-remote-resource-is-displayed-no-feed"></a>L’utilisateur perd le flux existant et aucune ressource distante n’est affichée (aucun flux)

Cette erreur se produit généralement lorsqu’un utilisateur a déplacé son abonnement d’un locataire Azure AD vers un autre. Par conséquent, le service perd le suivi de ses attributions d’utilisateurs, car celles-ci sont encore liées à l’ancien locataire Azure AD.

Pour résoudre ce problème, il vous suffit de réaffecter les utilisateurs à leurs groupes d’applications.

Cela peut également se produire si un fournisseur de services de chiffrement a créé l’abonnement, puis l’a transféré au client. Pour résoudre ce problème, réinscrivez le fournisseur de ressources.

1. Connectez-vous au portail Azure.
2. Accédez à **Abonnements** , puis sélectionnez votre abonnement.
3. Dans le menu sur le côté gauche de la page, sélectionnez **Fournisseur de ressources** .
4. Recherchez et sélectionnez **Microsoft.DesktopVirtualization** , puis **Réinscrire** .

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir une vue d’ensemble de la résolution des problèmes Windows Virtual Desktop et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview.md).
- Pour résoudre des problèmes lors de la création d’un environnement Windows Virtual Desktop et d’un pool d’hôtes dans un environnement Windows Virtual Desktop, consultez [Création d’un environnement et d’un pool d’hôtes](troubleshoot-set-up-issues.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).
- Pour résoudre les problèmes d’utilisation de PowerShell avec Windows Virtual Desktop, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md).
- Suivez le [Didacticiel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
