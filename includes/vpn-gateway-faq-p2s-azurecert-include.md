---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 08/14/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 5ef67580928a45609f50d3fe798eb9d054265c0a
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93376009"
---
[!INCLUDE [P2S FAQ All](vpn-gateway-faq-p2s-all-include.md)]

### <a name="what-should-i-do-if-im-getting-a-certificate-mismatch-when-connecting-using-certificate-authentication"></a>Que dois-je faire si j’obtiens une incompatibilité de certificat lors de la connexion à l’aide de l’authentification par certificat ?

Décochez **« Vérifier l’identité du serveur en validant le certificat »** ou **ajouter le nom de domaine complet du serveur avec le certificat** lors de la création manuelle d’un profil. Pour ce faire, exécutez **rasphone** à partir d’une invite de commandes et choisissez le profil dans la liste déroulante.

Ignorer la validation de l’identité du serveur n’est pas recommandé en général, mais avec l’authentification par certificat Azure, le même certificat est utilisé pour la validation du serveur dans le protocole de tunneling VPN (IKEv2/SSTP) et le protocole EAP. Étant donné que le certificat de serveur et le nom de domaine complet sont déjà validés par le protocole de tunneling VPN, il est redondant de les valider de nouveau dans EAP.

![authentification point à site](./media/vpn-gateway-faq-p2s-all-include/servercert.png "Certificat de serveur")

### <a name="can-i-use-my-own-internal-pki-root-ca-to-generate-certificates-for-point-to-site-connectivity"></a>Puis-je utiliser ma propre AC racine PKI interne pour générer des certificats pour une connectivité point à site ?

Oui. Auparavant, seuls les certificats racines auto-signés pouvaient être utilisés. Vous pouvez toujours charger 20 certificats racine.

### <a name="can-i-use-certificates-from-azure-key-vault"></a>Puis-je utiliser des certificats Azure Key Vault ?

Non.

### <a name="what-tools-can-i-use-to-create-certificates"></a>Quels outils puis-je utiliser pour créer des certificats ?

Vous pouvez utiliser votre solution de PKI d’entreprise (votre PKI interne), Azure PowerShell, MakeCert et OpenSSL.

### <a name="are-there-instructions-for-certificate-settings-and-parameters"></a><a name="certsettings"></a>Y a-t-il des instructions pour les paramètres de certificat ?

* **PKI interne/Solution PKI d’entreprise :** reportez-vous aux étapes de [génération des certificats](../articles/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#generatecert).

* **Azure PowerShell :** consultez l’article [Azure PowerShell](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md) pour connaître la procédure.

* **MakeCert :** consultez l’article [MakeCert](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md) pour connaître la procédure.

* **OpenSSL :** 

    * Lors de l’exportation de certificats, veillez à convertir le certificat racine en Base64.

    * Pour le certificat client :

      * Lorsque vous créez la clé privée, spécifiez la longueur 4096.
      * Lors de la création du certificat, pour le paramètre *-extensions* , spécifiez *usr_cert*.
