---
title: Présentation de Stockage Azure - Stockage cloud sur Azure | Microsoft Docs
description: La plateforme principale Stockage Azure est la solution de stockage cloud de Microsoft. Le Stockage Azure offre un stockage des objets de données hautement disponible, sécurisé, fiable, évolutif et redondant.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/08/2020
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: b49e780714bd5e0662c6cb7ef208c473a8b45ccc
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498147"
---
# <a name="introduction-to-the-core-azure-storage-services"></a>Présentation des principaux services de stockage Azure

La plateforme Stockage Azure est une solution de stockage cloud de Microsoft pour les scénarios de stockage de données actuels. Les principaux services de stockage offrent un magasin d’objets hautement scalable pour des objets de données, le stockage sur disque pour les machines virtuelles Azure, un service de système de fichiers pour le cloud, un magasin de messagerie pour une messagerie fiable et un magasin NoSQL. Les services sont les suivants :

- **Durable et hautement disponible.** La redondance garantit que vos données sont sécurisées en cas de défaillance matérielle temporaire. Vous pouvez également choisir de répliquer des données entre des centres de données ou des régions géographiques pour une protection supplémentaire contre les catastrophes locales ou les catastrophes naturelles. Les données ainsi répliquées restent hautement disponibles en cas de panne inattendue.
- **Sécurisé.** Toutes les données écrites dans un compte de stockage Azure sont chiffrées par le service. Le Stockage Azure vous permet de contrôler de manière plus précise qui a accès à vos données.
- **Évolutif.** Le Stockage Azure est conçu pour être hautement évolutif afin de répondre aux besoins de stockage de données et de performances des applications actuelles.
- **Géré.** Azure gère la maintenance du matériel, les mises à jour et les problèmes critiques pour vous.
- **Accessible.** Les données dans le Stockage Azure sont accessibles n’importe où dans le monde via HTTP ou HTTPS. Microsoft fournit des bibliothèques clientes pour le Stockage Azure dans une variété de langages, dont .NET, Java, Node.js, Python, PHP, Ruby, Go et autres encore, ainsi qu’une API REST avancée. Le stockage Azure prend en charge l’écriture de scripts Azure PowerShell ou l’interface de ligne de commande Azure. De plus, le portail Azure et l’Explorateur Stockage Azure offrent des solutions visuelles simples pour utiliser vos données.  

## <a name="core-storage-services"></a>Services principaux de stockage

La plateforme Stockage Azure comprend les services de données suivants :

- [Objets blob Azure](../blobs/storage-blobs-introduction.md) : magasin d’objets hautement scalable pour les données texte ou binaires. Prend également en charge l’analytique Big Data via Data Lake Storage Gen2.
- [Azure Files](../files/storage-files-introduction.md) : partages de fichiers gérés pour les déploiements sur le cloud ou locaux.
- [Files d’attente Azure](../queues/storage-queues-introduction.md) : magasin de messagerie pour une messagerie fiable entre les composants d’application.
- [Tables Azure](../tables/table-storage-overview.md) : magasin NoSQL pour le stockage sans schéma de données structurées.
- [Disques Azure](../../virtual-machines/managed-disks-overview.md) : Volumes de stockage de niveau bloc pour les machines virtuelles Azure.

Chaque service est accessible via un compte de stockage. Pour commencer, consultez [Créer un compte de stockage](storage-account-create.md).

## <a name="example-scenarios"></a>Exemples de scénarios

Le tableau suivant compare les options Fichiers, Objets blob, Disques, Files d’attente et Tables, et il présente des exemples de scénarios appropriés pour chacune.

