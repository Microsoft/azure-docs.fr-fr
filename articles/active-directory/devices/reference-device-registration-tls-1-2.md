---
title: Application de TLS 1.2 – Service d’inscription d’Azure Active Directory
description: Supprimer la prise en charge de TLS 1.0 et 1.1 pour le service Azure AD Device Registration
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: reference
ms.date: 07/10/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2bb8c6c64e0a68f5176c4eb0c0177c5220394695
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89268755"
---
# <a name="enforce-tls-12-for-the-azure-ad-registration-service"></a>Appliquer le protocole TLS 1.2 pour le service d’inscription d’Azure AD

Le service Azure Active Directory (Azure AD) Device Registration est utilisé pour connecter des appareils au cloud avec une identité d’appareil. Le service Azure AD Device Registration prend actuellement en charge l’utilisation du protocole TLS (Transport Layer Security) 1.2 pour les communications avec Azure. Pour garantir la sécurité et le meilleur cryptage de sa catégorie, Microsoft recommande de désactiver TLS 1.0 et 1.1. Ce document fournit des informations sur la façon de s’assurer que les machines utilisées pour effectuer l’inscription et communiquer avec le service Azure AD Device Registration utilisent TLS 1.2.

Le protocole TLS version 1.2 est un protocole de chiffrement conçu pour assurer des communications sécurisées. Le protocole TLS vise principalement à assurer la confidentialité et l’intégrité des données. Ce protocole a subi de nombreuses itérations avec la version 1.2 définie dans le document [RFC 5246 (lien externe)](https://tools.ietf.org/html/rfc5246).

L’analyse actuelle des connexions montre peu d’utilisation de TLS 1.1 et 1.0, mais nous fournissons ces informations afin que vous puissiez mettre à jour les clients ou serveurs concernés si nécessaire avant la fin de la prise en charge de TLS 1.1 et 1.0. Si vous utilisez une infrastructure locale pour des scénarios hybrides ou des services de fédération Active Directory (AD FS), assurez-vous que l’infrastructure peut prendre en charge les connexions entrantes et sortantes qui utilisent TLS 1.2.

## <a name="update-windows-servers"></a>Mettre à jour les serveurs Windows

Pour les serveurs Windows qui utilisent le service Azure AD Device Registration ou agissent en tant que proxys, procédez comme suit pour vous assurer que TLS 1.2 est activé :

> [!IMPORTANT]
> Une fois que vous avez mis à jour le registre, vous devez redémarrer le serveur Windows pour que les modifications prennent effet.

### <a name="enable-tls-12"></a>Activer le protocole TLS 1.2

Vérifiez que les chaînes de registre suivantes sont configurées comme indiqué :

- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client
  - "DisabledByDefault"=dword:00000000
  - "Enabled"=dword:00000001
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server
  - "DisabledByDefault"=dword:00000000
  - "Enabled"=dword:00000001
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
  - "SchUseStrongCrypto"=dword:00000001

## <a name="update-non-windows-proxies"></a>Mettre à jour les proxys non-Windows

Toutes les machines qui jouent un rôle de proxy entre les appareils et le service Azure AD Device Registration doivent s’assurer d’utiliser TLS 1.2. Suivez les instructions de votre fournisseur pour garantir la prise en charge.

## <a name="update-ad-fs-servers"></a>Mettre à jour les serveurs AD FS

Les serveurs AD FS utilisés pour communiquer avec le service Azure AD Device Registration doivent s’assurer d’utiliser TLS 1.2. Pour plus d’informations sur la manière d’activer et de vérifier cette configuration, consultez [Gestion des protocoles SSL/TLS et des suites de chiffrement pour AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs).

## <a name="client-updates"></a>Mises à jour de client

Comme toutes les combinaisons client-serveur et navigateur-serveur doivent utiliser TLS 1.2 pour se connecter au service Azure AD Device Registration, vous devrez peut-être mettre à jour ces appareils.

Les clients suivants sont connus pour ne pas prendre en charge TLS 1.2. Mettez à jour vos clients pour garantir un accès ininterrompu.

- Android version 4.3 et versions antérieures
- Firefox version 5.0 et versions antérieures
- Internet Explorer versions 8 à 10 sur Windows 7 et versions antérieures
- Internet Explorer 10 sur Windows Phone 8.0
- Safari version 6.0.4 sur OS X 10.8.4 et versions antérieures

## <a name="next-steps"></a>Étapes suivantes

[Vue d’ensemble de TLS/SSL (SSP Schannel)](/windows-server/security/tls/tls-ssl-schannel-ssp-overview)