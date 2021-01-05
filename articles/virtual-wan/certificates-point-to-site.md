---
title: Génération et exportation de certificats pour les connexions VPN utilisateur | Azure Virtual WAN
description: Créez un certificat racine autosigné, exportez la clé publique, puis générez des certificats clients pour des connexions VPN utilisateur avec PowerShell sur Windows 10 ou Windows Server 2016.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: 2205f170ee846d4db94db7f524a1c424cfbc8f7b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91328036"
---
# <a name="generate-and-export-certificates-for-user-vpn-connections"></a>Génération et exportation de certificats pour les connexions VPN utilisateur

Les connexions (point à site) VPN utilisateur utilisent des certificats pour l’authentification. Cet article explique comment créer un certificat racine auto-signé et générer des certificats clients à l’aide de PowerShell sous Windows 10 ou Windows Server 2016.

Vous devez suivre les étapes de cet article sur un ordinateur exécutant Windows 10 ou Windows Server 2016. Les applets de commande PowerShell que vous utilisez pour générer des certificats font partie du système d’exploitation et ne fonctionnent pas sur d’autres versions de Windows. L’ordinateur Windows 10 ou Windows Server 2016 est nécessaire uniquement pour générer les certificats. Une fois les certificats générés, vous pouvez les charger ou les installer sur n’importe quel système d’exploitation client pris en charge.

[!INCLUDE [Export public key](../../includes/vpn-gateway-generate-export-certificates-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez aux [étapes Virtual WAN pour la connexion VPN utilisateur](virtual-wan-about.md)
