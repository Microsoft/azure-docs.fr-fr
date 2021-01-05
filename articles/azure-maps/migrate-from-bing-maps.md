---
title: 'Tutoriel : Migrer de Bing Cartes vers Azure Maps | Microsoft Azure Maps'
description: Tutoriel sur la migration de Bing Cartes vers Microsoft Azure Maps. Vous aide à passer aux API et kits SDK Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 9/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 0045520849ea20d3e53a30101e6db0f5d495ab15
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897005"
---
# <a name="tutorial---migrate-from-bing-maps-to-azure-maps"></a>Tutoriel – Migrer de Bing Cartes vers Azure Maps

Ce guide fournit des insights sur la migration d’applications web, mobiles et basées sur serveur de Bing Cartes vers la plateforme Azure Maps. Ce guide comprend des exemples de code comparatifs, des suggestions de migration et des bonnes pratiques pour la migration vers Azure Maps.

## <a name="azure-maps-platform-overview"></a>Vue d’ensemble de la plateforme Azure Maps

Azure Maps offre aux développeurs de tous les secteurs d’activité de puissantes fonctionnalités géospatiales comprenant les toutes dernières données cartographiques pour fournir un contexte géographique aux applications mobiles et web. Azure Maps est un jeu d’API REST conforme à l’API Azure One pour les cartes, la recherche, les itinéraires, le trafic, les fuseaux horaires, le geofencing, les données cartographiques, les données météorologiques et de nombreux autres services, accompagné de kits SDK web et Android afin de rendre le développement facile, flexible et portable sur de nombreuses plateformes. [Azure Maps est également disponible dans Power BI](power-bi-visual-getting-started.md).

## <a name="high-level-platform-comparison"></a>Comparaison générale de plateformes

Le tableau suivant fournit une liste générale des fonctionnalités de Bing Cartes et la prise en charge relative de ces fonctionnalités dans Azure Maps. Cette liste ne contient pas d’autres fonctionnalités Azure Maps telles que l’accessibilité, les API de geofencing, les services de trafic, les opérations spatiales, l’accès direct aux mosaïques et les services Batch.

| Fonctionnalité Bing Cartes                     | Support Azure Maps |
|---------------------------------------|:------------------:|
| Kit de développement logiciel (SDK) web                               | ✓                  |
| Kit de développement logiciel Android                           | ✓                  |
| Kit de développement logiciel (SDK) iOS                               | Prévu            |
| SDK UWP                               | Prévu            |
| SDK WPF                               | Prévu            |
| API de service REST                     | ✓                  |
| Suggestion automatique                           | ✓                  |
| Directions (y compris camion)          | ✓                  |
| Matrice des distances                       | ✓                  |
| Élévations                            | Prévu            |
| Imagerie – Carte statique                  | ✓                  |
| Métadonnées d’imagerie                      | ✓                  |
| Isochrones                            | ✓                  |
| Local Insights                        | ✓                  |
| Recherche locale                          | ✓                  |
| Reconnaissance des localisations                  | ✓                  |
| Localisations (géocodage direct/inversé) | ✓                  |
| Itinéraires optimisés            | Prévu            |
| Snap to roads (ancrer sur les routes)                         | ✓                  |
| Services de données spatiales           | Partial            |
| Time Zone (Fuseau horaire)                             | ✓                  |
| Incidents de trafic                     | ✓                  |
| Cartes pilotées par la configuration             | N/A                |

Bing Cartes fournit une authentification basée sur les clés de base. Azure Maps fournit une authentification basée sur les clés de base, ainsi qu’une authentification Azure Active Directory hautement sécurisée.

## <a name="licensing-considerations"></a>Considérations relatives aux licences

Lors de la migration vers Azure Maps à partir de Bing Cartes, les points suivants doivent être pris en compte en ce qui concerne les licences.

-   Azure Maps facture l’utilisation de cartes interactives en fonction du nombre de mosaïques chargées, tandis que Bing Cartes facture le chargement du contrôle de carte (sessions). Sur Azure Maps, les mosaïques sont automatiquement mises en cache afin de réduire le coût pour le développeur. Une transaction Azure Maps est générée toutes les 15 mosaïques qui sont chargées. Les kits de développement logiciel (SDK) Azure Maps interactifs utilisent des mosaïques de 512 pixels et génèrent en moyenne une transaction ou moins par vue de page.

