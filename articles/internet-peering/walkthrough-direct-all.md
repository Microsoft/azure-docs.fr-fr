---
title: Procédure pas à pas pour le peering direct
titleSuffix: Azure
description: Procédure pas à pas pour le peering direct
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: 6a1fa7f582511a2089bab2694f9f5ec47634ff54
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97592379"
---
# <a name="direct-peering-walkthrough"></a>Procédure pas à pas pour le peering direct

Cette section explique les étapes à suivre pour configurer et gérer un Peering direct.

## <a name="create-a-direct-peering"></a>Créer un Peering direct
> [!div class="mx-imgBorder"]
> ![Workflow de Peering direct et états des connexions](./media/direct-peering.png)

Vous devez effectuer les étapes suivantes pour provisionner un Peering direct :
1. Consultez la [stratégie de Peering de Microsoft](https://peering.azurewebsites.net/peering) pour comprendre les exigences relatives au Peering direct.
1. Suivez les instructions fournies dans [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-powershell.md) pour soumettre une demande de Peering.
1. Une fois que vous aurez envoyé votre demande de Peering, Microsoft vous contactera par e-mail afin de vous fournir une lettre d’autorisation ou d’obtenir d’autres informations.
1. Une fois la demande de Peering approuvée, l’état de la connexion devient ProvisioningStarted.
1. Vous devez :
    1. Effectuer le câblage conformément à la lettre d’autorisation.
    1. (facultatif) Effectuer un test de liaison à l’aide de 169.254.0.0/16.
    1. Configurer la session BGP et nous en informer.
1. Microsoft provisionne une session BGP avec une stratégie DENY ALL et valide de bout en bout.
1. En cas de réussite, vous recevrez une notification indiquant que l’état de la connexion de Peering est « Active ».
1. Le trafic par le biais du nouveau Peering sera alors autorisé.

Notez qu’il ne faut pas confondre les états de connexion avec les états de session [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) standard.

## <a name="convert-a-legacy-direct-peering-to-azure-resource"></a>Convertir un peering direct existant en ressource Azure
Vous devez effectuer les étapes suivantes pour convertir un Peering direct hérité en ressource Azure :
1. Suivez les instructions de [Convertir un Peering direct hérité en ressource Azure](howto-legacy-direct-powershell.md).
1. Une fois que vous avez envoyé une demande de conversion, Microsoft examine la demande et vous contacte si nécessaire.
1. Une fois la demande approuvée, l’état de la connexion de Peering direct devient « Active ».

## <a name="deprovision-direct-peering"></a>Déprovisionner un Peering direct
Pour déprovisionner un Peering direct, Contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com).

Quand le déprovisionnement d’un Peering direct est prévu, l’état de la connexion devient **PendingRemove**.

> [!NOTE]
> Si vous exécutez l’applet de commande PowerShell pour supprimer le Peering direct quand l’état de la connexion est ProvisioningStarted ou ProvisioningCompleted, l’opération échoue.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur les [Prérequis pour configurer le Peering avec Microsoft](prerequisites.md).
