---
title: "Azure ExpressRoute : Exemples de configuration de routeur – Traduction d'adresses réseau"
description: Cette page fournit des exemples de configuration pour les routeurs Cisco et Juniper.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 12/06/2018
ms.author: duau
ms.openlocfilehash: 4b57dec32a3a008cadf68c19ea52353a23fb2cd3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89397492"
---
# <a name="router-configuration-samples-to-set-up-and-manage-nat"></a>Exemples de configuration de routeur pour configurer et gérer des NAT

Cette page fournit des exemples de configuration NAT pour les routeurs des séries Cisco ASA et Juniper SRX lors de l’utilisation d’ExpressRoute. Ces exemples sont fournis à titre indicatif uniquement et ne doivent pas être utilisés en l’état. Vous pouvez vous adresser au fournisseur pour rechercher les configurations adaptées à votre réseau.

> [!IMPORTANT]
> Les exemples de cette page sont fournis à titre purement indicatif. Vous devez vous adresser à l’équipe commerciale/technique de votre fournisseur et à votre équipe de mise en réseau pour rechercher les configurations adaptées à vos besoins. Microsoft ne prend pas en charge les problèmes liés aux configurations répertoriées dans cette page. Vous devez contacter le fournisseur de votre périphérique pour la prise en charge des problèmes.
> 
> 

* Les exemples de configuration de routeur ci-dessous s'appliquent aux peerings Azure Public et Microsoft. Vous ne devez pas configurer un système NAT pour le peering privé Azure. Pour plus d’informations, voir [Peerings ExpressRoute](expressroute-circuit-peerings.md) et [Configuration NAT requise pour ExpressRoute](expressroute-nat.md).

* Vous DEVEZ utiliser des pools IP NAT distincts pour la connectivité à Internet et ExpressRoute. L’utilisation du même pool IP NAT via internet et ExpressRoute entraîne un routage asymétrique et une perte de connectivité.


## <a name="cisco-asa-firewalls"></a>Pare-feu Cisco ASA
### <a name="pat-configuration-for-traffic-from-customer-network-to-microsoft"></a>Configuration PAT pour le trafic du réseau client à Microsoft

```console
object network MSFT-PAT
  range <SNAT-START-IP> <SNAT-END-IP>


object-group network MSFT-Range
  network-object <IP> <Subnet_Mask>

object-group network on-prem-range-1
  network-object <IP> <Subnet-Mask>

object-group network on-prem-range-2
  network-object <IP> <Subnet-Mask>

object-group network on-prem
  network-object object on-prem-range-1
  network-object object on-prem-range-2

nat (outside,inside) source dynamic on-prem pat-pool MSFT-PAT destination static MSFT-Range MSFT-Range
```

### <a name="pat-configuration-for-traffic-from-microsoft-to-customer-network"></a>Configuration PAT pour le trafic de Microsoft au réseau client

**Interfaces et sens :**

Interface source (où le trafic entre dans ASA) : dans l’interface de destination (où le trafic quitte ASA) : en dehors

**Configuration :**

Pool NAT :

```console
object network outbound-PAT
    host <NAT-IP>
```

Serveur cible :

```console
object network Customer-Network
    network-object <IP> <Subnet-Mask>
```

Groupe d’objets pour les adresses IP de clients :

```console
object-group network MSFT-Network-1
    network-object <MSFT-IP> <Subnet-Mask>

object-group network MSFT-PAT-Networks
    network-object object MSFT-Network-1
```

Commandes NAT :

```console
nat (inside,outside) source dynamic MSFT-PAT-Networks pat-pool outbound-PAT destination static Customer-Network Customer-Network
```


## <a name="juniper-srx-series-routers"></a>Routeurs de la série Juniper SRX
### <a name="1-create-redundant-ethernet-interfaces-for-the-cluster"></a>1. Création d’interfaces Ethernet redondantes pour le cluster

