---
title: Paramètres du portail Azure Load Balancer
description: Commencez à vous renseigner sur les paramètres du portail Azure Load Balancer
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/8/2020
ms.author: allensu
ms.openlocfilehash: e1080aea12e70f4312fbee07b063d5a5cfbd1201
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89596208"
---
# <a name="azure-load-balancer-portal-settings"></a>Paramètres du portail Azure Load Balancer

À mesure que vous créez une instance Azure Load Balancer, les informations contenues dans cet article vous aideront à en savoir plus sur les paramètres individuels et sur la configuration qui vous convient le mieux.

## <a name="create-load-balancer"></a>Créer un équilibreur de charge

Azure Load Balancer est un équilibreur de charge réseau qui répartit le trafic entre les instances de machine virtuelle dans le pool principal.

### <a name="basics"></a>Concepts de base

Dans l’onglet **Fonctions de base** de la page Créer un portail d’équilibrage de charge, entrez les informations suivantes :



| Paramètre |  Détails |
| ---------- | ---------- |
| Abonnement  | Sélectionnez votre abonnement. Cette sélection correspond à l’abonnement dans lequel vous souhaitez déployer votre équilibreur de charge. |
| Resource group | Sélectionnez **Créer nouveau** et saisissez le nom de votre groupe de ressources dans la zone de texte. Si vous avez déjà un groupe de ressources, sélectionnez-le. |
| Nom | Ce paramètre est le nom de votre instance Azure Load Balancer. |
| Région | Sélectionnez la région Azure dans laquelle vous souhaitez déployer votre équilibreur de charge. |
| Type | L’équilibreur de charge a deux types : </br> **Interne (privé)** </br> **Public (externe)**</br> Un équilibreur de charge interne (ILB) achemine le trafic vers les membres du pool principal via une adresse IP privée.</br> Un équilibreur de charge public dirige les requêtes des clients via Internet vers le pool principal.</br> En savoir plus sur les [types d’équilibreur de charge](components.md#frontend-ip-configuration-).|
| SKU  | Sélectionnez **Standard**. </br> L’équilibreur de charge a deux niveaux tarifaires : **De base** et **Standard**. </br> Le niveau De base offre des fonctionnalités limitées. </br> Le niveau **Standard** est recommandé pour les charges de travail de production. </br> En savoir plus sur les [niveaux tarifaires](skus.md). |

Si vous sélectionnez le type **Public**, les informations suivantes s’affichent :

| Paramètre |  Détails |
| ---------- | ---------- |
| Adresse IP publique | Sélectionnez **Créer nouveau** pour créer une IP publique pour votre équilibreur de charge public. </br> Si vous disposez d’une IP publique, sélectionnez **Utiliser l’IP existante**.  |
| Nom de l’adresse IP publique | Nom de l’IP publique.|
| Référence SKU d’adresse IP publique | Les IP publiques ont deux niveaux tarifaires : **De base** et **Standard**. </br> Le niveau De base ne prend pas en charge la résilience des zones ni les attributs de zone. </br> Le niveau **Standard** est recommandé pour les charges de travail de production. </br> Les niveaux tarifaires de l’équilibreur de charge et de l’IP publique **doivent correspondre**. |
| Affectation | **Statique** est sélectionné automatiquement pour le niveau Standard. </br> Les IP publiques du niveau De base ont deux types : **dynamique** et **statique**. </br> Les IP publiques dynamiques ne sont pas attribuées avant la création. </br> Les adresses IP peuvent être perdues si la ressource est supprimée. </br> L’utilisation d’adresses IP statiques est recommandée. |
| Zone de disponibilité | Sélectionnez **Redondant interzone** pour créer un équilibreur de charge résilient. </br> Pour créer un équilibreur de charge zonal, sélectionnez une zone spécifique parmi **1**, **2** ou **3**. </br> L’équilibreur de charge standard et les IP publiques prennent en charge les zones. </br> En savoir plus sur les [équilibreurs de charge et les zones de disponibilité](load-balancer-standard-availability-zones.md). </br> Aucune sélection de zone n’est disponible pour le niveau De base. L’équilibreur de charge de base ne prend pas en charge les zones. |
| Ajouter une adresse IPv6 publique | L’équilibreur de charge prend en charge les adresses **IPv6** pour votre serveur frontal. </br> En savoir plus sur les [équilibreurs de charge et IPv6](load-balancer-ipv6-overview.md).
| Préférence de routage | Sélectionnez **Réseau Microsoft**. </br> Réseau Microsoft signifie que le trafic est acheminé via le réseau mondial de Microsoft. </br> Internet signifie que le trafic est acheminé via le réseau du fournisseur de services Internet. </br> En savoir plus sur les [préférences de routage](../virtual-network/routing-preference-overview.md).|

:::image type="content" source="./media/manage/create-public-load-balancer-basics.png" alt-text="Créer un équilibreur de charge public." border="true":::

Si vous sélectionnez le type **Interne**, les informations suivantes s’affichent :

| Paramètre |  Détails |
| ---------- | ---------- |
| Réseau virtuel | Réseau virtuel dont vous souhaitez que votre équilibreur de charge interne fasse partie. </br> L’adresse IP privée du serveur frontal que vous sélectionnez pour votre équilibreur de charge interne sera celle de ce réseau virtuel. |
| Affectation d’adresses IP | Les options sont **Statique** ou **Dynamique**. </br> Statique garantit que l’adresse IP ne change pas. Une adresse IP dynamique peut changer. |
| Zone de disponibilité | Les options disponibles sont : </br> **Redondant dans une zone** </br> **Zone 1** </br> **Zone 2** </br> **Zone 3** </br> Pour créer un équilibreur de charge hautement disponible et résilient aux défaillances de zones de disponibilité, sélectionnez une IP **redondante interzone**. |

:::image type="content" source="./media/manage/create-internal-load-balancer-basics.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="frontend-ip-configuration"></a>Configuration d’adresses IP frontales

Adresse IP de l’équilibreur de charge Azure Load Balancer. Il s’agit du point de contact pour les clients. 

Vous pouvez avoir une ou plusieurs configurations d’adresses IP frontales. Si vous avez parcouru la section Concepts de base ci-dessus, vous devriez avoir déjà créé un serveur frontal pour votre équilibreur de charge. 

Si vous souhaitez ajouter une configuration d’adresse IP frontale à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Configuration d’adresse IP frontale**, puis sélectionnez **+Ajouter**.

| Paramètre |  Détails |
| ---------- | ---------- |
| Nom | Nom de votre configuration d’adresse IP frontale. |
| Version de l’adresse IP | Version de l’adresse IP que vous souhaitez avoir pour votre serveur frontal. </br> L’équilibreur de charge prend en charge les configurations d’adresses IP frontales IPv4 et IPv6. |
| Type IP | Le type d’adresse IP détermine si une adresse IP unique est associée à votre serveur frontal ou s’il s’agit d’une plage d’adresses IP utilisant un préfixe IP. </br> Un [préfixe d’IP publique](../virtual-network/public-ip-address-prefix.md) vous aide quand vous devez vous connecter au même point de terminaison à plusieurs reprises. Le préfixe garantit qu’un nombre suffisant de ports sont fournis pour faciliter la résolution des problèmes de port SNAT. |
| IP publique (ou Préfixe si vous avez sélectionné Préfixe ci-dessus) | Sélectionnez ou créez une nouvelle IP publique (ou un nouveau préfixe) pour le serveur frontal de votre équilibreur de charge. |

:::image type="content" source="./media/manage/frontend.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="backend-pools"></a>Pools de back-ends

Le pool d’adresses principal contient les adresses IP des interfaces réseau virtuelles dans le pool principal. 

Si vous souhaitez ajouter un pool principal à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Pools principaux**, puis sélectionnez **+Ajouter**.

| Paramètre | Détails |
| ---------- |  ---------- |
| Nom | Nom de votre pool principal. |
| Réseau virtuel | Réseau virtuel de vos instances de serveur principal. |
| Version de l’adresse IP | Les options sont **IPv4** ou **IPv6**. |

Vous pouvez ajouter des machines virtuelles ou des groupes de machines virtuelles identiques dans le pool principal de votre instance Azure Load Balancer. Créez d’abord les machines virtuelles ou les groupes de machines virtuelles identiques. Ensuite, ajoutez-les à l’équilibreur de charge dans le portail.

:::image type="content" source="./media/manage/backend.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="health-probes"></a>Sondes d’intégrité

Une sonde d’intégrité est utilisée pour surveiller l’état de vos instances ou machines virtuelles principales. L’état de la sonde d’intégrité détermine à quel moment les nouvelles connexions sont envoyées à une instance en fonction des contrôles d’intégrité. 

Si vous souhaitez ajouter une sonde d’intégrité à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Sondes d’intégrité**, puis sélectionnez **+Ajouter**.

| Paramètre | Détails |
| ---------- | ---------- |
| Nom | Nom de votre sonde d’intégrité. |
| Protocol | Le protocole que vous sélectionnez définit le type de vérification utilisé pour déterminer si la ou les instances principales sont saines. </br> Les options disponibles sont : </br> **TCP** </br> **HTTPS** </br> **HTTP** </br> Vérifiez que vous utilisez le protocole approprié. Cette sélection dépend de la nature de votre application. </br> La configuration de la sonde d’intégrité et les réponses de la sonde déterminent quelles instances de pool principal recevront de nouveaux flux. </br> Vous pouvez utiliser des sondes d’intégrité pour détecter la défaillance d’une application sur un point de terminaison back-end. </br> En savoir plus sur les [sondes d’intégrité](load-balancer-custom-probe-overview.md). |
| Port | Port de destination de la sonde d’intégrité. </br> Ce paramètre désigne le port sur l’instance principale que la sonde d’intégrité utilisera pour déterminer l’intégrité de l’instance. |
| Intervalle | Nombre de secondes entre les tentatives de la sonde. </br> L’intervalle détermine la fréquence à laquelle la sonde d’intégrité tentera d’atteindre l’instance principale. </br> Si vous sélectionnez 5, la sonde effectuera une deuxième tentative après 5 secondes, et ainsi de suite. |
| Seuil de défaillance sur le plan de l’intégrité | Nombre d’échecs de sonde consécutifs qui se produisent avant qu’une machine virtuelle soit considérée comme non saine.</br> Si vous sélectionnez 2, aucun flux n’est défini sur cette instance principale après deux échecs consécutifs. |

:::image type="content" source="./media/manage/health-probe.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="load-balancing-rules"></a>Règles d’équilibrage de charge

Définit la manière dont le trafic entrant est distribué à toutes les instances du pool principal. Une règle d’équilibrage de charge fait correspondre une configuration d’adresse IP frontale et un port donnés à plusieurs adresses IP frontales et ports.

Si vous souhaitez ajouter une règle d’équilibrage de charge à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Règles d’équilibrage de charge**, puis sélectionnez **+Ajouter**.
    
| Paramètre | Détails |
| ---------- | ---------- |
| Nom | Nom de la règle d’équilibrage de charge. |
| Version de l’adresse IP | Les options sont **IPv4** ou **IPv6**.  |
| Adresse IP du serveur frontal | Sélectionnez l’adresse IP frontale. </br> Adresse IP frontale de votre équilibreur de charge auquel vous souhaitez associer la règle d’équilibrage de charge.|
| Protocol | Azure Load Balancer est un équilibreur de charge réseau de couche 4. </br> Vos options sont : **TCP** ou **UDP**. |
| Port | Ce paramètre est le port associé à l’adresse IP frontale à laquelle vous souhaitez que le trafic soit distribué en fonction de cette règle d’équilibrage de charge. |
| Port principal | Ce paramètre est le port sur les instances du pool principal vers lequel vous souhaitez que l’équilibreur de charge envoie le trafic. Ce paramètre peut être le même que le port frontal ou différent si vous avez besoin de flexibilité pour votre application. |
| Pool principal | Pool principal sur lequel vous souhaitez appliquer cette règle d’équilibrage de charge. |
| Sonde d’intégrité | Sonde d’intégrité que vous avez créée pour vérifier l’état des instances dans le pool principal. </br> Seules les instances saines recevront un nouveau trafic. |
| Persistance de session |  Les options disponibles sont : </br> **Aucun** </br> **Adresse IP du client** </br> **Adresse IP du client et protocole**</br> </br> Maintenez le trafic d’un client vers la même machine virtuelle dans le pool principal. Ce trafic sera conservé pendant toute la durée de la session. </br> **Aucun** spécifie que les requêtes successives provenant du même client peuvent être gérées par n’importe quelle machine virtuelle. </br> **Adresse IP du client** spécifie que les requêtes successives provenant de la même adresse IP du client seront gérées par la même machine virtuelle. </br> **Adresse IP du client et protocole** spécifie que les requêtes successives provenant de la même adresse IP du client et via le même protocole seront gérées par la même machine virtuelle. </br> En savoir plus sur les [modes de distribution](load-balancer-distribution-mode.md). |
| Délai d’inactivité (minutes) | Maintenez une connexion **TCP** ou **HTTP** ouverte sans compter sur des clients pour l’envoi de messages de persistance de connexion. |  
| Réinitialisation du protocole TCP | L’équilibreur de charge peut envoyer des **réinitialisations du protocole TCP** pour aider à créer un comportement d’application plus prévisible lorsque la connexion est inactive. </br> En savoir plus sur la [réinitialisation du protocole TCP](load-balancer-tcp-reset.md).|
| IP flottante | « IP flottante » est le terme Azure désignant une partie de ce que l’on appelle le **retour direct du serveur (DSR)** . </br> La configuration DSR comprend deux parties : <br> 1. une topologie de flux ; </br> 2. Schéma de mappage d’adresses IP au niveau de la plateforme. </br></br> Azure Load Balancer fonctionne toujours dans une topologie de flux DSR, que l’IP flottante soit activée ou non. </br> Ce fonctionnement signifie que la partie sortante d’un flux est toujours correctement réécrite pour retourner directement à l’origine. </br> Sans adresse IP flottante, Azure expose un schéma traditionnel de mappage d’adresses IP d’équilibrage de charge : l’adresse IP des instances de machine virtuelle. </br> L’activation de l’IP flottante modifie le mappage des adresses IP à l’adresse IP frontale de l’équilibreur de charge afin de permettre davantage de flexibilité. </br> Pour plus d’informations, consultez [Serveurs frontaux multiples dans Azure Load Balancer](load-balancer-multivip-overview.md).|
| Créer des règles de trafic sortant implicites | Sélectionnez **Non**. </br> Valeur par défaut : **disableOutboundSnat = false**  </br> Dans ce cas, le trafic sortant se produit via la même adresse IP frontale. </br></br> **disableOutboundSnat = true** </br>Dans ce cas, des règles de trafic sortant sont nécessaires pour le trafic sortant. |

:::image type="content" source="./media/manage/load-balancing-rule.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="inbound-nat-rules"></a>Règles NAT entrantes

Une règle NAT de trafic entrant transfère le trafic entrant envoyé à la combinaison de l’adresse IP frontale et du port. 

Le trafic est envoyé à une machine virtuelle ou instance spécifique dans le pool back-end. Ce réacheminement de port est accompli à l’aide de la même distribution basée sur le hachage que l’équilibrage de charge.

Si votre scénario nécessite que des sessions RDP (Remote Desktop Protocol) ou SSH (Secure Shell) séparent des instances de machine virtuelle d’un pool principal, plusieurs points de terminaison internes peuvent être mappés à des ports sur la même adresse IP frontale. 

Les adresses IP frontales peuvent être utilisées pour administrer à distance vos machines virtuelles sans serveur de rebond supplémentaire.

Si vous souhaitez ajouter une règle NAT de trafic entrant à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Règles NAT de trafic entrant**, puis sélectionnez **+Ajouter**.

| Paramètre | Détails |
| ---------- | ---------- |
| Nom | Nom de votre règle NAT de trafic entrant. |
| Adresse IP du serveur frontal | Sélectionnez l’adresse IP frontale. </br> Adresse IP frontale de votre équilibreur de charge à laquelle vous souhaitez associer la règle NAT de trafic entrant. |
| Version de l’adresse IP | Les options sont IPv4 ou IPv6. |
| Service | Type de service que vous exécuterez sur Azure Load Balancer. </br> Une sélection permet de mettre à jour les informations de port de manière appropriée. |
| Protocol | Azure Load Balancer est un équilibreur de charge réseau de couche 4. </br> Vos options sont : TCP ou UDP. |
| Délai d’inactivité (minutes) | Maintenez une connexion TCP ou HTTP ouverte sans compter sur des clients pour l’envoi de messages de persistance de connexion. |
| Réinitialisation du protocole TCP | L’équilibreur de charge peut envoyer des réinitialisations du protocole TCP pour aider à créer un comportement d’application plus prévisible lorsque la connexion est inactive. </br> En savoir plus sur la [réinitialisation du protocole TCP](load-balancer-tcp-reset.md). |
| Port | Ce paramètre est le port associé à l’adresse IP frontale à laquelle vous souhaitez que le trafic soit distribué en fonction de cette règle NAT de trafic entrant. |
| Machine virtuelle cible | Partie de la machine virtuelle du pool principal à laquelle vous souhaitez associer cette règle. |
| Mappage de ports | Ce paramètre peut être défini par défaut ou personnalisé selon les préférences de votre application. |

:::image type="content" source="./media/manage/inbound-nat-rule.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="outbound-rules"></a>Règles de trafic sortant

Les règles de trafic sortant de l’équilibreur de charge configurent la SNAT sortante pour les machines virtuelles dans le pool principal.

Si vous souhaitez ajouter une règle de trafic sortant à votre équilibreur de charge, accédez à votre équilibreur de charge dans le portail Azure, sélectionnez **Règles de trafic sortant**, puis sélectionnez **+Ajouter**.

| Paramètre | Détails |
| ------- | ------ |
| Nom | Nom de votre règle de trafic sortant. |
| Adresse IP du serveur frontal | Sélectionnez l’adresse IP frontale. </br> Adresse IP frontale de votre équilibreur de charge à laquelle vous souhaitez associer la règle de trafic sortant. |
| Protocol | Azure Load Balancer est un équilibreur de charge réseau de couche 4. </br> Vos options sont : **Tout**, **TCP** ou **UDP**. |
| Délai d’inactivité (minutes) | Maintenez une connexion **TCP** ou **HTTP** ouverte sans compter sur des clients pour l’envoi de messages de persistance de connexion. |
| Réinitialisation du protocole TCP | L’équilibreur de charge peut envoyer des **réinitialisations du protocole TCP** pour aider à créer un comportement d’application plus prévisible lorsque la connexion est inactive. </br> En savoir plus sur la [réinitialisation du protocole TCP](load-balancer-tcp-reset.md). |
| Pool principal | Pool principal sur lequel vous souhaitez appliquer cette règle de trafic sortant. |

### <a name="port-allocation"></a>Allocation de ports

| Paramètre | Détails |
| ------- | ------ |
| Allocation de ports | Nous vous recommandons de sélectionner **Choisir manuellement le nombre de ports de sortie**.|

### <a name="outbound-ports"></a>Ports sortants

| Paramètre | Détails |
| ------- | ------ |
| Choisir par | Sélectionnez **Ports par instance** |
| Ports par instance | Entrez **10 000**. |

:::image type="content" source="./media/manage/outbound-rule.png" alt-text="Créer un équilibreur de charge public." border="true":::

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez pris connaissance des différents termes et paramètres du portail Azure pour Azure Load Balancer.

* [En savoir](./load-balancer-overview.md) plus sur Azure Load Balancer.
* [FAQ](./load-balancer-faqs.md) relatives à Azure Load Balancer.