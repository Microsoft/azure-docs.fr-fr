---
title: Sécurité de la base de données – Azure Cosmos DB
description: Découvrez comment Azure Cosmos DB garantit la protection de la base de données et la sécurité de vos données.
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: mjbrown
ms.openlocfilehash: 19b4c8466e88159839ce1f43a5ba282b1bb3ec9e
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636909"
---
# <a name="security-in-azure-cosmos-db---overview"></a>Indexation dans Azure Cosmos DB - Vue d’ensemble
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Cet article décrit les meilleures pratiques en matière de sécurité de la base de données et les principales fonctionnalités d’Azure Cosmos DB qui vous aident à prévenir et détecter les violations de la base de données et à y répondre.

## <a name="whats-new-in-azure-cosmos-db-security"></a>Nouveautés en matière de sécurité dans Azure Cosmos DB

Le chiffrement au repos est désormais disponible pour les documents et les sauvegardes stockés dans Azure Cosmos DB dans toutes les régions Azure. Un chiffrement au repos est automatiquement appliqué aux clients nouveaux et existants dans ces régions. Vous n’avez pas à configurer quoi que ce soit. Vous bénéficiez des mêmes performances remarquables en matière de temps de latence, de débit, de disponibilité et de fonctionnalité qu’auparavant, avec l’avantage de savoir que vos données sont en lieu sûr et sécurisée grâce à un chiffrement au repos.

## <a name="how-do-i-secure-my-database"></a>Comment sécuriser une base de données

La sécurité des données est une responsabilité partagée entre vous, le client et le fournisseur de votre base de données. Selon le fournisseur de base de données que vous choisissez, la part de votre responsabilité peut varier. Si vous choisissez une solution locale, vous devez fournir tous les éléments de la protection de point de terminaison à la sécurité physique de votre matériel, ce qui n’est pas une tâche facile. Si vous choisissez un fournisseur de base de données cloud PaaS comme Azure Cosmos DB, vous réduisez considérablement votre niveau d’inquiétude. L’image suivante, tirée du livre blanc Microsoft [Shared Responsibilities for Cloud Computing](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91) (Responsabilités partagées en matière de cloud computing), montre en quoi votre responsabilité diminue en faisant appel à un fournisseur PaaS tel qu’Azure Cosmos DB.

:::image type="content" source="./media/database-security/nosql-database-security-responsibilities.png" alt-text="Responsabilités du client et du fournisseur de base de données":::

Le diagramme précédent présente les composants de sécurité cloud de haut niveau. Quels sont ceux auxquels vous devez être particulièrement attentif pour votre solution de base de données ? Et comment comparer les solutions entre elles ?

Nous vous recommandons d’utiliser la liste de vérification suivante qui répertorie les critères selon lesquels comparer les systèmes de base de données :

- Sécurité du réseau et paramètres de pare-feu
- Authentification des utilisateurs et contrôles utilisateur affinés
- Possibilité de répliquer des données globalement en cas de défaillances régionales
- Possibilité de basculer d’un centre de données vers un autre
- Réplication locale des données dans un centre de données
- Sauvegardes automatiques des données
- Restauration des données supprimées à partir de sauvegardes
- Protection et isolement des données sensibles
- Surveillance des attaques
- Réponse aux attaques
- Possibilité de délimiter géographiquement les données pour respecter les restrictions de gouvernance des données
- Protection physique des serveurs dans les centres de données protégés
- Certifications

