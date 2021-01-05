---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 07/31/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: fc2393cfe87e2639ce40e66e6053d4d430518719
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87515346"
---
## <a name="1-download-the-file"></a>1. Télécharger le fichier

Exécutez les commandes suivantes : Copiez l’URL du résultat dans votre navigateur pour télécharger le fichier zip du profil.

```azurepowershell-interactive
$profile = New-AzVpnClientConfiguration -ResourceGroupName AADAuth -Name AADauthGW -AuthenticationMethod "EapTls"
   
$PROFILE.VpnProfileSASUrl
```

## <a name="2-extract-the-zip-file"></a>2. Extrayez le fichier zip.

Extrayez le fichier zip. Le fichier contient les dossiers suivants :

* AzureVPN
* Générique
* OpenVPN (si vous avez activé OpenVPN avec les paramètres **Certificat Azure** ou **Authentification RADIUS** sur la passerelle). Pour la passerelle VPN, consultez [Créer un locataire](../articles/vpn-gateway/openvpn-azure-ad-tenant.md). Pour Virtual WAN, consultez [Créer un locataire - VWAN](../articles/virtual-wan/openvpn-azure-ad-tenant.md).

## <a name="3-retrieve-information"></a>3. Récupérer des informations

Dans le dossier **AzureVPN**, accédez au fichier ***azurevpnconfig.xml*** et ouvrez-le avec le Bloc-notes. Prenez note du texte entre les balises suivantes.

```
<audience>          </audience>
<issuer>            </issuer>
<tennant>           </tennant>
<fqdn>              </fqdn>
<serversecret>      </serversecret>
```

## <a name="profile-details"></a>Détails du profil

Lorsque vous ajoutez une connexion, utilisez les informations que vous avez collectées à l’étape précédente pour la page Détails du profil. Les champs correspondent aux informations suivantes :

   * **Audience** : identifie la ressource de destination du jeton.
   * **Émetteur** : identifie le service d’émission de jeton de sécurité (STS) qui a émis le jeton ainsi que le locataire Azure AD
   * **Locataire :** contient un identificateur unique non modifiable du client du répertoire qui a émis le jeton.
   * **FQDN :** nom de domaine complet (FQDN) sur la passerelle VPN Azure
   * **ServerSecret :** clé prépartagée de la passerelle VPN

## <a name="folder-contents"></a>Contenu du dossier

* Le **dossier générique** contient le certificat de serveur public et le fichier VpnSettings.xml. Le fichier VpnSettings.xml contient les informations nécessaires à la configuration d’un client générique.

* Le fichier zip téléchargé peut également contenir des dossiers **WindowsAmd64** et **WindowsX86**. Ces dossiers contiennent le programme d’installation de SSTP et IKEv2 pour les clients Windows. Vous avez besoin de droits d’administrateur sur le client pour les installer.
