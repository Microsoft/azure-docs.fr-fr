---
title: Azure VMware Solution by CloudSimple - Configurer VPN entre le réseau local et le cloud privé
description: Décrit comment configurer une connexion VPN de site à site ou de point à site entre votre réseau local et votre cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 5cd08ce938a25e3311358892c9afa9eefd6c6dbf
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424682"
---
# <a name="configure-a-vpn-connection-to-your-cloudsimple-private-cloud"></a>Configurer une connexion VPN vers votre cloud privé CloudSimple

Les passerelles VPN vous permettent de vous connecter à un réseau CloudSimple à partir de votre réseau local et d’un ordinateur client à distance.  Dans cet article, vous trouverez des informations sur la configuration des passerelles VPN à partir du portail CloudSimple.  Une connexion VPN entre votre réseau local et votre réseau CloudSimple permet d’accéder à vCenter et aux charges de travail depuis votre cloud privé. CloudSimple prend en charge les passerelles VPN de point à site et de site à site.

## <a name="vpn-gateway-types"></a>Types de passerelle VPN

* Une connexion **VPN de point à site** est la façon la plus simple pour vous connecter à votre cloud privé à partir de votre ordinateur. Utilisez la connectivité VPN de point à site pour vous connecter au cloud privé à distance.
* La connexion **VPN de site à site** vous permet de configurer vos charges de travail de cloud privé pour accéder aux services locaux. Vous pouvez également utiliser Active Directory localement comme source d’identité pour l’authentification auprès de votre vCenter de cloud privé.  Actuellement, le type **VPN basé sur une stratégie** est pris en charge.

Dans une région, vous pouvez créer une passerelle VPN de site à site et une passerelle VPN de point à site.

## <a name="point-to-site-vpn"></a>VPN de point à site

