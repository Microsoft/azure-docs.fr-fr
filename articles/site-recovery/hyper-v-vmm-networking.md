---
title: Configurer l’adressage IP après le basculement vers un site secondaire avec Azure Site Recovery
description: Décrit comment configurer l’adressage IP pour se connecter à des machines virtuelles dans un site local secondaire après une reprise d’activité et un basculement avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 43942c20a353ff69383f3e721679e4c95ab9d230
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87495941"
---
# <a name="set-up-ip-addressing-to-connect-to-a-secondary-on-premises-site-after-failover"></a>Configurer l’adressage IP pour se connecter à un site local secondaire après un basculement

Après avoir basculé les machines virtuelles Hyper-V de clouds VMM (System Center Virtual Machine Manager) sur un site secondaire, vous devez pouvoir vous connecter aux machines virtuelles de réplication. C’est ce que cet article va vous aider à faire. 

## <a name="connection-options"></a>Options de connexion

Après un basculement, il existe deux façons de gérer l’adressage IP des machines virtuelles de réplication : 

- **Conserver la même adresse IP après un basculement** : dans ce scénario, la machine virtuelle répliquée a la même adresse IP que la machine virtuelle principale. Les problèmes liés au réseau s’en trouvent simplifiés après le basculement, mais un travail sur l’infrastructure est nécessaire.
- **Utiliser une adresse IP différente après un basculement** : dans ce scénario, la machine virtuelle reçoit une nouvelle adresse IP après le basculement. 
 

## <a name="retain-the-ip-address"></a>Conserver l’adresse IP

Si vous souhaitez conserver les adresses IP du site principal après le basculement vers le site secondaire, utilisez l’une des solutions données ici.

- Déployer un sous-réseau étiré entre le site principal et le site secondaire.
- Effectuer un basculement de sous-réseau complet à partir du site principal vers le secondaire. Vous devez mettre à jour les routes pour indiquer le nouvel emplacement des adresses IP.


### <a name="deploy-a-stretched-subnet"></a>Déployer un sous-réseau étiré

Dans une configuration étirée, le sous-réseau est disponible simultanément sur le site principal et sur le secondaire. Dans un sous-réseau étiré, lorsque vous déplacez une machine et sa configuration d’adresse IP (couche 3) vers le site secondaire, le réseau achemine automatiquement le trafic vers le nouvel emplacement. 

- En ce qui concerne la couche 2 (couche de liaison de données), vous devez disposer d’un équipement réseau qui peut gérer un VLAN étiré.
- En étirant le VLAN, le domaine d’erreur potentielle s’étend aux deux sites. C’est alors un point de défaillance. Bien que peu probable, dans ce genre de scénario vous pouvez vous retrouver malgré tout dans l’incapacité d’isoler un incident tel qu’une tempête de diffusion. 


### <a name="fail-over-a-subnet"></a>Basculer un sous-réseau

Vous pouvez basculer tout le sous-réseau pour bénéficier des avantages d’un sous-réseau étiré, sans réellement l’étirer. Dans cette solution, le sous-réseau est disponible dans le site source ou cible, mais pas dans les deux simultanément.

- Pour conserver l’espace d’adressage IP en cas de basculement, vous pouvez mettre en place un programme pour que l’infrastructure du routeur déplace les sous-réseaux d’un site à un autre.
- Lorsqu’un basculement se produit, les sous-réseaux se déplacent avec leurs machines virtuelles associées.
- En cas de défaillance, et c’est là le principal inconvénient de cette stratégie, vous devez déplacer le sous-réseau dans son intégralité.

#### <a name="example"></a>Exemple

Voici un exemple de basculement de sous-réseau complet. 

- Avant le basculement, le site principal compte des applications qui s’exécutent dans le sous-réseau 192.168.1.0/24.
- Pendant le basculement, toutes les machines virtuelles de ce sous-réseau sont basculées sur le site secondaire et conservent leur adresse IP. 
- Les routes entre tous les sites doivent être modifiées pour refléter le fait que toutes les machines virtuelles du sous-réseau 192.168.1.0/24 ont désormais été déplacées sur le site secondaire.

Les graphiques suivants illustrent les sous-réseaux avant et après le basculement :


**Avant le basculement**

