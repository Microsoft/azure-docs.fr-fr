---
title: Fichier include
titleSuffix: Azure
description: Fichier include
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: beffb2babefd86c2807e21e9337cba66f42fcfc2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81678485"
---
Avant de commencer la configuration, installez et importez les modules requis. Vous avez besoin des privilèges d’administrateur pour installer les modules dans PowerShell.

1. Installez et importez le module Az.
    ```powershell
    Install-Module Az -AllowClobber
    Import-Module Az
    ```
1. Installez et importez le module Az.Peering.
    ```powershell
    Install-Module -Name Az.Peering -AllowClobber
    Import-Module Az.Peering
    ```
1. Vérifiez que les modules ont été importés correctement à l’aide de cette commande :
    ```powershell
    Get-Module
    ```
1. Connectez-vous à votre compte Azure à l’aide de cette commande :
    ```powershell
    Connect-AzAccount
    ```
1. Vérifiez les abonnements pour le compte et sélectionnez l’abonnement dans lequel vous souhaitez créer un Peering.
    ```powershell
    Get-AzSubscription
    Select-AzSubscription -SubscriptionId "subscription-id"
    ```
1. Si vous n’avez pas déjà un groupe de ressources, vous devez en créer un avant de créer un peering. Pour cela, vous pouvez exécuter la commande suivante :

    ```powershell
    New-AzResourceGroup -Name "PeeringResourceGroup" -Location "Central US"
    ```
> [!IMPORTANT]
> Si vous n’avez pas encore associé votre ASN et votre abonnement, suivez les étapes de la procédure [Associer le numéro ASN de l’homologue](../howto-subscription-association-powershell.md). Cette action est requise pour demander un Peering.

> [!NOTE]
> L’emplacement d’un groupe de ressources est indépendant de l’emplacement où vous choisissez de configurer un Peering.
&nbsp;
