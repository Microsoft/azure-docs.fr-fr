---
title: Sécuriser Azure SQL Edge
description: Découvrir la sécurité dans Azure SQL Edge
keywords: SQL Edge, sécurité
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 56d5eb981aa02d9da83973d49e8df79fcd9c7e9c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021012"
---
# <a name="securing-azure-sql-edge"></a>Sécurisation d’Azure SQL Edge

L’augmentation de l’adoption de l’IoT et du computing en périphérie dans différents secteurs entraîne une augmentation du nombre d’appareils et des données qu’ils génèrent. L’augmentation du volume de données et du nombre de points de terminaison d’appareil pose un défi considérable en termes de sécurité des données et des appareils. 

Azure SQL Edge offre plusieurs fonctionnalités et capacités qui facilitent la sécurisation des données IoT dans les bases de données SQL Server. Azure SQL Edge est construit à l’aide du moteur SQL qui alimente Microsoft SQL Server et Azure SQL, et en partage les fonctionnalités de sécurité, ce qui facilite l’extension des mêmes stratégies et pratiques de sécurité du cloud à la périphérie.

Tout comme Microsoft SQL Server et Azure SQL, la sécurisation des déploiements Azure SQL Edge peut être considérée comme une série d’étapes impliquant quatre domaines : la plateforme, l’authentification, les objets (y compris les données) et les applications qui accèdent au système. 

## <a name="platform-and-system-security"></a>Sécurité de la plateforme et du système

La plateforme pour Azure SQL Edge inclut l’hôte Docker physique, le système d’exploitation de l’ordinateur hôte et les systèmes de mise en réseau qui connectent l’appareil physique aux applications et aux clients. 

L’implémentation de la sécurité de la plateforme commence par le maintien hors du réseau des utilisateurs non autorisés. Voici certaines des meilleures pratiques :
- Implémentation de règles de pare-feu pour garantir la stratégie de sécurité organisationnelle. 
- Vérifiez que les dernières mises à jour de sécurité sont appliquées au système d’exploitation de l’appareil physique. 
- Spécification et restriction des ports hôtes qu’Azure SQL Edge utilise
- Vérification qu’un contrôle d’accès approprié est appliqué à tous les volumes de données hébergeant des données Azure SQL Edge. 

Pour plus d’informations sur les protocoles réseau Azure SQL Edge et les points de terminaison TDS, consultez [Protocoles réseau et points de terminaison TDS](/previous-versions/sql/sql-server-2008-r2/ms191220(v=sql.105)).

## <a name="authentication-and-authorization"></a>Authentification et autorisation 

### <a name="authentication"></a>Authentification  
L’authentification est le processus consistant à prouver que l’utilisateur est bien celui qu’il prétend être. Azure SQL Edge ne prend actuellement en charge que le mécanisme `SQL Authentication`.

- *Authentification SQL* :

    authentification d’un utilisateur qui se connecte à Azure SQL Edge en utilisant un nom d’utilisateur et un mot de passe. Le mot de passe de connexion **sa** de SQL doit être spécifié lors du déploiement de SQL Edge. Ensuite, des connexions SQL et utilisateurs supplémentaires peuvent être créés par l'administrateur de serveur, ce qui permet aux utilisateurs de se connecter à l'aide d'un nom d'utilisateur et d'un mot de passe.

    Pour plus d’informations sur la création et la gestion des connexions et des utilisateurs dans SQL Edge, consultez [Créer un compte de connexion](/sql/relational-databases/security/authentication-access/create-a-login) et [Créer un utilisateur de base de données](/sql/relational-databases/security/authentication-access/create-a-database-user).

### <a name="authorization"></a>Autorisation   

L’autorisation fait référence aux autorisations accordées à un utilisateur au sein d’une base de données dans Azure SQL Edge, et détermine ce que l’utilisateur est autorisé à faire. Les autorisations sont contrôlées en ajoutant des comptes d’utilisateurs aux [rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles) et en attribuant des autorisations au niveau de la base de données à ces rôles ou en octroyant aux utilisateurs certaines [autorisations au niveau des objets](/sql/relational-databases/security/permissions-database-engine). Pour plus d’informations, consultez [Connexions et utilisateurs](../azure-sql/database/logins-create-manage.md).

Il est recommandé de créer des rôles personnalisés si nécessaire. Ajoutez des utilisateurs au rôle doté du niveau de privilèges le moins élevé pour qu’ils remplissent leur fonction. N’attribuez pas d’autorisations directement aux utilisateurs. Le compte d’administrateur de serveur est un membre du rôle db_owner intégré. Il est doté d’autorisations étendues et ne doit être octroyé qu’à quelques utilisateurs ayant des charges administratives. Pour les applications, utilisez la clause [EXECUTE AS](/sql/t-sql/statements/execute-as-clause-transact-sql) pour spécifier le contexte d’exécution du module appelé ou utilisez des [rôles d’application](/sql/relational-databases/security/authentication-access/application-roles) avec des autorisations limitées. Cette pratique permet de s’assurer que l’application qui se connecte à la base de données dispose du moins de privilèges possible. L’application de ces meilleures pratiques favorise également la séparation des tâches.

## <a name="database-object-security"></a>Sécurité de l’objet de base de données

