---
title: Vue d’ensemble de la sécurité
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Découvrez la sécurité dans Azure SQL Database et Azure SQL Managed Instance, notamment sur les différences avec SQL Server.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto, emlisa
ms.date: 10/26/2020
ms.openlocfilehash: 39119f62fa938f5f4f6529539d4ca9a84bdf8fd7
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94989188"
---
# <a name="an-overview-of-azure-sql-database-and-sql-managed-instance-security-capabilities"></a>Vue d’ensemble des capacités de sécurité d’Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Cet article décrit les bases de la sécurisation de la couche Données d'une application à l'aide d'[Azure SQL Database](sql-database-paas-overview.md), d'[Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et d'[Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). La stratégie de sécurité décrite suit une approche en couches de défense en profondeur comme illustré ci-après, et se déplace de l’extérieur vers :

![Diagramme de défense en profondeur par couches. Les données client sont encapsulées dans des couches de sécurité réseau, de gestion des accès et de protection des informations et contre les menaces.](./media/security-overview/sql-security-layer.png)

## <a name="network-security"></a>Sécurité du réseau

Microsoft Azure SQL Database, SQL Managed Instance et Azure Synapse Analytics fournissent un service de base de données relationnelle pour les applications cloud et d'entreprise. Pour protéger les données client, les pare-feu empêchent tout accès réseau au serveur, sauf octroi d’accès explicite en fonction de l’adresse IP ou de l’origine du trafic de réseau virtuel Azure.

### <a name="ip-firewall-rules"></a>Règles de pare-feu IP

Les règles de pare-feu IP octroient l’accès à la base de données en fonction de l’adresse IP d’origine de chaque requête. Pour plus d’informations, consultez [Vue d’ensemble des règles de pare-feu Azure SQL Database et Azure Synapse Analytics](firewall-configure.md).

### <a name="virtual-network-firewall-rules"></a>Règles de pare-feu de réseau virtuel

Les [points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md) étendent la connectivité de votre réseau virtuel sur la dorsale principale d’Azure et permettent à Azure SQL Database d’identifier le sous-réseau de réseau virtuel d’où provient le trafic. Pour permettre au trafic d'atteindre Azure SQL Database, utilisez les [balises de service](../../virtual-network/network-security-groups-overview.md) SQL et autorisez le trafic sortant via les groupes de sécurité réseau.

Les [règles de réseau virtuel](vnet-service-endpoint-rule-overview.md) permettent à Azure SQL Database de n’accepter que les communications provenant de sous-réseaux sélectionnés à l’intérieur d’un réseau virtuel.

> [!NOTE]
> Le contrôle d’accès avec des règles de pare-feu ne s’applique *pas* à **SQL Managed Instance**. Pour plus d’informations sur la configuration réseau requise, consultez [Connexion à une instance managée](../managed-instance/connect-application-instance.md).

## <a name="access-management"></a>Gestion de l’accès

> [!IMPORTANT]
> La gestion des bases de données et des serveurs dans Azure est contrôlée par les attributions de rôle de votre compte d’utilisateur du portail. Pour en savoir plus à ce sujet, consultez [Contrôle d’accès en fonction du rôle Azure dans le portail Azure](../../role-based-access-control/overview.md).

### <a name="authentication"></a>Authentification

L’authentification est le processus consistant à prouver que l’utilisateur est bien celui qu’il prétend être. Azure SQL Database et SQL Managed Instance prennent en charge deux types d’authentification :

- **Authentification SQL** :

    L’authentification SQL fait référence à l’authentification d’un utilisateur qui se connecte à Azure SQL Database ou Azure SQL Managed Instance à l’aide d’un nom d’utilisateur et d’un mot de passe. Un identifiant d’**administrateur de serveur** avec un nom d’utilisateur et un mot de passe doivent être spécifiés lors de la création du serveur. À l’aide de ces informations d’identification, un **administrateur de serveur** peut s’authentifier auprès de n’importe quelle base de données sur ce serveur ou cette instance en tant que propriétaire de la base de données. Ensuite, des connexions SQL et utilisateurs supplémentaires peuvent être créés par l'administrateur de serveur, ce qui permet aux utilisateurs de se connecter à l'aide d'un nom d'utilisateur et d'un mot de passe.

