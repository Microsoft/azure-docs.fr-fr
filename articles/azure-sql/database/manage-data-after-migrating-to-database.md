---
title: Gérer après la migration
titleSuffix: Azure SQL Database
description: Apprenez à gérer des bases de données uniques mises en pool après leur migration vers Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: joesackmsft
ms.author: josack
ms.reviewer: sstein
ms.date: 02/13/2019
ms.openlocfilehash: b34ac24cb26bf5db4a49a5ad5b531deb252f4695
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446126"
---
# <a name="new-dba-in-the-cloud--managing-azure-sql-database-after-migration"></a>Nouvel administrateur de base de données dans le cloud - Gérer Azure SQL Database après la migration
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Le passage d’un environnement autogéré et auto-contrôlé traditionnel à un environnement PaaS peut sembler un peu lourd dans un premier temps. En tant que développeur d’applications ou administrateur de base de données, vous avez besoin de connaître les principales fonctionnalités de la plateforme qui facilitent le maintien de la disponibilité, des performances, de la sécurité et de la résilience de votre application, en continu. Cet article va répondre précisément à ce besoin. L'article organise succinctement les ressources et vous fournit des conseils sur la façon de mieux utiliser les fonctionnalités clés d’Azure SQL Database avec des bases de données uniques et mises en pool pour gérer votre application, maintenir son efficacité et atteindre des résultats optimaux dans le cloud. Cet article vous intéresse si vous êtes dans les cas suivants :

- Vous êtes en train d'évaluer la migration de vos applications vers Azure SQL Database - Modernisation de vos applications.
- Vous êtes en train d’effectuer la migration de vos applications – Scénario de migration en cours.
- Vous avez récemment terminé la migration vers Azure SQL Database - Nouvel administrateur de base de données dans le cloud.

Cet article traite de certaines caractéristiques essentielles d'Azure SQL Database en tant que plateforme, auxquelles vous pouvez facilement avoir recours lorsque vous utilisez des bases de données uniques et des bases de données mises en pool dans des pools élastiques. Ces caractéristiques sont les suivantes :

- Analyser des bases de données au moyen du portail Azure
- Continuité d’activité et récupération d’urgence (BCDR)
- Sécurité et conformité
- Surveillance et maintenance de bases de données intelligentes
- Déplacement des données

## <a name="monitor-databases-using-the-azure-portal"></a>Analyser des bases de données au moyen du portail Azure

