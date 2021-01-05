---
title: Résoudre les problèmes de jonction à un domaine avec Azure AD Domain Services | Microsoft Docs
description: Découvrez comment résoudre les problèmes courants quand vous essayez de joindre une machine virtuelle à un domaine ou de connecter une application à Azure Active Directory Domain Services et que vous ne pouvez pas vous connecter au domaine managé ni vous authentifier auprès de celui-ci.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 86d6ca79a12e4706f558e92c3c83c5bddaa99b3c
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618601"
---
# <a name="troubleshoot-domain-join-problems-with-an-azure-active-directory-domain-services-managed-domain"></a>Résoudre les problèmes de jonction à un domaine avec un domaine managé Azure Active Directory Domain Services

Lorsque vous essayez de joindre une machine virtuelle ou de connecter une application à un domaine managé Azure Active Directory Domain Services (Azure AD DS), vous pouvez obtenir une erreur indiquant que vous ne pouvez pas le faire. Pour résoudre les problèmes de jonction à un domaine, identifiez les points où vous avez un problème parmi les suivants :

* Si vous ne recevez pas d’invite d’authentification, la machine virtuelle ou l’application ne peut pas se connecter au domaine managé Azure AD DS.
    * Commencez à résoudre les [problèmes de connectivité pour la jonction à un domaine](#connectivity-issues-for-domain-join).
* Si vous recevez une erreur lors de l’authentification, la connexion au domaine managé est établie.
    * Commencez à résoudre les [problèmes liés aux informations d’identification lors de la jonction à un domaine](#credentials-related-issues-during-domain-join).

## <a name="connectivity-issues-for-domain-join"></a>Problèmes de connectivité pour la jonction à un domaine

Si la machine virtuelle ne peut pas trouver le domaine managé, il s’agit généralement d’un problème de configuration ou de connexion réseau. Passez en revue les étapes suivantes de résolution des problèmes pour localiser et résoudre le problème :

1. Vérifiez que la machine virtuelle est connectée au même réseau virtuel, ou à un réseau virtuel appairé en tant que domaine managé. Si ce n’est pas le cas, la machine virtuelle ne peut pas trouver le domaine et s’y connecter pour le rejoindre.
    * Si la machine virtuelle n’est pas connectée au même réseau virtuel, vérifiez que l’appairage de réseau virtuel ou la connexion VPN est *Active* (Actif) ou *Connected* (Connecté) pour permettre au trafic de circuler correctement.
1. Effectuez un test Ping du domaine en utilisant le nom du domaine managé (par exemple, `ping aaddscontoso.com`).
    * Si la réponse au test Ping échoue, essayez d’effectuer un test Ping des adresses IP du domaine affiché dans la page de présentation du portail pour votre domaine managé, par exemple `ping 10.0.0.4`.
    * Si le test Ping de l’adresse IP aboutit contrairement à celui du domaine, il se peut que la fonction DNS ne soit pas correctement configurée. Vérifiez que vous avez [configuré les serveurs DNS du domaine managé pour le réseau virtuel][configure-dns].
1. Essayez de vider le cache de résolution DNS sur la machine virtuelle, par exemple `ipconfig /flushdns`.

### <a name="network-security-group-nsg-configuration"></a>Configuration du groupe de sécurité réseau

Lorsque vous créez un domaine managé, un groupe de sécurité réseau est également créé avec les règles appropriées pour une opération réussie du domaine. Si vous modifiez ou créez des règles de groupe de sécurité réseau, vous pouvez involontairement bloquer les ports requis pour Azure AD DS afin de fournir des services de connexion et d’authentification. Ces règles de groupe de sécurité réseau peuvent provoquer des problèmes comme une synchronisation de mot de passe qui ne se termine pas, des utilisateurs dans l’impossibilité de se connecter ou des problèmes de jonction à un domaine.

Si vous continuez de rencontrer des problèmes de connexion, passez en revue les étapes suivantes de résolution des problèmes :

1. Vérifiez l’état d’intégrité de votre domaine managé dans le portail Azure. Si vous avez une alerte pour *AADDS001*, une règle de groupe de sécurité réseau bloque l’accès.
1. Examinez les [règles de groupe de sécurité réseau et ports requis][network-ports]. Vérifiez qu’aucune règle de groupe de sécurité réseau appliquée à la machine virtuelle ou au réseau virtuel à partir duquel vous vous connectez ne bloque ces ports réseau.
1. Une fois les problèmes de configuration d’un groupe de sécurité réseau résolus, l’alerte *AADDS001* disparaît de la page d’intégrité en 2 heures environ. Maintenant que la connectivité réseau est disponible, essayez à nouveau de joindre la machine virtuelle au domaine.

## <a name="credentials-related-issues-during-domain-join"></a>Problèmes liés aux informations d’identification lors de la jonction à un domaine

Si une boîte de dialogue s’affiche et vous demande d’entrer des informations d’identification pour rejoindre le domaine managé, la machine virtuelle est en mesure de se connecter au domaine à l’aide du réseau virtuel Azure. Le processus de jonction au domaine échoue lors de l’authentification auprès du domaine ou de l’autorisation d’effectuer le processus de jonction au domaine à l’aide des informations d’identification fournies.

Pour résoudre les problèmes liés aux informations d’identification, passez en revue les étapes suivantes de résolution des problèmes :

1. Essayez en utilisant le format UPN pour spécifier les informations d’identification, par exemple `dee@contoso.onmicrosoft.com`. Vérifiez que cet UPN est correctement configuré dans Azure AD.
    * La valeur de *SAMAccountName* pour votre compte peut être générée automatiquement s’il existe plusieurs utilisateurs avec le même préfixe UPN dans votre locataire ou si votre préfixe UPN est trop long. Par conséquent, le format *SAMAccountName* de votre compte peut être différent de ce que vous attendiez ou de celui que vous utilisez dans votre domaine local.
1. Essayez d’utiliser les informations d’identification d’un compte d’utilisateur faisant partie du domaine managé pour joindre des machines virtuelles au domaine managé.
1. Vérifiez que vous avez [activé la synchronisation de mot de passe][enable-password-sync] et que vous avez attendu suffisamment de temps pour que la synchronisation de mot de passe initiale se termine.

## <a name="next-steps"></a>Étapes suivantes

Pour une compréhension plus approfondie des processus Active Directory dans le cadre de l’opération de jonction à un domaine, consultez [Problèmes de jonction et d’authentification][join-authentication-issues].

Si vous rencontrez toujours des problèmes pour joindre votre machine virtuelle au domaine managé, [trouvez de l’aide et ouvrez un ticket de support pour Azure Active Directory][azure-ad-support].

<!-- INTERNAL LINKS -->
[enable-password-sync]: tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds
[network-ports]: network-considerations.md#network-security-groups-and-required-ports
[azure-ad-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[configure-dns]: tutorial-create-instance.md#update-dns-settings-for-the-azure-virtual-network

<!-- EXTERNAL LINKS -->
[join-authentication-issues]: /previous-versions/windows/it-pro/windows-2000-server/cc961817(v=technet.10)
