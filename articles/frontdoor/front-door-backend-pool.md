---
title: Back-ends et pools de back-ends dans Azure Front Door | Microsoft Docs
description: Cet article décrit les serveurs principaux et pools principaux dans la configuration de Front Door.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 4beba141fec7a819df52e4c3a669312a4ad76998
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91449287"
---
# <a name="backends-and-backend-pools-in-azure-front-door"></a>Back-ends et pools de back-ends dans Azure Front Door
Cet article décrit des concepts sur la façon de mapper votre déploiement d’application web avec Azure Front Door. Il explique également les différentes terminologies utilisées dans la configuration de Front Door autour des serveurs principaux d’applications.

## <a name="backends"></a>Back-ends
Un serveur principal fait référence à un déploiement d’application web dans une région. Front Door prend en charge les ressources Azure et non Azure dans le pool principal. L’application peut se trouver dans votre centre de données local ou dans un autre fournisseur de cloud.

Les serveurs principaux de Front Door font référence au nom d’hôte ou à l’adresse IP publique de votre application qui sert les demandes de vos clients. Attention, il ne faut pas confondre les back-ends avec votre niveau de base de données, de stockage, etc. Les serveurs principaux doivent être considérés comme le point de terminaison public de votre serveur principal d’application. Lorsque vous ajoutez un serveur principal à un pool principal Front Door, vous devez également ajouter les éléments suivants :

- **Type d’hôte du back-end**. Le type de ressource que vous souhaitez ajouter. Front Door prend en charge la découverte automatique de vos back-ends d’application d’un service d’application, d’un service cloud ou d’un stockage. Si vous voulez une autre ressource Azure ou même un back-end non-Azure, sélectionnez **Hôte personnalisé**.

    >[!IMPORTANT]
    >Pendant la configuration, les API ne valident pas si le back-end est inaccessible depuis les environnements Front Door. Assurez-vous que Front Door peut atteindre votre back-end.

- **Abonnement et nom d’hôte du back-end**. Si vous n’avez pas sélectionné **Hôte personnalisé** pour le type d’hôte de back-end, sélectionnez votre back-end en choisissant l’abonnement approprié et le nom d’hôte de back-end correspondant dans l’interface utilisateur.

