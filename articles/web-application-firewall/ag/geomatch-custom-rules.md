---
title: Règles personnalisées de géocorrespondance du pare-feu d’applications web (WAF) Azure
description: Cet article fournit une vue d’ensemble des règles personnalisées de géocorrespondance du pare-feu d’applications web (WAF) pour la passerelle Azure Application Gateway.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 01/31/2020
ms.author: victorh
ms.openlocfilehash: bdb115cf6d0f0aa2c55f6143015a323df6faa114
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83744298"
---
# <a name="geomatch-custom-rules-preview"></a>Règles personnalisées Geomatch (préversion)

Vous pouvez créer des règles personnalisées pour répondre aux besoins exacts de vos applications et stratégies de sécurité. À présent, vous pouvez restreindre l’accès à vos applications web par pays ou par région. Comme avec toutes les règles personnalisées, cette logique peut être composée d’autres règles pour répondre aux besoins de votre application.

Pour créer une règle personnalisée de filtrage géographique, sélectionnez simplement *Géolocalisation* comme type de correspondance, puis sélectionnez le pays/la région que vous souhaitez autoriser ou bloquer à partir de votre application. Pour plus d’informations, consultez [Comment créer des règles personnalisées dans PowerShell](configure-waf-custom-rules.md) et d’autres exemples de règles personnalisées (create-custom-waf-rules.md).

> [!IMPORTANT]
> Cette préversion publique est fournie sans contrat de niveau de service et ne doit pas être utilisée pour les charges de travail de production. Certaines fonctionnalités peuvent ne pas être prises en charge, disposer de capacités limitées ou ne pas être disponibles dans tous les emplacements Azure. Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="countryregion-codes"></a>Codes pays ou région

Si vous utilisez l’opérateur Geomatch, les sélecteurs peuvent correspondre à l’un des codes pays/région à deux chiffres suivants. 

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

Maintenant que vous en savez plus sur les règles personnalisées, [créez vos propres règles personnalisées](create-custom-waf-rules.md).
