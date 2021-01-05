---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/17/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 55fa01d100c60c6411774373428ff4bbd9a56822
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80986808"
---
## <a name="windows-clients"></a><a name="windows"></a>Clients Windows

1. Téléchargez et installez le client OpenVPN (version 2.4 ou plus) depuis le [site web OpenVPN](https://openvpn.net/index.php/open-source/downloads.html) officiel.
2. Téléchargez le profil VPN pour la passerelle. Pour y parvenir, accédez à l’onglet Configurations point à site dans le Portail Azure, ou saisissez « New-AzVpnClientConfiguration » dans PowerShell.
3. Décompressez le profil. Utilisez ensuite e bloc-notes pour ouvrir le fichier de configuration *vpnconfig.ovpn* à partir du dossier OpenVPN.
4. Exportez le certificat client point à site que vous avez créé et chargé dans votre configuration P2S sur la passerelle. Consultez les articles suivants :

   * Instructions sur la [passerelle VPN](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport)
   
   * Instructions sur [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#clientexport)
5. Procédez à l’extraction de la clé privée et de l’empreinte numérique en base 64 à partir de la *.pfx*. Pour ce faire, plusieurs options s’offrent à vous. Vous devez utiliser OpenSSL sur votre machine. Le fichier *profileinfo.txt* contient la clé privée et l’empreinte numérique pour l’autorité de certification et le certificat client. Assurez-vous d’utiliser l’empreinte numérique du certificat client.

   ```
   openssl pkcs12 -in "filename.pfx" -nodes -out "profileinfo.txt"
   ```
6. Ouvrez *profileinfo.txt* dans le bloc-notes. Pour obtenir l’empreinte numérique du certificat client (enfant), sélectionnez le texte (y compris et entre) « ---BEGIN CERTIFICATE--- » et « ---END CERTIFICATE--- » pour le certificat enfant et copiez-le. Vous pouvez identifier le certificat enfant en examinant la ligne subject=/.
7. Basculez vers le fichier *vpnconfig.ovpn* que vous avez ouvert dans le Bloc-notes à l’étape 3. Recherchez la section ci-dessous et remplacez tout le contenu compris entre « cert » et « / cert ».

   ```
   # P2S client certificate
   # please fill this field with a PEM formatted cert
   <cert>
   $CLIENTCERTIFICATE
   </cert>
   ```
8. Ouvrez *profileinfo.txt* dans le Bloc-notes. Pour obtenir la clé privée, sélectionnez le texte (y compris et entre) « ---BEGIN PRIVATE KEY--- » et « ---END PRIVATE KEY--- », puis copiez-le.
9. Revenez au fichier vpnconfig.ovpn dans le Bloc-notes et recherchez cette section. Collez la clé privée, en remplaçant tout le contenu compris entre « key » et « /key ».

   ```
   # P2S client root certificate private key
   # please fill this field with a PEM formatted key
   <key>
   $PRIVATEKEY
   </key>
   ```
10. Ne modifiez aucun autre champ. Utilisez la configuration complétée dans l’entrée client pour vous connecter au VPN.
11. Copiez le fichier vpnconfig.ovpn dans C:\Program Files\OpenVPN\config folder.
12. Cliquez avec le bouton droit de la souris sur l’icône OpenVPN de la barre d’état du système et cliquez sur l’option de connexion.

## <a name="mac-clients"></a><a name="mac"></a>Clients Mac

1. Télécharger et installer un client OpenVPN, comme [TunnelBlick](https://tunnelblick.net/downloads.html). 
2. Téléchargez le profil VPN pour la passerelle. Pour y parvenir, accédez à l’onglet Configurations point à site dans le Portail Azure, ou saisissez « New-AzVpnClientConfiguration » dans PowerShell.
3. Décompressez le profil. Ouvrez le fichier de configuration vpnconfig.ovpn à partir du dossier OpenVPN dans un éditeur de texte.
4. Renseignez la section du certificat client P2S avec la clé publique du certificat client P2S en base64. Dans un certificat au format PEM, vous pouvez simplement ouvrir le fichier .cer et copier la clé base64 entre les en-têtes de certificat. Consultez les articles suivants pour plus d’informations sur la façon d’exporter un certificat pour obtenir la clé publique codée :

   * Instructions sur la [passerelle VPN](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#cer) 
   
   * Instructions sur [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#cer)
5. Renseignez la section de la clé privée avec la clé privée du certificat client P2S en base64. Pour plus d’informations sur la façon d’extraire une clé privée, consultez [Exporter votre clé privée](https://openvpn.net/community-resources/how-to/#pki) sur le site OpenVPN.
6. Ne modifiez aucun autre champ. Utilisez la configuration complétée dans l’entrée client pour vous connecter au VPN.
7. Double-cliquez sur le fichier de profil pour créer le profil dans Tunnelblick.
8. Lancez Tunnelblick à partir du dossier d’applications.
9. Cliquez sur l’icône Tunnelblick dans la zone de notification et choisissez l’option de connexion.

> [!IMPORTANT]
>Seuls iOS 11.0 et versions ultérieures et MacOS 10.13 et versions ultérieures sont pris en charge avec le protocole OpenVPN.
>
## <a name="ios-clients"></a><a name="iOS"></a>Clients iOS

1. Installez le client OpenVPN (version 2.4 ou ultérieure) à partir de l’App Store.
2. Téléchargez le profil VPN pour la passerelle. Pour y parvenir, accédez à l’onglet Configurations point à site dans le Portail Azure, ou saisissez « New-AzVpnClientConfiguration » dans PowerShell.
3. Décompressez le profil. Ouvrez le fichier de configuration vpnconfig.ovpn à partir du dossier OpenVPN dans un éditeur de texte.
4. Renseignez la section du certificat client P2S avec la clé publique du certificat client P2S en base64. Dans un certificat au format PEM, vous pouvez simplement ouvrir le fichier .cer et copier la clé base64 entre les en-têtes de certificat. Consultez les articles suivants pour plus d’informations sur la façon d’exporter un certificat pour obtenir la clé publique codée :

   * Instructions sur la [passerelle VPN](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#cer) 
   
   * Instructions sur [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#cer)
5. Renseignez la section de la clé privée avec la clé privée du certificat client P2S en base64. Pour plus d’informations sur la façon d’extraire une clé privée, consultez [Exporter votre clé privée](https://openvpn.net/community-resources/how-to/#pki) sur le site OpenVPN.
6. Ne modifiez aucun autre champ.
7. Envoyez par e-mail le fichier de profil (.ovpn) à votre compte de messagerie configuré dans l’application de messagerie sur votre iPhone. 
8. Ouvrez l’e-mail dans l’application de messagerie sur l’iPhone, puis appuyez sur le fichier joint

    ![Ouvrir l’e-mail](./media/vpn-gateway-vwan-config-openvpn-clients/ios2.png)

9. Appuyez sur **Plus** si vous ne voyez pas l’option **Copier dans OpenVPN**

    ![En savoir plus](./media/vpn-gateway-vwan-config-openvpn-clients/ios3.png)

10. Appuyez sur **Copier dans OpenVPN** 

    ![Copier dans OpenVPN](./media/vpn-gateway-vwan-config-openvpn-clients/ios4.png)

11. Appuyez sur **AJOUTER** dans la page **Importer un profil**.

    ![Ajouter](./media/vpn-gateway-vwan-config-openvpn-clients/ios5.png)

12. Appuyez sur **AJOUTER** dans la page **Profil importé**.

    ![Appuyez sur Ajouter.](./media/vpn-gateway-vwan-config-openvpn-clients/ios6.png)

13. Lancez l’application OpenVPN et faites glisser le commutateur vers la droite dans la page **Profil** pour vous connecter

    ![Se connecter](./media/vpn-gateway-vwan-config-openvpn-clients/ios8.png)


## <a name="linux-clients"></a><a name="linux"></a>Clients Linux

1. Ouvrez une nouvelle session de terminal. Vous pouvez ouvrir une nouvelle session en appuyant simultanément sur « Ctrl + Alt + t ».
2. Saisissez la commande suivante pour installer les composants nécessaires :

   ```
   sudo apt-get install openvpn
   sudo apt-get -y install network-manager-openvpn
   sudo service network-manager restart
   ```
3. Téléchargez le profil VPN pour la passerelle. Pour y parvenir, accédez à l’onglet Configurations point à site dans le Portail Azure.
4. Exportez le certificat client P2S que vous avez créé et chargé dans votre configuration P2S sur la passerelle. Consultez les articles suivants :

   * Instructions sur la [passerelle VPN](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport) 
   
   * Instructions sur [Virtual WAN](../articles/virtual-wan/certificates-point-to-site.md#clientexport)
5. Procédez à l’extraction de la clé privée et de l’empreinte numérique en base 64 à partir du fichier .pfx. Pour ce faire, plusieurs options s’offrent à vous. Vous devez utiliser OpenSSL sur votre ordinateur.

    ```
    openssl pkcs12 -in "filename.pfx" -nodes -out "profileinfo.txt"
    ```
   Le fichier *profileinfo.txt* contient la clé privée et l’empreinte numérique pour l’autorité de certification et le certificat client. Assurez-vous d’utiliser l’empreinte numérique du certificat client.

6. Ouvrez *profileinfo.txt* dans un éditeur de texte. Pour obtenir l’empreinte numérique du certificat client (enfant), sélectionnez le texte (y compris et entre) « ---BEGIN CERTIFICATE--- » et « ---END CERTIFICATE--- » pour le certificat enfant et copiez-le. Vous pouvez identifier le certificat enfant en examinant la ligne subject=/.

7. Ouvrez le fichier *vpnconfig.ovpn* et recherchez la section ci-dessous. Remplacez tout le contenu compris entre « cert » et « / cert ».

   ```
   # P2S client certificate
   # please fill this field with a PEM formatted cert
   <cert>
   $CLIENTCERTIFICATE
   </cert>
   ```
8. Ouvrez profileinfo.txt dans un éditeur de texte. Pour obtenir la clé privée, sélectionnez le texte (y compris et entre) « ---BEGIN PRIVATE KEY--- » et « ---END PRIVATE KEY--- », puis copiez-le.

9. Revenez au fichier vpnconfig.ovpn dans votre éditeur de texte et recherchez cette section. Collez la clé privée, en remplaçant tout le contenu compris entre « key » et « /key ».

   ```
   # P2S client root certificate private key
   # please fill this field with a PEM formatted key
   <key>
   $PRIVATEKEY
   </key>
   ```

10. Ne modifiez aucun autre champ. Utilisez la configuration complétée dans l’entrée client pour vous connecter au VPN.
11. Pour vous connecter en utilisant une ligne de commande, tapez la commande suivante :
  
    ```
    sudo openvpn --config <name and path of your VPN profile file>&
    ```
12. Pour vous connecter à l’aide de l’interface utilisateur graphique, accédez aux paramètres du système.
13. Cliquez sur **+** pour ajouter une nouvelle connexion VPN.
14. Sous **Ajouter un VPN**, choisissez **Importer à partir d’un fichier...**
15. Naviguez jusqu’au fichier de profil et double-cliquez dessus ou bien choisissez **Ouvrir**
16. Cliquez sur **Ajouter** dans la fenêtre **Ajouter un VPN**.
  
    ![Importer à partir d'un fichier](./media/vpn-gateway-vwan-config-openvpn-clients/import.png)
17. Vous pouvez vous connecter en mettant le VPN sur **ON** à partir de la page **Paramètres réseau**, ou sous l’icône de réseau dans la barre d’état système.