Les principaux désignent les individus, les groupes et les processus auxquels l’accès à SQL Edge a été accordé. Les « éléments sécurisables » sont le serveur, la base de données et les objets que contient la base de données. Chacun de ces éléments possède un ensemble d’autorisations que vous pouvez configurer pour réduire la surface d’exposition. Le tableau ci-dessous contient des informations sur les principaux et les éléments sécurisables.

|Pour obtenir des informations sur|Consultez|  
|---------------------------|---------|  
|Utilisateurs, rôles et processus de serveur et de base de données|[Moteur de base de données des principaux](/sql/relational-databases/security/authentication-access/principals-database-engine)|  
|Sécurité des serveurs et des objets de base de données|[Éléments sécurisables](/sql/relational-databases/security/securables)|
| &nbsp; | &nbsp; |

### <a name="encryption-and-certificates"></a>Chiffrement et certificats  
 
Le chiffrement ne résout pas les problèmes de contrôle d'accès. Toutefois, il améliore la sécurité en limitant la perte de données même dans le cas rare où les contrôles d'accès sont ignorés. Par exemple, si l'ordinateur hôte de la base de données est mal configuré et qu'un utilisateur malveillant parvient à se procurer des données sensibles, telles que des numéros de carte de crédit, les données subtilisées peuvent être inutilisables si elles sont chiffrées. Le tableau ci-dessous contient davantage d’informations sur le chiffrement dans Azure SQL Edge.  
  
|Pour obtenir des informations sur|Consultez|  
|---------------------------|---------|  
|Implémentation de connexions sécurisées|[Chiffrement des connexions](/sql/linux/sql-server-linux-encrypted-connections)|  
|Fonctions de chiffrement|[Fonctions de chiffrement &#40;Transact-SQL&#41;](/sql/t-sql/functions/cryptographic-functions-transact-sql)|
|Chiffrement des données au repos|[Chiffrement transparent des données](/sql/relational-databases/security/encryption/transparent-data-encryption)|
|Always Encrypted|[Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)|
| &nbsp; | &nbsp; |

> [!NOTE]
> Les limitations de sécurité décrites pour [SQL Server sur Linux](/sql/linux/sql-server-linux-security-overview) s’appliquent également à Azure SQL Edge. 


> [!NOTE]
> Azure SQL Edge n’inclut pas l’utilitaire mssql-conf. Toutes les configurations, y compris la configuration du chiffrement, doivent être exécutées par le biais du [fichier mssql.conf](configure.md#configure-by-using-an-mssqlconf-file) ou de [variables d’environnement](configure.md#configure-by-using-environment-variables). 


À l’instar d’Azure SQL et Microsoft SQL Server, Azure SQL Edge offre le même mécanisme de création et d’utilisation des certificats pour améliorer la sécurité des objets et des connexions. Pour plus d’informations, consultez [CREATE CERTIFICATE (TRANSACT-SQL)](/sql/t-sql/statements/create-certificate-transact-sql).


## <a name="application-security"></a>Sécurité des applications

### <a name="client-programs"></a>Programmes clients

Les meilleures pratiques de sécurité d’Azure SQL Edge incluent notamment l’écriture d’applications clientes sécurisées. Pour plus d’informations sur la façon de mieux sécuriser des applications clientes au niveau de la couche réseau, consultez [Configuration du réseau client](/sql/database-engine/configure-windows/client-network-configuration).

### <a name="security-catalog-views-and-functions"></a>Affichages catalogue et fonctions de sécurité  
Des informations de sécurité sont exposées dans plusieurs affichages et fonctions optimisés à des fins de performances et d’utilité. Le tableau ci-dessous contient des informations sur les affichages et les fonctions de sécurité dans Azure SQL Edge.  
  
|Fonctions et affichages|Liens|  
|---------------------------|---------|  
|Affichages catalogue de sécurité retournant des informations sur les autorisations de niveau serveur et base de données, les principaux, les rôles, etc. En outre, des affichages catalogue fournissent des informations sur les clés de chiffrement, les certificats et les informations d'identification.|[Affichages catalogue de sécurité &#40;Transact-SQL&#41;](/sql/relational-databases/system-catalog-views/security-catalog-views-transact-sql)|  
|Fonctions de sécurité retournant des informations sur l’utilisateur actuel, les autorisations et les schémas.|[Fonctions de sécurité &#40;Transact-SQL&#41;](/sql/t-sql/functions/security-functions-transact-sql)|  
|Affichages de gestion dynamique de la sécurité.|[Fonctions et vues de gestion dynamique relatives à la sécurité &#40;Transact-SQL&#41;](/sql/relational-databases/system-dynamic-management-views/security-related-dynamic-management-views-and-functions-transact-sql)|  
| &nbsp; | &nbsp; |

### <a name="auditing"></a>Audit 

Azure SQL Edge fournit les mêmes mécanismes d’audit que SQL Server. Pour plus d’informations, consultez [Audit SQL Server (moteur de base de données)](/sql/relational-databases/security/auditing/sql-server-audit-database-engine).


## <a name="next-steps"></a>Étapes suivantes

- [Prise en main des fonctionnalités de sécurité](/sql/linux/sql-server-linux-security-get-started)
- [Exécution d’Azure SQL Edge en tant qu’utilisateur non racine](configure.md#run-azure-sql-edge-as-non-root-user)
- [Azure Security Center pour IoT](../defender-for-iot/overview.md)