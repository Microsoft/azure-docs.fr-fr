---
title: Azure VMware Solution by CloudSimple – Configurer une connexion haute disponibilité entre un site local et une passerelle VPN CloudSimple
description: Décrit comment configurer une connexion haute disponibilité entre votre environnement local et une passerelle VPN CloudSimple haute disponibilité
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 6e3118814eacc6cc63b5db59bd7f1877c1d347dc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025263"
---
# <a name="configure-a-high-availability-connection-from-on-premises-to-cloudsimple-vpn-gateway"></a>Configurer une connexion haute disponibilité entre un site local et une passerelle VPN CloudSimple

Les administrateurs réseau peuvent configurer une connexion VPN de site à site IPsec haute disponibilité entre leur environnement local et une passerelle VPN CloudSimple.

Ce guide présente les étapes de configuration d’un pare-feu local pour une connexion VPN de site à site IPsec haute disponibilité. Les étapes détaillées sont spécifiques du type de pare-feu local. En guise d’exemples, ce guide présente les étapes pour deux types de pare-feu : Cisco ASA et Palo Alto Networks.

## <a name="before-you-begin"></a>Avant de commencer

Avant de configurer le pare-feu local, accomplissez les tâches suivantes.

1. Vérifiez que votre organisation a [approvisionné](create-nodes.md) les nœuds requis et créé au moins un cloud privé CloudSimple.
2. [Configurez une passerelle VPN de site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre votre réseau local et votre cloud privé CloudSimple.

Consultez [Vue d’ensemble des passerelles VPN](cloudsimple-vpn-gateways.md) pour connaître les propositions prises en charge des phases 1 et 2.

## <a name="configure-on-premises-cisco-asa-firewall"></a>Configurer un pare-feu Cisco ASA local

Les instructions de cette section s’appliquent à Cisco ASA versions 8.4 et ultérieures. Dans l’exemple de configuration, le logiciel Cisco Adaptive Security Appliance (ASA) version 9.10 est déployé et configuré en mode IKEv1.

Pour que le réseau VPN de site à site fonctionne, vous devez autoriser les protocoles UDP 500/4500 et ESP (protocole IP 50) des adresses IP publiques principale et secondaire de CloudSimple (adresses IP homologues) sur l’interface externe de la passerelle VPN Cisco ASA locale.

### <a name="1-configure-phase-1-ikev1"></a>1. Configurer la phase 1 (IKEv1)

Pour activer la phase 1 (IKEv1) sur l’interface externe, entrez la commande CLI suivante dans le pare-feu Cisco ASA.

```crypto ikev1 enable outside```

### <a name="2-create-an-ikev1-policy"></a>2. Créer une stratégie IKEv1

Créez une stratégie IKEv1 qui définit les algorithmes et les méthodes à utiliser pour le hachage, l’authentification, le groupe Diffie-Hellman, la durée de vie et le chiffrement.

```
crypto ikev1 policy 1
authentication pre-share
encryption aes-256
hash sha
group 2
lifetime 28800
```

### <a name="3-create-a-tunnel-group"></a>3. Créer un groupe de tunnels

Créez un groupe de tunnels sous les attributs IPsec. Configurez l’adresse IP de l’homologue et la clé prépartagée de tunnel, que vous avez définies lors de la [configuration de votre passerelle VPN de site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway).

```
tunnel-group <primary peer ip> type ipsec-l2l
tunnel-group <primary peer ip> ipsec-attributes
ikev1 pre-shared-key *****

tunnel-group <secondary peer ip> type ipsec-l2l
tunnel-group <secondary peer ip> ipsec-attributes
ikev1 pre-shared-key *****
```

### <a name="4-configure-phase-2-ipsec"></a>4. Configurer la phase 2 (IPsec)

Pour configurer la phase 2 (IPsec), créez une liste de contrôle d’accès (ACL) qui définit le trafic à chiffrer et à acheminer par tunnel. Dans l’exemple suivant, le trafic d’intérêt provient du tunnel qui va du sous-réseau local sur site (10.16.1.0/24) au sous-réseau distant du cloud privé (192.168.0.0/24). La liste de contrôle d’accès peut contenir plusieurs entrées s’il y a plusieurs sous-réseaux entre les sites.

Dans les versions Cisco ASA 8.4 et ultérieures, il est possible de créer des objets ou groupes d’objets qui servent de conteneurs pour les réseaux, les sous-réseaux, les adresses IP hôtes ou plusieurs objets. Créez un objet à usage local et un objet pour les sous-réseaux distants, et utilisez-les pour la liste de contrôle d’accès de chiffrement et les instructions NAT.

#### <a name="define-an-on-premises-local-subnet-as-an-object"></a>Définir un sous-réseau local sus site en tant qu’objet

```
object network AZ_inside
subnet 10.16.1.0 255.255.255.0
```

#### <a name="define-the-cloudsimple-remote-subnet-as-an-object"></a>Définir le sous-réseau distant CloudSimple en tant qu’objet

```
object network CS_inside
subnet 192.168.0.0 255.255.255.0
```

#### <a name="configure-an-access-list-for-the-traffic-of-interest"></a>Configurer une liste d’accès pour le trafic d’intérêt

```
access-list ipsec-acl extended permit ip object AZ_inside object CS_inside
```

### <a name="5-configure-the-transform-set"></a>5. Configurer le jeu de transformations

Configurez le jeu de transformations qui doit inclure le mot clé ```ikev1```. Les attributs de chiffrement et de hachage spécifiés dans le jeu de transformations doivent correspondre aux paramètres répertoriés dans la [Configuration par défaut pour les passerelles VPN CloudSimple](cloudsimple-vpn-gateways.md).

```
crypto ipsec ikev1 transform-set devtest39 esp-aes-256 esp-sha-hmac 
```

### <a name="6-configure-the-crypto-map"></a>6. Configurer le mappage de chiffrement

Configurez le mappage de chiffrement qui contient les composants suivants :

* Adresse IP de l’homologue
* Liste de contrôle d’accès définie contenant le trafic d’intérêt
* Jeu de transformations

```
crypto map mymap 1 set peer <primary peer ip> <secondary peer ip>
crypto map mymap 1 match address ipsec-acl
crypto map mymap 1 set ikev1 transform-set devtest39
```

### <a name="7-apply-the-crypto-map"></a>7. Appliquer le mappage de chiffrement

Appliquer le mappage de chiffrement sur l’interface externe :

```crypto map mymap interface outside```

### <a name="8-confirm-applicable-nat-rules"></a>8. Confirmer les règles NAT applicables

Voici la règle NAT utilisée. Assurez-vous que le trafic VPN n’est soumis à aucune autre règle NAT.

```nat (inside,outside) source static AZ_inside AZ_inside destination static CS_inside CS_inside```

### <a name="sample-ipsec-site-to-site-vpn-established-output-from-cisco-asa"></a>Exemple de sortie établie de VPN de site à site IPsec à partir de Cisco ASA

Sortie de la phase 1 :

![Sortie de la phase 1 pour le pare-feu Cisco ASA](media/ha-vpn-connection-cisco-phase1.png)

Sortie de la phase 2 :

![Sortie de la phase 2 pour le pare-feu Cisco ASA](media/ha-vpn-connection-cisco-phase2.png)

## <a name="configure-on-premises-palo-alto-networks-firewall"></a>Configurer un pare-feu Palo Alto Networks local

Les instructions de cette section s’appliquent à Palo Alto Networks versions 7 1 et ultérieures. Dans cet exemple de configuration, la version logicielle de série de machine virtuelle Palo Alto Networks 8.1.0 est déployée et configurée en mode IKEv1.

Pour que le réseau VPN de site à site fonctionne, vous devez autoriser les protocoles UDP 500/4500 et ESP (protocole IP 50) des adresses IP publiques principale et secondaire de CloudSimple (adresses IP homologues) sur l’interface externe de la passerelle Palo Alto Networks locale.

### <a name="1-create-primary-and-secondary-tunnel-interfaces"></a>1. Créer des interfaces de tunnel principale et secondaire

Connectez-vous au pare-feu Palo Alto, sélectionnez **Réseau** > **Interfaces** > **Tunnel** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

* Nom de l’interface. Le premier champ est renseigné automatiquement avec le mot clé « tunnel ». Dans le champ adjacent, entrez un nombre compris entre 1 et 9999. Cette interface sera utilisée comme interface du tunnel principal pour transporter le trafic de site à site entre le centre de données local et le cloud privé.
* Commentaire. Entrez des commentaires pour faciliter l’identification de l’objectif du tunnel.
* Profil NetFlow. Conservez la valeur par défaut.
* Config. Attribuer l’interface à : Routeur virtuel : Sélectionnez **par défaut**. 
        Zone de sécurité : Sélectionnez la zone pour le trafic LAN approuvé. Dans cet exemple, le nom de la zone pour le trafic LAN est « Trust ».
* IPv4. Cliquez sur **Ajouter**, puis ajoutez toute adresse IP /32 inutilisée non redondante dans votre environnement, qui sera affectée à l’interface du tunnel principal et utilisée pour surveiller les tunnels (comme expliqué plus loin).

Étant donné que cette configuration est destinée à un VPN haute disponibilité, deux interfaces de tunnel sont requises : Une principale et une secondaire. Répétez les étapes précédentes pour créer l’interface du tunnel secondaire. Sélectionnez un autre ID de tunnel et une autre adresse IP /32 inutilisée.

### <a name="2-set-up-static-routes-for-private-cloud-subnets-to-be-reached-over-the-site-to-site-vpn"></a>2. Configurer des itinéraires statiques pour les sous-réseaux de cloud privé à atteindre sur le VPN de site à site

Des itinéraires sont nécessaires pour permettre aux sous-réseaux locaux d’atteindre les sous-réseaux de cloud privé CloudSimple.

Sélectionnez **Réseau** > **Routeurs virtuel** > *par défaut* > **Itinéraires statiques** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

* Nom. Entrez un nom pour faciliter l’identification de l’objectif de l’itinéraire.
* Destination. Spécifiez les sous-réseaux de cloud privé CloudSimple à atteindre via les interfaces de tunnel S2S à partir du site local.
* Interface. Sélectionnez l’interface du tunnel principal créée à l’étape 1 (section 2) dans la liste déroulante. Dans cet exemple, il s’agit de tunnel.20.
* Tronçon suivant. Sélectionnez **Aucun**.
* Distance d’administration. Conservez la valeur par défaut.
* Métrique. Entrez une valeur comprise entre 1 et 65535. La clé consiste à entrer la métrique inférieure de l’itinéraire correspondant à l’interface du tunnel principal par rapport à l’interface du tunnel secondaire correspondante de l’itinéraire, en privilégiant l’itinéraire précédent. Si tunnel.20 a la valeur de métrique 20 par rapport une valeur de métrique 30 pour tunnel.30, tunnel.20 est préféré.
* Table de routage. Conservez la valeur par défaut.
* Profil BFD. Conservez la valeur par défaut.
* Surveillance du chemin. Laissez la case non activée.

Répétez les étapes précédentes pour créer un autre itinéraire pour les sous-réseaux de cloud privé à utiliser comme itinéraire secondaire/de secours via une interface de tunnel secondaire. Cette fois, sélectionnez un autre ID de tunnel et une métrique supérieure à celle de l’itinéraire principal.

### <a name="3-define-the-cryptographic-profile"></a>3. Définir le profil de chiffrement

Définissez un profil de chiffrement qui spécifie les protocoles et algorithmes pour l’identification, l’authentification et le chiffrement à utiliser pour la configuration de tunnels VPN dans IKEv1 Phase 1.

Sélectionnez **Réseau** > **Développer les profils réseau** > **Chiffrement IKE** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

* Nom. Entrez le nom pour le profil de chiffrement IKE.
* Groupe DH. Cliquez sur **Ajouter** et sélectionnez le groupe DH approprié.
* Chiffrement. Cliquez sur **Ajouter** et sélectionnez la méthode de chiffrement appropriée.
* Authentification. Cliquez sur **Ajouter** et sélectionnez la méthode d’authentification appropriée.
* Durée de vie de la clé. Conservez la valeur par défaut.
* Authentification IKEv2 multiple. Conservez la valeur par défaut.

### <a name="4-define-ike-gateways"></a>4. Définir des passerelles IKE

Définissez des passerelles IKE pour établir la communication entre les homologues à chaque extrémité du tunnel VPN.

Sélectionnez **Réseau** > **Développer les profils réseau** > **Passerelles IKE** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

Onglet Général :

* Nom. Entrez le nom de la passerelle IKE à homologuer avec l’homologue de VPN CloudSimple principal.
* Version. Sélectionnez **Mode IKEv1 uniquement**.
* Type d’adresse. Sélectionnez **IPv4**.
* Interface. Sélectionnez l’interface publique ou externe.
* Adresse IP locale. Conservez la valeur par défaut.
* Type d’adresse IP d’homologue. Sélectionnez **IP**.
* Adresse de l’homologue. Entrez l’adresse IP d’homologue du VPN CloudSimple principal.
* Authentification. Sélectionnez **Clé prépartagée**.
* Clé prépartagée/Confirmer la clé prépartagée. Entrez la clé prépartagée correspondant à la clé de passerelle VPN CloudSimple.
* Identification locale. Entrez l’adresse IP publique du pare-feu Palo Alto local.
* Identification d’homologue. Entrez l’adresse IP d’homologue du VPN CloudSimple principal.

Onglet Options avancées :

* Activez le mode passif. Laissez la case non activée.
* Activez NAT Traversal. Laissez la case désactivée si le pare-feu Palo Alto local n’est pas derrière un appareil NAT. Autrement, activez la case à cocher.

IKEv1 :

* Mode Exchange. Sélectionnez **principal**.
* Profil de chiffrement IKE. Sélectionnez le profil de chiffrement IKE que vous avez créé précédemment. N’activez pas la case à cocher Activer la fragmentation.
* Détection d’homologue mort. Laissez la case désactivée.

Répétez les étapes précédentes pour créer la passerelle IKE secondaire.

### <a name="5-define-ipsec-crypto-profiles"></a>5. Définir des profils de chiffrement IPSEC

Sélectionnez **Réseau** > **Développer les profils réseau** > **Chiffrement IPSEC** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

* Nom. Entrez un nom pour le profil de chiffrement IPsec.
* Protocole IPsec. Sélectionnez **ESP**.
* Chiffrement. Cliquez sur **Ajouter** et sélectionnez la méthode de chiffrement appropriée.
* Authentification. Cliquez sur **Ajouter** et sélectionnez la méthode d’authentification appropriée.
* Groupe DH. Sélectionnez **non-PFS**.
* Durée de vie. Définissez sur 30 minutes.
* Activer. Laissez la case désactivée.

Répétez les étapes précédentes pour créer un autre profil de chiffrement IPsec, qui sera utilisé comme homologue de VPN CloudSimple secondaire. Le même profil de chiffrement IPSEC peut également être utilisé pour les tunnels IPsec principal et secondaire (voir la procédure suivante).

### <a name="6-define-monitor-profiles-for-tunnel-monitoring"></a>6. Définir des profils de surveillance pour la surveillance du tunnel

Sélectionnez **Réseau** > **Développer les profils réseau** > **Surveiller** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

* Nom. Entrez un nom pour le profil de surveillance à utiliser pour la surveillance du tunnel pour une réaction proactive à la défaillance.
* Action. Sélectionnez **Basculer**.
* Intervalle. Entrez la valeur **3**.
* Seuil. Entrez la valeur **7**.

### <a name="7-set-up-primary-and-secondary-ipsec-tunnels"></a>7. Configurer des tunnels IPsec principal et secondaire.

Sélectionnez **Réseau** > **Tunnels IPsec** > **Ajouter**, configurez les champs suivants, puis cliquez sur **OK**.

Onglet Général :

* Nom. Entrez un nom pour le tunnel IPSEC principal à appairer avec l’homologue de VPN CloudSimple principal.
* Interface du tunnel. Sélectionnez l’interface du tunnel principal.
* Type. Conservez la valeur par défaut.
* Type d’adresse. Sélectionnez **IPv4**.
* Passerelle IKE. Sélectionnez la passerelle IKE principale.
* Profil de chiffrement IPsec. Sélectionnez le profil IPsec principal. Sélectionnez **Afficher les options avancées**.
* Activez la protection de la relecture. Conservez la valeur par défaut.
* Copier l’en-tête TOS. Laissez la case désactivée.
* Moniteur de tunnel. Activez la case.
* Adresse IP de destination. Entrez une adresse IP appartenant au sous-réseau de cloud privé CloudSimple autorisée sur la connexion de site à site. Assurez-vous que les interfaces de tunnel (telles que tunnel.20 – 10.64.5.2/32 et tunnel.30 – 10.64.6.2/32) sur Palo Alto sont autorisées à atteindre l’adresse IP du cloud privé CloudSimple via le VPN de site à site. Consultez la configuration suivante pour les ID de proxy.
* Profil. Sélectionnez le profil du moniteur.

Onglet ID de proxy : Cliquez sur **IPv4** > **Ajouter**, puis configurez les éléments suivants :

* ID du proxy. Entrez un nom pour le trafic intéressant. Plusieurs ID de proxy peuvent être transportés dans un seul tunnel IPsec.
* Local. Spécifiez les sous-réseaux locaux sur site qui sont autorisés à communiquer avec les sous-réseaux de cloud privé via le VPN de site à site.
* Distant. Spécifiez les sous-réseaux distants de cloud privé autorisés à communiquer avec les sous-réseaux locaux.
* Protocole. Sélectionnez **Tout**.

Répétez les étapes précédentes pour créer un autre tunnel IPsec à utiliser pour l’homologue de VPN CloudSimple secondaire.

## <a name="references"></a>References

Configuration de NAT sur Cisco ASA :

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa84/configuration/guide/asa_84_cli_config/nat_objects.html" target="_blank">Guide de configuration de la série Cisco ASA 5500</a>

Attributs IKEv1 et IKEv2 pris en charge sur Cisco ASA :

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa90/configuration/guide/asa_90_cli_config/vpn_ike.html#21661" target="_blank">Guide de configuration de l’interface de ligne de commande de la série Cisco ASA</a>

Configuration de VPN de site à site IPsec sur Cisco ASA avec les versions 8.4 et ultérieures :

<a href="https://www.cisco.com/c/en/us/support/docs/security/asa-5500-x-series-next-generation-firewalls/119141-configure-asa-00.html#anc8" target="_blank">Configurer des tunnels de site à site IPsec IKEv1 avec ASDM ou CLI sur l’ASA</a>

Configuration de l’appliance de sécurité adaptive virtuelle (ASAv) Cisco sur Azure :

<a href="https://www.cisco.com/c/en/us/td/docs/security/asa/asa96/asav/quick-start-book/asav-96-qsg/asav-azure.html" target="_blank">Guide de démarrage rapide de l’appliance de sécurité adaptive virtuelle (ASAv) Cisco</a>

Configuration d’un VPN de site à site avec des ID de proxy sur Palo Alto :

[Configurer un VPN de site à site](https://docs.paloaltonetworks.com/pan-os/9-0/pan-os-admin/vpns/set-up-site-to-site-vpn#)

Configuration de la supervision du tunnel :

[Configurer la surveillance du tunnel](https://docs.paloaltonetworks.com/pan-os/7-1/pan-os-admin/vpns/set-up-tunnel-monitoring.html)

Opérations de passerelle IKE ou de tunnel IPsec :

<a href="https://docs.paloaltonetworks.com/pan-os/9-0/pan-os-admin/vpns/set-up-site-to-site-vpn/enabledisable-refresh-or-restart-an-ike-gateway-or-ipsec-tunnel#" target="_blank">Activer/désactiver, actualiser ou redémarrer une passerelle IKE ou un tunnel IPsec</a>
