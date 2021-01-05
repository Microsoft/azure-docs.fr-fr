---
title: Analyser les performances du nœud de périmètre dans Azure CDN | Microsoft Docs
description: Analyser les performances du nœud de périmètre dans Microsoft Azure CDN. Edge Performance Analytics fournit des informations précises sur le trafic et la bande passante utilisés pour le CDN.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 8cc596a7-3e01-4f76-af7b-a05a1421517e
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: d8119c56c26ed32db136064d93775d21a71f238f
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778664"
---
# <a name="analyze-edge-node-performance-in-microsoft-azure-cdn"></a>Analyser les performances du nœud de périmètre dans Microsoft Azure CDN
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>Vue d’ensemble
Edge Performance Analytics fournit des informations précises sur le trafic et la bande passante utilisés pour le CDN. Ces informations peuvent ensuite être exploitées pour générer des statistiques de tendances, ce qui vous permet d’obtenir des informations sur la manière dont vos ressources sont mises en cache et distribuées à vos clients. Vous pouvez ensuite formuler une stratégie sur la façon d’optimiser la distribution de votre contenu et déterminer les problèmes à résoudre afin de mieux tirer parti du CDN. Vous pourrez ainsi non seulement améliorer les performances de diffusion des données mais également réduire vos coûts CDN.

> [!NOTE]
> Tous les rapports utilisent la notation UTC/GMT lorsque vous spécifiez une date ou une heure.
> 
> 

## <a name="reports-and-log-collection"></a>Rapports et collecte de journaux
Les données d’activité CDN doivent être collectées par le module Edge Performance Analytics pour que ce dernier puisse générer les rapports correspondants. Ce processus intervient une fois par jour et traite l’activité recensée au cours de la journée précédente. Autrement dit, les statistiques d’un rapport représentent un échantillon des statistiques du jour au moment du traitement, et ne contiennent donc pas nécessairement le jeu de données complet correspondant à la date du jour. Ces rapports visent essentiellement à évaluer les performances. Ils ne doivent pas être utilisés à des fins de facturation ou pour générer des statistiques numériques exactes.

> [!NOTE]
> Les données brutes à partir desquelles sont générés les rapports Edge Performance Analytics sont disponibles pendant au moins 90 jours.
> 
> 

## <a name="dashboard"></a>tableau de bord
Le tableau de bord Edge Performance Analytics assure le suivi du trafic CDN actuel et historique par le biais d’un graphique et de statistiques. Ce tableau de bord vous permet de détecter les tendances récentes et à long terme concernant les performances du trafic CDN pour votre compte.

Ce tableau de bord se décompose comme suit :

* Un graphique interactif permettant de visualiser les mesures et tendances clés.
* Une chronologie donnant une idée des schémas à long terme pour les mesures et tendances clés.
* Des mesures clés et des informations statistiques sur la façon dont notre réseau CDN améliore le trafic du site mesuré en termes de performances globales, d’utilisation et d’efficacité.

