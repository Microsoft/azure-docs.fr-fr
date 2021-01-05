---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/28/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 59d18c89a8f66f781f01d1dace9d8968df17a626
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041589"
---
Obtenez le fichier .cer pour le certificat racine. Vous pouvez utiliser un certificat racine qui a été généré à l’aide d’une solution d’entreprise (recommandé), ou générer un certificat auto-signé. Après avoir créé le certificat racine, exportez les données de certificat public (et non la clé privée) en tant que fichier .cer X.509 encodé en Base64. Vous téléchargez ce fichier plus tard dans Azure.

* **Certificat d’entreprise :** Si vous utilisez une solution d’entreprise, vous pouvez utiliser votre chaîne d’approbation existante. Obtenez le fichier .cer pour le certificat racine que vous souhaitez utiliser.
* **Certificat racine auto-signé :** Si vous n’utilisez pas de solution de certificat d’entreprise, créez un certificat racine auto-signé. Dans le cas contraire, les certificats que vous créez ne seront pas compatibles avec vos connexions P2S, et les clients recevront une erreur de connexion lorsqu’ils essaieront de se connecter. Vous pouvez utiliser Azure PowerShell, MakeCert ou OpenSSL. Les procédures décrites dans les articles suivants expliquent comment générer un certificat racine auto-signé compatible :

  * [Instructions pour PowerShell sur Windows 10](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md) : ces instructions requièrent Windows 10 et PowerShell pour générer des certificats. Les certificats clients qui sont générés à partir du certificat racine peuvent être installés sur n’importe quel client P2S pris en charge.
  * [Instructions pour MakeCert](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md) : si vous n’avez pas accès à un ordinateur Windows 10, utilisez MakeCert pour générer des certificats. Même si MakeCert est déconseillé, vous pouvez toujours l’utiliser pour générer des certificats. Les certificats clients que vous générez à partir du certificat racine peuvent être installés sur n’importe quel client P2S pris en charge.
  * [Instructions Linux](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-linux.md).