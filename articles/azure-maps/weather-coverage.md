---
title: Couverture du service Météo (préversion) de Microsoft Azure Maps
description: Découvrez le concept de couverture dans le service Météo (préversion) de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
ms.custom: references_regions
manager: philmea
ms.openlocfilehash: 6c4e9eb765a72b7a0b495f81a954b484ef6aa2b7
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905483"
---
# <a name="azure-maps-weather-services-preview-coverage"></a>Couverture du service Météo Azure Maps (préversion)

> [!IMPORTANT]
> Le service Météo d’Azure Maps est disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Cet article fournit des informations sur les [services météo](/rest/api/maps/weather) Azure Maps. Les services de données météorologiques Azure Maps retournent des détails tels que les vignettes radar, les conditions météorologiques actuelles, les prévisions météorologiques et la météo le long d’un itinéraire.

Azure Maps n’a pas le même niveau d’information et de précision pour tous les pays et régions.

Le tableau suivant fournit des informations sur les types de données météorologiques que vous pouvez solliciter pour chaque pays/région.

| Symbole | Signification |
|--------|---------|
|*       |Couvre la météo le long de l’itinéraire, les conditions actuelles, les prévisions horaires, les prévisions au quart de journée, les indices quotidiens et les API de prévisions quotidiennes. |


## <a name="americas"></a>Amérique

| Pays/région              |  Vignette Satellite | Prévision à la minute, vignettes radar | Alertes météorologiques graves | Autre* |
|-----------------------------|:----------------:|:-----------------:|:-----------------:|:--------:|  
| Anguilla                                 | ✓ |   | |  ✓| 
| Antarctique                               | ✓ |   | |✓|
| Antigua-et-Barbuda                      | ✓ |   | |✓| 
| Argentine                                | ✓ |   | |✓| 
| Aruba                                    | ✓ |   | |✓| 
| Les Bahamas                                  | ✓ |   | |✓| 
| Barbade                                 | ✓ |   | |✓| 
| Belize                                   | ✓ |   | |✓| 
| Bermudes                                  | ✓ |   | |✓| 
| Bolivie                                  | ✓ |   | |✓| 
| Bonaire                                  | ✓ |   | |✓| 
| Brésil                                   | ✓ |   | ✓ |✓| 
| Îles Vierges britanniques                   | ✓ |   | |✓| 
| Canada                                   | ✓ | ✓ | ✓ | ✓| 
| Caïmans (îles)                           | ✓ |   | |✓| 
| Chili                                    | ✓ |   | |✓| 
| Colombie                                 | ✓ |   | |✓| 
| Costa Rica                               | ✓ |   | |✓| 
| Cuba                                     | ✓ |   | |✓| 
| Curaçao                                  | ✓ |   | |✓| 
| Dominique                                 | ✓ |   | |✓| 
| République dominicaine                       | ✓ |   | |✓| 
| Équateur                                  | ✓ |   | |✓| 
| El Salvador                              | ✓ |   | |✓| 
| Îles Malouines                         | ✓ |   | |✓| 
| Guyane française                            | ✓ |   | |✓| 
| Groenland                                | ✓ |   | |✓| 
| Grenade                                  | ✓ |   | |✓| 
| Guadeloupe                               | ✓ |   | |✓| 
| Guatemala                                | ✓ |   | |✓| 
| Guyane                                   | ✓ |   | |✓| 
| Haïti                                    | ✓ |   | |✓| 
| Honduras                                 | ✓ |   | |✓| 
| Jamaïque                                  | ✓ |   | |✓| 
| Martinique                               | ✓ |   | |✓| 
| Mexique                                   | ✓ |   | |✓| 
| Montserrat                               | ✓ |   | |✓| 
| Nicaragua                                | ✓ |   | |✓| 
| Panama                                   | ✓ |   | |✓| 
| Paraguay                                 | ✓ |   | |✓| 
| Pérou                                     | ✓ |   | |✓| 
| Porto Rico                              | ✓ |   | ✓ |✓| 
| Saint-Barthélemy                         | ✓ |   | |✓| 
| Saint-Christophe-et-Niévès                    | ✓ |   | |✓| 
| Sainte-Lucie                              | ✓ |   | |✓| 
| Saint-Martin                             | ✓ |   | |✓| 
| Saint Pierre et Miquelon                | ✓ |   | |✓| 
| Saint-Vincent-et-les-Grenadines         | ✓ |   | |✓| 
| Saint-Eustache                           | ✓ |   | |✓|  
| Saint-Martin (partie néerlandaise)                             | ✓ |   | |✓| 
| Géorgie du Sud et Sandwich du Sud (îles) | ✓ |   | |✓| 
| Surinam                                 | ✓ |   | |✓| 
| Trinité-et-Tobago                      | ✓ |   | |✓| 
| Turques-et-Caïques (îles)                 | ✓ |   | |✓| 
| États-Unis Îles éloignées                    | ✓ |   | |✓| 
| États-Unis Îles Vierges                      | ✓ |   | ✓|✓| 
| États-Unis                            | ✓ | ✓ | ✓| ✓| 
| Uruguay                                  | ✓ |   | |✓| 
| Venezuela                                | ✓ |   | |✓| 