Bien que cela paraisse évident, de récents exemples de [violations de base de données à grande échelle](https://thehackernews.com/2017/01/mongodb-database-security.html) nous rappellent l’importance simple mais critique des exigences suivantes :

- Serveurs corrigés et régulièrement mis à jour
- Chiffrement HTTPS par défaut/TLS
- Comptes administratifs avec des mots de passe forts

## <a name="how-does-azure-cosmos-db-secure-my-database"></a>Comment Azure Cosmos DB sécurise ma base de données ?

Reprenons la liste précédente : combien de ces exigences de sécurité Azure Cosmos DB garantit-il ? Toutes, sans exception.

Examinons à présent chacune d’entre elles en détail.

|Exigence de sécurité|Approche de la sécurité d’Azure Cosmos DB|
|---|---|
|Sécurité du réseau|L’utilisation d’un pare-feu IP est la première couche de protection pour sécuriser votre base de données. Azure Cosmos DB prend en charge les contrôles d’accès basés sur une stratégie IP pour le pare-feu entrant. Les contrôles d’accès basés sur IP sont semblables aux règles de pare-feu utilisées par les systèmes de base de données traditionnels, mais ils rendent un compte de base de données Azure Cosmos accessible uniquement à partir d’un ensemble d’ordinateurs ou de services cloud approuvés. Pour plus d’informations, consultez l’article [Prise en charge du pare-feu dans Azure Cosmos DB](how-to-configure-firewall.md).<br><br>Azure Cosmos DB vous permet d’activer une adresse IP spécifique (168.61.48.0), une plage d’adresses IP (168.61.48.0/8) et des combinaisons d’adresses et de plages IP. <br><br>Toutes les demandes provenant d’ordinateurs ne figurant pas dans cette liste sont bloquées par Azure Cosmos DB. Les demandes issues d’ordinateurs et de services cloud approuvés doivent ensuite terminer le processus d’authentification pour bénéficier d’un contrôle d’accès sur les ressources.<br><br> Vous pouvez utiliser des [étiquettes de service de réseau virtuel](../virtual-network/service-tags-overview.md) pour isoler le réseau et protéger vos ressources Azure Cosmos DB de l’accès Internet général. Utilisez des étiquettes de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple, AzureCosmosDB) dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant.|
|Autorisation|Azure Cosmos DB utilise un code d’authentification de message basé sur le hachage (HMAC) pour l’autorisation. <br><br>Chaque demande est hachée à l’aide de la clé secrète du compte, et ce hachage encodé en base 64 suivant est envoyé avec chaque appel à Azure Cosmos DB. Afin de valider la demande, le service Azure Cosmos DB utilise la clé secrète et les propriétés appropriées pour générer un hachage dont il compare la valeur à celle de la demande. Si les deux valeurs correspondent, l’opération est autorisée et la demande est traitée. Dans le cas contraire, un échec d’autorisation est généré et la demande est rejetée.<br><br>Vous pouvez utiliser une [clé primaire](#primary-keys) ou un [jeton de ressource](secure-access-to-data.md#resource-tokens) autorisant un accès précis à une ressource telle qu’un document.<br><br>Pour en savoir plus, consultez [Sécurisation de l’accès aux données Azure Cosmos DB](secure-access-to-data.md).|
|Utilisateurs et autorisations|À l’aide de la clé primaire du compte, vous pouvez créer des ressources utilisateur et des ressources d’autorisation par base de données. Dans une base de données, un jeton de ressource est associé à une autorisation et il détermine si l'utilisateur a accès (lecture-écriture, lecture seule ou aucun accès) à une ressource d'application de la base de données. Les ressources d’application comprennent les conteneurs, les documents, les pièces jointes, les procédures stockées, les déclencheurs et les fonctions définies par l’utilisateur. La jeton de ressource est ensuite utilisé lors de l’authentification pour autoriser ou refuser l’accès à la ressource.<br><br>Pour en savoir plus, consultez [Sécurisation de l’accès aux données Azure Cosmos DB](secure-access-to-data.md).|
|Intégration d’Active Directory (Azure RBAC)| Vous pouvez également fournir ou restreindre l’accès au compte, à la base de données, au conteneur et aux offres (débit) Cosmos à l’aide du contrôle d’accès (IAM) dans le portail Azure. IAM fournit un contrôle d’accès basé sur les rôles et s’intègre à Active Directory. Vous pouvez utiliser des rôles intégrés ou des rôles personnalisés pour les individus et les groupes. Consultez l’article [Intégration Active Directory](role-based-access-control.md) pour plus d’informations.|
|Réplication mondiale|Azure Cosmos DB offre une diffusion mondiale clé en main, ce qui vous permet de répliquer vos données dans n’importe quel centre de données Azure dans le monde en un seul clic. La réplication mondiale vous permet une mise à l’échelle globale et fournit un accès à faible latence à vos données dans le monde entier.<br><br>En matière de sécurité, la réplication mondiale protège les données contre les défaillances régionales.<br><br>Pour en savoir plus, consultez [Distribution mondiale des données avec DocumentDB](distribute-data-globally.md).|
|Basculements régionaux|Si vous avez répliqué vos données dans plusieurs centres de données, Azure Cosmos DB bascule automatiquement vos opérations si un centre de données régional est indisponible. Vous pouvez créer une liste des régions de basculement prioritaires comportant les régions dans lesquelles vos données sont répliquées. <br><br>Pour en savoir plus, consultez [Basculements régionaux automatiques pour la continuité des activités dans Azure Cosmos DB](high-availability.md).|
|Réplication locale|Même dans un centre de données, Azure Cosmos DB réplique automatiquement les données pour garantir une haute disponibilité et vous donne la possibilité de choisir des [niveaux de cohérence](consistency-levels.md). Cette réplication garantit un [contrat SLA de disponibilité](https://azure.microsoft.com/support/legal/sla/cosmos-db) de 99,99 % pour tous les comptes monorégions et multirégions avec cohérence souple, ainsi qu’une disponibilité de lecture de 99,999 % pour tous les comptes de base de données multirégions.|
|Sauvegardes en ligne automatisées|Les bases de données Azure Cosmos sont régulièrement sauvegardées et stockées dans un magasin géoredondant. <br><br>Pour en savoir plus, consultez [Sauvegarde et restauration en ligne automatiques avec Azure Cosmos DB](online-backup-and-restore.md).|
|Restauration de données supprimées|Les sauvegardes en ligne automatisées peuvent être utilisées pour récupérer des données que vous avez accidentellement supprimées jusqu’à environ 30 jours après l’événement. <br><br>Pour en savoir plus, consultez [Sauvegarde et restauration en ligne automatiques avec Azure Cosmos DB](online-backup-and-restore.md).|
|Protection et isolement des données sensibles|Toutes les données stockées dans les régions répertoriées dans la section Nouveautés sont désormais chiffrées au repos.<br><br>Les données personnelles et d’autres données confidentielles peuvent être isolées dans un conteneur spécifique, et un accès en lecture-écriture ou en lecture seule peut être restreint à des utilisateurs spécifiques.|
|Surveillance des attaques|À l’aide de [l’enregistrement d’audit et des journaux d’activité](./monitor-cosmos-db.md), vous pouvez surveiller les activités normales et anormales de votre compte. Vous pouvez afficher les opérations qui ont été effectuées sur vos ressources, la personne qui a initié l’opération, le moment auquel l’opération a été réalisée, l’état de l’opération et bien plus encore, comme l’illustre la capture d’écran qui suit ce tableau.|
|Réponse aux attaques|Une fois que vous avez contacté le support Azure pour signaler une attaque potentielle, un processus de réponse aux incidents en 5 étapes est lancé. L’objectif de ce processus en 5 étapes est de restaurer aussi rapidement que possible dans des conditions normales les opérations et la sécurité du service suite à la détection d’un problème et au lancement d’une investigation.<br><br>Pour en savoir plus, consultez [Microsoft Azure Security Response in the Cloud](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91) (Réponse de Microsoft Azure en matière de sécurité dans le cloud).|
|Délimitation géographique|Azure Cosmos DB garantit la gouvernance des données dans les régions souveraines (par exemple, Allemagne, Chine, US Gov).|
|Installations protégées|Dans Azure Cosmos DB, les données sont stockées sur des disques SSD dans les centres de données protégées d’Azure.<br><br>Pour en savoir plus, consultez les [centres de données Microsoft globaux](https://www.microsoft.com/en-us/cloud-platform/global-datacenters).|
|Chiffrement HTTPS/SSL/TLS|Toutes les connexions à Azure Cosmos DB prennent en charge le protocole HTTPS. Azure Cosmos DB prend également en charge le protocole TLS 1.2.<br>Il est possible d’appliquer une version TLS minimale côté serveur. Pour ce faire, veuillez contacter [azurecosmosdbtls@service.microsoft.com](mailto:azurecosmosdbtls@service.microsoft.com).|
|Chiffrement au repos|Toutes les données stockées dans Azure Cosmos DB sont chiffrées au repos. Pour en savoir plus, consultez [Chiffrement de base de données Azure Cosmos DB au repos](./database-encryption-at-rest.md).|
|Serveurs corrigés|En tant que base de données NoSQL gérée, Azure Cosmos DB ne nécessite aucune gestion ou correction des serveurs. Tout est fait automatiquement.|
|Comptes administratifs avec des mots de passe forts|Il est difficile de croire que nous devions encore mentionner cette exigence, mais contrairement à certains de nos concurrents, il est impossible d’avoir un compte d’administrateur sans mot de passe dans Azure Cosmos DB.<br><br> La sécurité via TLS et l’authentification basée sur un secret HMAC sont intégrées par défaut.|
|Certifications de sécurité et de protection des données| Pour obtenir la toute dernière liste de certifications, consultez le [site de conformité Azure](https://www.microsoft.com/en-us/trustcenter/compliance/complianceofferings) général, ainsi que le [document dédié à la conformité Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) le plus récent, contenant toutes les certifications (recherchez Cosmos). Pour obtenir des informations plus ciblées, lisez l’article intitulé [Azure #CosmosDB: Secure, private, compliant] du 25 avril 2018 qui traite des normes SOCS 1/2 Type 2, HITRUST, PCI DSS niveau 1, ISO 27001, HIPAA, FedRAMP High et de nombreuses autres certifications.

La capture d’écran suivante montre comment utiliser l’enregistrement d’audit et les journaux d’activité pour surveiller votre compte : :::image type="content" source="./media/database-security/nosql-database-security-application-logging.png" alt-text="Journaux d’activité pour Azure Cosmos DB":::

<a id="primary-keys"></a>

## <a name="primary-keys"></a>Clés primaires

Les clés primaires donnent accès à toutes les ressources administratives du compte de base de données. Elles présentent les caractéristiques suivantes :

- Fournissent un accès aux comptes, aux bases de données, aux utilisateurs et aux autorisations. 
- Ne peuvent pas être utilisées pour fournir un accès précis aux conteneurs et aux documents.
- Sont créées lors de la création d’un compte.
- Peuvent être régénérées à tout moment.

Chaque compte comporte deux clés primaires : une clé primaire et une clé secondaire. L’objectif de ces paires de clés est de pouvoir régénérer ou restaurer des clés tout en fournissant un accès permanent à votre compte et à vos données.

Outre les deux clés primaires du compte Azure Cosmos DB, il existe deux clés en lecture seule. Ces clés en lecture seule autorisent uniquement les opérations de lecture sur le compte. Les clés en lecture seule ne permettent pas de lire les ressources d’autorisation.

Les clés primaire, secondaire, en lecture seule, et en lecture et en écriture peuvent être récupérées et régénérées à l’aide du Portail Azure. Pour connaître la procédure, consultez [Affichage, copie et régénération des clés d’accès](manage-with-cli.md#regenerate-account-key).

:::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-portal.png" alt-text="Contrôle d’accès (IAM) dans le portail Azure - Démonstration de la sécurité de la base de données NoSQL":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les clés primaires et les jetons de ressource, consultez [Sécurisation de l’accès aux données Azure Cosmos DB](secure-access-to-data.md).

Pour plus d’informations sur l’enregistrement d’audit, consultez [Journalisation des diagnostics Azure Cosmos DB](./monitor-cosmos-db.md).

Pour plus d’informations sur les certifications Microsoft, consultez le [Centre de confidentialité Azure](https://azure.microsoft.com/support/trust-center/).