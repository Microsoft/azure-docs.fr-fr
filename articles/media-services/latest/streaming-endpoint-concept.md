---
title: Points de terminaison de streaming (origine)
titleSuffix: Azure Media Services
description: Découvrez les points de terminaison de streaming (origine), un service dynamique d’empaquetage et de diffusion en continu fournissant du contenu directement à une application de lecteur cliente ou à un réseau de distribution de contenu (CDN).
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: inhenkel
ms.openlocfilehash: 9f17e8a09715ce9ff51715f17a449ec0a5b3f770
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89297193"
---
# <a name="streaming-endpoints-origin-in-azure-media-services"></a>Points de terminaison de streaming (origine) dans Azure Media Services

Dans Microsoft Azure Media Services, un [point de terminaison de streaming](/rest/api/media/streamingendpoints) représente un empaquetage dynamique (juste-à-temps) et un service d’origine qui permet de transmettre votre contenu en direct et à la demande directement à une application de lecteur cliente, à l’aide de l’un des protocoles de streaming multimédia courants (HLS ou DASH). En outre, le **point de terminaison de streaming** offre un chiffrement dynamique (juste-à-temps) aux systèmes DRM de pointe. 

Quand vous créez un compte Media Services, un point de terminaison de streaming **par défaut** est créé pour vous dans l’état Arrêté. Vous ne pouvez pas supprimer le point de terminaison de streaming **par défaut**. Vous pouvez créer d’autres points de terminaison de streaming sous votre compte (voir [Quotas et limites](limits-quotas-constraints.md)).

> [!NOTE]
> Pour démarrer le streaming de vidéos, vous devez démarrer le **point de terminaison de streaming** à partir duquel vous souhaitez diffuser la vidéo.
>
> Vous êtes facturé uniquement lorsque votre point de terminaison de streaming est en cours d’exécution.

Veillez à consulter également la rubrique [Empaquetage dynamique](dynamic-packaging-overview.md). 

## <a name="naming-convention"></a>Conventions d’affectation de noms

Le format du nom d’hôte de l’URL de streaming est le suivant : `{servicename}-{accountname}-{regionname}.streaming.media.azure.net`, où `servicename` est le nom du point de terminaison de streaming ou le nom de l’événement en direct.

Quand vous utilisez le point de terminaison de streaming par défaut, `servicename` est omis ; l’URL est alors : `{accountname}-{regionname}.streaming.azure.net`.

### <a name="limitations"></a>Limites

* Le nom du point de terminaison de streaming peut contenir 24 caractères au maximum.
* Le nom doit suivre ce modèle [regex](/dotnet/standard/base-types/regular-expression-language-quick-reference) : `^[a-zA-Z0-9]+(-*[a-zA-Z0-9])*$`.

## <a name="types"></a>Types

Il existe deux types de **points de terminaison de streaming** : **Standard** (préversion) et **Premium**. Le type est défini par le nombre d’unités d’échelle (`scaleUnits`) que vous allouez pour le point de terminaison de streaming.

