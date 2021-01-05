---
title: 'Connexion d’un ordinateur à un réseau virtuel avec authentification par certificat P2S: : Portail Azure Classic'
titleSuffix: Azure VPN Gateway
description: Créez une connexion de passerelle VPN point à site classique avec le Portail Azure.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: 42b0945de55775f55f20cefdeb547cb5d6492c06
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657072"
---
# <a name="configure-a-point-to-site-connection-by-using-certificate-authentication-classic"></a>Configurer une connexion point à site à l'aide d'une authentification par certificat (classique)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

Cet article explique comment créer un réseau virtuel avec une connexion point à site. Pour créer ce réseau virtuel, vous devez utiliser le modèle de déploiement Classic et le Portail Azure. Cette configuration utilise des certificats pour authentifier le client qui se connecte, qu’ils soient auto-signés ou délivrés par une autorité de certification. Vous pouvez également créer cette configuration avec un outil ou un modèle de déploiement différent en utilisant les options décrites dans les articles suivants :

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [Portail Azure (classique)](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
>

Vous utilisez une passerelle VPN point à site (P2S) pour créer une connexion sécurisée à votre réseau virtuel à partir d'un ordinateur client individuel. Les connexions VPN point à site vous permettent de vous connecter à votre réseau virtuel à partir d'un emplacement distant. Lorsqu'un nombre restreint de clients doit se connecter à un réseau virtuel, l'utilisation d'un VPN P2S est une solution utile qui constitue une alternative au VPN site à site. Une connexion VPN P2S est établie en étant démarrée à partir de l’ordinateur du client.

> [!IMPORTANT]
> Le modèle de déploiement classique prend en charge les clients VPN Windows uniquement et utilise le protocole de tunneling Secure Socket (SSTP), un protocole VPN basé sur SSL. Pour prendre en charge des clients VPN non Windows, vous devez créer votre réseau virtuel à l'aide du modèle de déploiement Resource Manager. Le modèle de déploiement Azure Resource Manager prend en charge les VPN IKEv2, en plus de SSTP. Pour plus d'informations, consultez [À propos des connexions P2S](point-to-site-about.md).
>
>

![Diagramme point à site](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/point-to-site-connection-diagram.png)

## <a name="settings-and-requirements"></a>Paramètres et prérequis

### <a name="requirements"></a>Configuration requise

Les éléments suivants sont nécessaires pour les connexions avec authentification par certificat point à site. Cet article contient des procédures qui vous aideront à les créer.

* Une passerelle VPN dynamique.
* La clé publique (fichier .cer) d’un certificat racine, chargée sur Azure. Cette clé est considérée comme un certificat approuvé et est utilisée pour l'authentification.
* Un certificat client généré à partir du certificat racine et installé sur chaque ordinateur client qui se connecte. Ce certificat est utilisé pour l’authentification du client.
* Un package de configuration du client VPN doit être généré et installé sur chaque ordinateur client qui se connecte. Le package de configuration du client configure le client VPN natif qui se trouve déjà sur le système d'exploitation en lui fournissant les informations nécessaires à la connexion au réseau virtuel.

Les connexions point à site ne nécessitent pas de périphérique VPN ou d'adresse IP publique locale. La connexion VPN est créée sur le protocole SSTP (Secure Socket Tunneling Protocol). Côté serveur, nous prenons en charge SSTP, versions 1.0, 1.1 et 1.2. Le client détermine la version à utiliser. Pour Windows 8.1 et supérieur, SSTP utilise la version 1.2 par défaut.

Pour plus d’informations, consultez [À propos des connexions point à site](point-to-site-about.md) et la [FAQ](#faq).

### <a name="example-settings"></a>Exemples de paramètres

Utilisez les valeurs suivantes pour créer un environnement de test, ou reportez-vous à celles-ci pour mieux comprendre les exemples fournis dans cet article :

* **Groupe de ressources :** TestRG
* **Nom du réseau virtuel :** VNet1
* **Espace d’adressage :** 192.168.0.0/16 <br>Pour cet exemple, nous n’utilisons qu’un seul espace d’adressage. Vous pouvez avoir plusieurs espaces d’adressage pour votre réseau virtuel.
* **Nom du sous-réseau** : FrontEnd
* **Plage d’adresses de sous-réseau :** 192.168.1.0/24
* **GatewaySubnet :** 10.11.255.0/27
* **Région :** (États-Unis) USA Est
* **Espace d’adressage du client :** 172.16.201.0/24 <br> Les clients VPN qui se connectent au réseau virtuel à l'aide de cette connexion point à site reçoivent une adresse IP de ce pool.
* **Type de connexion** : sélectionnez **Point à site**.
* **Plage d’adresses de GatewaySubnet (bloc CIDR) :** 192.168.200.0/24

Avant de commencer, vérifiez que vous disposez d'un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial).

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Créer un réseau virtuel

Si vous disposez déjà d’un réseau virtuel, vérifiez que les paramètres sont compatibles avec la conception de votre passerelle VPN, avec une attention particulière pour tous les sous-réseaux qui pourraient chevaucher d’autres réseaux.

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="create-a-vpn-gateway"></a><a name="gateway"></a>Créer une passerelle VPN

1. Accédez au réseau virtuel créé.
1. Sur la page du réseau virtuel, sélectionnez **Passerelle** sous Paramètres. Sur la page **Passerelle** figure la passerelle de votre réseau virtuel. Ce réseau virtuel ne dispose pas encore de passerelle. Cliquez sur la note indiquant **Cliquez ici pour ajouter une connexion et une passerelle**.
1. Sur la page **Configurer une connexion VPN et une passerelle**, sélectionnez les paramètres suivants :

   * Type de connexion : point à site
   * Espace d’adressage du client : ajoutez la plage d’adresses IP à partir de laquelle les clients VPN reçoivent une adresse IP lorsqu’ils se connectent. Utilisez une plage d'adresses IP privées qui ne chevauche ni l'emplacement local à partir duquel vous vous connectez ni le réseau virtuel auquel vous vous connectez.
1. Laissez la case **Ne pas configurer de passerelle pour l’instant** décochée. Nous allons créer une passerelle.
1. En bas de la page, sélectionnez **Suivant  : Passerelle >** .
1. Sur l’onglet **Passerelle**, sélectionnez les valeurs suivantes :

   * **Taille :** La taille correspond à la référence SKU de votre passerelle de réseau virtuel. Sur le portail Azure, la référence SKU par défaut est **Par défaut**. Pour plus d'informations sur les références SKU de passerelle, consultez [À propos des paramètres de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md#gwsku).
   * **Type de routage :** vous devez sélectionner **Dynamique** pour une configuration de point à site. Le routage statique ne fonctionnera pas.
   * **Sous-réseau de passerelle :** ce champ est déjà rempli automatiquement. Vous ne pouvez pas modifier le nom. Si vous tentez de le faire à l’aide de PowerShell ou autre, la passerelle ne fonctionnera pas correctement.
   * **Plage d’adresses (bloc CIDR) :** Bien qu’il soit possible de créer un sous-réseau de passerelle aussi petit que /29, nous vous recommandons de créer un sous-réseau plus vaste qui inclut un plus grand nombre d’adresses en sélectionnant au moins /28 ou /27. Cela permettra à un nombre suffisant d'adresses de s'adapter aux éventuelles configurations supplémentaires que vous pourriez souhaiter ajouter par la suite. Lorsque vous travaillez avec des sous-réseaux de passerelle, évitez d’associer un groupe de sécurité réseau (NSG) au sous-réseau de passerelle. Si vous associez un groupe de sécurité réseau à ce sous-réseau, votre passerelle VPN cessera peut-être de fonctionner normalement.
1. Sélectionnez **Vérifier + créer** pour valider vos paramètres.
1. Une fois la validation réussie, sélectionnez **Créer**. L'achèvement d'une passerelle VPN peut prendre jusqu'à 45 minutes en fonction de la référence SKU de passerelle que vous sélectionnez.

## <a name="create-certificates"></a><a name="generatecerts"></a>Créer des certificats

Azure utilise des certificats afin d'authentifier les clients VPN pour les VPN point à site. Vous chargez les informations de la clé publique du certificat racine vers Azure. La clé publique est alors considérée comme *approuvée*. Les certificats clients doivent être générés à partir du certificat racine approuvé, puis installés sur chaque ordinateur client dans le magasin de certificats Certificates-Current User\Personal\Certificates. Le certificat permet d'authentifier le client lorsqu'il se connecte au réseau virtuel. 

Si vous utilisez des certificats auto-signés, ceux-ci doivent être créés à l'aide de paramètres spécifiques. Vous pouvez créer un certificat auto-signé en suivant les instructions relatives à [PowerShell et Windows 10](vpn-gateway-certificates-point-to-site.md) ou [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Il est important de suivre les étapes décrites dans ces instructions lorsque vous utilisez des certificats racines auto-signés et que vous générez des certificats clients à partir du certificat racine auto-signé. Dans le cas contraire, les certificats que vous créez ne seront pas compatibles avec les connexions P2S, ce qui entraînera une erreur de connexion.

### <a name="acquire-the-public-key-cer-for-the-root-certificate"></a>Acquérir la clé publique (.cer) pour le certificat racine

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="generate-a-client-certificate"></a>Générer un certificat client

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="upload-the-root-certificate-cer-file"></a>Charger le fichier .cer de certificat racine

Une fois la passerelle créée, chargez le fichier .cer (qui contient les informations de clé publique) d'un certificat racine approuvé sur le serveur Azure. Ne chargez pas la clé privée du certificat racine. Une fois le certificat chargé, Azure l'utilise pour authentifier les clients qui ont installé un certificat client généré à partir du certificat racine approuvé. Si nécessaire, vous pourrez ultérieurement charger d'autres fichiers de certificat racine approuvé (20 maximum).

1. Accédez au réseau virtuel que vous avez créé.
1. Sous **Paramètres**, sélectionnez **Connexions point à site**.
1. Cliquez sur **Gérer le certificat**.
1. Sélectionnez **Télécharger**.
1. Dans le volet **Charger un certificat**, sélectionnez l’icône de dossier et accédez au certificat à charger.
1. Sélectionnez **Télécharger**.
1. Une fois chargé, le certificat apparaît sur la page Gérer le certificat. Vous devrez peut-être sélectionner **Actualiser** pour afficher le certificat que vous venez de charger.

## <a name="configure-the-client"></a>Configurer le client

Pour se connecter à un réseau virtuel à l'aide d'un VPN point à site, chaque client doit installer un package afin de configurer le client VPN Windows natif. Le package de configuration configure le client VPN Windows natif avec les paramètres nécessaires pour se connecter au réseau virtuel.

Vous pouvez utiliser le même package de configuration du client VPN sur chaque ordinateur client, tant que la version correspond à l’architecture du client. Pour connaître la liste des systèmes d’exploitation clients pris en charge, consultez [À propos des connexions point à site](point-to-site-about.md) et la [FAQ](#faq).

### <a name="generate-and-install-a-vpn-client-configuration-package"></a>Générer et installer le package de configuration du client VPN

1. Accédez aux paramètres **Connexions de point à site** de votre réseau virtuel.
1. En haut de la page, sélectionnez le package de téléchargement correspondant au système d’exploitation client sur lequel il sera installé :

   * Clients 64 bits : sélectionnez **Client VPN (64 bits)** .
   * Clients 32 bits : sélectionnez **Client VPN (32 bits)** .

1. Azure génère un package avec les paramètres spécifiques requis par le client. Chaque fois que vous apportez des modifications au réseau virtuel ou à la passerelle, vous devez télécharger un nouveau package de configuration de client et l’installer sur vos ordinateurs clients.
1. Une fois le package généré, sélectionnez **Télécharger**.
1. Installez le package de configuration de client sur votre ordinateur client. Si, lors de l’installation, il apparaît une fenêtre contextuelle SmartScreen indiquant que Windows a protégé votre PC, sélectionnez **Plus d’informations**, puis **Exécuter quand même**. Vous pouvez également enregistrer le package pour l’installer sur d’autres ordinateurs clients.

### <a name="install-a-client-certificate"></a>Installer un certificat client

Dans cet exercice, lorsque vous avez généré le certificat client, il a été automatiquement installé sur votre ordinateur. Pour créer une connexion P2S à partir d’un ordinateur client autre que celui utilisé pour générer les certificats clients, vous devez installer le certificat client généré sur cet ordinateur.

Pour installer un certificat client, vous devez disposer du mot de passe créé lors de l'exportation du certificat client. En règle générale, il vous suffit de double-cliquer sur le certificat pour l'installer. Pour plus d’informations, consultez la rubrique [Installer un certificat client exporté](vpn-gateway-certificates-point-to-site.md#install).

## <a name="connect-to-your-vnet"></a>Se connecter à votre réseau virtuel

>[!NOTE]
>Vous devez disposer de droits d’administrateur sur l’ordinateur client à partir duquel vous vous connectez.
>

1. Sur l’ordinateur client, accédez à Paramètres VPN.
1. Sélectionnez le VPN que vous avez créé. Si vous avez utilisé les paramètres de l’exemple, la connexion sera nommée **Group TestRG VNet1**.
1. Sélectionnez **Connecter**.
1. Dans la zone Réseau virtuel Windows Azure, sélectionnez **Se connecter**. Si un message contextuel relatif au certificat s’affiche, sélectionnez **Continuer** pour utiliser des privilèges élevés et **Oui** pour accepter les modifications de la configuration.
1. Si votre connexion aboutit, la notification **Connecté** apparaît.

[!INCLUDE [verify-client-certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

## <a name="verify-the-vpn-connection"></a>Vérifier la connexion VPN

1. Vérifiez que votre connexion VPN est active. Ouvrez une invite de commandes avec élévation de privilèges sur votre ordinateur client et exécutez **ipconfig/all**.
1. Affichez les résultats. Notez que l’adresse IP que vous avez reçue est l’une des adresses de la plage d’adresses de connectivité point à site que vous avez spécifiée quand vous avez créé votre réseau virtuel. Les résultats devraient être semblables à cet exemple :

   ```
    PPP adapter VNet1:
        Connection-specific DNS Suffix .:
        Description.....................: VNet1
        Physical Address................:
        DHCP Enabled....................: No
        Autoconfiguration Enabled.......: Yes
        IPv4 Address....................: 192.168.130.2(Preferred)
        Subnet Mask.....................: 255.255.255.255
        Default Gateway.................:
        NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a>Pour vous connecter à un ordinateur virtuel

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-classic-include.md)]

## <a name="to-add-or-remove-trusted-root-certificates"></a>Pour ajouter ou supprimer des certificats racine approuvés

Vous pouvez ajouter et supprimer des certificats racines approuvés à partir d'Azure. Lorsque vous supprimez un certificat racine, les clients qui possèdent un certificat généré à partir de cette racine ne sont plus en mesure de s'authentifier et de se connecter. Pour permettre à ces clients de s'authentifier et de se connecter, vous devez installer un nouveau certificat client généré à partir d'un certificat racine approuvé par Azure.

### <a name="add-a-trusted-root-certificate"></a>Ajout ou suppression d’un certificat racine approuvé

Vous pouvez ajouter à Azure 20 fichiers. cer de certificats racines approuvés au total, suivant le même processus que pour le premier.

### <a name="remove-a-trusted-root-certificate"></a>Suppression d’un certificat racine approuvé

1. Dans la section **Connexions point à site** de la page de votre réseau virtuel, sélectionnez **Gérer les certificats**.
1. Sélectionnez les points de suspension à côté du certificat à supprimer, puis sélectionnez **Supprimer**.

## <a name="to-revoke-a-client-certificate"></a>Révocation d'un certificat client

Si nécessaire, vous pouvez révoquer un certificat client. La liste de révocation de certificat vous permet de refuser sélectivement la connexion point à site en fonction des certificats clients individuels. Cette méthode est différente de la suppression d'un certificat racine approuvé. Si vous supprimez un fichier .cer de certificat racine approuvé d’Azure, vous révoquez l’accès pour tous les certificats clients générés/signés par le certificat racine révoqué. Révoquer un certificat client plutôt que le certificat racine permet de continuer à utiliser les autres certificats générés à partir du certificat racine pour l’authentification de la connexion Point à site.

La pratique courante consiste à utiliser le certificat racine pour gérer l'accès au niveaux de l'équipe ou de l'organisation, tout en utilisant des certificats clients révoqués pour le contrôle d'accès précis des utilisateurs individuels.

Vous pouvez révoquer un certificat client en ajoutant son empreinte à la liste de révocation.

1. Récupérez l’empreinte du certificat client. Pour plus d’informations, consultez l’article [Comment : récupérer l’empreinte numérique d’un certificat](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).
1. Copiez les informations dans un éditeur de texte et supprimez les espaces afin d'obtenir une chaîne continue.
1. Accédez à **Connexion VPN point à site**, puis sélectionnez **Gérer le certificat**.
1. Sélectionnez **Liste de révocation** pour ouvrir la page **Liste de révocation**.
1. Sous **Empreinte**, collez l'empreinte numérique du certificat sous la forme d'une seule ligne continue de texte, sans espaces.
1. Sélectionnez **+ Ajouter à la liste** pour ajouter l’empreinte à la liste de révocation de certificats.

Une fois la mise à jour terminée, le certificat ne peut plus être utilisé pour se connecter. Les clients qui tentent de se connecter à l'aide de ce certificat reçoivent un message indiquant que le certificat n'est plus valide.

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-point-to-site-classic-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez [Machines virtuelles](../index.yml).

* Pour plus d'informations sur la mise en réseau et les machines virtuelles Linux, consultez [Vue d'ensemble du réseau de machines virtuelles Azure et Linux](../virtual-machines/network-overview.md).

* Pour plus d’informations sur la résolution des problèmes liés aux connexions point à site, consultez l’article [Résolution des problèmes de connexion de point à site Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).