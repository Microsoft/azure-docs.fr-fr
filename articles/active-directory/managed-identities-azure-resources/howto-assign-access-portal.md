---
title: Attribuer à une identité managée l’accès à une ressource à l’aide du Portail Azure – Azure AD
description: Instructions détaillées pour attribuer à un identité managée sur une ressource un accès à une autre ressource, à l’aide du portail Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6584754edf3ff7ae31c3b9ace72baf16459dbc44
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359989"
---
# <a name="assign-a-managed-identity-access-to-a-resource-by-using-the-azure-portal"></a>Attribuer à une identité managée un accès à une ressource à l’aide du portail Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Après avoir configuré une ressource Azure avec une identité managée, vous pouvez accorder à cette dernière un accès à une autre ressource, tout comme n’importe quel principal de sécurité. Cet article montre comment accorder à l’identité managée d’une machine virtuelle ou d’un groupe de machines virtuelles identiques Azure un accès à un compte de stockage Azure, à l’aide du portail Azure.

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.

## <a name="use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource"></a>Utiliser Azure RBAC pour affecter à une identité managée l’accès à une autre ressource

Après avoir activé l’identité managée sur une ressource Azure, comme une [machine virtuelle Azure](qs-configure-portal-windows-vm.md) ou un [groupe de machines virtuelles identiques Azure](qs-configure-portal-windows-vmss.md) :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte associé à l’abonnement Azure sur lequel vous avez configuré l’identité managée.

2. Accédez à la ressource souhaitée sur laquelle vous voulez modifier le contrôle d’accès. Dans cet exemple, nous offrons à une machine virtuelle Azure l’accès à un compte de stockage auquel nous accédons.

3. Sélectionnez la page **Contrôle d’accès (IAM)** de la ressource, puis cliquez sur **+ Ajouter une attribution de rôle**. Spécifiez ensuite le **rôle** , la **destination de l’accès** et l’ **abonnement** correspondant. La ressource doit être située sous la zone de critères de recherche. Sélectionnez la ressource puis sélectionnez **Enregistrer**. 

   ![Capture d’écran du contrôle d’accès (IAM)](./media/msi-howto-assign-access-portal/assign-access-control-iam-blade-before.png)  
     
## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de l’identité managée pour les ressources Azure](overview.md)
- Pour activer l’identité managée sur une machine virtuelle Azure, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md).
- Pour activer l’identité managée sur un groupe de machines virtuelles identiques Azure, voir [Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant le portail Azure](qs-configure-portal-windows-vmss.md).