- **Authentification Azure Active Directory** :

    L’authentification Azure Active Directory est un mécanisme qui sert à se connecter à [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) à l’aide d’identités se trouvant dans Azure Active Directory (Azure AD). L’authentification Azure AD permet aux administrateurs de gérer de manière centralisée les identités et les autorisations des utilisateurs de base de données et d’autres services Azure dans un emplacement centralisé. Cela permet de réduire le stockage des mots de passe et d'activer des stratégies de rotation centralisée des mots de passe.

     Un administrateur de serveur appelé **administrateur Active Directory** doit être créé pour utiliser l’authentification Azure AD avec SQL Database. Pour plus d’informations, consultez [Connexion à SQL Database avec l’authentification Azure Active Directory](authentication-aad-overview.md). L’authentification Azure AD prend en charge les comptes managés et fédérés. Les comptes fédérés prennent en charge les utilisateurs et groupes Windows pour un domaine de client fédéré avec Azure AD.

    Les autres options d’authentification Azure AD disponibles correspondent aux connexions d'[authentification universelle Active Directory pour SQL Server Management Studio](authentication-mfa-ssms-overview.md), avec [authentification multifacteur](../../active-directory/authentication/concept-mfa-howitworks.md) et [accès conditionnel](conditional-access-configure.md).

> [!IMPORTANT]
> La gestion des bases de données et des serveurs dans Azure est contrôlée par les attributions de rôle de votre compte d’utilisateur du portail. Pour en savoir plus à ce sujet, consultez [Contrôle d’accès en fonction du rôle dans le portail Azure](../../role-based-access-control/overview.md). Le contrôle d’accès avec des règles de pare-feu ne s’applique *pas* à **SQL Managed Instance**. Consultez l’article suivant sur la [connexion à une instance gérée](../managed-instance/connect-application-instance.md) pour en savoir plus sur la configuration réseau nécessaire.

## <a name="authorization"></a>Autorisation

L’autorisation fait référence aux autorisations accordées à un utilisateur au sein d’une base de données dans Azure SQL Database ou Azure SQL Managed Instance, et détermine ce que l’utilisateur est autorisé à faire. Les autorisations sont contrôlées en ajoutant des comptes d’utilisateurs aux [rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles) et en attribuant des autorisations au niveau de la base de données à ces rôles ou en octroyant aux utilisateurs certaines [autorisations au niveau des objets](/sql/relational-databases/security/permissions-database-engine). Pour plus d’informations, consultez [Connexions et utilisateurs](logins-create-manage.md).

Il est recommandé de créer des rôles personnalisés si nécessaire. Ajoutez des utilisateurs au rôle doté du niveau de privilèges le moins élevé pour qu’ils remplissent leur fonction. N’attribuez pas d’autorisations directement aux utilisateurs. Le compte d’administrateur de serveur est un membre du rôle db_owner intégré. Il est doté d’autorisations étendues et ne doit être octroyé qu’à quelques utilisateurs ayant des charges administratives. Pour les applications, utilisez la clause [EXECUTE AS](/sql/t-sql/statements/execute-as-clause-transact-sql) pour spécifier le contexte d’exécution du module appelé ou utilisez des [rôles d’application](/sql/relational-databases/security/authentication-access/application-roles) avec des autorisations limitées. Cette pratique permet de s’assurer que l’application qui se connecte à la base de données dispose du moins de privilèges possible. L’application de ces meilleures pratiques favorise également la séparation des tâches.

### <a name="row-level-security"></a>Sécurité au niveau des lignes

La sécurité au niveau des lignes permet aux clients de contrôler l'accès aux lignes d'une table de base de données en fonction des caractéristiques de l'utilisateur exécutant une requête (appartenance à un groupe ou contexte d'exécution, par exemple). La sécurité au niveau des lignes permet également d’implémenter des concepts de sécurité personnalisés en fonction d’une étiquette. Pour plus d’informations, consultez [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security).

![Diagramme montrant que Sécurité au niveau des lignes protège les lignes individuelles d’une base de données SQL contre l’accès par des utilisateurs via une application cliente.](./media/security-overview/azure-database-rls.png)

## <a name="threat-protection"></a>Protection contre les menaces

SQL Database et SQL Managed Instance sécurisent les données des clients en fournissant des capacités d’audit et de détection des menaces.

### <a name="sql-auditing-in-azure-monitor-logs-and-event-hubs"></a>Audit SQL dans les journaux d’activité Azure Monitor et dans Event Hubs

L’audit SQL Database et SQL Managed Instance suit les activités de base de données et permet d’assurer la conformité avec les normes de sécurité en enregistrant les événements de base de données dans un journal d’audit d’un compte de stockage Azure appartenant au client. L’audit permet aux utilisateurs de surveiller les activités de base de données en cours, d’analyser et d’examiner l’historique des activités pour identifier les menaces potentielles ou les violations de sécurité et abus présumés. Pour plus d'informations, consultez [Prise en main de l’audit SQL Database](../../azure-sql/database/auditing-overview.md).  

### <a name="advanced-threat-protection"></a>Protection avancée contre les menaces

