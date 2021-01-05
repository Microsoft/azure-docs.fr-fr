---
title: Accélération de site dynamique via Azure CDN
description: CDN Azure prend en charge l’optimisation de l’accélération des sites dynamique (DSA) pour les fichiers avec un contenu dynamique.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/25/2019
ms.author: allensu
ms.openlocfilehash: 5387fdc224cd77ee5273767df5033a51dc27608c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008359"
---
# <a name="dynamic-site-acceleration-via-azure-cdn"></a>Accélération de site dynamique via Azure CDN

Avec l’explosion des médias sociaux, du e-commerce et du web hyper-personnalisé, un pourcentage rapidement croissant du contenu proposé aux utilisateurs finaux est généré en temps réel. Les utilisateurs attendent une expérience web rapide, fiable et personnalisée, indépendamment de leur navigateur, de leur emplacement géographique, de leur appareil ou du réseau. Toutefois, les innovations qui rendent ces expériences si engageantes ont également pour effet de ralentir les téléchargements de pages et risquent de compromettre la qualité de l’expérience des utilisateurs. 

La fonctionnalité de réseau de distribution de contenu (CDN) standard permet de rapprocher le cache de fichiers des utilisateurs finaux afin d’accélérer la distribution des fichiers statiques. Toutefois, avec des applications web dynamiques, la mise en cache de ce contenu dans des emplacements de périmètre n’est pas possible, car le serveur génère le contenu en réponse au comportement de l’utilisateur. L’accélération de la distribution d’un tel contenu est plus complexe que la mise en cache de périmètre traditionnelle, et nécessite une solution de bout en bout qui règle finement chaque élément avec le chemin d’accès aux données entier, du début jusqu’à la distribution. Grâce à l’optimisation de l’accélération de site dynamique (DSA) d’Azure CDN, le niveau de performance des pages web comprenant du contenu dynamique est sensiblement amélioré.

Les fonctionnalités **Azure CDN d’Akamai** et **Azure CDN de Verizon** offrent une fonction d’optimisation d’accélération de site dynamique au moyen du menu **Optimisé pour** durant la création du point de terminaison. L’accélération de site dynamique à partir de Microsoft est proposée par le biais d’[Azure Front Door Service](../frontdoor/front-door-overview.md).

> [!Important]
> Pour les profils **CDN Azure fournis par Akamai**, vous êtes autorisé à modifier l’optimisation d’un point de terminaison CDN après sa création.
>   
> Pour les profils **CDN Azure fournis par Verizon**, il n’est pas possible de modifier l’optimisation d’un point de terminaison CDN après sa création.

## <a name="cdn-endpoint-configuration-to-accelerate-delivery-of-dynamic-files"></a>Configuration d’un point de terminaison CDN pour accélérer la distribution de fichiers dynamiques

Pour configurer un point de terminaison CDN afin d’optimiser la remise de fichiers dynamiques, vous pouvez utiliser le portail Azure, les API REST ou l’un des Kits de développement logiciel (SDK) client pour faire la même chose par programmation. 

**Pour configurer un point de terminaison CDN pour l’optimisation DSA à l’aide du portail Azure :**

1. Dans la page **Profil CDN**, sélectionnez **Point de terminaison**.

   ![Ajouter un nouveau point de terminaison CDN](./media/cdn-dynamic-site-acceleration/cdn-endpoint-profile.png) 

   Le volet **Ajouter un point de terminaison** s’affiche.

2. Sous **Optimisé pour**, sélectionnez **Accélération de site dynamique**.

    ![Créer un point de terminaison CDN avec accélération de site dynamique](./media/cdn-dynamic-site-acceleration/cdn-endpoint-dsa.png)

3. Dans **Chemin d’analyse**, entrez un chemin d’accès à un fichier valide.

    Le chemin d’analyse est une fonctionnalité spécifique de DSA. Un chemin valide est requis pour la création. L’accélération de site dynamique utilise un petit fichier de *chemin d’analyse* placé sur le serveur d’origine pour optimiser les configurations de routage réseau pour le CDN. Comme chemin d’analyse, vous pouvez télécharger et charger l’exemple de fichier sur votre site, ou utiliser une ressource actuelle sur votre serveur d’origine d’environ 10 Ko.

