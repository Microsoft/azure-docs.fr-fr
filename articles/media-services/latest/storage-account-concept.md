---
title: Comptes de stockage Azure
titleSuffix: Azure Media Services
description: Découvrez comment créer un compte de stockage Azure à utiliser avec Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: f37b453a294a0d0a7b9a99bfebe8f3eff09e8956
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291192"
---
# <a name="azure-storage-accounts"></a>Comptes de stockage Azure

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Pour commencer à gérer, chiffrer, coder, analyser, gérer et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services. Lorsque vous créez un compte Media Services, vous devez indiquer le nom d’une ressource de compte de stockage Azure. Le compte de stockage spécifié est lié à votre compte Media Services.

Le compte Media Services et tous les comptes de stockage associés doivent être faire partie du même abonnement Azure. Il est fortement recommandé d’utiliser des comptes de stockage situés au même emplacement que le compte Media Services afin d’éviter des frais supplémentaires de sortie des données et une importante latence.

Vous devez disposer d’un compte de stockage **principal** et vous pouvez avoir n’importe quel nombre de comptes de stockage **secondaires** associés à votre compte Media Services. Media Services prend en charge les comptes **v2 à usage général** (GPv2) ou **v1 à usage général** (GPv1). Les comptes Blob uniquement ne sont pas autorisés en tant que comptes **principaux**.

Nous vous recommandons d’utiliser GPv2 afin de tirer parti des dernières fonctionnalités et performances. Pour plus d’informations sur les comptes de stockage, consultez la [présentation générale du compte de stockage Azure](../../storage/common/storage-account-overview.md).

> [!NOTE]
> Seul le niveau d’accès chaud est pris en charge pour une utilisation avec Azure Media Services, bien que les autres niveaux d’accès puissent être utilisés pour réduire les coûts de stockage sur le contenu qui n’est pas activement utilisé.

Vous pouvez choisir entre différentes références SKU pour votre compte de stockage. Pour plus d’informations, consultez [Comptes de stockage](/cli/azure/storage/account?view=azure-cli-latest). Si vous voulez faire des expériences avec des comptes de stockage, utilisez `--sku Standard_LRS`. Cependant, lors de la sélection d’une référence SKU pour la production, envisagez `--sku Standard_RAGRS`, qui offre la réplication géographique pour la continuité de l’activité.

## <a name="assets-in-a-storage-account"></a>Ressources d’un compte de stockage

Dans Media Services v3, les API Stockage sont utilisées pour charger des fichiers dans des éléments multimédias. Pour plus d’informations, consultez [Éléments multimédias dans Azure Media Services v3](assets-concept.md).

> [!Note]
> N’essayez pas de modifier le contenu des conteneurs de blobs qui ont été générés par le Kit de développement logiciel (SDK) Media Services sans utiliser les API Media Services.

## <a name="storage-side-encryption"></a>Chiffrement côté stockage

Pour protéger vos éléments multimédias au repos, les ressources doivent être chiffrées par le chiffrement côté stockage. Le tableau suivant montre comment fonctionne le chiffrement côté stockage dans Media Services v3 :

|Option de chiffrement|Description|Media Services v3|
|---|---|---|
|Chiffrement du stockage Media Services| Chiffrement AES-256, clé gérée par Media Services. |Non pris en charge.<sup>(1)</sup>|
|[Chiffrement du service de stockage pour les données au repos](../../storage/common/storage-service-encryption.md)|Chiffrement côté serveur proposé par le stockage Azure, clé gérée par Azure ou par un client.|Pris en charge.|
|[Chiffrement du stockage côté client](../../storage/common/storage-client-side-encryption.md)|Chiffrement côté client proposé par le stockage Azure, clé gérée par un client dans un coffre de clés.|Non pris en charge.|

<sup>1</sup> Dans Media Services v3, le chiffrement de stockage (chiffrement AES-256) est uniquement pris en charge pour la compatibilité descendante lorsque vos ressources ont été créées avec Media Services v2. Cela signifie que v3 fonctionne avec les ressources chiffrées du stockage existant, mais n’autorise pas la création de nouveaux éléments.

## <a name="storage-account-errors"></a>Erreurs relatives au compte de stockage

L’état « Déconnecté » pour un compte Media Services indique que ce compte n’a plus accès à un ou plusieurs des comptes de stockage attachés en raison d’un changement dans les clés d’accès du stockage. Des clés d’accès du stockage à jour sont nécessaires à Media Services pour effectuer de nombreuses tâches dans le compte.

Voici les principaux scénarios qui aboutissent à un compte Media Services n’ayant pas accès aux comptes de stockage attachés.

|Problème|Solution|
|---|---|
|Le compte Media Services ou un ou plusieurs comptes de stockage attachés ont été migrés vers des abonnements distincts. |Migrez les comptes de stockage ou le compte Media Services afin qu’ils soient tous dans le même abonnement. |
|Le compte Media Services utilise un compte de stockage attaché dans un abonnement différent car il s’agissait d’un ancien compte Media Services où ceci était pris en charge. Tous les anciens comptes Media Services ont été convertis en comptes modernes basés sur Azure Resource Manager et ont un état Déconnecté. |Migrez le compte de stockage ou le compte Media Services afin qu’ils soient tous dans le même abonnement.|

## <a name="azure-storage-firewall"></a>Pare-feu de Stockage Azure

La plateforme Azure Media Services ne prend pas en charge les comptes de stockage dont le pare-feu de Stockage Azure ou les [points de terminaison privés](../../storage/common/storage-network-security.md) sont activés.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment attacher un compte de stockage à votre compte Media Services, consultez [Créer un compte](./create-account-howto.md).
