---
title: 'Passerelle VPN Azure : Installer un certificat client point à site'
description: Installer un certificat client pour une authentification de certificat P2S - Windows, Mac, Linux.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 8c4d37f67d89ad1b7973a85fb6c2a9f7e818a028
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89421660"
---
# <a name="install-client-certificates-for-p2s-certificate-authentication-connections"></a>Installer des certificats client pour des connexions d’authentification par certificat P2S

Tous les clients qui se connectent à un réseau virtuel à l’aide d’une connexion point à site avec authentification par certificat Azure nécessitent un certificat client. Cet article vous aide à installer un certificat client utilisé pour l’authentification à la connexion à réseau virtuel via une connexion P2S.

## <a name="acquire-a-client-certificate"></a><a name="generate"></a>Acquérir un certificat client

Quel que soit le système d’exploitation client à partir duquel vous souhaitez vous connecter, vous devez toujours avoir un certificat client. Vous pouvez générer un certificat client à partir d’un certificat racine généré à l’aide d’une solution d’autorité de certification d’entreprise, ou à partir d’un certificat racine auto-signé. Pour connaître les étapes de génération d’un certificat client, voir les instructions [PowerShell](vpn-gateway-certificates-point-to-site.md), [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) ou [Linux](vpn-gateway-certificates-point-to-site-linux.md). 

## <a name="windows"></a><a name="installwin"></a>Windows

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

## <a name="mac"></a><a name="installmac"></a>Mac

>[!NOTE]
>Les clients VPN Mac sont pris en charge avec le modèle de déploiement de Resource Manager seulement. Ils ne sont pas pris en charge avec le modèle de déploiement Classic.
>
>

[!INCLUDE [Install on Mac](../../includes/vpn-gateway-certificates-install-mac-client-cert-include.md)]

## <a name="linux"></a><a name="installlinux"></a>Linux

Le certificat client Linux est installé sur le client dans le cadre de la configuration du client. Pour obtenir des instructions, voir [Configuration client - Linux](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli).

## <a name="next-steps"></a>Étapes suivantes

Passez aux étapes de configuration point à site pour [Créer et installer les fichiers de configuration du client VPN](point-to-site-vpn-client-configuration-azure-cert.md).
