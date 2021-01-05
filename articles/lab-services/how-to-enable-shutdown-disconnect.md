---
title: Configurer l’arrêt automatique des machines virtuelles pour un labo dans Azure Lab Services
description: Découvrez comment activer ou désactiver l’arrêt automatique des machines virtuelles lorsqu'une connexion Bureau à distance est déconnectée.
ms.topic: article
ms.date: 10/01/2020
ms.openlocfilehash: 8f9080f3b7b762d3b9fa448a903a4167cd2cec4a
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433933"
---
# <a name="configure-automatic-shutdown-of-vms-for-a-lab"></a>Configurer l’arrêt automatique des machines virtuelles pour un labo

Cet article explique comment configurer l’arrêt automatique des machines virtuelles pour un labo.

Vous pouvez activer plusieurs fonctionnalités de contrôle des coûts d’arrêt automatique afin d’éviter de manière proactive les coûts supplémentaires lorsque les machines virtuelles ne sont pas utilisées activement. La combinaison des trois fonctionnalités suivantes d’arrêt et de déconnexion automatiques permet d’intercepter la plupart des cas où les utilisateurs laissent accidentellement leurs machines virtuelles en marche :
 
* Déconnectez automatiquement les utilisateurs des machines virtuelles que le système d’exploitation juge inactives.
* Arrêt automatique des machines virtuelles quand les utilisateurs se déconnectent.
* Arrêter automatiquement les machines virtuelles démarrées, mais auxquelles les utilisateurs ne se connectent pas.

Pour plus d’informations sur les fonctionnalités d’arrêt automatique, consultez la section [Optimiser le contrôle des coûts à l’aide des paramètres d’arrêt automatique](cost-management-guide.md#automatic-shutdown-settings-for-cost-control).

Un administrateur de compte lab peut configurer ce paramètre pour le compte lab dans lequel vous créez des labs. Pour plus d’informations, consultez [Configurer l’arrêt automatique des machines virtuelles pour un compte lab](how-to-configure-lab-accounts.md). En tant que propriétaire de lab, vous pouvez remplacer le paramètre lors de la création d’un lab ou après la création de celui-ci. 

## <a name="configure-for-the-lab-level"></a>Configurer pour le niveau du laboratoire

Vous pouvez configurer le paramètre d’arrêt automatique dans [Azure Lab Services](https://labs.azure.com/).

* lors de la création d’un laboratoire (dans **Stratégies de laboratoire**) ou
* une fois le laboratoire créé (dans **Paramètres**)

> [!div class="mx-imgBorder"]
> ![Effectuer la configuration au moment de la création du lab](./media/how-to-enable-shutdown-disconnect/configure-lab-creation.png)

Veillez à consulter les détails de l’arrêt automatique dans la section [Optimiser le contrôle des coûts avec les paramètres d’arrêt automatique](cost-management-guide.md#automatic-shutdown-settings-for-cost-control).

> [!WARNING]
> Si vous arrêtez le système d’exploitation Linux ou Windows d’une machine virtuelle avant de vous déconnecter d’une session RDP sur cette machine virtuelle, la fonctionnalité d’arrêt automatique ne marche pas correctement.  
## <a name="next-steps"></a>Étapes suivantes

[Tableau de bord des labos](use-dashboard.md)
