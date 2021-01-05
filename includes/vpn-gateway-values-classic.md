---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/08/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 11d2172d085fe9b47587f4084908f99d7b54437e
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103235"
---
Lorsque vous créez des réseaux virtuels classiques dans le portail Azure, le nom que vous voyez ne correspond pas au nom complet que vous utilisez pour PowerShell. Par exemple, un réseau virtuel qui semble être nommé **TestVNet1** dans le portail Azure peut avoir un nom beaucoup plus long dans le fichier de configuration réseau. Pour un réseau virtuel dans le groupe de ressources « ClassicRG », le nom peut ressembler à ceci : **Groupe ClassicRG TestVNet1** . Lorsque vous créez vos connexions, il est important d’utiliser les valeurs que vous voyez dans le fichier de configuration réseau.

Dans les étapes suivantes, vous allez vous connecter à votre compte Azure et télécharger et afficher le fichier de configuration réseau pour obtenir les valeurs nécessaire pour établir les connexions.

1. Téléchargez et installez la dernière version des applets de commande PowerShell Azure Service Management (SM). La plupart des utilisateurs disposent des modules Resource Manager installés localement, mais pas des modules de gestion des services. Les modules de gestion des services sont hérités et doivent être installés séparément. Pour plus d’informations, consultez [Installer des applets de commande de gestion des services](/powershell/azure/servicemanagement/install-azure-ps).

1. Ouvrez la console PowerShell avec des droits élevés et connectez-vous à votre compte. Utilisez les exemples suivants pour faciliter votre connexion. Vous devez exécuter ces commandes localement à l’aide du module PowerShell Service Management. Se connecter à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

   ```powershell
   Add-AzureAccount
   ```
1. Vérifiez les abonnements associés au compte.

   ```powershell
   Get-AzureSubscription
   ```
1. Si vous avez plusieurs abonnements, sélectionnez celui que vous souhaitez utiliser.

   ```powershell
   Select-AzureSubscription -SubscriptionId "Replace_with_your_subscription_ID"
   ```
1. Créez un annuaire sur votre ordinateur. Par exemple, C:\AzureVNet
1. Exportez le fichier de configuration réseau dans le répertoire. Dans cet exemple, le fichier de configuration réseau est exporté vers **C:\AzureNet** .

   ```powershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
1. Dans un éditeur de texte, ouvrez le fichier, puis affichez les noms de vos réseaux virtuels et de vos sites. Ces noms sont ceux que vous utiliserez lorsque vous allez créer vos connexions.<br>Les noms des **réseaux virtuels** sont répertoriés comme suit : **VirtualNetworkSite name =**<br>Les noms des **sites** sont répertoriés comme suit : **LocalNetworkSiteRef name =**