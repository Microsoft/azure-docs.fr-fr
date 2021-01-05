---
title: Rôles RBAC Synapse
description: Cet article décrit les rôles RBAC Synapse intégrés.
author: billgib
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 12/1/2020
ms.author: billgib
ms.reviewer: jrasnick
ms.openlocfilehash: a978113265e5e61c0fc09ef0daeb1da9826f294d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572794"
---
# <a name="synapse-rbac-roles"></a>Rôles RBAC Synapse

L’article décrit les rôles RBAC Synapse intégrés, les autorisations qu’ils accordent, ainsi que les étendues au niveau desquelles ils peuvent être utilisés.  

## <a name="whats-changed-since-the-preview"></a>Qu’est-ce qui a changé depuis la préversion ?
Pour les utilisateurs connaissant les rôles RBAC Synapse fournis dans la préversion, voici les modifications apportées :
- L’Administrateur d’espace de travail a été renommé **Administrateur Synapse**
- L’Administrateur Apache Spark a été renommé **Administrateur Apache Spark Synapse** et est autorisé à voir tous les artefacts de code publiés, y compris les scripts SQL.  Ce rôle ne permet plus d’utiliser l’espace de travail MSI, qui requiert le rôle d’Utilisateur d’informations d’identification Synapse.  Cette autorisation est nécessaire pour exécuter des pipelines. 
- L’Administrateur SQL a été renommé **Administrateur Synapse SQL** et est autorisé à voir tous les artefacts de code publiés, y compris les notebooks et travaux Spark.  Ce rôle ne permet plus d’utiliser l’espace de travail MSI, qui requiert le rôle d’Utilisateur d’informations d’identification Synapse. Cette autorisation est nécessaire pour exécuter des pipelines.
- De **nouveaux rôles RBAC Synapse affinés** ont été introduits et se concentrent sur la prise en charge des personnages de développement et d’opérations plutôt que sur des runtimes d’analyse spécifiques.  
- De **nouvelles étendues de niveau inférieur** ont été introduites pour plusieurs rôles.  Ces étendues permettent de limiter les rôles à des ressources ou objets spécifiques.

>[!Note]
>Les **nouveaux rôles RBAC Synapse et étendues de niveau inférieur sont actuellement en préversion**.  Vous êtes invité à utiliser ces nouveaux rôles et étendues entièrement pris en charge, et à fournir des commentaires relatifs à leur utilisation.

## <a name="built-in-synapse-rbac-roles-and-scopes"></a>Rôles RBAC Synapse intégrés et étendues

Le tableau suivant décrit les rôles intégrés et les étendues à partir desquelles ils peuvent être utilisés.

>[!Note]
> Les utilisateurs disposant d’un rôle RBAC Synapse dans n’importe quelle étendue disposent automatiquement du rôle d’Utilisateur Synapse dans l’étendue de l’espace de travail. 

