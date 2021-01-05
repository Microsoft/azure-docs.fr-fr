---
title: 'Passerelle VPN Azure : Résoudre les problèmes de connexion point à site : Clients Mac OS X'
description: Découvrez comment résoudre les problèmes de connectivité point à site à partir de Mac OS X à l’aide du client VPN natif et d’IKEv2.
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/27/2018
ms.author: alzam
ms.openlocfilehash: be7b6349a7bf51a4fb99e3604a04f9d95403b74f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91359322"
---
# <a name="troubleshoot-point-to-site-vpn-connections-from-mac-os-x-vpn-clients"></a>Résoudre les problèmes de connexion VPN point à site à partir de clients Mac OS X

Cet article vous aidera à résoudre les problèmes de connectivité point à site à partir de Mac OS X à l’aide du client VPN natif et d’IKEv2. Très basique, le client VPN disponible dans Mac pour IKEv2 n’offre pas beaucoup de possibilités de personnalisation. Il y a seulement quatre paramètres à vérifier :

* Adresse du serveur
* Identifiant distant
* Identifiant local
* Paramètres d’authentification
* Version du système d’exploitation (10.11 ou ultérieure)


## <a name="troubleshoot-certificate-based-authentication"></a><a name="VPNClient"></a> Résoudre les problèmes d’authentification par certificat
1. Vérifiez les paramètres du client VPN. Accédez à la fenêtre **Réseau** en appuyant sur Commande + Maj, puis tapez « VPN » pour vérifier les paramètres du client VPN. Dans la liste, cliquez sur l’entrée VPN à examiner.

   ![Authentification par certificat IKEv2](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2cert1.jpg)
2. Vérifiez que l’**adresse du serveur** correspond au FQDN complet et inclut cloudapp.net.
3. L’**identifiant distant** doit être identique à l’adresse du serveur (FQDN de la passerelle).
4. L’**identifiant local** doit être identique à l’**objet** du certificat client.
5. Cliquez sur **Réglages d’authentification** pour ouvrir la page Réglages d’authentification.

   ![Capture d'écran représentant une boîte de dialogue Paramètres d'authentification dans laquelle Certificat est sélectionné.](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth2.jpg)
6. Vérifiez que **Certificat** est sélectionné dans la liste déroulante.
7. Cliquez sur le bouton **Sélectionner** et vérifiez que le certificat approprié est sélectionné. Cliquez sur **OK** pour enregistrer les éventuelles modifications apportées.

## <a name="troubleshoot-username-and-password-authentication"></a><a name="ikev2"></a>Résoudre les problèmes d’authentification par nom d’utilisateur et mot de passe

1. Vérifiez les paramètres du client VPN. Accédez à la fenêtre **Réseau** en appuyant sur Commande + Maj, puis tapez « VPN » pour vérifier les paramètres du client VPN. Dans la liste, cliquez sur l’entrée VPN à examiner.

   ![Nom d’utilisateur et mot de passe IKEv2](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2user3.jpg)
2. Vérifiez que l’**adresse du serveur** correspond au FQDN complet et inclut cloudapp.net.
3. L’**identifiant distant** doit être identique à l’adresse du serveur (FQDN de la passerelle).
4. L’**identifiant local** peut être vide.
5. Cliquez sur le bouton **Réglages d’authentification** et vérifiez que « Nom d’utilisateur » est sélectionné dans la liste déroulante.

   ![Capture d'écran représentant une boîte de dialogue Paramètres d'authentification dans laquelle Nom d'utilisateur est sélectionné.](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth4.png)
6. Vérifiez que les informations d’identification saisies sont correctes.

## <a name="additional-steps"></a><a name="additional"></a>Étapes supplémentaires

Si vous avez suivi les étapes précédentes et que tout est correctement configuré, téléchargez [Wireshark](https://www.wireshark.org/#download) et effectuez une capture de paquets.

1. Filtrez sur *isakmp* et examinez les paquets **IKE_SA**. Les détails de la proposition de l’association de sécurité doivent figurer sous **Payload: Security Association** (Charge utile : association de sécurité). 
2. Vérifiez que le client et le serveur présentent un jeu commun.

   ![Paquet](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/packet5.jpg) 
  
3. S’il n’existe aucune réponse du serveur sur les traces réseau, vérifiez que vous avez activé le protocole IKEv2 dans la page Configuration de la passerelle Azure sur le site web du Portail Azure.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir de l’aide supplémentaire, consultez [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
