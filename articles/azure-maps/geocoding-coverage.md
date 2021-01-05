---
title: Couverture du géocodage dans le service de recherche de Microsoft Azure Maps
description: Découvrez les régions couvertes par la recherche Azure Maps. Les catégories de géocodage incluent les points d’adresse, les numéros de maison, le niveau de rue, le niveau de ville et les points d’intérêt.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/28/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 3cf27319fa84ff5d693e74e90664b7a86f4d934a
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897209"
---
# <a name="azure-maps-geocoding-coverage"></a>Couverture de géocodage Azure Maps

Le [service de recherche](/rest/api/maps/search) de Azure Maps prend en charge le géocodage, ce qui signifie que votre demande d’API peut comporter des termes recherchés, tels qu’une adresse ou le nom d’un emplacement, et retourne le résultat sous la forme de coordonnées de latitude et de longitude. Par exemple, l’[API Get Search Address](/rest/api/maps/search/getsearchaddress) d’Azure Maps reçoit des requêtes contenant des informations d’emplacement, et retourne les résultats sous la forme de coordonnées de latitude et de longitude.

Toutefois, le [service Search](/rest/api/maps/search) d’Azure Maps n’a pas le même niveau d’information et de précision pour tous les pays et régions. Utilisez cet article pour déterminer quels types d’emplacements vous pouvez rechercher de manière fiable dans chaque région.

La possibilité de géocoder dans un pays/une région dépend de la couverture de données de route et de la précision du service de géocodage. Les catégorisations suivantes permettent de spécifier le niveau de prise en charge du géocodage dans chaque pays/région.

* **Points d’adresse**  : les données d’adresses peuvent être résolues en coordonnées de latitude/longitude à l’intérieur de la parcelle de l’adresse (limite de propriété). Les points d’adresse sont souvent qualifiés de « Rooftop », ce qui correspond au niveau de précision le plus élevé disponible pour les adresses.
* **Numéros de résidence**  : Les adresses sont interpolées en coordonnées latitude/longitude dans la rue.
* **Niveau de rue**  : Les adresses sont résolues en coordonnées latitude/longitude de la rue qui contient l’adresse. Le numéro de résidence ne peut pas être traité.
* **Niveau de ville**  : Les noms de ville sont pris en charge.

## <a name="americas"></a>Amérique

| Pays/région                                       | Points d’adresse | Numéros de résidence | Niveau de rue | Niveau ville | Points d’intérêt |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Anguilla                                            |                 |                |              |      ✓     |          ✓         |
| Antarctique                                          |                 |                |              |      ✓     |          ✓         |
| Antigua-et-Barbuda                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Argentine                                           |       ✓         |        ✓       |       ✓      |      ✓     |          ✓         |
| Aruba                                               |                 |                |              |      ✓     |          ✓         |
| Les Bahamas                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Barbade                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Belize                                              |                 |                |              |      ✓     |          ✓         |
| Bermudes                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Bolivie                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Bonaire, Saint-Eustache et Saba                   |                 |                |              |      ✓     |          ✓         |
| Brésil                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Canada                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Caïmans (îles)                                      |                 |                |       ✓      |      ✓     |          ✓         |
| Chili                                               |       ✓         |        ✓       |       ✓      |      ✓     |          ✓         |
| Colombie                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Costa Rica                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Cuba                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Dominique                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Dominicana                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Équateur                                             |                 |                |       ✓      |      ✓     |          ✓         |
| El Salvador                                         |                 |                |       ✓      |      ✓     |          ✓         |
| Îles Malouines                                    |                 |                |              |      ✓     |          ✓         |
| Guyane française                                       |                 |                |       ✓      |      ✓     |          ✓         |
| Grenade                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Guadeloupe                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Guam                                                |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Guatemala                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Guyane                                              |                |             |           |      ✓     |                 |
| Haïti                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Honduras                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Jamaïque                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Martinique                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Mexique                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Montserrat                                          |                 |                |              |      ✓     |          ✓         |
| Nicaragua                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Panama                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Paraguay                                            |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Pérou                                                |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Porto Rico                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Saint-Barthélemy                                    |                 |                |       ✓      |      ✓     |          ✓         |
| Saint-Christophe-et-Niévès                               |                 |                |       ✓      |      ✓     |          ✓         |
| Sainte-Lucie                                         |                 |                |              |      ✓     |          ✓         |
| Saint-Martin                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Saint Pierre et Miquelon                           |                 |                |       ✓      |      ✓     |          ✓         |
| Saint-Vincent-et-les-Grenadines                    |                 |                |              |      ✓     |          ✓         |
| Saint-Martin (partie néerlandaise)                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Géorgie du Sud et îles Sandwich du Sud        |                 |                |              |      ✓     |          ✓         |
| Surinam                                            |                 |                |              |      ✓     |          ✓         |
| Trinité-et-Tobago                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Îles mineures éloignées des États-Unis                |                 |                |              |      ✓     |          ✓         |
| États-Unis d’Amérique                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Uruguay                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Venezuela                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Îles Vierges britanniques                              |                 |                |              |      ✓     |          ✓         |
| États-Unis Îles Vierges                                 |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |

