---
title: Serveurs frontaux multiples – Azure Load Balancer
description: Dans ce parcours d’apprentissage, familiarisez-vous avec une vue d’ensemble des serveurs frontaux multiples sur Azure Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.custom: seodec18
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/07/2019
ms.author: allensu
ms.openlocfilehash: 5c2072d13cab9839a276c0437747d7075918e78a
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696878"
---
# <a name="multiple-frontends-for-azure-load-balancer"></a>Serveurs frontaux multiples pour Azure Load Balancer

Azure Load Balancer vous permet d’équilibrer la charge des services sur plusieurs ports, plusieurs adresses IP ou les deux. Vous pouvez utiliser les définitions d’équilibrage de charge public et interne pour charger les flux équilibrés sur un ensemble de machines virtuelles.

Cet article décrit les principes de base de cette capacité, les concepts importants et les contraintes. Si vous souhaitez uniquement exposer des services sur une adresse IP, des instructions simplifiées sont disponibles pour les configurations d’équilibreur de charge [publiques](./quickstart-load-balancer-standard-public-portal.md) ou [internes](./quickstart-load-balancer-standard-internal-portal.md). Les serveurs frontaux multiples s’ajoutent à une configuration de serveur frontal unique. À l’aide des concepts présentés dans cet article, vous pouvez étendre une configuration simplifiée à tout moment.

Lorsque vous définissez Azure Load Balancer, un serveur frontal et une configuration de pool principal sont connectés à des règles. La sonde d’intégrité référencée par la règle est utilisée pour déterminer comment les nouveaux flux sont envoyés à un nœud du pool principal. Le serveur frontal (adresse IP virtuelle aka) est défini par un élément à 3 tuples comprenant une adresse IP (publique ou interne), un protocole de transport (TCP ou UDP) et un numéro de port issu de la règle de l’équilibrage de charge. Le pool principal est une collection de configurations IP de machines virtuelles (partie de la ressource de la carte réseau) qui font référence au pool principal Load Balancer.

Le tableau suivant contient quelques exemples de configurations frontales :

| Serveur frontal | Adresse IP | protocol | port |
| --- | --- | --- | --- |
| 1 |65.52.0.1 |TCP |80 |
| 2 |65.52.0.1 |TCP |*8080* |
| 3 |65.52.0.1 |*UDP* |80 |
| 4 |*65.52.0.2* |TCP |80 |

Le tableau montre quatre serveurs frontaux différents. Les serveurs frontaux nº 1, 2 et 3 présentent un serveur frontal unique avec des règles multiples. La même adresse IP est utilisée, mais le port ou le protocole est différent pour chaque serveur frontal. Les serveurs frontaux nº 1 et 4 sont des exemples de serveurs frontaux multiples, avec la réutilisation d’un protocole et d’un port frontaux identiques sur plusieurs serveurs frontaux.

Azure Load Balancer offre une certaine flexibilité pour définir les règles d’équilibrage de charge. Une règle indique de quelle manière une adresse et un port du serveur frontal sont mappés à l’adresse et au port de destination du serveur principal. La réutilisation ou non des ports principaux pour différentes règles dépend du type de la règle. Chaque type de règle présente des exigences spécifiques qui peuvent affecter la configuration de l’hôte et la conception de la sonde. Il existe deux types de règle :

1. La règle par défaut, sans réutilisation des ports principaux
2. La règle d’adresse IP flottante, où les ports principaux sont réutilisés

Azure Load Balancer vous permet d’associer les deux types de règle sur la même configuration d’équilibrage de charge. L’équilibrage de charge peut les utiliser simultanément pour une machine virtuelle donnée, ou toute combinaison, tant que vous respectez les contraintes de la règle. Le choix du type de règle dépend des exigences de votre application et de la complexité de la prise en charge de cette configuration. Vous devez évaluer quels types de règle conviennent le mieux à votre scénario.

Nous examinons ces scénarios plus en détail en commençant par le comportement par défaut.

## <a name="rule-type-1-no-backend-port-reuse"></a>Type de règle nº 1 : pas de réutilisation des ports principaux

![Illustration de serveurs frontaux multiples avec serveur frontal vert et violet](./media/load-balancer-multivip-overview/load-balancer-multivip.png)

Dans ce scénario, les serveurs frontaux sont configurés comme suit :

