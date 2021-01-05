---
title: Résoudre les problèmes de connectivité du service NAT de réseau virtuel Azure
titleSuffix: Azure Virtual Network
description: Résoudre les problèmes liés au service NAT du réseau virtuel.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
Customer intent: As an IT administrator, I want to troubleshoot Virtual Network NAT.
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/20/2020
ms.author: allensu
ms.openlocfilehash: 690543ebc91e346e77509fbf993493f6978374ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87836103"
---
# <a name="troubleshoot-azure-virtual-network-nat-connectivity"></a>Résoudre les problèmes de connectivité du service NAT de réseau virtuel Azure

Cet article permet aux administrateurs de diagnostiquer et de résoudre les problèmes de connectivité lors de l’utilisation du service NAT du réseau virtuel.

## <a name="problems"></a>Problèmes

* [Épuisement des ressources SNAT](#snat-exhaustion)
* [Échec de la commande ping ICMP](#icmp-ping-is-failing)
* [Échecs de connectivité](#connectivity-failures)
* [Coexistence IPv6](#ipv6-coexistence)
* [La connexion ne provient pas de la ou des adresses IP de passerelle NAT](#connection-doesnt-originate-from-nat-gateway-ips)

Pour résoudre ces problèmes, effectuez les étapes de la section suivante.

## <a name="resolution"></a>Résolution

### <a name="snat-exhaustion"></a>Épuisement des ressources SNAT

Une seule [ressource de passerelle NAT](nat-gateway-resource.md) prend en charge entre 64 000 et 1 million de flux simultanés.  Chaque adresse IP fournit 64 000 ports SNAT à l’inventaire disponible. Vous pouvez utiliser jusqu’à 16 adresses IP par ressource de passerelle NAT.  Le mécanisme SNAT est décrit [ici](nat-gateway-resource.md#source-network-address-translation) de manière plus détaillée.

Souvent, la cause racine de l’épuisement des ressources SNAT est un anti-modèle du mode d’établissement et de gestion de la connectivité sortante, ou des minuteurs configurables dont la valeur par défaut a été changée.  Lisez attentivement cette section.

#### <a name="steps"></a>Étapes

1. Vérifiez si vous avez remplacé le délai d’inactivité par défaut par une valeur supérieure à 4 minutes.
2. Examinez comment votre application crée une connectivité sortante (par exemple, la révision de code ou la capture de paquets). 
3. Déterminez si cette activité est un comportement attendu ou si l’application ne fonctionne pas correctement.  Utilisez des [métriques](nat-metrics.md) dans Azure Monitor pour justifier vos découvertes. Utilisez la catégorie « Échec » pour la métrique Connexions SNAT.
4. Évaluez si les modèles appropriés sont suivis.
5. Évaluez si l’épuisement des ports SNAT doit être atténué avec des adresses IP supplémentaires affectées à la ressource de passerelle NAT.

#### <a name="design-patterns"></a>Modèles de conception

Tirez toujours parti de la réutilisation des connexions et du regroupement de connexions chaque fois que c’est possible.  Ces modèles évitent les problèmes d’épuisement des ressources et entraînent un comportement prévisible. Des primitives pour ces modèles sont disponibles dans beaucoup de frameworks et de bibliothèques de développement.

_**Solution :**_ Utiliser les modèles et bonnes pratiques appropriés

- Les ressources de passerelle NAT ont un délai d’inactivité TCP par défaut de 4 minutes.  Si ce paramètre est défini sur une valeur plus élevée, la traduction d’adresses réseau (NAT) accapare des flux plus longs et peut entraîner une [pression inutile sur l’inventaire des ports SNAT](nat-gateway-resource.md#timers).
- Les demandes atomiques (une demande par connexion) ne sont pas un bon choix de conception. Ce type d’anti-modèle limite la mise à l’échelle, réduit les performances et diminue la fiabilité. À la place, réutilisez les connexions HTTP/S pour réduire le nombre de connexions et les ports SNAT associés. L’échelle de l’application et les performances augmentent grâce à la réduction des connexions (handshakes), de la surcharge et du coût des opérations de chiffrement pendant l’utilisation de TLS.
- Le système DNS peut introduire un grand nombre de flux individuels quand le client ne met pas en cache le résultat des programmes de résolution DNS. Utilisez la mise en cache.
- Les flux UDP (par exemple, les recherches DNS) allouent des ports SNAT pour la durée du délai d’inactivité. Plus le délai d’inactivité est long, plus la sollicitation des ports SNAT est forte. Utilisez un délai d’inactivité court (par exemple, 4 minutes).
- Utilisez des pools de connexions pour déterminer votre volume de connexions.
- N’abandonnez jamais un flux TCP en mode silencieux et ne vous fiez pas aux minuteries TCP pour nettoyer un flux. Si vous ne laissez pas le protocole TCP fermer explicitement la connexion, l’état reste alloué aux systèmes intermédiaires et aux points de terminaison, et les ports SNAT ne sont alors pas disponibles pour les autres connexions. Ce modèle peut déclencher des échecs d’application et l’épuisement des ports SNAT. 
- Ne changez pas les valeurs de minuteur lié à la fermeture TCP au niveau du système d’exploitation sans en connaître précisément l’impact. Même si la pile TCP est récupérée, les performances de votre application risquent d’être affectées si les points de terminaison d’une connexion ont des attentes incompatibles. Quand vous voulez changer les minuteurs, c’est généralement qu’il y a un problème de conception sous-jacent. Tenez compte des recommandations suivantes.

L’épuisement des ports SNAT peut aussi être amplifié par d’autres anti-modèles dans l’application sous-jacente. Passez en revue ces modèles et bonnes pratiques supplémentaires pour améliorer la mise à l’échelle et la fiabilité de votre service.

- Explorez l’impact de la réduction du [délai d’inactivité TCP](nat-gateway-resource.md#timers) à des valeurs inférieures, notamment le délai d’inactivité par défaut de 4 minutes, pour libérer plus tôt l’inventaire des ports SNAT.
- Envisagez d’utiliser les [modèles d’interrogation asynchrone](https://docs.microsoft.com/azure/architecture/patterns/async-request-reply) pour les opérations de longue durée afin de libérer des ressources de connexion pour d’autres opérations.
- Les flux de longue durée (par exemple, les connexions TCP réutilisées) doivent utiliser des conservations de connexion TCP active ou des conservations de connexion active de la couche Application pour éviter l’expiration des systèmes intermédiaires. L’augmentation du délai d’inactivité est un dernier recours qui peut ne pas résoudre la cause racine. Un long délai d’expiration peut entraîner des échecs lié au bas débit quand le délai d’attente expire, ainsi qu’un retard et des échecs inutiles.
- Les [modèles de nouvelle tentative](https://docs.microsoft.com/azure/architecture/patterns/retry) sans perte de données doivent être utilisés pour éviter les nouvelles tentatives agressives/rafales en cas de défaillance temporaire ou de reprise d’activité après défaillance.
La création d’une connexion TCP pour chaque opération HTTP (également appelée « connexions atomiques ») est un anti-modèle.  Les connexions atomiques empêchent votre application d’être correctement mise à l’échelle et gaspillent des ressources.  Canalisez toujours plusieurs opérations dans la même connexion.  Votre application tire parti de la vitesse des transactions et des coûts des ressources.  Quand votre application utilise le chiffrement de la couche de transport (par exemple, TLS), un coût élevé est associé au traitement des nouvelles connexions.  Passez en revue les [Modèles de conception Azure Cloud](https://docs.microsoft.com/azure/architecture/patterns/) pour obtenir des modèles de bonnes pratiques supplémentaires.

#### <a name="additional-possible-mitigations"></a>Atténuations supplémentaires possibles

_**Solution :**_ Mettez à l’échelle la connectivité sortante comme suit :

| Scénario | Preuve |Limitation des risques |
|---|---|---|
| Vous rencontrez des problèmes de contention pour les ports SNAT et l’épuisement des ports SNAT au cours des périodes d’utilisation intensive. | La catégorie « Échec » pour la [métrique](nat-metrics.md) Connexions SNAT dans Azure Monitor présente des défaillances temporaires ou persistantes dans le temps et un volume de connexions élevé.  | Déterminez si vous pouvez ajouter des ressources d’adresses IP publiques ou des ressources de préfixes d’adresses IP publiques supplémentaires. Cet ajout permet jusqu’à 16 adresses IP au total pour votre passerelle NAT. Cet ajout fournit davantage d’inventaire pour les ports SNAT disponibles (64 000 par adresse IP) et vous permet d’affiner la misse à l’échelle de votre scénario.|
| Vous avez déjà donné 16 adresses IP et êtes toujours confronté à un épuisement des ports SNAT. | Échec de la tentative d’ajout d’une adresse IP. Le nombre total d’adresses IP provenant de ressources d’adresses IP publiques ou de ressources de préfixes d’adresses IP publiques est supérieur à 16. | Distribuez votre environnement d’application sur plusieurs sous-réseaux et fournissez une ressource de passerelle NAT pour chaque sous-réseau.  Réévaluez le ou les modèles de conception à optimiser en vous basant sur les [instructions](#design-patterns) précédentes. |

>[!NOTE]
>Il est important de comprendre la raison de l’épuisement des ressources SNAT. Veillez à utiliser les modèles appropriés pour des scénarios évolutifs et fiables.  L’ajout de ports SNAT supplémentaires à un scénario sans comprendre la cause de la demande doit être envisagé en dernier recours. Si vous ne comprenez pas pourquoi votre scénario applique une charge sur l’inventaire des ports SNAT, le fait d’ajouter des ports SNAT supplémentaires à l’inventaire en ajoutant plus d’adresses IP ne fera que retarder le même échec dû à l’épuisement quand votre application est mise à l’échelle.  Vous masquez peut-être d’autres manques d’efficacité et anti-modèles.

### <a name="icmp-ping-is-failing"></a>Échec de la commande ping ICMP

Le service [NAT du réseau virtuel](nat-overview.md) prend en charge les protocoles TCP et UDP IPv4. ICMP n’est pas pris en charge et est censé échouer.  

_**Solution :**_ Utilisez plutôt des tests de connexion TCP (par exemple « ping TCP ») et des tests de la couche Application spécifiques à UDP pour valider la connectivité de bout en bout.

Le tableau suivant peut être utilisé comme point de départ pour savoir quels outils utiliser pour démarrer des tests.

| Système d’exploitation | Test de connexion TCP générique | Test de la couche Application TCP | UDP |
|---|---|---|---|
| Linux | nc (test de connexion générique) | curl (test de la couche Application TCP) | spécifique à l’application |
| Windows | [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping) | [Invoke-WebRequest](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-webrequest) de PowerShell | spécifique à l’application |

### <a name="connectivity-failures"></a>Échecs de connectivité

Les échecs de connectivité avec le service [NAT de réseau virtuel](nat-overview.md) peuvent être dus à différents problèmes :

* Défaillances permanentes dues à des erreurs de configuration
* [Épuisement des ressources SNAT](#snat-exhaustion) de la passerelle NAT (temporaire ou persistant)
* Défaillances temporaires au niveau de l’infrastructure Azure 
* Défaillances temporaires au niveau du chemin entre Azure et la destination Internet publique 
* Défaillances temporaires ou persistantes au niveau de la destination Internet publique

Utilisez des outils tels que ceux mentionnés ci-après pour la validation de la connectivité. [Le ping ICMP n’est pas pris en charge](#icmp-ping-is-failing).

| Système d’exploitation | Test de connexion TCP générique | Test de la couche Application TCP | UDP |
|---|---|---|---|
| Linux | nc (test de connexion générique) | curl (test de la couche Application TCP) | spécifique à l’application |
| Windows | [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping) | [Invoke-WebRequest](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-webrequest) de PowerShell | spécifique à l’application |

#### <a name="configuration"></a>Configuration

Vérifiez votre configuration :
1. La ressource de passerelle NAT a-t-elle au moins une ressource d’adresse IP publique ou une ressource de préfixe d’adresse IP publique ? Au moins une adresse IP doit être associée à la passerelle NAT pour qu’elle puisse fournir une connectivité sortante.
2. Le sous-réseau du réseau virtuel est-il configuré pour utiliser la passerelle NAT ?
3. Utilisez-vous des routes définies par l’utilisateur (UDR) et remplacez-vous la destination ?  Les ressources de passerelle NAT deviennent la route par défaut (0/0) sur les sous-réseaux configurés.

#### <a name="snat-exhaustion"></a>Épuisement des ressources SNAT

Consultez la section de cet article relative à l’[épuisement des ressources SNAT](#snat-exhaustion).

#### <a name="azure-infrastructure"></a>Infrastructure Azure

Azure supervise et exécute son infrastructure avec beaucoup de soin. Des échecs temporaires peuvent se produire, il n’y aucune garantie de non-perte des transmissions.  Utilisez des modèles de conception qui autorisent les retransmissions SYN pour les applications TCP. Utilisez des délais de connexion suffisamment longs pour que la retransmission SYN TCP puisse réduire les impacts temporaires causés par un paquet SYN perdu.

_**Solution :**_

* Vérifiez l’[épuisement des ressources SNAT](#snat-exhaustion).
* Le paramètre de configuration d’une pile TCP qui contrôle le comportement de la retransmission SYN est le [délai d’attente de retransmission](https://tools.ietf.org/html/rfc793) (RTO, Retransmission Time-Out). La valeur RTO peut être ajustée. En général, elle est d’au moins 1 seconde par défaut avec une temporisation exponentielle.  Si le délai de connexion de votre application est trop court (par exemple, 1 seconde), vous pouvez constater des expirations de connexion sporadiques.  Augmentez le délai de connexion de l’application.
* Si vous observez des délais plus longs ou inattendus avec les comportements de l’application par défaut, ouvrez un cas de support pour obtenir une assistance supplémentaire en vue de la résolution du problème.

Nous vous déconseillons de réduire le délai d’expiration de connexion TCP de façon artificielle ou d’ajuster le paramètre RTO.

#### <a name="public-internet-transit"></a>Transit Internet public

Plus le chemin jusqu’à la destination est long et plus il existe de systèmes intermédiaires, plus les chances de défaillances temporaires sont élevées. Nous savons que la fréquence des défaillances temporaires sur l’[infrastructure Azure](#azure-infrastructure) peut augmenter. 

Suivez les instructions fournies dans la section précédente [Infrastructure Azure](#azure-infrastructure).

#### <a name="internet-endpoint"></a>Point de terminaison Internet

Les sections précédentes s’appliquent, ainsi que le point de terminaison Internet avec lequel la communication est établie. D’autres facteurs peuvent impacter la réussite de la connectivité :

* Gestion du trafic côté destination, y compris
- Limitation du taux d’API imposée côté destination
- Atténuations DDoS volumétriques ou régulation de flux de la couche de transport
* Pare-feu ou autres composants côté destination 

En général, des captures de paquets au niveau de la source et de la destination (le cas échéant) sont nécessaires pour déterminer ce qui se produit.

_**Solution :**_

* Vérifiez l’[épuisement des ressources SNAT](#snat-exhaustion). 
* Vérifiez la connectivité à un point de terminaison dans la même région et dans une région différente à des fins de comparaison.  
* Si vous effectuez des tests de taux de transactions ou de volume élevé, vérifiez si la réduction du taux réduit l’occurrence des défaillances.
* Si la modification du taux a un impact sur le taux de défaillances, vérifiez si les limites de taux d’API ont été atteintes et vérifiez l’impact éventuel d’autres contraintes côté destination.
* Si votre recherche n’aboutit pas, ouvrez un cas de support pour obtenir une assistance supplémentaire en vue de la résolution du problème.

#### <a name="tcp-resets-received"></a>Réinitialisations TCP reçues

La passerelle NAT génère des réinitialisations TCP sur la machine virtuelle source pour le trafic qui n’est pas reconnu comme étant en cours.

Ceci peut être dû, par exemple, à l’expiration du délai d’inactivité de la connexion TCP.  Vous pouvez définir le délai d’inactivité entre 4 minutes et 120 minutes.

Les réinitialisations TCP ne sont pas générées du côté public des ressources de passerelle NAT. Les réinitialisations TCP côté destination sont générées par la machine virtuelle source et non par la ressource de passerelle NAT.

_**Solution :**_

* Passez en revue les recommandations sur les [modèles de conception](#design-patterns).  
* Si nécessaire, ouvrez un cas de support pour obtenir une assistance supplémentaire en vue de la résolution du problème.

### <a name="ipv6-coexistence"></a>Coexistence IPv6

Le service [NAT de réseau virtuel](nat-overview.md) prend en charge les protocoles TCP et UDP IPv4. Le déploiement sur un [sous-réseau avec préfixe IPv6 n’est pas pris en charge](nat-overview.md#limitations).

_**Solution :**_ Déployer une passerelle NAT sur un sous-réseau sans préfixe IPv6.

Si vous êtes intéressé par d’autres fonctionnalités, vous pouvez l’indiquer sur le site [UserVoice](https://aka.ms/natuservoice) (service NAT de réseau virtuel).

### <a name="connection-doesnt-originate-from-nat-gateway-ips"></a>La connexion ne provient pas de la ou des adresses IP de passerelle NAT

Vous configurez la passerelle NAT, la ou les adresses IP à utiliser et le sous-réseau qui doit utiliser une ressource de passerelle NAT. Toutefois, les connexions provenant d’instances de machines virtuelles qui existaient avant le déploiement de la passerelle NAT n’utilisent pas la ou les adresses IP.  Elles semblent recourir à des adresses IP qui ne sont pas utilisées avec la ressource de passerelle NAT.

_**Solution :**_

Le [NAT de réseau virtuel](nat-overview.md) remplace la connectivité sortante pour le sous-réseau sur lequel il est configuré. Lors de la transition du port SNAT par défaut ou du port SNAT sortant de l’équilibreur de charge vers des passerelles NAT, les nouvelles connexions commencent immédiatement à utiliser la ou les adresses IP associées à la ressource de passerelle NAT.  Toutefois, si une machine virtuelle dispose toujours d’une connexion pendant le basculement vers la ressource de passerelle NAT, cette connexion continue d’utiliser l’ancienne adresse IP SNAT qui a été affectée lors de l’établissement de la connexion.  Établissez une nouvelle connexion au lieu de réutiliser celle qui existait déjà parce que le système d’exploitation ou le navigateur mettait en cache les connexions dans un pool de connexions.  Par exemple, lorsque vous utilisez _curl_ dans PowerShell, veillez à spécifier le paramètre _-DisableKeepalive_ pour forcer une nouvelle connexion.  Si vous utilisez un navigateur, les connexions peuvent également être mises en pool.

Il n’est pas nécessaire de redémarrer une machine virtuelle en configurant un sous-réseau pour une ressource de passerelle NAT.  Cependant, si la machine est redémarrée, l’état de la connexion est vidé.  De ce fait, toutes les connexions se mettent à utiliser la ou les adresses IP de la ressource de passerelle NAT.  Il s’agit toutefois d’un effet secondaire du redémarrage de la machine virtuelle et non d’un signe indiquant qu’un redémarrage est nécessaire.

Si vous rencontrez toujours des problèmes, ouvrez un dossier de support pour les résoudre.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur le service [Nat de Réseau virtuel](nat-overview.md).
* Découvrir la [ressource de passerelle NAT](nat-gateway-resource.md)
* Découvrez les [métriques et les alertes pour les ressources de passerelle NAT](nat-metrics.md).
* [Utilisez UserVoice pour nous faire part des prochains développements dont vous aimeriez bénéficier concernant le service NAT de réseau virtuel](https://aka.ms/natuservoice).

