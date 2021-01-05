---
title: Architecture Azure Blockchain Workbench
description: Présentation de l’architecture Azure Blockchain Workbench Preview et de ses composants.
ms.date: 09/05/2019
ms.topic: conceptual
ms.reviewer: brendal
ms.openlocfilehash: 1fff70ef2eeb1dc27d33769fd48fe5923f56717b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021603"
---
# <a name="azure-blockchain-workbench-architecture"></a>Architecture Azure Blockchain Workbench

Azure Blockchain Workbench Preview simplifie le développement d’applications blockchain en proposant une solution qui utilise plusieurs composants Azure. Blockchain Workbench peut être déployé à l’aide d’un modèle de solution dans la Place de marché Microsoft Azure. Ce modèle permet aux utilisateurs de choisir les modules et les composants à déployer, comme la pile blockchain, le type d’application cliente et la prise en charge de l’intégration IoT. Une fois déployé, Blockchain Workbench fournit un accès à une application web, une application iOS et une application Android.

![Architecture Blockchain Workbench](./media/architecture/architecture.png)

## <a name="identity-and-authentication"></a>Identité et authentification

À l’aide de Blockchain Workbench, un consortium peut fédérer des identités d’entreprise à l’aide d’Azure Active Directory (Azure AD). Workbench génère de nouveaux comptes d’utilisateur pour les identités on-chain avec avec les identités d’entreprise stockées dans Azure AD. Le mappage d’identité facilite la connexion authentifiée aux applications et API clients et utilise les stratégies d’authentification des organisations. Workbench fournit également la possibilité d’associer des identités d’entreprise à des rôles spécifiques au sein d’un contrat intelligent donné. En outre, Workbench fournit un mécanisme pour identifier les actions que ces rôles peuvent entreprendre et à quel moment.

Une fois Blockchain Workbench déployé, les utilisateurs interagissent avec Blockchain Workbench via les applications clientes, l’API cliente basée sur REST ou l’API de messagerie. Dans tous les cas, les interactions doivent être authentifiées via Azure Active Directory (Azure AD) ou par le biais d’informations d’identification spécifiques à l’appareil.

Les utilisateurs fédèrent leurs identités dans un consortium Azure AD en envoyant un e-mail d’invitation aux participants à leur adresse e-mail. Lors de la connexion, ces utilisateurs sont authentifiés à l’aide du nom, du mot de passe et des stratégies. Par exemple, l’authentification à deux facteurs de leur organisation.

Azure AD permet de gérer tous les utilisateurs qui ont accès à Blockchain Workbench. Chaque appareil se connectant à un contrat intelligent est également associé à Azure AD.

Azure AD est aussi utilisé pour affecter des utilisateurs à un groupe d’administration spécial. Les utilisateurs associés au groupe d’administrateurs ont accès à des droits et des actions dans Blockchain Workbench, comme le déploiement de contrats et l’octroi d’autorisations à un utilisateur pour qu’il accède à un contrat. Les utilisateurs en dehors de ce groupe n’ont pas accès aux actions de l’administrateur.

## <a name="client-applications"></a>Applications clientes

Workbench fournit des applications clientes générées automatiquement pour le web et les appareils mobiles (iOS, Android), qui peuvent être utilisées pour valider, tester et consulter des applications blockchain. L’interface de l’application est générée de façon dynamique en fonction des métadonnées du contrat intelligent et peut prendre en charge tous les cas d’usage. Les applications clientes proposent une interface frontale orientée utilisateur pour les applications blockchain complètes générées par Blockchain Workbench. Les applications clientes authentifient les utilisateurs via Azure Active Directory (Azure AD) et présentent une expérience utilisateur adaptée au contexte métier du contrat intelligent. L’expérience utilisateur permet de créer des instances de contrat intelligent par des personnes autorisées, puis d’exécuter certains types de transactions à des endroits appropriés du processus métier que le contrat intelligent représente.

Dans l’application web, les utilisateurs autorisés peuvent accéder à la console Administrateur. La console est disponible pour les utilisateurs du groupe d’administration dans Azure AD et fournit l’accès aux fonctionnalités suivantes :