## <a name="middle-east-and-africa"></a>Moyen-Orient et Afrique

| Pays/région              |  Vignette Satellite | Prévision à la minute, vignettes radar | Alertes météorologiques graves | Autre* | 
|-----------------------------|:----------------:|:-----------------:|:--------:|:--------:| 
| Algérie                     | ✓               |                              |     |  ✓| 
| Angola                      | ✓               |                              |     |  ✓| 
| Bahreïn                     | ✓               |                              |     |  ✓| 
| Bénin                       | ✓               |                              |     |  ✓| 
| Botswana                    | ✓               |                              |     |  ✓| 
| Bouvet (île)               | ✓               |                              |     |  ✓| 
| Burkina Faso                | ✓               |                              |     |  ✓| 
| Burundi                     | ✓               |                              |     |  ✓| 
| Cameroun                    | ✓               |                              |     |  ✓| 
| Cabo Verde                  | ✓               |                              |     |  ✓| 
| République centrafricaine    | ✓               |                              |     |  ✓| 
| Tchad                        | ✓               |                              |     |  ✓| 
| Comores                     | ✓               |                              |     |  ✓| 
| Congo (RDC)                 | ✓               |                              |     |  ✓|
| Côte d’Ivoire               | ✓               |                              |     |  ✓| 
| Djibouti                    | ✓               |                              |     |  ✓| 
| Égypte                       | ✓               |                              |     |  ✓| 
| Guinée équatoriale           | ✓               |                              |     |  ✓| 
| Érythrée                     | ✓               |                              |     |  ✓| 
| eSwatini                    | ✓               |                              |     |  ✓| 
| Éthiopie                    | ✓               |                              |     |  ✓| 
| Terres australes françaises | ✓               |                              |     |  ✓| 
| Gabon                       | ✓               |                              |     |  ✓| 
| Gambie                      | ✓               |                              |     |  ✓| 
| Ghana                       | ✓               |                              |     |  ✓| 
| Guinée                      | ✓               |                              |     |  ✓| 
| Guinée-Bissau               | ✓               |                              |     |  ✓| 
| Iran                        | ✓               |                              |     |  ✓| 
| Irak                        | ✓               |                              |     |  ✓| 
| Israël                      | ✓               |                              |   ✓   |  ✓| 
| Jordanie                      | ✓               |                              |     |  ✓| 
| Kenya                       | ✓               |                              |     |  ✓| 
| Koweït                      | ✓               |                              |     |  ✓| 
| Liban                     | ✓               |                              |     |  ✓| 
| Lesotho                     | ✓               |                              |     |  ✓| 
| Liberia                     | ✓               |                              |     |  ✓| 
| Libye                       | ✓               |                              |     |  ✓| 
| Madagascar                  | ✓               |                              |     |  ✓| 
| Malawi                      | ✓               |                              |     |  ✓| 
| Mali                        | ✓               |                              |     |  ✓| 
| Mauritanie                  | ✓               |                              |     |  ✓| 
| Maurice (île)                   | ✓               |                              |     |  ✓| 
| Mayotte                     | ✓               |                              |     |  ✓| 
| Maroc                     | ✓               |                              |     |  ✓| 
| Mozambique                  | ✓               |                              |     |  ✓| 
| Namibie                     | ✓               |                              |     |  ✓| 
| Niger                       | ✓               |                              |     |  ✓| 
| Nigeria                     | ✓               |                              |     |  ✓| 
| Oman                        | ✓               |                              |     |  ✓| 
| Autorité palestinienne       | ✓               |                              |     |  ✓| 
| Qatar                       | ✓               |                              |     |  ✓| 
| La Réunion                     | ✓               |                              |     |  ✓| 
| Rwanda                      | ✓               |                              |     |  ✓| 
| Sainte-Hélène, Ascension et Tristan da Cunha        | ✓               |            |     |  ✓| 
| São Tomé et Príncipe       | ✓               |                              |     |  ✓| 
| Arabie Saoudite                | ✓               |                              |     |  ✓| 
| Sénégal                     | ✓               |                              |     |  ✓| 
| Seychelles                  | ✓               |                              |     |  ✓| 
| Sierra Leone                | ✓               |                              |     |  ✓| 
| Somalie                     | ✓               |                              |     |  ✓| 
| Afrique du Sud                | ✓               |                              |     |  ✓| 
| Soudan du Sud                 | ✓               |                              |     |  ✓| 
| Soudan                       | ✓               |                              |     |  ✓| 
| Syrie                       | ✓               |                              |     |  ✓| 
| Tanzanie                    | ✓               |                              |     |  ✓| 
| Togo                        | ✓               |                              |     |  ✓| 
| Tunisie                     | ✓               |                              |     |  ✓| 
| Ouganda                      | ✓               |                              |     |  ✓| 
| Émirats Arabes Unis        | ✓               |                              |     |  ✓| 
| Yémen                       | ✓               |                              |     |  ✓| 
| Zambie                      | ✓               |                              |     |  ✓| 
| Zimbabwe                    | ✓               |                              |     |  ✓| 