|Role |Autorisations|Étendues|
|---|---|-----|
|Administrateur Synapse  |Accès Synapse complet aux pools SQL serverless, pools Apache Spark et runtimes d’intégration.  Comprend un accès en création, lecture, mise à jour et suppression portant sur tous les artefacts de code publiés.  Comprend les autorisations d’opérateur de calcul, de gestionnaire des données liées et d’utilisateur d’informations d’identification sur les informations d’identification système de l’espace de travail.  Comprend l’attribution de rôles RBAC Synapse.  Des autorisations Azure sont requises pour créer, supprimer et gérer des ressources de calcul. </br></br>_Peut lire et écrire des artefacts</br> Peut effectuer toutes les actions sur les activités Spark.</br> Peut afficher les journaux des pools Spark</br> Peut afficher le notebook enregistré et la sortie du pipeline</br> Peut utiliser les secrets stockés par les services liés ou les informations d’identification</br>Peut se connecter à des points de terminaison SQL serverless avec les autorisations SQL `db_datareader`, `db_datawriter`, `connect` et `grant`</br> Peut attribuer et révoquer des rôles RBAC Synapse au niveau de l’étendue actuelle_|Espace de travail </br> Pool Spark<br/>Runtime d’intégration </br>Service lié</br>Informations d'identification |
|Administrateur Synapse Apache Spark</br>|Accès Synapse complet aux pools Apache Spark.  Accès en création, lecture, mise à jour et suppression aux définitions de travail Spark publiées, aux notebooks et à leurs sorties, ainsi qu’aux bibliothèques, services liés et informations d’identification.  Comprend un accès en lecture à tous les autres artefacts de code publiés. Ne permet pas d’utiliser des informations d’identification et d’exécuter des pipelines. Ne comprend pas l’octroi d’accès. </br></br>_Peut effectuer toutes les actions sur les artefacts Spark</br>Peut effectuer toutes les actions sur les activités Spark_|Espace de travail</br>Pool Spark|
|Administrateur Synapse SQL|Accès Synapse complet aux pools SQL serverless.  Accès en création, lecture, mise à jour et suppression aux scripts SQL publiés, informations d’identification et services liés.  Comprend un accès en lecture à tous les autres artefacts de code publiés.  Ne permet pas d’utiliser des informations d’identification et d’exécuter des pipelines. Ne comprend pas l’octroi d’accès. </br></br>*Peut effectuer toutes les actions sur les scripts SQL<br/>Peut se connecter à des points de terminaison SQL serverless avec les autorisations SQL `db_datareader`, `db_datawriter`, `connect` et `grant`*|Espace de travail|
|Contributeur Synapse|Accès Synapse complet aux pools SQL serverless, pools Apache Spark et runtimes d’intégration.  Comprend un accès en création, lecture, mise à jour et suppression à tous les artefacts de code publiés et leurs sorties, y compris les informations d’identification et les services liés.  Comprend les autorisations d’opérateur de calcul. Ne permet pas d’utiliser des informations d’identification et d’exécuter des pipelines. Ne comprend pas l’octroi d’accès. </br></br>_Peut lire et écrire des artefacts</br>Peut afficher le notebook enregistré et la sortie du pipeline</br>Peut effectuer toutes les actions sur les activités Spark</br>Peut afficher les journaux des pools Spark_|Espace de travail </br> Pool Spark<br/> Runtime d’intégration|
|Éditeur d'artefact Synapse|Accès en création, lecture, mise à jour et suppression portant sur les artefacts de code publiés et leurs sorties. Ne comprend pas l’autorisation d’exécuter du code ou des pipelines, ou d’accorder l’accès. </br></br>_Peut lire les artefacts publiés et publier des artefacts</br>Peut afficher le notebook enregistré, le travail Spark et la sortie du pipeline_|Espace de travail
|Utilisateur d'artefact Synapse|Accès en lecture portant sur les artefacts de code publiés et leurs sorties. Peut créer des artefacts, mais ne peut pas publier de modifications, ni exécuter du code sans autorisations supplémentaires.|Espace de travail
|Opérateur de capacité de calcul Synapse |Soumettre des travaux et des notebooks Spark et afficher les journaux.  Comprend l’annulation des travaux Spark soumis par n’importe quel utilisateur. Nécessite des autorisations supplémentaires d’utilisation des informations d’identification sur l’identité du système de l’espace de travail pour exécuter des pipelines, afficher des exécutions de pipeline et des sorties. </br></br>_Peut soumettre et annuler des travaux, y compris des travaux soumis par d’autres</br>Peut afficher les journaux des pools Spark_|Espace de travail</br>Pool Spark</br>Runtime d’intégration|
|Utilisateur d’informations d’identification Synapse|Exécution et configuration, utilisation des secrets dans les informations d’identification et des services liés dans des activités telles que les exécutions de pipeline. Pour exécuter des pipelines, ce rôle est requis, étendu à l’identité du système de l’espace de travail. </br></br>_Étendu à des informations d’identification, permet l’accès aux données via un service lié protégé par des informations d’identification (nécessite également une autorisation d’utilisation du calcul) </br>Permet l’exécution de pipelines protégés par les informations d’identification de l’identité du système de l’espace de travail (avec une autorisation d’utilisation du calcul supplémentaire)_|Espace de travail </br>Service lié</br>Informations d'identification
|Gestionnaire des données liées Synapse|Création et gestion des points de terminaison privés managés, des services liés et des informations d’identification. Peut créer des points de terminaison privés managés utilisant des services liés protégés par des informations d’identification|Espace de travail|
|Utilisateur Synapse|Répertorier et afficher les détails des pools SQL, des pools Apache Spark, des runtimes d’intégration, des services liés et informations d’identification publiés. Ne comprend pas d’autres artefacts de code publiés.  Peut créer des artefacts, mais ne peut pas exécuter ou publier sans autorisations supplémentaires.</br></br>_Peut répertorier et lire les pools Spark, les runtimes d’intégration._|Espace de travail, pool Spark</br>Service lié </br>Informations d'identification|

