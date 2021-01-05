---
title: Utiliser des clés BYOK (clé gérées par le client) avec Media Services
description: Vous pouvez utiliser des clés gérées par le client (autrement dit apporter votre propre clé) avec Media Services.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: a56922c972efeb21c188413522bd05f83b74ca12
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681820"
---
# <a name="bring-your-own-key-customer-managed-keys-with-media-services"></a>Utiliser des clés BYOK (clé gérées par le client) avec Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Bring Your Own Key (BYOK) est une initiative à l’échelle d’Azure qui aide les clients à déplacer leurs charges de travail vers le cloud. Les clés gérées par le client permettent aux clients d’adhérer aux réglementations du secteur en matière de conformité et d’améliorer l’isolation des locataires d’un service. Le fait de donner le contrôle des clés de chiffrement aux clients est un moyen de réduire l’accès et le contrôle inutiles et d’améliorer la confiance dans les services Microsoft.

## <a name="keys-and-key-management"></a>Clés et gestion des clés

Vous pouvez utiliser votre propre clé avec Media Services lorsque vous utilisez l’API Media Services 2020-05-01. Une clé de compte par défaut est créée pour tous les comptes chiffrés par une clé système appartenant à Media Services. Lorsque vous utilisez votre propre clé, la clé de compte est chiffrée avec votre clé. Les clés de contenu sont chiffrées par la clé de compte. Les URL JobInputHttp et les clés de validation de jeton symétriques sont également chiffrées.

:::image type="content" source="./media/customer-managed-key/customer-managed-key.svg" alt-text="Une clé gérée par le client remplace une clé gérée par le système":::

Media Services utilise l’identité managée du compte Media Services pour lire votre clé à partir du Key Vault dont vous êtes propriétaire. Media Services exige que le Key Vault se trouve dans la même région que le compte, et que les options de suppression réversible et de protection contre la suppression définitive soient activées.

Votre clé peut être une clé RSA 2048, 3072 ou 4096, et les clés HSM et logicielles sont toutes deux prises en charge.

> [!NOTE]
> Les clés EC ne sont pas prises en charge.

Vous pouvez spécifier un nom et une version de clé, ou simplement un nom de clé. Si vous utilisez seulement un nom de clé, Media Services utilise la dernière version de la clé. Les nouvelles versions de clés de client sont détectées automatiquement et la clé de compte est à nouveau chiffrée.

> [!WARNING]
> Media Services surveille l’accès à la clé de client. Si la clé de client devient inaccessible (par exemple, si elle-même, le Key Vault ou l’autorisation d’accès est supprimé), Media Services définit l’état de la clé de client du compte sur Inaccessible (ce qui a pour effet de désactiver le compte). Le compte peut toutefois être supprimé même dans cet état. Les seules opérations prises en charge sont les opérations d’extraction (GET), de liste (LIST) et de suppression (DELETE) de compte. Toutes les autres requêtes (encodage, diffusion en continu, etc.) échouent tant que l’accès à la clé du compte n’est pas restauré.

## <a name="tutorials"></a>Tutoriels

- [Utiliser le portail Azure pour utiliser des clés managées par le client ou BYOK avec la plateforme Media Services](tutorial-byok-portal.md)
- [Utiliser des clés managées par le client ou BYOK avec l’API REST Media Services](tutorial-byok-postman.md).

## <a name="next-steps"></a>Étapes suivantes

[Protéger votre contenu à l’aide du chiffrement dynamique de Media Services](content-protection-overview.md)
