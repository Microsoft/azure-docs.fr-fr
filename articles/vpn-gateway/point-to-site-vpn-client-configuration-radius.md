---
title: 'Passerelle VPN Azure : Créer et installer les fichiers de configuration du client VPN - Connexions P2S RADIUS'
description: Créer des fichiers config du client VPN Windows, OS X et Linux pour les connexions utilisant l’authentification RADIUS.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: e6d811e19bb19c8c8bf96764cfcca2b1294f4a85
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91440056"
---
# <a name="create-and-install-vpn-client-configuration-files-for-p2s-radius-authentication"></a>Créer et installer les fichiers de configuration du client VPN pour une authentification P2S RADIUS

Pour vous connecter à un réseau virtuel de point à site, vous devez configurer l’appareil client à partir duquel vous allez vous connecter. Vous pouvez créer des connexions VPN de point à site à partir d’appareils clients Windows, OS X et Linux. 

Lorsque vous utilisez l’authentification RADIUS, vous disposez de plusieurs options d’authentification : authentification par nom d’utilisateur/mot de passe, authentification par certificat, et bien d’autres. La configuration du client VPN est différente pour chaque type d’authentification. Pour configurer le client VPN, vous utilisez les fichiers de configuration du client qui contiennent les paramètres requis. Cet article vous aide à créer et installer la configuration du client VPN pour le type d’authentification RADIUS que vous souhaitez utiliser.

>[!IMPORTANT]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

Le flux de travail de configuration pour l’authentification RADIUS point à site est le suivant :

