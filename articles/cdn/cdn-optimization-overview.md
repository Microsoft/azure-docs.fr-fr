---
title: Optimiser Azure CDN pour le type de distribution de contenu
description: Découvrez comment Azure Content Delivery Network peut optimiser la distribution en fonction du type de contenu. Les meilleures pratiques d’optimisation améliorent les performances et le déchargement de l’origin.
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
ms.openlocfilehash: 7c7e4ff05307452da67c37b23bf492db8855e2ef
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004498"
---
# <a name="optimize-azure-cdn-for-the-type-of-content-delivery"></a>Optimiser Azure CDN pour le type de distribution de contenu

En cas de distribution de contenu à une large audience internationale, il est essentiel de veiller à optimiser la distribution. [Azure Content Delivery Network (CDN)](cdn-overview.md) peut optimiser l’expérience de distribution en fonction du type de contenu. Le contenu peut être un site web, un flux temps réel, une vidéo ou un fichier volumineux pour téléchargement. Quand vous créez un point de terminaison CDN, vous spécifiez un scénario dans l’option **Optimisé pour**. Votre choix détermine l’optimisation appliquée au contenu distribué à partir du point de terminaison CDN.

Les choix d’optimisation sont conçus pour utiliser des comportements recommandés afin d’améliorer les performances de distribution de contenu et le déchargement de l’origine. Vos choix de scénario affectent les performances en modifiant les configurations de mise en cache partielle, la segmentation d’objet et la stratégie de nouvelle tentative en cas d’échec de l’origine. 

Cet article fournit une vue d’ensemble des différentes fonctionnalités d’optimisation et des situations dans lesquelles les utiliser. Pour plus d’informations sur les fonctionnalités et limitations, voir les articles concernant chaque type d’optimisation.

> [!NOTE]
> Lorsque vous créez un point de terminaison CDN, les options **Optimisé pour** peuvent varier en fonction du type de profil dans lequel le point de terminaison est créé. Les fournisseurs Azure CDN appliquent les améliorations de différentes façons, selon le scénario. 

## <a name="provider-options"></a>Options de fournisseur

Les profils **Azure CDN Standard fourni par Microsoft** prennent en charge les optimisations suivantes :