## <a name="synapse-rbac-roles-and-the-actions-they-permit"></a>Rôles RBAC Synapse et actions qu’ils permettent

>[!Note]
>- Toutes les actions répertoriées dans les tableaux ci-dessous sont préfixées, « Microsoft.Synapse/... »</br>
>- Toutes les actions de lecture, d’écriture et de suppression d’artefacts portent sur les artefacts publiés dans le service en ligne.  Ces autorisations n’affectent pas l’accès aux artefacts d’un référentiel Git connecté.  

Le tableau suivant répertorie les rôles intégrés et les actions/autorisations prises en charge par chacun.

Rôle|Actions
--|--
Administrateur Synapse|workspaces/read</br>workspaces/roleAssignments/write, delete</br>workspaces/managedPrivateEndpoint/write, delete</br>workspaces/bigDataPools/useCompute/action</br>workspaces/bigDataPools/viewLogs/action</br>workspaces/integrationRuntimes/useCompute/action</br>workspaces/artifacts/read</br>workspaces/notebooks/write, delete</br>workspaces/sparkJobDefinitions/write, delete</br>workspaces/sqlScripts/write, delete</br>workspaces/dataFlows/write, delete</br>workspaces/pipelines/write, delete</br>workspaces/triggers/write, delete</br>workspaces/datasets/write, delete</br>workspaces/libraries/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete</br>workspaces/notebooks/viewOutputs/action</br>workspaces/pipelines/viewOutputs/action</br>workspaces/linkedServices/useSecret/action</br>workspaces/credentials/useSecret/action|
|Administrateur Synapse Apache Spark|workspaces/read</br>workspaces/bigDataPools/useCompute/action</br>workspaces/bigDataPools/viewLogs/action</br>workspaces/notebooks/viewOutputs/action</br>workspaces/artifacts/read</br>workspaces/notebooks/write, delete</br>workspaces/sparkJobDefinitions/write, delete</br>workspaces/libraries/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete|
|Administrateur Synapse SQL|workspaces/read</br>workspaces/artifacts/read</br>workspaces/sqlScripts/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete|
|Contributeur Synapse|workspaces/read</br>workspaces/bigDataPools/useCompute/action</br>workspaces/bigDataPools/viewLogs/action</br>workspaces/integrationRuntimes/useCompute/action</br>workspaces/integrationRuntimes/viewLogs/action</br>workspaces/artifacts/read</br>workspaces/notebooks/write, delete</br>workspaces/sparkJobDefinitions/write, delete</br>workspaces/sqlScripts/write, delete</br>workspaces/dataFlows/write, delete</br>workspaces/pipelines/write, delete</br>workspaces/triggers/write, delete</br>workspaces/datasets/write, delete</br>workspaces/libraries/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete</br>workspaces/notebooks/viewOutputs/action</br>workspaces/pipelines/viewOutputs/action|
|Éditeur d'artefact Synapse|workspaces/read</br>workspaces/artifacts/read</br>workspaces/notebooks/write, delete</br>workspaces/sparkJobDefinitions/write, delete</br>workspaces/sqlScripts/write, delete</br>workspaces/dataFlows/write, delete</br>workspaces/pipelines/write, delete</br>workspaces/triggers/write, delete</br>workspaces/datasets/write, delete</br>workspaces/libraries/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete</br>workspaces/notebooks/viewOutputs/action</br>workspaces/pipelines/viewOutputs/action|
|Utilisateur d'artefact Synapse|workspaces/read</br>workspaces/artifacts/read</br>workspaces/notebooks/viewOutputs/action</br>workspaces/pipelines/viewOutputs/action|
|Opérateur de capacité de calcul Synapse |workspaces/read</br>workspaces/bigDataPools/useCompute/action</br>workspaces/bigDataPools/viewLogs/action</br>workspaces/integrationRuntimes/useCompute/action</br>workspaces/integrationRuntimes/viewLogs/action|
|Utilisateur d’informations d’identification Synapse|workspaces/read</br>workspaces/linkedServices/useSecret/action</br>workspaces/credentials/useSecret/action|
|Gestionnaire des données liées Synapse|workspaces/read</br>workspaces/managedPrivateEndpoint/write, delete</br>workspaces/linkedServices/write, delete</br>workspaces/credentials/write, delete|
|Utilisateur Synapse|workspaces/read|