1. [Configurez la passerelle VPN Azure pour une connectivité P2S](point-to-site-how-to-radius-ps.md).
2. [Configurez votre serveur RADIUS pour l’authentification](point-to-site-how-to-radius-ps.md#radius). 
3. **Obtenez la configuration du client VPN pour l’option d’authentification de votre choix et utilisez-la pour installer le client VPN** (cet article).
4. [Effectuez votre configuration P2S et connectez-vous](point-to-site-how-to-radius-ps.md).

>[!IMPORTANT]
>Si vous apportez des modifications à la configuration du VPN de point à site après avoir généré le profil de configuration du client VPN, comme le type de protocole ou d’authentification du VPN, vous devez générer et installer une nouvelle configuration du client VPN sur les appareils de vos utilisateurs.
>
>

Pour utiliser les sections de cet article, vous devez d’abord décider quel type d’authentification vous souhaitez utiliser : nom d’utilisateur/mot de passe, certificat ou d’autres types d’authentification. Les procédures pour Windows, OS X et Linux sont détaillées dans chaque section (pour le moment, les procédures disponibles sont limitées).


## <a name="usernamepassword-authentication"></a><a name="adeap"></a>Authentification par nom d’utilisateur/mot de passe

Vous pouvez configurer l’authentification par nom d’utilisateur/mot de passe pour qu’elle utilise Active Directory ou non. Dans ces deux scénarios, veillez à ce que tous les utilisateurs qui se connectent disposent d’informations d’identification (nom d’utilisateur et mot de passe) qui peuvent être authentifiées via RADIUS.

Lorsque vous configurez l’authentification par nom d’utilisateur/mot de passe, vous pouvez seulement créer une configuration pour le protocole d’authentification par nom d’utilisateur/mot de passe EAP-MSCHAPv2. Dans les commandes, `-AuthenticationMethod` est `EapMSChapv2`.

### <a name="1-generate-vpn-client-configuration-files"></a><a name="usernamefiles"></a> 1. Générer les fichiers de configuration du client VPN

Vous pouvez générer des fichiers de configuration du client VPN à l’aide du portail Azure, ou en utilisant Azure PowerShell.

#### <a name="azure-portal"></a>Portail Azure

1. Accès à la passerelle de réseau virtuel.
2. Cliquez sur **Configuration de point à site**.
3. Cliquez sur **Téléchargement du client VPN**.
4. Sélectionnez le client et remplissez les informations demandées.
5. Cliquez sur **Télécharger** pour générer le fichier .zip.
6. Le fichier .zip sera normalement télécharger dans votre dossier Téléchargements.

#### <a name="azure-powershell"></a>Azure PowerShell

Générez les fichiers de configuration du client VPN à utiliser avec l’authentification par nom d’utilisateur/mot de passe. Vous pouvez générer les fichiers de configuration du client VPN à l’aide de la commande suivante :

```azurepowershell-interactive
New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapMSChapv2"
```
 
L’exécution de la commande renvoie un lien. Copiez et collez ce lien dans un navigateur web pour télécharger **VpnClientConfiguration.zip**. Décompressez le fichier pour afficher les dossiers suivants : 
 
* **WindowsAmd64** et **WindowsX86** : Dans ces dossiers se trouvent respectivement les packages de programme d’installation pour Windows 64 bits et Windows 32 bits. 
* **Generic** : les informations générales utilisées pour créer la configuration de votre client VPN se trouvent dans ce dossier. Vous n’avez pas besoin de ce dossier pour les configurations d’authentification par nom d’utilisateur/mot de passe.
* **Mac** : si vous avez configuré IKEv2 lors de la création de la passerelle du réseau virtuel, un dossier nommé **Mac** s’affiche, dans lequel se trouve un fichier **mobileconfig**. Vous utilisez ce fichier pour configurer les clients Mac.

Si vous avez déjà créé les fichiers de configuration du client, vous pouvez les récupérer à l’aide de la cmdlet `Get-AzVpnClientConfiguration`. Toutefois, si vous apportez des modifications à votre configuration VPN de point à site, comme le type d’authentification ou de protocole VPN, elle ne se met pas à jour automatiquement. Vous devez exécuter la cmdlet  `New-AzVpnClientConfiguration` pour créer un téléchargement de configuration.

Pour récupérer les fichiers de configuration du client générés précédemment, exécutez la commande suivante :

```azurepowershell-interactive
Get-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW"
```

### <a name="2-configure-vpn-clients"></a><a name="setupusername"></a> 2. Configurer les clients VPN

Vous pouvez configurer les clients VPN suivants :

* [Windows](#adwincli)
* [Mac (OS X)](#admaccli)
* [Linux à l’aide de strongSwan](#adlinuxcli)
 
#### <a name="windows-vpn-client-setup"></a><a name="adwincli"></a>Configuration d’un client VPN Windows

Vous pouvez utiliser le même package de configuration du client VPN sur chaque ordinateur client Windows, tant que la version correspond à l’architecture du client. Pour obtenir la liste des systèmes d’exploitation clients pris en charge, consultez la [FAQ sur la passerelle VPN](vpn-gateway-vpn-faq.md#P2S).

Suivez les étapes suivantes pour configurer le client VPN Windows natif pour une authentification par certificat :

1. Sélectionnez les fichiers de configuration du client VPN qui correspondent à l’architecture de l’ordinateur Windows. Pour une architecture de processeur 64 bits, choisissez le package du programme d’installation **VpnClientSetupAmd64**. Pour une architecture de processeur 32 bits, choisissez le package du programme d’installation **VpnClientSetupX86**. 
2. Double-cliquez pour installer le package. Si vous voyez une fenêtre contextuelle SmartScreen, sélectionnez **More info** (Plus d’informations) > **Run anyway** (Exécuter quand même).
3. Sur l’ordinateur client, accédez à **Paramètres réseau**, puis sélectionnez **VPN**. La connexion VPN indique le nom du réseau virtuel auquel elle se connecte. 

#### <a name="mac-os-x-vpn-client-setup"></a><a name="admaccli"></a>Configuration d’un client VPN Mac (OS X)

1. Sélectionnez le fichier **VpnClientSetup mobileconfig** et envoyez-le à chaque utilisateur. Vous pouvez l’envoyer par e-mail, ou par n’importe quelle autre méthode.

2. Recherchez le fichier **mobileconfig** sur l’ordinateur Mac.

   ![Emplacement du fichier mobileconfig](./media/point-to-site-vpn-client-configuration-radius/admobileconfigfile.png)

3. Étape facultative : si vous souhaitez spécifier un DNS personnalisé, ajoutez les lignes suivantes au fichier **mobileconfig** :

   ```xml
    <key>DNS</key>
    <dict>
      <key>ServerAddresses</key>
        <array>
            <string>10.0.0.132</string>
        </array>
      <key>SupplementalMatchDomains</key>
        <array>
            <string>TestDomain.com</string>
        </array>
    </dict> 
   ```
4. Double-cliquez sur le profil pour l’installer, et sélectionnez **Continuer**. Le nom du profil correspond à celui de votre réseau virtuel.

   ![Message d’installation](./media/point-to-site-vpn-client-configuration-radius/adinstall.png)
5. Sélectionnez **Continuer** pour faire confiance à l’expéditeur du profil et poursuivre l’installation.

   ![Message de confirmation](./media/point-to-site-vpn-client-configuration-radius/adcontinue.png)
6. Pendant l’installation du profil, vous avez la possibilité de spécifier le nom d’utilisateur et le mot de passe pour l’authentification VPN. Vous n’êtes pas obligé de renseigner ces informations. Si vous le faites, les informations sont enregistrées et utilisées automatiquement à la connexion. Sélectionnez **Installer** pour continuer.

   ![Champs Nom d’utilisateur et Mot de passe pour VPN](./media/point-to-site-vpn-client-configuration-radius/adsettings.png)
7. Entrez un nom d’utilisateur et un mot de passe pour les privilèges requis pour installer le profil sur votre ordinateur. Sélectionnez **OK**.

   ![Champs Nom d’utilisateur et Mot de passe pour l’installation du profil](./media/point-to-site-vpn-client-configuration-radius/adusername.png)
8. Une fois le profil installé, il est visible dans la boîte de dialogue **Profils**. Vous pouvez également ouvrir cette boîte de dialogue à partir de **Préférences système**.

   ![Boîte de dialogue « Profils »](./media/point-to-site-vpn-client-configuration-radius/adsystempref.png)
9. Pour accéder à la connexion VPN, ouvrez la boîte de dialogue **Réseau** à partir de **Préférences système**.

   ![Icônes dans Préférences système](./media/point-to-site-vpn-client-configuration-radius/adnetwork.png)
10. La connexion VPN apparaît en tant que **IkeV2-VPN**. Vous pouvez changer le nom en mettant à jour le fichier **mobileconfig**.

    ![Détails de la connexion VPN](./media/point-to-site-vpn-client-configuration-radius/adconnection.png)
11. Sélectionnez **Paramètres d’authentification**. Sélectionnez **Nom d’utilisateur** dans la liste et entrez vos informations d’identification. Si vous avez entré les informations d’identification précédemment, alors **Nom d’utilisateur** est automatiquement sélectionné dans la liste et le nom d’utilisateur et le mot de passe sont déjà renseignés. Sélectionnez **OK** pour enregistrer les paramètres.

    ![Capture d’écran montrant la liste déroulante « Paramètres d’authentification » avec l’option « Nom d’utilisateur » sélectionnée.](./media/point-to-site-vpn-client-configuration-radius/adauthentication.png)
12. Dans la boîte de dialogue **Réseau**, sélectionnez **Appliquer** pour enregistrer les modifications. Pour lancer la connexion, sélectionnez **Connecter**.

#### <a name="linux-vpn-client-setup-through-strongswan"></a><a name="adlinuxcli"></a>Configuration du client VPN Linux via strongSwan

Les instructions suivantes ont été créées à l’aide de strongSwan 5.5.1 sur Ubuntu 17.0.4. Les écrans réels peuvent différer en fonction de votre version de Linux et de strongSwan.

1. Ouvrez le **Terminal** pour installer **strongSwan** et son gestionnaire de réseau en exécutant la commande dans l’exemple. Si vous recevez une erreur liée à `libcharon-extra-plugins`, remplacez la valeur par `strongswan-plugin-eap-mschapv2`.

   ```Terminal
   sudo apt-get install strongswan libcharon-extra-plugins moreutils iptables-persistent network-manager-strongswan
   ```
2. Sélectionnez l’icône du **Gestionnaire de réseau** (flèche vers le haut/flèche vers le bas), puis sélectionnez **Modifier les connexions**.

   ![Sélection de « Modifier les connexions » dans le Gestionnaire de réseau](./media/point-to-site-vpn-client-configuration-radius/EditConnection.png)
3. Sélectionnez le bouton **Ajouter** pour créer une connexion.

   ![Bouton « Ajouter » pour une connexion](./media/point-to-site-vpn-client-configuration-radius/AddConnection.png)
4. Sélectionnez **IPsec/IKEv2 (strongswan)** dans le menu déroulant, puis sélectionnez **Créer**. Vous pouvez renommer votre connexion à cette étape.

   ![Sélection du type de connexion](./media/point-to-site-vpn-client-configuration-radius/AddIKEv2.png)
5. Ouvrez le fichier **VpnSettings.xml** à partir du dossier **générique** des fichiers de configuration du client téléchargé. Recherchez la balise appelée `VpnServer` et copiez le nom, en commençant par `azuregateway` et en terminant par `.cloudapp.net`.

   ![Contenu du fichier VpnSettings.xml](./media/point-to-site-vpn-client-configuration-radius/VpnSettings.png)
6. Collez ce nom dans le champ **Adresse** de votre nouvelle connexion VPN sous la section **Passerelle**. Ensuite, sélectionnez l’icône du dossier à la fin du champ **Certificat**, accédez au dossier **Générique**, puis sélectionnez le fichier **VpnServerRoot**.
7. Dans la section **Client** de la connexion, sélectionnez **EAP** pour **Authentification**, puis entrez votre nom d’utilisateur et mot de passe. Vous devrez peut-être sélectionner l’icône du verrou à droite pour enregistrer ces informations. Sélectionnez ensuite **Enregistrer**.

   ![Modification des paramètres de connexion](./media/point-to-site-vpn-client-configuration-radius/editconnectionsettings.png)
8. Sélectionnez l’icône du **Gestionnaire de réseau** (flèche vers le haut/flèche vers le bas), puis survolez **Connexions VPN**. La connexion VPN que vous avez créée apparaît. Pour lancer la connexion, sélectionnez-la.

   ![Connexion « Radius VPN » dans le Gestionnaire de réseau](./media/point-to-site-vpn-client-configuration-radius/ConnectRADIUS.png)

## <a name="certificate-authentication"></a><a name="certeap"></a>Authentification par certificat
 
Vous pouvez créer les fichiers de configuration du client VPN pour une authentification par certificat RADIUS qui utilise le protocole EAP-TLS. En règle générale, un certificat émis par l’entreprise est utilisé pour authentifier un utilisateur pour un VPN. Veillez à ce que tous les utilisateurs qui se connectent disposent d’un certificat installé sur leurs appareils qui peut être validé par votre serveur RADIUS.

>[!NOTE]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

Dans les commandes, `-AuthenticationMethod` est `EapTls`. Lors de l’authentification par certificat, le client valide le serveur RADIUS en validant son certificat. `-RadiusRootCert` est le fichier .cer qui contient le certificat racine utilisé pour valider le serveur RADIUS.

Chaque appareil du client VPN doit avoir un certificat client installé. Un appareil Windows peut posséder plusieurs certificats de client. Lors de l’authentification, cela peut entraîner l’affichage d’une boîte de dialogue contextuelle qui répertorie tous les certificats. L’utilisateur doit alors choisir le certificat à utiliser. Le certificat approprié peut être filtré en spécifiant le certificat racine auquel le certificat client doit être attaché. 

`-ClientRootCert` est le fichier .cer qui contient le certificat racine. Ce paramètre est facultatif. Si l’appareil à partir duquel vous souhaitez vous connecter possède un seul certificat client, il est inutile de spécifier ce paramètre.

### <a name="1-generate-vpn-client-configuration-files"></a><a name="certfiles"></a>1. Générer les fichiers de configuration du client VPN

Générez les fichiers de configuration du client VPN à utiliser avec l’authentification par certificat. Vous pouvez générer les fichiers de configuration du client VPN à l’aide de la commande suivante :
 
```azurepowershell-interactive
New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls" -RadiusRootCert <full path name of .cer file containing the RADIUS root> -ClientRootCert <full path name of .cer file containing the client root> | fl
```

L’exécution de la commande renvoie un lien. Copiez et collez le lien dans un navigateur web pour télécharger VpnClientConfiguration.zip. Décompressez le fichier pour afficher les dossiers suivants :

* **WindowsAmd64** et **WindowsX86** : Dans ces dossiers se trouvent respectivement les packages de programme d’installation pour Windows 64 bits et Windows 32 bits. 
* **GenericDevice** : les informations générales utilisées pour créer la configuration de votre client VPN se trouvent dans ce dossier.

Si vous avez déjà créé les fichiers de configuration du client, vous pouvez les récupérer à l’aide de la cmdlet `Get-AzVpnClientConfiguration`. Toutefois, si vous apportez des modifications à votre configuration VPN de point à site, comme le type d’authentification ou de protocole VPN, elle ne se met pas à jour automatiquement. Vous devez exécuter la cmdlet  `New-AzVpnClientConfiguration` pour créer un téléchargement de configuration.

Pour récupérer les fichiers de configuration du client générés précédemment, exécutez la commande suivante :

```azurepowershell-interactive
Get-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" | fl
```
 
### <a name="2-configure-vpn-clients"></a><a name="setupusername"></a> 2. Configurer les clients VPN

Vous pouvez configurer les clients VPN suivants :

* [Windows](#certwincli)
* [Mac (OS X)](#certmaccli)
* Linux (pris en charge, pas encore d’article sur la procédure)

#### <a name="windows-vpn-client-setup"></a><a name="certwincli"></a>Configuration d’un client VPN Windows

1. Sélectionnez un package de configuration et installez-le sur l’appareil client. Pour une architecture de processeur 64 bits, choisissez le package du programme d’installation **VpnClientSetupAmd64**. Pour une architecture de processeur 32 bits, choisissez le package du programme d’installation **VpnClientSetupX86**. Si vous voyez une fenêtre contextuelle SmartScreen, sélectionnez **More info** (Plus d’informations) > **Run anyway** (Exécuter quand même). Vous pouvez également enregistrer le package pour l’installer sur d’autres ordinateurs clients.
2. Chaque client doit disposer d’un certificat client pour l’authentification. Installez le certificat client. Pour plus d’informations sur les certificats clients, consultez [Générer et exporter des certificats pour les connexions de point à site à l’aide de PowerShell sous Windows 10 ou Windows Server 2016](vpn-gateway-certificates-point-to-site.md). Pour installer un certificat qui a été généré, consultez [Installer un certificat sur des clients Windows](point-to-site-how-to-vpn-client-install-azure-cert.md).
3. Sur l’ordinateur client, accédez à **Paramètres réseau**, puis sélectionnez **VPN**. La connexion VPN indique le nom du réseau virtuel auquel elle se connecte.

#### <a name="mac-os-x-vpn-client-setup"></a><a name="certmaccli"></a>Configuration d’un client VPN Mac (OS X)

Vous devez créer un profil distinct pour chaque appareil Mac qui se connecte au réseau virtuel Azure. Les appareils Mac nécessitent la spécification des certificats client pour l’authentification. Le dossier **Générique** contient toutes les informations requises pour créer un profil :

* Le fichier **VpnSettings.xml** contient d’importants paramètres tels que l’adresse et le type de tunnel du serveur.
* Le fichier **VpnServerRoot.cer** contient le certificat racine requis pour valider la passerelle VPN lors de la configuration de la connexion de point à site.
* Le fichier **RadiusServerRoot.cer** contient le certificat racine requis pour valider le serveur RADIUS lors de l’authentification.

Suivez les étapes ci-dessous afin de configurer le client VPN Mac natif pour une authentification par certificat :

1. Importez les certificats racine **VpnServerRoot** et **RadiusServerRoot** sur votre Mac. Copiez chaque fichier sur votre Mac, double-cliquez dessus, puis sélectionnez **Ajouter**.

   ![Ajout du certificat VpnServerRoot](./media/point-to-site-vpn-client-configuration-radius/addcert.png)

   ![Ajout du certificat RadiusServerRoot](./media/point-to-site-vpn-client-configuration-radius/radiusrootcert.png)
2. Chaque client doit disposer d’un certificat client pour l’authentification. Installez le certificat client sur l’appareil client.
3. Ouvrez la boîte de dialogue **Réseau** sous **Network Preferences** (Préférences réseau). Sélectionnez **+** pour créer un profil de connexion de client VPN pour une connexion de point à site sur le réseau virtuel Azure.

   La valeur **Interface** est **VPN** et la valeur **Type de réseau privé virtuel** est **IKEv2**. Spécifiez un nom pour le profil dans le champ **Nom du service**, puis sélectionnez **Créer** pour créer le profil de connexion de client VPN.

   ![Informations sur le nom de service et l’interface](./media/point-to-site-vpn-client-configuration-radius/network.png)
4. Dans le dossier **Générique**, depuis le fichier **VpnSettings.xml**, copiez la valeur de la balise **VpnServer**. Collez cette valeur dans les champs **Adresse du serveur** et **ID distant** du profil. Laissez le champ **ID local** vide.

   ![Informations sur le serveur](./media/point-to-site-vpn-client-configuration-radius/servertag.png)
5. Sélectionnez **Paramètres d’authentification**, puis **Certificat**. 

   ![Authentication settings](./media/point-to-site-vpn-client-configuration-radius/certoption.png)
6. Cliquez sur **Sélectionner** pour choisir le certificat que vous souhaitez utiliser pour l’authentification.

   ![Sélection d’un certificat pour l’authentification](./media/point-to-site-vpn-client-configuration-radius/certificate.png)
7. **Choisir une identité** affiche une liste de certificats à choisir. Sélectionnez le certificat approprié, puis **Continuer**.

   ![Liste « Choisir une identité »](./media/point-to-site-vpn-client-configuration-radius/identity.png)
8. Dans le champ **ID local**, spécifiez le nom du certificat (renseigné à l’étape 6). Dans cet exemple, il s’agit de **ikev2Client.com**. Sélectionnez ensuite sur le bouton **Appliquer** pour enregistrer les modifications.

   ![Champ « ID local »](./media/point-to-site-vpn-client-configuration-radius/applyconnect.png)
9. Dans la boîte de dialogue **Réseau**, sélectionnez **Appliquer** pour enregistrer toutes les modifications. Sélectionnez ensuite **Connecter** pour lancer la connexion de point à site au réseau virtuel Azure.

## <a name="working-with-other-authentication-types-or-protocols"></a><a name="otherauth"></a>Utilisation d’autres types ou protocoles d’authentification

Pour utiliser un type d’authentification différent (OTP, par exemple) ou un protocole d’authentification différent (PEAP-MSCHAPv2 à la place d’un protocole EAP-MSCHAPv2), vous devez créer votre propre profil de configuration de client VPN. Pour créer le profil, vous avez besoin d’informations telles que l’adresse IP, du type de tunnel et des itinéraires de tunnel fractionné de la passerelle du réseau virtuel. Vous pouvez obtenir ces informations en procédant comme suit :

1. Utilisez la cmdlet `Get-AzVpnClientConfiguration` pour générer la configuration du client VPN pour le protocole EapMSChapv2.

2. Décompressez le fichier VpnClientConfiguration.zip et recherchez le dossier **GenericDevice**. Ignorez les dossiers contenant les programmes d’installation Windows pour architecture 64 bits et 32 bits.
 
3. Le dossier **GenericDevice** contient un fichier XML nommé **VpnSettings**. Ce fichier contient toutes les informations nécessaires :

   * **VpnServer** : nom de domaine complet de la passerelle VPN Azure. Il s’agit de l’adresse à laquelle le client se connecte.
   * **VpnType** : type de tunnel que vous utilisez pour vous connecter.
   * **Routes** : routes que vous devez configurer dans votre profil afin que seul le trafic en direction du réseau virtuel Azure passe par le tunnel P2S.
   
   Le dossier **GenericDevice** contient aussi un fichier .cer nommé **VpnServerRoot**. Ce fichier contient le certificat racine requis pour valider la passerelle VPN Azure lors de la configuration de la connexion de point à site. Installez le certificat sur tous les appareils qui se connecteront au réseau virtuel Azure.

## <a name="next-steps"></a>Étapes suivantes

Revenez à l’article pour [terminer la configuration P2S](point-to-site-how-to-radius-ps.md).

Pour plus d’informations sur la résolution des problèmes liés aux connexions point à site, voir [Résolution des problèmes de connexion de point à site Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
