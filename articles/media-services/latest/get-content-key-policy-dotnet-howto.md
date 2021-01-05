---
title: Obtenir une clé de signature à partir d’une stratégie à l’aide du kit SDK .NET d’Azure Media Services v3
description: Cette rubrique explique comment obtenir une clé de signature à partir de la stratégie existante à l'aide du kit SDK .NET de Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 7107a2acdc0ca430797245125ca903f7414908da
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427065"
---
# <a name="get-a-signing-key-from-the-existing-policy"></a>Obtenir une clé de signature à partir de la stratégie existante

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

L’un des principes de conception clés de l’API v3 est de renforcer la sécurité de l’API. Les API v3 ne retournent pas de secrets ou d’informations d’identification lors des opérations **Get** ou **List** . Consultez l’explication détaillée ici : Pour plus d’informations, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC) et comptes Media Services](rbac-overview.md)

L'exemple présenté dans cet article explique comment utiliser .NET pour obtenir une clé de signature à partir de la stratégie existante. 
 
## <a name="download"></a>Téléchargement 

Clonez un référentiel GitHub contenant l'exemple .NET complet sur votre machine à l'aide de la commande suivante :  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
L'exemple de stratégie ContentKeyPolicy contenant les secrets se trouve dans le dossier [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM).

## <a name="get-contentkeypolicy-with-secrets"></a>Obtenir la stratégie ContentKeyPolicy contenant les secrets 

Pour accéder à la clé, utilisez **GetPolicyPropertiesWithSecretsAsync** , comme indiqué dans l'exemple ci-dessous.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="next-steps"></a>Étapes suivantes

[Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md) 
