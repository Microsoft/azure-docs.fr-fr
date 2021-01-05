---
title: 'Problèmes connus : Migrer de MongoDB vers Azure CosmosDB'
titleSuffix: Azure Database Migration Service
description: Découvrez les problèmes connus et les limitations de migration avec les migrations de MongoDB vers Azure Cosmos DB avec Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: troubleshooting
ms.date: 02/27/2020
ms.openlocfilehash: be6d9d3b8a20e11c874234baae26629ce0187e4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91291808"
---
# <a name="known-issuesmigration-limitations-with-migrations-from-mongodb-to-azure-cosmos-dbs-api-for-mongodb"></a>Problèmes connus/limitations de migration avec les migrations de MongoDB vers l’API Azure Cosmos DB pour MongoDB

Les sections suivantes décrivent les problèmes connus et les limitations associées aux migrations de MongoDB vers l’API Cosmos DB pour MongoDB.

## <a name="migration-fails-as-a-result-of-using-the-incorrect-ssl-cert"></a>La migration échoue à cause de l’utilisation d’un certificat SSL incorrect

* **Symptôme** : Ce problème est apparent quand un utilisateur ne parvient pas à se connecter au serveur source MongoDB. Bien que tous les ports de pare-feu soient ouverts, l’utilisateur ne peut toujours pas se connecter.

| Cause         | Résolution |
| ------------- | ------------- |
| L’utilisation d’un certificat auto-signé dans Azure Database Migration Service peut entraîner l’échec de la migration car le certificat SSL est incorrect. Le message d’erreur peut inclure le texte « Le certificat distant n’est pas valide selon la procédure de validation. ». | Utilisez un certificat authentique délivré par une autorité de certification.  Les certificats auto-signés sont généralement utilisés uniquement lors des tests internes. Quand vous installez un certificat authentique délivré par une autorité de certification, vous pouvez utiliser SSL dans Azure Database Migration Service sans problème (les connexions à Cosmos DB utilisent SSL sur l’API Mongo).<br><br> |

## <a name="unable-to-get-the-list-of-databases-to-map-in-dms"></a>Impossible d’obtenir la liste des bases de données pour le mappage dans DMS

* **Symptôme** : Impossible d’accéder à la liste des bases de données dans le panneau **Paramètre de base de données** lors de l’utilisation du mode **Données du stockage Azure** dans le panneau **Sélectionner une source**.

| Cause         | Résolution |
| ------------- | ------------- |
| La chaîne de connexion du compte de stockage ne contient pas les informations de la signature d’accès partagé et ne peut donc pas être authentifiée. | Créez la signature d’accès partagé sur le conteneur d’objets blob dans l’Explorateur Stockage et utilisez l’URL avec les informations de signature d’accès partagé du conteneur comme chaîne de connexion source.<br><br> |

## <a name="using-an-unsupported-version-of-the-database"></a>Utilisation d’une version non prise en charge de la base de données

* **Symptôme** : La migration échoue.

| Cause         | Résolution |
| ------------- | ------------- |
| Vous tentez de migrer vers Azure Cosmos DB à partir d’une version non prise en charge de MongoDB. | À mesure que de nouvelles versions de MongoDB sont publiées, elles sont testées pour garantir la compatibilité avec Azure Database Migration Service, et le service est régulièrement mis à jour pour accepter la ou les versions les plus récentes. En cas de besoin de migration immédiat, en guise de solution de contournement vous pouvez exporter les bases de données/collections vers Stockage Azure et faire pointer la source vers l’image mémoire résultante. Créez la signature d’accès partagé sur le conteneur d’objets blob dans l’Explorateur Stockage, puis utilisez l’URL avec les informations de signature d’accès partagé du conteneur comme chaîne de connexion source.<br><br> |

## <a name="next-steps"></a>Étapes suivantes

* Consultez le tutoriel [Migrer MongoDB vers l’API Azure Cosmos DB pour MongoDB en ligne à l’aide de DMS](tutorial-mongodb-cosmos-db-online.md).
* Consultez le tutoriel [Migrer MongoDB vers l’API Azure Cosmos DB pour MongoDB hors connexion à l’aide de DMS](tutorial-mongodb-cosmos-db.md).