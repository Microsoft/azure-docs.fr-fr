---
title: Diffuser du contenu avec l’intégration de CDN
titleSuffix: Azure Media Services
description: Apprenez-en davantage sur la diffusion de contenu avec l’intégration de CDN, ainsi que la prérécupération et Origin-Assist CDN-Prefetch.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 6bdf6015ca5633c77280111a55055a7394cee5bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001369"
---
# <a name="stream-content-with-cdn-integration"></a>Diffuser du contenu avec l’intégration de CDN

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Le réseau de diffusion de contenu (CDN) Azure offre aux développeurs une solution globale pour la distribution rapide de contenu haut débit aux utilisateurs en mettant en cache leur contenu sur des nœuds physiques disposés stratégiquement dans le monde entier.  

CDN met en cache le contenu diffusé en continu à partir d’un [point de terminaison de streaming (origine)](streaming-endpoint-concept.md) Media Services par codec, par protocole de diffusion en continu, par vitesse de transmission, par format de conteneur et par chiffrement/DRM. Un cache CDN distinct est associé à chaque combinaison de codec-protocole de diffusion en continu-format de conteneur-vitesse de transmission-chiffrement.

Le contenu qui est souvent demandé est fourni directement par le cache CDN tant que le fragment vidéo est mis en cache. Le contenu en direct est susceptible d’être mis en cache, car en général, beaucoup de gens regardent exactement la même chose. Le contenu à la demande peut être plus problématique, car vous pouvez avoir du contenu avec une demande forte et du contenu peu demandé. Si vous avez des millions de ressources vidéo avec peu de demandes de visualisation (seulement un ou deux visiteurs par semaine), mais que vous avez des milliers de personnes regardant chacune des vidéos différentes, le CDN devient beaucoup moins efficace.

Vous devez également prendre en compte le fonctionnement du streaming adaptif. Chaque fragment vidéo individuel est mis en cache comme entité propre. Par exemple, imaginez la première fois qu’une vidéo est visionnée. Si le viewer ne regarde que quelques secondes ici et là, seuls les fragments vidéo associés à ce que la personne a regardé sont mis en cache dans CDN. Avec le streaming adaptatif, vous avez en général 5 à 7 vitesses de transmission de vidéo différentes. Si une personne regarde à une certaine vitesse de transmission et qu’une autre personne regarde à une autre vitesse de transmission, le contenu qu’ils regardent chacun est mis en cache séparément dans le CDN. Même si deux personnes regardent du contenu à la même vitesse de transmission, leur streaming peut se faire via des protocoles différents. Chaque protocole (HLS, MPEG-DASH, Smooth Streaming) est mis en cache séparément. Ainsi, chaque vitesse de transmission et chaque protocole sont mis en cache séparément, et seuls les fragments vidéo qui ont été demandés sont mis en cache.

À l’exception de l’environnement de test, nous vous recommandons d’activer le CDN pour les points de terminaison de streaming Standard et Premium. Chaque type de point de terminaison de streaming a une limite de débit prise en charge différente.
Il est difficile d’effectuer un calcul précis quant au nombre maximal de flux simultanés pris en charge par un point de terminaison de streaming, car différents facteurs doivent être pris en compte. notamment :

- Vitesse de transmission maximale utilisée pour le streaming
- Comportement du lecteur avant la mise en mémoire tampon et comportement de commutation. Les lecteurs tentent de lire en rafale les segments en provenance d’une origine, et utilisent la vitesse de chargement pour calculer la commutation à débit adaptatif. Si un point de terminaison de streaming est proche de la saturation, les temps de réponse peuvent varier et les lecteurs commencent à basculer vers une qualité inférieure. Ceci réduisant la charge sur les lecteurs de point de terminaison de streaming, le retour à une qualité supérieure crée des déclencheurs de commutation indésirables.
Globalement, il est possible d’estimer de manière sécurisée le nombre maximal de flux simultanés en prenant le débit maximal de point de terminaison de streaming et en le divisant par la vitesse de transmission maximale (en supposant que tous les lecteurs utilisent la vitesse de transmission la plus élevée). Par exemple, vous pouvez avoir un point de terminaison de streaming Standard qui est limité à 600 Mbits/s et une vitesse de transmission la plus élevée de 3 Mbits/s. Dans ce cas, environ 200 flux simultanés sont pris en charge à la vitesse de transmission la plus élevée. N’oubliez pas de tenir compte des exigences de bande passante audio. Bien qu’un flux audio ne puisse être diffusé qu’à 128 Kbits/s, le streaming total augmente rapidement lorsque vous le multipliez par le nombre de flux simultanés.

