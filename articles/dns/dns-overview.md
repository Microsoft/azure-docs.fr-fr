---
title: Présentation d’Azure DNS
description: Vue d’ensemble des services d’hébergement DNS sur Microsoft Azure. Héberger votre domaine sur Microsoft Azure
author: rohinkoul
ms.service: dns
ms.topic: overview
ms.date: 3/21/2019
ms.author: rohink
ms.openlocfilehash: 7c3aa5505ca15238b2f9e84069de58ab6b700521
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608485"
---
# <a name="what-is-azure-dns"></a>Présentation d’Azure DNS

Azure DNS est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS à l’aide des mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.

Vous ne pouvez pas utiliser Azure DNS pour acheter un nom de domaine. Moyennant un abonnement annuel, vous pouvez acheter un nom de domaine à l’aide des [domaines App Service](../app-service/manage-custom-dns-buy-domain.md#buy-an-app-service-domain) ou d’un bureau d’enregistrement de nom de domaine tiers. Vos domaines peuvent alors être hébergés dans Azure DNS pour la gestion des enregistrements. Pour plus d’informations, voir [Délégation de domaine à Azure DNS](dns-domain-delegation.md).

Les fonctionnalités suivantes sont incluses dans Azure DNS.

## <a name="reliability-and-performance"></a>Fiabilité et performances

Les domaines DNS dans Azure DNS sont hébergés sur un réseau global de serveurs de noms DNS. Azure DNS utilise la mise en réseau Anycast. Chaque requête DNS obtient une réponse du serveur DNS disponible le plus proche, pour des performances élevées et une haute disponibilité de domaine.

## <a name="security"></a>Sécurité

 Azure DNS est basé sur Azure Resource Manager, vous donnant l’accès à des fonctionnalités telles que :

* [Contrôle d’accès en fonction de rôles (Azure RBAC)](../azure-resource-manager/management/overview.md) : pour déterminer les utilisateurs qui sont autorisés à accéder à des actions spécifiques pour votre organisation.

* [Journaux d’activité](../azure-resource-manager/management/overview.md) : pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

* [Verrouillage de ressource](../azure-resource-manager/management/lock-resources.md) : pour verrouiller un abonnement, un groupe de ressources ou une ressource. Le verrouillage empêche les utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources critiques.

Pour plus d’informations, consultez [Comment protéger les enregistrements et zones DNS](dns-protect-zones-recordsets.md). 

## <a name="dnssec"></a>DNSSEC

Azure DNS ne prend actuellement pas en charge DNSSEC. Dans la plupart des cas, vous pouvez limiter le recours à DNSSEC en utilisant systématiquement HTTPS/TLS dans vos applications. Si DNSSEC est une exigence incontournable pour vos zones DNS, vous pouvez héberger ces zones avec des fournisseurs d’hébergement DNS tiers.

## <a name="ease-of-use"></a>Simplicité d'utilisation

 Azure DNS peut gérer les enregistrements DNS de vos services Azure et fournir un DNS pour vos ressources externes. Azure DNS est intégré au portail Azure et utilise les mêmes informations d’identification, de facturation et de contrat d’assistance que vos autres services Azure. 

La facturation DNS est basée sur le nombre de zones DNS hébergées dans Azure et sur le nombre de requêtes DNS reçues. Pour en savoir plus sur la tarification, consultez [Tarification d’Azure DNS](https://azure.microsoft.com/pricing/details/dns/).

Vos domaines et enregistrements peuvent être gérés via le portail Azure, des cmdlets Azure PowerShell et l’interface CLI Azure multiplateforme. Les applications nécessitant une gestion automatisée de DNS peuvent s’intégrer au service par le biais de l’API REST et des Kits de développement logiciel (SDK).

## <a name="customizable-virtual-networks-with-private-domains"></a>Réseaux virtuels personnalisables avec des domaines privés

Azure DNS prend désormais également en charge les domaines DNS privés. Cette fonctionnalité vous permet d’utiliser vos propres noms de domaine personnalisés dans vos réseaux virtuels privés au lieu des noms fournis par Azure.

Pour plus d’informations, consultez [Utilisation d’Azure DNS pour les domaines privés](private-dns-overview.md).

## <a name="alias-records"></a>Enregistrements d’alias

Azure DNS prend en charge les jeux d’enregistrements d’alias. Vous pouvez utiliser un jeu d’enregistrements d’alias pour référencer une ressource Azure, comme une adresse IP publique Azure, un profil Azure Traffic Manager ou point de terminaison Azure Content Delivery Network. Si l’adresse IP de la ressource sous-jacente change, le jeu d’enregistrements d’alias se met à jour de façon fluide pendant la résolution DNS. Le jeu d’enregistrements d’alias pointe vers l’instance du service, laquelle est associée à une adresse IP.

Par ailleurs, vous pouvez maintenant pointer votre apex ou votre domaine nu vers un profil Traffic Manager ou un point de terminaison CDN avec un enregistrement d’alias. contoso.com en est un exemple.

Pour plus d’informations, consultez [Vue d’ensemble des enregistrements d’alias Azure DNS](dns-alias.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les enregistrements et zones DNS, consultez [Vue d’ensemble des enregistrements et zones DNS](dns-zones-records.md).

* Pour en savoir plus sur la création d’une zone dans Azure DNS, consultez [Créer une zone DNS](./dns-getstarted-portal.md).

* Pour les questions fréquemment posées sur Azure DNS, consultez le [FAQ sur Azure DNS](dns-faq.md).