### <a name="accessing-the-edge-performance-dashboard"></a>Accès au tableau de bord des performances de périmètre
1. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton de gestion du panneau de profil CDN](./media/cdn-edge-performance/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **Analytics** , puis sur le menu volant **Analyse des performances de périmètre** .  Cliquez sur **Tableau de bord** .
   
    Le tableau de bord d’analyse des nœuds de périmètre s’affiche.

### <a name="chart"></a>Graphique
Le tableau de bord contient un graphique qui assure le suivi d’une mesure pendant la période sélectionnée dans la chronologie affichée en dessous.  Une chronologie retraçant les deux dernières années d’activité CDN s’affiche directement sous le graphique.

#### <a name="using-the-chart"></a>Utilisation du graphique
* Par défaut, le taux de rendement du cache pour les 30 derniers jours est représenté sur le graphique.
* Ce graphique est généré à partir des données recueillies chaque jour.
* Lorsque vous pointez sur une journée du graphique linéaire, vous obtenez une date ainsi que la valeur de la métrique correspondante.
* Cliquez sur Highlight Weekends pour activer une superposition de barres verticales en gris clair représentant les week-ends sur le graphique. Ce type de superposition est utile pour identifier les modèles de trafic pendant les week-ends.
* Cliquez sur View One Year Ago pour superposer sur le graphique l’activité de l’année précédente sur la même période de temps. Ce type de comparaison permet de connaître les modèles d’utilisation à long terme du CDN. La partie en haut à droite du graphique contient une légende indiquant le code couleur pour chaque graphique linéaire.

#### <a name="updating-the-chart"></a>Mise à jour du graphique
* Time Range : effectuez l’une des opérations suivantes :
  * Sélectionnez la région souhaitée dans la chronologie. Le graphique sera actualisé avec les données correspondant à la période sélectionnée.
  * Double-cliquez sur le graphique pour afficher toutes les données historiques disponibles sur une période maximum de deux ans.
* Metric : cliquez sur l’icône de graphique qui s’affiche en regard de la mesure souhaitée. Le graphique et la chronologie seront actualisés avec les données de la mesure correspondante.

### <a name="key-metrics-and-statistics"></a>Statistiques et mesures clés
#### <a name="efficiency-metrics"></a>Mesures de l’efficacité
Ces mesures visent à déterminer si l’efficacité du cache peut être améliorée. L’efficacité du cache présente essentiellement les avantages suivants :

* Réduction de la charge sur le serveur d'origine pouvant conduire à :
  * une amélioration des performances du serveur web ;
  * une réduction des coûts d’exploitation.
* Accélération de la diffusion des données grâce à un traitement d’un plus grand nombre de requêtes directement à partir du CDN.

| Champ | Description |
| --- | --- |
| Cache Efficiency |Indique le pourcentage de données transférées à partir du cache. Cette mesure permet de déterminer à quel moment une version mise en cache du contenu demandé a été directement prise en charge par le CDN (serveurs Edge) pour les demandeurs (par exemple, le navigateur web). |
| Hit Rate |Indique le pourcentage de requêtes qui ont été traitées à partir du cache. Cette mesure permet de déterminer à quel moment une version mise en cache du contenu demandé a été directement prise en charge par le CDN (serveurs Edge) pour les demandeurs (par exemple, le navigateur web). |
| % of Remote Bytes - No Cache Config |Indique le pourcentage de trafic envoyé par les serveurs d’origine au CDN (serveurs Edge) qui n’est pas mis en cache en raison de la fonctionnalité de cache de contournement (moteur de règles HTTP). |
| % of Remote Bytes - Expired Cache |Indique le pourcentage de trafic envoyé par les serveurs d’origine au CDN (serveurs Edge) en raison d’une revalidation de contenu obsolète. |

#### <a name="usage-metrics"></a>Mesures d’utilisation
Ces mesures ont pour but de fournir des informations sur les mesures de réduction des coûts suivantes :

* Réduction des coûts d'exploitation via le CDN.
* Réduction des dépenses CDN grâce à l’efficacité du cache et à la compression.

> [!NOTE]
> Les nombres associés au volume de trafic représentent le trafic qui a été utilisé pour les calculs de rapports et de pourcentages ; ils ne peuvent afficher qu’une partie du trafic total pour les clients consommant de gros volumes de données.
> 
> 

| Champ | Description |
| --- | --- |
| Ave Bytes Out |Indique le nombre moyen d’octets transférés pour chaque demande traitée entre le CDN (serveurs Edge) et le demandeur (par exemple, le navigateur web). |
| No Cache Config Byte Rate |Indique le pourcentage de trafic traité entre le CDN (serveurs Edge) et le demandeur (par exemple, le navigateur web) qui n’est pas mis en cache en raison de la fonctionnalité de cache de contournement. |
| Compressed Byte Rate |Indique le pourcentage de trafic envoyé par le CDN (serveurs Edge) aux demandeurs (par exemple, le navigateur web) dans un format compressé. |
| Bytes Out |Indique la quantité de données, en octets, distribuées entre le CDN (serveurs Edge) et le demandeur (par exemple, le navigateur web). |
| Bytes In |Indique la quantité de données, en octets, envoyées par les demandeurs (par exemple, le navigateur web) au CDN (serveurs Edge). |
| Bytes Remote |Indique la quantité de données, en octets, envoyées par le CDN et les serveurs d’origine du client au CDN (serveurs Edge). |

#### <a name="performance-metrics"></a>Mesures de performances
Ces mesures ont pour but de suivre les performances globales du CDN du point de vue votre trafic.

| Champ | Description |
| --- | --- |
| Transfer Rate |Indique la vitesse moyenne de transfert du contenu entre le CDN et un demandeur. |
| Duration |Indique la durée moyenne, en millisecondes, nécessaire à la distribution d’une ressource à un demandeur (par exemple, le navigateur web). |
| Compressed Request Rate |Indique le pourcentage de correspondances envoyées par le CDN (serveurs Edge) au demandeur (par exemple, le navigateur web) dans un format compressé. |
| 4xx Error Rate |Indique le pourcentage de correspondances ayant généré un code d’état 4xx. |
| 5xx Error Rate |Indique le pourcentage de correspondances ayant généré un code d’état 5xx. |
| Correspondances |Indique le nombre de demandes de contenu CDN. |

#### <a name="secure-traffic-metrics"></a>Mesures de trafic sécurisé
Ces mesures ont pour but de suivre les performances du CDN du point de vue du trafic HTTPS.

| Champ | Description |
| --- | --- |
| Secure Cache Efficiency |Indique le pourcentage de données transférées pour les demandes HTTPS qui ont été traitées à partir du cache. Cette mesure permet de déterminer à quel moment une version mise en cache du contenu demandé a été directement prise en charge par le CDN (serveurs Edge) pour les demandeurs (par exemple, le navigateur web) via HTTPS. |
| Secure Transfer Rate |Indique la vitesse moyenne de transfert du contenu entre le CDN (serveurs Edge) et les demandeurs (par exemple, les serveurs web) via le protocole HTTPS. |
| Average Secure Duration |Indique la durée moyenne, en millisecondes, nécessaire à la distribution d’une ressource à un demandeur (par exemple, le navigateur web) via le protocole HTTPS. |
| Secure Hits |Indique le nombre de demandes HTTPS de contenu CDN. |
| Secure Bytes Out |Indique la quantité de trafic HTTPS, en octets, distribué entre le CDN (serveurs Edge) et le demandeur (par exemple, le navigateur web). |

## <a name="reports"></a>Rapports
Chaque rapport de ce module contient un graphique et des statistiques sur l’utilisation de la bande passante et du trafic pour différents types de mesures (par exemple, codes d’état HTTP, codes d’état du cache, URL de demande, etc..). Ces informations peuvent être utilisées pour explorer plus en détail la manière dont le contenu est diffusé à vos clients et pour ajuster le comportement du CDN afin d’améliorer les performances de diffusion des données.

### <a name="accessing-the-edge-performance-reports"></a>Accès aux rapports de performances de périmètre
1. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton de gestion du panneau de profil CDN](./media/cdn-edge-performance/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **Analytics** , puis sur le menu volant **Analyse des performances de périmètre** .  Cliquez sur **HTTP Large Object** .
   
    L’écran des rapports d’analyse des nœuds de périmètre s’affiche.

| Rapport | Description |
| --- | --- |
| Daily Summary |Permet d’afficher les tendances du trafic quotidien sur une période spécifiée. Chaque barre sur ce graphique représente une date particulière. La taille de la barre indique le nombre total de correspondances recensées à cette date. |
| Hourly Summary |Permet d’afficher les tendances du trafic horaire sur une période spécifiée. Chaque barre de ce graphique représente une heure donnée à une date particulière. La taille de la barre indique le nombre total de correspondances recensées pendant l’heure en question. |
| Protocoles |Affiche la répartition du trafic entre les protocoles HTTP et HTTPS. Un graphique en anneau indique le pourcentage de correspondances recensées pour chaque type de protocole. |
| HTTP Methods |Permet d’identifier rapidement les méthodes HTTP utilisées pour les demandes de données. En règle générale, les méthodes de demande HTTP les plus courantes sont GET, HEAD et POST. Un graphique en anneau indique le pourcentage de correspondances recensées pour chaque type de méthode de demande HTTP. |
| URLs |Contient un graphique affichant les 10 URL les plus demandées. Une barre s’affiche pour chaque URL. La hauteur de la barre indique le nombre de correspondances qu’une URL donnée a généré au cours de la période couverte par le rapport. Les statistiques correspondant aux 100 URL les plus demandées s’affichent directement sous ce graphique. |
| CNAMEs |Contient un graphique qui affiche les 10 principaux enregistrements CNAME utilisés pour demander des ressources au cours de la période couverte par le rapport. Les statistiques correspondant aux 100 enregistrements CNAME les plus demandés s’affichent directement sous ce graphique. |
| Origins |Contient un graphique qui affiche les 10 principaux CDN ou serveurs d’origine du client à partir desquels ont émises des demandes de ressources sur une période de temps spécifiée. Les statistiques correspondant aux 100 CDN ou serveurs d’origine client les plus demandés s’affichent directement sous ce graphique. Les serveurs d’origine client sont identifiés par le nom défini dans l’option Directory Name. |
| Geo POPs |Affiche la quantité de trafic acheminé via un point de présence (POP) particulier. L’abréviation à trois lettres représente un POP dans notre réseau CDN. |
| Clients |Contient un graphique qui affiche les 10 clients ayant demandé des ressources sur une période de temps spécifiée. Dans le cadre de ce rapport, on considère que toutes les demandes provenant de la même adresse IP émanent du même client. Les statistiques correspondant aux 100 premiers clients s’affichent directement sous ce graphique. Ce rapport est utile pour déterminer les modèles d’activité de téléchargement pour vos principaux clients. |
| États du cache |Fournit une analyse détaillée du comportement de cache, qui peut révéler certaines approches afin d’améliorer l’expérience utilisateur globale. Étant donné que les meilleures performances proviennent de correspondances dans le cache, vous pouvez optimiser les vitesses de remise de données en réduisant les absences dans le cache et les correspondances dans le cache expirées. |
| NONE Details |Contient un graphique qui affiche les 10 principales URL associées aux ressources pour lesquelles l’actualisation du contenu du cache n’a pas été vérifiée sur une période de temps spécifiée. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| CONFIG_NOCACHE Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources qui n’ont pas été mises en cache en raison de la configuration du CDN client. Ces types de ressources ont été directement pris en charge par le serveur d’origine. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| UNCACHEABLE Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources qui n’ont pas pu être mises en cache en raison des données d’en-tête de la demande. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| TCP_HIT Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources qui ont été immédiatement prises en charge à partir du cache. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| TCP_MISS Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources dont l’état du cache est défini sur « TCP_MISS ». Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| TCP_EXPIRED_HIT Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources obsolètes qui ont été traitées directement à partir du point de présence. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| TCP_EXPIRED_MISS Details |Contient un graphique qui affiche les 10 principales URL associées à des ressources obsolètes dont une nouvelle version a dû être récupérée à partir du serveur d’origine. Les statistiques correspondant aux 100 premières URL associées à ces types de ressources s’affichent directement sous ce graphique. |
| TCP_CLIENT_REFRESH_MISS Details |Contient un graphique à barres qui affiche les 10 premières URL associées à des ressources qui ont été récupérées à partir d’un serveur d’origine en raison d’une demande « no-cache » du client. Les statistiques correspondant aux 100 premières URL associées à ces types de demandes s’affichent directement sous ce graphique. |
| Client Request Types |Indique le type de demandes exécutées par les clients HTTP (par exemple, les navigateurs). Ce rapport inclut un graphique en anneau offrant une vue d’ensemble du traitement de ces demandes. Les informations de trafic et de bande passante sont affichées sous le graphique pour chaque type de demande. |
| User Agent |Contient un graphique à barres affichant les 10 principaux agents utilisateurs qui demandent du contenu via notre CDN. En règle générale, un agent utilisateur est un navigateur web, un lecteur multimédia ou un navigateur de téléphone mobile. Les statistiques correspondant aux 100 premiers agents utilisateurs s’affichent directement sous ce graphique. |
| Referrers |Contient un graphique à barres affichant les 10 principales références au contenu consulté via notre CDN. En règle générale, une référence correspond à l’URL de la page web ou d’une ressource comportant un lien vers votre contenu. Des informations détaillées sont fournies sous le graphique pour les 100 principales références. |
| Compression Types |Contient un graphique en anneau qui répartit les ressources demandées selon qu’elles ont été ou non compressées par nos serveurs Edge. Le pourcentage de ressources compressées est réparti selon le type de compression utilisé. Des informations détaillées sont fournies sous le graphique pour chaque type de compression et pour chaque état. |
| Types de fichiers |Contient un graphique à barres qui affiche les 10 principaux types de fichiers qui ont été demandés via notre CDN pour votre compte. Dans le cadre de ce rapport, un type de fichier est défini par l’extension du nom de fichier de la ressource et par le type de support Internet (par exemple, .html \[texte/html\], .htm \[texte/html\], .aspx \[texte/html\], etc.). Des informations détaillées sont fournies sous le graphique pour les 100 principaux types de fichiers. |
| Unique Files |Contient un graphique représentant le nombre total de ressources uniques qui ont été demandées sur une journée donnée au cours d’une période de temps spécifiée. |
| Token Auth Summary |Contient un graphique à secteurs décrivant brièvement si les ressources demandées étaient ou non protégées par une authentification par jeton. Les ressources protégées s’affichent sur le graphique en fonction des résultats de leur tentative d’authentification. |
| Token Auth Deny Details |Contient un graphique à barres qui vous permet d’afficher les 10 principales demandes refusées en raison de l’authentification par jeton. |
| HTTP Response Codes |Fournit une liste des codes d’état HTTP (par exemple, 200 OK, 403 Forbidden, 404 Not Found, etc.) qui ont été remis à vos clients HTTP par nos serveurs Edge. Un graphique à secteurs vous permet d’évaluer rapidement la façon dont vos ressources ont été traitées. Des données statistiques détaillées sont fournies pour chaque code de réponse sous le graphique. |
| 404 Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse « 404 Not Found ». |
| 403 Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse « 403 Forbidden ». Un code de réponse 403 Forbidden est renvoyé lorsqu’une demande est refusée par un serveur d’origine client ou un serveur Edge dans notre point de présence. |
| 4xx Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse dans la plage 400. Les codes de réponse « 403 Not Found » et « 404 Forbidden » sont exclus de ce rapport. En règle générale, un code de réponse 4xx est renvoyé lorsqu’une demande est refusée en raison d’une erreur client. |
| 504 Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse « 504 Gateway Timeout ». Un code de réponse « 504 Gateway Timeout » est renvoyé en cas d’expiration du délai d’attente lorsqu’un proxy HTTP tente de communiquer avec un autre serveur. Dans le cas de notre CDN, un code de réponse « 504 Gateway Timeout » se produit généralement lorsqu’un serveur Edge ne parvient pas à établir la communication avec un serveur d’origine client. |
| 502 Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse « 502 Bad Gateway ». Un code de réponse « 502 Bad Gateway » est renvoyé en cas d'échec du protocole HTTP entre un serveur et un proxy HTTP. Dans le cas de notre CDN, un code de réponse « 502 Bad Gateway » se produit généralement lorsqu’un serveur d'origine client renvoie une réponse non valide à un serveur Edge. Une réponse est considérée comme non valide si elle ne peut pas être analysée ou si elle est incomplète. |
| 5xx Errors |Contient un graphique à barres qui vous permet d’afficher les 10 premières requêtes ayant abouti à un code de réponse dans la plage 500.  Les codes de réponse « 502 Bad Gateway » et « 504 Gateway Timeout » sont exclus de ce rapport. |

## <a name="see-also"></a>Voir aussi
* [Vue d'ensemble d'Azure CDN](cdn-overview.md)
* [Statistiques en temps dans Microsoft Azure CDN](cdn-real-time-stats.md)
* [Remplacement du comportement HTTP par défaut à l’aide du moteur de règles](./cdn-verizon-premium-rules-engine.md)
* [Rapports HTTP avancés](cdn-advanced-http-reports.md)