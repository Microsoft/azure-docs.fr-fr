---
title: Playbook pour répondre aux exigences de sécurité courantes
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Cet article présente les exigences de sécurité courantes et les bonnes pratiques dans Azure SQL Database et Azure SQL Managed Instance.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=2
author: VanMSFT
ms.author: vanto
ms.topic: article
ms.date: 09/21/2020
ms.reviewer: ''
ms.openlocfilehash: 1217d3af855e96b6d6a0f403c2ff351a6b957d9a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459664"
---
# <a name="playbook-for-addressing-common-security-requirements-with-azure-sql-database-and-azure-sql-managed-instance"></a>Playbook pour répondre aux exigences de sécurité courantes avec Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cet article fournit les bonnes pratiques relatives au traitement des exigences de sécurité courantes. Toutes les exigences ne s'appliquent pas aux différents environnements, et vous devez consulter votre équipe de sécurité et de base de données sur les fonctionnalités à implémenter.

## <a name="solving-common-security-requirements"></a>Répondre aux exigences de sécurité courantes

Ce document fournit des conseils sur la façon de répondre à des exigences de sécurité courantes en lien avec des applications nouvelles ou existantes utilisant Azure SQL Database et Azure SQL Managed Instance. Il s’articule autour de grands axes en matière de sécurité. Pour résoudre des menaces spécifiques, voir la section [Menaces de sécurité courantes et atténuations potentielles](#common-security-threats-and-potential-mitigations). Bien que certaines des recommandations présentées soient applicables lors de la migration d’applications locales vers Azure, les scénarios de migration ne sont pas l’objet de ce document.

### <a name="azure-sql-database-deployment-offers-covered-in-this-guide"></a>Offres de déploiement d’Azure SQL Database traitées dans ce guide

- [Azure SQL Database](./index.yml) : [bases de données uniques](single-database-overview.md) et [pools élastiques](elastic-pool-overview.md) sur des [serveurs](logical-servers.md)
- [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md)

### <a name="deployment-offers-not-covered-in-this-guide"></a>Offres de déploiement non traitées dans ce guide

- Azure Synapse Analytics
- Machines virtuelles Azure SQL (IaaS)
- SQL Server

### <a name="audience"></a>Public visé

Ce guide vise le public des clients qui sont confrontés à des questions sur la sécurisation d’Azure SQL Database. Les rôles qu’intéresse cet article sur les meilleures pratiques sont notamment les suivants :

- Architectes de sécurité
- Gestionnaires de sécurité
- Responsables de la mise en conformité
- Responsables de la confidentialité
- Ingénieurs de la sécurité

### <a name="using-this-guide"></a><a id="using"></a> Comment utiliser ce guide

Ce document accompagne notre documentation existante sur la [sécurité d’Azure SQL Database](security-overview.md).

Sauf indication contraire, nous vous recommandons de suivre toutes les meilleures pratiques décrites dans chaque section pour atteindre l’objectif ou satisfaire l’exigence en question. Pour respecter des normes de conformité ou des pratiques recommandées en matière de sécurité, des contrôles de conformité réglementaires importants sont énumérés dans la section Exigences ou Objectifs, le cas échéant. Voici les normes et réglementations de sécurité référencées dans ce document :

- [FedRAMP](https://www.fedramp.gov/documents/) : AC-04, AC-06
- [SOC](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/sorhome.html) : CM-3, SDL-3
- [ISO/IEC 27001](https://www.iso27001security.com/html/27001.html) : Contrôle d’accès, chiffrement
- [Pratiques de Microsoft Operational Security Assurance (OSA)](https://www.microsoft.com/securityengineering/osa/practices) : Pratique #1-6 et #9
- [Contrôles de sécurité de NIST Special Publication 800-53](https://nvd.nist.gov/800-53) : AC-5, AC-6
- [PCI DSS](https://www.pcisecuritystandards.org/document_library) : 6.3.2, 6.4.2

Nous prévoyons de continuer d’actualiser les recommandations et meilleures pratiques énumérées ici. Fournissez vos contributions ou corrections en lien avec ce document en suivant le lien **Commentaires** au bas de cet article.

## <a name="authentication"></a>Authentification

L’authentification est le processus consistant à prouver que l’utilisateur est bien celui qu’il prétend être. Azure SQL Database et SQL Managed Instance prennent en charge deux types d’authentification :

- Authentification SQL
- Authentification Azure Active Directory

> [!NOTE]
> L’authentification Azure Active Directory peut ne pas être prise en charge pour tous les outils et applications tierces.

### <a name="central-management-for-identities"></a>Gestion centralisée des identités

La gestion centralisée des identités offre les avantages suivants :

- Gérer les comptes de groupe et contrôler les autorisations des utilisateurs sans dupliquer les connexions à des serveurs, bases de données et instances managées.
- Gestion des autorisations simplifiée et flexible.
- Gestion des applications à grande échelle.

**Mode d’implémentation** :

- Utilisez l’authentification Azure Active Directory (Azure AD) pour la gestion centralisée des identités.

**Bonnes pratiques** :

- Créez un locataire Azure AD, [créez des utilisateurs](../../active-directory/fundamentals/add-users-azure-active-directory.md) pour représenter des utilisateurs humains et créez des [principaux de service](../../active-directory/develop/app-objects-and-service-principals.md) pour représenter des applications, des services et des outils d’automatisation. Les principaux de service sont équivalents aux comptes de service dans Windows et Linux.

- Attribuez des droits d’accès à des ressources à des principaux Azure AD par le biais d’une affectation de groupe : Créez des groupes de Azure AD, accordez l’accès à des groupes et ajouter des membres aux groupes. Dans votre base de données, créez des utilisateurs de base de données autonome qui mappent vos groupes Azure AD. Pour attribuer des autorisations à l’intérieur de la base de données, placez les utilisateurs associés à vos groupes Azure AD dans les rôles de base de données avec les autorisations appropriées.
  - Consultez les articles [Configurer et gérer l’authentification Azure Active Directory avec SQL](authentication-aad-configure.md) et [Utiliser Azure AD pour l’authentification auprès de SQL](authentication-aad-overview.md).
  > [!NOTE]
  > Dans SQL Managed Instance, vous pouvez également créer des connexions qui mappent à des principaux Azure AD dans la base de données MASTER. Voir [CREATE LOGIN (Transact-SQL)](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current).

- L’utilisation de groupes Azure AD simplifie la gestion des autorisations. Tant le propriétaire du groupe que le propriétaire de la ressource peuvent ajouter/supprimer des membres du groupe.

- Créez un groupe distinct pour les administrateurs Azure AD de chaque serveur ou instance managée.

  - Consultez l’article [Provisionner un administrateur Azure Active Directory pour votre serveur](authentication-aad-configure.md#provision-azure-ad-admin-sql-database).

- Surveillez les changements d’appartenance de groupe Azure AD à l’aide de rapports d’activité d’audit Azure AD.

- Pour une instance managée, une étape distincte est requise pour créer un administrateur Azure AD.
  - Voir l’article [Approvisionner un administrateur d’Azure Active Directory pour votre instance gérée](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

> [!NOTE]
>
> - L’authentification Azure AD est enregistrée dans des journaux d’audit d’Azure SQL, mais pas dans les journaux d’activité de connexion d’Azure AD.
> - Les autorisations Azure RBAC accordées dans Azure ne s’appliquent pas aux autorisations Azure SQL Database ni SQL Managed Instance. Ces autorisations doivent être créées/mappées manuellement à l’aide d’autorisations SQL existantes.
> - Côté client, l’authentification Azure AD a besoin d’accéder à Internet ou à un réseau virtuel par le biais d’une route définie par l’utilisateur.
> - Le jeton d’accès Azure AD est mis en cache côté client et sa durée de vie dépend de sa configuration. Voir l’article [Durées de vie des jetons configurables dans Azure Active Directory](../../active-directory/develop/active-directory-configurable-token-lifetimes.md)
> - Pour obtenir des conseils afin de résoudre les problèmes liés à Azure AD Authentication, consultez le blog suivant : [Résolution des problèmes liés à Azure AD](https://techcommunity.microsoft.com/t5/azure-sql-database/troubleshooting-problems-related-to-azure-ad-authentication-with/ba-p/1062991).

### <a name="azure-ad-multi-factor-authentication"></a>Authentification multifacteur Azure AD

> Mentionné dans : OSA Practice #2, ISO Access Control (AC)

Azure AD Multi-Factor Authentication contribue à renforcer la sécurité en exigeant plus d’une forme d’authentification.

**Mode d’implémentation** :

- [Activez Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md) dans Azure AD en utilisant un accès conditionnel et utilisez une authentification interactive.

- L’alternative consiste à activer Multi-Factor Authentication pour l’ensemble du domaine AD ou Azure AD.

**Bonnes pratiques** :

- Activez l’accès conditionnel dans Azure AD (nécessite un abonnement Premium).
  - Voir l’article [Accès conditionnel dans Azure AD](../../active-directory/conditional-access/overview.md).  

- Créez des groupes Azure AD et activez la stratégie Multi-Factor Authentication pour les groupes sélectionnés en utilisant l’accès conditionnel Azure AD.
  - Voir l’article [Planifier un déploiement d’accès conditionnel](../../active-directory/conditional-access/plan-conditional-access.md).

- Multi-Factor Authentication peut être activé pour l’ensemble de l’annuaire Azure AD ou pour l’ensemble de l’annuaire Active Directory fédéré avec Azure AD.

- Utilisez le mode d’authentification interactive Azure AD pour Azure SQL Database et Azure SQL Managed Instance où un mot de passe est demandé de manière interactive, suivi de l’authentification multifacteur :
  - Utilisez une authentification universelle dans SSMS. Consultez l’article [Utilisation de l’authentification Azure AD multifacteur avec Azure SQL Database, SQL Managed Instance et Azure Synapse (prise en charge de SSMS pour Multi-Factor Authentication)](authentication-mfa-ssms-overview.md).
  - Utilisez une authentification interactive prise en charge dans SQL Server Data Tools (SSDT). Voir l’article [Prise en charge d’Azure Active Directory dans SQL Server Data Tools (SSDT)](/sql/ssdt/azure-active-directory?view=azuresqldb-current).
  - Utilisez d’autres outils SQL prenant en charge Multi-Factor Authentication.
    - Prise en charge par l’Assistant SSMS pour l’exportation, l’extraction et le déploiement de base de données  
    - [sqlpackage.exe](/sql/tools/sqlpackage) : option ‘/ua’
    - [Utilitaire sqlcmd](/sql/tools/sqlcmd-utility) : option -G (interactive)
    - [utilitaire bcp](/sql/tools/bcp-utility) : option -G (interactive)

- Implémentez vos applications pour qu’elles se connectent à Azure SQL Database ou Azure SQL Managed Instance à l’aide d’une authentification interactive avec prise en charge de Multi-Factor Authentication.
  - Voir l’article [Se connecter à Azure SQL Database avec Azure AD Multi-Factor Authentication](active-directory-interactive-connect-azure-sql-db.md).
  > [!NOTE]
  > Ce mode d’authentification nécessite des identités basées sur l’utilisateur. Dans les cas où le modèle d’identité approuvé utilisé ignore l’authentification d’utilisateur Azure AD individuel (par exemple en cas d’utilisation d’une identité managée pour les ressources Azure), Multi-Factor Authentication ne s’applique pas.

### <a name="minimize-the-use-of-password-based-authentication-for-users"></a>Réduire l’utilisation de l’authentification par mot de passe pour les utilisateurs

> Mentionné dans : OSA Practice #4, ISO Access Control (AC)

Les méthodes d’authentification par mot de passe constituent une forme d’authentification plus faible. Des informations d’identification peuvent être compromises ou fournies par erreur.

**Mode d’implémentation** :

- Utilisez l’authentification Azure AD intégrée qui élimine l’utilisation de mots de passe.

**Bonnes pratiques** :

- Utilisez l’authentification unique à l’aide d’informations d’identification Windows. Fédérez le domaine AD local avec Azure AD et utilisez l’authentification Windows intégrée (pour les machines jointes à un domaine avec Azure AD).
  - Voir l’article [Prise en charge de SSMS pour l’authentification intégrée Azure AD](authentication-aad-configure.md#active-directory-integrated-authentication).

### <a name="minimize-the-use-of-password-based-authentication-for-applications"></a>Réduire l’utilisation de l’authentification par mot de passe pour les applications

> Mentionné dans : OSA Practice #4, ISO Access Control (AC)

**Mode d’implémentation** :

- Activez l’identité managée d’Azure. Vous pouvez également utiliser une authentification intégrée ou basée sur des certificats.

**Bonnes pratiques** :

- Utilisez des [identités managées pour des ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).
  - [Identité managée affectée par le système](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql.md)
  - [Identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)
  - [Utiliser Azure SQL Database à partir d’Azure App Service avec une identité managée (sans changement de code)](https://github.com/Azure-Samples/app-service-msi-entityframework-dotnet)

- Utilisez une authentification basée sur des certificats pour une application.
  - Voir cet [exemple de code](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth/token).

- Utilisez l’authentification Azure AD pour un domaine fédéré intégré et une machine jointe à un domaine (voir la section ci-dessus).
  - Voir l’[exemple d’application pour l’authentification intégrée](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth/integrated).

### <a name="protect-passwords-and-secrets"></a>Protéger les mots de passe et les secrets

Dans les cas où les mots de passe sont inévitables, assurez-vous qu’ils sont sécurisés.

**Mode d’implémentation** :

- Utilisez Azure Key Vault pour stocker les mots de passe et les secrets. Le cas échéant, utilisez Multi-Factor Authentication pour Azure SQL Database avec des utilisateurs Azure AD.

**Bonnes pratiques** :

- S’il n’est pas possible d’éviter l’usage de mots de passe ou de secrets, stockez les mots de passe d’utilisateurs et les secrets d’applications dans Azure Key Vault, et gérez l’accès via des stratégies d’accès à Key Vault.

- Diverses infrastructures de développement d’applications peuvent également offrir des mécanismes spécifiques de l’infrastructure pour protéger les secrets dans l’application. Par exemple : [ASP.NET core app](/aspnet/core/security/app-secrets?tabs=windows&view=aspnetcore-2.1).

### <a name="use-sql-authentication-for-legacy-applications"></a>Activer l’authentification directe pour les applications héritées

L’authentification SQL fait référence à l’authentification d’un utilisateur qui se connecte à Azure SQL Database ou SQL Managed Instance à l’aide d’un nom d’utilisateur et d’un mot de passe. Une connexion doit être créée dans chaque serveur ou instance managée, et un utilisateur doit être créé dans chaque base de données.

**Mode d’implémentation** :

- Utilisez l’authentification SQL.

**Bonnes pratiques** :

- En tant qu’administrateur de serveur ou d’instance, créez des connexions et des utilisateurs. À moins d’utiliser des utilisateurs de base de données autonome avec des mots de passe, tous les mots de passe sont stockés dans la base de données master.
  - Consultez l’article [Contrôle et autorisation d’accès aux bases de données à SQL Database, SQL Managed Instance et Azure Synapse Analytics](logins-create-manage.md).

## <a name="access-management"></a>Gestion de l’accès

La gestion des accès (également appelée Autorisation) est le processus de contrôle et de gestion des accès et privilèges des utilisateurs autorisés relatifs à Azure SQL Database ou SQL Managed Instance.

### <a name="implement-principle-of-least-privilege"></a>Appliquer le principe du privilège minimum

> Mentionné dans : FedRamp controls AC-06, NIST : AC-6, OSA Practice #3

Le principe du privilège minimum stipule que les utilisateurs ne doivent pas avoir plus de privilèges que nécessaire pour accomplir leurs tâches. Pour plus d’informations, voir l’article [Just Enough Administration](/powershell/scripting/learn/remoting/jea/overview).

**Mode d’implémentation** :

Attribuez uniquement les [autorisations](/sql/relational-databases/security/permissions-database-engine) nécessaires pour accomplir les tâches requises :

- Dans les bases de données SQL :
  - Utilisez des autorisations précises et des rôles de base de données définis par l’utilisateur (ou des rôles de serveur dans Managed Instance) :
    1. Créez les rôles requis
       - [CREATE ROLE](/sql/t-sql/statements/create-role-transact-sql)
       - [CREATE SERVER ROLE](/sql/t-sql/statements/create-server-role-transact-sql)
    1. Créez les utilisateurs requis
       - [CREATE USER](/sql/t-sql/statements/create-user-transact-sql)
    1. Ajoutez des utilisateurs en tant que membres à des rôles
       - [ALTER ROLE](/sql/t-sql/statements/alter-role-transact-sql)
       - [ALTER SERVER ROLE](/sql/t-sql/statements/alter-server-role-transact-sql)
    1. Affectez ensuite des autorisations aux rôles.
       - [GRANT](/sql/t-sql/statements/grant-transact-sql)
  - Veillez à ne pas affecter d’utilisateurs à des rôles inutiles.

- Dans Azure Resource Manager :
  - Utilisez des rôles intégrés s’il y en a qui sont disponibles ou des rôles personnalisés Azure, et attribuez les autorisations nécessaires.
    - [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md)
    - [Rôle personnalisés Azure](../../role-based-access-control/custom-roles.md)

**Bonnes pratiques** :

Les meilleures pratiques suivantes sont facultatives, mais entraînent une meilleure capacité de gestion et de prise en charge de votre stratégie de sécurité :

- Si vous le pouvez, commencez avec l’ensemble d’autorisations le plus petit possible, et ajoutez des autorisations une à une si elles sont véritablement nécessaires (et justifiées), par opposition à l’approche inverse consistant à retirer progressivement des autorisations.

- Évitez d’attribuer des autorisations à des utilisateurs individuels. Utilisez plutôt des rôles (de base de données ou de serveur) de manière cohérente. Les rôles facilitent considérablement la création de rapports et la résolution de problèmes d’autorisation. (Azure RBAC prend uniquement en charge l’attribution d’autorisations via des rôles).

- Créez et utilisez des rôles personnalisés avec les autorisations spécifiques nécessaires. Rôles typiques utilisés dans la pratique :
  - Déploiement de sécurité
  - Administrateur
  - Développeur
  - Personnel de support
  - Auditeur
  - Processus automatisés
  - Utilisateur final
  
- Utilisez uniquement des rôles intégrés lorsque les autorisations des rôles correspondent exactement aux autorisations nécessaires pour l’utilisateur. Vous pouvez affecter des utilisateurs à plusieurs rôles.

- N’oubliez pas que les autorisations dans le moteur de base de données peuvent être appliquées aux étendues suivantes (plus l’étendue est petite, plus l’impact des autorisations accordées est restreint) :
  - Serveur (rôles spéciaux dans la base de données MASTER) dans Azure
  - Base de données
  - schéma
    - Il est recommandé d’utiliser des schémas pour accorder des autorisations à l’intérieur d’une base de données. (voir aussi : [Conception de schéma : recommandations pour la conception de schéma dans une optique de sécurité](http://andreas-wolter.com/en/schema-design-for-sql-server-recommendations-for-schema-design-with-security-in-mind/))
  - Objet (table, vue, procédure, etc.)

  > [!NOTE]
  > Il n’est pas recommandé d’appliquer des autorisations au niveau objet, car celui-ci ajoute une complexité superflue à l’implémentation globale. Si vous décidez d’utiliser des autorisations au niveau objet, celles-ci doivent être clairement documentées. Il en va de même des autorisations au niveau colonne qui sont encore moins recommandées pour les mêmes raisons. Sachez également que, par défaut, une autorisation au niveau table [DENY](/sql/t-sql/statements/deny-object-permissions-transact-sql) ne remplace pas une autorisation au niveau colonne GRANT. Cela nécessiterait l’activation de la [configuration du serveur de conformité des critères communs](/sql/database-engine/configure-windows/common-criteria-compliance-enabled-server-configuration-option).

- Effectuez des vérifications régulières à l’aide d’une [évaluation des vulnérabilités (VA)](/sql/relational-databases/security/sql-vulnerability-assessment) pour tester l’existence d’un nombre excessif d’autorisations.

### <a name="implement-separation-of-duties"></a>Implémenter une séparation des tâches

> Mentionné dans : FedRamp : AC-04, NIST : AC-5, ISO : A.6.1.2, PCI 6.4.2, SOC : CM-3, SDL-3

La séparation des tâches, également appelée répartition des tâches, décrit l’exigence de fractionner les tâches sensibles en plusieurs tâches affectées à différents utilisateurs. La séparation des tâches aide à éviter les violations de données.

**Mode d’implémentation** :

- Identifiez le niveau requis de séparation des tâches. Exemples :
  - Entre environnements de développement/test et de production
  - Tâches sensibles sur le plan de la sécurité, tâches au niveau gestion de l’administrateur de base de données (DBA) et tâches de développeur.
    - Exemples : Auditeur, création de stratégie de sécurité pour la sécurité au niveau du rôle, implémentation d’objets SQL Database avec des autorisations DDL.

- Identifiez une hiérarchie complète d’utilisateurs (et de processus automatisés) qui accèdent au système.

- Créez des rôles en fonction des groupes d’utilisateurs nécessaires et attribuez des autorisations aux rôles.
  - Pour les tâches au niveau gestion dans le portail Azure ou via l’automatisation PowerShell, utilisez des rôles Azure. Vous pouvez soit trouver un rôle intégré correspondant à la spécification, soit créer un rôle personnalisé Azure utilisant des autorisations disponibles.
  - Créez des rôles de serveur pour les tâches à l’échelle du serveur (création de nouvelles connexions, bases de données) dans une instance gérée.
  - Créez des rôles de base de données pour les tâches de niveau base de données.

- Pour certaines tâches sensibles, pensez à créer des procédures stockées spéciales signées par un certificat pour exécuter les tâches pour le compte des utilisateurs. L’un des principaux avantages des procédures stockées signées numériquement est que si la procédure est modifiée, les autorisations qui ont été accordées à la version précédente de la procédure sont immédiatement supprimées.
  - Exemple : [Tutoriel : Signature de procédures stockées à l’aide d’un certificat](/sql/relational-databases/tutorial-signing-stored-procedures-with-a-certificate)

- Implémentez la technologie Transparent Data Encryption (TDE) avec des clés gérées par le client dans Azure Key Vault pour permettre une séparation des tâches entre le propriétaire des données et le propriétaire de la sécurité.
  - Voir l’article [Configurer les clés gérées par le client pour le chiffrement du Stockage Azure depuis le Portail Azure](../../storage/common/customer-managed-keys-configure-key-vault.md).

- Pour être certain qu’un administrateur de bases de données ne puisse pas voir des données considérées comme hautement sensibles et que vous puissiez toujours effectuer des tâches d’administrateur de bases de données, vous pouvez utiliser la fonctionnalité Always Encrypted avec la séparation des rôles.
  - Voir les articles [Vue d’ensemble de la gestion des clés pour Always Encrypted](/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted), [Mise en service des clés avec séparation des rôles](/sql/relational-databases/security/encryption/configure-always-encrypted-keys-using-powershell#KeyProvisionWithRoles) et [Permutation des clés principales de colonne avec séparation des rôles](/sql/relational-databases/security/encryption/rotate-always-encrypted-keys-using-powershell#column-master-key-rotation-with-role-separation).

- Dans les cas où l’utilisation d’Always Encrypted n’est pas possible, du moins sans consentir des coûts et des efforts qui sont même susceptibles de rendre le système inutilisable, des compromis peuvent être effectués et atténués par des contrôles de compensation tels que :
  - Intervention humaine dans les processus.
  - Pistes d’audit : pour plus d’informations sur l’audit, voir [Auditer des événements de sécurité critiques](#audit-critical-security-events).

**Bonnes pratiques** :

- Assurez-vous que des comptes distincts sont utilisés pour les environnements de développement/test et de production. L’usage de comptes distincts permet de respecter l’exigence de séparation des systèmes de test et de production.

- Évitez d’attribuer des autorisations à des utilisateurs individuels. Utilisez plutôt des rôles (de base de données ou de serveur) de manière cohérente. L’utilisation de rôles facilite considérablement la création de rapports et la résolution des problèmes d’autorisations.

- Utilisez des rôles intégrés lorsque les autorisations sont exactement celles qui sont nécessaires : si l’union de toutes les autorisations de plusieurs rôles intégrés permet d’obtenir une correspondance à 100 %, vous pouvez également affecter plusieurs rôles simultanément.

- Créez et utilisez des rôles définis par l’utilisateur quand les rôles intégrés accordent trop ou pas assez d’autorisations.

- Les attributions de rôles peuvent également être effectuées temporairement. Cela s’appelle également la séparation dynamique des tâches (DSD) dans des étapes de travail de l’agent SQL dans T-SQL ou en utilisant Azure PIM pour les rôles Azure.

- Assurez-vous que les administrateurs de base de données n’ont pas accès aux clés de chiffrement ou aux magasins de clés, et que les administrateurs de sécurité ayant accès aux clés n’ont pas accès à la base de données. L’utilisation de la [gestion de clés extensible (EKM)](/sql/relational-databases/security/encryption/extensible-key-management-ekm) peut faciliter cette séparation. [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) peut être utilisé pour implémenter la gestion de clés extensible (EKM).

- Veillez à toujours disposer d’une piste d’audit pour les actions liées à la sécurité.

- Vous pouvez récupérer la définition des rôles intégrés Azure pour voir les autorisations utilisées et créer un rôle personnalisé basé sur des extraits et cumuls de ceux-ci via PowerShell.

- Comme tout membre du rôle de base de données db_owner peut modifier des paramètres de sécurité tels que la technologie Transparent Data Encryption (TDE) ou modifier le SLO, cette appartenance doit être accordée avec précaution. Toutefois, il existe de nombreuses tâches qui demandent des privilèges db_owner, telles que les tâches consistant à changer un paramètre de base de données (à l’image des options de base de données). L’audit joue un rôle clé dans toute solution.

- Il n’est pas possible de restreindre les autorisations d’un db_owner et, donc, d’empêcher un compte d’administrateur de voir les données utilisateur. S’il existe des données très sensibles dans une base de données, la fonctionnalité Always Encrypted permet d’empêcher en toute sécurité des db_owners ou tout autre administrateur de base de données de les consulter.

> [!NOTE]
> L’accomplissement de la séparation des tâches (SoD) est difficile pour les tâches liées à la sécurité ou à la résolution des problèmes. D’autres aspects tels que les rôles de développement et d’utilisateur final sont plus faciles à séparer. La plupart des contrôles liés à la conformité autorisent l’utilisation de fonctions de contrôle alternatives, telles que l’audit lorsque d’autres solutions ne sont pas praticables.

Aux lecteurs qui souhaitent approfondir la séparation des tâches, nous recommandons de consulter les ressources suivantes :

- Pour Azure SQL Database et SQL Managed Instance :  
  - [Contrôle et autorisation d’accès aux bases de données](logins-create-manage.md)
  - [Séparation des tâches pour le développeur d’applications](/previous-versions/sql/sql-server-2008/cc974525(v=sql.100))
  - [Séparation des tâches](https://www.microsoft.com/download/details.aspx?id=39269)
  - [Signature de procédures stockées](/dotnet/framework/data/adonet/sql/signing-stored-procedures-in-sql-server)

- Pour Gestion des ressources Azure :
  - [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md)
  - [Rôle personnalisés Azure](../../role-based-access-control/custom-roles.md)
  - [Utilisation d’Azure AD Privileged Identity Management pour l’accès avec élévation de privilèges](https://www.microsoft.com/itshowcase/using-azure-ad-privileged-identity-management-for-elevated-access)

### <a name="perform-regular-code-reviews"></a>Effectuer des révisions de code régulières

> Mentionné dans : PCI : 6.3.2, SOC : SDL-3

La séparation des tâches n’est pas limitée aux données d’une base de données. Elle inclut également le code de l’application. Un code malveillant peut contourner des contrôles de sécurité. Avant de déployer du code personnalisé en production, il est essentiel d’examiner ce qui est déployé.

**Mode d’implémentation** :

- Utilisez un outil de base de données tel qu’Azure Data Studio, qui prend en charge le contrôle de code source.

- Implémentez un processus de déploiement de code séparé.

- Avant de passer à la branche principale, une personne (autre que l’auteur du code) doit inspecter le code pour identifier des risques d’élévation de privilèges, ainsi que des modifications de données malveillantes afin de protéger contre les fraudes et les accès non autorisés. Cela peut être effectué à l’aide de mécanismes de contrôle de code source.

**Bonnes pratiques** :

- Standardisation : Elle facilite l’implémentation d’une procédure standard qui doit être suivie pour toute mise à jour de code.

- L’évaluation des vulnérabilités inclut des règles qui vérifient l’existence d’autorisations excessives, l’utilisation d’anciens algorithmes de chiffrement et d’autres problèmes de sécurité au sein d’un schéma de base de données.

- Des contrôles supplémentaires peuvent être effectués dans un environnement d’assurance qualité ou de test à l’aide d’Advanced Threat Protection qui analyse le code vulnérable à une injection de code SQL.

- Voici des exemples de ce que vous pouvez rechercher :
  - Création d’un utilisateur ou modification des paramètres de sécurité à partir d’un déploiement de mise à jour de code SQL automatisé.
  - Procédure stockée, qui, selon les paramètres fournis, met à jour une valeur monétaire dans une cellule d’une manière non conforme.

- Assurez-vous que la personne effectuant la révision n’est pas l’auteur du code d’origine et qu’elle est compétente en matière de révision du code et de sécurisation du codage.

- Veillez à connaître toutes les sources de changement de code. Le code peut se trouver dans des scripts T-SQL. Il peut s’agir de commandes ad hoc à exécuter ou à déployer sous forme de vues, de fonctions, de déclencheurs et de procédures stockées. Il peut faire partie de définitions de travaux de SQL Agent (étapes). Il peut également être exécuté à partir de packages SSIS, d’Azure Data Factory et d’autres services.

## <a name="data-protection"></a>Protection de données

La protection des données est un ensemble de fonctionnalités permettant de protéger des informations importantes de toute compromission par chiffrement ou obfuscation.

> [!NOTE]
> Microsoft atteste qu’Azure SQL Database et SQL Managed Instance sont conformes au niveau 1 de la norme FIPS 140-2. Cette attestation est faite après vérification de l’utilisation stricte des algorithmes acceptables de niveau 1 de la norme FIPS 140-2 et des instances validées de niveau 1 de la norme FIPS 140-2 de ces algorithmes, notamment la cohérence avec les longueurs de clé requises, la gestion des clés, la génération des clés et le stockage des clés. L’attestation est destinée à permettre à nos clients de répondre au besoin ou à la nécessité d’utiliser des instances validées conformément au niveau 1 de la norme FIPS 140-2 pour le traitement de données ou la livraison de systèmes ou d’applications. Nous définissons les termes « conforme au niveau 1 de la norme FIPS 140-2 » et « conformité au niveau 1 de la norme FIPS 140-2 » utilisés dans l’instruction ci-dessus pour démontrer leur applicabilité conformément à l’usage aux États-Unis et au Canada des termes « FIPS 140-2 Level 1 validated ».

### <a name="encrypt-data-in-transit"></a>Chiffrer les données en transit

> Mentionné dans : OSA Practice #6, ISO Control Family : Chiffrement

Protège vos données pendant leur déplacement entre votre client et votre serveur. Voir [Sécurité réseau](#network-security).

### <a name="encrypt-data-at-rest"></a>Chiffrer des données au repos

> Mentionné dans : OSA Practice #6, ISO Control Family : Chiffrement

Le chiffrement au repos est la protection par chiffrement des données conservées dans des fichiers de base de données, de journal et de sauvegarde.

**Mode d’implémentation** :

- Le [chiffrement transparent de base de données (TDE)](transparent-data-encryption-tde-overview.md) avec des clés gérées par le service est activé par défaut pour toutes les bases de données créées après 2017 dans Azure SQL Database et SQL Managed Instance.
- Dans une instance managée, si la base de données est créée à partir d’une opération de restauration à l’aide d’un serveur local, le paramètre TDE de la base de données d’origine est respecté. Si la technologie TDE n’est pas activée pour la base de données d’origine, nous recommandons de l’activer manuellement pour l’instance managée.

**Bonnes pratiques** :

- Ne stockez pas les données nécessitant un chiffrement au repos dans la base de données MASTER. La base de données master ne peut pas être chiffrée avec la technologie TDE.

- Utilisez des clés gérées par le client dans Azure Key Vault si vous avez besoin de davantage de transparence et d’un contrôle précis de la protection TDE. Azure Key Vault permet de révoquer des autorisations à tout moment pour rendre la base de données inaccessible. Vous pouvez gérer de manière centralisée les protecteurs TDE ainsi que d’autres clés, ou faire pivoter le protecteur TDE selon votre propre planification à l’aide d’Azure Key Vault.

- Si vous utilisez des clés gérées par le client dans Azure Key Vault, voir les articles [Instructions de configuration de TDE avec Azure Key Vault](transparent-data-encryption-byok-overview.md#recommendations-when-configuring-akv) et [Comment configurer la récupération d’urgence avec Azure Key Vault](transparent-data-encryption-byok-overview.md#geo-dr-and-customer-managed-tde).

### <a name="protect-sensitive-data-in-use-from-high-privileged-unauthorized-users"></a>Protéger les données sensibles en cours d’utilisation d’utilisateurs non autorisés disposant de privilèges élevés

Les données utilisées sont celles stockées dans la mémoire du système de base de données lors de l’exécution de requêtes SQL. Si votre base de données stocke des données sensibles, votre organisation peut être amenée à s’assurer que les utilisateurs disposant de privilèges élevés ne peuvent pas voir ces données. Les utilisateurs disposant de privilèges élevés, tels que les opérateurs Microsoft ou les administrateurs de base de données de votre organisation, doivent pouvoir gérer la base de données, mais ne pas être en mesure de voir ni, éventuellement, d’exfiltrer des données sensibles de la mémoire du processus SQL ou en interrogeant la base de données.

Les stratégies déterminant les données sensibles et si celles-ci doivent être chiffrées en mémoire et non accessibles aux administrateurs en texte en clair, sont spécifiques à votre organisation et aux réglementations de conformité qu'il vous faut respecter. Veuillez consulter l'exigence correspondante : [Identifier et baliser les données sensibles](#identify-and-tag-sensitive-data).

**Mode d’implémentation** :

- Utilisez le [chiffrement intégral](/sql/relational-databases/security/encryption/always-encrypted-database-engine) pour vous assurer que les données sensibles ne sont pas exposées en texte en clair dans Azure SQL Database ni SQL Managed Instance, même si elles sont en mémoire ou en cours d’utilisation. Always Encrypted protège les données d’administrateurs de base de données (DBA) et d’administrateurs du cloud (ou d’acteurs malveillants susceptibles d’emprunter l’identité d’utilisateurs disposant de privilèges élevés mais non autorisés), et vous permet de mieux contrôler qui peut accéder à vos données.

**Bonnes pratiques** :

- Always Encrypted ne remplace pas le chiffrement des données au repos (TDE) ou en transit (SSL/TLS). Always Encrypted ne doit pas être utilisé pour les données non sensibles afin de réduire l’impact sur les performances et les fonctionnalités. L’utilisation de la fonctionnalité Always Encrypted conjointement avec TDE et TLS (Transport Layer Security) est recommandée pour assurer une protection complète des données au repos, en transit et en cours d’utilisation.

- Évaluez l’impact du chiffrement des colonnes de données sensibles identifiées avant de déployer Always Encrypted dans une base de données de production. En général, Always Encrypted réduit la fonctionnalité des requêtes sur les colonnes chiffrées, entre autres limitations répertoriées dans [Always Encrypted - Détails de la fonctionnalité](/sql/relational-databases/security/encryption/always-encrypted-database-engine#feature-details). Dès lors, vous serez peut-être amené à devoir recréer l’architecture de votre application pour réimplémenter la fonctionnalité, sans prise en charge de requête, côté client ou/et refactoriser votre schéma de base de données, y compris les définitions des procédures stockées, fonctions, vues et déclencheurs. Les applications existantes peuvent ne pas fonctionner avec des colonnes chiffrées si elles ne respectent pas les restrictions et limitations Always Encrypted. Bien que l’écosystème des outils, produits et services de Microsoft prenant en charge Always Encrypted se développe, tous ne sont pas compatibles avec les colonnes chiffrées. Le chiffrement d’une colonne peut aussi avoir un impact sur les performances des requêtes, selon les caractéristiques de votre charge de travail.

- Gérez les clés de la fonctionnalité Always Encrypted avec une séparation des rôles si vous utilisez cette fonctionnalité pour protéger les données d’administrateurs de bases de données malveillants. Avec une séparation des rôles, un administrateur de la sécurité crée les clés physiques. L’administrateur de base de données crée des objets métadonnées de clé principale de colonne et de clé de chiffrement de colonne décrivant les clés physiques dans la base de données. Au cours de ce processus, l’administrateur de la sécurité n’a pas besoin d’accéder à la base de données, et l’administrateur de base de données n’a pas besoin d’accéder aux clés physiques en texte clair.
  - Pour plus d’informations, voir l’article [Gestion des clés avec séparation des rôles](/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted#managing-keys-with-role-separation).

- Stockez vos clés principales de colonne dans Azure Key Vault pour faciliter la gestion. Évitez d’utiliser un magasin de certificats Windows (et, en général, des solutions de magasin de clés distribuée, par opposition à des solutions de gestion des clés centralisée) qui complique la gestion des clés.

- Réfléchissez soigneusement aux inconvénients liés à l’utilisation de plusieurs clés (clé principale de colonne ou clés de chiffrement de colonne). Veillez à conserver un nombre de clés réduit afin de réduire le coût de gestion des clés. Une clé principale de colonne et une clé de chiffrement de colonne par base de données sont généralement suffisantes dans des environnements dont l’état est stable (pas au milieu d’une rotation de clé). Vous pouvez avoir besoin de clés supplémentaires si vous avez des groupes d’utilisateurs différents utilisant des clés distinctes et accédant à des données distinctes.  

- Assurez une rotation des clés principales de colonne en fonction de vos exigences de conformité. Si vous devez également assurer une rotation des clés de chiffrement de colonne, songez à utiliser un chiffrement en ligne pour réduire le temps d’arrêt des applications.
  - Voir l’article [Considérations relatives à la disponibilité et aux performances](/sql/relational-databases/security/encryption/configure-column-encryption-using-powershell#performance-and-availability-considerations).

- Utilisez un chiffrement déterministe si des calculs (égalité) sur des données doivent être pris en charge. Dans le cas contraire, utilisez un chiffrement aléatoire. Évitez d’utiliser un chiffrement déterministe pour des jeux de données à faible entropie ou des jeux de données avec une distribution publiquement connue.

- Si vous craignez que des tiers accèdent à vos données légalement sans votre consentement, assurez-vous que toutes les applications et tous les outils qui ont accès aux clés et aux données en texte clair s’exécutent en dehors de Microsoft Azure Cloud Services. Sans accès aux clés, le tiers n’aura aucun moyen de déchiffrer les données, sauf à contournent le chiffrement.

- La fonctionnalité Always Encrypted ne prend pas facilement en charge l’octroi d’un accès temporaire aux clés (et aux données protégées). C’est, par exemple, le cas si vous devez partager les clés avec un administrateur de bases de données pour permettre à celui-ci d’effectuer des opérations de nettoyage sur des données sensibles et chiffrées. La seule façon de révoquer l’accès aux données de l’administrateur de base de données de façon fiable consiste à effectuer une rotation des clés de chiffrement de colonne et des clés principales de colonne protégeant les données, opération qui est coûteuse.

- Pour accéder aux valeurs en texte en clair dans des colonnes chiffrées, un utilisateur doit avoir accès à la clé principale de colonne (CMK), qui protège les colonnes et est configurée dans le magasin de clés qui la contient. L’utilisateur doit disposer des autorisations de base de données permettant d’afficher toute définition de clé principale de colonne (**VIEW ANY COLUMN MASTER KEY DEFINITION**) ainsi que toute définition de clé de chiffrement de colonne (**VIEW ANY COLUMN ENCRYPTION KEY DEFINITION**).

### <a name="control-access-of-application-users-to-sensitive-data-through-encryption"></a>Contrôler l’accès des utilisateurs de l’application à des données sensibles par le biais d’un chiffrement

Vous pouvez utiliser un chiffrement pour vous assurer que seuls des utilisateurs de l’application spécifiques ayant accès aux clés de chiffrement peuvent consulter ou mettre à jour les données.

**Mode d’implémentation** :

- Utilisez un chiffrement au niveau des cellules (CLE). Pour plus d’informations, voir l’article [Chiffrer une colonne de données](/sql/relational-databases/security/encryption/encrypt-a-column-of-data).
- Utilisez Always Encrypted, mais tenez compte de ses limites. Ces dernières sont listées ci-dessous.

**Bonnes pratiques**

Quand vous utilisez un chiffrement au niveau des cellules (CLE) :

- Contrôlez l’accès aux clés par le biais d’autorisations et de rôles SQL.

- Utilisez AES (AES 256 recommandé) pour le chiffrement des données. Des algorithmes tels que RC4, DES et TripleDES, sont dépréciés et ne doivent pas être utilisés en raison de vulnérabilités connues.

- Protégez les clés symétriques avec des clés/certificats asymétriques (non des mots de passe) pour éviter d’utiliser 3DES.

- Soyez prudent lors de la migration d’une base de données en utilisant un chiffrement au niveau des cellules via une exportation/importation (fichiers bacpac).
  - Voir l’article [Recommandations relatives à l’utilisation d’un chiffrement au niveau des cellules dans Azure SQL Database](/archive/blogs/sqlsecurity/recommendations-for-using-cell-level-encryption-in-azure-sql-database) expliquant comment éviter de perdre des clés lors de la migration de données, et fournissant d’autres conseils sur les meilleures pratiques.

Gardez à l’esprit que Always Encrypted est principalement conçu pour protéger des données sensibles en cours d’utilisation d’utilisateurs d’Azure SQL Database disposant de privilèges élevés (opérateurs cloud, administrateurs de bases de données). Voir [Protéger les données sensibles en cours d’utilisation d’utilisateurs non autorisés disposant de privilèges élevés](#protect-sensitive-data-in-use-from-high-privileged-unauthorized-users). Lors de l’utilisation de la fonctionnalité Always Encrypted pour protéger des données d’utilisateurs de l’application, gardez à l’esprit les difficultés suivantes :

- Par défaut, tous les pilotes clients Microsoft qui prennent en charge la fonctionnalité Always Encrypted gèrent un cache global (un par application) de clés de chiffrement de colonne. Une fois qu’un pilote de client acquiert une clé de chiffrement de colonne en texte clair en contactant un magasin de clés contenant une clé principale de colonne, la clé de chiffrement de colonne en texte clair est mise en cache. Cette opération complique l’isolation des données des utilisateurs d’une application multi-utilisateurs. Si votre application emprunte l’identité d’utilisateurs finaux lors de l’interaction avec un magasin de clés (tel qu’Azure Key Vault), une fois que la requête d’un utilisateur a rempli le cache avec une clé de chiffrement de colonne, une requête suivante nécessitant la même clé mais déclenchée par un autre utilisateur utilise la clé mise en cache. Le pilote n’appelle pas le magasin de clés et ne vérifie pas si le deuxième utilisateur a l’autorisation d’accéder à la clé de chiffrement de colonne. Ainsi, l’utilisateur peut voir les données chiffrées même s’il n’a pas accès aux clés. Pour obtenir l’isolation des utilisateurs dans une application multi-utilisateur, vous pouvez désactiver la mise en cache des clés de chiffrement de colonne. La désactivation de la mise en cache a un impact supplémentaire sur les performances, car le pilote doit contacter le magasin de clés pour chaque opération de chiffrement ou de déchiffrement de données.

### <a name="protect-data-against-unauthorized-viewing-by-application-users-while-preserving-data-format"></a>Protéger des données contre des affichages non autorisés par des utilisateurs de l’application tout en préservant le format des données

Une autre technique pour empêcher les utilisateurs non autorisés d’afficher des données consiste à obfusquer ou à masquer les données tout en préservant les types et les formats de données afin de s’assurer que des applications d’utilisateurs pissent continuer à gérer et à afficher les données.

**Mode d’implémentation** :

- Utilisez la fonctionnalité [Dynamic Data Masking](/sql/relational-databases/security/dynamic-data-masking) pour obfusquer les colonnes de table.

> [!NOTE]
> La fonctionnalité Always Encrypted ne fonctionne pas avec la fonctionnalité Dynamic Data Masking. Il n’est pas possible de chiffrer et de masquer la même colonne. Cela implique que vous devez hiérarchiser la protection des données en cours d’utilisation par rapport au masquage des données pour les utilisateurs de votre application via la fonctionnalité Dynamic Data Masking.

**Bonnes pratiques** :

> [!NOTE]
> La fonctionnalité Dynamic Data Masking ne peut pas être utilisée pour protéger des données d’utilisateurs disposant de privilèges élevés. Les stratégies de masquage ne s’appliquent pas aux utilisateurs disposant d’un accès administratif, tel que db_owner.

- N’autorisez pas les utilisateurs de l’application à exécuter des requêtes ad hoc (car ils pourraient contourner la fonctionnalité Dynamic Data Masking).  
  - Pour plus d’informations, voir l’article [Ignorer le masquage à l’aide de techniques d’inférence ou de force brute](/sql/relational-databases/security/dynamic-data-masking#security-note-bypassing-masking-using-inference-or-brute-force-techniques).  

- Utilisez une stratégie de contrôle d’accès appropriée (via des autorisations SQL, des rôles, des SNL) pour limiter les autorisations des utilisateurs et effectuer des mises à jour dans les colonnes masquées. La création d’un masque sur une colonne n’empêche pas les mises à jour de celle-ci. Les utilisateurs qui reçoivent des données masquées quand ils interrogent une colonne masquée peuvent mettre à jour les données s’ils disposent d’autorisations en écriture.

- Dynamic Data Masking ne conserve pas les propriétés statistiques des valeurs masquées. Cela peut avoir un impact sur les résultats des requêtes (par exemple, les requêtes contenant des prédicats de filtre ou des jointures sur les données masquées).

## <a name="network-security"></a>Sécurité du réseau

La sécurité réseau fait référence aux contrôles d’accès et aux meilleures pratiques pour sécuriser vos données en transit vers Azure SQL Database.

### <a name="configure-my-client-to-connect-securely-to-sql-databasesql-managed-instance"></a>Configurer mon client pour qu’il se connecte en toute sécurité à SQL Database/SQL Managed Instance

Bonnes pratiques pour empêcher des ordinateurs clients et des applications comportant des vulnérabilités bien connues (par exemple, l’utilisation de protocoles et de suites de chiffrement TLS plus anciens) de se connecter à Azure SQL Database et SQL Managed Instance.

**Mode d’implémentation** :

- Assurez-vous que les ordinateurs clients se connectant à Azure SQL Database et SQL Managed Instance utilisent le [protocole TLS](security-overview.md#transport-layer-security-encryption-in-transit).

**Bonnes pratiques** :

- Configurez l’ensemble de vos applications et outils pour la connexion à SQL Database avec le chiffrement activé
  - Encrypt = On, TrustServerCertificate = Off (ou équivalent avec des pilotes non-Microsoft).

- Si votre application utilise un pilote qui ne prend pas en charge le protocole TLS ou prend en charge une version antérieure de celui-ci, remplacez le pilote autant que possible. Si ce n’est pas possible, évaluez soigneusement les risques en matière de sécurité.

- Réduisez les vecteurs d’attaque exploitant des vulnérabilités dans SSL 2.0, SSL 3.0, TLS 1.0 et TLS 1.1 en les désactivant sur les ordinateurs clients qui se connectent à Azure SQL Database conformément aux [paramètres du Registre du protocole TLS](/windows-server/security/tls/tls-registry-settings#tls-10).

- Vérifiez les suites de chiffrement disponibles sur le client : [Suites de chiffrement dans TLS/SSL (SSP Schannel)](/windows/desktop/SecAuthN/cipher-suites-in-schannel). En particulier, désactivez 3DES en suivant les instructions indiquées dans [Configuration de l’ordre de la suite de chiffrement TLS](/windows-server/security/tls/manage-tls#configuring-tls-cipher-suite-order).

- Concernant Azure SQL Database et SQL Managed Instance, le chiffrement est appliqué pour les types de connexions Proxy et Redirection. Pour Azure SQL Managed Instance, utilisez le type de connexion **Proxy** (par défaut), car cela a pour effet d’appliquer le chiffrement du côté serveur. Le type de connexion **Redirect** ne prend actuellement pas en charge le chiffrement et n’est disponible que sur des connexions IP privées.

- Pour plus d’informations, consultez [Architecture de la connectivité Azure SQL Database – Stratégie de connexion](connectivity-architecture.md#connection-policy).

### <a name="minimize-attack-surface"></a>Réduire la surface d’attaque

Réduisez le nombre de fonctionnalités qui peuvent être attaquées par un utilisateur malveillant. Implémentez des contrôles d’accès réseau pour Azure SQL Database.

> Mentionné dans : OSA Practice #5

**Mode d’implémentation** :

Dans SQL Database :

- Supprimez l’autorisation d’accès aux services Azure au niveau du serveur
- Utilisez des points de terminaison de service VNet et des règles de pare-feu de réseau virtuel.
- Utilisez la fonctionnalité Liaison privée (préversion).

Dans SQL Managed Instance :

- Suivez les instructions de [Configuration requise pour le réseau](../managed-instance/connectivity-architecture-overview.md#network-requirements).

**Bonnes pratiques** :

- Restriction de l’accès à Azure SQL Database et SQL Managed Instance en se connectant sur un point de terminaison privé (par exemple, en utilisant un chemin de données privé) :
  - Une instance gérée peut être isolée à l’intérieur d’un réseau virtuel pour empêcher tout accès externe. Les applications et outils qui sont sur le même réseau virtuel ou un réseau virtuel appairé dans la même région peuvent y accéder directement. Les applications et outils qui se trouvent dans une autre région peuvent utiliser une connexion de réseau virtuel à réseau virtuel ou un peering de circuit ExpressRoute pour établir la connexion. Le client doit utiliser des groupes de sécurité réseau (NSG) pour restreindre l’accès sur le port 1433 uniquement aux ressources nécessitant un accès à une instance managée.
  - Pour SQL Database, utilisez la fonctionnalité [Liaison privée](../../private-link/private-endpoint-overview.md) qui fournit une adresse IP privée dédiée pour le serveur à l’intérieur de votre réseau virtuel. Vous pouvez également utiliser des [points de terminaison de service de réseau virtuel avec des règles de pare-feu de réseau virtuel](vnet-service-endpoint-rule-overview.md) pour restreindre l’accès à vos serveurs.
  - Les utilisateurs mobiles doivent utiliser des connexions VPN point à site pour se connecter via le chemin d’accès aux données.
  - Les utilisateurs connectés à leur réseau local doivent utiliser une connexion VPN site à site ou ExpressRoute pour se connecter via le chemin d’accès aux données.

- Vous pouvez accéder à Azure SQL Database et SQL Managed Instance en vous connectant à un point de terminaison public (par exemple, en utilisant un chemin de données public). Les meilleures pratiques suivantes doivent être prises en compte :
  - Pour un serveur dans SQL Database, utilisez des [règles de pare-feu IP](firewall-configure.md) pour limiter l’accès aux seules adresses IP autorisées.
  - Pour SQL Managed Instance, utilisez des groupes de sécurité réseau (NSG) afin de restreindre l’accès sur le port 3342 uniquement aux ressources requises. Pour plus d’informations, consultez [Utiliser une instance managée en toute sécurité avec des points de terminaison publics](../managed-instance/public-endpoint-overview.md).

> [!NOTE]
> Le point de terminaison public SQL Managed Instance n’est pas activé par défaut. Il doit être activé explicitement. Si la stratégie d’entreprise interdit l’utilisation de points de terminaison publics, utilisez tout d’abord [Azure Policy](../../governance/policy/overview.md) pour empêcher l’activation de points de terminaison publics.

- Configurez des composants de la mise en réseau Azure :
  - Suivez les [Meilleures pratiques Azure pour la sécurité réseau](../../security/fundamentals/network-best-practices.md).
  - Planifiez la configuration du réseau virtuel conformément au bonnes pratiques décrites dans le [Forum aux questions (FAQ) sur le réseau virtuel Azure](../../virtual-network/virtual-networks-faq.md) et le plan.
  - Segmentez un réseau virtuel en plusieurs sous-réseaux et affectez des ressources pour un rôle similaire au même sous-réseau (par exemple ressources frontales et principales).
  - Utilisez des [Groupes de sécurité réseau (NSG)](../../virtual-network/network-security-groups-overview.md) pour contrôler le trafic entre sous-réseaux à l’intérieur de la limite de réseau virtuel Azure.
  - Activez le service [Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) pour votre abonnement afin de surveiller le trafic réseau entrant et sortant.

### <a name="configure-power-bi-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configurer Power BI pour des connexions sécurisées à SQL Database/SQL Managed Instance

**Bonnes pratiques** :

- Pour Power BI Desktop, utilisez autant que possible le chemin d’accès aux données privées.

- Assurez-vous que Power BI Desktop se connecte à l’aide du protocole TLS 1.2 en définissant la clé de Registre sur l’ordinateur client conformément aux paramètres du Registre du [protocole TLS](/windows-server/security/tls/tls-registry-settings).

- Limitez l’accès aux données pour des utilisateurs spécifiques via une [Sécurité au niveau des lignes (SNL) avec Power BI](/power-bi/service-admin-rls).

- Pour le service Power BI, utilisez la [passerelle de données locale](/power-bi/service-gateway-onprem), en gardant à l’esprit les [limitations et considérations](/power-bi/service-gateway-deployment-guidance#installation-considerations-for-the-on-premises-data-gateway) applicables.

### <a name="configure-app-service-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configurer App Service pour des connexions sécurisées à SQL Database/SQL Managed Instance

**Bonnes pratiques** :

- Pour une application web simple, la connexion via un point de terminaison public requiert l’activation de l’option **Autoriser les services Azure**.

- [Intégrez votre application à un réseau virtuel Azure](../../app-service/web-sites-integrate-with-vnet.md) pour la connectivité du chemin d’accès aux données privées à une instance gérée. Vous pouvez également déployer une application web avec [Azure App Service Environment](../../app-service/environment/intro.md).

- Pour une application web avec App Service Environment ou intégrée à un réseau virtuel qui se connecte à une base de données dans SQL Database, vous pouvez utiliser des [points de terminaison de service de réseau virtuel et des règles de pare-feu de réseau virtuel](vnet-service-endpoint-rule-overview.md) pour limiter l’accès à partir d’un réseau virtuel et d’un sous-réseau spécifiques. Désactivez ensuite l’option **Autoriser les services Azure**. Vous pouvez également connecter App Service Environment à une instance managée dans SQL Managed Instance sur un chemin de données privé.  

- Vérifiez que votre application web est configurée conformément à la description de l’article [Bonnes pratiques pour la sécurisation des applications mobiles et web PaaS avec Azure App Service](../../security/fundamentals/paas-applications-using-app-services.md).

- Installez un [pare-feu d’applications web (WAF)](../../web-application-firewall/ag/ag-overview.md) pour protéger votre application web des attaques et vulnérabilités courantes.

### <a name="configure-azure-virtual-machine-hosting-for-secure-connections-to-sql-databasesql-managed-instance"></a>Configurer un hébergement de machine virtuelle Azure pour des connexions sécurisées à SQL Database/SQL Managed Instance

**Bonnes pratiques** :

- Utilisez une combinaison de règles d’autorisation et de refus sur les groupes de sécurité réseau de machines virtuelles Azure pour contrôler les régions accessibles à partir de la machine virtuelle.

- Assurez-vous que votre machine virtuelle est configurée conformément à la description de l’article [Meilleures pratiques de sécurité pour les charges de travail IaaS dans Azure](../../security/fundamentals/iaas.md).

- Vérifiez que toutes les machines virtuelles sont associées à un réseau virtuel et un sous-réseau spécifiques.

- Évaluez si vous avez besoin de l’itinéraire par défaut 0.0.0.0/Internet conformément aux conseils prodigués dans [À propos du tunneling forcé](../../vpn-gateway/vpn-gateway-forced-tunneling-rm.md#about-forced-tunneling).
  - Si oui (par exemple, sous-réseau frontal), conservez l’itinéraire par défaut.
  - Si non (par exemple, sous-réseau de niveau intermédiaire ou principal), activez le tunneling forcé afin qu’aucun trafic ne transite par Internet pour atteindre un site local (également appelé intersite).

- Implémentez des [itinéraires par défaut facultatifs](../../virtual-network/virtual-networks-udr-overview.md#optional-default-routes) si vous utilisez un peering ou une connexion à un site local.

- Implémentez des [routes définies par l’utilisateur](../../virtual-network/virtual-networks-udr-overview.md#user-defined) si vous devez envoyer tout le trafic du réseau virtuel à une appliance virtuelle réseau pour l’inspection des paquets.

- Utilisez des [points de terminaison de service de réseau virtuel](vnet-service-endpoint-rule-overview.md) pour un accès sécurisé à des services PaaS tels que Stockage Azure via le réseau principal Azure.

### <a name="protect-against-distributed-denial-of-service-ddos-attacks"></a>Protéger contre les attaques par déni de service distribué (DDoS)

Des attaques par déni de service distribué (DDoS) sont des tentatives effectuées par un utilisateur malveillant d’envoyer un flux de trafic réseau à Azure SQL Database dans le but de saturer l’infrastructure Azure afin qu’elle rejette des connexions et charges de travail valides.

> Mentionné dans : OSA Practice #9

**Mode d’implémentation** :

La protection contre les attaques par déni de service distribué est automatiquement activée sur la plateforme Azure. Elle inclut une surveillance permanente du trafic et l’atténuation en temps réel des attaques au niveau réseau de points de terminaison publics.

- Utilisez [Azure DDoS Protection](../../ddos-protection/ddos-protection-overview.md) pour surveiller les adresses IP publiques associées aux ressources déployées dans des réseaux virtuels.

- Utilisez [Advanced Threat Protection pour Azure SQL Database](threat-detection-overview.md) pour détecter les attaques par déni de service (DoS) dirigées contre des bases de données.

**Bonnes pratiques** :

- Suivez les pratiques décrites dans [Réduire la surface d’attaque](#minimize-attack-surface) pour réduire les menaces d’attaques DDoS.

- L’alerte **Informations d’identification SQL par force brute** d’Advanced Threat Protection vous aide à détecter les attaques par force brute. Dans certains cas, l’alerte peut même distinguer des charges de travail de tests d’intrusion.

- Pour les applications d’hébergement de machine virtuelle Azure se connectant à SQL Database :
  - Suivez les recommandations pour restreindre l’accès via des points de terminaison accessibles sur Internet dans Azure Security Center.
  - Utilisez des groupes de machines virtuelles identiques pour exécuter plusieurs instances de votre application sur des machines virtuelles Azure.
  - Désactivez les protocoles RDP et SSH à partir d’Internet pour empêcher les attaques par force brute.

## <a name="monitoring-logging-and-auditing"></a>Supervision, journalisation et audit

Cette section fait référence aux fonctionnalités destinées à vous aider à détecter des activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données. Elle décrit également les meilleures pratiques de configuration de l’audit de base de données pour suivre et capturer des événements de base de données.

### <a name="protect-databases-against-attacks"></a>Protéger les bases de données contre des attaques

La protection avancée contre les menaces vous permet de détecter et traiter des menaces potentielles à mesure qu’elles se présentent en générant des alertes de sécurité sur les activités anormales.

**Mode d’implémentation** :

- Utilisez [Advanced Threat Protection pour SQL](threat-detection-overview.md#alerts) pour détecter les tentatives inhabituelles et potentiellement dangereuses d’accès à des bases de données ou d’exploitation de celles-ci, à savoir :
  - attaque par injection de code SQL ;
  - vols/fuites d’informations d’identification ;
  - abus de privilège ;
  - exfiltration de données.

**Bonnes pratiques** :

- Configurez [Azure Defender pour SQL](azure-defender-for-sql.md) pour un serveur spécifique ou une instance gérée. Vous pouvez également configurer Azure Defender pour SQL pour tous les serveurs et instances gérées dans un abonnement en passant au [niveau Standard Azure Security Center](../../security-center/security-center-pricing.md).

- Pour une expérience d’investigation complète, nous vous recommandons d’activer l’ [audit de SQL Database](../../azure-sql/database/auditing-overview.md). Avec l’audit, vous pouvez suivre les événements de base de données et les écrire dans un journal d’audit au sein d’un compte de stockage Azure ou d’un espace de travail Azure Log Analytics.

### <a name="audit-critical-security-events"></a>Auditer les événements de sécurité critiques

Le suivi des événements de base de données vous aide à comprendre l’activité de la base de données. Vous pouvez découvrir des discordances et anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité. Il permet et facilite également le respect des normes de conformité.

**Mode d’implémentation** :

- Activez l’ [audit SQL Database](../../azure-sql/database/auditing-overview.md) ou l’[audit Managed Instance](../managed-instance/auditing-configure.md) pour suivre les événements de base de données et les consigner dans un journal d’audit de votre compte de stockage Azure, de l’espace de travail Log Analytics (préversion) ou d’Event Hubs (préversion).

- Des journaux d’audit peuvent être consignés dans un compte de stockage Azure ou dans un espace de travail Log Analytics en vue de leur utilisation par des journaux Azure Monitor, ou dans Event Hub en vue de leur utilisation par ce hub. Vous pouvez associer ces options comme vous le souhaitez. Les journaux d’audit seront écrits dans chacun des emplacements choisis.

**Bonnes pratiques** :

- Si vous configurez l’[audit SQL Database](../../azure-sql/database/auditing-overview.md) sur votre serveur ou l’[audit Managed Instance](../managed-instance/auditing-configure.md) pour auditer des événements, toutes les bases de données existantes et nouvellement créées sur ce serveur sont auditées.
- Par défaut, la stratégie d’audit couvre toutes les actions (requêtes, procédures stockées et connexions ayant réussi et échoué) sur les bases de données. Cela peut entraîner un volume élevé de journaux d’audit. Il est recommandé aux clients de [configurer l’audit pour différents types d’actions et de groupes d’actions à l’aide de PowerShell](./auditing-overview.md#manage-auditing). Cette configuration aide à contrôler le nombre d’actions auditées et à réduire le risque de perte d’événements. Les configurations d’audit personnalisées permettent aux clients de capturer uniquement les données d’audit nécessaires.
- Les journaux d’audit peuvent être utilisés directement dans le [portail Azure](https://portal.azure.com/), ou à partir de l’emplacement de stockage configuré.

> [!NOTE]
> L’activation de l’audit sur Log Analytics implique des frais selon les taux d’ingestion. Notez le coût associé à l’utilisation de cette [option](https://azure.microsoft.com/pricing/details/monitor/), ou envisagez de stocker les journaux d’audit dans un compte de stockage Azure.

**Ressources supplémentaires** :

- [Audit SQL Database](../../azure-sql/database/auditing-overview.md)
- [Audit SQL Server](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="secure-audit-logs"></a>Sécuriser les journaux d’audit

Limitez l’accès au compte de stockage pour prendre en charge la séparation des tâches et pour séparer l’administrateur de base de données des auditeurs.

**Mode d’implémentation** :

- Lors de l’enregistrement de journaux d’audit dans Stockage Azure, assurez-vous que l’accès au compte de stockage respecte les principes de sécurité minimaux. Contrôlez qui a accès au compte de stockage.
- Pour plus d’informations, voir [Autorisation d’accès au Stockage Azure](../../storage/common/storage-auth.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

**Bonnes pratiques** :

- Le contrôle de l’accès à la cible d’audit est un concept clé dans la séparation de l’administrateur de base de données des auditeurs.

- Lors de l’audit de l’accès aux données sensibles, songez à sécuriser les données avec un chiffrement afin d’éviter toute fuite d’informations vers l’auditeur. Pour plus d’informations, voir la section [Protéger les données sensibles en cours d’utilisation d’utilisateurs non autorisés disposant de privilèges élevés](#protect-sensitive-data-in-use-from-high-privileged-unauthorized-users).

## <a name="security-management"></a>Gestion de la sécurité

Cette section décrit les différents aspects et les meilleures pratiques de la gestion de la sécurité de vos bases de données. Elle inclut les meilleures pratiques pour s’assurer que vos bases de données sont configurées pour répondre aux normes de sécurité, pour la découverte, la classification et le suivi de l’accès aux données potentiellement sensibles de vos bases de données.

### <a name="ensure-that-the-databases-are-configured-to-meet-security-best-practices"></a>S’assurer que les bases de données sont configurées conformément aux bonnes pratiques de sécurité

Améliorez de manière proactive la sécurité de votre base de données en détectant et corrigeant les vulnérabilités potentielles de la base de données.

**Mode d’implémentation** :

- Activez l’[Évaluation des vulnérabilités SQL](/sql/relational-databases/security/sql-vulnerability-assessment) pour analyser votre base de données à la recherche de problèmes de sécurité, et pour une exécution périodique automatique de cette analyse sur vos bases de données.

**Bonnes pratiques** :

- Commencez par exécuter l’Évaluation des vulnérabilités sur vos bases de données, puis passez en revue les résultats de manière à corriger les vérifications en échec qui enfreignent les bonnes pratiques de sécurité. Configurez des bases de référence pour les configurations acceptables jusqu’à ce que l’analyse soit _propre_ ou que toutes les vérifications aient réussi.  

- Configurez des analyses périodiques pour qu’elles s’exécutent une fois par semaine et désignez la personne appropriée pour recevoir les e-mails de synthèse.

- Examinez le résumé de l’évaluation des vulnérabilités après chaque analyse hebdomadaire. Pour toutes les vulnérabilités trouvées, évaluez la dérive par rapport au résultat de l’analyse précédente et déterminez si la vérification doit être résolue. Vérifiez s’il existe une raison légitime de changer la configuration.

- Résolvez les vérifications et mettez à jour les bases de référence, le cas échéant. Créez des éléments de ticket pour la résolution des actions et suivez-les jusqu’à ce qu’elles soient résolues.

**Ressources supplémentaires** :

- [Évaluation des vulnérabilités SQL](/sql/relational-databases/security/sql-vulnerability-assessment)
- [Le service d’évaluation des vulnérabilités SQL vous permet d’identifier des vulnérabilités de base de données](sql-vulnerability-assessment.md)

### <a name="identify-and-tag-sensitive-data"></a>Identifier et baliser des données sensibles

Découvrez les colonnes susceptibles de contenir des données sensibles. La nature des données sensibles dépend considérablement du client, de la réglementation, etc., et doit être évaluée par les utilisateurs en charge de ces données. Classifiez les colonnes pour utiliser des scénarios d’audit et de protection avancés basés sur la sensibilité.

**Mode d’implémentation** :

- Utilisez une [Découverte et classification des données SQL](data-discovery-and-classification-overview.md) pour découvrir, classifier, libeller et protéger les données sensibles de vos bases de données.
  - Consultez les recommandations de classification créées par la découverte automatique dans le tableau de bord Découverte et classification des données SQL. Acceptez les classifications appropriées, de telle sorte que vos données sensibles soient marquées de façon permanente avec des étiquettes de classification.
  - Ajoutez manuellement des classifications pour tous les champs de données sensibles supplémentaires qui n’ont pas été découverts par le mécanisme automatisé.
- Pour plus d’informations, consultez [Découverte et classification des données SQL](/sql/relational-databases/security/sql-data-discovery-and-classification).

**Bonnes pratiques** :

- Surveillez régulièrement le tableau de bord classification pour obtenir une évaluation précise de l’état de classification de la base de données. Vous pouvez exporter ou imprimer un rapport sur l’état de classification de la base de données pour le partager à des fins de conformité et d’audit.

- Supervisez en permanence l’état des données sensibles recommandées dans l’Évaluation des vulnérabilités SQL. Effectuez le suivi de la règle de découverte des données sensibles et identifiez les dérives dans les colonnes recommandées pour la classification.  

- Utilisez la classification d’une manière adaptée aux besoins spécifiques de votre organisation. Personnalisez votre stratégie de protection des informations (étiquettes de sensibilité, types d’informations la logique de découverte) dans la stratégie [SQL Information Protection](../../security-center/security-center-info-protection-policy.md) dans Azure Security Center.

### <a name="track-access-to-sensitive-data"></a>Gérer l’accès aux données sensibles

Analysez qui accède à des données sensibles et capturez des requêtes sur les données sensibles dans des journaux d’audit.

**Mode d’implémentation** :

- Utilisez une combinaison d’audit SQL et de classification des données.
  - Dans votre journal d’[audit de SQL Database](../../azure-sql/database/auditing-overview.md), vous pouvez suivre spécifiquement l’accès aux données sensibles. Vous pouvez également afficher des informations telles que les données auxquelles l’utilisateur a accédé, ainsi que leur étiquette de sensibilité. Pour plus d’informations, consultez [Découverte et classification des données](data-discovery-and-classification-overview.md) et [Audit de l’accès aux données sensibles](data-discovery-and-classification-overview.md#audit-sensitive-data).

**Bonnes pratiques** :

- Consultez meilleures pratiques pour les sections Audit et Classification des données :
  - [Auditer les événements de sécurité critiques](#audit-critical-security-events)
  - [Identifier et baliser les données sensibles](#identify-and-tag-sensitive-data)

### <a name="visualize-security-and-compliance-status"></a>Visualiser l’état de sécurité et de conformité

Utilisez un système unifié de gestion de la sécurité de l’infrastructure qui renforce la sécurité de vos centres de données (y compris des bases de données dans SQL Database). Affichez la liste des recommandations relatives à la sécurité de vos bases de données et à l’état de conformité.

**Mode d’implémentation** :

- Surveillez les recommandations de sécurité liées à SQL et les menaces actives dans [Azure Security Center](https://azure.microsoft.com/documentation/services/security-center/).

## <a name="common-security-threats-and-potential-mitigations"></a>Menaces de sécurité courantes et atténuations potentielles

Cette section vous aide à trouver des mesures de sécurité pour vous protéger contre certains vecteurs d’attaque. La plupart des atténuations peuvent être mises en œuvre en suivant une ou plusieurs des instructions de sécurité ci-dessus.

### <a name="security-threat-data-exfiltration"></a>Menace de sécurité : Exfiltration de données

L’exfiltration de données est la copie, le transfert et la récupération non autorisés de données d’un ordinateur ou d’un serveur. Consultez la définition de l’[exfiltration de données](https://en.wikipedia.org/wiki/Data_exfiltration) sur Wikipédia.

Une connexion à un serveur via un point de terminaison public présente un risque d’exfiltration de données, car elle nécessite que les clients ouvrent leurs pare-feu aux adresses IP publiques.  

**Scenario 1** : Une application sur une machine virtuelle Azure se connecte à une base de données dans Azure SQL Database. Un acteur non autorisé obtient l’accès à la machine virtuelle et la compromet. Dans ce scénario, l’exfiltration de données signifie qu’une entité externe utilisant la machine virtuelle non autorisée se connecte à la base de données, copie les données à caractère personnel et les stocke dans un stockage blob ou une autre base de données SQL dans un autre abonnement.

**Scénario 2** : Administrateur de base de données rouge. Ce scénario est souvent déclenché par des clients sensibles à la sécurité opérant dans des secteurs réglementés. Dans ce scénario, un utilisateur disposant de privilèges élevés peut copier des données à partir d’Azure SQL Database vers un autre abonnement non contrôlé par le propriétaire des données.

**Atténuations potentielles** :

Aujourd’hui, Azure SQL Database et SQL Managed Instance offrent les techniques suivantes pour atténuer les menaces d’exfiltration de données :

- Utilisez une combinaison de règles d’autorisation et de refus sur les groupes de sécurité réseau de machines virtuelles Azure pour contrôler les régions accessibles à partir de la machine virtuelle.
- Si vous utilisez un serveur dans SQL Database, définissez les options suivantes :
  - Désactivez l’autorisation d’accès aux services.
  - Autorisez uniquement le trafic provenant du sous-réseau contenant votre machine virtuelle Azure en définissant une règle de pare-feu de réseau virtuel.
  - Utiliser une [Liaison privée](../../private-link/private-endpoint-overview.md)
- Pour SQL Managed Instance, l’utilisation d’un accès IP privé par défaut résout le premier problème d’exfiltration de données d’une machine virtuelle non autorisée. Activez la fonctionnalité de délégation de sous-réseau sur un sous-réseau pour définir automatiquement la stratégie la plus restrictive sur un sous-réseau SQL Managed Instance.
- Le problème d’administrateur de base de données non autorisé est plus exposé avec SQL Managed Instance, car sa surface d’exposition est plus importante et la configuration réseau requise est visible pour les clients. La meilleure méthode d’atténuation de ce problème consiste à appliquer toutes les pratiques décrites dans ce guide de sécurité pour empêcher le scénario d’administrateur de base de données non autorisé avant tout (pas seulement pour l’exfiltration de données). La fonctionnalité Always Encrypted est une méthode permettant de protéger des données sensibles en les chiffrant et en conservant la clé hors de portée de l’administrateur de bases de données.

## <a name="security-aspects-of-business-continuity-and-availability"></a>Aspects de sécurité liés à la continuité d’activité et à la disponibilité

La plupart des normes de sécurité ont trait à la disponibilité des données en termes de continuité opérationnelle, obtenue grâce à l’implémentation des fonctionnalités de redondance et de basculement pour éviter les points de défaillance unique. Pour les scénarios d’urgence, il est de pratique courante de conserver les sauvegardes des données et des fichiers journaux. La section suivante fournit une vue d’ensemble des fonctionnalités intégrées à Azure. Elle fournit également des options supplémentaires qui peuvent être configurées pour répondre à des besoins spécifiques :

- Azure offre une haute disponibilité intégrée : [Haute disponibilité avec SQL Database et SQL Managed Instance](high-availability-sla.md)

- Le niveau critique pour l’entreprise comprend des groupes de basculement, des sauvegardes de journaux complètes et différentielles, ainsi que des sauvegardes de restauration dans le temps qui sont activées par défaut :  
  - [Sauvegardes automatisées](automated-backups-overview.md)
  - [Récupérer une base de données à l’aide des sauvegardes de bases de données automatisées – Restauration dans le temps](recovery-using-backups.md#point-in-time-restore)

- Des fonctionnalités de continuité d’activité supplémentaires, telles que la configuration de zone redondante, des groupes de basculement automatique entre différents zones géographiques Azure peuvent être configurées : 
    - [Disponibilité élevée - Configuration de zone redondante pour les niveaux de services Premium et Critique pour l’entreprise](high-availability-sla.md#premium-and-business-critical-service-tier-zone-redundant-availability)
    - [Disponibilité élevée - Configuration de zone redondante pour le niveau de services Usage général](high-availability-sla.md#general-purpose-service-tier-zone-redundant-availability-preview)
    - [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md)

## <a name="next-steps"></a>Étapes suivantes

- Voir [Une vue d’ensemble des fonctionnalités de sécurité d’Azure SQL Database](security-overview.md)