Cette rubrique explique comment activer l’[intégration CDN](#enable-azure-cdn-integration). Elle explique également la prérécupération (mise en cache active) et le concept[Origin-Assist CDN-Prefetch](#origin-assist-cdn-prefetch).

## <a name="considerations"></a>Considérations

- Le [point de terminaison de streaming](streaming-endpoint-concept.md) `hostname` et l’URL de diffusion en continu restent identiques, que vous activiez CDN ou non.
- Si vous avez besoin de tester votre contenu avec ou sans CDN, créez un autre point de terminaison de streaming pour lequel CDN n’est pas activé.

## <a name="enable-azure-cdn-integration"></a>Activer l’intégration au CDN Azure

> [!IMPORTANT]
> Vous ne pouvez pas activer le CDN pour les comptes Azure d’évaluation ou d’étudiant.
>
> L’intégration du CDN est activée dans tous les centres de données Azure, sauf dans les régions Chine et du gouvernement fédéral.

Une fois le point de terminaison de streaming configuré et le CDN activé, il existe un délai d’attente défini sur Media Services avant la mise à jour des DNS pour le mappage du point de terminaison de streaming au point de terminaison CDN.

Si vous souhaitez activer/désactiver le CDN ultérieurement, votre point de terminaison de streaming doit avoir l’état **Arrêté**. Une fois le point de terminaison de streaming démarré, il peut s’écouler jusqu’à quatre heures avant que l’intégration d’Azure CDN soit activée et que les modifications soient actives sur tous les comptes POP CDN. Toutefois, vous pouvez démarrer votre point de terminaison de streaming et le flux sans interruptions depuis le point de terminaison de streaming. Une fois l’intégration terminée, le flux est remis à partir du CDN. Pendant la durée de l’approvisionnement, votre point de terminaison de streaming est en état de **démarrage** et vous pouvez observer une dégradation des performances.

Lorsque le point de terminaison de streaming Standard est créé, il est configuré par défaut avec Standard Verizon. Vous pouvez configurer les fournisseurs Premium Verizon ou Standard Akamai à l’aide des API REST.

L’intégration d’Azure Media Services au CDN Azure est implémentée sur le **CDN Azure fourni par Verizon** pour les points de terminaison de streaming Standard. Les points de terminaison de streaming Premium peuvent être configurés à l’aide de l’ensemble des **fournisseurs et niveaux de tarification Azure CDN**.

> [!NOTE]
> Pour plus d’informations sur Azure CDN, consultez [Présentation du CDN](../../cdn/cdn-overview.md).

## <a name="determine-if-a-dns-change-was-made"></a>Déterminer si une modification a été apportée au DNS

Vous pouvez déterminer si le DNS a été modifié sur un point de terminaison de streaming (le trafic est dirigé vers Azure CDN) à l’aide de <https://www.digwebinterface.com>. Si vous voyez des noms de domaine azureedge.net parmi les résultats, le trafic est maintenant dirigé vers le CDN.

## <a name="origin-assist-cdn-prefetch"></a>Origin-Assist CDN-Prefetch

La mise en cache CDN est un processus réactif. Si CDN peut prédire quel sera le prochain objet demandé, CDN peut le demander et le mettre en cache de manière proactive. Grâce à ce processus, vous pouvez obtenir un accès en cache pour l’ensemble (ou la plupart) des objets, améliorant ainsi les performances.

Le concept de prérécupération vise à positionner les objets à la « périphérie d’Internet » en prévision d’une demande imminente du lecteur, ce qui réduit le temps nécessaire pour remettre cet objet au lecteur.

Pour atteindre cet objectif, un point de terminaison de streaming (origine) et CDN doivent travailler main dans la main de plusieurs façons :

- l’origine Media Services doit avoir l’« intelligence » (Origin-Assist) d’informer CDN de l’objet suivant à prérécupérer.
- CDN effectue la prérécupération et la mise en cache (partie CDN-Prefetch). CDN doit également avoir l’« intelligence » d’indiquer à l’origine s’il s’agit d’une prérécupération ou d’une récupération classique, de traiter les réponses 404 et d’éviter une boucle de prérécupération infinie.

### <a name="benefits"></a>Avantages

Les avantages de la fonctionnalité *Origin-Assist CDN-Prefetch* incluent :

- La prérécupération améliore la qualité de la lecture vidéo en prépositionnant les segments vidéo anticipés à la périphérie pendant la lecture, en réduisant la latence pour le viewer et en améliorant les temps de téléchargement des segments vidéo. Cela permet d’accélérer le démarrage des vidéos et de réduire le nombre de mise en mémoire tampon.
- Ce concept s’applique au scénario CDN-origin général et n’est pas limité aux médias.
- Akamai a ajouté cette fonctionnalité à la solution [Cloud Embed d’Akamai (ACE)](https://learn.akamai.com/en-us/products/media_delivery/cloud_embed.html).

> [!NOTE]
> Cette fonctionnalité n’est pas encore applicable au CDN Akamai intégré au point de terminaison de streaming Media Services. Toutefois, elle est disponible pour les clients Media Services disposant d’un contrat Akamai préexistant et nécessitant une intégration personnalisée entre le CDN Akamai et l’origine Media Services.

### <a name="how-it-works"></a>Fonctionnement

La prise en charge de CDN pour les en-têtes `Origin-Assist CDN-Prefetch` (pour le streaming vidéo à la demande et en direct) est disponible pour les clients disposant d’un contrat direct avec le CDN Akamai. La fonctionnalité implique les échanges d’en-têtes HTTP suivants entre le CDN Akamai et l’origine Media Services :

|En-tête HTTP|Valeurs|Expéditeur|Destinataire|Objectif|
| ---- | ---- | ---- | ---- | ----- |
|`CDN-Origin-Assist-Prefetch-Enabled` | 1 (par défaut) ou 0 |CDN|Origine|Pour indiquer que la prérécupération est activée pour le CDN.|
|`CDN-Origin-Assist-Prefetch-Path`| Exemple : <br/>Fragments(video=1400000000,format=mpd-time-cmaf)|Origine|CDN|Pour fournir le chemin de prérécupération du CDN.|
|`CDN-Origin-Assist-Prefetch-Request`|1 (demande de prérécupération) ou 0 (demande normale)|CDN|Origine|Pour indiquer que la demande du CDN est une prérécupération.|

Pour voir une partie de l’échange d’en-têtes en action, vous pouvez essayer les étapes suivantes :

1. Utilisez Postman ou cURL pour émettre une requête destinée à l’origine Media Services pour un segment ou un fragment audio ou vidéo. Veillez à ajouter l’en-tête `CDN-Origin-Assist-Prefetch-Enabled: 1` dans la requête.
2. Dans la réponse, vous devez voir l’en-tête `CDN-Origin-Assist-Prefetch-Path` avec un chemin relatif comme valeur.

### <a name="supported-streaming-protocols"></a>Protocoles de diffusion en continu pris en charge

La fonctionnalité `Origin-Assist CDN-Prefetch` prend en charge les protocoles de diffusion en continu suivants pour le streaming en direct et à la demande :

* HLS v3
* HLS v4
* HLS CMAF
* DASH (CSF)
* DASH (CMAF)
* Diffusion en continu lisse

### <a name="faqs"></a>FAQ

* Que se passe-t-il si l’URL d’un chemin de prérécupération n’est pas valide et que la prérécupération CDN obtient une réponse 404 ?

    CDN met en cache une réponse 404 uniquement pendant 10 secondes (ou une autre valeur configurée).

* Supposons que vous disposiez d’une vidéo à la demande. Si la fonctionnalité CDN-prefetch est activée, cette fonctionnalité implique-t-elle qu’une fois qu’un client demande le premier segment vidéo, la prérécupération démarre une boucle pour prérécupérer tous les segments vidéo suivants à la même vitesse de transmission ?

    Non, CDN-prefetch n’est effectuée qu’après une demande/réponse initiée par le client. CDN-prefetch n’est jamais déclenchée par une prérécupération, afin d’éviter une boucle de prérécupération.

* La fonctionnalité Origin-Assist CDN-Prefetch est-elle toujours activée ? Comment l’activer ou la désactiver ?

    Cette fonctionnalité est désactivée par défaut. Les clients doivent l’activer via l’API Akamai.

* Pour le streaming en direct, que se passe-t-il pour Origin-Assist si le segment ou le fragment suivant n’est pas encore disponible ?

    Dans ce cas, l’origine Media Services ne fournira pas d’en-tête `CDN-Origin-Assist-Prefetch-Path` et CDN-prefetch n’aura pas lieu.

* Comment `Origin-Assist CDN-Prefetch` fonctionne-t-il avec les filtres de manifeste dynamiques ?

    Cette fonctionnalité fonctionne indépendamment du filtre de manifeste. Lorsque le fragment suivant est en dehors d’une fenêtre de filtre, son URL est toujours localisée en regardant dans le manifeste brut du client, puis retournée comme en-tête de réponse de prérécupération CDN. CDN obtiendra donc l’URL d’un fragment filtré à partir du manifeste DASH/HLS/Smooth. Toutefois, le lecteur n’effectue jamais de requête GET auprès de CDN pour récupérer ce fragment, car ce dernier n’est pas inclus dans le manifeste DASH/HLS/Smooth détenu par le lecteur (le lecteur ne connaît pas l’existence de ce fragment).

* Le manifeste DASH MPD/Liste de lecture HLS/Smooth peut-il être prérécupéré ?

    Non, l’URL du manifeste DASH MPD, Liste de lecture principale HLS, Variante de liste de lecture HLS ou Smooth n’est pas ajoutée à l’en-tête de prérécupération.

* Les URL de prérécupération sont-elles relatives ou absolues ?

    Le CDN Akamai autorise les deux, mais l’origine Media Services fournit uniquement des URL relatives pour le chemin de prérécupération, car il n’existe aucun avantage apparent à utiliser des URL absolues.

* Cette fonctionnalité fonctionne-t-elle avec des contenus protégés par DRM ?

    Oui, puisque cette fonctionnalité fonctionne au niveau HTTP, elle ne décode ni n’analyse aucun segment/fragment. Elle ne se soucie pas de savoir si le contenu est chiffré ou non.

* Cette fonctionnalité fonctionne-t-elle avec l’insertion de publicités côté serveur (SSAI) ?
    
    Elle fonctionne pour le contenu d’origine/principal (le contenu vidéo d’origine avant l’insertion de publicités), car SSAI ne modifie pas l’horodateur du contenu source provenant de l’origine Media Services. Le fonctionnement de cette fonctionnalité avec des publicités dépend de la prise en charge ou non d’Origin-Assist par l’origine des publicités. Par exemple, si les publicités sont également hébergées dans Azure Media Services (même origine ou origine distincte), le contenu des publicités sera également prérécupéré.

* Cette fonctionnalité fonctionne-t-elle avec un contenu UHD/HEVC ?

    Oui.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

* Veillez à consulter le document [Point de terminaison de streaming (origine)](streaming-endpoint-concept.md).
* L’exemple [de ce dépôt](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs) montre comment démarrer le point de terminaison de streaming par défaut avec .NET.
