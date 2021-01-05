---
title: Concepts clés et présentation de la gestion des API Azure | Microsoft Docs
description: Learn about APIs, products, roles, groups, and other API Management key concepts.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: overview
ms.date: 11/15/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: 85fa79cdfc7036be5b0ab20e49986a1d075152c5
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "86254654"
---
# <a name="about-api-management"></a>En savoir plus sur la Gestion des API

La Gestion des API (APIM) est une méthode permettant de créer des passerelles API cohérentes et modernes pour des services back-end existants.

Gestion des API aide les organisations à publier des API pour des développeurs externes, partenaires et internes, afin de libérer le potentiel de leurs données et services. Toutes les entreprises cherchent à étendre leurs opérations en tant que plateforme numérique, en créant de nouveaux canaux, en trouvant de nouveaux clients et en suscitant un engagement supérieur auprès de ceux existants. Il offre les compétences essentielles qui garantissent un programme d’API réussi au travers de l’engagement des développeurs, des perspectives commerciales, de l’analytique, de la sécurité et de la protection. Vous pouvez utiliser la gestion des API Azure pour prendre n’importe quel serveur principal et lancer un programme d’API à part entière qui repose sur ce dernier.

Cet article fournit une vue d’ensemble des scénarios courants qui impliquent APIM.  Elle donne également une vue d’ensemble des principaux composants du système APIM. L’article fournit ensuite une présentation plus détaillée de chaque composant.

## <a name="overview"></a>Vue d’ensemble

Pour utiliser Gestion des API, les administrateurs créent des API. Chaque API se compose d'une ou plusieurs opérations et chacune peut être ajoutée à un produit ou plusieurs. Pour utiliser une API, les développeurs s'abonnent à un produit qui contient cette API, puis ils peuvent appeler l'opération de l'API, sujette à toutes les stratégies d'utilisation pouvant être en vigueur. Scénarios courants :

* **Sécurisation d’infrastructures mobiles** par régulation de l’accès à l’aide de clés d’API pour éviter les attaques par déni de service (DOS) en utilisant la limitation ou des stratégies de sécurité avancées, telles que la validation de jeton JWT.
* **Activation d’écosystèmes de partenaires ISV** en proposant une intégration rapide des partenaires via le portail des développeurs et la création d’une façade d’API afin de se dissocier des implémentations internes qui ne sont pas appropriées à la consommation des partenaires.
* **Exécution d’un programme d’API interne** en offrant un emplacement centralisé à l’organisation pour communiquer sur la disponibilité et les dernières modifications apportées aux API, tout en régulant l’accès basé sur des comptes professionnels, tous basés sur un canal sécurisé entre la passerelle de l’API et le serveur principal.

Le système est constitué des composants suivants :

* La **passerelle d’API** est le point de terminaison qui :
  
  * accepte les appels d’API et les dirige vers vos serveurs principaux ;
  * vérifie les clés d’API, les jetons JWT, les certificats et les autres informations d’identification ;
  * applique des quotas d’utilisation et des limites de débit ;
  * transforme votre API à la volée sans modification de code ;
  * met en cache les réponses du serveur principal lorsqu’il est configuré ;
  * enregistre les métadonnées relatives aux appels à des fins d’analyse.
* Le **portail Azure** est l’interface d’administration où vous configurez votre programme d’API. Utilisez-le pour :
  
  * définir ou importer le schéma d’API ;
  * intégrer des API aux produits sous forme de packages ;
  * définir des stratégies, telles que des quotas ou des transformations sur les API ;
  * obtenir des informations issues de l’analyse ;
  * gérer les utilisateurs.
* Le **portail des développeurs** est le principal lieu sur le web où les développeurs peuvent :
  
  * lire la documentation de l’API ;
  * essayer une API via la console interactive ;
  * créer un compte et s’abonner pour obtenir les clés d’une API ;
  * accéder aux analyses relatives à leur propre utilisation.

