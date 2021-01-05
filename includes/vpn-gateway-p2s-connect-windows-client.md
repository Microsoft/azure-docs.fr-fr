---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/29/2020
ms.author: cherylmc
ms.openlocfilehash: 15c29648e42ba190991d51188489883e29bee165
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041530"
---
>[!NOTE]
>Vous devez disposer de droits d’administrateur sur l’ordinateur client Windows à partir duquel vous vous connectez.
>

1. Pour vous connecter à votre réseau virtuel, sur l’ordinateur client, accédez aux paramètres VPN et recherchez la connexion VPN que vous avez créée. Elle porte le même nom que votre réseau virtuel. Sélectionnez **Connecter**. Un message contextuel faisant référence à l’utilisation du certificat peut s’afficher. Sélectionnez **Continuer** pour utiliser des privilèges élevés.

1. Sur la page d'état **Connexion** , sélectionnez **Connecter** pour établir la connexion. Si un écran **Sélectionner un certificat** apparaît, vérifiez que le certificat client affiché est celui que vous souhaitez utiliser pour la connexion. Dans le cas contraire, utilisez la flèche déroulante pour sélectionner le certificat approprié, puis sélectionnez **OK**.

   :::image type="content" source="./media/vpn-gateway-p2s-connect-windows-client/connection-status.png" alt-text="Se connecter à partir d’un client VPN Windows":::

1. Votre connexion est établie.

   :::image type="content" source="./media/vpn-gateway-p2s-connect-windows-client/connected.png" alt-text="Diagramme de connexion point à site d’un ordinateur à un réseau virtuel Azure":::
