---
title: Protocole TLS dans la sauvegarde Azure
description: Découvrez comment activer la sauvegarde Azure pour utiliser le protocole de chiffrement TLS (Transport Layer Security) et sécuriser les données lors de leur transfert sur un réseau.
ms.topic: conceptual
ms.date: 11/01/2020
ms.openlocfilehash: ba9c9d91f562f54695a0739908c8a409d14d5852
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327115"
---
# <a name="transport-layer-security-in-azure-backup"></a>Protocole TLS dans la sauvegarde Azure

Le protocole TLS (Transport Layer Security) est un protocole de chiffrement qui assure la sécurité des données lors de leur transfert sur un réseau. La sauvegarde Azure utilise le protocole TLS pour protéger la confidentialité des données de sauvegarde en cours de transfert. Cet article décrit les étapes à suivre pour activer le protocole TLS 1.2 qui offre une sécurité améliorée par rapport aux versions précédentes.

## <a name="earlier-versions-of-windows"></a>Versions antérieures de Windows

Si la machine exécute des versions antérieures de Windows, les mises à jour correspondantes indiquées ci-dessous doivent être installées, et les modifications de Registre documentées dans les articles de la Base de connaissances doivent être appliqués.

|Système d’exploitation  |article de la Base de connaissances |
|---------|---------|
|Windows Server 2008 SP2   |   <https://support.microsoft.com/help/4019276>      |
|Windows Server 2008 R2, Windows 7, Windows Server 2012   | <https://support.microsoft.com/help/3140245>         |

>[!NOTE]
>La mise à jour installe les composants de protocole nécessaires. Après l’installation, vous devez procéder aux modifications de clé de Registre mentionnées dans les articles de la Base de connaissances ci-dessus pour activer correctement les protocoles demandés.

## <a name="verify-windows-registry"></a>Vérifier le Registre Windows

### <a name="configuring-schannel-protocols"></a>Configuration des protocoles SChannel

Les clés de Registre suivantes garantissent que le protocole TLS 1.2 est activé au niveau du composant SChannel :

```reg
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
    "Enabled"=dword:00000001

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
    "DisabledByDefault"=dword:00000000
```

>[!NOTE]
>Les valeurs affichées sont définies par défaut dans Windows Server 2012 R2 et les versions plus récentes. Pour ces versions de Windows, si les clés de Registre sont absentes, elles n’ont pas besoin d’être créées.

### <a name="configuring-net-framework"></a>Configuration de .NET Framework

Les clés de Registre suivantes configurent .NET Framework pour prendre en charge un chiffrement renforcé. Vous trouverez plus d’informations sur la [configuration de .NET Framework ici](/dotnet/framework/network-programming/tls#configuring-schannel-protocols-in-the-windows-registry).

```reg
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto" = dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto" = dword:00000001
```

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="why-enable-tls-12"></a>Pourquoi activer TLS 1.2 ?

Le protocole TLS 1.2 est plus sécurisé que les protocoles de chiffrement précédents, tels que SSL 2.0, SSL 3.0, TLS 1.0 et TLS 1.1. Les services de Sauvegarde Azure prennent déjà en charge TLS 1.2 dans son intégralité.

### <a name="what-determines-the-encryption-protocol-used"></a>Qu’est-ce qui détermine le protocole de chiffrement utilisé ?

La version de protocole la plus élevée qui est prise en charge par le client et le serveur est négociée pour établir la conversation chiffrée. Pour plus d’informations sur le protocole de négociation TLS, consultez [Établissement d’une session sécurisée à l’aide du protocole TLS](/windows/win32/secauthn/tls-handshake-protocol#establishing-a-secure-session-by-using-tls).

### <a name="what-is-the-impact-of-not-enabling-tls-12"></a>Quel est l’impact de la non-activation de TLS 1.2 ?

Pour améliorer la sécurité contre les attaques par passage à une version antérieure de protocole, la sauvegarde Azure commence par désactiver graduellement les versions de TLS antérieures à 1.2. Il s’agit d’un changement à long terme opéré entre les services pour interdire les connexions de suite de chiffrement et de protocole héritées. Les services et les composants de Sauvegarde Azure prennent en charge le protocole TLS 1.2 dans son intégralité. Toutefois, les versions de Windows non pourvues des mises à jour obligatoires ou de certaines configurations personnalisées peuvent toujours empêcher la mise à disposition des protocoles TLS 1.2. Des défaillances peuvent en résulter, parmi lesquelles une ou plusieurs des situations suivantes, mais pas seulement :

- Des opérations de sauvegarde et de restauration peuvent échouer.
- Échecs de connexion des composants de sauvegarde avec l’erreur 10054 (une connexion existante a été fermée de force par l’hôte distant).
- Les services liés à la sauvegarde Azure ne s’arrêtent pas ou ne démarrent pas comme d’habitude.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Protocole TLS](/windows/win32/secauthn/transport-layer-security-protocol)
- [Assurer la prise en charge de TLS 1.2 sur les systèmes d’exploitation déployés](/security/engineering/solving-tls1-problem#ensuring-support-for-tls-12-across-deployed-operating-systems)
- [Meilleures pratiques Transport Layer Security (TLS) avec .NET Framework](/dotnet/framework/network-programming/tls)