```console
    interfaces {
        reth0 {
            description "To Internal Network";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 1;
            }
            unit 100 {
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
        reth1 {
            description "To Microsoft via Edge Router";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 2;
            }
            unit 100 {
                description "To Microsoft via Edge Router";
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
    }
```

### <a name="2-create-two-security-zones"></a>2. Création de deux zones de sécurité
* Zone approuvée pour le réseau interne et zone non approuvée pour les routeurs accessibles depuis le réseau externe
* Affectation des interfaces appropriées aux zones
* Autorisation des services sur les interfaces

```console
    security {
        zones {
            security-zone Trust {
                host-inbound-traffic {
                    system-services {
                        ping;
                    }
                    protocols {
                        bgp;
                    }
                }
                interfaces {
                    reth0.100;
                }
            }
            security-zone Untrust {
                host-inbound-traffic {
                    system-services {
                        ping;
                    }
                    protocols {
                        bgp;
                    }
                }
                interfaces {
                    reth1.100;
                }
            }
        }
    }
```


### <a name="3-create-security-policies-between-zones"></a>3. Création des stratégies de sécurité entre les zones

```console
    security {
        policies {
            from-zone Trust to-zone Untrust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
            from-zone Untrust to-zone Trust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
        }
    }
```

### <a name="4-configure-nat-policies"></a>4. Configuration des stratégies NAT
* Créez deux pools NAT. Un pool servira au trafic NAT sortant vers Microsoft et un autre au trafic de Microsoft vers le client.
* Création des règles NAT pour le trafic respectif

```console
       security {
           nat {
               source {
                   pool SNAT-To-ExpressRoute {
                       routing-instance {
                           External-ExpressRoute;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   pool SNAT-From-ExpressRoute {
                       routing-instance {
                           Internal;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   rule-set Outbound_NAT {
                       from routing-instance Internal;
                       to routing-instance External-ExpressRoute;
                       rule SNAT-Out {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-To-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
                   rule-set Inbound-NAT {
                       from routing-instance External-ExpressRoute;
                       to routing-instance Internal;
                       rule SNAT-In {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-From-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
               }
           }
       }
```

### <a name="5-configure-bgp-to-advertise-selective-prefixes-in-each-direction"></a>5. Configuration de BGP pour publier les préfixes sélectifs dans chaque direction
Consultez les exemples de la page [Exemples de configuration de routage ](expressroute-config-samples-routing.md).

### <a name="6-create-policies"></a>6. Création des stratégies

```console
    routing-options {
                  autonomous-system <Customer-ASN>;
    }
    policy-options {
        prefix-list Microsoft-Prefixes {
            <IP-Address/Subnet-Mask;
            <IP-Address/Subnet-Mask;
        }
        prefix-list private-ranges {
            10.0.0.0/8;
            172.16.0.0/12;
            192.168.0.0/16;
            100.64.0.0/10;
        }
        policy-statement Advertise-NAT-Pools {
            from {
                protocol static;
                route-filter <NAT-Pool-Address/Subnet-mask> prefix-length-range /32-/32;
            }
            then accept;
        }
        policy-statement Accept-from-Microsoft {
            term 1 {
                from {
                    instance External-ExpressRoute;
                    prefix-list-filter Microsoft-Prefixes orlonger;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
        policy-statement Accept-from-Internal {
            term no-private {
                from {
                    instance Internal;
                    prefix-list-filter private-ranges orlonger;
                }
                then reject;
            }
            term bgp {
                from {
                    instance Internal;
                    protocol bgp;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
    }
    routing-instances {
        Internal {
            instance-type virtual-router;
            interface reth0.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Microsoft;
            }
            protocols {
                bgp {
                    group customer {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-ASN-1>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
        External-ExpressRoute {
            instance-type virtual-router;
            interface reth1.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Internal;
            }
            protocols {
                bgp {
                    group edge-router {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-Public-ASN>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
    }
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez le [Forum Aux Questions sur ExpressRoute](expressroute-faqs.md) .

