---
title: 'Générer et exporter des certificats pour point à site : Linux : Interface de ligne de commande'
description: Créez un certificat racine auto-signé, exportez la clé publique et générez des certificats clients à l’aide de l’interface de ligne de commande Linux (strongSwan).
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: alzam
ms.openlocfilehash: 0be5bb042649b0fe425077b5b3feb3cea728218c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89394004"
---
# <a name="generate-and-export-certificates"></a>Générer et exporter des certificats

Les connexions de point à site utilisent des certificats pour l’authentification. Cet article vous explique comment créer un certificat racine auto-signé et générer des certificats clients à l’aide de l’interface de ligne de commande Linux et de strongSwan. Pour des instructions sur d’autres certificats, voir les articles [PowerShell](vpn-gateway-certificates-point-to-site.md) ou [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Pour plus d’informations sur l’installation de strongSwan à l’aide de l’interface utilisateur graphique au lieu de l’interface de ligne de commande, consultez les étapes de l’article [Configuration du client](point-to-site-vpn-client-configuration-azure-cert.md#install).

## <a name="install-strongswan"></a>Installer strongSwan

[!INCLUDE [strongSwan Install](../../includes/vpn-gateway-strongswan-install-include.md)]

## <a name="generate-and-export-certificates"></a>Générer et exporter des certificats

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à votre configuration point à site pour [Créer et installer les fichiers de configuration du client VPN](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli).
