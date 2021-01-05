---
title: Entités nommées générales
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 05/13/2020
ms.author: aahi
ms.openlocfilehash: 630f04bf2cc9e7de6331f9d25754a20fe2327d76
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91779705"
---
Les catégories d’entité suivantes sont retournées lors de l’envoi de requêtes au point de terminaison `/entities/recognition/general`.

| Category   | Sous-catégorie | Description                          | Version de modèle de départ                                                    | Notes |
|------------|-------------|--------------------------------------|-------------------------------------------------------------|--------------------------------------|
| Personne     | N/A         | Noms des personnes.  | `2019-10-01`  | Également retourné par NER v2.1 |
| PersonType | N/A         | Type de tâche ou rôle d’une personne. | `2020-02-01` | |
|Emplacement    | N/A         | Points de repère, structures et caractéristiques géographiques et entités géopolitiques naturels et créés par l’homme     |  `2019-10-01` | Également retourné par NER v2.1 |
|Emplacement     | Entité géopolitique (GPE)        | Villes, pays/régions, États.      | `2020-02-01` | |
|Emplacement     | Structurel                       | Structures faites par l’homme. | `2020-04-01` | |
|Emplacement     | Géographique       | Caractéristiques géographiques et naturelles telles que les rivières, les océans et les déserts. |  `2020-04-01` | |
|Organisation  | N/A | Sociétés, partis politiques, groupes de musique, clubs de sport, organismes gouvernementaux et organisations publiques.  | `2019-10-01` | Les nationalités et les religions ne sont pas incluses dans ce type d’entité. Également retourné par NER v2.1 |
|Organisation | Médecine | Sociétés et groupes médicaux. | `2020-04-01` |  |
|Organisation | Bourse | Groupes de bourse. | `2020-04-01` | |
| Organisation | Sports | Les organisations liées au sport. | `2020-04-01` |  |
| Événement  | N/A | Événements historiques, sociaux et naturels. | `2020-02-01` |  |
| Événement  | Culturel | Événements culturels et jours fériés. | `2020-04-01` | |
| Événement  | Natural | Événements naturels. | `2020-04-01` |  |
| Événement  | Sports | Événements sportifs.  | `2020-04-01` | |
| Produit | N/A | Objets physiques de différentes catégories. | `2020-02-01` | |
| Produit | Calcul des produits | Calcul des produits. |  `2020-02-01 ` | |
| Compétence | N/A | Une capacité, une compétence ou une expertise. | `2020-02-01` |  |
| Adresse | N/A | Adresses postales complètes.  | `2020-04-01` |  |
| PhoneNumber | N/A | Numéros de téléphone (Numéros de téléphone américains et européens uniquement). | `2019-10-01` | Également retourné par NER v2.1 |
| E-mail | N/A | Adresses e-mail. | `2019-10-01` | Également retourné par NER v2.1 |
| URL | N/A | URL vers des sites web. | `2019-10-01` | Également retourné par NER v2.1  |
| IP | N/A | Adresses IP du réseau. | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | N/A | Dates et heures du jour. | `2019-10-01` | Également retourné par NER v2.1 | 
| DateTime | Date | Dates du calendrier. | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | Temps | Heures de la journée | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | DateRange | Plages de dates. | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | TimeRange | Intervalles de temps. | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | Duration | Durées. | `2019-10-01` | Également retourné par NER v2.1 |
| DateTime | Définissez | Définie, répétée tant de fois. |  `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | N/A | Nombres et quantités numériques. | `2019-10-01` | Également retourné par NER v2.1  |
| Quantité | Number | Nombres. | `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | Pourcentage | Pourcentages.| `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | Ordinal | Nombres ordinaux. | `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | Age | Ages. | `2019-10-01` |  Également retourné par NER v2.1 |
| Quantité | Devise | Devises. | `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | Dimension | Dimensions et mesures. | `2019-10-01` | Également retourné par NER v2.1 |
| Quantité | Température | Températures. | `2019-10-01` | Également retourné par NER v2.1 |