* [Livraison web générale](#general-web-delivery). Cette optimisation est également utilisée pour le streaming de contenu multimédia et le téléchargement de fichiers volumineux.

> [!NOTE]
> L’accélération de site dynamique à partir de Microsoft est proposée par le biais d’[Azure Front Door Service](../frontdoor/front-door-overview.md).

Les profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon** prennent en charge les optimisations suivantes :

* [Livraison web générale](#general-web-delivery). Cette optimisation est également utilisée pour le streaming de contenu multimédia et le téléchargement de fichiers volumineux.

* [Accélération de site dynamique](#dynamic-site-acceleration) 


Les profils **Azure CDN Standard fourni par Akamai** prennent en charge les optimisations suivantes :

* [Livraison web générale](#general-web-delivery) 

* [Diffusion multimédia en continu générale](#general-media-streaming)

* [Diffusion multimédia en continu de vidéo à la demande](#video-on-demand-media-streaming)

* [Téléchargement de fichiers volumineux](#large-file-download)

* [Accélération de site dynamique](#dynamic-site-acceleration) 

Microsoft vous recommande de tester les écarts de performances entre les différents fournisseurs afin de sélectionner celui qui convient le mieux pour votre distribution.

## <a name="select-and-configure-optimization-types"></a>Sélectionner et configurer les types d’optimisations

Lorsque vous créez un point de terminaison CDN, sélectionnez le type d’optimisation qui correspond le mieux au scénario et au type de contenu que le point de terminaison doit distribuer. **Livraison web générale** est la sélection par défaut. Pour les points de terminaison **Azure CDN Standard fourni par Akamai** existants uniquement, vous pouvez mettre à jour l’option d’optimisation à tout moment. Cette modification n’interrompt pas la distribution à partir d’Azure CDN. 

1. Dans un profil **Azure CDN Standard fourni par Akamai**, sélectionnez un point de terminaison.

    ![Sélection de point de terminaison](./media/cdn-optimization-overview/01_Akamai.png)

2. Sous PARAMÈTRES, sélectionnez **Optimisation**. Sélectionnez ensuite un type dans la liste déroulante **Optimisé pour**.

    ![Optimisation et sélection de type](./media/cdn-optimization-overview/02_Select.png)

## <a name="optimization-for-specific-scenarios"></a>Optimisation pour des scénarios spécifiques

Vous pouvez optimiser le point de terminaison CDN pour l’un de ces scénarios. 

### <a name="general-web-delivery"></a>Livraison web générale

La livraison web générale est l’option d’optimisation la plus courante. Elle est conçue pour l’optimisation de contenu web générale, par exemple, de pages web et d’applications web. Cette optimisation peut également être utilisée pour les téléchargements de fichiers et de vidéos.

Un site web classique contient du contenu statique et dynamique. Le contenu statique inclut des images, des bibliothèques JavaScript et des feuilles de style qui peuvent être mis en cache et distribués à différents utilisateurs. Le contenu dynamique est personnalisé pour un utilisateur, par exemple, à l’aide d’éléments d’actualités adaptés à un profil utilisateur. Le contenu dynamique, comme le contenu d’un panier, n’est pas mis en cache, car il est unique pour chaque utilisateur. La livraison web générale peut optimiser votre site web tout entier. 

> [!NOTE]
> Si vous utilisez un profil **Azure CDN Standard fourni par Akamai**, sélectionnez ce type d’optimisation lorsque la taille de fichier moyenne est inférieure à 10 Mo. Sinon, lorsque la taille de fichier moyenne est supérieure à 10 Mo, sélectionnez **Téléchargement de fichiers volumineux** dans la liste déroulante **Optimisé pour**.

### <a name="general-media-streaming"></a>Diffusion multimédia en continu générale

Si vous avez besoin d’utiliser le point de terminaison pour le streaming en direct et le streaming vidéo à la demande, sélectionnez le type d’optimisation de streaming multimédia générale.

Le streaming multimédia est soumis à une contrainte de temps, car des paquets qui arrivent en retard sur le client, comme la mise en mémoire tampon fréquente du contenu vidéo, peuvent entraîner une dégradation de l’expérience de visualisation. L’optimisation de la diffusion multimédia en continu réduit la latence de distribution de contenu multimédia et offre une expérience de diffusion en continu lisse aux utilisateurs. 

Ce scénario est courant pour les clients du service multimédia Azure. Lorsque vous utilisez des services multimédias Azure, vous obtenez un point de terminaison de streaming unique qui peut être utilisé pour le streaming en direct et à la demande. Avec ce scénario, les clients n’ont pas besoin de basculer d’un point de terminaison à un autre quand ils passent d’une diffusion en direct à une diffusion à la demande. L’optimisation de la diffusion multimédia en continu générale prend en charge ce type de scénario.

Pour **Azure CDN Standard de Microsoft**, **Azure CDN Standard de Verizon** et **Azure CDN Premium de Verizon**, utilisez le type d’optimisation Livraison web générale pour distribuer du contenu multimédia général en streaming.

Pour plus d’informations sur l’optimisation de la diffusion multimédia en continu, consultez [Optimisation de la diffusion multimédia en continu](cdn-media-streaming-optimization.md).

### <a name="video-on-demand-media-streaming"></a>Diffusion multimédia en continu de vidéo à la demande

L’optimisation de la diffusion multimédia en continu de vidéo à la demande améliore le contenu de diffusion en continu de vidéo à la demande. Si vous utilisez un point de terminaison de streaming vidéo à la demande, utilisez cette option.

Pour les profils **Azure CDN Standard fourni par Microsoft**, **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, utilisez le type d’optimisation Livraison web générale pour distribuer du contenu de streaming vidéo à la demande.

Pour plus d’informations sur l’optimisation de la diffusion multimédia en continu, consultez [Optimisation de la diffusion multimédia en continu](cdn-media-streaming-optimization.md).

> [!NOTE]
> Si le point de terminaison CDN sert principalement du contenu de vidéo à la demande, utilisez ce type d’optimisation. La principale différence entre ce type d’optimisation et le type d’optimisation du streaming multimédia générale est le délai d’attente de nouvelle tentative de connexion. Le délai d’attente est beaucoup plus court pour les scénarios de diffusion en continu en direct.
>

### <a name="large-file-download"></a>Téléchargement de fichiers volumineux

Pour les profils **Azure CDN Standard fourni par Akamai**, les téléchargements de fichiers volumineux sont optimisés pour du contenu d’une taille supérieure à 10 Mo. Si la taille moyenne des fichiers est inférieure à 10 Mo, utilisez la livraison web générale. Si la taille moyenne de vos fichiers est constamment supérieure à 10 Mo, il peut être plus efficace de créer un point de terminaison distinct pour les fichiers volumineux. Par exemple, les mises à jour de microprogrammes ou de logiciels sont généralement des fichiers volumineux. Pour distribuer des fichiers de plus de 1,8 Go, l’optimisation de téléchargement de fichier volumineux est obligatoire.

Pour les profils **Azure CDN Standard fourni par Microsoft**, **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, utilisez le type d’optimisation Livraison web générale pour distribuer du contenu de téléchargement de fichiers volumineux. Il n’existe aucune limitation quant à la taille du téléchargement de fichier.

Pour plus d’informations sur l’optimisation des fichiers volumineux, consultez [Optimisation des fichiers volumineux](cdn-large-file-optimization.md).

### <a name="dynamic-site-acceleration"></a>Accélération de site dynamique

 L’accélération de site dynamique est disponible pour les profils **Azure CDN Standard fourni par Akamai**, **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**. Cette optimisation implique des frais supplémentaires d’utilisation. Pour plus d’informations, consultez [Tarification Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

> [!NOTE]
> L’accélération de site dynamique à partir de Microsoft est proposée par le biais d’[Azure Front Door Service](../frontdoor/front-door-overview.md), service [anycast](https://en.wikipedia.org/wiki/Anycast) global tirant parti du réseau mondial privé de Microsoft pour fournir vos charges de travail d’application.

L’accélération de site dynamique inclut diverses techniques bénéfiques pour la latence et les performances du contenu dynamique. Ces techniques incluent l’optimisation d’itinéraire et de réseau, l’optimisation TCP, et bien plus. 

Vous pouvez utiliser cette optimisation pour accélérer une application web incluant de nombreuses réponses qui ne peuvent pas être mises en cache. Les résultats de recherche, les transactions de validation ou les données en temps réel en sont des exemples. Vous pouvez continuer d’utiliser les fonctionnalités de mise en cache Azure CDN essentielles pour les données statiques. 

Pour plus d’informations sur l’accélération de site dynamique, consultez [Accélération de site dynamique](cdn-dynamic-site-acceleration.md).