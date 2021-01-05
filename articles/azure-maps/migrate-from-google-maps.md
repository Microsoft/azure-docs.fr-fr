---
title: Tutoriel – Migrer de Google Maps vers Azure Maps | Microsoft Azure Maps
description: Tutoriel sur la migration de Google Maps vers Microsoft Azure Maps. Vous aide à passer aux API et kits SDK Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 09/23/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 61b6f4a530ab1861c67b0bd4983167546e268957
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896325"
---
# <a name="tutorial---migrate-from-google-maps-to-azure-maps"></a>Tutoriel – Migrer de Google Maps vers Azure Maps

Cet article fournit des insights sur la migration d’applications web, mobiles et basées sur serveur de Google Maps vers la plateforme Microsoft Azure Maps. Ce didacticiel comprend des exemples de code comparatifs, des suggestions de migration et des meilleures pratiques pour la migration vers Azure Maps. Ce tutoriel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Comparaison générale entre les fonctionnalités Google Maps et leurs équivalents dans Azure Maps
> * Différences de gestion des licences à prendre en compte
> * Planification de votre migration
> * Où trouver des ressources techniques et un support technique

## <a name="prerequisites"></a>Prérequis 

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](how-to-manage-authentication.md).

## <a name="azure-maps-platform-overview"></a>Vue d’ensemble de la plateforme Azure Maps

Azure Maps fournit aux développeurs de tous les secteurs d’activité des fonctionnalités géospatiales puissantes. Les fonctionnalités tirent parti de données cartographiques régulièrement mises à jour, afin de fournir un contexte géographique pour les applications web et mobiles. Azure Maps dispose d’un ensemble d’API REST conformes à l’API Azure One. Les API REST offrent des fonctionnalités de rendu de cartes, de recherche, d’itinéraires, de trafic, de fuseaux horaires, de géolocalisation, de geofencing, de données cartographiques, de météo, de mobilité et d’opérations spatiales. Les opérations sont accompagnées de SDK web et Android pour rendre le développement facile, flexible et portable sur de nombreuses plateformes.

## <a name="high-level-platform-comparison"></a>Comparaison générale de plateformes

Le tableau suivant fournit une liste générale de fonctionnalités Azure Maps qui correspondent aux fonctionnalités Google Maps. Cette liste ne contient pas toutes les fonctionnalités Azure Maps. Azure Maps propose d’autres fonctionnalités telles que l’accessibilité, le geofencing, les isochrones, les opérations spatiales, l’accès direct aux mosaïques de carte, les services Batch et les comparaisons de couverture de données (autrement dit, la couverture d’imagerie).

| Fonctionnalité Google Maps         | Support Azure Maps                     |
|-----------------------------|:--------------------------------------:|
| Kit de développement logiciel (SDK) web                     | ✓                                      |
| Kit de développement logiciel Android                 | ✓                                      |
| Kit de développement logiciel (SDK) iOS                     | Prévu                                |
| API de service REST           | ✓                                      |
| Directions (routage)        | ✓                                      |
| Matrice des distances             | ✓                                      |
| Elevation                   | Prévu                                |
| Géocodage (transfert/inversé) | ✓                                      |
| Géolocalisation                 | N/A                                    |
| Nearest Roads (routes les plus proches)               | ✓                                      |
| Recherche de lieux               | ✓                                      |
| Détails des lieux              | N/A : site web et numéro de téléphone disponibles |
| Photos des lieux               | N/A                                    |
| Autocomplétion des lieux          | ✓                                      |
| Snap to Roads (aligner sur les routes)                | ✓                                      |
| Speed Limits (limitations de vitesse)                | ✓                                      |
| Cartes statiques                 | ✓                                      |
| Affichage des rues statiques          | N/A                                    |
| Time Zone (Fuseau horaire)                   | ✓                                      |
| API Incorporation de cartes           | N/A                                    |
| URL de cartes                    | N/A                                    |

Google Maps fournit une authentification basée sur les clés de base. Azure Maps fournit l’authentification basée sur les clés de base et l’authentification Azure Active Directory. L’authentification Azure Active Directory offre davantage de fonctionnalités de sécurité que l’authentification basée sur les clés de base.

## <a name="licensing-considerations"></a>Considérations relatives aux licences

Lors de la migration vers Azure Maps à partir de Google Maps, vous devez prendre en compte les points suivants en ce qui concerne les licences.