| Serveur frontal | Adresse IP | protocol | port |
| --- | --- | --- | --- |
| ![serveur frontal vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |
| ![serveur frontal violet](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |*65.52.0.2* |TCP |80 |

L’adresse IP dédiée est la destination du flux entrant. Dans le pool principal, chaque machine virtuelle expose le service souhaité sur un port unique sur une adresse IP dédiée. Ce service est associé avec le serveur frontal via une définition de règles.

Nous définissons deux règles :

| Règle | Mapper le serveur frontal | Au pool principal |
| --- | --- | --- |
| 1 |![serveur frontal vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) Frontend1:80 |![backend vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) DIP1:80, ![backend vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) DIP2:80 |
| 2 |![Adresse IP virtuelle](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) Frontend2:80 |![backend violet](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) DIP1:81, ![backend violet](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) DIP2:81 |

Le mappage complet dans Azure Load Balancer se présente désormais comme suit :

| Règle | Adresse IP du serveur frontal | protocol | port | Destination | port |
| --- | --- | --- | --- | --- | --- |
| ![règle verte](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |Adresse IP dédiée |80 |
| ![règle violette](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |65.52.0.2 |TCP |80 |Adresse IP dédiée |81 |

Chaque règle doit produire un flux avec une combinaison unique d’adresse IP de destination et de port de destination. En modifiant le port de destination du flux, il est possible de fournir des flux à la même adresse IP dédiée sur différents ports avec plusieurs règles.

Les sondes d’intégrité sont toujours dirigées vers l’adresse IP dédiée d’une machine virtuelle. Vous devez vous assurer que votre sonde reflète l’intégrité de la machine virtuelle.

## <a name="rule-type-2-backend-port-reuse-by-using-floating-ip"></a>Type de règle nº 2 : réutilisation des ports principaux à l’aide d’une adresse IP flottante

Azure Load Balancer offre la possibilité de réutiliser le port frontal sur plusieurs serveurs frontaux, quel que soit le type de règle utilisé. En outre, dans certains scénarios d’application, il est préférable ou nécessaire que plusieurs instances d’application utilisent le même port sur une machine virtuelle du pool principal. Des exemples courants de réutilisation de port incluent le clustering pour la haute disponibilité, les appliances réseau virtuelles, ainsi que l’exposition de plusieurs points de terminaison TLS sans nouveau chiffrement.

Si vous souhaitez réutiliser le port principal pour plusieurs règles, vous devez activer la fonctionnalité d’adresse IP flottante dans la définition des règles.

L’adresse IP flottante est le terme Azure désignant une partie de ce que l’on appelle le « retour direct du serveur » (DSR). Le DSR se compose de deux éléments : une topologie de flux et un schéma de mappage d’adresses IP. Du point de vue de la plate-forme, Azure Load Balancer fonctionne toujours dans une topologie de flux DSR, que l’adresse IP flottante soit activée ou non. Cela signifie que la partie sortante d’un flux est toujours correctement réécrite pour retourner directement à l’origine.

Avec le type de règle par défaut, Azure expose un schéma classique de mappage d’adresses IP à équilibrage de charge pour simplifier l’utilisation. L’activation de l’adresse IP flottante modifie le schéma de mappage d’adresses IP pour offrir davantage de flexibilité, comme expliqué ci-dessous.

Le schéma suivant illustre cette configuration :

![Illustration de serveurs frontaux multiples avec serveur frontal vert et violet, avec DSR](./media/load-balancer-multivip-overview/load-balancer-multivip-dsr.png)

Pour ce scénario, chaque machine virtuelle du pool principal a trois interfaces réseau :

* DIP : carte réseau virtuelle associée à la machine virtuelle (configuration IP de ressource de carte réseau d’Azure)
* Frontend1 : interface de bouclage du SE invité qui est configurée avec l’adresse IP de Frontend1
* Frontend2 : interface de bouclage du SE invité qui est configurée avec l’adresse IP de Frontend2

Pour chaque machine virtuelle du pool de back-ends, exécutez les commandes suivantes à une invite de commandes Windows.

Pour obtenir la liste des noms d’interfaces que vous avez sur votre machine virtuelle, tapez la commande suivante :

```console
netsh interface show interface 
```

Pour la carte réseau de machine virtuelle (managée par Azure), tapez la commande suivante :

```console
netsh interface ipv4 set interface “interfacename” weakhostreceive=enabled
```

(remplacez interfacename par le nom de cette interface)

Pour chaque interface de bouclage que vous avez ajoutée, répétez les commandes suivantes :

```console
netsh interface ipv4 set interface “interfacename” weakhostreceive=enabled 
```

(remplacez interfacename par le nom de cette interface de bouclage)

```console
netsh interface ipv4 set interface “interfacename” weakhostsend=enabled 
```

(remplacez interfacename par le nom de cette interface de bouclage)

> [!IMPORTANT]
> La configuration des interfaces de bouclage est effectuée dans le SE invité. Cette configuration n’est pas exécutée ou gérée par Azure. Sans cette configuration, les règles ne fonctionneront pas. Les définitions de sonde d’intégrité utilisent l’adresse IP dédiée de la machine virtuelle, plutôt que l’interface de bouclage représentant le serveur frontal DSR. Par conséquent, votre service doit fournir les réponses de sonde sur un port d’adresse IP dédiée qui reflète l’état du service offert sur l’interface de bouclage représentant le serveur frontal DSR.


Prenons pour exemple la même configuration frontale que dans le scénario précédent :

| Serveur frontal | Adresse IP | protocol | port |
| --- | --- | --- | --- |
| ![serveur frontal vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |
| ![serveur frontal violet](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |*65.52.0.2* |TCP |80 |

Nous définissons deux règles :

| Règle | Serveur frontal | Mapping au pool principal |
| --- | --- | --- |
| 1 |![règle verte](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) Frontend1:80 |![backend vert](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) Frontend1:80 (dans VM1 et VM2) |
| 2 |![règle violette](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) Frontend2:80 |![backend violet](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) Frontend2:80 (dans VM1 et VM2) |

Le tableau suivant présente le mappage complet de l’équilibrage de charge :

| Règle | Adresse IP du serveur frontal | protocol | port | Destination | port |
| --- | --- | --- | --- | --- | --- |
| ![règle verte](./media/load-balancer-multivip-overview/load-balancer-rule-green.png) 1 |65.52.0.1 |TCP |80 |identique au serveur frontal (65.52.0.1) |identique au serveur frontal (80) |
| ![règle violette](./media/load-balancer-multivip-overview/load-balancer-rule-purple.png) 2 |65.52.0.2 |TCP |80 |identique au serveur frontal (65.52.0.2) |identique au serveur frontal (80) |

La destination du flux entrant est l’adresse IP de serveur frontal sur l’interface de bouclage de la machine virtuelle. Chaque règle doit produire un flux avec une combinaison unique d’adresse IP de destination et de port de destination. En modifiant l’adresse IP de destination du flux, la réutilisation de port est possible sur la même machine virtuelle. Votre service est exposé à l’équilibrage de charge en le liant à l’adresse IP du serveur frontal et au port de l’interface de bouclage associée.

Notez que cet exemple ne modifie pas le port de destination. Bien qu’il s’agisse d’un scénario d’adresse IP flottante, l’équilibrage de charge Azure prend également en charge la définition d’une règle pour réécrire le port de destination principal afin qu’il soit différent du port de destination frontal.

Le type de règle faisant appel à l’adresse IP flottante constitue la base de plusieurs modèles de configuration d’équilibrage de charge. Un exemple disponible actuellement est la configuration [SQL AlwaysOn avec plusieurs écouteurs](../azure-sql/virtual-machines/windows/availability-group-listener-powershell-configure.md) . Au fil du temps, nous documenterons un plus grand nombre de ces scénarios.

## <a name="limitations"></a>Limites

* Les configurations de serveurs frontaux multiples sont uniquement prises en charge avec les machines virtuelles IaaS.
* Avec la règle IP flottante, votre application doit utiliser la configuration IP principale pour les flux SNAT sortants. Si votre application se lie à l’adresse IP du serveur frontal configurée sur l’interface de bouclage du SE invité, la SNAT sortant d’Azure n’est pas disponible pour réécrire le flux sortant et le flux échoue.  Passez en revue les [scénarios de sortie](load-balancer-outbound-connections.md).
* Une adresse IP flottante n’est actuellement pas prise en charge sur des configurations IP secondaires pour des scénarios d’équilibrage de charge interne.
* Les adresses IP publiques ont une incidence sur la facturation. Pour plus d’informations, voir la page [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses/)
* Des limites d’abonnement s’appliquent. Pour plus d’informations, voir les [limites de service](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) .

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Connexions sortantes](load-balancer-outbound-connections.md) pour comprendre l’impact de plusieurs serveurs frontaux sur le comportement de la connexion sortante.