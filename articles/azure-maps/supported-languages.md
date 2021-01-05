---
title: Prise en charge de la localisation avec Microsoft Azure Maps
description: Découvrez les régions prises en charge par Azure Maps avec les services tels que les cartes, la recherche, le routage, la météo et les incidents de trafic. Découvrez comment configurer le paramètre Affichage.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 50e5d0721eb14d1fcdfad26aaf081bfa370e954e
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904514"
---
# <a name="localization-support-in-azure-maps"></a>Prise en charge de la localisation dans Azure Maps

Azure Maps prend en charge différentes langues et différents affichages basés sur le pays ou la région. Cet article présente les langues et affichages pris en charge pour vous aider à implémenter Azure Maps.


## <a name="azure-maps-supported-languages"></a>Langues prises en charge dans Azure Maps

Les services Azure Maps ont été localisés en différentes langues. Le tableau suivant indique les codes des langues prises en charge pour chaque service.  
  

| id         | Nom                   |  Cartes | Recherche | Routage | Météo | Incidents de trafic | Commandes de la carte JS |
|------------|------------------------|:-----:|:------:|:-------:|:--------:|:-----------------:|:--------------:|
| af-ZA      | Afrikaans              |       |    ✓   |    ✓    |         |                   |                |
| ar-SA      | Arabe                 |   ✓   |    ✓   |    ✓    |    ✓      |         ✓         |        ✓       |
| bn-BD      | Bengali (Bangladesh)    |       |       |         |     ✓    |                   |                |
| bn-IN      | Bengali (Inde)         |       |       |         |     ✓    |                   |                |
| bs-BA      | Bosniaque                 |       |       |         |     ✓    |                   |                |
| eu-ES      | Basque                 |       |    ✓   |         |         |                   |                |
| bg-BG      | Bulgare              |   ✓   |    ✓   |    ✓    |     ✓     |                   |        ✓       |
| ca-ES      | Catalan                |       |    ✓   |         |    ✓      |                   |                |
| zh-hans    | Chinois (simplifié)   |       |  zh-CN |         |     zh-CN   |                   |                |
| zh-hant    | Chinois (Hong Kong R.A.S.)  |  |   |    |    zh-HK   |                   |           |
| zh-hant    | Chinois (Taïwan)  | zh-TW |  zh-TW |  zh-TW  |    zh-TW   |                   |      zh-TW     |
| hr-HR      | Croate               |       |    ✓   |         |    ✓      |                   |                |
| cs-CZ      | Tchèque                  |   ✓   |    ✓   |    ✓    |    ✓      |         ✓         |        ✓       |
| da-DK      | Danois                 |   ✓   |    ✓   |    ✓    |     ✓     |         ✓         |        ✓       |
| nl-BE      | Néerlandais (Belgique)        |       |    ✓   |         |      ✓    |                   |                |
| nl-NL      | Néerlandais (Pays-Bas)    |   ✓   |    ✓   |    ✓    |     ✓     |         ✓         |        ✓       |
| en-AU      | Anglais (Australie)    |   ✓   |    ✓   |    ✓    |     ✓     |         ✓         |        ✓       |
| en-NZ      | Anglais (Nouvelle-Zélande)  |   ✓   |    ✓   |    ✓    |     ✓     |         ✓         |        ✓       |
| en-GB      | Anglais (Grande-Bretagne) |   ✓   |    ✓   |    ✓    |     ✓     |         ✓         |        ✓       |
| fr-FR      | Anglais (États-Unis)          |   ✓   |    ✓   |    ✓    |      ✓    |         ✓         |        ✓       |
| Et-EE      | Estonien               |       |    ✓   |         |      ✓    |         ✓         |                |
| fil-PH     | Filipino               |       |       |         |     ✓    |                   |                |
| fi-FI      | Finnois                |   ✓   |    ✓   |    ✓    |      ✓    |         ✓         |        ✓       |
| fr-FR      | Français                 |   ✓   |    ✓   |    ✓    |      ✓    |         ✓         |        ✓       |
| fr-CA      | Français (Canada)      |       |    ✓   |         |     ✓     |                   |                |
| gl-ES      | Galicien               |       |    ✓   |         |         |                   |                |
| de-DE      | Allemand                 |   ✓   |    ✓   |    ✓    |   ✓      |         ✓         |        ✓       |
| el-GR      | Grec                  |   ✓   |    ✓   |    ✓    |    ✓     |         ✓         |        ✓       |
| gu-IN      | Goudjrati                |       |       |         |     ✓    |                   |                |
| he-IL      | Hébreu                 |       |    ✓   |         |     ✓    |         ✓         |                |
| hi-IN      | Hindi                  |       |        |         |     ✓    |                   |                |
| hu-HU      | Hongrois              |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| is-IS      | Islandais              |       |       |         |     ✓    |                   |                |
| id-ID      | Indonésien             |   ✓   |    ✓    |    ✓    |     ✓    |         ✓         |        ✓       |
| it-IT      | Italien                |   ✓   |    ✓   |    ✓    |      ✓   |         ✓         |        ✓       |
| ja-JP      | Japonais               |       |        |         |     ✓    |                   |                |
| kn-IN      | Kannada                |       |       |         |     ✓    |                   |                |
| kk-KZ      | Kazakh                 |       |    ✓   |         |     ✓    |                   |                |
| ko-KR      | Coréen                 |   ✓   |        |    ✓    |     ✓    |                   |        ✓       |
| es-419     | Espagnol (Amérique latine) |       |    ✓   |         |         |                   |                |
| lv-LV      | Letton                |       |    ✓   |         |     ✓    |         ✓         |                |
| lt-LT      | Lituanien             |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| mk-MK      | Macédonien             |       |       |         |     ✓    |                   |                |
| ms-MY      | Malais (latin)          |   ✓   |    ✓   |    ✓    |    ✓   |                   |        ✓       |
| mr-IN      | Marathi                 |       |       |         |     ✓    |                   |                |
| nb-NO      | Norwegian Bokmål       |   ✓   |    ✓   |    ✓    |      ✓   |         ✓         |        ✓       |
| NGT        | Neutral Ground Truth : langues officielles pour toutes les régions dans les scripts locaux s’ils sont disponibles |   ✓     |        |         |       |        |      ✓          |
| NGT-Latn   | Neutral Ground Truth : exonymes du Latin. Le script Latin sera utilisé s’il est disponible |   ✓     |        |         |         |                |        ✓         |
| pl-PL      | Polonais                 |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| pt-br      | Portugais (Brésil)    |   ✓   |    ✓   |    ✓    |      ✓   |                   |        ✓       |
| pt-PT      | Portugais (Portugal)  |   ✓   |    ✓   |    ✓    |      ✓   |         ✓         |        ✓       |
| pa-IN      | Pendjabi                 |       |       |         |     ✓    |                   |                |
| ro-RO      | Roumain               |       |    ✓    |         |     ✓    |         ✓         |                |
| ru-RU      | Russe                |   ✓   |    ✓   |    ✓    |      ✓   |         ✓         |        ✓       |
| sr-Cyrl-RS | Serbe (cyrillique)     |       |   sr-RS  |         |    sr-RS     |                   |                |
| sr-Latn-RS | Serbe (latin)        |       |       |         |     sr-latn    |                   |                |
| sk-SK      | Slovaque             |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| sl-SL      | Slovène              |   ✓   |    ✓   |    ✓    |     ✓    |                   |        ✓       |
| es-ES      | Espagnol                |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| es-MX      | Espagnol (Mexique)       |   ✓   |        |    ✓    |     ✓    |                   |        ✓       |
| sv-SE      | Suédois                |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| ta-IN      | Tamoul (Inde)                 |       |       |         |     ✓    |                   |                |
| te-IN      | Télougou (Inde)                 |       |       |         |     ✓    |                   |                |
| th-TH      | Thaï                   |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| tr-TR      | Turc                |   ✓   |    ✓   |    ✓    |     ✓    |         ✓         |        ✓       |
| uk-UA      | Ukrainien               |       |    ✓   |         |     ✓    |                   |                |
| ur-PK      | Ourdou                 |       |       |         |     ✓    |                   |                |
| uz-Latn-UZ | Ouzbek                 |       |       |         |     ✓    |                   |                |
| vi-VN      | Vietnamien             |       |    ✓   |         |      ✓    |                  |                |


