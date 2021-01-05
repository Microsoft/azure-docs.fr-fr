---
title: Activer Azure Peering Service sur un peering direct en utilisant le portail Azure
titleSuffix: Azure
description: Activer Azure Peering Service sur un peering direct en utilisant le portail Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 3/18/2020
ms.author: derekol
ms.openlocfilehash: a52e6038b622c004dc0d133394cd4f53600b2935
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84700040"
---
# <a name="enable-azure-peering-service-on-a-direct-peering-by-using-the-azure-portal"></a>Activer Azure Peering Service sur un peering direct en utilisant le portail Azure

Cet article explique comment activer Azure [Peering Service](overview-peering-service.md) sur un Peering direct en utilisant le portail Azure.

Si vous préférez, vous pouvez suivre ce guide à l’aide de [PowerShell](howto-peering-service-powershell.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) avant de commencer la configuration.
* Choisissez un Peering direct dans votre abonnement sur lequel vous souhaitez activer Peering Service. Si vous n’en avez pas, vous pouvez soit convertir un Peering direct hérité, soit créer un nouveau Peering direct :
    * Pour convertir un Peering direct hérité, suivez les instructions fournies dans [Convertir un Peering direct existant en ressource Azure à l’aide du portail](howto-legacy-direct-portal.md).
    * Pour créer un Peering direct, suivez les instructions fournies dans [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-portal.md).

## <a name="enable-peering-service-on-a-direct-peering"></a>Activer Peering Service sur un peering direct

### <a name="view-direct-peering"></a><a name= get></a>Afficher un Peering direct
[!INCLUDE [peering-direct-get-portal](./includes/direct-portal-get.md)]

### <a name="enable-the-direct-peering-for-peering-service"></a><a name= get></a>Activer le Peering direct pour Peering Service

Après avoir ouvert le Peering direct à l’étape précédente, activez-le pour Peering Service.
[!INCLUDE [peering-direct-modify](./includes/peering-service-direct-portal.md)]

## <a name="modify-a-direct-peering-connection"></a>Modifier une connexion de Peering direct

Pour modifier les paramètres de connexion, consultez la section « Modifier un Peering direct  » dans [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-portal.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering Exchange à l’aide du portail](howto-exchange-portal.md)
* [Convertir un Peering Exchange hérité en ressource Azure à l’aide du portail](howto-legacy-exchange-portal.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir des réponses aux questions les plus fréquentes, consultez [FAQ sur Peering Service](service-faqs.md).