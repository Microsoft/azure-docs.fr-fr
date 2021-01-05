---
title: Affichages de la base de données Azure Blockchain Workbench
description: Aperçu des affichages de la base de données SQL DB en préversion Azure Blockchain Workbench.
ms.date: 09/05/2019
ms.topic: article
ms.reviewer: mmercuri
ms.openlocfilehash: e715bd2297c1e051ae92b15d73c945c9e80c3d94
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87003319"
---
# <a name="azure-blockchain-workbench-database-views"></a>Affichages de la base de données Azure Blockchain Workbench

Azure Blockchain Workbench Preview fournit des données provenant de registres distribués à une base de données SQL DB *off-chain*. La base de données « off-chain» permet d’utiliser SQL et les outils existants, tels que [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017), pour interagir avec les données de blockchain.

Azure Blockchain Workbench fournit un ensemble de vues de base de données qui permettent d’accéder aux données utiles lors de l’exécution de vos requêtes. Ces vues sont fortement dénormalisées afin de faciliter l’obtention rapide de rapports de génération et d’analyses, et le cas échéant utiliser les données de blockchain avec les outils existants sans avoir à former le personnel chargé de la base de données.

Cette section inclut une présentation des vues de la base de données et des données qu’elles contiennent.

> [!NOTE]
> L’utilisation directe de tables de base de données localisées dans la base de données en dehors de ces vues est possible mais elle n’est pas prise en charge.
>

## <a name="vwapplication"></a>vwApplication

Cette vue fournit des détails sur les **applications** qui ont été chargées vers Azure Blockchain Workbench.

| Nom                             | Type          | Peut être Null | Description                                                                                                                                                                                                                                                   |
|----------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                    | int           | Non          | Identificateur unique de l’application |
| ApplicationName                  | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDescription           | nvarchar(255) | Oui         | Description de l’application |
| ApplicationDisplayName           | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled               | bit           | Non          | Identifie si l’application est actuellement activée<br /> **Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données. |
| UploadedDtTm                     | datetime2(7)  | Non          | Date et heure de chargement d’un contrat |
| UploadedByUserId                 | int           | Non          | ID de l’utilisateur ayant chargé l’application |
| UploadedByUserExternalId         | nvarchar(255) | Non          | Identificateur externe de l’utilisateur ayant chargé l’application. Par défaut, il s’agit de l’ID de l’utilisateur provenant d’Azure Active Directory pour le consortium.                                                                                                |
| UploadedByUserProvisioningStatus | int           | Non          | Identifie l’état actuel du processus d’approvisionnement pour l’utilisateur. Les valeurs possibles sont les suivantes : <br />0 : l’utilisateur a été créé par l’API<br />1 : une clé a été associée à l’utilisateur dans la base de données<br />2 : l’utilisateur est entièrement configuré                         |
| UploadedByUserFirstName          | nvarchar(50)  | Oui         | Prénom de l’utilisateur qui a chargé le contrat |
| UploadedByUserLastName           | nvarchar(50)  | Oui         | Nom de l’utilisateur qui a chargé le contrat |
| UploadedByUserEmailAddress       | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur qui a chargé le contrat |

## <a name="vwapplicationrole"></a>vwApplicationRole

Cette vue fournit des détails sur les rôles qui ont été définis dans les applications Azure Blockchain Workbench.

Dans une application de *transfert de ressources*, par exemple, il est possible de définir des rôles du type *Acheteur* et *Vendeur*.

| Nom                   | Type             | Peut être Null | Description                                       |
|------------------------|------------------|-------------|---------------------------------------------------|
| ApplicationId          | int              | Non          | Identificateur unique de l’application           |
| ApplicationName        | nvarchar(50)     | Non          | Nom de l’application                       |
| ApplicationDescription | nvarchar(255)    | Oui         | Description de l’application                  |
| ApplicationDisplayName | nvarchar(255)    | Non          | Nom d’affichage dans une interface utilisateur      |
| RoleId                 | int              | Non          | Identificateur unique d’un rôle dans l’application |
| RoleName               | nvarchar50)      | Non          | Nom du rôle                              |
| RoleDescription        | description(255) | Oui         | Description du rôle                         |

## <a name="vwapplicationroleuser"></a>vwApplicationRoleUser

Cette vue fournit des détails sur les rôles qui ont été définis dans les applications Azure Blockchain Workbench et les utilisateurs associés.

Dans une application de *transfert de ressources*, par exemple, *John Smith* peut être associé au rôle *Acheteur*.