La limite d’unités de streaming est généralement de 10. Contactez-nous [ici](https://azure.microsoft.com/support/create-ticket/) pour augmenter la limite de votre compte.

Le tableau décrit les types :

|Type|Unités d’échelle|Description|
|--------|--------|--------|  
|**Standard**|0|Le point de terminaison de streaming par défaut est de type **Standard**, mais vous pouvez le passer au type Premium en ajustant `scaleUnits`.|
|**Premium**|>0|Les points de terminaison de streaming **Premium** sont conçus pour les charges de travail avancées et fournissent une capacité de bande passante dédiée et évolutive. Vous pouvez passer à un type **Premium** en ajustant `scaleUnits` (unités de streaming). Les `scaleUnits` vous offrent une capacité de sortie dédiée qui peut être achetée par incréments de 200 Mbits/s. Quand vous utilisez le type **Premium**, chaque unité activée fournit une capacité de bande passante supplémentaire à l’application. |

> [!NOTE]
> Pour les clients qui souhaitent distribuer du contenu à un large public sur internet, il est conseillé d’activer le CDN sur le point de terminaison de streaming.

Pour plus d’informations sur le contrat SLA, consultez [Tarifs et contrat SLA](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="comparing-streaming-types"></a>Comparaison des types de streaming

Fonctionnalité|Standard|Premium
---|---|---
Débit |Jusqu’à 600 Mbits/s, et le débit fourni peut être beaucoup plus élevé avec un CDN.|200 Mbits/s par unité de streaming (SU). Le débit fourni peut être beaucoup plus élevé avec un CDN.
CDN|Azure CDN, CDN tiers ou sans CDN.|Azure CDN, CDN tiers ou sans CDN.
La facturation est calculée sur la base d'un taux| Quotidien|Quotidien
Chiffrement dynamique|Oui|Oui
l’empaquetage dynamique|Oui|Oui
Scale|Mise à l’échelle automatique vers le débit cible.|Unités de streaming supplémentaires
Hôte de filtrage d’IP/G20/personnalisé <sup>1</sup>|Oui|Oui
Téléchargement progressif|Oui|Oui
Utilisation recommandée |Recommandé pour la plupart des scénarios de streaming.|Utilisation professionnelle.

<sup>1</sup> Uniquement utilisé directement sur le point de terminaison de streaming quand le CDN n’est pas activé sur le point de terminaison.<br/>

## <a name="streaming-endpoint-properties"></a>Propriétés des points de terminaison de streaming

Cette section fournit des détails sur certaines propriétés du point de terminaison de streaming. Pour obtenir des exemples montrant comment créer un nouveau point de terminaison de streaming et les descriptions de toutes les propriétés, consultez [Point de terminaison de streaming](/rest/api/media/streamingendpoints/create).

- `accessControl`: Permet de configurer les paramètres de sécurité suivants pour ce point de terminaison de diffusion en continu : Les clés d’authentification d’en-tête de signature Akamai et les adresses IP qui sont autorisées à se connecter à ce point de terminaison. Cette propriété peut uniquement être définie quand `cdnEnabled` est défini sur false.

- `cdnEnabled`: Indique si l’intégration d’Azure CDN pour ce point de terminaison de streaming est activée (elle est désactivée par défaut). Si vous définissez `cdnEnabled` sur true, les configurations suivantes sont désactivées : `customHostNames` et `accessControl`.

    Certains centres de données ne prennent pas en charge l’intégration d’Azure CDN. Pour vérifier si l’intégration d’Azure CDN est disponible pour votre centre de données, procédez comme suit :

  - Essayez de définir `cdnEnabled` sur true.
  - Vérifiez le résultat retourné pour `HTTP Error Code 412` (PreconditionFailed) avec le message « La propriété CdnEnabled du point de terminaison de streaming ne peut pas être définie sur true, car la fonctionnalité CDN n’est pas disponible dans la région actuelle ».

    Si vous recevez cette erreur, cela signifie que le centre de données ne la prend pas en charge. Essayez un autre centre de données.

- `cdnProfile`: Quand `cdnEnabled` est définie sur true, vous pouvez également passer des valeurs de `cdnProfile` valeurs. `cdnProfile` est le nom du profil CDN où le point du point de terminaison CDN doit être créé. Vous pouvez fournir un cdnProfile existant ou en utiliser un nouveau. Si la valeur est NULL et que `cdnEnabled` est true, la valeur par défaut « AzureMediaStreamingPlatformCdnProfile » est utilisée. Si le `cdnProfile` fourni existe déjà, un point de terminaison est créé sous celui-ci. Si le profil n’existe pas, un nouveau profil est créé automatiquement.
- `cdnProvider`: Quand le CDN est activé, vous pouvez également passer des valeurs de `cdnProvider`. `cdnProvider` contrôle les fournisseurs à utiliser. Actuellement, trois valeurs sont prises en charge : « StandardVerizon », « PremiumVerizon » et « StandardAkamai ». Si aucune valeur n’est fournie et que `cdnEnabled` a la valeur true, « StandardVerizon » est utilisé (c’est la valeur par défaut).
- `crossSiteAccessPolicies`: Utilisée pour spécifier des stratégies d’accès entre sites pour différents clients. Pour plus d’informations, consultez [Cross-domain policy file specification](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html) et [Rendre un service disponible entre des limites de domaine](/previous-versions/azure/azure-services/gg185950(v=azure.100)). Les paramètres s’appliquent uniquement au format Smooth Streaming.
- `customHostNames`: Utilisée pour configurer un point de terminaison de streaming pour qu’il accepte le trafic dirigé vers un nom d’hôte personnalisé. Cette propriété est valide pour les points de terminaison de streaming Standard et Premium, et elle peut être définie quand `cdnEnabled` est définie sur false.

    La propriété du nom de domaine doit être vérifiée par Media Services. Media Services vérifie la propriété du nom de domaine en demandant un enregistrement `CName` qui contient l’ID de compte Media Services comme composant à ajouter au domaine utilisé. Par exemple, pour utiliser « sports.contoso.com » comme nom d’hôte personnalisé pour le point de terminaison de streaming, un enregistrement pour `<accountId>.contoso.com` doit être configuré de façon à pointer vers un des noms d’hôte de vérification de Media Services. Le nom d’hôte de vérification est composé de verifydns.\<mediaservices-dns-zone>.

    Voici les zones DNS attendues dans l’enregistrement de vérification pour les différentes régions Azure.
  
  - Amérique du Nord, Europe, Singapour, Hong Kong (R.A.S.), Japon :

    - `media.azure.net`
    - `verifydns.media.azure.net`

  - Chine :

    - `mediaservices.chinacloudapi.cn`
    - `verifydns.mediaservices.chinacloudapi.cn`

    Par exemple, un enregistrement `CName` qui mappe « 945a4c4e-28ea-45cd-8ccb-a519f6b700ad.contoso.com » à « verifydns.media.azure.net » prouve que l’ID Media Services 945a4c4e-28ea-45cd-8ccb-a519f6b700ad est propriétaire du domaine contoso.com, permettant ainsi l’utilisation de n’importe quel nom sous contoso.com comme nom d’hôte personnalisé pour un point de terminaison de streaming sous ce compte. Pour rechercher la valeur de l’ID du Service multimédia, accédez au [portail Azure](https://portal.azure.com/) et sélectionnez votre compte Service multimédia. L’**ID de compte** s’affiche dans le coin supérieur droit de la page.

    Si vous tentez de définir un nom d’hôte personnalisé sans vérification appropriée de l’enregistrement `CName`, la réponse DNS échoue, puis elle est mise en cache pendant un certain temps. Une fois qu’un enregistrement approprié est en place, un certain temps peut être nécessaire pour que la réponse mise en cache soit revalidée. Selon le fournisseur DNS pour le domaine personnalisé, quelques minutes à une heure sont nécessaires pour revalider l’enregistrement.

    En plus du `CName` qui mappe `<accountId>.<parent domain>` à `verifydns.<mediaservices-dns-zone>`, vous devez créer un autre `CName` qui mappe le nom d’hôte personnalisé (par exemple `sports.contoso.com`) au nom d’hôte de votre point de terminaison de streaming Media Services (par exemple `amstest-usea.streaming.media.azure.net`).

    > [!NOTE]
    > Des points de terminaison de streaming qui se trouvent dans le même centre de données ne peuvent pas partager le même nom d’hôte personnalisé.

    Actuellement, Media Services ne prend pas en charge TLS avec des domaines personnalisés.

- `maxCacheAge` : remplace l’en-tête de contrôle du cache HTTP max-age par défaut défini par le point de terminaison de streaming sur des fragments de médias et des manifestes à la demande. La valeur est définie en secondes.
- `resourceState` -

    - Arrêté : état initial d’un point de terminaison de streaming après sa création.
    - Démarrage : le point de terminaison de streaming est en transition vers l’état « Exécution en cours ».
    - Exécution en cours : le point de terminaison de streaming peut diffuser du contenu aux clients.
    - Mise à l’échelle : les unités d’échelle sont augmentées ou diminuées.
    - Arrêt : le point de terminaison de streaming est en transition vers l’état « Arrêté ».
    - Suppression : votre travail est en cours de suppression.

- `scaleUnits`: Offrent une capacité de sortie dédiée qui peut être achetée par incréments de 200 Mbits/s. Si vous devez passer à un type **Premium**, ajustez `scaleUnits`.

## <a name="why-use-multiple-streaming-endpoints"></a>Pourquoi utiliser plusieurs points de terminaison de streaming ?

Un seul point de terminaison de streaming permet le streaming de vidéos en direct et à la demande. La plupart des clients utilisent un seul point de terminaison de streaming. Cette section donne quelques exemples des raisons pour lesquelles vous pouvez être amené à utiliser plusieurs points de terminaison de streaming.

* Chaque unité réservée permet l’utilisation de 200 Mbits/s de bande passante. Si vous avez besoin de plus de 2 000 Mbits/s (2 Gbits/s) de bande passante, vous pouvez utiliser le deuxième point de terminaison de streaming et l’équilibrage de charge pour disposer d’une bande passante supplémentaire.

    Notez que le CDN est le meilleur moyen d’effectuer un scale-out pour du streaming de contenu. Toutefois, si vous fournissez du contenu en très grande quantité, à tel point que le CDN doit extraire plus de 2 Gbits/s, vous pouvez ajouter des points de terminaison de streaming supplémentaires (origines). Dans ce cas, vous devez distribuer des URL de contenu équilibrées entre les deux points de terminaison de streaming. Cette approche offre une meilleure mise en cache que la tentative d’envoi de requêtes à chaque origine de manière aléatoire (par exemple, via un gestionnaire de trafic). 
    
    > [!TIP]
    > En règle générale, si le CDN extrait plus de 2 Gbits/s, cela est peut-être dû à un problème de configuration (par exemple l’absence de protection de l’origine).
    
* Équilibrage de charge de différents fournisseurs CDN. Par exemple, vous configurez le point de terminaison de streaming par défaut pour utiliser le CDN Verizon, et vous en créez un second pour utiliser Akamai. Vous ajoutez ensuite un équilibrage de charge entre les deux pour obtenir l’équilibrage de plusieurs CDN. 

    Toutefois, il arrive souvent que les clients effectuent un équilibrage de charge entre plusieurs fournisseurs CDN via une seule origine.
* Streaming de contenu mixte : Vidéo en direct et à la demande. 

    Les modèles d’accès au contenu en direct et à la demande sont très différents. Le contenu en direct a tendance à faire l’objet d’une forte demande de manière simultanée. Le contenu vidéo à la demande (contenu d’archive « à longue traîne » par exemple) est peu utilisé pour un même contenu. Ainsi, la mise en cache fonctionne très bien pour du contenu en direct, mais pas aussi bien pour du contenu à longue traîne.

    Imaginez le scénario suivant : vos clients regardent principalement du contenu en direct et parfois du contenu à la demande. Le contenu est fourni à partir du même point de terminaison de streaming. La faible utilisation du contenu à la demande entraîne une occupation du cache qui serait mieux utilisée pour du contenu en direct. Dans ce scénario, nous vous recommandons de fournir le contenu en direct à partir d’un point de terminaison de streaming, et le contenu à longue traîne à partir d’un autre point de terminaison de streaming. Cela permet d’améliorer le niveau de performance pour le streaming de contenu d’événement en direct.
    
## <a name="scaling-streaming-with-cdn"></a>Mise à l’échelle du streaming avec CDN

Voir les articles suivants :

- [Présentation du CDN](../../cdn/cdn-overview.md)
- [Mise à l’échelle du streaming avec un CDN](scale-streaming-cdn.md)

## <a name="ask-questions-and--get-updates"></a>Poser des questions et obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="see-also"></a>Voir aussi

[mise en package dynamique](dynamic-packaging-overview.md)

## <a name="next-steps"></a>Étapes suivantes

[Gérer les points de terminaison de streaming](manage-streaming-endpoints-howto.md)
