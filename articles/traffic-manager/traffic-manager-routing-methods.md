---
title: Méthodes de routage du trafic Azure Traffic Manager
description: Cet article va vous aider à comprendre les différentes méthodes de routage du trafic utilisées par Traffic Manager.
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: duau
ms.openlocfilehash: 3cf493beab6dfe1767ae35ea36732dc364e29736
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89401654"
---
# <a name="traffic-manager-routing-methods"></a>Méthodes de routage de Traffic Manager

Azure Traffic Manager prend en charge six méthodes de routage du trafic pour déterminer comment acheminer le trafic réseau vers les différents points de terminaison de service. Pour tout profil, Traffic Manager applique à chaque requête DNS qu’il reçoit la méthode de routage du trafic associée au profil. La méthode de routage du trafic détermine le point de terminaison retourné dans la réponse DNS.

Les méthodes de routage du trafic suivantes sont disponibles dans Traffic Manager :

* **[Priorité ](#priority-traffic-routing-method):** sélectionnez **Priority** (Priorité) si vous souhaitez utiliser un point de terminaison de service principal pour tout le trafic et disposer de sauvegardes au cas où les points de terminaison principaux ou de sauvegarde ne sont pas disponibles.
* **[Pondération ](#weighted):** sélectionnez **Weighted** (Pondéré) si vous souhaitez distribuer le trafic entrant sur un ensemble de points de terminaison, soit uniformément, soit en fonction du poids que vous définissez.
* **[Performances ](#performance):** sélectionnez **Performance** quand vos points de terminaison se trouvent sur des emplacements géographiques différents et que vous souhaitez que les utilisateurs finaux utilisent le point de terminaison « le plus proche » en termes de latence réseau la plus faible.
* **[Géographique ](#geographic):** sélectionnez **Geographic** (Géographique) pour diriger les utilisateurs vers des points de terminaison spécifiques (Azure, externes ou imbriqués) selon l’emplacement géographique dont leur requête DNS provient. Cette option permet aux clients Traffic Manager de mettre en œuvre des scénarios où il est important de connaître la région géographique des utilisateurs et de router leur trafic en fonction de celle-ci. Exemples : respect des obligations en matière de souveraineté des données, localisation de contenu et d’expérience utilisateur, mesure du trafic en provenance de différentes régions.
* **[Valeurs multiples](#multivalue) :** sélectionnez **Valeurs multiples** pour les profils Traffic Manager qui ne peuvent avoir que des adresses IPv4/IPv6 en tant que points de terminaison. Quand une demande est reçue pour ce profil, tous les points de terminaison intègres sont retournés.
* **[Sous-réseau](#subnet) :** sélectionnez la méthode de routage du trafic **Sous-réseau** pour mapper des ensembles de plages d’adresses IP d’utilisateur final à un point de terminaison spécifique au sein d’un profil Traffic Manager. Quand une demande est reçue, le point de terminaison retourné est celui qui est mappé pour l’adresse IP source de cette demande. 


Tous les profils Traffic Manager incluent une surveillance de l’intégrité des points de terminaison et un basculement de point de terminaison automatique. (pour plus d’informations, voir la rubrique relative à la [surveillance des points de terminaison avec Traffic Manager](traffic-manager-monitoring.md)) ; Un profil Traffic Manager donné ne peut utiliser qu’une seule méthode de routage du trafic. Vous pouvez sélectionner une méthode de routage du trafic différente pour votre profil à tout moment. Les modifications sont appliquées dans la minute, sans aucun temps d’arrêt. Les méthodes de routage du trafic peuvent être combinées dans des profils Traffic Manager imbriqués. Une imbrication permet de créer des configurations de routage du trafic sophistiquées et flexibles répondant aux besoins d’applications complexes plus importantes. Pour plus d’informations, consultez [Profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md).

## <a name="priority-traffic-routing-method"></a>Méthode de routage du trafic basé sur la priorité

Souvent, une organisation souhaite assurer la fiabilité de ses services en déployant un ou plusieurs services de sauvegarde utilisables en cas de défaillance du service principal. La méthode de routage du trafic « Priorité » permet aux clients Azure d’implémenter facilement ce modèle de basculement.

![Méthode de routage du trafic « Priorité » d’Azure Traffic Manager](media/traffic-manager-routing-methods/priority.png)

Le profil Traffic Manager contient une liste hiérarchisée de points de terminaison de service. Par défaut, Traffic Manager envoie tout le trafic vers le point de terminaison principal (priorité la plus élevée). Si le point de terminaison principal n’est pas disponible, Traffic Manager route le trafic vers le deuxième point de terminaison. Si les points de terminaison de type principal et secondaire ne sont pas disponibles, le trafic est envoyé vers le troisième, et ainsi de suite. La disponibilité du point de terminaison est basée sur l’état configuré (activé ou désactivé) et la surveillance du point de terminaison en cours.

### <a name="configuring-endpoints"></a>Configuration de points de terminaison

Avec Azure Resource Manager, vous configurez la priorité de point de terminaison explicitement à l’aide de la propriété « priority » pour chaque point de terminaison. Cette propriété est une valeur comprise entre 1 et 1000. Plus la valeur est basse, plus la priorité est élevée. Des points de terminaison ne peuvent pas partager des valeurs de priorité. La définition de la propriété est facultative. En cas d’omission, une priorité par défaut basée sur l’ordre du point de terminaison est utilisée.

## <a name="weighted-traffic-routing-method"></a><a name = "weighted"></a>Méthode de routage du trafic basé sur la pondération
La méthode de routage du trafic « Pondéré » vous permet de répartir le trafic uniformément ou d’utiliser une pondération prédéfinie.

![Méthode de routage du trafic « Pondéré » d’Azure Traffic Manager](media/traffic-manager-routing-methods/weighted.png)

Dans la méthode de routage du trafic Pondéré, vous affectez une pondération à chaque point de terminaison dans la configuration du profil Traffic Manager. La pondération est un entier compris entre 1 et 1000. Ce paramètre est facultatif. En cas d’omission, Traffic Manager utilise la pondération par défaut « 1 ». Plus le poids est élevé, plus la priorité est forte.

Pour chaque requête DNS reçue, Traffic Manager choisit au hasard un point de terminaison disponible. La probabilité de choisir un point de terminaison dépend des pondérations assignées à tous les points de terminaison disponibles. L’utilisation de la même pondération pour tous les points de terminaison produit une distribution uniforme du trafic. L’utilisation de pondérations supérieures ou inférieures sur certains points de terminaison a pour effet que ceux-ci sont retournés plus ou moins fréquemment dans les réponses DNS.

La méthode pondérée permet des scénarios utiles :

* Mise à niveau progressive d’une application : allouez un pourcentage de trafic à acheminer vers un nouveau point de terminaison, puis augmentez progressivement le trafic au fil du temps jusqu’à 100 %.
* Migration d’application vers Azure : créez un profil avec des points de terminaison Azure et externes. Ajustez la pondération des points de terminaison pour privilégier les nouveaux points de terminaison.
* Extension du cloud pour une capacité supplémentaire : étendez rapidement un déploiement local dans le cloud en le plaçant derrière un profil Traffic Manager. Quand vous avez besoin d'une capacité supplémentaire dans le cloud, vous pouvez ajouter ou activer des points de terminaison supplémentaires et spécifier quelle partie du trafic est destinée à chaque point de terminaison.

En plus de l’utilisation du portail Azure, vous pouvez configurer des pondérations à l’aide d’Azure PowerShell, d’Azure CLI et d’API REST.

Il est important de comprendre que les réponses DNS sont mises en cache par les clients et les serveurs DNS récursifs que les clients utilisent pour résoudre les noms DNS. Cette mise en cache peut avoir une incidence sur les distributions de trafic pondérées. Lorsque le nombre de clients et de serveurs DNS récursifs est élevé, la distribution du trafic fonctionne comme prévu. En revanche, si le nombre de clients ou de serveurs DNS récursifs est peu élevé, la mise en cache peut fausser sensiblement la distribution du trafic.

Les cas d’utilisation courants sont les suivants :

* les environnements de développement et de test ;
* les communications d’application à application ;
* les applications destinées à une base étroite d’utilisateurs qui partagent une infrastructure DNS récursive commune (par exemple, des employés d’une entreprise se connectant via un proxy).

Ces effets de mise en cache DNS sont communs à tous les systèmes de routage du trafic basés sur DNS, pas seulement à Azure Traffic Manager. Dans certains cas, la solution peut constituer à effacer le cache DNS de manière explicite. Dans d’autres cas, une autre méthode de routage du trafic peut être plus appropriée.

## <a name="performance-traffic-routing-method"></a><a name = "performance"></a>Méthode de routage du trafic basé sur les performances

Un déploiement de points de terminaison en deux emplacements ou plus dans le monde peut améliorer la réactivité de nombreuses applications en routant le trafic vers l’emplacement « le plus proche » de vous. La méthode de routage du trafic « Performance » offre cette possibilité.

![Méthode de routage du trafic « Performance » d’Azure Traffic Manager](media/traffic-manager-routing-methods/performance.png)

Le point de terminaison « le plus proche » n’est pas nécessairement le plus proche en termes de distance géographique. Au lieu de cela, la méthode de routage du trafic « Performance » détermine le point de terminaison le plus proche en mesurant le temps de réponse (ou latence) du réseau. Traffic Manager tient à jour une Table de latence Internet pour suivre le temps d’aller-retour entre les plages d’adresses IP et chaque centre de données Azure.

Traffic Manager recherche l’adresse IP source de la demande DNS entrante dans la Table de latence Internet. Traffic Manager choisit ensuite un point de terminaison disponible dans le centre de données Azure, dont la latence est la plus faible pour la plage d’adresses IP, puis retourne ce point de terminaison dans la réponse DNS.

Comme expliqué dans [Fonctionnement de Traffic Manager](traffic-manager-how-it-works.md), Traffic Manager ne reçoit pas de requêtes DNS provenant directement de clients. Au lieu de cela, les requêtes DNS proviennent du service DNS récursif que les clients sont configurés pour utiliser. Par conséquent, l’adresse IP utilisée pour déterminer le point de terminaison « le plus proche » n’est pas l’adresse IP du client, mais celle du service DNS récursif. Dans la pratique, cette adresse IP est un bon proxy pour le client.


Traffic Manager met régulièrement à jour la Table de latence Internet pour refléter les modifications de l’Internet global et des nouvelles régions Azure. Toutefois, les performances des applications varient en fonction des variations en temps réel de la charge sur Internet. Le routage du trafic Performance ne surveille pas la charge sur un point de terminaison de service donné. En revanche, si un point de terminaison devient indisponible, Traffic Manager ne l’inclut pas dans les réponses aux requêtes DNS.


Points à noter :

* Si votre profil contient plusieurs points de terminaison situés dans la même région Azure, Traffic Manager répartit uniformément le trafic entre les points de terminaison disponibles dans cette région. Si vous préférez une distribution de trafic différente au sein d’une région, vous pouvez utiliser des [profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md).
* Si tous les points de terminaison activés dans la région Azure la plus proche sont défectueux, Traffic Manager déplace le trafic vers les points de terminaison dans la région Azure la plus proche suivante. Si vous souhaitez définir une séquence de basculement préférée, utilisez des [profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md).
* Si vous utilisez la méthode de routage du trafic basé sur les performances avec des points de terminaison externes ou des points de terminaison imbriqués, vous devez spécifier l’emplacement de ces points de terminaison. Choisissez la région Azure la plus proche de votre déploiement. Ces emplacements sont les valeurs prises en charge par la Table de latence Internet.
* L’algorithme qui choisit le point de terminaison est déterministe. Des requêtes DNS répétées à partir du même client sont dirigées vers le même point de terminaison. En règle générale, les clients utilisent des serveurs DNS récursifs différents quand ils se déplacent. Le client peut être routé vers un point de terminaison différent. Le routage peut également être affecté par des mises à jour de la Table de latence Internet. Par conséquent, la méthode de routage du trafic Performance ne garantit pas qu’un client est toujours routé vers le même point de terminaison.
* Lorsque la Table de latence Internet change, il se peut que certains clients soient dirigés vers un point de terminaison différent. La précision de ce changement de routage varie en fonction des données de latence en cours. Ces mises à jour sont essentielles pour conserver l’exactitude du routage du trafic « Performance », car Internet évolue en permanence.

## <a name="geographic-traffic-routing-method"></a><a name = "geographic"></a>Méthode de routage du trafic Geographic (Géographique)

Les profils Traffic Manager peuvent être configurés de manière à utiliser la méthode de routage Geographic (Géographique) pour diriger les utilisateurs vers des points de terminaison spécifiques (Azure, externes ou imbriqués) selon l’emplacement géographique dont leur requête DNS provient. Cette option permet aux clients Traffic Manager de mettre en œuvre des scénarios où il est important de connaître la région géographique des utilisateurs et de router leur trafic en fonction de celle-ci. Exemples : respect des obligations en matière de souveraineté des données, localisation de contenu et d’expérience utilisateur, mesure du trafic en provenance de différentes régions.
Lorsqu’un profil est configuré pour le routage géographique, un ensemble de régions géographiques doit être affecté à chaque point de terminaison associé à ce profil. Une région géographique peut présenter les niveaux de granularité suivants : 
- World (Monde) : n’importe quelle région
- Regional Grouping (Regroupement régional) : Afrique, Moyen-Orient, Australie/Pacifique, etc. 
- Country/Region (Pays/Région) : Irlande, Pérou, Hong Kong (R.A.S.), etc. 
- State/Province (État/Province) : États-Unis-Californie, Australie-Queensland, Canada-Alberta, etc. (remarque : ce niveau de granularité est pris en charge uniquement pour les états/provinces de l’Australie, du Canada et des États-Unis).

Lorsqu’une région ou un ensemble de régions est affecté à un point de terminaison, toutes les demandes provenant de ces régions sont routées exclusivement vers ce point de terminaison. Traffic Manager utilise l’adresse IP source de la requête DNS pour déterminer la région où l’utilisateur effectue sa requête. Il s’agit en général de l’adresse IP du programme de résolution DNS local qui effectue la requête pour le compte de l’utilisateur.  

![Méthode de routage du trafic « Geographic » (Géographique) d’Azure Traffic Manager](./media/traffic-manager-routing-methods/geographic.png)

Traffic Manager analyse l’adresse IP source de la requête DNS afin de déterminer la région géographique d’origine de cette dernière. Il examine ensuite s’il existe un point de terminaison auquel cette région géographique est mappée. Cette recherche commence au niveau de granularité le plus bas (State/Province [État/Province] où il est pris en charge, Pays/Région ailleurs) et remonte jusqu’au niveau le plus élevé, à savoir **World** (Monde). La première correspondance trouvée à l’aide de ce balayage est désignée en tant que point de terminaison à renvoyer dans la réponse à la requête. En cas de correspondance avec un point de terminaison de type imbriqué, un point de terminaison de ce profil enfant est renvoyé en fonction de sa méthode de routage. Les points suivants s’appliquent à ce comportement :

- Une zone géographique peut être mappée à un seul point de terminaison dans un profil Traffic Manager lorsque le type de routage est défini sur Geographic (Géographique). Cela garantit un routage déterministe du trafic des utilisateurs et permet aux clients de mettre en œuvre des scénarios nécessitant des limites géographiques claires.
- Si la région d’un utilisateur correspond au mappage géographique de deux points de terminaison différents, Traffic Manager sélectionne le point de terminaison présentant le niveau de granularité le plus bas et n’envisage pas la possibilité de router les demandes provenant de cette région vers l’autre point de terminaison. Par exemple, considérons un profil présentant le type de routage Geographic (Géographique) et deux points de terminaison, Endpoint1 et Endpoint2. Endpoint1 est configuré pour recevoir le trafic provenant d’Irlande et Endpoint2 est configuré pour recevoir le trafic provenant d’Europe. Une demande provenant d’Irlande est systématiquement routée vers le point de terminaison Endpoint1.
- Comme une région peut être mappée à un seul point de terminaison, Traffic Manager renvoie ce point de terminaison, qu’il soit intègre ou non.

    >[!IMPORTANT]
    >Il est vivement recommandé aux clients utilisant la méthode de routage Geographic (Géographique) de l’associer avec les points de terminaison de type imbriqué qui présentent des profils enfants contenant au moins deux points de terminaison.
- Si une correspondance de point de terminaison est trouvée et que ce point de terminaison se trouve à l’état **Arrêté**, Traffic Manager renvoie une réponse NODATA. Dans ce cas, aucune autre recherche n’est effectuée plus haut dans la hiérarchie de régions géographiques. Ce comportement s’applique également aux points de terminaison de type imbriqué lorsque le profil enfant se trouve à l’état **Arrêté** ou **Désactivé**.
- Si un point de terminaison se trouve à l’état **Désactivé**, il n’est pas inclus dans le processus de mise en correspondance avec la région. Ce comportement s’applique également aux points de terminaison de type imbriqué lorsque le point de terminaison se trouve à l’état **Désactivé**.
- Si une requête provient d’une région géographique qui n’a aucun mappage dans ce profil, Traffic Manager renvoie une réponse NODATA. Par conséquent, il est vivement recommandé aux clients d’utiliser le routage géographique avec un point de terminaison, idéalement de type imbriqué avec au moins deux points de terminaison dans le profil enfant, auquel la région **World** (Monde) est affectée. Cela garantit également que les adresses IP qui ne sont mappées à aucune région sont gérées.

Comme expliqué dans [Fonctionnement de Traffic Manager](traffic-manager-how-it-works.md), Traffic Manager ne reçoit pas de requêtes DNS provenant directement de clients. Au lieu de cela, les requêtes DNS proviennent du service DNS récursif que les clients sont configurés pour utiliser. Par conséquent, l’adresse IP utilisée pour déterminer la région n’est pas l’adresse IP du client, mais celle du service DNS récursif. Dans la pratique, cette adresse IP est un bon proxy pour le client.

### <a name="faqs"></a>FAQ

* [Quels sont les cas d’utilisation dans lesquels le routage géographique est utile ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-some-use-cases-where-geographic-routing-is-useful)

* [Comment décider si je dois utiliser la méthode de routage de performances ou la méthode de routage géographique ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-do-i-decide-if-i-should-use-performance-routing-method-or-geographic-routing-method)

* [Quelles sont les régions prises en charge par Traffic Manager pour le routage géographique ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-the-regions-that-are-supported-by-traffic-manager-for-geographic-routing)

* [Comment Traffic Manager détermine-t-il l’emplacement à partir duquel un utilisateur exécute une requête ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-traffic-manager-determine-where-a-user-is-querying-from)

* [Est-il garanti que Traffic Manager détermine correctement l’emplacement géographique exact de l’utilisateur dans tous les cas ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#is-it-guaranteed-that-traffic-manager-can-correctly-determine-the-exact-geographic-location-of-the-user-in-every-case)

* [Pour le routage géographique, un point de terminaison doit-il être situé physiquement dans la même région que celle avec laquelle il est configuré ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#does-an-endpoint-need-to-be-physically-located-in-the-same-region-as-the-one-it-is-configured-with-for-geographic-routing)

* [Puis-je affecter des régions géographiques aux points de terminaison dans un profil qui n’est pas configuré pour procéder au routage géographique ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-assign-geographic-regions-to-endpoints-in-a-profile-that-is-not-configured-to-do-geographic-routing)

* [Pourquoi une erreur est-elle générée lorsque j’essaie de modifier la méthode de routage d’un profil existant en méthode de routage géographique ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#why-am-i-getting-an-error-when-i-try-to-change-the-routing-method-of-an-existing-profile-to-geographic)

* [Pourquoi est-il vivement recommandé que les clients créent des profils imbriqués à la place de points de terminaison sous un profil avec le routage géographique activé ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#why-is-it-strongly-recommended-that-customers-create-nested-profiles-instead-of-endpoints-under-a-profile-with-geographic-routing-enabled)

* [Existe-t-il des restrictions quant à la version de l’API qui prend en charge ce type de routage ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#are-there-any-restrictions-on-the-api-version-that-supports-this-routing-type)

## <a name="multivalue-traffic-routing-method"></a><a name = "multivalue"></a>Méthode de routage du trafic Valeurs multiples
La méthode de routage du trafic **Valeurs multiples** vous permet d’obtenir plusieurs points de terminaison intègres dans une seule réponse à une requête DNS. Cela permet à l’appelant de faire de nouvelles tentatives côté client avec d’autres points de terminaison quand un point de terminaison retourné ne répond pas. Ce modèle peut augmenter la disponibilité d’un service et réduire la latence associée à une nouvelle requête DNS pour obtenir un point de terminaison sain. La méthode de routage Valeurs multiples ne fonctionne que si tous les points de terminaison sont de type « Externe » et sont des adresses IPv4 ou IPv6 spécifiées. Lors de la réception d’une requête pour ce profil, tous les points de terminaison sains sont retournés et soumis à un nombre de retours maximal configurable.

### <a name="faqs"></a>FAQ

* [Quels sont les cas d’utilisation dans lesquels le routage MultiValue est utile ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-some-use-cases-where-multivalue-routing-is-useful)

* [Combien de points de terminaison sont renvoyés avec le routage MultiValue ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-many-endpoints-are-returned-when-multivalue-routing-is-used)

* [Obtiendrais-je le même ensemble de points de terminaison avec le routage MultiValue ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#will-i-get-the-same-set-of-endpoints-when-multivalue-routing-is-used)

## <a name="subnet-traffic-routing-method"></a><a name = "subnet"></a>Méthode de routage du trafic Sous-réseau
La méthode de routage du trafic **Sous-réseau** vous permet de mapper un ensemble de plages d’adresses IP d’utilisateur final à des points de terminaison spécifiques dans un profil. Ensuite, si Traffic Manager reçoit une requête DNS pour ce profil, il inspecte l’adresse IP source de cette demande (dans la plupart des cas, il s’agit de l’adresse IP sortante de la résolution DNS utilisée par l’appelant), détermine le point de terminaison auquel elle est mappée, et retourne ce point de terminaison dans la réponse à la requête. 

L’adresse IP à associer à un point de terminaison peut être spécifiée en tant que plage CIDR (par exemple, 1.2.3.0/24) ou plage d’adresses (par exemple, 1.2.3.4-5.6.7.8). Les plages d’adresses IP associées à un point de terminaison doivent être uniques au sein de ce profil, et il ne peut pas y avoir de chevauchement avec l’ensemble d’adresses IP d’un autre point de terminaison dans le même profil.
Si vous définissez un point de terminaison sans plage d’adresses, il fait office de secours et récupère le trafic de n’importe quel sous-réseau restant. Si aucun point de terminaison de secours n’est inclus, Traffic Manager envoie une réponse NODATA pour toutes les plages non définies. Il est par conséquent hautement recommandé de définir un point de terminaison de secours ou de vérifier que toutes les plages d’adresses IP possibles sont spécifiées sur vos points de terminaison.

Un routage de sous-réseau permet d’offrir une expérience différente au utilisateurs qui se connectent à partir d’un espace d’adressage IP spécifique. Par exemple, en utilisant un routage de sous-réseau, un client peut faire en sorte que toutes les demandes émanant de son bureau d’entreprise soient acheminées vers un autre point de terminaison où il peut tester une version uniquement interne de son application. Un autre scénario est quand vous souhaitez offrir une expérience différente aux utilisateurs qui se connectent à partir d’un fournisseur de services Internet (ISP) spécifique (par exemple, pour bloquer les utilisateurs d’un ISP donné).

### <a name="faqs"></a>FAQ

* [Quels sont les cas d’utilisation dans lesquels le routage en fonction du sous-réseau est utile ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-some-use-cases-where-subnet-routing-is-useful)

* [Comment Traffic Manager connaît-il l’adresse IP de l’utilisateur final ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-traffic-manager-know-the-ip-address-of-the-end-user)

* [Comment puis-je spécifier des adresses IP avec le routage en fonction du sous-réseau ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-specify-ip-addresses-when-using-subnet-routing)

* [Comment puis-je spécifier un point de terminaison de secours avec le routage en fonction du sous-réseau ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-specify-a-fallback-endpoint-when-using-subnet-routing)

* [Que se passe-t-il si un point de terminaison est désactivé dans un profil de type de routage en fonction du sous-réseau ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-happens-if-an-endpoint-is-disabled-in-a-subnet-routing-type-profile)


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment développer des applications à haute disponibilité à l’aide de la [surveillance de points de terminaison Traffic Manager](traffic-manager-monitoring.md)