| Nom                       | Type          | Peut être Null | Description                                                                                                                                                                                                                           |
|----------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId              | int           | Non          | Identificateur unique de l’application                                                                                                                                                                                               |
| ApplicationName            | nvarchar(50)  | Non          | Nom de l’application                                                                                                                                                                                                           |
| ApplicationDescription     | nvarchar(255) | Oui         | Description de l’application                                                                                                                                                                                                      |
| ApplicationDisplayName     | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur                                                                                                                                                                                          |
| ApplicationRoleId          | int           | Non          | Identificateur unique d’un rôle dans l’application                                                                                                                                                                                     |
| ApplicationRoleName        | nvarchar50)   | Non          | Nom du rôle                                                                                                                                                                                                                  |
| ApplicationRoleDescription | nvarchar(255) | Oui         | Description du rôle                                                                                                                                                                                                             |
| UserId                     | int           | Non          | ID de l’utilisateur associé au rôle |
| UserExternalId             | nvarchar(255) | Non          | Identificateur externe de l’utilisateur associé au rôle. Par défaut, il s’agit de l’ID de l’utilisateur provenant d’Azure Active Directory pour le consortium.                                                                     |
| UserProvisioningStatus     | int           | Non          | Identifie l’état actuel du processus d’approvisionnement pour l’utilisateur. Les valeurs possibles sont les suivantes : <br />0 : l’utilisateur a été créé par l’API<br />1 : une clé a été associée à l’utilisateur dans la base de données<br />2 : l’utilisateur est entièrement configuré |
| UserFirstName              | nvarchar(50)  | Oui         | Prénom de l’utilisateur associé au rôle |
| UserLastName               | nvarchar(255) | Oui         | Nom de l’utilisateur associé au rôle |
| UserEmailAddress           | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur associé au rôle |

## <a name="vwconnectionuser"></a>vwConnectionUser

Cette vue fournit des détails sur les connexions qui ont été définies dans Azure Blockchain Workbench et les utilisateurs associés. Pour chaque connexion, cette vue contient les données suivantes :

-   Détails du registre associé
-   Informations sur l’utilisateur associé

| Nom                     | Type          | Peut être Null | Description                                                                                                                                                                                                                           |
|--------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId             | int           | Non          | Identificateur unique pour une connexion dans Azure Blockchain Workbench |
| ConnectionEndpointUrl    | nvarchar(50)  | Non          | URL de point de terminaison d’une connexion |
| ConnectionFundingAccount | nvarchar(255) | Oui         | Compte de financement associé à une connexion, le cas échéant |
| LedgerId                 | int           | Non          | Identificateur unique d’un registre |
| LedgerName               | nvarchar(50)  | Non          | Nom du registre |
| LedgerDisplayName        | nvarchar(255) | Non          | Nom d’affichage du registre dans l’interface utilisateur |
| UserId                   | int           | Non          | ID de l’utilisateur associé à la connexion |
| UserExternalId           | nvarchar(255) | Non          | Identificateur externe de l’utilisateur associé à la connexion. Par défaut, il s’agit de l’ID de l’utilisateur provenant d’Azure Active Directory pour le consortium. |
| UserProvisioningStatus   | int           | Non          |Identifie l’état actuel du processus d’approvisionnement pour l’utilisateur. Les valeurs possibles sont les suivantes : <br />0 : l’utilisateur a été créé par l’API<br />1 : une clé a été associée à l’utilisateur dans la base de données<br />2 : l’utilisateur est entièrement configuré |
| UserFirstName            | nvarchar(50)  | Oui         | Prénom de l’utilisateur associé à la connexion |
| UserLastName             | nvarchar(255) | Oui         | Nom de l’utilisateur associé à la connexion |
| UserEmailAddress         | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur associé à la connexion |

## <a name="vwcontract"></a>vwContract

Cette vue fournit des détails sur les contrats déployés. Pour chaque contrat, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Implémentation du registre associé pour la fonction
-   Détails de l’utilisateur qui a initié l’action
-   Détails concernant le bloc de blockchain et la transaction