## <a name="asia-pacific"></a>Asie-Pacifique

| Pays/région                                      | Points d’adresse |Numéros de résidence | Niveau de rue | Niveau ville | Points d’intérêt |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Samoa américaines                                      |                 |                |       ✓      |      ✓     |          ✓         |
| Australie                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bangladesh                                          |                 |                |              |      ✓     |          ✓         |
| Bhoutan                                              |                 |                |              |      ✓     |          ✓         |
| Territoire britannique de l'océan Indien                      |                 |                |              |      ✓     |          ✓         |
| Brunéi Darussalam                                              |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Cambodge                                            |                 |                |              |      ✓     |          ✓         |
| Chine                                               |                 |                |              |      ✓     |          ✓         |
| Christmas (île)                                    |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Îles Cocos (Keeling)                             |                 |                |              |      ✓     |          ✓         |
| Comores                                             |                 |                |              |      ✓     |          ✓         |
| Cook (îles)                                        |                 |                |              |      ✓     |          ✓         |
| Fidji                                                |                  |                |              |      ✓     |          ✓        |
| Polynésie française                                    |                 |                |              |      ✓     |          ✓         |
| Heard et McDonald (îles)                   |                 |                |              |      ✓     |          ✓         |
| Hong Kong (R.A.S.)                                       |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Indonésie                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Inde                                               |        ✓        |        ✓       |       ✓      |      ✓     |                   |
| Japon                                               |                 |                |              |      ✓     |          ✓         |
| Kiribati                                            |                 |                |              |      ✓     |          ✓         |
| Corée du Sud                                         |                 |                |              |      ✓     |          ✓         |
| Laos                                                |                 |                |              |      ✓     |          ✓         |
| Macao R.A.S.                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Malaisie                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Micronésie                                          |                 |                |              |      ✓     |          ✓         |
| Mongolie                                            |                 |                |              |      ✓     |          ✓         |
| Nauru                                               |                 |                |              |      ✓     |          ✓         |
| Népal                                               |                 |                |              |      ✓     |          ✓         |
| Nouvelle-Calédonie                                       |                 |                |              |      ✓     |          ✓         |
| Nouvelle-Zélande                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Niue                                                |                 |                |              |      ✓     |          ✓         |
| Norfolk (île)                                      |                 |                |              |      ✓     |          ✓         |
| Corée du Nord                                         |                 |                |              |      ✓     |          ✓         |
| Îles Marianne du Nord                            |                 |                |       ✓      |      ✓     |          ✓         |
| Pakistan                                            |                 |                |              |      ✓     |          ✓         |
| Palau                                               |                 |                |              |      ✓     |          ✓         |
| Papouasie Nouvelle Guinée                                    |                 |                |              |      ✓     |          ✓         |
| Philippines                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Île Pitcairn                                            |                 |                |              |      ✓     |          ✓         |
| Samoa                                               |                 |                |              |      ✓     |          ✓         |
| Îles Senkaku/Diaoyutai                                     |        ✓        |                |              |      ✓     |          ✓         |
| Singapour                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Salomon (îles)                                     |                 |                |              |      ✓     |          ✓         |
| Îles Kouriles                                     |        ✓        |                |              |      ✓     |          ✓         |
| Sri Lanka                                           |                 |                |              |      ✓     |          ✓         |
| Taïwan                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Thaïlande                                            |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Tokelau                                             |                 |                |              |      ✓     |          ✓         |
| Tonga                                               |                 |                |              |      ✓     |          ✓         |
| Turques-et-Caïques (îles)                            |                 |                |              |      ✓     |          ✓         |
| Tuvalu                                              |                 |                |              |      ✓     |          ✓         |
| Vanuatu                                             |                 |                |              |      ✓     |          ✓         |
| Vietnam                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Wallis et Futuna                                   |                 |                |              |      ✓     |          ✓         |

## <a name="europe"></a>Europe

