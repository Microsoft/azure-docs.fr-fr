---
title: Procédure pas à pas pour le peering Exchange
titleSuffix: Azure
description: Procédure pas à pas pour le peering Exchange
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: 97430d86cba1107ef67264a181925d2a4d7c12c3
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586820"
---
# <a name="exchange-peering-walkthrough"></a>Procédure pas à pas pour le peering Exchange

Cette section explique les étapes à suivre pour configurer et gérer un Peering Exchange.

## <a name="create-an-exchange-peering"></a>Créer un Peering Exchange
> [!div class="mx-imgBorder"]
> ![Workflow de Peering Exchange et états des connexions](./media/exchange-peering.png)

Vous devez effectuer les étapes suivantes pour configurer un Peering Exchange :
1. Consultez la [stratégie de Peering de Microsoft](https://peering.azurewebsites.net/peering) pour comprendre les exigences relatives au Peering Exchange.
1. Recherchez l’emplacement de Peering Microsoft et l’ID d’installation de Peering dans [PeeringDB](https://www.peeringdb.com/net/694).
1. Demandez le Peering Exchange pour un emplacement de Peering en suivant les instructions fournies dans l’article [Créer et modifier un Peering Exchange à l’aide de PowerShell](howto-exchange-powershell.md).
1. Une fois que vous avez envoyé une demande de Peering, Microsoft examine la demande et vous contacte si nécessaire.
1. Une fois la demande approuvée, l’état de la connexion devient « Approved ».
1. Configurez la session BGP à votre extrémité et informez Microsoft.
1. Nous provisionnerons la session BGP avec une stratégie DENY ALL et validerons de bout en bout.
1. En cas de réussite, vous recevrez une notification indiquant que l’état de la connexion de Peering est « Active ».
1. Le trafic par le biais du nouveau Peering sera alors autorisé.

Notez qu’il ne faut pas confondre les états de connexion avec les états de session [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) standard.

## <a name="convert-a-legacy-exchange-peering-to-azure-resource"></a>Convertir un peering Exchange existant en ressource Azure
Vous devez effectuer les étapes suivantes pour convertir un Peering Exchange hérité en ressource Azure :
1. Suivez les instructions de [Convertir un Peering Exchange hérité en ressource Azure](howto-legacy-exchange-powershell.md).
1. Une fois que vous avez envoyé une demande de conversion, Microsoft examine la demande et vous contacte si nécessaire.
1. Une fois la demande approuvée, l’état de la connexion de Peering Exchange devient « Active ».

## <a name="deprovision-exchange-peering"></a>Annuler le provisionnement du Peering Exchange
Pour annuler le provisionnement du Peering Exchange, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com).

Quand l’annulation du provisionnement d’un Peering Exchange est prévue, l’état de la connexion devient **PendingRemove**.

> [!NOTE]
> Si vous exécutez l’applet de commande PowerShell pour supprimer le Peering Exchange quand l’état de la connexion est ProvisioningStarted ou ProvisioningCompleted, l’opération échoue.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur les [Prérequis pour configurer le Peering avec Microsoft](prerequisites.md).
