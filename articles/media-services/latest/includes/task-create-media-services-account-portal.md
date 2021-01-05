---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 11/04/2020
ms.author: inhenkel
ms.custom: portal
ms.openlocfilehash: 72857564a6b195353e7bf2a27c8369004d52d3f2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95971358"
---
<!-- Use the portal to create a media services account. -->

## <a name="create-a-media-services-account"></a>Créer un compte Media Services

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Cliquez sur **+ Créer une ressource** > **Média** > **Media Services**.
1. Dans la section **Créer un compte Media Services**, entrez les valeurs requises.

    | Nom | Description |
    | ---|---|
    |**Nom du compte**|Entrez le nom du nouveau compte Media Services. Un nom de compte Media Services se compose de lettres en minuscules ou de chiffres, sans espaces. Sa longueur est comprise entre 3 et 24 caractères.|
    |**Abonnement**|Si vous disposez de plusieurs abonnements, sélectionnez-en un dans la liste des abonnements Azure auxquels vous avez accès.|
    |**Groupe de ressources**|Sélectionnez la ressource (nouvelle ou existante). Un groupe de ressources désigne une collection de ressources qui partagent un cycle de vie, des autorisations et des stratégies. En savoir plus [ici](../../../azure-resource-manager/management/overview.md#resource-groups).|
    |**Lieu**|Sélectionnez la zone géographique à utiliser pour stocker les enregistrements multimédias et les métadonnées de votre compte Media Services. Cette région servira à traiter et diffuser vos médias. Seules les régions Media Services disponibles s’affichent dans la liste déroulante. |
    |**Compte de stockage**|Sélectionnez le compte de stockage qui fournira le stockage d'objets blob du contenu multimédia provenant de votre compte Media Services. Vous pouvez sélectionner un compte de stockage existant dans la même région géographique que votre compte Media Services ou en créer un. Ce dernier sera créé dans la même région. Les règles des noms de compte de stockage sont identiques à celles des comptes Media Services.<br/><br/>Vous devez disposer d’un compte de stockage **principal** et vous pouvez avoir n’importe quel nombre de comptes de stockage **secondaires** associés à votre compte Media Services. Vous pouvez utiliser le portail Azure pour ajouter des comptes de stockage secondaires. Pour plus d'informations, consultez [Comptes de stockage Azure avec comptes Azure Media Services](../storage-account-concept.md).<br/><br/>Le compte Media Services et tous les comptes de stockage associés doivent être faire partie du même abonnement Azure. Il est fortement recommandé d’utiliser des comptes de stockage situés au même emplacement que le compte Media Services afin d’éviter des frais supplémentaires de sortie des données et une importante latence.|
    |**Paramètres avancés**| Vous pouvez créer un compte à l’aide d’une identité managée par le système en sélectionnant la case d’option **Managée par le système**.  Cette sélection vous permet d’utiliser des clés managées par le client, ou d’apporter votre propre clé (BYOK) et Media Services pour activer le stockage approuvé.  Pour plus d’informations sur les clés managées par le client, consultez [Utiliser des clés BYOK (clé managées par le client) avec Media Services](../concept-use-customer-managed-keys-byok.md). De plus, [les identités managées](../concept-managed-identities.md) seront également activées.

1. Sélectionnez **Épingler au tableau de bord** pour voir la progression du déploiement du compte.
1. Cliquez sur **Créer** en bas du formulaire.