| Nom                                     | Type           | Peut être Null | Description                                                                                                                                                                                                                                                   |
|------------------------------------------|----------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ConnectionId                             | int            | Non          | Identificateur unique pour une connexion dans Azure Blockchain Workbench.                                                                                                                                                                                         |
| ConnectionEndpointUrl                    | nvarchar(50)   | Non          | URL de point de terminaison d’une connexion |
| ConnectionFundingAccount                 | nvarchar(255)  | Oui         | Compte de financement associé à une connexion, le cas échéant |
| LedgerId                                 | int            | Non          | Identificateur unique d’un registre |
| LedgerName                               | nvarchar(50)   | Non          | Nom du registre |
| LedgerDisplayName                        | nvarchar(255)  | Non          | Nom d’affichage du registre dans l’interface utilisateur |
| ApplicationId                            | int            | Non          | Identificateur unique de l’application |
| ApplicationName                          | nvarchar (50)  | Non          | Nom de l’application |
| ApplicationDisplayName                   | nvarchar (255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                       | bit            | Non          | Identifie si l’application est actuellement activée.<br /> **Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données.  |
| WorkflowId                               | int            | Non          | Identificateur unique du workflow associé à un contrat |
| WorkflowName                             | nvarchar(50)   | Non          | Nom du workflow associé à un contrat |
| WorkflowDisplayName                      | nvarchar(255)  | Non          | Nom d’affichage du workflow associé au contrat dans l’interface utilisateur |
| WorkflowDescription                      | nvarchar(255)  | Oui         | Description du workflow associé à un contrat |
| ContractCodeId                           | int            | Non          | Identificateur unique du code de contrat associé au contrat |
| ContractFileName                         | int            | Non          | Nom du fichier contenant le code de contrat intelligent de ce workflow. |
| ContractUploadedDtTm                     | int            | Non          | Date et heure de chargement du code de contrat |
| ContractId                               | int            | Non          | Identificateur unique du contrat |
| ContractProvisioningStatus               | int            | Non          | Identifie l’état actuel du processus de configuration pour le contrat. Les valeurs possibles sont les suivantes : <br />0 : le contrat a été créé par l’API dans la base de données<br />1 : le contrat a été envoyé au registre<br />2 : le contrat a été déployé avec succès dans le registre<br />3 ou 4 : le déploiement du contrat dans le registre a échoué<br />5 : le contrat a été déployé avec succès dans le registre <br /><br />Les valeurs de 0 à 5 sont prises en charge à partir de la version 1.5. Pour assurer la compatibilité descendante, la version actuelle propose le mode d’affichage **vwContractV0**, qui prend uniquement en charge les valeurs de 0 à 2. |
| ContractLedgerIdentifier                 | nvarchar (255) |             | Adresse e-mail de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserId                 | int            | Non          | Identificateur externe pour l’utilisateur qui a déployé le contrat. Par défaut, il s’agit du GUID représentant l’ID Azure Active Directory de l’utilisateur.                                                                                                          |
| ContractDeployedByUserExternalId         | nvarchar(255)  | Non          | Identificateur externe pour l’utilisateur qui a déployé le contrat. Par défaut, il s’agit du GUID représentant l’ID Azure Active Directory de l’utilisateur.                                                                                                         |
| ContractDeployedByUserProvisioningStatus | int            | Non          | Identifie l’état actuel du processus de configuration pour l’utilisateur. Les valeurs possibles sont les suivantes : <br />0 : l’utilisateur a été créé par l’API<br />1 : une clé a été associée à l’utilisateur dans la base de données <br />2 : l’utilisateur est entièrement configuré                     |
| ContractDeployedByUserFirstName          | nvarchar(50)   | Oui         | Prénom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserLastName           | nvarchar(255)  | Oui         | Nom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserEmailAddress       | nvarchar(255)  | Oui         | Adresse e-mail de l’utilisateur qui a déployé le contrat |

## <a name="vwcontractaction"></a>vwContractAction

Cette vue représente la majorité des informations relatives aux actions effectuées sur les contrats et elle est destinée à faciliter les scénarios de création de rapports. Pour chaque action entreprise, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Définition des paramètres et fonctions de contrat intelligent associés
-   Implémentation du registre associé pour la fonction
-   Valeurs d’instances spécifiques fournies pour les paramètres
-   Détails de l’utilisateur qui a initié l’action
-   Détails concernant le bloc de blockchain et la transaction

| Nom                                     | Type          | Peut être Null | Description                                                                                                                                                                                                                                                                                                    |
|------------------------------------------|---------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                            | int           | Non          | Identificateur unique de l’application |
| ApplicationName                          | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName                   | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                       | bit           | Non          | Ce champ identifie si l’application est actuellement activée. Remarque : même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur le blockchain et les données relatives à ces contrats restent dans la base de données.                                                  |
| WorkflowId                               | int           | Non          | Identificateur unique d’un workflow |
| WorkflowName                             | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName                      | nvarchar(255) | Non          | Nom d’affichage du workflow dans une interface utilisateur |
| WorkflowDescription                      | nvarchar(255) | Oui         | Description du workflow |
| ContractId                               | int           | Non          | Identificateur unique du contrat |
| ContractProvisioningStatus               | int           | Non          | Identifie l’état actuel du processus de configuration pour le contrat. Les valeurs possibles sont les suivantes : <br />0 : le contrat a été créé par l’API dans la base de données<br />1 : le contrat a été envoyé au registre<br />2 : le contrat a été déployé avec succès dans le registre<br />3 ou 4 : le déploiement du contrat dans le registre a échoué<br />5 : le contrat a été déployé avec succès dans le registre <br /><br />Les valeurs de 0 à 5 sont prises en charge à partir de la version 1.5. Pour assurer la compatibilité descendante, la version actuelle propose le mode d’affichage **vwContractActionV0**, qui prend uniquement en charge les valeurs de 0 à 2. |
| ContractCodeId                           | int           | Non          | Identificateur unique pour l’implémentation de code du contrat |
| ContractLedgerIdentifier                 | nvarchar(255) | Oui         | Identificateur unique associé à la version déployée d’un contrat intelligent pour un registre distribué spécifique. Par exemple, Ethereum. |
| ContractDeployedByUserId                 | int           | Non          | Identificateur unique de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserFirstName          | nvarchar(50)  | Oui         | Prénom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserLastName           | nvarchar(255) | Oui         | Nom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserExternalId         | nvarchar(255) | Non          | Identificateur externe de l’utilisateur qui a déployé le contrat. Par défaut, il s’agit du GUID qui représente son identité dans le consortium Azure Active Directory.                                                                                                                                                |
| ContractDeployedByUserEmailAddress       | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur qui a déployé le contrat |
| WorkflowFunctionId                       | int           | Non          | Identificateur unique d’une fonction de workflow |
| WorkflowFunctionName                     | nvarchar(50)  | Non          | Nom de la fonction |
| WorkflowFunctionDisplayName              | nvarchar(255) | Non          | Nom d’affichage d’une fonction dans l’interface utilisateur. |
| WorkflowFunctionDescription              | nvarchar(255) | Non          | Description de la fonction |
| ContractActionId                         | int           | Non          | Identificateur unique d’une action de contrat |
| ContractActionProvisioningStatus         | int           | Non          | Identifie l’état actuel du processus de configuration pour l’action de contrat. Les valeurs possibles sont les suivantes : <br />0 : l’action de contrat a été créée par l’API dans la base de données<br />1 : l’action de contrat a été envoyée au registre<br />2 : l’action de contrat a été déployée avec succès dans le registre<br />3 ou 4 : le déploiement du contrat dans le registre a échoué<br />5 : le contrat a été déployé avec succès dans le registre <br /><br />Les valeurs de 0 à 5 sont prises en charge à partir de la version 1.5. Pour assurer la compatibilité descendante, la version actuelle propose le mode d’affichage **vwContractActionV0**, qui prend uniquement en charge les valeurs de 0 à 2. |
| ContractActionTimestamp                  | datetime(2,7) | Non          | Timestamp de l’action de contrat |
| ContractActionExecutedByUserId           | int           | Non          | Identificateur unique de l’utilisateur qui a exécuté l’action de contrat |
| ContractActionExecutedByUserFirstName    | int           | Oui         | Prénom de l’utilisateur qui a exécuté l’action de contrat |
| ContractActionExecutedByUserLastName     | nvarchar(50)  | Oui         | Nom de l’utilisateur qui a exécuté l’action de contrat |
| ContractActionExecutedByUserExternalId   | nvarchar(255) | Oui         | Identificateur externe de l’utilisateur qui a exécuté l’action de contrat. Par défaut, il s’agit du GUID qui représente son identité dans le consortium Azure Active Directory. |
| ContractActionExecutedByUserEmailAddress | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur qui a exécuté l’action de contrat |
| WorkflowFunctionParameterId              | int           | Non          | Identificateur unique d’un paramètre de la fonction |
| WorkflowFunctionParameterName            | nvarchar(50)  | Non          | Nom d’un paramètre de la fonction |
| WorkflowFunctionParameterDisplayName     | nvarchar(255) | Non          | Nom d’affichage d’un paramètre de fonction dans l’interface utilisateur. |
| WorkflowFunctionParameterDataTypeId      | int           | Non          | Identificateur unique pour le type de données associé à un paramètre de fonction de workflow |
| WorkflowParameterDataTypeName            | nvarchar(50)  | Non          | Nom d’un type de données associé à un paramètre de fonction de workflow |
| ContractActionParameterValue             | nvarchar(255) | Non          | Valeur du paramètre stocké dans le contrat intelligent |
| BlockHash                                | nvarchar(255) | Oui         | Code de hachage du bloc |
| BlockNumber                              | int           | Oui         | Numéro du bloc sur le registre |
| BlockTimestamp                           | datetime(2,7) | Oui         | Timestamp du bloc |
| TransactionId                            | int           | Non          | Identificateur unique de la transaction |
| TransactionFrom                          | nvarchar(255) | Oui         | Partie qui est à l’origine de la transaction |
| TransactionTo                            | nvarchar(255) | Oui         | Partie avec laquelle la transaction a eu lieu |
| TransactionHash                          | nvarchar(255) | Oui         | Code de hachage d’une transaction |
| TransactionIsWorkbenchTransaction        | bit           | Oui         | Bit qui indique si la transaction est une transaction Azure Blockchain Workbench |
| TransactionProvisioningStatus            | int           | Oui         | Identifie l’état actuel du processus de configuration pour la transaction. Les valeurs possibles sont les suivantes : <br />0 : la transaction a été créée par l’API dans la base de données<br />1 : la transaction a été envoyée au registre<br />2 : la transaction a été déployée avec succès dans le registre                 |
| TransactionValue                         | decimal(32,2) | Oui         | Valeur de la transaction |

## <a name="vwcontractproperty"></a>vwContractProperty

Cette vue représente la majorité des informations relatives aux propriétés associées à un contrat et elle est destinée à faciliter les scénarios communs de création de rapports. Pour chaque propriété, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Détails de l’utilisateur qui a déployé le workflow
-   Définition de la propriété de contrat intelligent associée
-   Valeurs d’instances spécifiques pour les propriétés
-   Détails de la propriété d’état du contrat

| Nom                               | Type          | Peut être Null | Description                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | Non          | Identificateur unique de l’application |
| ApplicationName                    | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName             | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                 | bit           | Non          | Identifie si l’application est actuellement activée.<br />**Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données.                      |
| WorkflowId                         | int           | Non          | Identificateur unique du workflow |
| WorkflowName                       | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName                | nvarchar(255) | Non          | Nom d’affichage du workflow dans l’interface utilisateur |
| WorkflowDescription                | nvarchar(255) | Oui         | Description du workflow |
| ContractId                         | int           | Non          | Identificateur unique du contrat |
| ContractProvisioningStatus         | int           | Non          | Identifie l’état actuel du processus de configuration pour le contrat. Les valeurs possibles sont les suivantes : <br />0 : le contrat a été créé par l’API dans la base de données<br />1 : le contrat a été envoyé au registre<br />2 : le contrat a été déployé avec succès dans le registre<br />3 ou 4 : le déploiement du contrat dans le registre a échoué<br />5 : le contrat a été déployé avec succès dans le registre <br /><br />Les valeurs de 0 à 5 sont prises en charge à partir de la version 1.5. Pour assurer la compatibilité descendante, la version actuelle propose le mode d’affichage **vwContractPropertyV0**, qui prend uniquement en charge les valeurs de 0 à 2. |
| ContractCodeId                     | int           | Non          | Identificateur unique pour l’implémentation de code du contrat |
| ContractLedgerIdentifier           | nvarchar(255) | Oui         | Identificateur unique associé à la version déployée d’un contrat intelligent pour un registre distribué spécifique. Par exemple, Ethereum. |
| ContractDeployedByUserId           | int           | Non          | Identificateur unique de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserFirstName    | nvarchar(50)  | Oui         | Prénom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserLastName     | nvarchar(255) | Oui         | Nom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserExternalId   | nvarchar(255) | Non          | Identificateur externe de l’utilisateur qui a déployé le contrat. Par défaut, il s’agit du GUID qui représente son identité dans le consortium Azure Active Directory |
| ContractDeployedByUserEmailAddress | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur qui a déployé le contrat |
| WorkflowPropertyId                 | int           |             | Identificateur unique d’une propriété de workflow |
| WorkflowPropertyDataTypeId         | int           | Non          | ID du type de données de la propriété |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | Non          | Nom du type de données de la propriété |
| WorkflowPropertyName               | nvarchar(50)  | Non          | Nom de la propriété du workflow |
| WorkflowPropertyDisplayName        | nvarchar(255) | Non          | Nom d’affichage de la propriété du workflow |
| WorkflowPropertyDescription        | nvarchar(255) | Oui         | Description de la propriété |
| ContractPropertyValue              | nvarchar(255) | Non          | Valeur d’une propriété sur le contrat |
| StateName                          | nvarchar(50)  | Oui         | Si cette propriété contient l’état du contrat, il s’agit du nom d’affichage de l’état. Si elle n’est pas associé à l’état, la valeur sera Null. |
| StateDisplayName                   | nvarchar(255) | Non          | Si cette propriété contient l’état, il s’agit du nom d’affichage de l’état. Si elle n’est pas associé à l’état, la valeur sera Null. |
| StateValue                         | nvarchar(255) | Oui         | Si cette propriété contient l’état, il s’agit de la valeur de ce dernier. Si elle n’est pas associé à l’état, la valeur sera Null. |

## <a name="vwcontractstate"></a>vwContractState

Cette vue représente la majorité des informations relatives à l’état d’un contrat spécifique et elle est destinée à faciliter les scénarios communs de création de rapports. Chaque enregistrement de cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Détails de l’utilisateur qui a déployé le workflow
-   Définition de la propriété de contrat intelligent associée
-   Détails de la propriété d’état du contrat

| Nom                               | Type          | Peut être Null | Description                                                                                                                                                                                                                                                                        |
|------------------------------------|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                      | int           | Non          | Identificateur unique de l’application |
| ApplicationName                    | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName             | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                 | bit           | Non          | Identifie si l’application est actuellement activée.<br />**Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données. |
| WorkflowId                         | int           | Non          | Identificateur unique du workflow |
| WorkflowName                       | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName                | nvarchar(255) | Non          | Nom affiché dans l’interface utilisateur |
| WorkflowDescription                | nvarchar(255) | Oui         | Description du workflow |
| ContractLedgerImplementationId     | nvarchar(255) | Oui         | Identificateur unique associé à la version déployée d’un contrat intelligent pour un registre distribué spécifique. Par exemple, Ethereum. |
| ContractId                         | int           | Non          | Identificateur unique du contrat |
| ContractProvisioningStatus         | int           | Non          |Identifie l’état actuel du processus de configuration pour le contrat. Les valeurs possibles sont les suivantes : <br />0 : le contrat a été créé par l’API dans la base de données<br />1 : le contrat a été envoyé au registre<br />2 : le contrat a été déployé avec succès dans le registre<br />3 ou 4 : le déploiement du contrat dans le registre a échoué<br />5 : le contrat a été déployé avec succès dans le registre <br /><br />Les valeurs de 0 à 5 sont prises en charge à partir de la version 1.5. Pour assurer la compatibilité descendante, la version actuelle propose le mode d’affichage **vwContractStateV0**, qui prend uniquement en charge les valeurs de 0 à 2. |
| ConnectionId                       | int           | Non          | Identificateur unique de l’instance blockchain sur laquelle le workflow est déployé |
| ContractCodeId                     | int           | Non          | Identificateur unique pour l’implémentation de code du contrat |
| ContractDeployedByUserId           | int           | Non          | Identificateur unique de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserExternalId   | nvarchar(255) | Non          | Identificateur externe de l’utilisateur qui a déployé le contrat. Par défaut, il s’agit du GUID qui représente son identité dans le consortium Azure Active Directory. |
| ContractDeployedByUserFirstName    | nvarchar(50)  | Oui         | Prénom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserLastName     | nvarchar(255) | Oui         | Nom de l’utilisateur qui a déployé le contrat |
| ContractDeployedByUserEmailAddress | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur qui a déployé le contrat |
| WorkflowPropertyId                 | int           | Non          | Identificateur unique d’une propriété de workflow |
| WorkflowPropertyDataTypeId         | int           | Non          | ID du type de données de la propriété de workflow |
| WorkflowPropertyDataTypeName       | nvarchar(50)  | Non          | Nom du type de données de la propriété de workflow |
| WorkflowPropertyName               | nvarchar(50)  | Non          | Nom de la propriété du workflow |
| WorkflowPropertyDisplayName        | nvarchar(255) | Non          | Nom d’affichage de la propriété dans une interface utilisateur |
| WorkflowPropertyDescription        | nvarchar(255) | Oui         | Description de la propriété |
| ContractPropertyValue              | nvarchar(255) | Non          | Valeur d’une propriété stockée dans le contrat |
| StateName                          | nvarchar(50)  | Oui         | Si cette propriété contient l’état, il s’agit du nom d’affichage de ce dernier. Si elle n’est pas associé à l’état, la valeur sera Null. |
| StateDisplayName                   | nvarchar(255) | Non          | Si cette propriété contient l’état, il s’agit du nom d’affichage de l’état. Si elle n’est pas associé à l’état, la valeur sera Null. |
| StateValue                         | nvarchar(255) | Oui         | Si cette propriété contient l’état, il s’agit de la valeur de ce dernier. Si elle n’est pas associé à l’état, la valeur sera Null. |

## <a name="vwuser"></a>vwUser

Cette vue fournit des détails sur les membres du consortium qui peuvent utiliser Azure Blockchain Workbench. Par défaut, les données sont remplies par le biais de la configuration initiale de l’utilisateur.

| Nom               | Type          | Peut être Null | Description                                                                                                                                                                                                                               |
|--------------------|---------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                 | int           | Non          | Identificateur unique d’un utilisateur |
| ExternalID         | nvarchar(255) | Non          | Identificateur externe d’un utilisateur. Par défaut, il s’agit du GUID représentant l’ID Azure Active Directory de l’utilisateur. |
| ProvisioningStatus | int           | Non          |Identifie l’état actuel du processus d’approvisionnement pour l’utilisateur. Les valeurs possibles sont les suivantes : <br />0 : l’utilisateur a été créé par l’API<br />1 : une clé a été associée à l’utilisateur dans la base de données<br />2 : l’utilisateur est entièrement configuré |
| FirstName          | nvarchar(50)  | Oui         | Prénom de l’utilisateur |
| LastName           | nvarchar(50)  | Oui         | Nom de l’utilisateur |
| EmailAddress       | nvarchar(255) | Oui         | Adresse e-mail de l’utilisateur |

## <a name="vwworkflow"></a>vwWorkflow

Cette vue représente les métadonnées du principal workflow de détails, ainsi que les fonctions et les paramètres du workflow. Conçue pour la création de rapports, elle contient également les métadonnées relatives à l’application associée au workflow. Cette vue comporte les données issues de plusieurs tables sous-jacentes pour faciliter la création de rapports sur les workflows. Pour chaque workflow, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Informations d’état de démarrage du workflow associé

| Nom                              | Type          | Peut être Null | Description                                                                                                                                |
|-----------------------------------|---------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                     | int           | Non          | Identificateur unique de l’application |
| ApplicationName                   | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName            | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                | bit           | Non          | Identifie si l’application est activée |
| WorkflowId                        | int           | Oui         | Identificateur unique d’un workflow |
| WorkflowName                      | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName               | nvarchar(255) | Non          | Nom affiché dans l’interface utilisateur |
| WorkflowDescription               | nvarchar(255) | Oui         | Description du workflow. |
| WorkflowConstructorFunctionId     | int           | Non          | Identificateur de la fonction de workflow qui sert de constructeur pour le workflow |
| WorkflowStartStateId              | int           | Non          | Identificateur unique de l’état |
| WorkflowStartStateName            | nvarchar(50)  | Non          | Nom de l’état |
| WorkflowStartStateDisplayName     | nvarchar(255) | Non          | Nom d’affichage de l’état dans l’interface utilisateur |
| WorkflowStartStateDescription     | nvarchar(255) | Oui         | Description de l’état du workflow |
| WorkflowStartStateStyle           | nvarchar(50)  | Oui         | Cette valeur identifie le pourcentage d’achèvement du workflow dans cet état |
| WorkflowStartStateValue           | int           | Non          | Valeur de l’état |
| WorkflowStartStatePercentComplete | int           | Non          | Texte descriptif donnant aux clients des conseils sur le rendu de cet état dans l’interface utilisateur. Les états pris en charge incluent *Succès* et *Échec* |

## <a name="vwworkflowfunction"></a>vwWorkflowFunction

Cette vue représente les métadonnées du principal workflow de détails, ainsi que les fonctions et les paramètres du workflow. Conçue pour la création de rapports, elle contient également les métadonnées relatives à l’application associée au workflow. Cette vue comporte les données issues de plusieurs tables sous-jacentes pour faciliter la création de rapports sur les workflows. Pour chaque fonction de workflow, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Détails relatifs à la fonction de workflow

| Nom                                 | Type          | Peut être Null | Description                                                                          |
|--------------------------------------|---------------|-------------|--------------------------------------------------------------------------------------|
| ApplicationId                        | int           | Non          | Identificateur unique de l’application |
| ApplicationName                      | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName               | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled                   | bit           | Non          | Identifie si l’application est activée |
| WorkflowId                           | int           | Non          | Identificateur unique d’un workflow |
| WorkflowName                         | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName                  | nvarchar(255) | Non          | Nom d’affichage du workflow dans l’interface utilisateur |
| WorkflowDescription                  | nvarchar(255) | Oui         | Description du workflow |
| WorkflowFunctionId                   | int           | Non          | Identificateur unique d’une fonction |
| WorkflowFunctionName                 | nvarchar(50)  | Oui         | Nom de la fonction |
| WorkflowFunctionDisplayName          | nvarchar(255) | Non          | Nom d’affichage d’une fonction dans l’interface utilisateur. |
| WorkflowFunctionDescription          | nvarchar(255) | Oui         | Description de la fonction de workflow |
| WorkflowFunctionIsConstructor        | bit           | Non          | Indique si la fonction de workflow est le constructeur de ce dernier |
| WorkflowFunctionParameterId          | int           | Non          | Identificateur unique d’un paramètre de fonction |
| WorkflowFunctionParameterName        | nvarchar(50)  | Non          | Nom d’un paramètre de la fonction |
| WorkflowFunctionParameterDisplayName | nvarchar(255) | Non          | Nom d’affichage d’un paramètre de fonction dans l’interface utilisateur. |
| WorkflowFunctionParameterDataTypeId  | int           | Non          | Identificateur unique pour le type de données associé à un paramètre de fonction de workflow |
| WorkflowParameterDataTypeName        | nvarchar(50)  | Non          | Nom d’un type de données associé à un paramètre de fonction de workflow |

## <a name="vwworkflowproperty"></a>vwWorkflowProperty

Cette vue représente les propriétés définies pour un workflow. Pour chaque propriété, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Détails de la propriété de workflow

| Nom                         | Type          | Peut être Null | Description                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | Non          | Identificateur unique de l’application |
| ApplicationName              | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName       | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| ApplicationEnabled           | bit           | Non          | Identifie si l’application est actuellement activée.<br />**Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données. |
| WorkflowId                   | int           | Non          | Identificateur unique du workflow |
| WorkflowName                 | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName          | nvarchar(255) | Non          | Nom affiché pour le workflow dans une interface utilisateur |
| WorkflowDescription          | nvarchar(255) | Oui         | Description du workflow |
| WorkflowPropertyID           | int           | Non          | Identificateur unique d’une propriété de workflow |
| WorkflowPropertyName         | nvarchar(50)  | Non          | Nom de la propriété |
| WorkflowPropertyDescription  | nvarchar(255) | Oui         | Description de la propriété |
| WorkflowPropertyDisplayName  | nvarchar(255) | Non          | Nom d’affichage dans une interface utilisateur |
| WorkflowPropertyWorkflowId   | int           | Non          | ID du workflow auquel cette propriété est associée |
| WorkflowPropertyDataTypeId   | int           | Non          | ID du type de données défini pour la propriété |
| WorkflowPropertyDataTypeName | nvarchar(50)  | Non          | Nom du type de données défini pour la propriété |
| WorkflowPropertyIsState      | bit           | Non          | Ce champ identifie si la propriété de workflow contient l’état du workflow |

## <a name="vwworkflowstate"></a>vwWorkflowState

Cette vue représente les propriétés associées à un workflow. Pour chaque contrat, cette vue contient les données suivantes :

-   Définition d’application associée
-   Définition de workflow associé
-   Informations d’état du workflow

| Nom                         | Type          | Peut être Null | Description                                                                                                                                                                                                                                                   |
|------------------------------|---------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ApplicationId                | int           | Non          | Identificateur unique de l’application |
| ApplicationName              | nvarchar(50)  | Non          | Nom de l’application |
| ApplicationDisplayName       | nvarchar(255) | Non          | Description de l’application |
| ApplicationEnabled           | bit           | Non          | Identifie si l’application est actuellement activée.<br />**Remarque :** Même si une application peut être indiquée comme étant désactivée dans la base de données, les contrats associés restent sur la blockchain et les données relatives à ces contrats restent dans la base de données. |
| WorkflowId                   | int           | Non          | Identificateur unique du workflow |
| WorkflowName                 | nvarchar(50)  | Non          | Nom du workflow |
| WorkflowDisplayName          | nvarchar(255) | Non          | Nom affiché pour le workflow dans l’interface utilisateur |
| WorkflowDescription          | nvarchar(255) | Oui         | Description du workflow |
| WorkflowStateID              | int           | Non          | Identificateur unique de l’état |
| WorkflowStateName            | nvarchar(50)  | Non          | Nom de l’état |
| WorkflowStateDisplayName     | nvarchar(255) | Non          | Nom d’affichage de l’état dans l’interface utilisateur |
| WorkflowStateDescription     | nvarchar(255) | Oui         | Description de l’état du workflow |
| WorkflowStatePercentComplete | int           | Non          | Cette valeur identifie le pourcentage d’achèvement du workflow dans cet état |
| WorkflowStateValue           | nvarchar(50)  | Non          | Valeur de l’état |
| WorkflowStateStyle           | nvarchar(50)  | Non          | Texte descriptif donnant aux clients des conseils sur le rendu de cet état dans l’interface utilisateur. Les états pris en charge incluent *Succès* et *Échec* |