4. Entrez les autres options de point de terminaison requises (pour plus d’informations, consultez [Créer un point de terminaison CDN](cdn-create-new-endpoint.md#create-a-new-cdn-endpoint)), puis sélectionnez **Ajouter**.

   Une fois créé, le point de terminaison CDN applique les optimisations d’accélération de site dynamique pour tous les fichiers correspondant à certains critères. 


**Pour configurer un point de terminaison existant pour DSA (profils Azure CDN d’Akamai uniquement) :**

1. Dans la page **Profil CDN**, sélectionnez le point de terminaison que vous souhaitez modifier.

2. Dans le volet gauche, sélectionnez **Optimisation**. 

   La page **Optimisation** s’affiche.

3. Sous **Optimisé pour**, sélectionnez **Accélération de site dynamique**, puis sélectionnez **Enregistrer**.

> [!Note]
> L’accélération de site dynamique entraîne des frais supplémentaires. Pour plus d’informations, consultez [Tarifs Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

## <a name="dsa-optimization-using-azure-cdn"></a>Optimisation d’accélération de site dynamique à l’aide de la fonctionnalité Azure CDN

L’accélération de site dynamique sur l’Azure CDN accélère la distribution de ressources dynamiques à l’aide des techniques suivantes :

-   [Optimisation du routage](#route-optimization)
-   [Optimisations de protocole TCP](#tcp-optimizations)
-   [Prérécupération d’objet (Azure CDN d’Akamai uniquement)](#object-prefetch-azure-cdn-from-akamai-only)
-   [Compression d’image adaptative (Azure CDN d’Akamai uniquement)](#adaptive-image-compression-azure-cdn-from-akamai-only)

### <a name="route-optimization"></a>Optimisation du routage

L’optimisation du routage est importante, car Internet est un espace dynamique où le trafic et les pannes temporaires modifient constamment la topologie du réseau. Le protocole de passerelle frontière (BGP) est le protocole de routage d’internet, mais il peut y avoir des routages plus rapides via des serveurs PoP (Point of Presence) intermédiaires. 

L’optimisation du routage choisit le chemin d’accès optimal à l’origine, afin qu’un site soit accessible en continu et que le contenu dynamique soit distribué aux utilisateurs finaux via le routage le plus rapide et le plus fiable possible. 

Le réseau Akamai utilise des techniques pour collecter des données en temps réel et comparer divers chemins d’accès via différents nœuds dans le serveur Akamai, ainsi que l’itinéraire BGP par défaut sur l’Internet ouvert afin de déterminer le routage le plus rapide entre l’origine et le périmètre CDN. Ces techniques évitent les points de surcharge d’Internet et les itinéraires longs. 

De même, le réseau Verizon combine DNS Anycast, PoP haute capacité et contrôles d’intégrité pour déterminer les meilleures passerelles afin d’acheminer au mieux les données depuis le client vers l’origine.
 
Ainsi, le contenu entièrement dynamique et transactionnel est distribué plus rapidement et de façon plus fiable aux utilisateurs finaux, même s’il ne peut pas être mis en cache. 

### <a name="tcp-optimizations"></a>Optimisations de protocole TCP

Le protocole TCP est le standard de la suite de protocoles Internet utilisé pour distribuer des informations entre des applications sur un réseau IP.  Par défaut, plusieurs demandes dans les deux sens sont requises pour configurer une connexion TCP, et il existe des limites pour éviter des surcharges du réseau entraînant des inefficacités à grande échelle. **Azure CDN d’Akamai** traite ce problème en optimisant de trois façons : 

 - [Élimination des démarrages lents TCP](#eliminating-tcp-slow-start)
 - [Exploitation des connexions persistantes](#leveraging-persistent-connections)
 - [Réglage des paramètres de paquet TCP](#tuning-tcp-packet-parameters)

#### <a name="eliminating-tcp-slow-start"></a>Élimination des démarrages lents TCP

Le *démarrage lent* TCP est un algorithme du protocole TCP qui empêche la surcharge du réseau en limitant la quantité de données envoyées sur le réseau. Il démarre avec des fenêtres de congestion de petite taille entre l’expéditeur et le récepteur, jusqu’à ce que la taille maximale soit atteinte ou une perte de paquet détectée.

 Les profils **CDN Azure fournis par Akamai** et **CDN Azure fournis par Verizon** éliminent tous les deux les démarrages TCP lents selon les trois étapes suivantes :

1. La surveillance de l’intégrité et de la bande passante est utilisée pour mesurer la bande passante des connexions entre les serveurs PoP de périphérie.
    
2. Des métriques sont partagées entre les serveurs PoP de périphérie afin que chaque serveur tienne compte des conditions de réseau et de l’intégrité des autres serveurs PoP alentour.  
    
3. Les serveurs de périphérie CDN bâtissent des hypothèses sur certains paramètres de transmission, tels que la taille de fenêtre optimale lors de la communication avec d’autres serveurs de périphérie CDN à proximité. Cette étape signifie que la taille de la fenêtre de congestion initiale peut être augmentée si l’intégrité de la connexion entre les serveurs de périphérie CDN permet des transferts de données par paquets de plus grande taille.  

#### <a name="leveraging-persistent-connections"></a>Exploitation des connexions persistantes

Avec un CDN, moins de machines se connectent directement à votre serveur d’origine par rapport aux utilisateurs qui se connectent directement à votre origine. Azure CDN regroupe également les demandes utilisateur afin de réduire le nombre de connexions établies avec l’origine.

Comme mentionné précédemment, plusieurs demandes d’établissement de liaison sont requises pour établir une connexion TCP. Les connexions persistantes, qui sont implémentées par l’en-tête `Keep-Alive` HTTP, réutilisent les connexions TCP existantes pour plusieurs requêtes HTTP afin de raccourcir les durées de boucles et d’accélérer la distribution. 

**Azure CDN de Verizon** envoie également des paquets de persistance périodiques via la connexion TCP pour empêcher la fermeture d’une connexion.

#### <a name="tuning-tcp-packet-parameters"></a>Réglage des paramètres de paquet TCP

**Azure CDN d’Akamai** règle les paramètres qui régissent les connexions de serveur à serveur, et réduit la quantité de boucles longues requises pour récupérer le contenu incorporé dans le site en utilisant les techniques suivantes :

- augmentation de la fenêtre de congestion initiale afin que davantage de paquets puissent être envoyés sans attendre d’accusé de réception ;
- réduction du délai de retransmission initial permettant la détection de perte et l’accélération de la retransmission ;
- réduction des délais de retransmission minimal et maximal pour réduire le temps d’attente avant de considérer que des paquets ont été perdus durant la transmission.

### <a name="object-prefetch-azure-cdn-from-akamai-only"></a>Prérécupération d’objet (Azure CDN d’Akamai uniquement)

La plupart des sites web sont constitués d’une page HTML, qui fait référence à diverses autres ressources telles que des images et des scripts. En général, quand un client demande une page web, le navigateur commence par télécharger et analyser l’objet HTML, puis envoie des demandes supplémentaires à des ressources liées requises pour le chargement complet de la page. 

La *prérécupération* est une technique permettant de récupérer des images et des scripts incorporés dans la page HTML pendant que le code HTML est envoyé au navigateur, et avant même que celui-ci demande ces objets. 

Avec l’option de prérécupération activée au moment où le CDN sert la page de base HTML au navigateur du client, le CDN analyse le fichier HTML, effectue des demandes supplémentaires de ressources liées et stocke celles-ci dans son cache. Quand le client effectue les demandes de ressources liées, le serveur de périphérie CDN dispose déjà des objets demandés, et peut servir ceux-ci immédiatement sans effectuer d’aller-retour avec l’origine. Cette optimisation est bénéfique pour le contenu tant mis en cache que non mis en cache.

### <a name="adaptive-image-compression-azure-cdn-from-akamai-only"></a>Compression d’image adaptative (Azure CDN d’Akamai uniquement)

Certains appareils, en particulier les appareils mobiles, sont parfois confrontés à des ralentissements de réseau. Dans ces scénarios, il est préférable pour l’utilisateur de recevoir plus rapidement des images de plus petite taille dans sa page web que d’attendre longtemps des images en pleine résolution.

Cette fonctionnalité surveille automatiquement la qualité du réseau et utilise des méthodes de compression JPEG standard lorsque la vitesse du réseau ralentit afin de raccourcir le temps de distribution.

Compression d’image adaptative | Extensions de fichier  
--- | ---  
Compression JPEG | .jpg, .jpeg, .jpe, .jig, .jgig, .jgi

## <a name="caching"></a>Mise en cache

Avec l’accélération de site dynamique, la mise en cache est désactivée par défaut sur le CDN, même quand l’origine inclut les en-têtes `Cache-Control` ou `Expires` dans la réponse. DSA est généralement utilisé pour les ressources dynamiques qui ne doivent pas être mises en cache, car elles sont uniques à chaque client. La mise en cache peut provoquer des problèmes ce comportement.

Si vous avez un site web combinant ressources statiques et ressources dynamiques, il est préférable d’adopter une approche hybride pour obtenir le meilleure niveau de performance. 

Pour les profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Standard fourni par Akamai**, vous pouvez activer la mise en cache des points de terminaison DSA spécifiques à l’aide des [règles de mise en cache](cdn-caching-rules.md).

Pour accéder à des règles de mise en cache :

1. Dans la page **Profil CDN**, sous Paramètres, sélectionnez **Règles de mise en cache**.  
    
    ![Bouton Règles de mise en cache CDN](./media/cdn-dynamic-site-acceleration/cdn-caching-rules-btn.png)

    La page **Règles de mise en cache** s’affiche.

2. Créez une règle de mise en cache globale ou personnalisée pour activer la mise en cache de votre point de terminaison DSA. 

Pour les profils **Azure CDN Premium fourni par Verizon** uniquement, vous pouvez activer la mise en cache des points de terminaison DSA spécifiques à l’aide du [moteur de règles](./cdn-verizon-premium-rules-engine.md). Les règles créées n’affectent que les points de terminaison de votre profil qui sont optimisés pour DSA. 

Pour accéder au moteur de règles :
    
1. Dans la page **Profil CDN**, sélectionnez **Gérer**.  
    
    ![Bouton de gestion du profil CDN](./media/cdn-dynamic-site-acceleration/cdn-manage-btn.png)

    Le portail de gestion CDN s'ouvre.

2. Dans le portail de gestion CDN, sélectionnez **ADN**, puis sélectionnez **Moteur de règles**. 

    ![Moteur de règles pour DSA](./media/cdn-dynamic-site-acceleration/cdn-dsa-rules-engine.png)



Vous pouvez également utiliser deux points de terminaison CDN : un point de terminaison optimisé avec DSA pour fournir des ressources dynamiques et un autre point de terminaison optimisé avec un type d’optimisation statique, tel qu’une livraison web générale, pour fournir les ressources pouvant être mises en cache. Modifiez les URL de vos pages web afin qu’elles pointent directement sur le point de terminaison CDN vers la ressource que vous envisagez d’utiliser. 

Par exemple : `mydynamic.azureedge.net/index.html` est une page dynamique et est chargée à partir du point de terminaison avec accélération de site dynamique.    La page html fait référence à plusieurs ressources statiques telles que des images ou des bibliothèques JavaScript qui sont chargées à partir du point de terminaison CDN statique, telles que `mystatic.azureedge.net/banner.jpg` et `mystatic.azureedge.net/scripts.js`.