---
title: Chiffrement au repos dans Azure Cosmos DB
description: Découvrez comment Azure Cosmos DB assure le chiffrement des données au repos, et comment s’effectue son implémentation.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.custom: seodec18
ms.openlocfilehash: abe4bb3a53e3177857cb8041fd255ec916985054
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93089863"
---
# <a name="data-encryption-in-azure-cosmos-db"></a>Chiffrement des données dans Azure Cosmos DB 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Le chiffrement au repos est une expression qui se réfère en général au chiffrement des données situées sur des appareils de stockage non volatile, comme des disques SSD et des lecteurs de disques durs. Cosmos DB stocke ses bases de données primaires sur des SSD. Ses pièces jointes multimédias et ses sauvegardes sont stockées dans le Stockage Blob Azure, qui est généralement sauvegardé sur des disques durs. Avec l’apparition du chiffrement au repos pour Cosmos DB, toutes vos bases de données, pièces jointes multimédias et sauvegardes sont maintenant chiffrées. Vos données sont désormais chiffrées pendant le transit (sur le réseau) et au repos (stockage non volatile), ce qui vous donne un chiffrement de bout en bout.

Azure Cosmos DB, en tant que service PaaS, est très facile à utiliser. Comme toutes les données utilisateur stockées dans Azure Cosmos DB sont chiffrées au repos et lors du transport, vous n’avez aucune action à effectuer. En d’autres termes, le chiffrement au repos est « activé » par défaut. Il n’existe aucun contrôle à activer ou à désactiver. Azure Cosmos DB utilise le chiffrement AES-256 dans toutes les régions où le compte est exécuté. Nous fournissons cette fonctionnalité dans le cadre de nos efforts pour respecter notre [contrat SLA de disponibilité et de performances](https://azure.microsoft.com/support/legal/sla/cosmos-db). Les données stockées dans votre compte Azure Cosmos sont chiffrées automatiquement et de façon fluide avec des clés gérées par Microsoft (clés gérées par le service). Vous pouvez aussi choisir d’ajouter une deuxième couche de chiffrement avec vos propres clés, comme décrit dans l’article [Clés gérées par le client](how-to-setup-cmk.md).

## <a name="implementation-of-encryption-at-rest-for-azure-cosmos-db"></a>Implémentation du chiffrement au repos pour Azure Cosmos DB

Le chiffrement au repos est implémenté à l’aide d’un certain nombre de technologies de sécurité, notamment des systèmes de stockage de clés sécurisés, des réseaux chiffrés et des API de chiffrement. Les systèmes qui déchiffrent et traitent les données doivent communiquer avec les systèmes de gestion de clés. Le schéma montre la façon dont le stockage des données chiffrées et la gestion des clés sont séparés. 

:::image type="content" source="./media/database-encryption-at-rest/design-diagram.png" alt-text="Schéma montrant la conception" border="false":::

Le flux de base d’une demande d’un utilisateur est le suivant :
- Le compte de base de données de l’utilisateur est préparé et les clés de stockage sont récupérées via une demande au fournisseur de ressources du service de gestion.
- Un utilisateur crée une connexion à Cosmos DB via le transport HTTPS/sécurisé. (Les SDK permettent de s’abstraire des détails.)
- L’utilisateur envoie un document JSON à stocker sur la connexion sécurisée créée précédemment.
- Le document JSON est indexé, à moins que l’utilisateur ait désactivé l’indexation.
- Le document JSON et les données d’index sont écrits dans le stockage sécurisé.
- Périodiquement, les données sont lues à partir du stockage sécurisé et sauvegardées dans le magasin d’objets blob chiffré d’Azure.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="q-how-much-more-does-azure-storage-cost-if-storage-service-encryption-is-enabled"></a>Q : Quel est le coût supplémentaire du stockage Azure si la fonctionnalité Storage Service Encryption est activée ?
A : Aucun coût supplémentaire n’est facturé.

### <a name="q-who-manages-the-encryption-keys"></a>Q : Qui gère les clés de chiffrement ?
A : Les clés sont gérées par Microsoft.

### <a name="q-how-often-are-encryption-keys-rotated"></a>Q : À quelle fréquence les clés de chiffrement tournent-elles ?
A : Microsoft a constitué un ensemble de règles internes pour la rotation des clés de chiffrement, celles-ci sont suivies par Cosmos DB. Les règles spécifiques ne sont pas publiées. Microsoft publie le [Security Development Lifecycle (SDL)](https://www.microsoft.com/sdl/default.aspx), qui est vu comme un sous-ensemble de règles internes et qui contient des bonnes pratiques utiles pour les développeurs.

### <a name="q-can-i-use-my-own-encryption-keys"></a>Q : Puis-je utiliser mes propres clés de chiffrement ?
A : Oui, cette fonctionnalité est maintenant disponible pour les nouveaux comptes Azure Cosmos DB, et ceci doit être fait au moment de la création du compte. Pour plus d’informations, consultez le document [Clés gérées par le client](./how-to-setup-cmk.md).

### <a name="q-what-regions-have-encryption-turned-on"></a>Q : Dans quelles régions le chiffrement est-il activé ?
A : Le chiffrement est activé dans toutes les régions Azure Cosmos DB pour l’ensemble des données utilisateur.

### <a name="q-does-encryption-affect-the-performance-latency-and-throughput-slas"></a>Q : Le chiffrement modifie-t-il les SLA en termes de débit et de latence des performances ?
A : L’activation du chiffrement au repos sur tous les comptes (existants et nouveaux) ne modifie en rien les SLA en termes de performance. Pour en savoir plus sur les dernières garanties, consultez la page [SLA pour Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db).

### <a name="q-does-the-local-emulator-support-encryption-at-rest"></a>Q : L’émulateur local prend-il en charge le chiffrement au repos ?
A : L’émulateur est un outil de développement/de test autonome ; il n’exploite pas les services de gestion de clés que le service Cosmos DB managé utilise. Nous vous conseillons d’activer BitLocker sur les lecteurs où vous stockez des données de test d’émulateur sensibles. [L’émulateur prend en charge la modification du répertoire de données par défaut](local-emulator.md) ainsi que l’utilisation d’un emplacement connu.

## <a name="next-steps"></a>Étapes suivantes

* Vous pouvez choisir d’ajouter une deuxième couche de chiffrement avec vos propres clés. Pour en savoir plus, consultez l’article [Clés gérées par le client](how-to-setup-cmk.md).
* Pour une vue d’ensemble de la sécurité de Cosmos DB et des dernières améliorations, consultez [Sécurité de la base de données Azure Cosmos](database-security.md).
* Pour plus d’informations sur les certifications Microsoft, consultez le [Centre de confidentialité Azure](https://azure.microsoft.com/support/trust-center/).