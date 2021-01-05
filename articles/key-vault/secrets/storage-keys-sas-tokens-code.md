---
title: Récupérer les jetons de signature d’accès partagé dans le code | Azure Key Vault
description: La fonctionnalité de compte de stockage géré assure une intégration fluide entre Azure Key Vault et un compte de stockage Azure. Cet exemple utilise le kit SDK Azure pour .NET qui permet de gérer les jetons SAS.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 2a0202c5259ccebedf03ade217f57b6305b9fa1b
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444926"
---
# <a name="create-sas-definition-and-fetch-shared-access-signature-tokens-in-code"></a>Créer une définition SAS et récupérer les jetons de signature d’accès partagé dans le code

Vous pouvez gérer votre compte de stockage avec les jetons de signature d’accès partagé(SAS) dans votre coffre de clés. Pour plus d’informations, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des SAS](../../storage/common/storage-sas-overview.md).

> [!NOTE]
> Nous vous recommandons d’utiliser le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../storage/common/storage-auth-aad.md) pour sécuriser votre compte de stockage et bénéficier ainsi d’une sécurité renforcée et d’une plus grande facilité d’utilisation par rapport à une autorisation de clé partagée.

Cet article fournit des exemples de code .NET qui crée une définition SAS et récupère des jetons SAS. Pour plus d’informations, consultez notre exemple [ShareLink](/samples/azure/azure-sdk-for-net/share-link/), notamment le client généré pour les comptes de stockage gérés par Key Vault. Pour plus d’informations sur la façon de créer et de stocker des jetons SAS, consultez [Gérer les clés de compte de stockage avec Key Vault et l’interface de ligne de commande Azure](overview-storage-keys.md) ou [Gérer les clés de compte de stockage avec Key Vault et Azure PowerShell](overview-storage-keys-powershell.md).

## <a name="code-samples"></a>Exemples de code

Dans l’exemple suivant, nous allons créer un modèle SAS :

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="91-97":::

À l’aide de ce modèle, nous pouvons créer une définition SAS au moyen de 

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="137-156":::

Une fois la définition SAS créée, vous pouvez récupérer des jetons SAS comme des secrets au moyen de `SecretClient`. Vous devez faire précéder le nom du secret par le nom du compte de stockage suivi d’un tiret :

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="52-58":::

Si votre jeton de signature d’accès partagé arrive à expiration, vous pouvez récupérer le même secret pour en générer un nouveau.

Vous obtenez des instructions sur l’utilisation d’un jeton SAS récupéré à partir de Key Vault pour accéder aux services de stockage Azure en consultant [Utiliser une signature d’accès partagé de compte pour accéder au service BLOB.](../../storage/common/storage-account-sas-create-dotnet.md#use-an-account-sas-from-a-client)

> [!NOTE]
> Votre application doit être préparée à l’actualisation de la SAP si elle reçoit du stockage une erreur 403, afin de pouvoir gérer le cas où une clé a été compromise de sorte que vous devez opérer une rotation de clé avant la fin de la période de rotation normale. 

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [Accorder un accès limité aux ressources du Stockage Azure à l’aide des SAS](../../storage/common/storage-sas-overview.md).
- Découvrez comment [Gérer les clés de compte de stockage avec Key Vault et l’interface de ligne de commande Azure](overview-storage-keys.md) ou [Azure PowerShell](overview-storage-keys-powershell.md).
- Consultez [Exemples de clés de compte de stockage managé](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=).