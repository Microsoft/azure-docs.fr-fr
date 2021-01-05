---
title: Fichier include
description: Fichier Include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/06/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 9caf63fc90be7bae0461ddc24c94594a32199765
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91812702"
---
#### <a name="microsoft-windows"></a>Microsoft Windows

##### <a name="openvpn"></a>OpenVPN

1. Téléchargez et installez le client OpenVPN depuis le site web officiel.
1. Téléchargez le profil VPN pour la passerelle. Pour cela, accédez à l’onglet Configurations de VPN utilisateur dans le portail Azure, ou saisissez New-AzureRmVpnClientConfiguration dans PowerShell.
1. Décompressez le profil. Utilisez le bloc-notes pour ouvrir le fichier de configuration vpnconfig.ovpn à partir du dossier OpenVPN.
1. Renseignez la section du certificat client P2S avec la clé publique du certificat client P2S en base64. Dans un certificat au format PEM, vous pouvez ouvrir le fichier .cer et copier la clé base64 entre les en-têtes de certificat. Pour connaître les étapes à suivre, consultez [Guide pratique pour exporter un certificat afin d’obtenir la clé publique encodée](../articles/virtual-wan/certificates-point-to-site.md).
1. Renseignez la section de la clé privée avec la clé privée du certificat client P2S en base64. Pour connaître les étapes à suivre, consultez [Guide pratique pour extraire la clé privée](../articles/virtual-wan/howto-openvpn-clients.md#windows).
1. Ne modifiez aucun autre champ. Utilisez la configuration complétée dans l’entrée client pour vous connecter au VPN.
1. Copiez le fichier vpnconfig.ovpn dans C:\Program Files\OpenVPN\config folder.
1. Cliquez avec le bouton droit de la souris sur l’icône OpenVPN dans la barre d’état système et sélectionnez **Connexion**.

##### <a name="ikev2"></a>IKEv2

1. Sélectionnez les fichiers de configuration du client VPN qui correspondent à l’architecture de l’ordinateur Windows. Pour une architecture de processeur 64 bits, choisissez le package du programme d’installation « VpnClientSetupAmd64 ». Pour une architecture de processeur 32 bits, choisissez le package du programme d’installation « VpnClientSetupX86 ».
1. Double-cliquez sur le package pour lancer l’installation. Si une fenêtre contextuelle SmartScreen s’affiche, sélectionnez **Plus d’infos**, puis **Exécuter quand même**.
1. Sur l’ordinateur client, accédez à **Paramètres réseau**, puis sélectionnez **VPN**. La connexion VPN indique le nom du réseau virtuel auquel elle se connecte.
1. Avant de tenter de vous connecter, vérifiez que vous avez installé un certificat client sur l’ordinateur client. Un certificat client est requis pour l’authentification lors de l’utilisation du type d’authentification de certificat Azure natif. Pour plus d’informations sur la génération de certificats, consultez [Générer des certificats](../articles/virtual-wan/certificates-point-to-site.md). Pour plus d’informations sur l’installation d’un certificat client, consultez [Installer un certificat client](../articles/vpn-gateway/point-to-site-how-to-vpn-client-install-azure-cert.md).