-   Azure Maps permet de stocker les données de sa plateforme dans Azure. Elles peuvent également être mises en cache ailleurs pendant six mois au maximum, en fonction des [conditions d’utilisation](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31).

Voici quelques ressources liées aux licences pour Azure Maps :

-   [Page des tarifs Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/)
-   [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/?service=azure-maps)
-   [Conditions d’utilisation Azure Maps](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31) (incluses dans les conditions d’utilisation de Microsoft Online Services)
-   [Choix du bon niveau tarifaire dans Azure Maps](./choose-pricing-tier.md)

## <a name="suggested-migration-plan"></a>Plan de migration suggéré

Voici un plan de migration général.

1.  Effectuez l’inventaire des services et des kits SDK Bing Cartes que votre application utilise, et vérifiez qu’Azure Maps fournit d’autres kits SDK et services vers lesquels migrer.
2.  Créez un abonnement Azure (si vous n’en avez pas encore) à l’adresse <https://azure.com>.
3.  Créez un compte Azure Maps ([documentation](./how-to-manage-account-keys.md)) et une clé d’authentification ou Azure Active Directory ([documentation](./how-to-manage-authentication.md)).
4.  Migrez votre code d’application.
5.  Testez votre application migrée.
6.  Déployez votre application migrée en production.

## <a name="create-an-azure-maps-account"></a>Créer un compte Azure Maps

Pour créer un compte Azure Maps et accéder à la plateforme Azure Maps, effectuez les étapes suivantes :

1. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. Connectez-vous au [portail Azure](https://portal.azure.com/).
3. Créez un [compte Azure Maps](./how-to-manage-account-keys.md). 
4. [Procurez-vous votre clé d’abonnement Azure Maps](./how-to-manage-authentication.md#view-authentication-details) ou configurez l’authentification Azure Active Directory pour renforcer la sécurité.

## <a name="azure-maps-technical-resources"></a>Ressources techniques Azure Maps

Voici une liste de ressources techniques utiles pour Azure Maps.

-   Vue d’ensemble : https://azure.com/maps
-   Documentation : <https://aka.ms/AzureMapsDocs>
-   Exemples de code de kit de développement logiciel web : <https://aka.ms/AzureMapsSamples>
-   Forums des développeurs : <https://aka.ms/AzureMapsForums>
-   Vidéos : <https://aka.ms/AzureMapsVideos>
-   Blog : <https://aka.ms/AzureMapsBlog>
-   Commentaires Azure Maps (UserVoice) : <https://aka.ms/AzureMapsFeedback>

## <a name="migration-support"></a>Prise en charge de la migration

Les développeurs peuvent rechercher la prise en charge de la migration via les [forums](/answers/topics/azure-maps.html) ou via l’une des nombreuses options de support Azure : <https://azure.microsoft.com/support/options/>

## <a name="new-terminology"></a>Nouvelle terminologie 

Voici une liste de termes courants dans Bing Cartes qui ont une appellation différente dans Azure Maps.

| Terme Bing Cartes                    | Terme Azure Maps                                                |
|-----------------------------------|----------------------------------------------------------------|
| Aérien                            | Satellite ou aérien                                            |
| Directions                        | Peut également être appelé « itinéraire »                             |
| Entités                          | Géométries ou caractéristiques                                         |
| `EntityCollection`                | Couche ou source de données                                           |
| `Geopoint`                        | Position                                                       |
| `GeoXML`                          | Fichiers XML dans le module d’E/S spatiales                             |
| Superposition au sol                    | Couche d’images                                                    |
| Hybride (en référence au type de carte) | Satellite avec routes                                           |
| Zone d’informations                           | Fenêtre contextuelle                                                          |
| Emplacement                          | Position                                                       |
| `LocationRect`                    | Rectangle englobant                                                   |
| Type de mappage                          | Style de carte                                                      |
| Barre de navigation                    | Sélecteur de style de carte, contrôle de zoom, contrôle de tangage, contrôle de boussole |
| Punaise                           | Calque de bulles, calque de symboles ou marqueur HTML                      |

## <a name="next-steps"></a>Étapes suivantes

Découvrez en détail comment migrer votre application Bing Cartes avec les articles suivants :

> [!div class="nextstepaction"]
> [Migrer une application web](migrate-from-bing-maps-web-app.md)

> [!div class="nextstepaction"]
> [Migrer un service web](migrate-from-bing-maps-web-services.md)