- Azure Maps facture l’utilisation de cartes interactives, qui est basée sur le nombre de mosaïques de carte chargées. Google Maps, quant à lui, facture le chargement du contrôle de carte. Dans les SDK Azure Maps interactifs, les mosaïques sont automatiquement mises en cache afin de réduire le coût de développement. Une transaction Azure Maps est générée toutes les 15 mosaïques qui sont chargées. Les SDK Azure Maps interactifs utilisent des mosaïques de 512 pixels et génèrent en moyenne une transaction ou moins par vue de page.
- Souvent, il est plus économique de remplacer les images de carte statiques des services web Google Maps par le SDK web Azure Maps. Le SDK web Azure Maps utilise des mosaïques. Sauf si l’utilisateur effectue des panoramiques et des zooms sur la carte, le service ne génère souvent qu’une fraction de transaction par chargement de carte. Le SDK web Azure Maps dispose d’options pour désactiver le panoramique et le zoom, si vous le souhaitez. Il offre également beaucoup plus d’options de visualisation que le service web de cartes statiques.
- Azure Maps permet de stocker les données de sa plateforme dans Azure. Les données peuvent aussi être mises en cache ailleurs pendant six mois au maximum, en fonction des [conditions d’utilisation](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=46).

Voici quelques ressources associées pour Azure Maps :

- [Page des tarifs Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/)
- [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/?service=azure-maps)
- [Conditions d’utilisation Azure Maps](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=46) (incluses dans les conditions d’utilisation de Microsoft Online Services)
- [Choix du bon niveau tarifaire dans Azure Maps](./choose-pricing-tier.md)

## <a name="suggested-migration-plan"></a>Plan de migration suggéré

Voici un plan de migration général.

1. Effectuez l’inventaire des services et des SDK Google Maps utilisés par votre application. Vérifiez qu’Azure Maps fournit des alternatives sous forme de SDK et de services.
2. Si vous n’en avez pas encore, créez un abonnement Azure sur [https://azure.com](https://azure.com).
3. Créez un compte Azure Maps ([documentation](./how-to-manage-account-keys.md)) et une clé d’authentification ou Azure Active Directory ([documentation](./how-to-manage-authentication.md)).
4. Migrez votre code d’application.
5. Testez votre application migrée.
6. Déployez votre application migrée en production.

## <a name="create-an-azure-maps-account"></a>Créer un compte Azure Maps

Pour créer un compte Azure Maps et accéder à la plateforme Azure Maps, effectuez les étapes suivantes :

1. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
2. Connectez-vous au [portail Azure](https://portal.azure.com/).
3. Créez un [compte Azure Maps](./how-to-manage-account-keys.md). 
4. [Procurez-vous votre clé d’abonnement Azure Maps](./how-to-manage-authentication.md#view-authentication-details) ou configurez l’authentification Azure Active Directory pour renforcer la sécurité.

## <a name="azure-maps-technical-resources"></a>Ressources techniques Azure Maps

Voici une liste de ressources techniques utiles pour Azure Maps.

- Vue d’ensemble : [https://azure.com/maps](https://azure.com/maps)
- Documentation : [https://aka.ms/AzureMapsDocs](./index.yml)
- Exemples de code de kit de développement logiciel web : [https://aka.ms/AzureMapsSamples](https://aka.ms/AzureMapsSamples)
- Forums des développeurs : [https://aka.ms/AzureMapsForums](/answers/topics/azure-maps.html)
- Vidéos : [https://aka.ms/AzureMapsVideos](https://aka.ms/AzureMapsVideos)
- Blog : [https://aka.ms/AzureMapsBlog](https://aka.ms/AzureMapsBlog)
- Blog Tech Community : [https://aka.ms/AzureMapsTechBlog](https://aka.ms/AzureMapsTechBlog)
- Commentaires Azure Maps (UserVoice) : [https://aka.ms/AzureMapsFeedback](https://aka.ms/AzureMapsFeedback)
- [Azure Maps Jupyter Notebook](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook)

## <a name="migration-support"></a>Prise en charge de la migration

Les développeurs peuvent rechercher la prise en charge de la migration via les [forums](/answers/topics/azure-maps.html) ou via l’une des nombreuses options de support Azure : [https://azure.microsoft.com/support/options](https://azure.microsoft.com/support/options)

## <a name="next-steps"></a>Étapes suivantes

Découvrez en détail comment migrer votre application Google Maps avec les articles suivants :

> [!div class="nextstepaction"]
> [Migrer une application web](migrate-from-google-maps-web-app.md)

> [!div class="nextstepaction"]
> [Migrer un service web](migrate-from-google-maps-web-services.md) 

> [!div class="nextstepaction"]
> [Migrer une application Android](migrate-from-google-maps-android-app.md) 