## <a name="asia-pacific"></a>Asie-Pacifique

| Pays/région              |  Vignette Satellite | Prévision à la minute, vignettes radar | Alertes météorologiques graves | Autre* |
|-----------------------------|:----------------:|:-----------------:|:--------:|  :--------:|
| Afghanistan                       | ✓ |   | | ✓| 
| Samoa américaines                    | ✓ |   |  ✓| ✓| 
| Australie                         | ✓ | ✓ |  ✓ | ✓| 
| Bangladesh                        | ✓ |   | | ✓| 
| Bhoutan                            | ✓ |   | | ✓| 
| Territoire britannique de l'océan Indien    | ✓ |   | | ✓| 
| Brunéi Darussalam                            | ✓ |   | | ✓| 
| Cambodge                          | ✓ |   | | ✓| 
| Chine                             | ✓ | ✓ |  ✓ | ✓| 
| Christmas (île)                  | ✓ |   | | ✓| 
| Îles Cocos (Keeling)           | ✓ |   | | ✓| 
| Cook (îles)                      | ✓ |   | | ✓| 
| Fidji                              | ✓ |   | | ✓| 
| Polynésie française                  | ✓ |   | | ✓| 
| Guam                              | ✓ |   |  ✓| ✓| 
| Heard et McDonald (îles) | ✓ |   | | ✓| 
| Hong Kong (R.A.S.)                     | ✓ |   | | ✓| 
| Inde                             | ✓ |   | | ✓| 
| Indonésie                         | ✓ |   | | ✓| 
| Japon                             | ✓ | ✓ |  ✓| ✓| 
| Kazakhstan                        | ✓ |   | | ✓| 
| Kiribati                          | ✓ |   | | ✓| 
| Corée du Sud                             | ✓ | ✓ | ✓ |  ✓| 
| Kirghizistan                        | ✓ |   | | ✓| 
| Laos                              | ✓ |   | | ✓| 
| Macao R.A.S.                         | ✓ |   | | ✓| 
| Malaisie                          | ✓ |   | | ✓| 
| Maldives                          | ✓ |   | | ✓| 
| Marshall (îles)                  | ✓ |   | ✓ | ✓| 
| Micronésie                        | ✓ |   | ✓ | ✓| 
| Mongolie                          | ✓ |   | | ✓| 
| Myanmar                           | ✓ |   | | ✓| 
| Nauru                             | ✓ |   | | ✓| 
| Népal                             | ✓ |   | | ✓| 
| Nouvelle-Calédonie                     | ✓ |   | | ✓| 
| Nouvelle-Zélande                       | ✓ |   | ✓ | ✓| 
| Niue                              | ✓ |   | | ✓| 
| Norfolk (île)                    | ✓ |   | | ✓| 
| Corée du Nord                       | ✓ |   | | ✓| 
| Îles Marianne du Nord          | ✓ |   | ✓ | ✓| 
| Pakistan                          | ✓ |   | | ✓| 
| Palau                             | ✓ |   | ✓ | ✓| 
| Papouasie Nouvelle Guinée                  | ✓ |   | | ✓| 
| Philippines                       | ✓ |   | ✓ | ✓| 
| Pitcairn (îles)                  | ✓ |   | | ✓| 
| Samoa                             | ✓ |   | | ✓| 
| Singapour                         | ✓ |   | | ✓| 
| Salomon (îles)                   | ✓ |   | | ✓| 
| Sri Lanka                         | ✓ |   | | ✓| 
| Taïwan                            | ✓ |   | | ✓| 
| Tadjikistan                        | ✓ |   | | ✓| 
| Thaïlande                          | ✓ |   | | ✓| 
| Timor-Leste                       | ✓ |   | | ✓| 
| Tokelau                           | ✓ |   | | ✓| 
| Tonga                             | ✓ |   | | ✓| 
| Turkménistan                      | ✓ |   | | ✓| 
| Tuvalu                            | ✓ |   | | ✓| 
| Ouzbékistan                        | ✓ |   | | ✓| 
| Vanuatu                           | ✓ |   | | ✓| 
| Vietnam                           | ✓ |   | | ✓| 
| Wallis et Futuna                 | ✓ |   | | ✓| 


