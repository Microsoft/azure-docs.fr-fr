---
title: Vue d’ensemble du contrôle de maintenance pour les mises à niveau d’images de système d’exploitation sur les groupes de machines virtuelles identiques Azure
description: Découvrez comment contrôler le moment où les mises à niveau automatiques des images de système d’exploitation sont déployées sur vos groupes de machines virtuelles identiques Azure à l’aide du contrôle de maintenance.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: jushiman
ms.openlocfilehash: 55cbc5db534dd1b05f91a24e0104b1f2dc110547
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532931"
---
# <a name="preview-maintenance-control-for-azure-virtual-machine-scale-sets"></a>Aperçu : Contrôle maintenance pour les groupes de machines virtuelles identiques Azure 

Gérez les [mises à niveau automatiques des images de système d’exploitation](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) pour vos groupes de machines virtuelles identiques à l’aide du contrôle de maintenance.

Le contrôle de maintenance vous permet de décider quand appliquer des mises à jour aux disques de système d’exploitation dans vos groupes de machines virtuelles identiques, avec une expérience plus facile et plus prévisible. 

Les configurations de maintenance fonctionnent entre les abonnements et les groupes de ressources.

Le workflow complet se résume aux étapes suivantes : 
- Créez une configuration de maintenance.
- Associez un groupe de machines virtuelles identiques à une configuration de maintenance.
- Activez les mises à niveau automatiques des systèmes d’exploitation.

> [!IMPORTANT]
> Le contrôle de maintenance des mises à niveau d’images de système d’exploitation sur les groupes de machines virtuelles identiques Azure est actuellement disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="limitations"></a>Limites

- Les machines virtuelles doivent se trouver dans un groupe identique.
- L’utilisateur doit disposer d’un accès **Contributeur de ressource**.
- La durée de la maintenance doit être de 5 heures ou plus dans la configuration de la maintenance.
- La périodicité de la maintenance doit être définie sur « jour » dans la configuration de la maintenance


## <a name="management-options"></a>Options de gestion

Vous pouvez créer et gérer des configurations de maintenance à l’aide de l’une des options suivantes :

- [Azure PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment contrôler le moment où la maintenance est appliquée à vos groupes de machines virtuelles identiques Azure à l’aide du contrôle de maintenance et de PowerShell.

> [!div class="nextstepaction"]
> [Contrôle de maintenance d’un groupe de machines virtuelles identiques avec PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)