Pour créer une passerelle VPN de point à site, consultez [Créer une passerelle VPN de point à site](vpn-gateway.md#create-point-to-site-vpn-gateway).

### <a name="connect-to-cloudsimple-using-point-to-site-vpn"></a>Se connecter à CloudSimple à l’aide du VPN point à site

Un client VPN est nécessaire pour la connexion à CloudSimple depuis votre ordinateur.  Téléchargez le [client OpenVPN](https://openvpn.net/community-downloads/) pour Windows ou [Viscosity](https://www.sparklabs.com/viscosity/download/) pour Mac OS et OS X.

1. Lancez le portail CloudSimple et sélectionnez **Réseau**.
2. Sélectionnez **Passerelle VPN**.
3. Dans la liste des passerelles VPN, cliquez sur la passerelle VPN de point à site.
4. Sélectionnez **Utilisateurs**.
5. Cliquez sur **Télécharger la configuration de mon VPN**

    ![Télécharger une configuration VPN](media/download-p2s-vpn-configuration.png)

6. Importer la configuration sur votre client VPN

    * Instructions pour [l’importation de la configuration sur un client Windows](https://openvpn.net/vpn-server-resources/connecting-to-access-server-with-windows/#openvpn-open-source-openvpn-gui-program)
    * Instructions pour [l’importation de la configuration sur un client macOS ou OS X](https://www.sparklabs.com/support/kb/article/getting-started-with-viscosity-mac/#creating-your-first-connection)

7. Connectez-vous à la passerelle VPN CloudSimple.

L’exemple ci-dessous illustre l’importation de la connexion à l’aide du client **Viscosity**.

#### <a name="import-connection-on-viscosity-client"></a>Importer la connexion sur le client Viscosity

1. Extrayez le contenu de la configuration VPN à partir du fichier .zip téléchargé.

2. Ouvrez Viscosity sur votre ordinateur.

3. Cliquez sur l’icône **+** et sélectionnez **Importer connexion** > **À partir d’un fichier**.

    ![Importer la configuration VPN à partir d’un fichier](media/import-p2s-vpn-config.png)

4. Sélectionnez le fichier de configuration OpenVPN (. ovpn) pour le protocole que vous souhaitez utiliser, puis cliquez sur **Ouvrir**.

    ![Capture d'écran mettant en évidence les fichiers de configuration OpenVPN que vous pouvez sélectionner.](media/import-p2s-vpn-config-choose-ovpn.png)

La connexion s’affiche maintenant dans le menu Viscosity.

#### <a name="connect-to-the-vpn"></a>Connexion au VPN

Pour vous connecter au VPN à l’aide du client OpenVPN Viscosity, sélectionnez la connexion dans le menu. L’icône de menu se met à jour pour indiquer que la connexion est établie.

![Capture d'écran représentant l'état de connectivité VPN CloudSimple.](media/vis03.png)

### <a name="connecting-to-multiple-private-clouds"></a>Connexion à plusieurs clouds privés

Une connexion VPN de point à site résout les noms DNS du premier cloud privé que vous créez. Lorsque vous souhaitez accéder à d’autres clouds privés, vous devez mettre à jour le serveur DNS sur votre client VPN.

1. Lancez le [portail CloudSimple](access-cloudsimple-portal.md).

2. Accédez à **Ressources** > **Clouds privés** , puis sélectionnez le cloud privé auquel vous souhaitez vous connecter.

3. Sur la page **Résumé** du cloud privé sous **Informations de base** , copiez l’adresse IP du serveur DNS du cloud privé.

    ![Serveurs DNS de cloud privé](media/private-cloud-dns-server.png)

4. Cliquez avec le bouton droit sur l’icône de Viscosity dans la barre d' état système de votre ordinateur, puis sélectionnez **Préférences**.

    ![VPN](media/vis00.png)

5. Sélectionnez la connexion VPN CloudSimple.

    ![Connexion VPN](media/viscosity-client.png)

6. Cliquez sur **Modifier** pour modifier les propriétés de connexion.

    ![Modifier la connexion VPN](media/viscosity-edit-connection.png)

7. Cliquez sur l'onglet **Réseau** et entrez les adresses IP du serveur DNS du cloud privé, séparées par des virgules ou des espaces, et le domaine sous la forme ```cloudsimple.io```.  Sélectionnez **Ignorer les paramètres DNS envoyés par le serveur VPN**.

    ![Mise en réseau VPN](media/viscosity-edit-connection-networking.png)

> [!IMPORTANT]
> Pour vous connecter à votre premier cloud privé, supprimez ces paramètres et connectez-vous au serveur VPN.

## <a name="site-to-site-vpn"></a>VPN de site à site

Pour créer une passerelle VPN de site à site, consultez [Créer une passerelle VPN de site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway).  La connexion VPN de site à site à partir de votre réseau local vers votre cloud privé offre les avantages suivants.  

* Accessibilité de votre cloud privé vCenter à partir de toute station de travail dans votre réseau local
* Utilisation de votre Active Directory local comme source d’identité vCenter
* Transfert pratique de modèles de machines virtuelles, ISO et autres fichiers depuis vos ressources locales vers votre cloud privé vCenter
* Accessibilité des charges de travail en cours d’exécution sur votre cloud privé à partir de votre réseau local

Pour configurer votre passerelle VPN locale en mode haute disponibilité, consultez [Configurer une connexion VPN à haute disponibilité](high-availability-vpn-connection.md).

> [!IMPORTANT]
>    1. Définissez le clamping du MSS TCP sur 1200 sur votre périphérique VPN. Dans le cas où vos périphériques VPN ne prendraient pas en charge le réglage de la taille maximale de segment, vous pouvez à la place définir l’unité de transmission maximale dans l’interface de tunnel sur 1240 octets.
> 2. Après la configuration du VPN de site à site, transférez les requêtes DNS pour *.cloudsimple.io vers les serveurs DNS du cloud privé.  Suivez les instructions de [Configuration du DNS local](on-premises-dns-setup.md).
