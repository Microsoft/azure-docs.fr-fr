---
title: Azure VMware Solution by CloudSimple - Étendre un réseau de couche 2 local au cloud privé
description: Explique comment configurer un VPN de couche 2 entre NSX-T sur un cloud privé CloudSimple et un client NSX Edge autonome local
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: f524bf6af66d44bc13b7c0957de7977968cbef28
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427262"
---
# <a name="migrate-workloads-using-layer-2-stretched-networks"></a>Migrer des charges de travail à l’aide de réseaux étendus de couche 2

Dans ce guide, vous apprendrez à utiliser un VPN de couche 2 (L2VPN) pour étendre un réseau de couche 2 de votre environnement local à votre cloud privé CloudSimple. Cette solution permet de migrer les charges de travail exécutées dans votre environnement VMware local vers le cloud privé Azure dans le même espace d'adressage de sous-réseau, sans avoir à redéfinir les adresses IP de vos charges de travail.

L'extension de réseaux de couche 2 basée sur L2VPN peut fonctionner avec ou sans réseaux NSX dans votre environnement VMware local. Si vous ne disposez pas de réseaux NSX pour les charges de travail locales, vous pouvez utiliser une passerelle de services NSX Edge autonome.

> [!NOTE]
> Ce guide couvre le scénario dans lequel des centres de données locaux et sur le cloud privé sont connectés par le biais d’un VPN site à site.

## <a name="deployment-scenario"></a>Scénario de déploiement

Pour étendre votre réseau local en utilisant L2VPN, vous devez configurer un serveur L2VPN (routeur NSX-T Tier0 de destination) et un client L2VPN (client autonome source).  

Dans ce scénario de déploiement, votre cloud privé est connecté à votre environnement local par le biais d’un tunnel VPN site à site qui permet aux sous-réseaux de gestion locaux et vMotion de communiquer avec les sous-réseaux de gestion du cloud privé et vMotion. Cette configuration est nécessaire pour Cross vCenter vMotion (xVC-vMotion). Un routeur NSX-T Tier0 est déployé comme serveur L2VPN dans le cloud privé.

La passerelle NSX Edge autonome est déployée dans votre environnement local en tant que client L2VPN, avant d’être jumelée au serveur L2VPN. Un point de terminaison de tunnel GRE est créé de chaque côté, puis configuré pour « étendre » le réseau de couche 2 local à votre cloud privé. Cette configuration est illustrée dans la figure suivante.

![Scénario de déploiement](media/l2vpn-deployment-scenario.png)

Pour en savoir plus sur la migration à l'aide d’un VPN L2, voir [Réseaux privés virtuels](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) dans la documentation VMware.

## <a name="prerequisites-for-deploying-the-solution"></a>Conditions préalables au déploiement de la solution

Vérifiez que les éléments suivants sont en place avant de déployer et de configurer la solution :

* La version locale de vSphere est 6.7U1+ ou 6.5P03+.
* La licence vSphere locale est au niveau Enterprise Plus (pour vSphere Distributed Switch).
* Identifiez la charge de travail du réseau de couche 2 à étendre à votre cloud privé.
* Identifiez un réseau de couche 2 dans votre environnement local pour déployer votre appliance client L2VPN.
* [Un cloud privé a déjà été créé](create-private-cloud.md).
* La version de l'appliance autonome NSX-T Edge est compatible avec la version NSX-T Manager (NSX-T 2.3.0) utilisée dans votre environnement de cloud privé.
* Un groupe de ports de jonction a été créé dans le vCenter local avec de fausses transmissions activées.
* Une adresse IP publique a été réservée pour l'adresse IP de la liaison montante autonome du client NSX-T, et un NAT 1:1 est en place pour la traduction entre les deux adresses.
* La redirection DNS est définie sur les serveurs DNS locaux pour le domaine az.cloudsimple.io afin de pointer vers les serveurs DNS du cloud privé.
* La latence RTT est inférieure ou égale à 150 ms, comme requis afin que vMotion fonctionne sur les deux sites.

## <a name="limitations-and-considerations"></a>Limitations et considérations

