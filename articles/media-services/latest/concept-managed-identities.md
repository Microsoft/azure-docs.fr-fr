---
title: Identités managées et stockage approuvé avec Media Services
description: La plateforme Media Services peut être utilisée avec des identités managées pour permettre un stockage approuvé.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.date: 11/04/2020
ms.author: inhenkel
ms.openlocfilehash: d0811e8f9183ee334d413bcad69f2c7b32023be3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499354"
---
# <a name="managed-identities-and-trusted-storage-with-media-services"></a>Identités managées et stockage approuvé avec Media Services

La plateforme Media Services peut être utilisée avec des [identités managées](../../active-directory/managed-identities-azure-resources/overview.md) pour permettre un stockage approuvé. Lorsque vous créez un compte Media Services, vous devez l’associer à un compte de stockage. La plateforme Media Services peut accéder à ce compte de stockage à l’aide d’une authentification système. Media Services vérifie que le compte Media Services et le compte de stockage se trouvent dans le même abonnement, et que l’utilisateur qui ajoute l’association a accès au compte de stockage avec le RBAC Azure Resource Manager.

## <a name="trusted-storage"></a>Stockage approuvé

Toutefois, si vous souhaitez utiliser un pare-feu pour sécuriser votre compte de stockage, vous devez utiliser une authentification d’identité managée. Celle-ci permet à la plateforme Media Services d’accéder au compte de stockage configuré avec un pare-feu ou une restriction de réseau virtuel via un accès à un stockage approuvé.  Pour plus d’informations sur les services Microsoft approuvés, consultez [Configurer des pare-feux et des réseaux virtuels dans Stockage Azure](../../storage/common/storage-network-security.md#trusted-microsoft-services).

## <a name="media-services-managed-identity-scenarios"></a>Scénarios d’identité managée de la plateforme Media Services

Il existe actuellement deux scénarios où une identité managée peut être utilisée avec la plateforme Media Services :

- utiliser l’identité managée du compte Media Services pour accéder à des comptes de stockage ;

- utiliser l’identité managée du compte Media Services pour accéder au Key Vault afin d’accéder à des clés client.

Les deux sections suivantes décrivent les différences entre les deux scénarios.

### <a name="use-the-managed-identity-of-the-media-services-account-to-access-storage-accounts"></a>Utiliser l’identité managée du compte Media Services pour accéder à des comptes de stockage

1. Créez un compte Media Services avec une identité managée.
1. Accordez au principal de l’identité managée l’accès à un compte de stockage que vous possédez.
1. La plateforme Media Services peut alors accéder au compte de stockage en votre nom à l’aide de l’identité managée.

### <a name="use-the-managed-identity-of-the-media-services-account-to-access-key-vault-to-access-customer-keys"></a>Utiliser l’identité managée du compte Media Services pour accéder au Key Vault afin d’accéder à des clés client

1. Créez un compte Media Services avec une identité managée.
1. Accordez au principal de l’identité managée l’accès à un Key Vault que vous possédez.
1. Configurez le compte Media Services pour utiliser un chiffrement de compte basé sur une clé client.
1. La plateforme Media Services accède au Key Vault en votre nom à l’aide de l’identité managée.

Pour plus d’informations sur les clés managées par le client et le Key Vault, consultez [Utiliser des clés BYOK (clé gérées par le client) avec Media Services](concept-use-customer-managed-keys-byok.md).

## <a name="tutorials"></a>Tutoriels

Ces tutoriels incluent les deux scénarios mentionnés ci-dessus.

- [Utiliser le portail Azure pour utiliser des clés managées par le client ou BYOK avec la plateforme Media Services](tutorial-byok-portal.md)
- [Utiliser des clés managées par le client ou BYOK avec l’API REST Media Services](tutorial-byok-postman.md)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur ce que les identités managées peuvent faire pour vous et vos applications Azure, consultez [Identités managées Azure AD](../../active-directory/managed-identities-azure-resources/overview.md).