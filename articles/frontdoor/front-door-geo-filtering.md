---
title: Géofiltrage sur un domaine Azure Front Door | Microsoft Docs
description: Dans cet article, vous en apprendrez plus sur la stratégie de géofiltrage Azure Front Door
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2020
ms.author: duau
ms.reviewer: tyao
ms.openlocfilehash: 42697a57d39f4a34eee4866b67e2cde947db1ff5
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91449254"
---
# <a name="geo-filtering-on-a-domain-for-azure-front-door"></a>Filtrage géographique sur un domaine pour Azure Front Door

Par défaut, Azure Front Door répond aux demandes de l’utilisateur, quel que soit l’emplacement d’origine de la demande. Dans certains scénarios, vous devrez peut-être limiter l’accès à votre applications web en fonction du pays/de la région. Le service de pare-feu d’applications web (WAF) au niveau de Front Door vous permet de définir une stratégie à l’aide de règles d’accès personnalisées pour un chemin d’accès spécifique à votre point de terminaison. Cette stratégie autorisera ou bloquera l’accès à partir de pays/régions spécifiés. 

Une stratégie WAF contient un ensemble de règles personnalisées. La règle se compose de conditions de correspondance, d’une action et d’une priorité. Dans une condition de correspondance, vous pourrez définir une variable de correspondance, un opérateur et une valeur de correspondance. Pour une règle de filtrage géographique, la variable de correspondance est REMOTE_ADDR, l’opérateur est GeoMatch, et la valeur correspond à un indicatif à deux lettres du pays/région d’intérêt. Vous pouvez combiner une condition GeoMatch et une condition de correspondance de chaîne REQUEST_URI pour créer une règle de filtrage géographique basé sur le chemin d’accès.

Vous pouvez configurer une stratégie de filtrage géographique pour votre porte d’entrée à l’aide d’[Azure PowerShell](front-door-tutorial-geo-filtering.md) ou de notre [modèle de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering).

## <a name="countryregion-code-reference"></a>Référence du code pays ou région

|Code pays ou région | Nom du pays/de la région |
| ----- | ----- |
| AD | Andorre |
| AE | Émirats Arabes Unis|
| AF | Afghanistan|
| Groupe de disponibilité | Antigua-et-Barbuda|
| AL | Albanie|
| AM | Arménie|
| AO | Angola|
| AR | Argentine|
| AS | Samoa américaines|
| AT | Autriche|
| AU | Australie|
| AZ | Azerbaïdjan|
| BA | Bosnie-Herzégovine|
| BB | Barbade|
| BD | Bangladesh|
| BE | Belgique|
| BF | Burkina Faso|
| BG | Bulgarie|
| BH | Bahreïn|
| BI | Burundi|
| BJ | Bénin|
| BL | Saint-Barthélemy|
| BN | Brunéi Darussalam|
| BO | Bolivie|
| BR | Brésil|
| BS | Les Bahamas|
| BT | Bhoutan|
| BW | Botswana|
| BY | Bélarus|
| BZ | Belize|
| CA | Canada|
| CD | République démocratique du Congo|
| CF | République centrafricaine|
| CH | Suisse|
| CI | Côte d'Ivoire|
| CL | Chili|
| CM | Cameroun|
| CN | Chine|
| CO | Colombie|
| CR | Costa Rica|
| CU | Cuba|
| CV | Cabo Verde|
| CY | Chypre|
| CZ | République tchèque|
| DE | Allemagne|
| DK | Danemark|
| DO | République dominicaine|
| DZ | Algérie|
| EC | Équateur|
| EE | Estonie|
| EG | Égypte|
| ES | Espagne|
| ET | Éthiopie|
| FI | Finlande|
| FJ | Fidji|
| FM | Micronésie, États fédérés de|
| FR | France|
| Go | Royaume-Uni|
| GE | Géorgie|
| GF | Guyane française|
| GH | Ghana|
| GN | Guinée|
| GP | Guadeloupe|
| GR | Grèce|
| GT | Guatemala|
| GY | Guyane|
| HK | Hong Kong (R.A.S.)|
| HN | Honduras|
| HR | Croatie|
| HT | Haïti|
| HU | Hongrie|
| id | Indonésie|
| IE | Irlande|
| IL | Israël|
| IN | Inde|
| IQ | Irak|
| IR | Iran, République islamique|
| IS | Islande|
| IT | Italie|
| JM | Jamaïque|
| JO | Jordanie|
| JP | Japon|
| KE | Kenya|
| KG | Kirghizistan|
| KH | Cambodge|
| KI | Kiribati|
| KN | Saint-Christophe-et-Niévès|
| KP | Corée, République populaire démocratique de|
| KR | Corée, République de|
| KW | Koweït|
| KY | Caïmans (îles)|
| KZ | Kazakhstan|
| LA | Laos, République démocratique|
| LB | Liban|
| LI | Liechtenstein|
| LK | Sri Lanka|
| LR | Liberia|
| LS | Lesotho|
| LT | Lituanie|
| LU | Luxembourg|
| LV | Lettonie|
| LY | Libye |
| MA | Maroc|
| MD | Moldova|
| MG | Madagascar|
| MK | Macédoine du Nord|
| ML | Mali|
| MM | Myanmar|
| MN | Mongolie|
| MO | Macao R.A.S.|
| MQ | Martinique|
| MR | Mauritanie|
| MT | Malte|
| MV | Maldives|
| MW | Malawi|
| MX | Mexique|
| MY | Malaisie|
| MZ | Mozambique|
| N/D | Namibie|
| NE | Niger|
| NG | Nigeria|
| NI | Nicaragua|
| NL | Pays-Bas|
| Non | Norvège|
| NP | Népal|
| NR | Nauru|
| NZ | Nouvelle-Zélande|
| OM | Oman|
| PA | Panama|
| PE | Pérou|
| PH | Philippines|
| PK | Pakistan|
| PL | Pologne|
| PR | Porto Rico|
| PT | Portugal|
| PW | Palau|
| PY | Paraguay|
| QA | Qatar|
| RE | La réunion|
| RO | Roumanie|
| RS | Serbie|
| RU | Fédération de Russie|
| L/E | Rwanda|
| SA | Arabie Saoudite|
| SD | Soudan|
| SE | Suède|
| SG | Singapour|
| SI | Slovénie|
| SK | Slovaquie|
| SN | Sénégal|
| SO | Somalie|
| SR | Surinam|
| SS | Soudan du Sud|
| SV | El Salvador|
| SY | République arabe syrienne|
| SZ | Swaziland|
| TC | Turques-et-Caïques (îles)|
| TG | Togo|
| MJ | Thaïlande|
| TN | Tunisie|
| TR | Turquie|
| TT | Trinité-et-Tobago|
| TW | Taïwan|
| TZ | Tanzanie, République de|
| UA | Ukraine|
| UG | Ouganda|
| US | États-Unis|
| UY | Uruguay|
| UZ | Ouzbékistan|
| VC | Saint-Vincent-et-les-Grenadines|
| VE | Venezuela|
| VG | Vierges britanniques (îles)|
| VI | Vierges américaines (îles)|
| VN | Vietnam|
| ZA | Afrique du Sud|
| ZM | Zambie|
| ZW | Zimbabwe|

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Apprenez à [configurer une stratégie WAF de géofiltrage](front-door-tutorial-geo-filtering.md).