| Fonctionnalité | Description | Quand l’utiliser |
|--------------|-------------|-------------|
| **Azure Files** |Offre des partages de fichiers cloud complètement managés qui sont accessibles n’importe où via le protocole SMB (Server Message Block) standard.<br><br>Vous pouvez monter des partages de fichiers Azure à partir de déploiements cloud ou locaux de Windows, Linux et macOS. | Vous souhaitez développer et transférer une application dans le cloud qui utilise déjà les API du système de fichiers natif pour partager des données avec d’autres applications s’exécutant dans Azure.<br/><br/>Vous voulez remplacer ou compléter les serveurs de fichiers locaux ou les appareils NAS.<br><br> Vous souhaitez stocker les outils de développement et de débogage qui doivent être accessibles à partir de nombreuses machines virtuelles. |
| **Objets blob Azure** | Permet de stocker des données non structurées et d’y accéder à grande échelle dans des objets blob de blocs.<br/><br/>Prend également en charge [Azure Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md) pour les solutions d’analytique d’entreprise Big Data. | Vous souhaitez que votre application prenne en charge le streaming et l’accès aléatoire.<br/><br/>Vous souhaitez être en mesure d’accéder aux données d’application à partir de n’importe quel endroit.<br/><br/>Vous souhaitez générer un Data Lake d’entreprise sur Azure et effectuer une analytique de Big Data. |
| **Disques Azure** | Permet de stocker de manière permanente les données et d’y accéder à partir d’un disque dur virtuel joint. | Vous souhaitez développer et transférer des applications qui utilisent les API du système de fichiers natif pour lire et écrire des données sur des disques persistants.<br/><br/>Vous souhaitez stocker des données dont l’accès n’est pas requis à l’extérieur de la machine virtuelle à laquelle le disque est joint. |
| **Files d’attente Azure** | Permet la mise en file d’attente asynchrone des messages entre les composants de l’application. | Vous souhaitez découpler des composants de l’application et utiliser la messagerie asynchrone pour qu’ils communiquent entre eux.<br><br>Pour plus d’informations sur l’utilisation du stockage File d’attente par rapport aux files d’attente Service Bus, consultez [Files d’attente de stockage et files d’attente Service Bus - Comparaison et différences](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md). |
| **Tables Azure** | Vous permet de stocker des données NoSQL structurées dans le cloud, en fournissant un magasin de clés/attributs reposant sur une conception sans schéma. | Vous voulez stocker des jeux de données flexibles, comme des données utilisateur pour des applications web, des carnets d’adresses, des informations sur les appareils ou d’autres types de métadonnées requis par votre service. <br/><br/>Pour plus d’informations sur l’utilisation du stockage Table par rapport à l’API Azure Cosmos DB Table, consultez [Développement avec l’API Azure Cosmos DB Table et le stockage Table Azure](../../cosmos-db/table-support.md). |

## <a name="blob-storage"></a>Stockage d'objets blob

Le stockage Blob Azure est la solution de stockage d’objet de Microsoft pour le cloud. Le stockage Blob est optimisé pour stocker de grandes quantités de données non structurées, telles que des données texte ou binaires.

Le stockage Blob est idéal pour :

- Mise à disposition d’images ou de documents directement dans un navigateur.
- Stockage de fichiers pour un accès distribué.
- Diffusion en continu de vidéo et d’audio.
- Stockage de données pour la sauvegarde et la restauration, la récupération d’urgence et l’archivage.
- Stockage des données pour l’analyse par un service local ou hébergé par Azure.