Le tableau suivant répertorie les versions vSphere prises en charge et les types d'adaptateurs réseau.  

| Version vSphere | Type vSwitch source | Pilote de carte d’interface réseau virtuelle | Type vSwitch cible | Pris en charge ? |
------------ | ------------- | ------------ | ------------- | ------------- 
| Tous | DVS | Tous | DVS | Oui |
| vSphere 6.7UI ou version supérieure, 6.5P03 ou version supérieure | DVS | VMXNET3 | N-VDS | Oui |
| vSphere 6.7UI ou version supérieure, 6.5P03 ou version supérieure | DVS | E1000 | N-VDS | [Non pris en charge par VWware](https://kb.vmware.com/s/article/56991) |
| vSphere 6.7UI ou 6.5P03, NSX-V ou version inférieure à NSX-T2.2, 6.5P03 ou version supérieure | Tous | Tous | N-VDS | [Non pris en charge par VWware](https://kb.vmware.com/s/article/56991) |

À compter de la version VMware NSX-T 2.3 :

* Le commutateur logique du côté du cloud privé qui est étendu en local via L2VPN ne peut pas être routé en même temps. Le commutateur logique étendu ne peut pas être connecté à un routeur logique.
* Les VPN L2VPN et IPSEC basés sur itinéraires ne peuvent être configurés qu'avec des appels d’API.

Pour plus d'informations, voir [Réseaux privés virtuels](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58.html#GUID-A8B113EC-3D53-41A5-919E-78F1A3705F58__section_44B4972B5F12453B90625D98F86D5704) dans la documentation VMware.

### <a name="sample-l2-vpn-deployment-addressing"></a>Exemple d'adressage d'un déploiement VPN L2

### <a name="on-premises-network-where-the-standalone-esg-l2-vpn-client-is-deployed"></a>Réseau local où le client autonome ESG (client VPN L2) est déployé

| **Item** | **Valeur** |
|------------|-----------------|
| Nom du réseau | MGMT_NET_VLAN469 |
| VLAN | 469 |
| CIDR| 10.250.0.0/24 |
| Adresse IP de l'appliance Edge autonome | 10.250.0.111 |
| Adresse IP NAT de l'appliance Edge autonome | 192.227.85.167 |

### <a name="on-premises-network-to-be-stretched"></a>Réseau local à étendre

| **Item** | **Valeur** |
|------------|-----------------|
| VLAN | 472 |
| CIDR| 10.250.3.0/24 |

### <a name="private-cloud-ip-schema-for-nsx-t-tier0-router-l2-vpn-serve"></a>Schéma IP de cloud privé pour routeur NSX-T Tier0 (service VPN L2)

| **Item** | **Valeur** |
|------------|-----------------|
| Interface de bouclage | 192.168.254.254/32 |
| Interface de tunnel | 5.5.5.1/29 |
| Commutateur logique (étendu) | Stretch_LS |
| Interface de bouclage (Adresse IP NAT) | 104.40.21.81 |

### <a name="private-cloud-network-to-be-mapped-to-the-stretched-network"></a>Réseau de cloud privé à mapper au réseau étendu

| **Item** | **Valeur** |
|------------|-----------------|
| VLAN | 712 |
| CIDR| 10.200.15.0/24 |

## <a name="fetch-the-logical-router-id-needed-for-l2vpn"></a>Récupérer l'ID du routeur logique nécessaire pour L2VPN

Les étapes suivantes montrent comment récupérer l'ID du routeur logique de l'instance de routeur logique Tier0 DR pour les services IPsec et L2VPN. L'ID du routeur logique sera nécessaire plus tard lors de l'implémentation de L2VPN.

1. Connectez-vous à NSX-T Manager `https://*nsx-t-manager-ip-address*` et sélectionnez **Networking (Mise en réseau)**  > **Routers (Routeurs)**  > **Provider-LR** > **Overview (Vue d’ensemble)** . Pour le **mode Haute disponibilité** , sélectionnez **Actif / passif** . Cette action ouvre une fenêtre contextuelle qui affiche la machine virtuelle Edge sur laquelle le routeur Tier0 est actuellement actif.

    ![Sélectionner Actif / passif](media/l2vpn-fetch01.png)

2. Sélectionnez **Fabric (Structure)**  > **Nodes (Nœuds)**  > **Edges (Arêtes)** . Notez l'adresse IP de gestion de la machine virtuelle Edge active (Edge VM1) identifiée à l'étape précédente.

    ![Noter l’adresse IP de gestion](media/l2vpn-fetch02.png)

3. Ouvrez une session SSH vers l'adresse IP de gestion de la machine virtuelle Edge. Exécutez la commande ```get logical-router``` avec le nom d'utilisateur **admin** et le mot de passe **CloudSimple 123!** .

    ![Capture d’écran montrant une session SSH ouverte.](media/l2vpn-fetch03.png)

4. Si vous ne voyez pas d'entrée 'DR-Provider-LR', procédez comme suit.

5. Créez deux commutateurs logiques superposés. Un commutateur logique est étendu au site local où se trouvent les charges de travail migrées. Un autre interrupteur logique est un interrupteur factice. Pour obtenir des instructions, voir [Créer un commutateur logique](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-23194F9A-416A-40EA-B9F7-346B391C3EF8.html) dans la documentation VMware.

    ![Créer un commutateur logique](media/l2vpn-fetch04.png)

6. Attachez le commutateur fictif au routeur de niveau 1 avec une adresse IP locale de liaison ou tout sous-réseau ne se chevauchant pas à partir de votre serveur local ou cloud privé. Voir [Ajouter un port de liaison descendante sur un routeur logique de niveau 1](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.3/com.vmware.nsxt.admin.doc/GUID-E7EA867C-604C-4224-B61D-2A8EF41CB7A6.html) dans la documentation VMware.

    ![Attacher un commutateur factice](media/l2vpn-fetch05.png)

7. Exécutez à nouveau la commande `get logical-router` sur la session SSH de la machine virtuelle Edge. L'UUID du routeur logique 'DR-Provider-LR' est affiché. Notez l'UUID, nécessaire lors de la configuration de L2VPN.

    ![Capture d’écran montrant l’UUID du routeur logique.](media/l2vpn-fetch06.png)

## <a name="fetch-the-logical-switch-id-needed-for-l2vpn"></a>Récupérer l'ID du commutateur logique nécessaire pour L2VPN

1. Se connecter à NSX-T Manager (`https://nsx-t-manager-ip-address`).
2. Sélectionnez **Mis en réseau** > **Commutation** > **Commutateurs** >  **<\Commutateur logique\>**  > **Vue d'ensemble** .
3. Notez l'UUID du commutateur logique étendu, nécessaire lors de la configuration de L2VPN.

    ![get logical-router output](media/l2vpn-fetch-switch01.png)

## <a name="routing-and-security-considerations-for-l2vpn"></a>Considérations relatives à la mise en réseau et à la sécurité pour L2VPN

Pour établir un VPN IPsec basé sur itinéraires entre le routeur NSX-T Tier0 et le client NSX Edge autonome, l'interface de bouclage du routeur NSX-T Tier0 doit pouvoir communiquer avec l'adresse IP publique du client autonome NSX local sur UDP 500/4500.

### <a name="allow-udp-5004500-for-ipsec"></a>Autoriser UDP 500/4500 pour IPsec

1. [Créez une adresse IP publique](public-ips.md) pour l'interface de bouclage NSX-T Tier0 dans le portail CloudSimple.

2. [Créez une table de pare-feu](firewall.md) en utilisant des règles avec état qui autorisent le trafic entrant sur UDP 500/4500, puis attachez la table de pare-feu au sous-réseau NSX-T HostTransport.

### <a name="advertise-the-loopback-interface-ip-to-the-underlay-network"></a>Annoncer l'interface de bouclage IP sur le réseau sous-jacent

1. Créez un itinéraire nul pour le réseau d'interface de bouclage. Connectez-vous à NSX-T Manager et sélectionnez **Networking (Mise en réseau)**  > **Routing (Routage)**  > **Routers (Routeurs)**  > **Provider-LR** > **Routing (Routage)**  > **Static Routes (Itinéraires statiques)** . Cliquez sur **Add** . Pour **Network (Réseau)** , entrez l’adresse IP de l’interface de bouclage. Pour **Next Hops (Prochains tronçons)** , cliquez sur **Add (Ajouter)** , spécifiez 'Null' comme prochain tronçon, puis conservez la valeur par défaut 1 pour Admin Distance (Distance admin).

    ![Ajouter un itinéraire statique](media/l2vpn-routing-security01.png)

2. Créez une liste de préfixes IP. Connectez-vous à NSX-T Manager et sélectionnez **Networking (Mise en réseau)**  > **Routing (Routage)**  > **Routers (Routeurs)**  > **Provider-LR** > **Routing (Routage)**  > **IP Prefix Lists (Listes de préfixes IP)** . Cliquez sur **Add** . Entrez un nom pour identifier la liste. Pour **Préfixes** , cliquez deux fois sur **Ajouter** . Sur la première ligne, entrez '0.0.0.0/0 pour **Network (Réseau)** et 'Deny' pour **Action** . Sur la deuxième ligne, sélectionnez **Any (N’importe quel)** pour **Network (Réseau)** et **Permit (Autoriser)** pour **Action** .
3. Attachez la liste des préfixes IP aux deux voisins BGP (TOR). Attacher la liste des préfixes IP au voisin BGP empêche l’itinéraire par défaut d'être annoncé dans BGP aux commutateurs TOR. Cependant, tout autre itinéraire incluant l’itinéraire nul annoncera l'adresse IP de l'interface de bouclage aux commutateurs TOR.

    ![Créer une liste de préfixes IP](media/l2vpn-routing-security02.png)

4. Connectez-vous à NSX-T Manager et sélectionnez **Networking (Mise en réseau)**  > **Routing (Routage)**  > **Routers (Routeurs)**  > **Provider-LR** > **Routing (Routage)**  > **BGP** > **Neighbors (Voisins)** . Sélectionnez le premier voisin. Cliquez sur **Edit (Modifier)**  > **Address Families (Familles d’adresses)** . Pour la famille IPv4, modifiez la colonne **Out Filter (Filtre sortant)** et sélectionnez la liste des préfixes IP que vous avez créée. Cliquez sur **Enregistrer** . Répétez cette étape pour le deuxième voisin.

    ![Joindre la liste des préfixes IP 1](media/l2vpn-routing-security03.png) ![Joindre la liste des préfixes IP 2](media/l2vpn-routing-security04.png)

5. Redistribuez l’itinéraire statique nul dans BGP. Pour annoncer l’itinéraire de l'interface de bouclage à la sous-couche, vous devez redistribuer l’itinéraire statique nul dans BGP. Connectez-vous à NSX-T Manager et sélectionnez **Networking (Mise en réseau)**  > **Routing (Routage)**  > **Routers (Routeurs)**  > **Provider-LR** > **Routing (Routage)**  > **Route Redistribution (Redistribution d’itinéraire)**  > **Neighbors (Voisins)** . Sélectionnez **Provider-LR-Route_Redistribution** et cliquez sur **Edit (Modifier)** . Cochez la case **Static (Statique)** , puis cliquez sur **Save (Enregistrer)** .

    ![Redistribuer l’itinéraire statique nul dans BGP](media/l2vpn-routing-security05.png)

## <a name="configure-a-route-based-vpn-on-the-nsx-t-tier0-router"></a>Configurer un VPN basé sur itinéraires sur le routeur NSX-T Tier0

Utilisez le modèle suivant pour renseigner tous les détails de la configuration d'un VPN basé sur itinéraires sur le routeur NSX-T Tier0. Les UUIDs de chaque appel POST sont nécessaires pour les appels POST suivants. Les adresses IP pour les interfaces de bouclage et de tunnel pour L2VPN doivent être uniques et ne pas chevaucher les réseaux locaux ou du cloud privé.

Les adresses IP choisies pour l'interface de bouclage et de tunnel, et utilisées pour L2VPN, doivent être uniques et ne pas chevaucher les réseaux locaux ou du cloud privé. Le réseau d'interface de bouclage doit toujours être /32.

```
Loopback interface ip : 192.168.254.254/32
Tunnel interface subnet : 5.5.5.0/29
Logical-router ID : UUID of Tier0 DR logical router obtained in section "Steps to fetch Logical-Router ID needed for L2VPN"
Logical-switch ID(Stretch) : UUID of Stretch Logical Switch obtained earlier
IPSec Service ID :
IKE profile ID :
DPD profile ID :
Tunnel Profile ID :
Local-endpoint ID :
Peer end-point ID :
IPSec VPN session ID (route-based) :
L2VPN service ID :
L2VPN session ID :
Logical-Port ID :
Peer Code :
```

Pour tous les appels d’API suivants, remplacez l'adresse IP par votre adresse IP NSX-T Manager. Vous pouvez exécuter tous ces appels d’API depuis le client POSTMAN ou en utilisant les commandes `curl`.

### <a name="enable-the-ipsec-vpn-service-on-the-logical-router"></a>Activer le service VPN IPSec sur le routeur logique

```
POST   https://192.168.110.201/api/v1/vpn/ipsec/services/
{
"resource_type": "IPSecVPNService",
"description": "Manage VPN service",
"display_name": "IPSec VPN service",
"logical_router_id": "Logical-router ID",
"ike_log_level": "INFO",
"enabled": true
}
```

### <a name="create-profiles-ike"></a>Créer des profils : IKE

```
POST https://192.168.110.201/api/v1/vpn/ipsec/ike-profiles
 
{
"resource_type": "IPSecVPNIKEProfile",
"description": "IKEProfile for siteA",
"display_name": "IKEProfile siteA",
"encryption_algorithms": ["AES_128"],
"ike_version": "IKE_V2",
"digest_algorithms": ["SHA2_256"],
"sa_life_time":21600,
"dh_groups": ["GROUP14"]
}
```

### <a name="create-profiles-dpd"></a>Créer des profils : DPD

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/dpd-profiles  

{
"resource_type": "IPSecVPNDPDProfile",
"display_name": "nsx-default-dpd-profile",
"enabled": true
}
```

### <a name="create-profiles-tunnel"></a>Créer des profils : Tunnel

```
POST  https://192.168.110.201/api/v1/vpn/ipsec/tunnel-profiles
 
{
"resource_type": "IPSecVPNTunnelProfile",
"display_name": "nsx-default-tunnel-profile",
"enable_perfect_forward_secrecy": true,
"encryption_algorithms": ["AES_GCM_128"],
"digest_algorithms": [],
"sa_life_time":3600,
"dh_groups": ["GROUP14"],
"encapsulation_mode": "TUNNEL_MODE",
"transform_protocol": "ESP",
"df_policy": "COPY"
}
```

### <a name="create-a-local-endpoint"></a>Créer un point de terminaison local

``` 
POST https://192.168.110.201/api/v1/vpn/ipsec/local-endpoints
 
{
"resource_type": "IPSecVPNLocalEndpoint",
"description": "Local endpoint",
"display_name": "Local endpoint",
"local_id": "<Public IP of Loopback interface>",
"ipsec_vpn_service_id": {
"target_id": "IPSec VPN service ID"},
"local_address": "<IP of Loopback interface>",
"trust_ca_ids": [],
"trust_crl_ids": []
}
```

### <a name="create-a-peer-endpoint"></a>Créer un point de terminaison homologue

```
POST https://192.168.110.201/api/v1/vpn/ipsec/peer-endpoints

{
"resource_type": "IPSecVPNPeerEndpoint",
"description": "Peer endpoint for site B",
"display_name": "Peer endpoint for site B",
"connection_initiation_mode": "INITIATOR",
"authentication_mode": "PSK",
"ipsec_tunnel_profile_id": "IPSec Tunnel profile ID",
"dpd_profile_id": "DPD profile ID",
"psk":"nsx",
"ike_profile_id": "IKE profile ID",
"peer_address": "<Public IP of Standalone client",
"peer_id": "<Public IP of Standalone client>"
}
```

### <a name="create-a-route-based-vpn-session"></a>Créer une session VPN basée sur itinéraires

```
POST :  https://192.168.110.201/api/v1/vpn/ipsec/sessions
 
{
"resource_type": "RouteBasedIPSecVPNSession",
"peer_endpoint_id": "Peer Endpoint ID",
"ipsec_vpn_service_id": "IPSec VPN service ID",
"local_endpoint_id": "Local Endpoint ID",
"enabled": true,
"tunnel_ports": [
{
"ip_subnets": [
{
"ip_addresses": [
 "5.5.5.1"
],
"prefix_length": 24
}
  ]
}
]
}
```

## <a name="configure-l2vpn-on-nsx-t-tier0-router"></a>Configurer L2VPN sur le routeur NSX-T Tier0

Renseignez les informations suivantes après chaque appel POST. Les identifiants sont requis dans les appels POST ultérieurs.

```
L2VPN Service ID:
L2VPN Session ID:
Logical Port ID:
```

### <a name="create-the-l2vpn-service"></a>Créer le service L2VPNC

La sortie de la commande GET suivante sera vide car la configuration n'est pas encore terminée.

```
GET : https://192.168.110.201/api/v1/vpn/l2vpn/services
```

Pour la commande POST suivante, l'ID du routeur logique est l'UUID du routeur logique Tier0 DR obtenu précédemment.

```
POST : https://192.168.110.201/api/v1/vpn/l2vpn/services

{
"logical_router_id": "Logical Router ID",
"enable_full_mesh" : true
}
```

### <a name="create-the-l2vpn-session"></a>Créer la session L2VPN

Pour la commande POST suivante, l'ID de service L2VPN est l'ID que vous venez d'obtenir et l'ID de session VPN IPsec est celui obtenu dans la section précédente.

```    
POST: https://192.168.110.201/api/v1/vpn/l2vpn/sessions

{
"l2vpn_service_id" : "L2VPN service ID",
"transport_tunnels" : [
    {
    "target_id" : "IPSec VPN session ID"
    }]
}
```

Ces appels créent un point de terminaison de tunnel GRE. Pour vérifier l’état, exécutez la commande suivante.

```
edge-2> get tunnel-port
Tunnel      : 44648dae-8566-5bc9-a065-b1c4e5c3e03f
IFUID       : 328
LOCAL       : 169.254.64.1
REMOTE      : 169.254.64.2
ENCAP       : GRE

Tunnel      : cf950ca1-5cf8-5438-9b1a-d2c8c8e7229b
IFUID       : 318
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.152
ENCAP       : GENEVE

Tunnel      : 63639321-87c5-529e-8a61-92c1939799b2
IFUID       : 304
LOCAL       : 192.168.140.155
REMOTE      : 192.168.140.156
ENCAP       : GENEVE
```

### <a name="create-logical-port-with-the-tunnel-id-specified"></a>Créer un port logique avec l'ID de tunnel spécifié

```
    POST https://192.168.110.201/api/v1/logical-ports/

{
"resource_type": "LogicalPort",
"display_name": "Extend logicalSwitch, port for service",
"logical_switch_id": "Logical switch ID",
"admin_state" : "UP",
"attachment": {
"attachment_type":"L2VPN_SESSION",
"id":"L2VPN session ID",
"context" : {
"resource_type" : "L2VpnAttachmentContext",
    "tunnel_id" : 10
}
    }
        }
```

## <a name="obtain-the-peer-code-for-l2vpn-on-the-nsx-t-side"></a>Obtenir le code homologue pour L2VPN côté NSX-T

Obtenez le code homologue du point de terminaison NSX-T. Le code homologue est requis lors de la configuration du point de terminaison distant. La valeur L2VPN <session-id> peut être obtenue à partir de la section précédente. Pour plus d’informations, consultez le [Guide de l’API NSX-T 2.3](https://www.vmware.com/support/nsxt/doc/nsxt_23_api.html).

```
GET https://192.168.110.201/api/v1/vpn/l2vpn/sessions/<session-id>/peer-codes
```

## <a name="deploy-the-nsx-t-standalone-client-on-premises"></a>Déployer le client autonome NSX-T (en local)

Avant le déploiement, vérifiez que vos règles de pare-feu locales autorisent le trafic UDP 500/4500 entrant et sortant de/vers l'adresse IP publique CloudSimple précédemment réservée à l'interface de bouclage du routeur NSX-T T0. 

1. [Téléchargez le client autonome NSX Edge](https://my.vmware.com/group/vmware/details?productId=673&rPId=33945&downloadGroup=NSX-T-230) OVF et extrayez les fichiers du bundle téléchargé dans un dossier.

    ![Télécharger le client autonome NSX Edge](media/l2vpn-deploy-client01.png)

2. Ouvrez le dossier contenant tous les fichiers extraits. Sélectionnez tous les disques de machines virtuelles (NSX-l2t-client-large.mf et NSX-l2t-client-large.ovf pour les appliances de grande taille, ou NSX-l2t-client-Xlarge.mf et NSX-l2t-client-Xlarge.ovf pour les appliances de très grande taille). Cliquez sur **Suivant** .

    ![Sélectionner le modèle](media/l2vpn-deploy-client02.png) ![Capture d’écran montrant les fichiers VMDK sélectionnés.](media/l2vpn-deploy-client03.png)

3. Entrez un nom pour le client autonome NSX-T, puis cliquez sur **Suivant** .

    ![Entrer le nom du modèle](media/l2vpn-deploy-client04.png)

4. Le cas échéant, cliquez sur **Suivant** pour accéder aux paramètres du magasin de données. Sélectionnez le magasin de données approprié pour le client autonome NSX-T, puis cliquez sur **Suivant** .

    ![Sélectionner la banque de données](media/l2vpn-deploy-client06.png)

5. Sélectionnez les groupes de ports corrects pour l’interface Trunk (Trunk PG), Public (Uplink PG) et HA (Uplink PG) pour le client autonome NSX-T. Cliquez sur **Suivant** .

    ![Sélectionner des groupes de ports](media/l2vpn-deploy-client07.png)

6. Renseignez les détails suivants dans l'écran **Customize template (Personnaliser le modèle)** , puis cliquez sur **Next (Suivant)**  :

    Développez L2T :

    * **Adresse de l’homologue** . Entrez l'adresse IP réservée sur le portail Azure CloudSimple pour l'interface de bouclage NSX-T Tier0.
    * **Code de l’homologue** . Collez le code homologue obtenu lors de la dernière étape du déploiement du serveur L2VPN.
    * **VLAN d’interfaces secondaires (ID de tunnel)** . Entrez l'ID du VLAN à étendre. Entre parenthèses (), entrez l'ID du tunnel qui a été configuré précédemment.

    Développez l’interface de la liaison montante :

    * **Adresse IP du DNS** . Entrez l'adresse IP DNS locale.
    * **Passerelle par défaut** .  Entrez la passerelle par défaut du VLAN qui servira de passerelle par défaut pour ce client.
    * **Adresse IP** . Entrez l'adresse IP de liaison montante du client autonome.
    * **Longueur du préfixe** . Entrez la longueur du préfixe du VLAN/sous-réseau de liaison montante.
    * **CLI admin/enable/root User Password** . Définissez le mot de passe pour le compte admin /enable /root.

      ![Personnaliser le modèle](media/l2vpn-deploy-client08.png)
      ![Personnaliser le modèle - plus](media/l2vpn-deploy-client09.png)

7. Vérifiez les paramètres, puis cliquez sur **Terminer** .

    ![Terminer la configuration](media/l2vpn-deploy-client10.png)

## <a name="configure-an-on-premises-sink-port"></a>Configurer un port récepteur local

Si NSX n’est pas déployé sur l'un des sites VPN, vous pouvez configurer un VPN L2 en déployant un client NSX Edge autonome sur ce site. Un client NSX Edge autonome est déployé en utilisant un fichier OVF sur un hôte qui n'est pas géré par NSX. Cette méthode déploie une appliance NSX Edge Services Gateway qui fonctionnera comme un client VPN L2.

Si une carte virtuelle de jonction autonome est connectée à un commutateur vSphere Distributed Switch, le mode Promiscuous ou un port récepteur est nécessaire pour VPN L2. L'utilisation du mode Promiscuous peut entraîner des pings et des réponses en double. Pour cette raison, utilisez le mode port récepteur dans la configuration autonome NSX Edge de VPN L2. Voir la section [Configure a sink port (Configurer un port récepteur)](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.admin.doc/GUID-3CDA4346-E692-4592-8796-ACBEEC87C161.html) dans la documentation VMware.

## <a name="ipsec-vpn-and-l2vpn-verification"></a>Vérification IPsec VPN et L2VPN

Utilisez les commandes suivantes pour vérifier les sessions IPsec et L2VPN du routeur autonome NSX-T Edge.

```
nsx-l2t-edge> show service ipsec
-----------------------------------------------------------------------
vShield Edge IPSec Service Status:
IPSec Server is running.
AESNI is enabled.
Total Sites: 1, 1 UP, 0 Down
Total Tunnels: 1, 1 UP, 0 Down
----------------------------------
Site:  10.250.0.111_0.0.0.0/0-104.40.21.81_0.0.0.0/0
Channel: PeerIp: 104.40.21.81    LocalIP: 10.250.0.111  Version: IKEv2  Status: UP
Tunnel: PeerSubnet: 0.0.0.0/0    LocalSubnet: 0.0.0.0/0   Status: UP
----------------------------------
```

```
nsx-l2t-edge> show service l2vpn
L2 VPN is running
----------------------------------------
L2 VPN type: Client/Spoke

SITENAME                       IPSECSTATUS          VTI                  GRE
1ecb00fb-a538-4740-b788-c9049e8cb6c6 UP                   vti-100              l2t-1
```

Utilisez les commandes suivantes pour vérifier les sessions IPsec et L2VPN du routeur NSX-T Tier0.

```
edge-2> get ipsecvpn session
Total Number of Sessions: 1

IKE Session ID : 3
UUID           : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Type           : Route

Local IP       : 192.168.254.254      Peer IP        : 192.227.85.167
Local ID       : 104.40.21.81         Peer ID        : 192.227.85.167
Session Status : Up

Policy Rules
    VTI UUID       : 4bf96e3b-e50b-49cc-a16e-43a6390e3d53
    ToRule ID      : 560874406            FromRule ID    : 2708358054
    Local Subnet   : 0.0.0.0/0            Peer Subnet    : 0.0.0.0/0
    Tunnel Status  : Up
```

```
edge-2> get l2vpn session
Session       : f7147137-5dd0-47fe-9e53-fdc2b687b160
Tunnel        : b026095b-98c8-5932-96ff-dda922ffe316
IPSEC Session : 1ecb00fb-a538-4740-b788-c9049e8cb6c6
Status        : UP
```

Utilisez les commandes suivantes pour vérifier le port récepteur sur l'hôte ESXi où la machine virtuelle client autonome NSX-T réside dans l'environnement local.

```
 [root@esxi02:~] esxcfg-vswitch -l |grep NSX
  53                  1           NSXT-client-large.eth2
  225                1           NSXT-client-large.eth1
  52                  1           NSXT-client-large.eth0
```

```
[root@esxi02:~] net-dvs -l | grep "port\ [0-9]\|SINK\|com.vmware.common.alias"
                com.vmware.common.alias = csmlab2DS ,   propType = CONFIG
        port 24:
        port 25:
        port 26:
        port 27:
        port 13:
        port 19:
        port 169:
        port 54:
        port 110:
        port 6:
        port 107:
        port 4:
        port 199:
        port 168:
        port 201:
        port 0:
        port 49:
        port 53:
        port 225:
                com.vmware.etherswitch.port.extraEthFRP =   SINK
        port 52:
```
