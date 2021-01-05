---
title: Entités nommées pour les informations personnelles
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/16/2020
ms.author: aahi
ms.openlocfilehash: eedfe4f24797a9b564479b2f8d3f4d04b0751272
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91779549"
---
> [!NOTE]
> Pour détecter des informations médicales protégées, utilisez le paramètre `domain=phi` et la version du modèle `2020-04-01` (ou une version ultérieure).
>
> Par exemple : `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.2/entities/recognition/pii?domain=phi&model-version=2020-07-01`
 
Les catégories d’entité suivantes sont retournées lorsque vous envoyez des requêtes au point de terminaison `/v3.1-preview.2/entities/recognition/pii`.

| Category   | Sous-catégorie | Description                          | Version de modèle de départ | Notes |
|------------|-------------|--------------------------------------|------------------------|---|
| Personne     | N/A         | Noms des personnes.  | `2019-10-01`  | Également retourné avec `domain=phi`. |
| PersonType | N/A         | Type de tâche ou rôle d’une personne. | `2020-02-01` | |
| PhoneNumber | N/A | Numéros de téléphone (Numéros de téléphone américains et européens uniquement). | `2019-10-01` | Également retourné avec `domain=phi`. |
|Organisation  | N/A | Sociétés, partis politiques, groupes de musique, clubs de sport, organismes gouvernementaux et organisations publiques.  | `2019-10-01` | Les nationalités et les religions ne sont pas incluses dans ce type d’entité.  |
|Organisation | Médecine | Sociétés et groupes médicaux. | `2020-04-01` |  |
|Organisation | Bourse | Groupes de bourse. | `2020-04-01` |  |
| Organisation | Sports | Les organisations liées au sport. | `2020-04-01` |  |
| Adresse | N/A | Adresses postales complètes.  | `2020-04-01` | Également retourné avec `domain=phi`. |
| Coordonnées GPS UE | N/A | Coordonnées GPS de lieux situés au sein de l’Union européenne.  | `2019-10-01` |  |
| E-mail | N/A | Adresses e-mail. | `2019-10-01` | Également retourné avec `domain=phi`.   |
| URL | N/A | URL vers des sites web. | `2019-10-01` | Également retourné avec `domain=phi`. |
| IP | N/A | Adresses IP du réseau. | `2019-10-01` | Également retourné avec `domain=phi`. |
| DateTime | N/A | Dates et heures du jour. | `2019-10-01` |  | 
| DateTime | Date | Dates du calendrier. | `2019-10-01` | Également retourné avec `domain=phi`. |
| Quantité | N/A | Nombres et quantités numériques. | `2019-10-01` |  |
| Quantité | Age | Ages. | `2019-10-01` | | |

## <a name="azure-information"></a>Informations Azure

Cette catégorie d’entité englobe les informations identifiables d’Azure, notamment les informations d’authentification et les chaînes de connexion. Disponible à partir de la version `2019-10-01` du modèle. Non retourné avec le paramètre `domain=phi`.

| Sous-catégorie                           | Description                                                                 |
|---------------------------------------|-----------------------------------------------------------------------------|
| Clé d’autorisation Azure DocumentDB             | Clé d’autorisation d’un serveur Azure CosmosDB.                           |
| Chaîne de connexion à la base de données Azure IAAS et chaîne de connexion Azure SQL | Chaîne de connexion pour une base de données IaaS (infrastructure as a service) Azure et chaîne de connexion SQL. |
| Chaîne de connexion Azure SQL           | Chaîne de connexion pour une base de données dans Azure SQL Database.                                |
| Chaîne de connexion Azure IoT           | Chaîne de connexion pour Azure IoT.                        |
| Mot de passe de paramètre de publication Azure        | Mot de passe pour les paramètres de publication Azure.                                        |
| Chaîne de connexion Azure Cache pour Redis   | Chaîne de connexion pour un cache Redis                             |
| Azure SAAS                             | Chaîne de connexion pour SaaS (software as a service) Azure.                     |
| Chaîne de connexion Azure Service Bus   | Chaîne de connexion pour Azure Service Bus.                                 |
| Clé du compte de Stockage Azure             | Clé d’un compte de stockage Azure.                                   |
| Clé du compte de Stockage Azure (générique)   | Clé générique d’un compte de stockage Azure.                           |
| Chaîne de connexion SQL Server          | Chaîne de connexion pour un ordinateur exécutant SQL Server.                                         |

## <a name="identification"></a>Identification

[!INCLUDE [supported identification entities](./identification-entities.md)]