Pour plus d’informations, consultez le livre blanc au format PDF [Cloud-based API Management: Harnessing the Power of APIs](https://j.mp/ms-apim-whitepaper). Ce livre blanc d’introduction à la gestion des API, publié par CITO Research, présente les opérations suivantes : 
 
 * Configuration requise des API et défis communs
 * Découplage des API et présentation des façades
 * Motivation des développeurs et exécution rapide
 * Sécurisation de l'accès
 * Analyse et mesures
 * Prise de contrôle et aperçu d’une plate-forme de gestion des API
 * Utilisation du cloud par rapport aux solutions locales
 * Gestion des API Azure
 
## <a name="apis-and-operations"></a><a name="apis"> </a>API et opérations
Les API sont la base d'une instance du service Gestion des API. Chaque API représente un ensemble d'opérations disponibles pour les développeurs. Chaque API contient une référence au service principal qui implémente l'API, et ses opérations effectuent un mappage vers les opérations implémentées par le service principal. Les opérations dans Gestion des API sont hautement configurables. Elles contrôlent le mappage d'URL, les paramètres de requête et de chemin d'accès, le contenu de la demande et de la réponse, et la mise en cache de la réponse de l'opération. La limite de débit, les quotas et les stratégies de restriction de l'adresse IP peuvent également être implémentés au niveau de l'API ou du fonctionnement individuel.

Pour plus d'informations, consultez les pages [Création d'API][How to create APIs] et [Ajout d'opérations à une API][How to add operations to an API].

## <a name="products"></a><a name="products"> </a> Produits
Les produits sont la façon dont les API sont présentées en surface aux développeurs. Les produits dans Gestion des API possèdent une ou plusieurs API et sont configurés avec un titre, une description et des conditions d'utilisation. Les produits peuvent être **ouverts** ou **protégés**. Les produits protégés doivent faire l’objet d’un abonnement avant de pouvoir être utilisés, alors que les produits ouverts peuvent être utilisés sans abonnement. Lorsqu'un produit est prêt à l'emploi pour les développeurs, il peut être publié. Une fois le produit publié, les développeurs peuvent l’afficher et s’y abonner (dans le cas des produits protégés). L'approbation d'abonnement est configurée au niveau du produit et peut nécessiter l'approbation de l'administrateur ou être automatiquement approuvée.

Les groupes permettent de gérer la visibilité des produits pour les développeurs. Les produits accordent de la visibilité aux groupes. Les développeurs peuvent afficher les produits visibles pour les groupes auxquels ils appartiennent et s'y abonner. 

## <a name="groups"></a><a name="groups"> </a> Groupes
Les groupes permettent de gérer la visibilité des produits pour les développeurs. Le service Gestion des API possède les groupes système suivants, qui ne sont pas modifiables :

* **Administrateurs** : les administrateurs d’abonnements Azure sont membres de ce groupe. Les administrateurs gèrent les instances du service Gestion des API, créant les API, opérations et produits qui sont utilisés par les développeurs.
* **Développeurs** : les utilisateurs authentifiés du portail des développeurs appartiennent à ce groupe. Les développeurs sont les clients qui génèrent des applications grâce à vos API. Les développeurs bénéficient d'un accès au portail des développeurs et génèrent des applications qui appellent les opérations d'une API.
* **Invités** : les utilisateurs non authentifiés du portail des développeurs, comme les prospects, qui consultent le portail des développeurs d’une instance d’API Management appartiennent à ce groupe. Ils peuvent recevoir certains accès en lecture seule, comme la possibilité d'afficher les API, mais pas de les appeler.

Outre ces groupes système, les administrateurs peuvent créer des groupes personnalisés ou [utiliser des groupes externes dans des locataires Azure Active Directory qui leur sont associés](api-management-howto-aad.md). Des groupes externes et personnalisés peuvent être utilisés avec des groupes système offrant une certaine visibilité aux développeurs et un accès aux produits d’API. Vous pourriez, par exemple, créer un groupe personnalisé pour les développeurs affiliés à une organisation partenaire spécifique et leur permettre d’accéder aux API à partir d’un produit contenant uniquement des API pertinentes. Un utilisateur peut être membre de plusieurs groupes.

Pour plus d'informations, consultez la page [Création et utilisation de groupes][How to create and use groups].

## <a name="developers"></a><a name="developers"> </a> Développeurs
Les développeurs représentent les comptes d'utilisateur dans une instance du service Gestion des API. Ils peuvent être créés ou invités à rejoindre le groupe par les administrateurs, ou ils peuvent s'inscrire au [Portail des développeurs][Developer portal]. Chaque développeur est membre d'un ou plusieurs groupes, et peut s'abonner aux produits qui accordent de la visibilité à ces groupes.

Lorsque des développeurs s’abonnent à un produit, ils reçoivent les clés principale et secondaire pour le produit. Cette clé est utilisée pour les appels dans les API du produit.

Pour plus d'informations, consultez les pages [Création ou invitation de développeurs][How to create or invite developers] et [Association de groupes à des développeurs][How to associate groups with developers].

## <a name="policies"></a><a name="policies"> </a> Stratégies
Les stratégies sont une fonctionnalité puissante de Gestion des API. Elles permettent au portail Azure de modifier le comportement de l’API grâce à la configuration. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Les instructions les plus utilisées comprennent la conversion du format XML au format JSON et la limitation du débit d'appels pour restreindre le nombre d'appels entrants d'un développeur. De nombreuses autres stratégies sont disponibles.

Les expressions de stratégie peuvent être utilisées comme valeurs d’attribut ou valeurs de texte dans l’une des stratégies de Gestion des API, sauf si la stratégie le spécifie autrement. Certaines stratégies, telles que les stratégies [Control flow](./api-management-advanced-policies.md#choose) et [Set variable](./api-management-advanced-policies.md#set-variable), sont basées sur des expressions de stratégie. Pour plus d’informations, consultez les rubriques [Stratégies avancées](./api-management-advanced-policies.md#AdvancedPolicies) et [Expressions de stratégie](./api-management-policy-expressions.md).


Pour obtenir la liste complète des stratégies Gestion des API, consultez la page [Référence de stratégie][Policy reference]. Pour plus d'informations sur l'utilisation et la configuration des stratégies, consultez la page [Stratégies Gestion des API][API Management policies]. Pour consulter un didacticiel sur la création d'un produit avec des stratégies de limite de débit et de quota, consultez la page [Création et configuration de paramètres produit avancés][How create and configure advanced product settings].


## <a name="developer-portal"></a><a name="developer-portal"> </a> Portail des développeurs
Le portail des développeurs est l'endroit où les développeurs peuvent en savoir plus sur votre API, afficher et appeler des opérations et s'abonner à des produits. Les prospects peuvent consulter le portail des développeurs, afficher les API et opérations et se connecter. L’URL du portail des développeurs se trouve sur le tableau de bord du portail Azure pour votre instance de service Gestion des API.

Vous pouvez personnaliser l'aspect de votre portail des développeurs en ajoutant du contenu personnalisé, en personnalisant des styles et en ajoutant votre marque.

## <a name="api-management-and-the-api-economy"></a>Gestion et économie des API

Pour en savoir plus sur la gestion des API, regardez la présentation suivante issue de la conférence Microsoft Ignite 2017.

> [!VIDEO https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK2186/player]
> 
> 

## <a name="next-steps"></a>Étapes suivantes

Terminez le démarrage rapide suivant et commencez à utiliser la Gestion des API Azure :

> [!div class="nextstepaction"]
> [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)

[APIs and operations]: #apis
[Products]: #products
[Groups]: #groups
[Developers]: #developers
[Policies]: #policies
[Developer portal]: #developer-portal

[How to create APIs]: ./import-and-publish.md
[How to add operations to an API]: ./mock-api-responses.md
[How to create and publish a product]: api-management-howto-add-products.md
[How to create and use groups]: api-management-howto-create-groups.md
[How to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer
[How create and configure advanced product settings]: transform-api.md
[How to create or invite developers]: api-management-howto-create-or-invite-developers.md
[Policy reference]: ./api-management-policies.md
[API Management policies]: api-management-howto-policies.md
[Create an API Management service instance]: get-started-create-service-instance.md
