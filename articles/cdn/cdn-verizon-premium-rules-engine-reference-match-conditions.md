---
title: Conditions de correspondance du moteur de règles Azure CDN de Verizon Premium
description: Documentation de référence des conditions de correspondance du moteur de règles Azure Content Delivery Network de Verizon Premium.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: efd6e6a93cd4ca79e6c4b6de69f8514e2d71b252
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84323312"
---
# <a name="azure-cdn-from-verizon-premium-rules-engine-match-conditions"></a>Conditions de correspondance du moteur de règles Azure CDN de Verizon Premium

Cet article fournit les descriptions détaillées des conditions de correspondance disponibles pour le [moteur de règles](cdn-verizon-premium-rules-engine.md) Azure Content Delivery Network (CDN) de Verizon Premium.

La deuxième partie d’une règle est la condition de correspondance. Une condition de correspondance identifie des types spécifiques de requêtes pour lesquelles un ensemble de fonctionnalités est exécuté.

Par exemple, vous pouvez utiliser une condition de correspondance pour les tâches suivantes :

- Filtrer les requêtes ciblant le contenu dans un emplacement particulier.
- Filtrer les requêtes générées à partir d’une adresse IP ou d’un pays/d'une région en particulier.
- Filtrer les requêtes d’après les informations d’en-tête.

## <a name="match-conditions"></a><a name="top"></a>Conditions de correspondance

