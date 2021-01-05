---
title: Fonctionnement d’une passerelle d’application
description: Cet article fournit des informations sur la façon dont une passerelle d’application accepte les requêtes entrantes et dont elle les route vers le serveur principal.
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: conceptual
ms.date: 11/16/2019
ms.author: absha
ms.openlocfilehash: 9166125fac28f43a93cbee2875b91bee986b1400
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397465"
---
# <a name="how-an-application-gateway-works"></a>Fonctionnement d’une passerelle d’application

Cet article explique comment une passerelle d’application accepte les requêtes entrantes, et comment elle les route vers le back-end.

![Comment une passerelle d’application accepte une requête](./media/how-application-gateway-works/how-application-gateway-works.png)

## <a name="how-an-application-gateway-accepts-a-request"></a>Comment une passerelle d’application accepte une requête

1. Avant qu’un client envoie une requête à une passerelle d’application, il résout le nom de domaine de la passerelle d’application à l’aide d’un serveur DNS (Domain Name System). Azure contrôle l’entrée DNS, car toutes les passerelles d’application sont dans le domaine azure.com.

2. Le DNS Azure retourne l’adresse IP au client. Il s’agit de l’adresse IP front-end de la passerelle d’application.

3. La passerelle d’application accepte le trafic entrant sur un ou plusieurs écouteurs. Un écouteur est une entité logique qui vérifie les requêtes de connexion. Il est configuré avec un numéro de port, un protocole et une adresse IP front-end pour les connexions des clients à la passerelle d’application.

4. Si un WAF (pare-feu d’applications web) est utilisé, la passerelle d’application vérifie les en-têtes de requête et le corps, le cas échéant, par rapport aux règles WAF. Cette action détermine si la requête est valide ou si elle représente une menace pour la sécurité. Si la requête est valide, elle est routée vers le back-end. Si la requête est non valide et que le pare-feu d’applications web est en mode Prévention, elle est bloquée en tant que menace pour la sécurité. S’il est en mode Détection, la requête est évaluée et journalisée, mais toujours transférée au serveur back-end.

Azure Application Gateway peut être utilisé en tant qu’équilibreur de charge d’application interne ou en tant qu’équilibreur de charge d’application accessible sur Internet. Une passerelle d’application accessible sur Internet utilise des adresses IP publiques. Le nom DNS d’une passerelle d’application accessible sur Internet peut être résolu publiquement en son adresse IP publique. Ainsi, les passerelles d’application accessibles sur Internet peuvent router les requêtes des clients vers Internet.

Les passerelles d’application internes utilisent uniquement des adresses IP privées. Si vous utilisez une [zone DNS privée](../dns/private-dns-overview.md) ou personnalisée, le nom de domaine doit pouvoir être résolu en l’adresse IP privée d’Application Gateway en interne. Les équilibreurs de charge internes peuvent donc router uniquement les requêtes des clients ayant accès à un réseau virtuel pour la passerelle d’application.

## <a name="how-an-application-gateway-routes-a-request"></a>Comment une passerelle d’application route une requête

Si une requête est valide et qu’elle n’est pas bloquée par le pare-feu d’applications web, la passerelle d’application évalue la règle de routage de requête associée à l’écouteur. Cette action permet de déterminer le pool de back-ends vers lequel router la requête.

Selon la règle de routage de requête, la passerelle d’application détermine si toutes les requêtes de l’écouteur doivent être routées vers un pool de back-ends spécifique, si elles doivent être routées vers des pools de back-ends distincts en fonction du chemin de l’URL, ou si elles doivent être redirigées vers un autre port ou un site externe.
>[!NOTE]
>Les règles sont traitées dans l’ordre où elles sont listées dans le portail pour la référence SKU v1. 

Quand la passerelle d’application sélectionne le pool de back-ends, elle envoie la requête à l’un des back-ends sains du pool (y.y.y.y). L’intégrité du serveur est déterminée par une sonde. Si le pool de back-ends contient plusieurs serveurs, la passerelle d’application utilise un algorithme de tourniquet (round robin) pour router les requêtes entre les serveurs sains. Cela permet d’équilibrer la charge des requêtes sur les serveurs.

Une fois que la passerelle d’application a sélectionné le back-end, elle ouvre une nouvelle session TCP auprès du back-end en fonction des paramètres HTTP. Les paramètres HTTP spécifient le protocole, le port et les autres paramètres de routage nécessaires à l’établissement d’une nouvelle session auprès du back-end.

Le port et le protocole utilisés dans les paramètres HTTP permettent de déterminer si le trafic entre la passerelle d’application et les back-ends est chiffré (chiffrement TLS de bout en bout) ou non chiffré.

Quand une passerelle d’application envoie la requête d’origine au back-end, elle respecte la configuration personnalisée définie dans les paramètres HTTP associés au remplacement du nom d’hôte, du chemin et du protocole. Cette action permet de conserver l’affinité de session basée sur les cookies, le drainage de connexion, la sélection du nom d’hôte à partir du back-end, etc.

 >[!NOTE]
>Si le pool de back-ends :
> - **Est un point de terminaison public** , la passerelle d’application utilise son adresse IP publique front-end pour joindre le serveur. En l’absence d’adresse IP publique front-end, une adresse est affectée pour la connectivité externe sortante.
> - **Contient un FQDN pouvant être résolu de façon interne ou une adresse IP privée** , la passerelle d’application route la requête vers le back-end à l’aide de ses adresses IP privées d’instance.
> - **Contient un point de terminaison externe ou un FQDN pouvant être résolu de manière externe** , la passerelle d’application route la requête vers le back-end à l’aide de son adresse IP publique front-end. La résolution DNS est basée sur une zone DNS privée ou un serveur DNS personnalisé, si ce type de configuration existe, ou utilise le DNS par défaut fourni par Azure. En l’absence d’adresse IP publique front-end, une adresse est affectée pour la connectivité externe sortante.

### <a name="modifications-to-the-request"></a>Modifications apportées à la requête

Une passerelle d’application insère quatre en-têtes supplémentaires dans toutes les requêtes avant de les transférer au back-end. Il s’agit d’en-têtes x-forwarded-for, x-forwarded-proto, x-forwarded-port et x-original-host. Le format d’en-tête x-forwarded-for est une liste d’éléments IP:Port séparés par des virgules.

Les valeurs valides pour x-forwarded-proto sont HTTP ou HTTPS. X-forwarded-port spécifie le port où la requête a pu joindre la passerelle d’application. L’en-tête X-original-host contient l’en-tête d’origine de l’hôte avec lequel la requête est arrivée. Cet en-tête est utile dans l’intégration de sites web Azure, où l’en-tête de l’hôte entrant est modifié avant que le trafic ne soit routé vers le back-end. Si l’option d’affinité de session est activée, un cookie d’affinité géré par la passerelle est ajouté.

Vous pouvez configurer la passerelle d’application pour qu’elle modifie les en-têtes de requête et de réponse par la [réécriture des en-têtes HTTP et URL](rewrite-http-headers-url.md) ou pour qu’elle modifie le chemin de l’URI à l’aide d’un paramètre de substitution de chemin. Toutefois, à moins que la passerelle d’application ne soit configurée dans ce but, toutes les requêtes entrantes sont proxysées vers le back-end.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur les composants de passerelle d’application](application-gateway-components.md)