| Pays/région                                      | Points d’adresse |Numéros de résidence | Niveau de rue | Niveau ville | Points d’intérêt |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Albanie                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Andorre                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Arménie                                             |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Autriche                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Azerbaïdjan                                          |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Belgique                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bosnie-Herzégovine                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bulgarie                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bélarus                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Croatie                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Chypre                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| République tchèque                                      |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Danemark                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Estonie                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Féroé (îles)                                       |                 |                |              |      ✓     |          ✓         |
| Finlande                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| France                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Géorgie                                             |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Allemagne                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Gibraltar                                           |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Grèce                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Groenland                                           |                 |                |              |      ✓     |          ✓         |
| Guernesey                                            |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Hongrie                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islande                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Irlande                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Île de Man                                         |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Italie                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Jan Mayen                                           |        ✓        |                |              |      ✓     |          ✓         |
| Jersey                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Kazakhstan                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Kosovo                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Kirghizistan                                          |                 |                |              |      ✓     |          ✓         |
| Lettonie                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Liechtenstein                                       |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Lituanie                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Luxembourg                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Macédoine du Nord                                     |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Malte                                               |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Moldova                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Monaco                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Monténégro                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Pays-Bas                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Norvège                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Pologne                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Portugal                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| +Açores et Madère                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Roumanie                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Fédération de Russie                                  |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Saint-Marin                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Serbie                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Slovaquie                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Slovénie                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Espagne                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Svalbard                                            |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Suède                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Suisse                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Tadjikistan                                          |                 |                |              |      ✓     |          ✓         |
| Turquie                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Turkménistan                                        |                 |                |              |      ✓     |          ✓         |
| Ukraine                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Royaume-Uni                                      |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Ouzbékistan                                          |                 |                |              |      ✓     |          ✓         |
| État de la Cité du Vatican                                        |                 |                |       ✓      |      ✓     |          ✓         |

## <a name="middle-east-and-africa"></a>Moyen-Orient et Afrique

| Pays/région                                      | Points d’adresse |Numéros de résidence | Niveau de rue | Niveau ville | Points d’intérêt |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Afghanistan                                         |                 |                |              |      ✓     |          ✓         |
| Algérie                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Angola                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Bahreïn                                             |        ✓        |       ✓        |       ✓      |      ✓     |          ✓         |
| Bénin                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Botswana                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Bouvet (île)                                       |                 |                |              |      ✓     |          ✓         |
| Burkina Faso                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Burundi                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Cameroun                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Cabo Verde                                          |                 |                |       ✓      |      ✓     |          ✓         |
| République centrafricaine                            |                 |                |       ✓      |      ✓     |          ✓         |
| Tchad                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Congo                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Côte d’Ivoire                                       |                 |                |       ✓      |      ✓     |          ✓         |
| République démocratique du Congo                    |                 |                |       ✓      |      ✓     |          ✓         |
| Djibouti                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Égypte                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Guinée équatoriale, République de                      |                 |                |       ✓      |      ✓     |          ✓         |
| Érythrée                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Éthiopie                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Terres australes françaises|                        |                |              |      ✓     |          ✓         |
| Gabon                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Gambie                                              |                 |                |              |      ✓     |          ✓         |
| Ghana                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Guinée                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Guinée-Bissau                                       |                 |                |       ✓      |      ✓     |          ✓         |
| Iran                                                |                 |                |              |      ✓     |          ✓         |
| Irak                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Israël                                              |        ✓        |       ✓        |              |      ✓     |          ✓         |
| Jordanie                                              |        ✓        |       ✓        |       ✓      |      ✓     |          ✓         |
| Kenya                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Koweït                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Liban                                             |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Lesotho                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Liberia                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Libye                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Madagascar                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Malawi                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Maldives                                            |                 |                |              |      ✓     |          ✓         |
| Mali                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Marshall (îles)                                    |                 |                |              |      ✓     |          ✓         |
| Mauritanie                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Maurice (île)                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Mayotte                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Maroc                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Mozambique                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Myanmar                                             |                 |                |              |      ✓     |          ✓         |
| Namibie                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Niger                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Nigeria                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Oman                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Qatar                                               |        ✓        |                |       ✓      |      ✓     |          ✓         |
| La Réunion                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Rwanda                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Sainte-Hélène                                        |                 |                |              |      ✓     |          ✓         |
| Arabie Saoudite                                        |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Sénégal                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Seychelles                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Sierra Leone                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Somalie                                             |                 |                |              |      ✓     |          ✓         |
| Afrique du Sud                                        |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Soudan du Sud                                         |                 |                |       ✓      |      ✓     |          ✓         |
| Soudan                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Swaziland                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Syrie                                               |                 |                |              |      ✓     |          ✓         |
| São Tomé et Príncipe                               |                 |                |       ✓      |      ✓     |          ✓         |
| Tanzanie                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Togo                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Tunisie                                             |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Ouganda                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Émirats Arabes Unis                                |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Yémen                                               |                 |                |              |      ✓     |          ✓         |
| Zambie                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Zimbabwe                                            |                 |                |       ✓      |      ✓     |          ✓         |

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur le géocodage d’Azure Maps :
> [!div class="nextstepaction"]
> [Service de recherche d’Azure Maps](/rest/api/maps/search)