## <a name="azure-maps-supported-views"></a>Affichages pris en charge dans Azure Maps

> [!Note]
> Azure Maps a été lancé le 1er août 2019 dans les pays/régions suivants :
>  * Argentine
>  * Inde
>  * Maroc
>  * Pakistan
>
> À compter du 1er août 2019, le paramètre **Vue** définira le contenu cartographique retourné pour les régions/pays supplémentaires ci-dessus. Le paramètre **Vue** d’Azure Maps (ou « paramètre régional de l’utilisateur ») est un code de pays ISO-3166 à deux lettres qui est utilisé pour afficher les cartes de ce pays ou cette région, spécifiant quel contenu géopolitique sensible est retourné par les services Azure Maps, y compris les frontières et les étiquettes affichées sur les cartes. 

Assurez-vous d’avoir configuré le paramètre **Vue** de façon appropriée pour les API REST et kits de développement logiciel (SDK) utilisés par vos services.
  

### <a name="rest-apis"></a>API Rest
  
Vérifiez que vous avez configuré le paramètre Vue comme il convient. Le paramètre Vue spécifie quel contenu géopolitique sensible est retourné par les services Azure Maps. 

API REST Azure Maps concernées :
    
 * Get Map Tile
 * Get Map Image 
 * Get Search Fuzzy
 * Get Search POI
 * Get Search POI Category
 * Get Search Nearby
 * Get Search Address
 * Get Search Address Structured
 * Get Search Address Reverse
 * Get Search Address Reverse Cross Street
 * Post Search Inside Geometry
 * Post Search Address Batch
 * Post Search Address Reverse Batch
 * Post Search Along Route
 * Post Search Fuzzy Batch

 
