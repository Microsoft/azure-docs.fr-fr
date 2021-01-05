---
title: Utiliser l’interface de ligne de commande Azure pour créer une application Azure AD et la configurer pour accéder à l’API Azure Media Services | Microsoft Docs
description: Cette rubrique montre comment utiliser l’interface de ligne de commande Azure pour créer une application Azure AD et la configurer pour accéder à l’API d’Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/26/2019
ms.author: juliako
ms.openlocfilehash: 12d6beac446f8a15a41daf0dfe8ca1783c95c775
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021110"
---
# <a name="use-azure-cli-to-create-an-azure-ad-app-and-configure-it-to-access-media-services-api"></a>Utiliser l'interface de ligne de commande Azure pour créer une application Azure AD et configurer celle-ci pour accéder à l'API Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Cette rubrique vous montre comment utiliser l’interface de ligne de commande Azure pour créer une application Azure Active Directory (Azure AD) et un principal de service pour accéder aux ressources Azure Media Services. 

## <a name="prerequisites"></a>Conditions préalables requises

- Un compte Azure. Pour plus d’informations, consultez la page [Version d’évaluation gratuite d’Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Un compte Media Services. Pour plus d’informations, voir [Création d’un compte Azure Media Services à l’aide du portail Azure](media-services-portal-create-account.md).

## <a name="use-the-azure-cloud-shell"></a>Utiliser Azure Cloud Shell

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Lancez Cloud Shell à partir du volet de navigation supérieur du portail.

    ![Cloud Shell](./media/media-services-cli-create-and-configure-aad-app/media-services-cli-create-and-configure-aad-app01.png) 

Pour plus d’informations, voir la [présentation d’Azure Cloud Shell](../../cloud-shell/overview.md).

## <a name="create-an-azure-ad-app-and-configure-access-to-the-media-account-with-azure-cli"></a>Créer une application Azure AD et configurer l’accès au compte multimédia avec l’interface de ligne de commande Azure
 
```azurecli
az login
az ad sp create-for-rbac --name <appName> 
az role assignment create --assignee < user/app id> --role Contributor --scope <subscription/subscription id>
```

Par exemple :

```azurecli
az role assignment create --assignee a3e068fa-f739-44e5-ba4d-ad57866e25a1 --role Contributor --scope /subscriptions/0b65e280-7917-4874-9fed-1307f2615ea2/resourceGroups/Default-AzureBatch-SouthCentralUS/providers/microsoft.media/mediaservices/sbbash
```

Dans cet exemple, l’**étendue** est le chemin d’accès complet de la ressource pour le compte de services multimédia. Toutefois, l’**étendue** peut être à n’importe quel niveau.

Par exemple, il peut s’agir d’un des niveaux suivants :
 
* Niveau d'**abonnement**.
* Niveau de **groupe de ressources**.
* Niveau de **ressource** (par exemple, compte multimédia).

Pour plus d’informations, consultez [Créez un principal du service avec Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli).

Consultez également [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../../role-based-access-control/role-assignments-cli.md). 

## <a name="next-steps"></a>Étapes suivantes

Commencez le [chargement de fichiers sur votre compte](media-services-portal-upload-files.md).