![Diagramme montrant les sous-réseaux avant le basculement.](./media/hyper-v-vmm-networking/network-design2.png)

**Après le basculement**

![Diagramme montrant les sous-réseaux après le basculement.](./media/hyper-v-vmm-networking/network-design3.png)

Après le basculement, Site Recovery attribue une adresse IP à chaque interface réseau présent sur la machine virtuelle. L’adresse est allouée depuis le pool d’adresses IP statiques du réseau approprié, pour chaque instance de machine virtuelle.

- Si le pool d’adresses IP sur le site secondaire est identique à celui du site source, Site Recovery alloue la même adresse IP (de la machine virtuelle source) à la machine virtuelle de réplication. L’adresse IP est réservée dans VMM, mais n’est pas définie comme l’adresse IP de basculement sur l’hôte Hyper-V. L’adresse IP de basculement sur un hôte Hyper-V est définie juste avant le basculement.
- Si la même adresse IP n'est pas disponible, Site Recovery alloue une autre adresse IP disponible du pool.
- Si les machines virtuelles utilisent DHCP, Site Recovery ne gère pas les adresses IP. Vous devez vérifier que le serveur DHCP sur le site secondaire peut allouer des adresses provenant de la même plage que celle du site source.

### <a name="validate-the-ip-address"></a>Valider l’adresse IP

Une fois la protection activée pour une machine virtuelle, vous pouvez utiliser l’exemple de script suivant pour vérifier l’adresse attribuée à la machine virtuelle. Cette adresse IP est définie comme adresse IP de basculement, et est attribuée à la machine virtuelle au moment du basculement :

```powershell
$vm = Get-SCVirtualMachine -Name <VM_NAME>
$na = $vm[0].VirtualNetworkAdapters>
$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
$ip.address
```

## <a name="use-a-different-ip-address"></a>Utiliser une autre adresse IP

Dans ce scénario, les adresses IP des machines virtuelles basculées sont modifiées. L’inconvénient de cette solution est la maintenance requise.  Il est possible que les entrées DNS et de cache doivent être mises à jour. Un temps d’arrêt peut en découler, ce qui peut être pallié comme suit :

- Utilisez des valeurs TTL faibles pour les applications intranet.
- Utilisez le script suivant dans un plan de récupération Site Recovery, pour une mise à jour en temps voulu du serveur DNS. Vous n’avez pas besoin du script si vous utilisez l’inscription DNS dynamique.

    ```powershell
    param(
    string]$Zone,
    [string]$name,
    [string]$IP
    )
    $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
    $newrecord = $record.clone()
    $newrecord.RecordData[0].IPv4Address  =  $IP
    Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord
    ```
    
### <a name="example"></a>Exemple 

Dans cet exemple, nous avons des adresses IP différentes entre le site principal et le site secondaire, sachant par ailleurs qu’il existe un troisième site depuis lequel il est possible d’accéder aux applications hébergées sur le site principal ou le site de récupération.

- Avant le basculement, les applications sont hébergées dans le sous-réseau 192.168.1.0/24 du site principal.
- Après le basculement, les applications sont configurées dans le sous-réseau 172.16.1.0/24 du site secondaire.
- Les trois sites peuvent accéder les uns aux autres.
- Après le basculement, les applications sont restaurées sur le sous-réseau de récupération.
- Dans ce scénario, il n’est pas nécessaire de basculer tout le sous-réseau et aucune modification n’est requise pour reconfigurer les itinéraires réseau ou VPN. Le basculement et certaines mises à jour DNS garantissent le maintien de l’accessibilité des applications.
- Si le système DNS est configuré pour autoriser des mises à jour dynamiques, les machines virtuelles peuvent alors s’inscrire à l’aide de la nouvelle adresse IP une fois qu’elles démarrent après un basculement.

**Avant le basculement**

![Diagramme montrant différentes adresses IP avant le basculement.](./media/hyper-v-vmm-networking/network-design10.png)

**Après le basculement**

![Diagramme montrant différentes adresses IP après le basculement.](./media/hyper-v-vmm-networking/network-design11.png)


## <a name="next-steps"></a>Étapes suivantes

[Exécuter un basculement](hyper-v-vmm-failover-failback.md)