### <a name="sdks"></a>Kits SDK

Assurez-vous d’avoir configuré le paramètre **Vue** comme il convient et d’utiliser les dernières versions de Web SDK et Android SDK. SDK concernés :

 * Kit de développement logiciel (SDK) web Azure Maps
 * Android SDK Azure Maps

Par défaut, le paramètre Vue est défini sur **Unifié**, même si vous n’avez pas défini ce paramètre dans la requête. Déterminez l'emplacement de vos utilisateurs. Définissez ensuite le paramètre **Vue** correctement pour cet emplacement. Vous pouvez aussi définir le paramètre d’affichage « Vue=Auto », qui retourne les données cartographiques correspondant à l’adresse IP de la requête.  Le paramètre **Vue** dans Azure Maps doit être utilisé dans le respect des lois en vigueur, y compris celles relatives à la cartographie du pays/région où les cartes, images et autres données et contenus tiers auxquels vous êtes autorisé à accéder via Azure Maps sont mis à disposition.


Le tableau suivant présente les affichages pris en charge.

| Affichage         | Description                            |  Cartes | Recherche | Contrôle de carte JS |
|--------------|----------------------------------------|:-----:|:------:|:--------------:|
| AE           | Émirats Arabes Unis (affichage arabe)    |   ✓   |        |     ✓          |
| AR           | Argentine (affichage argentin)           |   ✓   |    ✓   |     ✓          |
| BH           | Bahreïn (affichage arabe)                 |   ✓   |        |     ✓          |
| IN           | Inde (affichage indien)                    |   ✓   |   ✓     |     ✓          |
| IQ           | Irak (affichage arabe)                    |   ✓   |        |     ✓          |
| JO           | Jordanie (affichage arabe)                  |   ✓   |        |     ✓          |
| KW           | Koweït (affichage arabe)                  |   ✓   |        |     ✓          |
| LB           | Liban (affichage arabe)                 |   ✓   |        |     ✓          |
| MA           | Maroc (affichage marocain)                |   ✓   |   ✓     |     ✓          |
| OM           | Oman (affichage arabe)                    |   ✓   |        |     ✓          |
| PK           | Pakistan (affichage pakistanais)              |   ✓   |    ✓    |     ✓          |
| PS           | Autorité palestinienne (affichage arabe)    |   ✓   |        |     ✓          |
| QA           | Qatar (affichage arabe)                   |   ✓   |        |     ✓          |
| SA           | Arabie Saoudite (affichage arabe)            |   ✓   |        |     ✓          |
| SY           | Syrie (affichage arabe)                   |   ✓   |        |     ✓          |
| YE           | Yémen (affichage arabe)                   |   ✓   |        |     ✓          |
| Auto         | Retourne les données cartographiques correspondant à l’adresse IP de la requête.|   ✓   |    ✓   |     ✓          |
| Unifié      | Affichage unifié (Autres)                  |   ✓   |   ✓     |     ✓          |