* Déploiement de contrats intelligents fournis par Microsoft pour des scénarios courants. Par exemple, un scénario de transfert de ressources.
* Chargement et déploiement de leurs propres contrats intelligents.
* Attribution d’un accès utilisateur au contrat intelligent dans le contexte d’un rôle spécifique.

Pour plus d'informations, consultez les [exemples d'applications clientes Azure Blockchain Workbench sur GitHub](https://github.com/Azure-Samples/blockchain-devkit/tree/master/connect/mobile).

## <a name="gateway-service-api"></a>API de service de passerelle

Blockchain Workbench inclut une API de service de passerelle basée sur REST. Lorsque vous écrivez dans un blockchain, l’API génère des messages et les transmet à un répartiteur d’événements. Lorsque l’API demande des données, les demandes sont envoyées à la base de données off-chain. La base de données contient un réplica des métadonnées et données on-chain qui fournit des informations relatives à la configuration et au contexte pour les contrats intelligents pris en charge. Les demandes renvoient les données requises à partir du réplica off-chain dans un format spécifié par les métadonnées pour le contrat.

Les développeurs peuvent accéder à l’API de service de passerelle pour créer ou intégrer les solutions blockchain sans dépendre des applications clientes Blockchain Workbench.

> [!NOTE]
> Pour activer l’accès authentifié à l’API, deux applications clientes sont inscrites dans Azure Active Directory. Azure Active Directory requiert des inscriptions d’applications distinctes pour chaque type d’application (native et web).

## <a name="message-broker-for-incoming-messages"></a>Répartiteur de messages pour les messages entrants

Les développeurs qui souhaitent envoyer des messages directement à Blockchain Workbench peuvent envoyer des messages directement à Service Bus. Par exemple, les API de messages peuvent être utilisées pour l’intégration entre les systèmes ou les appareils IoT.

## <a name="message-broker-for-downstream-consumers"></a>Répartiteur de messages pour les consommateurs en aval

Pendant le cycle de vie de l’application, des événements se produisent. Des événements peuvent être déclenchés par l’API de passerelle ou sur le registre. Des notifications d’événements peuvent initier le code en aval, en fonction de l’événement.

Blockchain Workbench déploie automatiquement les deux types de consommateurs d’événements. L’un est déclenché par des événements blockchain pour remplir le magasin SQL off-chain. L’autre permet de capturer les métadonnées des événements générés par l’API associée au chargement et au stockage des documents.

## <a name="message-consumers"></a>Consommateurs de messages

 Des consommateurs de messages prennent des messages de Service Bus. Le modèle d’événement sous-jacent pour les consommateurs de messages autorise les extensions de systèmes et services supplémentaires. Par exemple, vous pouvez ajouter un support pour remplir CosmosDB ou évaluer des messages à l’aide d’Azure Stream Analytics. Les sections suivantes décrivent les consommateurs de messages inclus dans Blockchain Workbench.

### <a name="distributed-ledger-consumer"></a>Consommateur de registre distribué

Les messages de la technologie de registre distribué (DLT, Distributed Ledger Technology) contiennent les métadonnées des transactions à écrire dans le blockchain. Le consommateur récupère les messages et envoie (push) les données à un générateur de transactions, à un signataire et à un routeur.

### <a name="database-consumer"></a>Consommateur de bases de données

Le consommateur de base de données prend des messages de Service Bus et envoie (push) les données à une base de données jointe, comme une base de données dans Azure SQL Database.

### <a name="storage-consumer"></a>Consommateur de stockage

Le consommateur de stockage prend des messages de Service Bus et envoie (push) les données à un stockage joint. Par exemple, le stockage des documents hachés dans Stockage Azure.

## <a name="transaction-builder-and-signer"></a>Générateur de transactions et signataire

Si un message sur le répartiteur de messages entrants doit être écrit dans le blockchain, il sera traité par le consommateur DLT. Le consommateur DLT est un service qui récupère le message contenant les métadonnées d’une transaction à exécuter, puis envoie les informations au *générateur de transactions et au signataire*. Le *générateur de transactions et le signataire* assemblent une transaction de blockchain basée sur les données et la destination de blockchain souhaitée. Une fois assemblée, la transaction est signée. Les clés privées sont stockées dans Azure Key Vault.

 Blockchain Workbench récupère la clé privée appropriée auprès de Key Vault et signe la transaction en dehors de Key Vault. Une fois signée, la transaction est envoyée aux routeurs et registres de transaction.

## <a name="transaction-routers-and-ledgers"></a>Routeurs et registres de transaction

Des routeurs et registres de transaction prennent les transactions signées et les acheminent vers le blockchain approprié. Actuellement, Blockchain Workbench prend en charge Ethereum comme blockchain cible.

## <a name="dlt-watcher"></a>Observateur DLT

Un observateur DLT surveille les événements se produisant sur les blockchains joints à Blockchain Workbench.
Les événements reflètent les informations relatives aux individus et aux systèmes. Par exemple, la création d’instances de contrat, l’exécution de transactions, ainsi que les modifications d’état. Les événements sont capturés et envoyés au répartiteur de messages sortants pour qu’ils puissent être utilisés par les consommateurs en aval.

Par exemple, le consommateur SQL surveille des événements, les utilise et remplit la base de données avec les valeurs incluses. La copie active la recréation d’un réplica des données on-chain dans un magasin off-chain.

## <a name="azure-sql-database"></a>Azure SQL Database

La base de données jointe à Blockchain Workbench stocke les définitions de contrat, les métadonnées de configuration et un réplica compatible SQL des données stockées dans le blockchain. Ces données peuvent facilement être interrogées, visualisées ou analysées en accédant directement à la base de données. Les développeurs et les autres utilisateurs peuvent utiliser la base de données pour créer des rapports, effectuer des analyses ou d’autres intégrations orientées données. Par exemple, les utilisateurs peuvent visualiser des données de transaction à l’aide de Power BI.

Ce stockage off-chain offre aux organisations professionnelles la possibilité d’interroger des données dans SQL plutôt que dans un registre blockchain. En outre, en effectuant une standardisation sur un schéma standard indépendant des piles de la technologie blockchain, le stockage off-chain permet de réutiliser les rapports et les autres artefacts entre les projets, scénarios et organisations.

## <a name="azure-storage"></a>Stockage Azure

Stockage Azure est utilisé pour stocker les contrats, ainsi que les métadonnées associées.

Des bons de commandes et connaissements aux images utilisées dans les actualités et l’imagerie médicale, en passant par les vidéos de diverses origines, notamment des caméras mobiles des forces de l’ordre et des films, les documents occupent une place importante dans de nombreux scénarios orientés blockchain. Les documents ne sont pas appropriés pour être placés directement sur le blockchain.

Blockchain Workbench prend en charge la possibilité d’ajouter des documents ou d’autres contenus multimédias avec une logique métier blockchain. Un code de hachage du contenu du document ou du contenu multimédia est stocké dans le blockchain, et le document ou le contenu multimédia réel est stocké dans Stockage Azure. Les informations de transaction associées sont transmises au répartiteur de messages entrants, empaquetées, signées et acheminées vers le blockchain. Ce processus déclenche des événements, qui sont partagés via le répartiteur de messages sortants. La base de données SQL utilise ces informations et les envoie à la base de données pour les demander plus tard. Des systèmes en aval peuvent également utiliser ces événements pour agir comme il se doit.

## <a name="monitoring"></a>Surveillance

Workbench propose un journal des applications utilisant Application Insights et Azure Monitor. Application Insights est utilisé pour stocker toutes les informations enregistrées à partir de Blockchain Workbench et inclut les erreurs, avertissements et opérations réussies. Application Insights peut aider les développeurs à déboguer des problèmes avec Blockchain Workbench. 

Azure Monitor fournit des informations sur l’intégrité du réseau blockchain. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer Azure Blockchain Workbench](./deploy.md)