* [Toujours](#always)
* [Appareil](#device)
* [Lieu](#location)
* [Origine](#origin)
* [Requête](#request)
* [URL](#url)

### <a name="always"></a><a name="always"></a>Toujours

[La condition de correspondance Toujours](https://docs.vdms.com/cdn/Content/HRE/M/Always.htm) est conçue pour appliquer un ensemble de fonctionnalités par défaut à toutes les demandes.

### <a name="device"></a><a name="device"></a>Appareil

Ces conditions de correspondance sont conçues pour identifier les demandes selon l’agent utilisateur du client.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| Nom de la marque | Identifie les demandes en déterminant si le nom de la marque de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du nom de la marque](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Literal.htm)) <br> **-** Expression régulière ([expression régulière du nom de la marque](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du nom de la marque](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Wildcard.htm)) |
| Système d’exploitation de l’appareil | Identifie les demandes en déterminant si le système d’exploitation de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Literal.htm)) <br> **-** Expression régulière ([expression régulière du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Wildcard.htm)) |
| Version du système d’exploitation de l’appareil | Identifie les demandes en déterminant si la version du système d’exploitation de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de la version du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Literal.htm)) <br> **-** Expression régulière ([expression régulière de la version du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques de la version du système d’exploitation de l’appareil](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Wildcard.htm)) |
| [Orientation double ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Dual-Orientation.htm) | Identifie les demandes en déterminant si l’appareil prend en charge l’orientation double. |
| DTD HTML par défaut | Identifie les demandes en déterminant si le DTD HTML par défaut de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du DTD HTML par défaut](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Literal.htm)) <br> **-** Expression régulière ([expression régulière du DTD HTML par défaut](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du DTD HTML par défaut](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Wildcard.htm)) |
| [Incorporation d’images ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Image-Inlining.htm) | Identifie les demandes en déterminant si l’appareil prend en les images codées en Base64. |
| [Android ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Android.htm) | Identifie les demandes en déterminant si l’appareil utilise le système d’exploitation Android. |
| [Application ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-App.htm) | Identifie les demandes en déterminant si une application native a demandé du contenu. |
| [Bureau complet ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Full-Desktop.htm) | Identifie les demandes en déterminant si l’appareil fournit une expérience de bureau complète. |
| [iOS ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-iOS.htm) | Identifie les demandes en déterminant si l’appareil utilise le système d’exploitation iOS. |
| [Robot ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Robot.htm) | Identifie les demandes en déterminant si l’appareil est considéré comme étant un client HTTP automatisé (par exemple, un robot). |
| [Télévision connectée ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Smart-TV.htm) | Identifie les demandes en déterminant si l’appareil est une télévision connectée. |
| [Smartphone ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Smartphone.htm) | Identifie les demandes en déterminant si l’appareil est un smartphone.
| [Tablette ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Tablet.htm) | Identifie les demandes en déterminant si l’appareil est une tablette. |
| [Écran tactile ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Touchscreen.htm) | Identifie les demandes en déterminant si le périphérique de pointage principal de l’appareil est un écran tactile. |
| [Windows Phone ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Windows-Phone.htm) | Identifie les demandes en déterminant si l’appareil est de type Windows Mobile 6.5 ou Windows Phone 7 ou version ultérieure. |
| [Appareil sans fil ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Wireless-Device.htm) | Identifie les demandes en déterminant si l’appareil est sans fil. 
| Nom marketing | Identifie les demandes en déterminant si le nom marketing de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral fu nom marketing](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Literal.htm)) <br> **-** Expression régulière ([expression régulière du nom marketing](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du nom marketing](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Wildcard.htm)) |
| Navigateur mobile | Identifie les demandes en déterminant si le navigateur de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Literal.htm)) <br> **-** Expression régulière ([expression régulière du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Wildcard.htm)) |
| Version du navigateur mobile | Identifie les demandes en déterminant si la version du navigateur de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de la version du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Literal.htm)) <br> **-** Expression régulière ([expression régulière de la version du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques de la version du navigateur mobile](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Wildcard.htm)) |
| Nom du modèle | Identifie les demandes en déterminant si le nom de modèle de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du nom de modèle](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Literal.htm)) <br> **-** Expression régulière ([expression régulière du nom de modèle](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du nom de modèle](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Wildcard.htm)) |
| [Téléchargement progressif ?](https://docs.vdms.com/cdn/Content/HRE/M/D-Progressive-Download.htm) | Identifie les demandes en déterminant si l’appareil prend en charge le téléchargement progressif. |
| Date de sortie | Identifie les demandes en déterminant si la date de mise en production de l’appareil correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de date de mise en production](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Literal.htm)) <br> **-** Expression régulière ([expression régulière de la date de mise en production](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques de la date de mise en production](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Wildcard.htm)) |
| [Hauteur de résolution](https://docs.vdms.com/cdn/Content/HRE/M/D-Resolution-Height.htm) | Identifie les demandes à la hauteur de l’appareil. |
| [Largeur de résolution](https://docs.vdms.com/cdn/Content/HRE/M/D-Resolution-Width.htm) | Identifie les demandes à la largeur de l’appareil. |

**[Retour au début](#top)**

### <a name="location"></a><a name="location"></a>Emplacement

Ces conditions de correspondance sont conçues pour identifier les requêtes selon l’emplacement du demandeur.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Numéro AS](https://docs.vdms.com/cdn/Content/HRE/M/AS-Number.htm) | Identifie les requêtes issues d’un réseau particulier. |
| Nom de la ville | Identifie les demandes en déterminant si elles proviennent d’une ville dont le nom correspond aux critères suivants : <br> **-** Valeur spécifique ([littéral du nom de la ville](https://docs.vdms.com/cdn/Content/HRE/M/City-Name-Literal.htm)) <br> **-** Expression régulière ([expression régulière du nom de la ville](https://docs.vdms.com/cdn/Content/HRE/M/City-Name-Regex.htm)) |
| [Continent](https://docs.vdms.com/cdn/Content/HRE/M/Continent.htm) | Identifie les demandes provenant des continents spécifiés. |
| [Pays](https://docs.vdms.com/cdn/Content/HRE/M/Country.htm) | Identifie les demandes provenant des pays spécifiés. |
| [Code DMA](https://docs.vdms.com/cdn/Content/HRE/M/DMA-Code.htm) | Identifie les demandes provenant des zones métropolitaines (DMA) spécifiées. |
| [Latitude](https://docs.vdms.com/cdn/Content/HRE/M/Latitude.htm) | Identifie les demandes provenant des latitudes spécifiées. |
| [Longitude](https://docs.vdms.com/cdn/Content/HRE/M/Longitude.htm) | Identifie les demandes provenant des longitudes spécifiées. |
| [Code Métropolitain](https://docs.vdms.com/cdn/Content/HRE/M/Metro-Code.htm) | Identifie les demandes provenant des zones métropolitaines (DMA) spécifiées. |
| [Postal Code](https://docs.vdms.com/cdn/Content/HRE/M/Postal-Code.htm) | Identifie les demandes provenant des codes postaux spécifiés. |
| [Code de la région](https://docs.vdms.com/cdn/Content/HRE/M/Region-Code.htm) | Identifie les demandes provenant des régions spécifiées. |

> [!NOTE]
> **Dois-je utiliser le code métropolitain ou le code DMA ?** <br>
Ces deux conditions de correspondance fournissent la même fonctionnalité. Toutefois, nous vous recommandons d’utiliser la condition de correspondance Code Métropolitain pour identifier les demandes par DMA.

**[Retour au début](#top)**

### <a name="origin"></a><a name="origin"></a>Origine

Ces conditions de correspondance sont conçues pour identifier les requêtes qui pointent vers le stockage CDN ou le serveur d’origine d’un client.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Origine CDN](https://docs.vdms.com/cdn/Content/HRE/M/CDN-Origin.htm) | Identifie les requêtes de contenu stocké sur le stockage CDN. |
| [Origine client](https://docs.vdms.com/cdn/Content/HRE/M/Customer-Origin.htm) | Identifie les requêtes de contenu stocké sur le serveur d’origine d’un client spécifique. |

**[Retour au début](#top)**

### <a name="request"></a><a name="request"></a>Requête

Ces conditions de correspondance sont conçues pour identifier les requêtes selon leurs propriétés.

| Nom              | Objectif                                                                |
|-------------------|------------------------------------------------------------------------|
| [Adresse IP du client](https://docs.vdms.com/cdn/Content/HRE/M/Client-IP-Address.htm) | Identifie les requêtes issues d’une adresse IP particulière. |
| Paramètre de cookie  | Identifie une demande en déterminant si elle contient un cookie correspondant à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du paramètre de cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Literal.htm)) <br> **-** Expression régulière ([expression régulière du paramètre de cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du paramètre de cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Wildcard.htm)) |
| [CNAME de périphérie](https://docs.vdms.com/cdn/Content/HRE/M/Edge-CNAME.htm) | Identifie les requêtes qui pointent vers un CNAME de périphérie spécifique. |
| Domaine de référence | Identifie une demande en déterminant si elle a été référée par un nom d’hôte correspondant à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du domaine de référence](https://docs.vdms.com/cdn/Content/HRE/M/Referring-Domain-Literal.htm)) <br> **-** Modèle spécifique ([caractères génériques du domaine de référence](https://docs.vdms.com/cdn/Content/HRE/M/Referring-Domain-Wildcard.htm)) |
| En-tête de requête | Identifie une demande en déterminant si elle contient un en-tête correspondant à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de l’en-tête de la demande](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Literal.htm)) <br> **-** Expression régulière ([expression régulière de l’en-tête de la demande](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques de l’en-tête de la demande](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Wildcard.htm)) |
| [Méthode de requête](https://docs.vdms.com/cdn/Content/HRE/M/Request-Method.htm) | Identifie les requêtes par leur méthode HTTP. |
| [Modèle de requête](https://docs.vdms.com/cdn/Content/HRE/M/Request-Scheme.htm) | Identifie les requêtes par leur protocole HTTP. |

**[Retour au début](#top)**

### <a name="url"></a><a name="url"></a>URL

| Nom              | Objectif                                                                |
|-------------------|------------------------------------------------------------------------|
| Chemin d’URL | Identifie les demandes en déterminant si leur chemin d’accès relatif, y compris le nom de fichier, correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Literal.htm)) <br> **-** Expression régulière ([expression régulière du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Wildcard.htm)) |
| Répertoire du chemin d’URL | Identifie les demandes en déterminant si leur chemin d’accès relatif correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du répertoire du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Directory-Literal.htm)) <br> **-** Modèle spécifique ([caractères génériques du répertoire du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Directory-Wildcard.htm)) |
| Extension du chemin d’URL | Identifie les demandes en déterminant si leur extension de fichier correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de l’extension du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Extension-Literal.htm)) <br> **-** Modèle spécifique ([caractères génériques de l’extension du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Extension-Wildcard.htm)) |
| Nom de fichier du chemin d’URL | Identifie les demandes en déterminant si leur nom de fichier correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du nom de fichier du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Filename-Literal.htm)) <br> **-** Modèle spécifique ([caractères génériques du nom de fichier du chemin d’accès de l’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Filename-Wildcard.htm)) |
| Requête d’URL | Identifie les demandes en déterminant si leur chaîne de requête correspond à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral de la requête d’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Literal.htm)) <br> **-** Expression régulière ([expression régulière de la requête d’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Regex.htm)) <br> **-** Modèle spécifique ([caractères génériques de la requête d’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Wildcard.htm)) |
| Paramètre de requête d’URL | Identifie les demandes en déterminant si elles contiennent un paramètre de chaîne de requête défini sur une valeur correspondant à l’un des critères suivants : <br> **-** Valeur spécifique ([littéral du paramètre de requête d’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Parameter-Literal.htm)) <br> **-** Modèle spécifique ([caractères génériques du paramètre de requête d’URL](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Parameter-Wildcard.htm)) |

**[Retour au début](#top)**

Pour connaître les dernières conditions de correspondance, consultez la [Documentation du moteur de règles Verizon](https://docs.vdms.com/cdn/index.html#Quick_References/HRE_QR.htm#Conditio).

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Content Delivery Network](cdn-overview.md)
- [Informations de référence du moteur de règles](cdn-verizon-premium-rules-engine-reference.md)
- [Expressions conditionnelles du moteur de règles](cdn-verizon-premium-rules-engine-reference-conditional-expressions.md)
- [Fonctionnalités du moteur de règles](cdn-verizon-premium-rules-engine-reference-features.md)
- [Remplacement du comportement HTTP par défaut à l’aide du moteur de règles](cdn-verizon-premium-rules-engine.md)