Advanced Threat Protection analyse vos journaux afin de détecter les éventuels comportements inhabituels ou les tentatives d’accès ou d’exploitation des bases de données potentiellement dangereuses. Des alertes sont créées pour les activités suspectes, telles que les attaques par injection de code SQL, infiltration potentielle de données et force brute, ainsi que pour les anomalies des modèles d’accès afin d’intercepter les réaffectations de privilèges et l’utilisation d’informations d’identification ayant fait l’objet de violation de sécurité. Les alertes s’affichent dans [Azure Security Center](https://azure.microsoft.com/services/security-center/), où les détails relatifs aux activités suspectes, les suggestions d’examen approfondi ainsi que les actions visant à atténuer les menaces sont indiqués. Le service Advanced Threat Protection peut être activé par serveur pour un coût supplémentaire. Pour en savoir plus, consultez [Prise en main de Advanced Threat Protection pour Azure SQL Database](threat-detection-configure.md).

![Diagramme montrant la détection de menace SQL surveillant l’accès à la base de données SQL pour une application web depuis un attaquant externe et une personne interne malveillante.](./media/security-overview/azure-database-td.jpg)

## <a name="information-protection-and-encryption"></a>Protection et chiffrement des informations

### <a name="transport-layer-security-encryption-in-transit"></a>Transport Layer Security (Chiffrement en transit)

SQL Database, SQL Managed Instance et Azure Synapse Analytics sécurisent les données des clients en utilisant le protocole [Transport Layer Security (TLS)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) pour chiffrer les données en mouvement.

SQL Database, SQL Managed Instance et Azure Synapse Analytics appliquent en permanence le chiffrement (SSL/TLS) aux connexions. De cette façon, toutes les données sont chiffrées « en transit » entre le client et le serveur, quel que soit le paramètre **Encrypt** ou **TrustServerCertificate** de la chaîne de connexion.

Dans la chaîne de connexion utilisée par l’application, nous vous recommandons de spécifier une connexion chiffrée et de ne _**pas**_ approuver le certificat de serveur. L’application étant contrainte de vérifier le certificat de serveur, elle n’est donc pas vulnérable aux attaques de type intercepteur.

Par exemple, lorsque vous utilisez le pilote ADO.NET, cette opération s’effectue via **Encrypt=True** et **TrustServerCertificate=False**. Si vous obtenez votre chaîne de connexion à partir du portail Azure, elle présentera les paramètres appropriés.

> [!IMPORTANT]
> Notez que certains pilotes autres que Microsoft peuvent ne pas utiliser le protocole TLS par défaut ou en utiliser une ancienne version (< 1.2) pour fonctionner. Dans ce cas, le serveur vous permet toujours de vous connecter à votre base de données. Toutefois, nous vous recommandons d’évaluer les risques de sécurité si vous autorisez ces pilotes et cette application à se connecter à SQL Database, en particulier si vous stockez des données sensibles.
>
> Pour plus d’informations sur le protocole TLS et la connectivité, consultez [Considérations relatives au protocole TLS](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

### <a name="transparent-data-encryption-encryption-at-rest"></a>Transparent Data Encryption (chiffrement des données au repos)

[Transparent Data Encryption (TDE) pour SQL Database, SQL Managed Instance et Azure Synapse Analytics](transparent-data-encryption-tde-overview.md) ajoute une couche de sécurité pour protéger les données au repos contre tout accès non autorisé ou hors connexion aux fichiers bruts ou aux sauvegardes. Parmi les scénarios courants, citons le vol de centre de données ou l’élimination non sécurisée de matériel ou de supports tels que les lecteurs de disque et les bandes de sauvegarde. La fonctionnalité TDE chiffre l’intégralité de la base de données à l’aide d’un algorithme de chiffrement AES, sans que les développeurs d’applications aient besoin de modifier les applications existantes.

Dans Azure, toutes les bases de données créées sont chiffrées par défaut, et la clé de chiffrement de ces bases de données est protégée par un certificat de serveur intégré.  La maintenance et la rotation des certificats sont gérées par le service, sans intervention de l’utilisateur. Les clients peuvent également gérer eux-mêmes les clés de chiffrement dans [Azure Key Vault](../../key-vault/general/secure-your-key-vault.md).

### <a name="key-management-with-azure-key-vault"></a>Gestion des clés dans Azure Key Vault

[Bring Your Own Key](transparent-data-encryption-byok-overview.md) (BYOK) prenant en charge la fonctionnalité  [Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption) (TDE), les clients peuvent assurer eux-mêmes la gestion et la rotation des clés à l’aide d’ [Azure Key Vault](../../key-vault/general/secure-your-key-vault.md), le système informatique de gestion des clés externes d’Azure. Si l’accès de la base de données au coffre de clés est révoqué, une base de données ne peut pas être déchiffrée et lue en mémoire. Azure Key Vault propose une plateforme de gestion centrale des clés, utilise des modules de sécurité matériels étroitement surveillés (HSM) et permet la séparation des responsabilités entre la gestion de clés et des données pour aider à répondre aux exigences de conformité en matière de sécurité.

### <a name="always-encrypted-encryption-in-use"></a>Always Encrypted (Chiffrement en cours d’utilisation)

![Diagramme montrant les concepts de base de la fonctionnalité Always Encrypted. Une base de données SQL avec un verrou est accessible uniquement par une application contenant une clé.](./media/security-overview/azure-database-ae.png)

[Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) est une fonctionnalité conçue pour protéger les données sensibles stockées dans des colonnes de base de données spécifiques (numéros de cartes de crédit, de carte d'identité ou autres données _sensibles_). Les administrateurs de base de données ou autres utilisateurs disposant de privilèges autorisés peuvent accéder à la base de données afin d'y effectuer des tâches de gestion, sans devoir accéder aux données contenues dans les colonnes chiffrées. Les données sont systématiquement chiffrées, ce qui signifie qu'elles sont uniquement déchiffrées à des fins de traitement par les applications clientes ayant accès à la clé de chiffrement. La clé de chiffrement n’est jamais exposée dans SQL Database ou SQL Managed Instance, et peut être stockée dans le [magasin de certificats Windows](always-encrypted-certificate-store-configure.md) ou dans [Azure Key Vault](always-encrypted-azure-key-vault-configure.md).

### <a name="dynamic-data-masking"></a>Masquage dynamique des données

![Diagramme montrant le masquage dynamique des données. Une application métier envoie des données à une base de données SQL qui masque les données avant de les renvoyer à l’application métier.](./media/security-overview/azure-database-ddm.png)

Le masquage dynamique des données limite l’exposition des données sensibles en les masquant aux utilisateurs sans privilège. Le masquage des données dynamiques détecte automatiquement les données potentiellement sensibles dans Azure SQL Database et SQL Managed Instance et fournit des recommandations pouvant donner lieu à une action permettant de masquer ces champs, avec un impact minimal sur la couche d’application. Il dissimule les données sensibles dans le jeu de résultats d’une requête dans les champs de la base de données désignés. Les données de la base de données ne sont pas modifiées. Pour en savoir plus, consultez [Prise en main du masquage de données dynamiques dans SQL Database et SQL Managed Instance](dynamic-data-masking-overview.md).

## <a name="security-management"></a>Gestion de la sécurité

### <a name="vulnerability-assessment"></a>Évaluation des vulnérabilités

[L’évaluation des vulnérabilités ](sql-vulnerability-assessment.md) est un service simple à configurer, qui vous permet de découvrir, suivre et vous aide à corriger des vulnérabilités de base de données potentielles, avec pour objectif d’améliorer de manière proactive la sécurité globale des bases de données. L’évaluation des vulnérabilités (VA) fait partie de l’offre Azure Defender pour SQL, qui est un package unifié pour les fonctionnalités avancées de sécurité SQL. L’évaluation des vulnérabilités est accessible et gérée par le biais de l’Azure Defender central pour le portail SQL.

### <a name="data-discovery-and-classification"></a>Découverte et classification des données

La découverte et classification des données (actuellement en préversion) offre des capacités avancées intégrées à Azure SQL Database et SQL Managed Instance pour la découverte, la classification, l’étiquetage et la protection des données sensibles dans vos bases de données. La découverte et la classification de vos données les plus sensibles (professionnelles/financières, soins de santé, données personnelles, etc.) peuvent jouer un rôle essentiel dans la protection des informations de l’organisation. Elles peuvent servir d’infrastructure pour :

- Divers scénarios de sécurité, comme la surveillance (audit) et la génération d’alertes en cas d’accès anormaux aux données sensibles.
- Contrôler l’accès et renforcer la sécurité des bases de données contenant des données sensibles.
- Aider à répondre aux normes de confidentialité des données et aux exigences de conformité aux normes.

Pour plus d’informations, consultez [Bien démarrer avec Découverte et classification des données](data-discovery-and-classification-overview.md).

### <a name="compliance"></a>Conformité

Non seulement Azure SQL Database propose les fonctions ci-dessus et des fonctionnalités permettant à votre application de répondre à différentes exigences en matière de sécurité, mais il participe également à des audits réguliers et est certifié conforme à de nombreuses normes actuelles. Pour en savoir plus, accédez au [Centre de confidentialité Microsoft Azure](https://www.microsoft.com/trust-center/compliance/compliance-overview), qui inclut la liste la plus à jour des certifications de conformité de SQL Database.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’utilisation de connexions, de comptes d’utilisateur, de rôles de base de données et d’autorisations dans SQL Database et SQL Managed Instance, consultez [Gérer les connexions et les comptes d’utilisateurs](logins-create-manage.md).
- Pour une présentation de l’audit de base de données, consultez la section [Audit](../../azure-sql/database/auditing-overview.md).
- Pour une description de la détection des menaces, consultez la rubrique [Détection des menaces](threat-detection-configure.md).