## <a name="europe"></a>Europe

| Pays/région              |  Vignette Satellite | Prévision à la minute, vignettes radar | Alertes météorologiques graves | Autre* | 
|-----------------------------|:----------------:|:-----------------:|:--------:|:--------:|
| Albanie                | ✓ |   | | ✓| 
| Andorre                | ✓ |   | ✓ | ✓| 
| Arménie                | ✓ |   | | ✓| 
| Autriche                | ✓ | ✓ | ✓ | ✓|
| Azerbaïdjan             | ✓ |   | | ✓| 
| Bélarus                | ✓ |   | | ✓| 
| Belgique                | ✓ | ✓ |  ✓| ✓| 
| Bosnie-Herzégovine | ✓ | ✓ | ✓ | ✓| 
| Bulgarie               | ✓ |   |  ✓| ✓| 
| Croatie                | ✓ | ✓ |  ✓| ✓| 
| Chypre                 | ✓ |   | ✓ | ✓| 
| Tchéquie                | ✓ | ✓ | ✓ | ✓| 
| Danemark                | ✓ | ✓ | ✓ | ✓| 
| Estonie                | ✓ | ✓ |  ✓| ✓| 
| Féroé (îles)          | ✓ |   | | ✓| 
| Finlande                | ✓ | ✓ | ✓ | ✓| 
| France                 | ✓ | ✓ | ✓ | ✓| 
| Géorgie                | ✓ |   | | ✓| 
| Allemagne                | ✓ | ✓ | ✓ | ✓| 
| Gibraltar              | ✓ | ✓ | | ✓| 
| Grèce                 | ✓ |   |  ✓| ✓| 
| Guernesey               | ✓ |   | | ✓| 
| Hongrie                | ✓ | ✓ |  ✓| ✓| 
| Islande                | ✓ |   | ✓ | ✓| 
| Irlande                | ✓ | ✓ |  ✓| ✓| 
| Italie                  | ✓ |   |  ✓| ✓|
| Île de Man            | ✓ |   | | ✓| 
| Jan Mayen              | ✓ |   | | ✓| 
| Jersey                 | ✓ |   | | ✓| 
| Kosovo                 | ✓ |   |  ✓| ✓| 
| Lettonie                 | ✓ |   | ✓ | ✓| 
| Liechtenstein          | ✓ | ✓ |  ✓| ✓| 
| Lituanie              | ✓ |   | ✓ | ✓| 
| Luxembourg             | ✓ | ✓ |  ✓| ✓| 
| Macédoine du Nord        | ✓ |   | ✓ | ✓| 
| Malte                  | ✓ |   | ✓ | ✓| 
| Moldova                | ✓ | ✓ | ✓ | ✓| 
| Monaco                 | ✓ | ✓ |  ✓| ✓| 
| Monténégro             | ✓ | ✓ |  ✓| ✓| 
| Pays-Bas            | ✓ | ✓ |  ✓| ✓| 
| Norvège                 | ✓ | ✓ |  ✓| ✓| 
| Pologne                 | ✓ | ✓ |  ✓| ✓| 
| Portugal               | ✓ | ✓ |  ✓| ✓| 
| Roumanie                | ✓ | ✓ |  ✓| ✓| 
| Russie                 | ✓ |   |  ✓| ✓| 
| Saint-Marin             | ✓ |   |  ✓| ✓| 
| Serbie                 | ✓ | ✓ |  ✓| ✓| 
| Slovaquie               | ✓ | ✓ |  ✓| ✓| 
| Slovénie               | ✓ | ✓ |  ✓| ✓| 
| Espagne                  | ✓ | ✓ |  ✓| ✓| 
| Svalbard               | ✓ |   | | ✓|
| Suède                 | ✓ | ✓ |  ✓| ✓| 
| Suisse            | ✓ | ✓ | ✓| ✓| 
| Turquie                 | ✓ |   | | ✓| 
| Ukraine                | ✓ |   | | ✓| 
| Royaume-Uni         | ✓ | ✓ | ✓| ✓| 
| État de la Cité du Vatican           | ✓ |   |✓ | ✓|