---
title: Activer Azure Peering Service sur un Peering direct en utilisant PowerShell
titleSuffix: Azure
description: Activer Azure Peering Service sur un Peering direct en utilisant PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 98341fbbbcafb6aee938870c22050c6edec352ac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89079045"
---
# <a name="enable-azure-peering-service-on-a-direct-peering-by-using-powershell"></a>Activer Azure Peering Service sur un Peering direct en utilisant PowerShell

Cet article explique comment activer [Azure Peering Service](overview-peering-service.md) sur un Peering direct à l’aide de cmdlets PowerShell et du modèle de déploiement Azure Resource Manager.

Si vous préférez, vous pouvez suivre ce guide en utilisant le [Portail Azure](howto-peering-service-portal.md).

## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) avant de commencer la configuration.
* Choisissez un Peering direct dans votre abonnement sur lequel vous souhaitez activer Peering Service. Si vous n’en avez pas, vous pouvez soit convertir un Peering direct hérité, soit créer un nouveau Peering direct :
    * Pour convertir un Peering direct hérité, suivez les instructions fournies dans [Convertir un Peering direct hérité en ressource Azure à l’aide de PowerShell](howto-legacy-direct-powershell.md).
    * Pour créer un Peering direct, suivez les instructions fournies dans [Créer ou modifier un Peering direct à l’aide de PowerShell](howto-direct-powershell.md).

### <a name="work-with-azure-powershell"></a>Utiliser Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="enable-peering-service-on-a-direct-peering"></a>Activer Peering Service sur un peering direct

### <a name="view-direct-peering"></a><a name= get></a>Afficher un Peering direct
[!INCLUDE [peering-direct-get](./includes/direct-powershell-get.md)]

### <a name="enable-the-direct-peering-for-peering-service"></a><a name= get></a>Activer le Peering direct pour Peering Service

Après avoir obtenu le Peering direct à l’étape précédente, activez-le pour Peering Service.
[!INCLUDE [peering-direct-modify](./includes/peering-service-direct-powershell.md)]

## <a name="modify-a-direct-peering-connection"></a>Modifier une connexion de Peering direct

Si vous avez besoin de modifier les paramètres de connexion, consultez la section « Modifier un Peering direct » dans [Créer ou modifier un Peering direct à l’aide de PowerShell](howto-direct-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering Exchange à l’aide de PowerShell](howto-exchange-powershell.md)
* [Convertir un Peering Exchange hérité en ressource Azure à l’aide de PowerShell](howto-legacy-exchange-powershell.md)

## <a name="additional-resources"></a>Ressources supplémentaires
Vous pouvez obtenir une description détaillée de tous les paramètres en exécutant la commande suivante :

```powershell
Get-Help Get-AzPeering -detailed
```

Pour obtenir des réponses aux questions les plus fréquentes, consultez le [FAQ sur Peering Service](service-faqs.md).