- **En-tête de l’hôte du back-end**. La valeur d’en-tête de l’hôte envoyée au back-end pour chaque requête. Pour plus d’informations, veuillez consulter [En-tête d’hôte du back-end](#hostheader).

- **Priorité**. Attribuez des priorités à vos back-end lorsque vous souhaitez utiliser un back-end de service principal pour tout le trafic. Fournissez également des sauvegardes si les back-ends principaux ou de secours ne sont pas disponibles. Pour plus d'informations, veuillez consulter [Priorité](front-door-routing-methods.md#priority).

- **Poids**. Attribuez des poids à vos différents back-ends si vous voulez répartir le trafic entre un ensemble de back-ends, que ce soit de façon équitable ou selon des coefficients de pondération. Pour plus d’informations, veuillez consulter [Poids](front-door-routing-methods.md#weighted).

### <a name="backend-host-header"></a><a name = "hostheader"></a>En-tête de l’hôte du backend

Les demandes transférées par Front Door à un back-end incluent un champ En-tête de l’hôte dont se sert le back-end pour récupérer la ressource cible. La valeur de ce champ provient généralement de l’URI du serveur principal qui contient l’en-tête et le port de l’hôte.

Par exemple, une demande formulée pour `www.contoso.com` aura l’en-tête d’hôte www.contoso.com. Si vous utilisez le portail Microsoft Azure pour configurer votre back-end, la valeur par défaut pour ce champ sera le nom d’hôte du back-end. Si votre back-end est contoso-westus.azurewebsites.net, dans le portail Microsoft Azure, la valeur remplie automatiquement pour l’en-tête d’hôte du back-end sera contoso-westus.azurewebsites.net. Toutefois, si vous utilisez des modèles Azure Resource Manager ou une autre méthode sans définir explicitement ce champ, Front Door transmet le nom d’hôte entrant comme valeur pour l’en-tête d’hôte. Si la requête a été faite pour www\.contoso.com et que votre back-end est contoso-westus.azurewebsites.net, dont le champ d’en-tête est vide, Front Door définit l’en-tête de l’hôte en tant que www\.contoso.com.

La plupart des back-endq d’application (comme Azure Web Apps, Stockage Blob et Cloud Services) exigent une correspondance entre l’en-tête de l’hôte et le domaine du back-end. Cependant, l’hôte front-end qui route vers votre back-end aura un nom d’hôte différent, comme www.contoso.net.

Si votre serveur principal requiert une correspondance entre l’en-tête de l’hôte et le nom d’hôte du serveur principal, veuillez vous assurer que l’en-tête de l’hôte principal inclut le nom de l’hôte du serveur principal.

#### <a name="configuring-the-backend-host-header-for-the-backend"></a>Configuration du champ En-tête de l’hôte backend pour le backend

Pour configurer le champ **En-tête de l’hôte du back-end** pour un back-end dans la section Pool de back-ends :

1. Ouvrez votre ressource Front Door et sélectionnez le pool de back-ends contenant le back-end à configurer.

2. Si vous ne l’avez pas déjà fait, ajoutez un back-end, ou modifiez-en un.

3. Définissez le champ d’en-tête de l’hôte du back-end sur une valeur personnalisée ou laissez-le vide. Le nom d’hôte pour la requête entrante sera utilisé en tant que valeur d’en-tête hôte.

## <a name="backend-pools"></a>Pools de back-ends
Dans Front Door, un pool de back-ends désigne l’ensemble des back-ends qui reçoivent un trafic similaire pour leur application. En d’autres termes, il s’agit d’un regroupement logique des instances de votre application dans le monde entier, qui reçoivent le même trafic et répondent avec un comportement attendu. Ces back-ends sont généralement déployés dans différentes régions ou au sein d’une même région. Tous les back-end peuvent être en mode de déploiement actif/actif (ou ce qui est défini en tant que configuration active/passive).

Un pool de back-ends définit comment différents back-ends doivent être évalués par le biais de sondes d’intégrité. Il définit également comment l’équilibrage de charge entre eux se produit.

### <a name="health-probes"></a>Sondes d’intégrité
Front Door envoie des requêtes de sonde HTTP/HTTPS périodiques à chacun de vos back-ends configurés. Les requêtes de sondes déterminent la proximité et l’intégrité de chaque back-end, afin d’équilibrer la charge des requêtes de vos utilisateurs finaux. Les paramètres de sonde d’intégrité d’un pool de back-ends définissent la façon dont nous sondons l’étant d’intégrité des back-ends de l’application. Les paramètres de configuration de l’équilibrage de charge disponibles sont les suivants :

- **Chemin d’accès** : L’URL utilisée pour les requêtes de sondage pour tous les back-ends du pool de back-ends. Par exemple, si un de vos back-ends est contoso-westus.azurewebsites.net et que le chemin est défini sur /probe/test.aspx, les environnements Front Door, en supposant que le protocole est défini sur HTTP, envoient les requêtes de sondes d’intégrité au chemin http\://contoso-westus.azurewebsites.net/probe/test.aspx.

- **Protocole** : Détermine si les requêtes de sondes d’intégrité de Front Door vers vos back-ends sont envoyées via le protocole HTTP ou HTTPS.

- **Méthode** : méthode HTTP à utiliser pour l’envoi des sondes d’intégrité. Les options incluent GET ou HEAD (par défaut).
    > [!NOTE]
    > Afin de limiter la charge et les coûts qui pèsent sur vos back-ends, Front Door recommande d’utiliser des requêtes HEAD pour les sondes d’intégrité.

- **Intervalle (en secondes)**  : Définit la fréquence des sondes d’intégrité de vos back-ends, ou les intervalles dans lequel chacun des environnements Front Door envoie une sonde.

    >[!NOTE]
    >Pour accélérer les basculements, baissez la valeur de l’intervalle. Plus la valeur est faible, plus le nombre de sondes d’intégrité de vos back-ends augmente. Par exemple, si l’intervalle est défini sur 30 secondes avec 100 points de présence (POP) Front Door dans le monde entier, chaque back-end reçoit environ 200 requêtes d’intégrité par minute.

Pour plus d’informations, veuillez consulter la section [Sondes d’intégrité](front-door-health-probes.md).

### <a name="load-balancing-settings"></a>Paramètres d’équilibrage de charge
Les paramètres d’équilibrage de charge du pool de back-ends définissent la façon dont nous évaluons les sondes d’intégrité. Ces paramètres déterminent si le serveur principal est sain ou pas. Ils vérifient également comment équilibrer la charge du trafic entre différents back-ends dans le pool de back-ends. Les paramètres de configuration de l’équilibrage de charge disponibles sont les suivants :

- **Taille d’échantillon** : Cette propriété identifie le nombre d’échantillons de sondes d’intégrité à prendre en compte pour évaluer l’intégrité d’un back-end.

- **Taille d’échantillon réussi**. Définit la taille d’échantillon comme mentionnée précédemment, le nombre d’échantillons réussis nécessaires pour appeler le back-end sain. Par exemple, imaginons le cas d’un intervalle de sonde d’intégrité Front Door de 30 secondes, dont la taille d’échantillon est de 5, et la taille d’échantillon réussi de 3. Chaque fois que nous évaluons les sondes d’intégrité pour votre back-end, nous examinons les 5 derniers échantillons pendant 150 secondes (5 x 30). Au moins 3 sondages réussis sont requis pour déclarer que le back-end est sain.

- **Sensibilité de latence (latence supplémentaire)** . Définit si vous souhaitez que Front Door envoie la requête aux back-ends dans la plage de sensibilité de mesure de latence ou transfère la requête vers le back-end le plus proche.

Pour plus d’informations, consultez [Routage du trafic basé sur les latences les plus faibles](front-door-routing-methods.md#latency).

## <a name="next-steps"></a>Étapes suivantes

- [Créer un profil Front Door](quickstart-create-front-door.md)
- [Fonctionnement de Front Door](front-door-routing-architecture.md)