## <a name="synapse-rbac-actions-and-the-roles-that-permit-them"></a>Actions RBAC Synapse et rôles qui les permettent

Le tableau suivant répertorie les actions Synapse et les rôles intégrés qui permettent ces actions :

Action|Role
--|--
workspaces/read|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Administrateur Synapse SQL</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Utilisateur d'artefact Synapse</br>Opérateur de capacité de calcul Synapse </br>Utilisateur d’informations d’identification Synapse</br>Gestionnaire des données liées Synapse</br>Utilisateur Synapse 
workspaces/roleAssignments/write, delete|Administrateur Synapse
workspaces/managedPrivateEndpoint/write, delete|Administrateur Synapse</br>Gestionnaire des données liées Synapse
workspaces/bigDataPools/useCompute/action|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Opérateur de capacité de calcul Synapse 
workspaces/bigDataPools/viewLogs/action|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Opérateur de capacité de calcul Synapse 
workspaces/integrationRuntimes/useCompute/action|Administrateur Synapse</br>Contributeur Synapse</br>Opérateur de capacité de calcul Synapse 
workspaces/artifacts/read|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Administrateur Synapse SQL</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Utilisateur d'artefact Synapse
workspaces/notebooks/write, delete|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/sparkJobDefinitions/write, delete|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/sqlScripts/write, delete|Administrateur Synapse</br>Administrateur Synapse SQL</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/dataFlows/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/pipelines/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/triggers/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/datasets/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/libraries/write, delete|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Éditeur d'artefact Synapse
workspaces/linkedServices/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Gestionnaire des données liées Synapse
workspaces/credentials/write, delete|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Gestionnaire des données liées Synapse
workspaces/notebooks/viewOutputs/action|Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Utilisateur d'artefact Synapse
workspaces/pipelines/viewOutputs/action|Administrateur Synapse</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Utilisateur d'artefact Synapse
workspaces/linkedServices/useSecret/action|Administrateur Synapse</br>Utilisateur d’informations d’identification Synapse
workspaces/credentials/useSecret/action|Administrateur Synapse</br>Utilisateur d’informations d’identification Synapse

## <a name="synapse-rbac-scopes-and-their-supported-roles"></a>Étendues RBAC Synapse et rôles correspondants pris en charge

Le tableau ci-dessous répertorie les étendues RBAC Synapse et les rôles qui pouvant être attribués à chaque étendue. 

>[!note]
>Pour créer ou supprimer un objet, vous devez disposer d’autorisations dans une étendue de niveau supérieur.

Étendue|Rôles
--|--
Espace de travail |Administrateur Synapse</br>Administrateur Synapse Apache Spark</br>Administrateur Synapse SQL</br>Contributeur Synapse</br>Éditeur d'artefact Synapse</br>Utilisateur d'artefact Synapse</br>Opérateur de capacité de calcul Synapse </br>Utilisateur d’informations d’identification Synapse</br>Gestionnaire des données liées Synapse</br>Utilisateur Synapse
Pool Apache Spark | Administrateur Synapse </br>Contributeur Synapse </br> Opérateur de capacité de calcul Synapse
Runtime d’intégration | Administrateur Synapse </br>Contributeur Synapse </br> Opérateur de capacité de calcul Synapse
Service lié |Administrateur Synapse </br>Utilisateur d’informations d’identification Synapse
Informations d'identification |Administrateur Synapse </br>Utilisateur d’informations d’identification Synapse
 
>[!note]
>Tous les rôles et actions d’artefact sont étendus au niveau de l’espace de travail. 

## <a name="next-steps"></a>Étapes suivantes

Découvrir [comment passer en revue les attributions de rôles RBAC Synapse](./how-to-review-synapse-rbac-role-assignments.md) pour un espace de travail

Découvrir [comment attribuer des rôles RBAC Synapse](./how-to-manage-synapse-rbac-role-assignments.md)