Les objets du stockage Blob sont accessibles n’importe où dans le monde via HTTP ou HTTPS. Les utilisateurs ou applications clientes peuvent accéder aux objets blob via des URL, l’[API REST Stockage Azure](/rest/api/storageservices/blob-service-rest-api), [Azure PowerShell](/powershell/module/azure.storage), [Azure CLI](/cli/azure/storage) ou une bibliothèque de client Stockage Azure. Les bibliothèques de client de stockage sont disponibles dans plusieurs langages, tels que [.NET](/dotnet/api/overview/azure/storage), [Java](/java/api/overview/azure/storage), [Node.js](https://azure.github.io/azure-storage-node), [Python](https://azure-storage.readthedocs.io/), [PHP](https://azure.github.io/azure-storage-php/) et [Ruby](https://azure.github.io/azure-storage-ruby).

Pour plus d’informations sur le stockage Blob, consultez [Présentation du Stockage Blob](../blobs/storage-blobs-introduction.md).

## <a name="azure-files"></a>Azure Files

[Azure Files](../files/storage-files-introduction.md) vous permet de configurer les partages de fichiers réseau hautement disponibles qui sont accessibles à l’aide du protocole Server Message Block (SMB) standard. Cela signifie que plusieurs machines virtuelles peuvent partager les mêmes fichiers avec accès en lecture et en écriture. Vous pouvez également consulter les fichiers à l’aide de l’interface REST ou des bibliothèques clientes de stockage.

Ce qui distingue Azure Files des fichiers sur un partage de fichiers d’entreprise est le fait de pouvoir accéder aux fichiers depuis n’importe où dans le monde, à l’aide d’une URL qui pointe vers le fichier et inclut un jeton de signature (SAP) d’accès partagé. Vous pouvez générer des jetons SAP, qui autorisent un accès spécifique à une ressource privée pour une durée définie.

Les partages de fichiers peuvent être utilisés dans de nombreux scénarios courants :

- Par exemple, de nombreuses applications locales utilisent les partages de fichiers. Cette fonctionnalité simplifie la migration de ces applications qui partagent des données vers Azure. Si vous montez le partage de fichiers sur la même lettre de lecteur que celle utilisée par l’application locale, la partie de votre application qui accède au partage de fichiers doit fonctionner, avec très peu de modifications à apporter (voire aucune).

- Les fichiers de configuration peuvent être stockés sur un partage de fichiers et sont accessibles à partir de plusieurs machines virtuelles. Les outils et utilitaires utilisés par plusieurs développeurs d’un groupe peuvent être stockés sur un partage de fichiers, en veillant à ce que chacun puisse les trouver et à ce qu’ils utilisent la même version.

- Les journaux de ressources, les métriques et les vidages sur incident sont trois exemples de données qui peuvent être écrites dans un partage de fichiers et traitées ou analysées ultérieurement.

Pour plus d’informations sur Azure Files, consultez [Présentation d’Azure Files](../files/storage-files-introduction.md).

Certaines fonctionnalités SMB ne sont pas applicables au cloud. Pour plus d’informations, consultez [Features not supported by the Azure File service](/rest/api/storageservices/features-not-supported-by-the-azure-file-service) (Fonctionnalités non prises en charge par le service Azure File).

## <a name="queue-storage"></a>Stockage de files d'attente

Le service de files d’attente Azure sert à stocker et à récupérer des messages. La taille maximale des messages de file d’attente est de 64 Ko et une file d’attente peut contenir des millions de messages. Les files d’attente servent en général à stocker des listes de messages qui seront traités de façon asynchrone.

Par exemple, vous souhaitez que vos clients soient en mesure de charger des images, et vous souhaitez créer des miniatures de chaque image. La procédure normale serait de faire patienter votre client, le temps que vous créiez les miniatures tout en chargeant les images. L’utilisation d’une file d’attente est une autre possibilité. Lorsque le client a terminé le chargement, écrivez un message dans la file d’attente. Utilisez ensuite une fonction Azure afin de récupérer le message dans la file d’attente pour créer les miniatures. Chaque étape de ce processus peut être développée séparément, vous offrant ainsi plus de contrôle lorsque vous le réglez.

Pour plus d’informations sur les files d’attentes Azure, consultez [Introduction to Queues](../queues/storage-queues-introduction.md) (Présentation des files d’attente).

## <a name="table-storage"></a>Stockage de tables

Stockage de tables Azure fait maintenant partie d’Azure Cosmos DB. Pour consulter la documentation Stockage de tables Azure, consultez [Vue d’ensemble du stockage de table Azure](../tables/table-storage-overview.md). En plus du service Stockage de tables Azure existant, il existe une nouvelle API de Table d’Azure Cosmos DB qui propose des tables optimisées pour le débit, la distribution globale et les index secondaires automatiques. Pour en savoir plus et essayer la nouvelle expérience Premium, consultez [API Azure Cosmos DB Table](../../cosmos-db/table-introduction.md).

Pour plus d’informations sur le stockage de table, consultez [Overview of Azure Table storage](../tables/table-storage-overview.md) (Vue d’ensemble du stockage de table Azure).

## <a name="disk-storage"></a>Stockage sur disque

Un disque managé Azure est un disque dur virtuel (VHD). Vous pouvez le voir comme un disque physique sur un serveur local mais virtualisé. Les disques managés Azure sont stockés en tant qu’objets blob de page, qui sont un objet de stockage d’E/S aléatoire dans Azure. Nous appelons un disque managé « managé » car il s’agit d’une abstraction sur les objets blob de pages, les conteneurs d’objets de couleur et les comptes de stockage Azure. Avec les disques managés, il vous suffit d’approvisionner le disque et Azure s’occupe du reste.

Pour plus d’informations sur les disques managés, voir [Présentation des disques managés Azure](../../virtual-machines/managed-disks-overview.md).

## <a name="types-of-storage-accounts"></a>Types de compte de stockage

Le stockage Azure offre plusieurs types de comptes de stockage. Chaque type prend en charge différentes fonctionnalités et a son propre modèle tarifaire. Pour plus d’informations sur les types de comptes de stockage, voir [Vue d’ensemble des comptes de stockage Azure](storage-account-overview.md).

## <a name="secure-access-to-storage-accounts"></a>Sécuriser l’accès aux comptes de stockage

Chaque demande adressée au Stockage Azure doit être autorisée. Stockage Azure prend en charge les méthodes d’autorisation suivantes :

- **Intégration d’Azure Active Directory (Azure AD) pour les données Blob et de files d’attente.** Stockage Azure prend en charge l'authentification et l'autorisation avec Azure AD pour les services Blob et File d'attente via le Contrôle d'accès en fonction du rôle Azure (Azure RBAC). L’autorisation des demandes avec Azure AD est recommandée pour davantage de sécurité et de facilité d’utilisation. Pour plus d’informations, voir [Autoriser l’accès aux objets blob et files d’attente Azure à l’aide d’Azure Active Directory](storage-auth-aad.md).
- **Autorisation Azure AD sur SMB pour Azure Files.** Azure Files prend en charge l’autorisation basée sur l’identité sur SMB (Server Message Block) via Azure Active Directory Domain Services (Azure AD DS) ou Active Directory Domain Services local (préversion). Vos machines virtuelles Windows jointes à un domaine peuvent accéder aux partages de fichiers Azure en utilisant des informations d’identification Azure AD. Pour plus d’informations, consultez [Vue d’ensemble de la prise en charge de l’authentification basée sur l’identité Azure Files pour l’accès SMB](../files/storage-files-active-directory-overview.md) et [Planification d’un déploiement Azure Files](../files/storage-files-planning.md#identity).
- **Autorisation avec Clé partagée.** Les services Azure Stockage Blob, Fichiers, Files d’attente et Table prennent en charge l’autorisation avec une clé partagée. Un client qui utilise l’autorisation Clé partagée transmet avec chaque demande un en-tête signé à l’aide de la clé d’accès au compte de stockage. Pour plus d’informations, consultez [Autoriser avec une clé partagée](/rest/api/storageservices/authorize-with-shared-key).
- **Autorisation à l’aide de signatures d’accès partagé (SAP).** Une signature d’accès partagé (SAP) est une chaîne contenant un jeton de sécurité pouvant être ajouté à l’URI pour une ressource de stockage. Le jeton de sécurité encapsule des contraintes telles que les autorisations et l’intervalle d’accès. Pour plus d’informations, consultez la page [Utiliser des signatures d’accès partagé (SAS)](storage-sas-overview.md).
- **Accès anonyme aux conteneurs et objets blob.** Un conteneur et ses objets blob peuvent être publiquement disponibles. Lorsque vous spécifiez qu’un conteneur ou un objet blob est public, n’importe qui peut le lire de manière anonyme ; aucune authentification n’est requise. Pour plus d’informations, consultez [Gestion de l’accès en lecture anonyme aux conteneurs et aux objets blob](../blobs/anonymous-read-access-configure.md).

## <a name="encryption"></a>Chiffrement

Il existe deux types de chiffrement de base pour les services principaux de stockage. Pour en savoir plus sur la sécurité et le chiffrement, consultez le [guide de sécurité pour le stockage Azure](../blobs/security-recommendations.md).

### <a name="encryption-at-rest"></a>Chiffrement au repos

Le chiffrement de Stockage Azure protège et sauvegarde vos données afin de vous permettre de respecter les engagements de votre organisation en matière de sécurité et de conformité. Stockage Azure chiffre automatiquement vos données avant de les conserver sur le compte de stockage, et les déchiffre avant leur récupération. Les processus de gestion du chiffrement, du déchiffrement et des clés sont transparents pour les utilisateurs. Les clients peuvent également choisir de gérer leurs propres clés à l’aide d’Azure Key Vault. Pour plus d'informations, consultez [Fonctionnalité de chiffrement du service Stockage Azure pour les données au repos](storage-service-encryption.md).

### <a name="client-side-encryption"></a>chiffrement côté client

Les bibliothèques clientes de Stockage Azure fournissent des méthodes pour chiffrer les données à partir de la bibliothèque cliente avant de les envoyer sur le réseau et de déchiffrer la réponse. Les données chiffrées via le chiffrement côté client sont également chiffrées au repos par Stockage Azure. Pour plus d’informations sur le chiffrement côté client, voir [Chiffrement côté client avec .NET pour Stockage zure](storage-client-side-encryption.md).

## <a name="redundancy"></a>Redondance

Pour assurer la durabilité de vos données, Stockage Azure stocke plusieurs copies de vos données. Quand vous configurez votre compte de stockage, vous sélectionnez une option de redondance. Pour plus d’informations, consultez [Redondance de Stockage Azure](./storage-redundancy.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="transfer-data-to-and-from-azure-storage"></a>Transférer des données vers et depuis Stockage Azure

Vous avez plusieurs options de déplacement des données dans ou en dehors de Stockage Azure. L’option que vous choisissez dépend de la taille de votre jeu de données et de votre bande passante réseau. Pour plus d’informations, consultez [Choisir une solution Azure pour le transfert de données](storage-choose-data-transfer-solution.md).

## <a name="pricing"></a>Tarifs

Lors des décisions concernant le stockage et l’accès de vos données, vous devez également prendre en considération les coûts impliqués. Pour plus d’informations, consultez [Tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

## <a name="storage-apis-libraries-and-tools"></a>Outils, bibliothèques et API de stockage

Vous pouvez accéder aux ressources dans un compte de stockage à l’aide de tout langage capable de créer des requêtes HTTP/HTTPS. Par ailleurs, les services principaux Stockage Azure offrent des bibliothèques de programmation pour plusieurs langages répandus. Ces bibliothèques simplifient l’utilisation du stockage Azure sous de nombreux aspects en gérant des détails tels que l’invocation synchrone et asynchrone, le traitement par lots des opérations, la gestion des exceptions, les nouvelles tentatives automatiques, le comportement opérationnel, etc. Des bibliothèques sont actuellement disponibles pour les langages et les plateformes suivants (d’autres sont à l’étude) :

### <a name="azure-storage-data-api-and-library-references"></a>Références d’API et de bibliothèque de données de stockage Azure

- [API REST d’Azure Storage](/rest/api/storageservices/)
- [Bibliothèque cliente Stockage Azure pour .NET](/dotnet/api/overview/azure/storage)
- [Bibliothèque cliente Stockage Azure pour Java/Android](/java/api/overview/azure/storage)
- [Bibliothèque cliente Stockage Azure pour Node.js](/javascript/api/overview/azure/storage-overview)
- [Bibliothèque cliente Stockage Azure pour Python](https://github.com/Azure/azure-storage-python)
- [Bibliothèque cliente Stockage Azure pour PHP](https://github.com/Azure/azure-storage-php)
- [Bibliothèque cliente Stockage Azure pour Ruby](https://github.com/Azure/azure-storage-ruby)
- [Bibliothèque cliente Stockage Azure pour C++](https://github.com/Azure/azure-storage-cpp)

### <a name="azure-storage-management-api-and-library-references"></a>Références d’API et de bibliothèque de gestion du stockage Azure

- [API REST des fournisseurs de ressources de stockage](/rest/api/storagerp/)
- [Bibliothèque cliente des fournisseurs de ressources de stockage pour .NET](/dotnet/api/overview/azure/storage/management)
- [API REST de gestion des services de stockage (classique)](/previous-versions/azure/reference/ee460790(v=azure.100))

### <a name="azure-storage-data-movement-api-and-library-references"></a>Références d’API et de bibliothèque de déplacement de données de stockage Azure

- [API REST du service Import/Export Storage](/rest/api/storageimportexport/)
- [Bibliothèque cliente de déplacement de données de stockage pour .NET](/dotnet/api/microsoft.azure.storage.datamovement)

### <a name="tools-and-utilities"></a>Outils et utilitaires

- [Applets de commande Azure PowerShell pour le stockage](/powershell/module/az.storage)
- [Applets de commande Azure CLI pour le stockage](/cli/azure/storage)
- [Utilitaire de ligne de commande AzCopy](https://aka.ms/downloadazcopy)
- L’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
- [Modèles Azure Resource Manager pour Stockage Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Storage)

## <a name="next-steps"></a>Étapes suivantes

Pour démarrer et exécuter avec les services Stockage Azure, consultez la section [Créer un compte de stockage](storage-account-create.md).