Sur le [portail Azure](https://portal.azure.com/), vous pouvez superviser l'utilisation d'une base de données individuelle en la sélectionnant et en cliquant sur le graphique **Supervision**. Une fenêtre **Métrique** apparaît. Vous pouvez la modifier en cliquant sur le bouton **Modifier le graphique**. Ajoutez les mesures suivantes :

- Pourcentage UC
- Pourcentage DTU
- Pourcentage E/S des données
- Pourcentage de la taille de la base de données

Une fois que vous avez ajouté ces métriques, vous pouvez continuer à les afficher dans le graphique **Surveillance** avec plus d’informations dans la fenêtre **Métrique**. Les quatre mesures montrent le pourcentage d’utilisation moyen correspondant aux **DTU** de votre base de données. Consultez les articles [Modèle d’achat DTU](service-tiers-dtu.md) et [Modèle d’achat vCore](service-tiers-vcore.md) pour en savoir plus sur les niveaux de service.  

![Surveillance des niveaux de service des performances de la base de données.](./media/manage-data-after-migrating-to-database/sqldb_service_tier_monitoring.png)

Vous pouvez également configurer des alertes sur les mesures de performances. Cliquez sur le bouton **Ajouter une alerte** situé dans la fenêtre **Métrique**. Suivez l'assistant pour configurer votre alerte. Vous avez la possibilité de configurer une alerte si les mesures dépassent un certain seuil ou si la mesure tombe en dessous d’un certain seuil.

Par exemple, si vous pensez que la charge de travail dans votre base de données va augmenter, vous pouvez choisir de configurer une alerte par courrier électronique chaque fois que votre base de données atteint 80 % de n'importe quelle mesure de performances. Vous pouvez l’utiliser comme un avertissement anticipé pour déterminer le moment auquel vous devez passer à la taille de calcul supérieure.

Les métriques de performances peuvent également vous aider à déterminer si vous pouvez passer à une taille de calcul inférieure. Supposons que vous utilisez une base de données standard S2 et que toutes les mesures de performance indiquent que la base de données n'utilise pas plus de 10 % des performances en moyenne. Cette base de données fonctionnerait très bien en version S1 standard. Toutefois, prenez en considération les éventuels pics ou baisses de charges de travail avant de passer à une taille de calcul inférieure.

## <a name="business-continuity-and-disaster-recovery-bcdr"></a>Continuité d’activité et récupération d’urgence (BCDR)

Les possibilités de continuité d’activité et de récupération d’urgence vous permettent de poursuivre votre activité, comme à l’accoutumée, en cas de sinistre. Un sinistre peut correspondre à un événement au niveau de la base de données (par exemple, une personne supprime par erreur une table essentielle) ou à un événement au niveau du centre de données (catastrophe naturelle dans la région, un tsunami par exemple).

### <a name="how-do-i-create-and-manage-backups-on-sql-database"></a>Comment créer et gérer les sauvegardes dans SQL Database

Vous ne créez pas de sauvegardes sur Azure SQL Database parce que vous n'avez pas à le faire. SQL Database sauvegarde automatiquement les bases de données pour vous, donc vous n’avez plus à vous inquiéter de leur planification, leur exécution et leur gestion. La plateforme effectue une sauvegarde complète toutes les semaines, une sauvegarde différentielle à l’issue de quelques heures et une sauvegarde de journal toutes les 5 minutes pour garantir l’efficacité de la récupération d’urgence et une perte de données minimale. La première sauvegarde complète se produit dès que vous créez une base de données. Ces sauvegardes sont à votre disposition pendant un certain temps appelé « période de conservation », qui varie selon la taille de calcul que vous choisissez. SQL Database vous offre la possibilité d’effectuer une restauration à partir de n’importe quel point dans le temps compris dans cette période de rétention à l’aide d’une [récupération jusqu’à une date et heure](recovery-using-backups.md#point-in-time-restore).

|Niveau de service|Période de rétention en jours|
|---|:---:|
|De base|7|
|Standard|35|
|Premium|35|
|||

De plus, la fonctionnalité de [rétention à long terme](long-term-retention-overview.md) vous permet de stocker vos fichiers de sauvegarde pendant une période plus longue, pouvant aller jusqu’à 10 ans, et de restaurer les données de ces sauvegardes à partir de n’importe quel point compris dans cette période. Par ailleurs, les sauvegardes de bases de données sont conservées dans un stockage géorépliqué pour assurer leur résilience en cas de catastrophe régionale. Vous pouvez également restaurer ces sauvegardes dans n’importe quelle région Azure à tout point dans le temps compris dans la période de rétention. Consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

### <a name="how-do-i-ensure-business-continuity-in-the-event-of-a-datacenter-level-disaster-or-regional-catastrophe"></a>Comment assurer la continuité de l’activité en cas de sinistre au niveau du centre de données ou de catastrophe régionale

Comme vos sauvegardes de bases de données sont stockées dans un stockage géorépliqué, en cas de sinistre régional, vous pouvez restaurer la sauvegarde dans une autre région Azure. Cette fonctionnalité s’appelle la géorestauration. L’objectif de point de récupération (RPO) correspondant est généralement inférieur à 1 heure et le temps de récupération estimé est compris entre quelques minutes et quelques heures.

Pour les bases de données stratégiques, Azure SQL Database propose une géoréplication active. En bref, une copie secondaire géorépliquée de votre base de données d’origine est créée dans une autre région. Par exemple, votre base de données est initialement hébergée dans la région Azure USA Ouest et vous avez besoin d’une résilience en cas de sinistre régional. Vous créez alors un géoréplica actif de la base de données de la région USA Ouest dans la région USA Est. Quand la catastrophe frappe la région USA Ouest, vous pouvez basculer vers la région USA Est. Leur configuration dans un groupe de basculement automatique est préférable, pour garantir que la base de données bascule automatiquement vers le serveur secondaire de la région USA Est en cas de sinistre. Le RPO correspondant est inférieur à 5 secondes et le temps de récupération estimé est inférieur à 30 secondes.

Si aucun groupe de basculement automatique n’est configuré, votre application doit effectuer une surveillance active et déclencher le basculement vers les bases de données secondaires en cas d’urgence. Vous pouvez créer jusqu’à 4 géoréplicas actifs de ce type dans les différentes régions Azure. Vous obtenez ainsi des résultats encore meilleurs. Vous pouvez également accéder à ces géoréplicas actifs secondaires en lecture seule. Cette possibilité s’avère très pratique pour réduire la latence dans un scénario d’application géodistribuée.

### <a name="how-does-my-disaster-recovery-plan-change-from-on-premises-to-sql-database"></a>En quoi mon plan de reprise d’activité local diffère de celui de SQL Database

En résumé, le programme d'installation de SQL Server vous oblige à gérer activement votre disponibilité à l'aide de fonctionnalités comme le clustering de basculement, la mise en miroir de bases de données, la réplication transactionnelle ou la copie des journaux de transaction, et à assurer la maintenance et la gestion de sauvegardes pour garantir la continuité de l'activité. Avec SQL Database, la plateforme gère tout cela pour vous, donc vous pouvez vous concentrer sur le développement et l’optimisation de votre application de base de données, sans avoir à vous soucier autant de la gestion des sinistres. Vous pouvez avoir des plans de sauvegarde et de récupération d’urgence, configurables et opérationnels en quelques clics seulement par le biais du portail Azure (ou en utilisant quelques commandes des API PowerShell).

Pour plus d’informations sur la récupération d’urgence, consultez : [Récupération d'urgence Azure SQL Database 101](https://azure.microsoft.com/blog/azure-sql-databases-disaster-recovery-101/)

## <a name="security-and-compliance"></a>Sécurité et conformité

SQL Database prend très au sérieux la sécurité et la confidentialité. La sécurité au sein de SQL Database est disponible au niveau de la base de données et au niveau de la plateforme. Il est plus facile de la comprendre en la classant dans plusieurs couches. Au niveau de chaque couche, vous pouvez contrôler et fournir une sécurité optimale pour votre application. Ces couches sont les suivantes :

- Identité et authentification ([authentification SQL et authentification Azure Active Directory [Azure AD]](logins-create-manage.md)).
- Activité de surveillance ([audit](../../azure-sql/database/auditing-overview.md) et [détection des menaces](threat-detection-configure.md)).
- Protection des données réelles ([Transparent Data Encryption [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) et [Always Encrypted [AE]](/sql/relational-databases/security/encryption/always-encrypted-database-engine)).
- Contrôle de l’accès aux données sensibles et privilégiées ([sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) et [Dynamic Data Masking](/sql/relational-databases/security/dynamic-data-masking)).

[Azure Security Center](https://azure.microsoft.com/services/security-center/) offre une gestion centralisée de la sécurité pour les charges de travail qui s’exécutent dans Azure, localement et dans d’autres clouds. Vous pouvez voir si les fonctionnalités de protection essentielles de SQL Database, telles que l’[audit](../../azure-sql/database/auditing-overview.md) et [Transparent Data Encryption [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql), sont configurées pour toutes les ressources. Si nécessaire, vous pouvez créer des stratégies basées sur vos propres exigences.

### <a name="what-user-authentication-methods-are-offered-in-sql-database"></a>Quelles sont les méthodes d’authentification utilisateur proposées dans SQL Database

SQL Database propose deux méthodes d’authentification :

- [Authentification Azure Active Directory](authentication-aad-overview.md)
- [Authentification SQL](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication)

L’authentification Windows traditionnelle n’est pas prise en charge. Azure Active Directory (Azure AD) est un service centralisé de gestion des identités et des accès. Ce service vous permet de proposer très facilement un accès par authentification unique à tout le personnel de votre organisation. Autrement dit, les informations d’identification sont partagées entre tous les services Azure pour une authentification plus simple. 

Azure AD prend en charge [Azure AD Multi-Factor Authentication](authentication-mfa-ssms-overview.md) et, en [quelques clics](../../active-directory/hybrid/how-to-connect-install-express.md), peut être intégré à Windows Server Active Directory. L’authentification SQL fonctionne exactement comme par le passé. Vous fournissez un nom d'utilisateur/mot de passe et vous pouvez authentifier des utilisateurs après de toute base de données sur un serveur donné. Cela permet également à SQL Database et Azure Synapse Analytics d'offrir une authentification multifacteur et des comptes d'utilisateurs invités dans un domaine Azure AD. Si vous disposez déjà d’Active Directory localement, vous pouvez fédérer l’annuaire avec Azure Active Directory pour étendre votre annuaire à Azure.

|**Si vous…**|**SQL Database / Azure Synapse Analytics**|
|---|---|
|Préférez ne pas utiliser Azure Active Directory (Azure AD) dans Azure|Utilisez l’[authentification SQL](security-overview.md)|
|Avez utilisé AD avec SQL Server localement|[Fédérez AD avec Azure AD](../../active-directory/hybrid/whatis-hybrid-identity.md), puis utilisez l’authentification basée sur Azure AD Authentication. Ainsi, vous pouvez utiliser l’authentification unique.|
|Devez appliquer l’authentification multifacteur|Imposez l’authentification multifacteur en tant que stratégie via l’[accès conditionnel Microsoft](conditional-access-configure.md), et utilisez l’[authentification universelle Azure AD avec prise en charge de l’authentification multifacteur](authentication-mfa-ssms-overview.md).|
|Avez des comptes Invité issus de comptes Microsoft (live.com, outlook.com) ou d’autres domaines (gmail.com)|Utilisez l’[authentification universelle Azure AD](authentication-mfa-ssms-overview.md) de SQL Database/Data Warehouse, qui tire profit d’[Azure AD B2B Collaboration](../../active-directory/external-identities/what-is-b2b.md).|
|Êtes connecté à Windows avec des informations d’identification Azure AD provenant d’un domaine fédéré|Utilisez l’[authentification intégrée Azure AD](authentication-aad-configure.md).|
|Êtes connecté à Windows avec des informations d’identification d’un domaine non fédéré avec Azure|Utilisez l’[authentification intégrée Azure AD](authentication-aad-configure.md).|
|Avez des services de niveau intermédiaire qui doivent se connecter à SQL Database ou Azure Synapse Analytics|Utilisez l’[authentification intégrée Azure AD](authentication-aad-configure.md).|
|||

### <a name="how-do-i-limit-or-control-connectivity-access-to-my-database"></a>Comment limiter ou contrôler l’accès à la connectivité de ma base de données

Il existe plusieurs techniques à votre disposition pour garantir une organisation de connectivité optimale à votre application.

- Règles de pare-feu
- Points de terminaison de service de réseau virtuel
- IP réservées

#### <a name="firewall"></a>Pare-feu

Un pare-feu empêche l'accès à votre serveur à partir d'une entité externe en autorisant uniquement l'accès à votre serveur à des entités spécifiques. Par défaut, toutes les connexions à des bases de données sont refusées au sein du serveur, à l'exception des connexions provenant d'autres services Azure. Avec une règle de pare-feu, vous pouvez ouvrir l’accès à votre serveur à certaines entités uniquement (par exemple, un ordinateur de développeur) que vous approuvez, en autorisant l’adresse IP correspondante à traverser le pare-feu. Une règle vous permet également de spécifier une plage d'adresses IP auxquelles vous autorisez l'accès au serveur. Par exemple, vous pouvez rapidement ajouter toutes les adresses IP des ordinateurs de développeur de votre organisation en spécifiant une plage dans la page des paramètres du pare-feu.

Vous pouvez créer des règles de pare-feu au niveau du serveur ou de la base de données. Des règles de pare-feu IP de niveau serveur peuvent être créées à l’aide du portail Azure ou avec SSMS. Pour en savoir plus sur la façon de définir une règle de pare-feu au niveau du serveur et de la base de données, consultez : [Créer des règles de pare-feu IP dans SQL Database](secure-database-tutorial.md#create-firewall-rules).

#### <a name="service-endpoints"></a>Points de terminaison de service

Par défaut, votre base de données est configurée pour « Autoriser les services Azure à accéder au serveur », ce qui signifie que toutes les machines virtuelles Azure peuvent essayer de se connecter à votre base de données. Ces tentatives doivent quand même faire l’objet d’une authentification. Néanmoins, si vous ne voulez pas que votre base de données soit accessible par toutes les adresses IP Azure, vous pouvez désactiver l’option « Autoriser les services Azure à accéder au serveur ». De plus, vous pouvez configurer des [points de terminaison de service de réseau virtuel](vnet-service-endpoint-rule-overview.md).

Les points de terminaison de service vous permettent d’exposer vos ressources Azure critiques uniquement à votre propre réseau privé virtuel dans Azure. Ainsi, vous éliminez simplement tout accès public à vos ressources. Le trafic entre votre réseau virtuel et Azure reste sur le réseau principal Azure. Sans point de terminaison de service, vous obtenez un routage de paquets par tunneling forcé. Votre réseau virtuel force le trafic Internet vers votre organisation et le trafic du service Azure à emprunter le même itinéraire. Avec des points de terminaison de service, vous pouvez optimiser ce trafic puisque les paquets transitent directement depuis votre réseau virtuel vers le service sur le réseau principal Azure.

![Points de terminaison de service VNet](./media/manage-data-after-migrating-to-database/vnet-service-endpoints.png)

#### <a name="reserved-ips"></a>IP réservées

Vous pouvez également provisionner des [adresses IP réservées](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip) pour vos machines virtuelles, et ajouter ces adresses IP de machines virtuelles spécifiques dans les paramètres de pare-feu du serveur. En affectant des adresses IP réservées, vous évitez de devoir mettre à jour les règles du pare-feu en cas de changement des adresses IP.

### <a name="what-port-do-i-connect-to-sql-database-on"></a>Sur quel port dois-je me connecter à SQL Database

Port 1433. SQL Database communique par le biais de ce port. Pour vous connecter à partir d’un réseau d’entreprise, vous devez ajouter une règle de trafic sortant dans les paramètres du pare-feu de votre organisation. En règle générale, évitez d’exposer le port 1433 hors de la limite Azure.

### <a name="how-can-i-monitor-and-regulate-activity-on-my-server-and-database-in-sql-database"></a>Comment superviser et réguler l’activité sur mon serveur et ma base de données dans SQL Database

#### <a name="sql-database-auditing"></a>Audit SQL Database

Avec SQL Database, vous pouvez activer l’audit pour suivre les événements de la base de données. [L’audit SQL Database](../../azure-sql/database/auditing-overview.md) enregistre les événements de base de données et les écrit dans un fichier journal d’audit de votre compte de stockage Azure. L’audit s’avère particulièrement utile si vous voulez obtenir un aperçu des violations potentielles de la sécurité et des stratégies, assurer une mise en conformité avec les réglementations applicables, etc. Il vous permet de définir et configurer certaines catégories d’événements à auditer. Ainsi, vous pouvez obtenir des rapports préconfigurés et un tableau de bord pour obtenir une vue d’ensemble des événements qui se produisent sur votre base de données. Vous pouvez appliquer ces stratégies d’audit au niveau de la base de données ou au niveau du serveur. Pour savoir comment activer l’audit pour votre serveur/base de données, consultez : [Activer l’audit SQL Database](secure-database-tutorial.md#enable-security-features).

#### <a name="threat-detection"></a>Détection de menaces

Avec la [détection des menaces](threat-detection-configure.md), vous avez la possibilité de réagir aux violations de stratégie ou de sécurité découvertes par l’audit très facilement. Nul besoin d’être un expert en matière de sécurité pour traiter les menaces ou violations potentielles dans votre système. La détection des menaces propose également des fonctions intégrées comme la détection d’injection de code SQL. Une injection de code SQL est une tentative de modifier ou de compromettre les données et une manière très courante d’attaquer une application de base de données de façon générale. La détection des menaces exécute plusieurs ensembles d'algorithmes qui détectent les vulnérabilités potentielles et les attaques par injection de code SQL, ainsi que les modèles anormaux d'accès aux bases de données (par exemple, les accès à partir d'un emplacement inhabituel ou par un principal inconnu). Les responsables sécurité ou les administrateurs désignés reçoivent une notification par e-mail si une menace est détectée sur la base de données. Chaque notification fournit des détails sur l’activité suspecte, ainsi que des recommandations sur l’analyse et l’atténuation de la menace. Pour savoir comment activer la détection des menaces, consultez : [Activer la détection des menaces](secure-database-tutorial.md#enable-security-features).

### <a name="how-do-i-protect-my-data-in-general-on-sql-database"></a>Comment protéger mes données de façon générale sur SQL Database

Le chiffrement constitue un puissant mécanisme de protection et de sécurisation de vos données sensibles contre les intrus. Vos données chiffrées ne sont d’aucune utilité à l’intrus s’il n’a pas la clé de déchiffrement. Par conséquent, le chiffrement ajoute une couche supplémentaire de protection aux couches de sécurité existantes intégrées à SQL Database. La protection de vos données dans SQL Database porte sur deux catégories de données :

- Vos données au repos dans les fichiers journaux et de données.
- Vos données en transit

Dans SQL Database, par défaut, vos données au repos dans les fichiers journaux et de données du sous-système de stockage sont entièrement et toujours chiffrées par le biais de [Transparent Data Encryption [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql). Vos sauvegardes sont également chiffrées. Avec TDE, aucune modification n’est exigée du côté de l’application qui accède à ces données. Le chiffrement et le déchiffrement se produisent de manière transparente, d’où le nom.
Pour protéger vos données sensibles en transit et au repos, SQL Database propose une fonctionnalité appelée [Always Encrypted (AE)](/sql/relational-databases/security/encryption/always-encrypted-database-engine). AE est une forme de chiffrement côté client qui permet de chiffrer les colonnes sensibles de votre base de données (elles sont en texte chiffré pour les administrateurs de base de données et les utilisateurs non autorisés). Au départ, le serveur reçoit les données chiffrées. La clé de chiffrement Always Encrypted est également stockée côté client, pour permettre uniquement aux clients autorisés de déchiffrer les colonnes sensibles. Les administrateurs du serveur et des données ne peuvent pas voir les données sensibles, car les clés de chiffrement sont stockées sur le client. AE chiffre les colonnes sensibles de la table, de bout en bout, depuis les clients non autorisés jusqu’au disque physique. AE prend en charge les comparaisons d’égalité, ce qui permet aux administrateurs de base de données de continuer à interroger les colonnes chiffrées quand ils exécutent des commandes SQL. Vous pouvez utiliser Always Encrypted avec diverses options de magasin de clés, par exemple [Azure Key Vault](always-encrypted-azure-key-vault-configure.md), le magasin de certificats Windows et les modules de sécurité matériels locaux.

|**Caractéristiques**|**Always Encrypted**|**Chiffrement transparent des données**|
|---|---|---|
|**Étendue de chiffrement**|Bout en bout|Données au repos|
|**Le serveur peut accéder aux données sensibles**|Non|Oui, étant donné que le chiffrement concerne les données au repos|
|**Opérations T-SQL autorisées**|Comparaison d’égalité|Toute la surface d’exposition T-SQL est disponible|
|**Modifications de l’application exigées pour utiliser la fonctionnalité**|Minimales|Très minimes|
|**Granularité de chiffrement**|Au niveau des colonnes|Au niveau de la base de données|
||||

### <a name="how-can-i-limit-access-to-sensitive-data-in-my-database"></a>Comment limiter l’accès aux données sensibles dans ma base de données

Chaque application comporte des données sensibles dans la base de données qui ont besoin d’une protection afin de ne pas être visibles par tous. Certains employés de l’organisation ont besoin de consulter ces données, tandis que d’autres ne doivent pas être en mesure de les afficher. C’est le cas des salaires des employés, par exemple. Un manager a besoin d’accéder aux informations sur les salaires de ses collaborateurs directs, tandis que les membres de son équipe ne doivent pas pouvoir accéder aux informations salariales individuelles de chacun. Un autre scénario a trait aux développeurs de données amenés à manipuler des données sensibles lors des phases de développement ou de test, par exemple, les numéros de sécurité sociale de clients. Là encore, ces informations n’ont pas besoin d’être exposées aux développeurs. Dans de telles situations, vos données sensibles ont besoin d’être masquées ou carrément non exposées. SQL Database propose deux approches permettant d’empêcher les utilisateurs non autorisés d’afficher des données sensibles :

[Dynamic Data Masking](dynamic-data-masking-overview.md) est une fonctionnalité de masquage de données qui vous permet de limiter l’exposition des données sensibles en les masquant aux utilisateurs qui n’ont aucun privilège sur la couche Application. Vous définissez une règle de masquage qui peut créer un modèle de masquage (par exemple, pour afficher uniquement les quatre derniers chiffres d’un numéro d’identification quelconque, utilisez un modèle de type XXX-XX-0000 qui permet de masquer le reste avec des X) et d’identifier les utilisateurs à exclure de la règle de masquage. Le masquage se produit à la volée et il existe diverses fonctions de masquage disponibles pour diverses catégories de données. Le masquage dynamique des données vous permet de détecter automatiquement les données sensibles dans votre base de données et de leur appliquer des masques.

La fonctionnalité [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) vous permet de contrôler l’accès au niveau des lignes. Cela signifie que certaines lignes d’une table de base de données sont masquées en fonction de l’utilisateur qui exécute la requête (par exemple, en fonction de son appartenance à un groupe ou d’un contexte d’exécution). La restriction d’accès s’effectue sur la couche Base de données au lieu de la couche Application, ce qui simplifie votre logique d’application. Vous commencez par créer un prédicat de filtre, lequel permet de filtrer les lignes à ne pas exposer, puis vous créez la stratégie de sécurité pour définir qui a accès à ces lignes. Enfin, l’utilisateur final exécute sa requête et, en fonction de ses privilèges, il peut afficher ces lignes à accès restreint ou il ne les voit pas du tout.

### <a name="how-do-i-manage-encryption-keys-in-the-cloud"></a>Comment gérer les clés de chiffrement dans le cloud

Il existe des options relatives à la gestion des clés pour Always Encrypted (chiffrement côté client) et Transparent Data Encryption (chiffrement au repos). Il est recommandé de faire tourner régulièrement les clés de chiffrement. La fréquence de rotation doit respecter à la fois la réglementation et les exigences de conformité de votre organisation interne.

#### <a name="transparent-data-encryption-tde"></a>Transparent Data Encryption (TDE)

il existe une hiérarchie à deux clés dans TDE. Les données de chaque base de données utilisateur sont chiffrées par une clé de chiffrement de base de données AES-256 symétrique (spécifique à la base de données), qui est à son tour chiffrée par une clé principale RSA 2048 asymétrique (spécifique au serveur). La clé principale peut être gérée de plusieurs façons :

- Automatiquement par la plateforme : SQL Database.
- En utilisant [Azure Key Vault](always-encrypted-azure-key-vault-configure.md) comme magasin de clés.

Par défaut, la clé principale liée à Transparent Data Encryption est gérée par le service SQL Database pour des raisons pratiques. Si votre organisation souhaite contrôler la clé principale, il est possible d'utiliser Azure Key Vault](always-encrypted-azure-key-vault-configure.md) comme magasin de clés. Avec Azure Key Vault, votre organisation contrôle le provisionnement, la rotation et les autorisations spécifiques aux clés. [La rotation ou le changement de type d’une clé principale TDE](/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql-key-rotation) demande peu de temps, car il suffit de rechiffrer la clé de chiffrement de base de données. Dans les organisations qui appliquent une séparation des rôles entre la gestion de la sécurité et celle des données, l’administrateur de la sécurité peut provisionner le matériel de clé pour la clé principale TDE dans Azure Key Vault, et fournir à l’administrateur de base de données un identificateur de clé Azure Key Vault pour le chiffrement au repos sur un serveur. Key Vault a été conçu de façon que Microsoft ne puisse pas voir ni extraire des clés de chiffrement. Vous bénéficiez également d’une gestion centralisée des clés pour votre organisation.

#### <a name="always-encrypted"></a>Always Encrypted

Il existe aussi une [hiérarchie à deux clés](/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted) dans Always Encrypted : une colonne de données sensibles est chiffrée par une clé de chiffrement de colonne AES 256, qui est à son tour chiffrée par une clé principale de colonne. Les pilotes clients fournis pour Always Encrypted n’ont pas de limites concernant la longueur des clés principales de colonne. La valeur chiffrée de la clé de chiffrement de colonne est stockée dans la base de données, et la clé principale de colonne est stockée dans un magasin de clés approuvé, par exemple le magasin de certificats Windows, Azure Key Vault ou un module de sécurité matériel.

- Vous pouvez changer [la clé de chiffrement de colonne et la clé principale de colonne](/sql/relational-databases/security/encryption/rotate-always-encrypted-keys-using-powershell).
- La rotation de la clé de chiffrement de colonne est une opération qui dépend de la taille des données ; elle peut prendre beaucoup de temps selon la taille des tables qui contiennent les colonnes chiffrées. Ainsi, il est préférable de planifier les rotations de clés de chiffrement de colonne en conséquence.
- La rotation des clés principales de colonne, quant à elle, n’interfère pas avec les performances des bases de données. Vous pouvez l’effectuer avec des rôles distincts.

Le diagramme suivant montre les options relatives au magasin de clés pour les clés principales de colonne dans Always Encrypted

![Fournisseurs de magasins de clés principales de colonne Always Encrypted](./media/manage-data-after-migrating-to-database/always-encrypted.png)

### <a name="how-can-i-optimize-and-secure-the-traffic-between-my-organization-and-sql-database"></a>Comment optimiser et sécuriser le trafic entre mon organisation et SQL Database

Le trafic réseau entre votre organisation et SQL Database est généralement acheminé par le biais du réseau public. Toutefois, si vous choisissez d’optimiser ce parcours et de le rendre plus sécurisé, vous pouvez envisager d’utiliser ExpressRoute. ExpressRoute vous permet avant tout d’étendre votre réseau d’entreprise dans la plateforme Azure par le biais d’une connexion privée. Ainsi, vous ne passez pas par l’Internet public. Vous obtenez également une sécurité, une fiabilité et une optimisation du routage accrues qui se traduisent par des latences réseau inférieures et des débits beaucoup plus rapides que ceux que vous expérimenteriez par le bais de l’Internet public en temps normal. Si vous envisagez de transférer un bloc important de données entre votre organisation et Azure, l’utilisation d’ExpressRoute peut permettre de réduire vos coûts. Vous avez trois modèles de connectivité différents à votre disposition pour la connexion entre votre organisation et Azure :

- [Colocalisation avec échange de cloud](../../expressroute/expressroute-connectivity-models.md#CloudExchange)
- [Connexion universelle](../../expressroute/expressroute-connectivity-models.md#IPVPN)
- [Connexion point à point](../../expressroute/expressroute-connectivity-models.md#Ethernet)

ExpressRoute vous permet également de doubler la limite de bande passante que vous achetez sans frais supplémentaires. Vous pouvez également configurer une connectivité interrégion à l’aide d’ExpressRoute. Pour obtenir la liste des fournisseurs de connectivité ExpressRoute, consultez : [Partenaires ExpressRoute et emplacements de peering](../../expressroute/expressroute-locations.md). Les articles suivants décrivent Express Route de façon plus approfondie :

- [Présentation d’ExpressRoute](../../expressroute/expressroute-introduction.md)
- [Composants requis](../../expressroute/expressroute-prerequisites.md)
- [Flux de travail](../../expressroute/expressroute-workflows.md)

### <a name="is-sql-database-compliant-with-any-regulatory-requirements-and-how-does-that-help-with-my-own-organizations-compliance"></a>Est-ce que SQL Database est conforme aux exigences réglementaires et comment cela peut-il répondre aux exigences de conformité de mon organisation

SQL Database est conforme à un certain nombre d’exigences réglementaires. Pour voir quels sont les derniers points de conformité respectés par SQL Database, visitez le [Centre de gestion de la confidentialité Microsoft](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) et recherchez les conformités importantes pour votre organisation. Ainsi, vous pourrez déterminer si SQL Database fait partie des services Azure conformes. Il est important de noter que même si SQL Database peut être certifié en tant que service conforme, il contribue à la conformité du service de votre organisation sans représenter pour autant une garantie automatique.

## <a name="intelligent-database-monitoring-and-maintenance-after-migration"></a>Surveillance et maintenance de bases de données intelligentes après la migration

Une fois que vous avez effectué la migration de votre base de données vers SQL Database, vous avez besoin de surveiller votre base de données (par exemple, vérifier l’utilisation des ressources ou effectuer des vérifications DBCC) et d’effectuer une maintenance régulière (par exemple, regénérer ou réorganiser les index, les statistiques, etc.). Heureusement, SQL Database est intelligent dans le sens où il utilise les tendances historiques et les métriques et statistiques enregistrées pour vous aider à surveiller et gérer de manière proactive votre base de données, afin que votre application s’exécute toujours de façon optimale. Dans certains cas, Azure SQL Database peut effectuer automatiquement des tâches de maintenance en fonction de vos paramètres de configuration. La surveillance de votre base de données revêt trois facettes dans SQL Database :

- Surveillance et optimisation des performances.
- Optimisation de la sécurité.
- Optimisation des coûts.

### <a name="performance-monitoring-and-optimization"></a>Supervision et optimisation des performances

avec Query Performance Insight, vous pouvez obtenir des recommandations sur mesure pour votre charge de travail de base de données afin que vos applications puissent continuer à s’exécuter à un niveau optimal, de façon permanente. Vous pouvez également le configurer de sorte que ces recommandations s’appliquent automatiquement et que vous n’ayez pas à vous soucier d’effectuer d’autres tâches de maintenance. Avec SQL Database Advisor, vous pouvez implémenter automatiquement des recommandations d’index en fonction de votre charge de travail : il s’agit d’un paramétrage automatique. Les recommandations évoluent au même rythme que la charge de travail de votre application pour fournir des suggestions pertinentes. Vous avez également la possibilité de passer manuellement en revue ces recommandations et de les appliquer à votre convenance.  

### <a name="security-optimization"></a>Optimisation de la sécurité

SQL Database fournit des recommandations de sécurité pratiques pour vous aider à sécuriser vos données. De plus, la détection des menaces vous permet d'identifier et d'analyser les activités de base de données suspectes qui constituent une menace potentielle pour la base de données. L'[évaluation des vulnérabilités](sql-vulnerability-assessment.md) est un service d'analyse et de rapport de base de données qui vous permet de surveiller l'état de sécurité de vos bases de données à grande échelle, et d'identifier les risques de sécurité et les dérives par rapport à la base de référence que vous avez définie pour la sécurité. Après chaque analyse, une liste personnalisée d’étapes à entreprendre et de scripts de correction est proposée, ainsi qu’un rapport d’évaluation qui peut vous aider à répondre aux exigences de conformité.

Avec Azure Security Center, vous identifiez les recommandations en matière de sécurité dans le tableau de bord et vous les appliquez en un seul clic.

### <a name="cost-optimization"></a>Optimisation des coûts

la plateforme Azure SQL analyse l’historique d’utilisation de toutes les bases de données d’un serveur pour évaluer et vous recommander des options d’optimisation des coûts. Cette analyse prend généralement une quinzaine de jours pour générer des recommandations actionnables. Le pool élastique est l’une de ces options. La recommandation apparaît dans le portail sous forme de bannière :

![recommandations de pool élastique](./media/manage-data-after-migrating-to-database/elastic-pool-recommendations.png)

Vous pouvez également afficher cette analyse sous la section « Advisor » :

![recommandations de pool élastique - Advisor](./media/manage-data-after-migrating-to-database/advisor-section.png)

### <a name="how-do-i-monitor-the-performance-and-resource-utilization-in-sql-database"></a>Comment superviser les performances et l’utilisation des ressources dans SQL Database

Dans SQL Database, vous pouvez exploiter les analyses intelligentes de la plateforme pour surveiller les performances et les ajuster en conséquence. Vous pouvez surveiller les performances et l’utilisation des ressources dans SQL Database à l’aide des méthodes suivantes :

#### <a name="azure-portal"></a>Portail Azure

Le portail Azure montre l’utilisation d’une base de données lorsque vous la sélectionnez et que vous cliquez sur le graphique du volet Vue d’ensemble. Vous pouvez modifier le graphique pour afficher plusieurs métriques, notamment le pourcentage d’UC, le pourcentage de DTU, le pourcentage d’E/S de données, le pourcentage de sessions et le pourcentage de taille de base de données.

![Graphique de surveillance](./media/manage-data-after-migrating-to-database/monitoring-chart.png)

![Graphique de surveillance2](./media/manage-data-after-migrating-to-database/chart.png)

À partir de ce graphique, vous pouvez également configurer des alertes par ressource. Ces alertes vous permettent de réagir à certaines situations spécifiques aux ressources par l’envoi d’un e-mail, d’écrire sur un point de terminaison HTTPS/HTTP ou d’effectuer une action. Pour plus d’informations, consultez [Créer des alertes](alerts-insights-configure-portal.md).

#### <a name="dynamic-management-views"></a>Vues de gestion dynamique (DMV)

vous pouvez interroger la vue de gestion dynamique [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) pour retourner l’historique des statistiques de consommation des ressources de la dernière heure, et la vue de catalogue système [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) pour retourner l’historique des 14 derniers jours.

#### <a name="query-performance-insight"></a>Query Performance Insight

[Query Performance Insight](query-performance-insight-use.md) vous permet de voir l’historique des requêtes les plus consommatrices de ressources et des requêtes les plus longues pour une base de données spécifique. Vous pouvez rapidement identifier les requêtes principales par utilisation des ressources, durée et fréquence d’exécution. Vous pouvez effectuer le suivi des requêtes et détecter une régression. Cette fonctionnalité nécessite l’activation du [Magasin des requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) pour la base de données.

![Query Performance Insight](./media/manage-data-after-migrating-to-database/query-performance-insight.png)

#### <a name="azure-sql-analytics-preview-in-azure-monitor-logs"></a>Azure SQL Analytics (préversion) dans les journaux d'activité Azure Monitor

Les [journaux Azure Monitor](../../azure-monitor/insights/azure-sql.md) vous permettent de collecter et de visualiser des indicateurs de performance clés Azure SQL Database, prenant en charge jusqu'à 150 000 bases de données et 5 000 pools élastiques SQL par espace de travail. Vous pouvez l’utiliser à des fins de surveillance et pour recevoir des notifications. Vous pouvez surveiller les métriques de SQL Database et des pools élastiques de plusieurs abonnements Azure. De plus, vous pouvez utiliser les pools élastiques pour identifier les problèmes à chaque couche d’une pile d’applications.

### <a name="i-am-noticing-performance-issues-how-does-my-sql-database-troubleshooting-methodology-differ-from-sql-server"></a>Je constate des problèmes de performances : En quoi ma méthodologie de résolution de problèmes SQL Database est-elle différente de celle de SQL Server

La majorité des techniques de résolution de problèmes que vous utiliseriez pour diagnostiquer des problèmes de performance des requêtes et des bases de données restent les mêmes. Après tout, c'est le même moteur de base de données qui alimente le cloud. Toutefois, la plateforme Azure SQL Database est dotée d'une « intelligence » intégrée. Celle-ci peut vous aider à résoudre et détecter les problèmes de performance encore plus facilement. Elle peut également effectuer certaines actions correctives à votre place et dans certains cas, résoudre les problèmes automatiquement de façon proactive.

Votre approche de la résolution des problèmes de performance peut considérablement tirer parti des fonctionnalités intelligentes comme [Query Performance Insight (QPI)](query-performance-insight-use.md) et [Database Advisor](database-advisor-implement-performance-recommendations.md) utilisées conjointement ; c’est là que la méthodologie diffère car vous n’avez plus besoin d’examiner manuellement les détails essentiels pouvant vous aider à résoudre le problème. La plateforme fait le travail difficile à votre place. La fonctionnalité QPI en est l’exemple. En effet, elle vous permet d’explorer jusqu’au niveau de la requête et d’examiner les tendances historiques pour déterminer à quel moment précis la requête a régressé. Database Advisor vous fournit des recommandations visant à améliorer les performances globales, par exemple, sur les index manquants, les suppressions d’index, le paramétrage de vos requêtes, etc.

Avec la résolution des problèmes de performance, il est important de déterminer si c’est uniquement l’application ou aussi la base de données qui l’accompagne, qui affectent les performances de votre application. Le problème de performance se trouve souvent dans la couche Application. Il peut être lié à l’architecture ou au modèle d’accès aux données. Supposez, par exemple, que vous avez une application bavarde sensible à la latence du réseau. Ainsi, votre application souffre, car de nombreuses requêtes courtes vont et viennent (dues à ce bavardage intense) entre l’application et le serveur et, sur un réseau encombré, ces allers-retours s’accumulent rapidement. Pour améliorer les performances dans cet exemple, vous pouvez utiliser un [traitement par lot des requêtes](performance-guidance.md#batch-queries). L’utilisation de lots vous aide considérablement, car vos requêtes sont alors traitées dans un lot ; ainsi, vous réduisez la latence des allers-retours et vous améliorez les performances de votre application.

De plus, si vous constatez une dégradation des performances globales de votre base de données, vous pouvez surveiller les vues de gestion dynamique [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) et [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) pour comprendre comment sont consommées l’UC, les E/S et la mémoire. Vos performances sont peut-être affectées parce que votre base de données est sous-alimentée en ressources. Dans ce cas, vous devrez peut-être changer la taille de calcul ou le niveau de service en fonction de l’augmentation ou de la baisse du nombre de demandes de charge de travail.

Pour obtenir un ensemble complet de recommandations pour résoudre les problèmes de performance, consultez : [Optimiser votre base de données](performance-guidance.md#tune-your-database).

### <a name="how-do-i-ensure-i-am-using-the-appropriate-service-tier-and-compute-size"></a>Comment savoir si j’utilise un niveau de service et une taille de calcul adaptés

SQL Database propose plusieurs niveaux de service : De base, Standard et Premium. Chaque niveau de service vous permet d’obtenir les performances prévisibles garanties qui lui sont associées. En fonction de votre charge de travail, vous pouvez avoir des pics d’activité qui font atteindre à l’utilisation de vos ressources le plafond de votre taille de calcul. Le cas échéant, commencez judicieusement par déterminer si un paramétrage peut résoudre le problème (par exemple, en ajoutant ou en modifiant un index, etc.). Si vous rencontrez toujours des problèmes de limite, passez à un niveau de service ou à une taille de calcul supérieurs.

|**Niveau de service**|**Scénarios de cas d’usage courants**|
|---|---|
|**De base**|Applications avec une poignée d’utilisateurs et une base de données sans besoins de concurrence, de mise à l’échelle et de performances élevés. |
|**Standard**|Applications qui ont des besoins de concurrence, de mise à l’échelle et de performances considérables associés à des demandes d’E/S faibles à moyennes. |
|**Premium**|Applications avec un grand nombre d’utilisateurs simultanés, des besoins d’UC/mémoire élevés et des demandes d’E/S élevées. Les applications à concurrence élevée, à débit élevé et sensibles à la latence peuvent exploiter le niveau Premium. |
|||

Pour vérifier que votre taille de calcul est adaptée, vous pouvez superviser votre consommation des ressources de requête et de base de données de l’une des manières mentionnées ci-dessus dans « Comment faire pour surveiller les performances et l’utilisation des ressources dans SQL Database ? ». Si vous trouvez que vos requêtes ou vos bases de données s’approchent constamment des limites d’UC/mémoire, passez à une taille de calcul supérieure. De même, si vous remarquez que vous n’utilisez pas autant de ressources que vous le pensiez pendant vos heures de pointe, vous pouvez passer à une taille de calcul inférieure.

Si vous utilisez un modèle d’application SaaS ou un scénario de consolidation de bases de données, vous pouvez utiliser un pool élastique pour optimiser les coûts. Un pool élastique est un excellent moyen pour consolider des bases de données et optimiser les coûts. Pour en savoir plus sur la gestion de plusieurs bases de données à l’aide d’un pool élastique, consultez : [Gérer des pools et des bases de données](elastic-pool-manage.md#azure-portal).

### <a name="how-often-do-i-need-to-run-database-integrity-checks-for-my-database"></a>À quelle fréquence ai-je besoin d’exécuter les vérifications de l’intégrité de ma base de données

SQL Database utilise des techniques intelligentes pour gérer certaines classes d’altération des données automatiquement et sans perte de données. Ces techniques sont intégrées au service et exploitées par lui au moment opportun. Régulièrement, vos sauvegardes de base de données sont testées sur le service en effectuant leur restauration et en exécutant DBCC CHECKDB. SQL Database traite les problèmes de façon proactive le cas échéant. La [réparation de page automatique](/sql/sql-server/failover-clusters/automatic-page-repair-availability-groups-database-mirroring) est exploitée pour corriger les pages endommagées ou présentant des problèmes d’intégrité des données. Les pages de base de données sont toujours vérifiées avec le paramètre CHECKSUM par défaut qui vérifie l’intégrité de la page. SQL Database surveille et examine l’intégrité des données de votre base de données de manière proactive et, si des problèmes surgissent, les traite avec la priorité la plus élevée. De plus, vous pouvez éventuellement choisir d’exécuter vos propres vérifications d’intégrité à votre convenance.  Pour plus d’informations, consultez [Intégrité des données dans SQL Database](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/).

## <a name="data-movement-after-migration"></a>Mouvement des données après la migration

### <a name="how-do-i-export-and-import-data-as-bacpac-files-from-sql-database-using-the-azure-portal"></a>Exporter et importer des données sous forme de fichiers BACPAC à partir de SQL Database via le portail Azure

- **Exporter** : vous pouvez exporter votre base de données Azure SQL Database en tant que fichier BACPAC à partir du portail Azure.

   ![exportation de base de données](./media/manage-data-after-migrating-to-database/database-export1.png)

- **Importer** : vous pouvez aussi importer des données sous forme de fichier BACPAC dans votre base de données Azure SQL Database à partir du portail Azure.

   ![importation de base de données](./media/manage-data-after-migrating-to-database/import1.png)

### <a name="how-do-i-synchronize-data-between-sql-database-and-sql-server"></a>Comment synchroniser des données entre SQL Database et SQL Server

Il existe plusieurs méthodes :

- La fonctionnalité **[Data Sync](sql-data-sync-data-sql-server-sql-database.md)** vous permet de synchroniser les données de façon bidirectionnelle entre plusieurs bases de données SQL Server et SQL Database. Pour synchroniser des bases de données SQL Server, vous devez installer et configurer l'agent de synchronisation sur un ordinateur local ou une machine virtuelle, et ouvrir le port TCP sortant 1433.
- La fonctionnalité **[Réplication transactionnelle](https://azure.microsoft.com/blog/transactional-replication-to-azure-sql-database-is-now-generally-available/)** vous permet de synchroniser vos données d'une base de données SQL Server vers Azure SQL Database, le serveur de publication étant l'instance SQL Server et l'abonné étant Azure SQL Database. Pour l’instant, seule cette configuration est prise en charge. Pour plus d'informations sur la migration de vos données d'une base de données SQL Server vers Azure SQL avec un temps d'arrêt minimal, consultez : [Utiliser la réplication transactionnelle](migrate-to-database-from-sql-server.md#method-2-use-transactional-replication)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [SQL Database](sql-database-paas-overview.md).