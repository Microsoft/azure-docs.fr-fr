---
title: Modèles de chiffrement de données dans Microsoft Azure
description: Cet article fournit une vue d’ensemble des modèles de chiffrement de données dans Microsoft Azure.
services: security
documentationcenter: na
author: msmbaldwin
manager: rkarlin
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mbaldwin
ms.openlocfilehash: 141daa485fae5aba2db23647fada30ba5b621cd0
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854612"
---
# <a name="data-encryption-models"></a>Modèles de chiffrement des données

Une compréhension des différents modèles de chiffrement, ainsi que de leurs avantages et inconvénients, est essentielle pour comprendre comment les différents fournisseurs de ressources dans Azure implémentent le chiffrement au repos. Ces définitions sont partagées par tous les fournisseurs de ressources dans Azure, ce qui garantit un langage et une taxonomie communs.

Il existe trois scénarios de chiffrement côté serveur :

- Chiffrement côté serveur à l’aide de clés gérées par le service
  - Les fournisseurs de ressources Azure effectuent les opérations de chiffrement et de déchiffrement
  - Microsoft gère les clés
  - Fonctionnalité cloud complète

- Chiffrement côté serveur à l’aide de clés gérées par le client dans Azure Key Vault
  - Les fournisseurs de ressources Azure effectuent les opérations de chiffrement et de déchiffrement
  - Le client contrôle les clés via Azure Key Vault
  - Fonctionnalité cloud complète

- Chiffrement côté serveur à l’aide de clés gérées par le client sur du matériel contrôlé par le client
  - Les fournisseurs de ressources Azure effectuent les opérations de chiffrement et de déchiffrement
  - Le client contrôle les clés sur du matériel contrôlé par le client
  - Fonctionnalité cloud complète

Les modèles de chiffrement côté serveur se réfèrent au chiffrement qui est effectué par le service Azure. Dans ce modèle, le fournisseur de ressources effectue les opérations de chiffrement et de déchiffrement. Par exemple, le stockage Azure peut recevoir des données dans des opérations en texte brut, et effectue le chiffrement et le déchiffrement en interne. Le fournisseur de ressources peut utiliser des clés de chiffrement qui sont gérées par Microsoft ou par le client, selon la configuration fournie.

![Serveur](./media/encryption-models/azure-security-encryption-atrest-fig3.png)

Chacun des modèles de chiffrement au repos côté serveur a des particularités distinctives dans la gestion des clés. Il s’agit notamment de l’endroit et de la façon dont les clés de chiffrement sont créées et stockées, ainsi que des modèles d’accès et des procédures de rotation des clés.  

Pour le chiffrement côté client, considérez les éléments suivants :

- Les services Azure ne peuvent pas voir les données déchiffrées
- Les clients gèrent et stockent les clés localement (ou dans d’autres magasins sécurisés). Les clés ne sont pas disponibles pour les services Azure
- Fonctionnalité cloud réduite

Les modèles de chiffrement pris en charge dans Azure se divisent en deux groupes principaux : « Chiffrement client » et « Chiffrement côté serveur », comme mentionné précédemment. Indépendamment du modèle de chiffrement au repos utilisé, les services Azure recommandent toujours l’utilisation d’un transport sécurisé, comme TLS ou HTTPS. Ainsi, le chiffrement du transport doit être traité par le protocole de transport et il ne doit pas être un facteur majeur pour déterminer le modèle de chiffrement au repos à utiliser.

## <a name="client-encryption-model"></a>Modèle de chiffrement client

Le modèle de chiffrement client fait référence au chiffrement qui est effectué en dehors du fournisseur de ressources ou d’Azure par le service ou l’application appelante. Le chiffrement peut être effectué par l’application de service dans Azure, ou par une application s’exécutant dans le centre de données du client. Dans les deux cas, quand vous tirez parti de ce modèle de chiffrement, le fournisseur de ressources Azure reçoit un objet blob chiffré des données sans possibilité de déchiffrer les données en aucune façon ou d’avoir accès aux clés de chiffrement. Dans ce modèle, la gestion des clés est effectuée par le service ou l’application appelant et est opaque pour le service Azure.

![Client](./media/encryption-models/azure-security-encryption-atrest-fig2.png)

## <a name="server-side-encryption-using-service-managed-keys"></a>Chiffrement côté serveur à l’aide de clés gérées par le service

Pour de nombreux clients, l’exigence principale est de garantir que les données sont chiffrées quand elles sont au repos. Le chiffrement côté serveur à l’aide de clés gérées par le service autorise ce modèle en permettant aux clients de marquer la ressource spécifique (compte de stockage, base de données SQL, etc.) pour le chiffrement et de laisser tous les aspects de la gestion des clés, comme l’émission, la sauvegarde et la rotation des clés, à Microsoft. La plupart des services Azure prenant en charge le chiffrement au repos prennent généralement en charge ce modèle de déchargement de la gestion des clés de chiffrement sur Azure. Le fournisseur de ressources Azure crée les clés, les place dans un stockage sécurisé et les récupère quand c’est nécessaire. Cela signifie que le service a un accès complet aux clés et qu’il a un contrôle total sur la gestion du cycle de vie des informations d’identification.

![géré](./media/encryption-models/azure-security-encryption-atrest-fig4.png)

Le chiffrement côté serveur à l’aide de clés gérées par le service répond ainsi rapidement au besoin de chiffrement au repos avec une charge de travail peu importante au niveau du client. Quand il est disponible, un client ouvre en général le portail Azure pour l’abonnement cible et le fournisseur de ressources, et il coche une case indiquant qu’il veut que les données soient chiffrées. Dans certains gestionnaires de ressources, le chiffrement côté serveur avec des clés gérées par le service est activé par défaut.

Le chiffrement côté serveur avec des clés gérées par Microsoft implique que le service dispose d’un accès complet pour le stockage et la gestion des clés. Si certains clients peuvent souhaiter gérer les clés en pensant qu’ils peuvent assurer une meilleure sécurité, le coût et le risque associés à une solution de stockage de clés personnalisée doivent être considérés lors de l’évaluation de ce modèle. Dans de nombreux cas, une organisation peut déterminer que, dans le cadre d’une solution locale, les contraintes liées aux ressources ou les risques peuvent être supérieurs aux risques liés à la gestion des clés de chiffrement au repos dans le cloud.  Cependant, il se peut que ce modèle ne soit pas adapté aux organisations qui doivent répondre à des exigences quant au contrôle de la création ou du cycle de vie des clés de chiffrement, ou encore à celles qui souhaitent que les gestionnaires des clés de chiffrement d’un service ne soient pas les personnes qui gèrent le service (séparer la gestion des clés du modèle de gestion global du service).

### <a name="key-access"></a>Accès aux clés

Quand le chiffrement côté serveur avec des clés gérées par le service est utilisé, la création des clés, leur stockage et l’accès du service à ces clés sont gérés par le service. En règle générale, les principaux fournisseurs de ressources Azure stockent les clés de chiffrement des données dans un magasin qui est proche des données, et qui est disponible et accessible rapidement, alors que les clés de chiffrement des clés sont stockées dans un magasin interne sécurisé.

**Avantages**

- Simplicité de la configuration
- Microsoft gère la rotation, la sauvegarde et la redondance des clés
- Le client ne doit pas faire face au coût associé à l’implémentation ni au risque d’un schéma de gestion des clés personnalisé.

**Inconvénients**

- Pas de contrôle du client sur les clés de chiffrement (spécification, cycle de vie, révocation des clés, etc.)
- Pas de possibilité de séparer la gestion de clés du modèle de gestion global pour le service

## <a name="server-side-encryption-using-customer-managed-keys-in-azure-key-vault"></a>Chiffrement côté serveur à l’aide de clés gérées par le client dans Azure Key Vault

Dans les scénarios où les données doivent être chiffrées au repos et les clés de chiffrement doivent être contrôlées, les clients peuvent utiliser le chiffrement côté serveur à l’aide de clés gérées par le client dans Azure Key Vault. Certains services peuvent stocker seulement la clé de chiffrement des clés racine dans Azure Key Vault et stocker la clé de chiffrement des données chiffrée à un emplacement interne plus proche des données. Dans ce scénario, les clients peuvent apporter leurs propres clés au coffre de clés (BYOK, Bring Your Own Key), ou en générer de nouvelles et les utiliser pour chiffrer les ressources souhaitées. Quand le fournisseur de ressources effectue les opérations de chiffrement et de déchiffrement, il utilise la clé de chiffrement de la clé configurée comme clé racine pour toutes les opérations de chiffrement.

La perte de clés de chiffrement de clé signifie la perte de données. Pour cette raison, les clés ne doivent pas être supprimées. Les clés doivent être sauvegardées chaque fois qu’elles sont créées ou pivotées. [La suppression réversible](../../key-vault/general/soft-delete-overview.md) doit être activée sur tout coffre stockant des clés de chiffrement de clé. Au lieu de supprimer une clé, affectez à l’activé la valeur false ou définissez la date d’expiration.

### <a name="key-access"></a>Accès aux clés

Le modèle de chiffrement côté serveur avec des clés gérées par le client dans Azure Key Vault implique que le service accède aux clés pour chiffrer et déchiffrer lorsque nécessaire. Les clés de chiffrement au repos sont rendues accessibles à un service via une stratégie de contrôle d’accès. Cette stratégie accorde à cette identité de service un accès pour recevoir la clé. Un service Azure s’exécutant pour le compte d’un abonnement associé peut être configuré avec une identité dans cet abonnement. Le service peut effectuer l’authentification Azure Active Directory et recevoir un jeton d’authentification en s’identifiant lui-même comme étant ce service agissant pour le compte de l’abonnement. Ce jeton peut ensuite être présenté au coffre de clés pour obtenir une clé à laquelle l’accès lui a été donné.

Pour les opérations utilisant des clés de chiffrement, une identité de service peut être autorisée à accéder aux opérations suivantes : déchiffrer, chiffrer, unwrapKey, wrapKey, vérifier, signer, obtenir, répertorier, mettre à jour, créer, importer, supprimer, sauvegarder et restaurer.

Pour obtenir une clé à utiliser dans le chiffrement ou le déchiffrement des données au repos, l’identité de service pour le compte duquel l’instance du service Resource Manager s’exécutera doit avoir les autorisations UnwrapKey (pour obtenir la clé de déchiffrement) et WrapKey (pour insérer une clé dans le coffre de clés lors de la création d’une nouvelle clé).

>[!NOTE]
>Pour plus d’informations sur les autorisations de Key Vault, consultez la page consacrée à la sécurisation de votre coffre de clés dans la [documentation d’Azure Key Vault](../../key-vault/general/secure-your-key-vault.md).

**Avantages**

- Contrôle total sur les clés utilisées : les clés de chiffrement sont gérées dans le coffre de clés du client, sous le contrôle du client.
- Possibilité de chiffrer de plusieurs services en un seul service maître
- Possibilité de séparer la gestion de clés du modèle de gestion global pour le service
- Possibilité de définir l’emplacement du service et des clés dans des régions différentes

**Inconvénients**

- Le client a l’entière responsabilité de la gestion de l’accès aux clés
- Le client a l’entière responsabilité de la gestion du cycle de vie des clés
- Charge de travail supplémentaire pour l’installation et la configuration

## <a name="server-side-encryption-using-customer-managed-keys-in-customer-controlled-hardware"></a>Chiffrement côté serveur à l’aide de clés gérées par le client dans du matériel contrôlé par le client

Certains services Azure activent le modèle de gestion de clés HYOK (Host Your Own Key). Ce mode de gestion est utile dans les scénarios où il est nécessaire de chiffrer les données au repos et de gérer les clés dans un dépôt propriétaire non contrôlé par Microsoft. Dans ce modèle, le service doit récupérer la clé à partir d’un site externe. Les garanties de disponibilité et de performances sont affectées et la configuration est plus complexe. En outre, comme le service n’a pas accès à la clé de chiffrement des données pendant les opérations de chiffrement et de déchiffrement, les garanties de sécurité globale de ce modèle sont similaires à celles du modèle où les clés sont gérées par le client dans Azure Key Vault.  Par conséquent, ce modèle n’est pas approprié pour la plupart des organisations, sauf si elles ont des exigences spécifiques de gestion des clés. En raison de ces limitations, la plupart des services Azure ne gèrent pas le chiffrement côté serveur à l’aide de clés gérées par le serveur dans le matériel contrôlé par le client.

### <a name="key-access"></a>Accès aux clés

Quand le chiffrement côté serveur à l’aide de clés gérées par le service dans du matériel contrôlé par le client est utilisé, les clés sont conservées sur un système configuré par le client. Les services Azure qui prennent en charge ce modèle fournissent un moyen d’établir une connexion sécurisée avec un magasin de clés fourni par le client.

**Avantages**

- Contrôle total sur la clé racine utilisée : les clés de chiffrement sont gérées par un magasin fourni par le client
- Possibilité de chiffrer de plusieurs services en un seul service maître
- Possibilité de séparer la gestion de clés du modèle de gestion global pour le service
- Possibilité de définir l’emplacement du service et des clés dans des régions différentes

**Inconvénients**

- Entière responsabilité pour le stockage, la sécurité, les performances et la disponibilité des clés
- Entière responsabilité de la gestion de l’accès aux clés
- Entière responsabilité de la gestion du cycle de vie des clés
- Coûts considérables d’installation, de configuration et de maintenance
- Dépendance accrue de la disponibilité du réseau entre le centre de données du client et les centres de données Azure.

## <a name="supporting-services"></a>Services associés
Services Azure qui prennent en charge chaque modèle de chiffrement :

| Produit, Fonctionnalité ou Service | Côté serveur à l’aide d’une clé gérée par le service   | Côté serveur à l’aide d’une clé gérée par le client | Côté client à l’aide d’une clé gérée par le client  |
|----------------------------------|--------------------|-----------------------------------------|--------------------|
| **IA et Machine Learning**      |                    |                    |                    |
| Recherche cognitive Azure           | Oui                | Oui                | -                  |
| Azure Cognitive Services         | Oui                | Oui                | -                  |
| Azure Machine Learning           | Oui                | Oui                | -                  |
| Azure Machine Learning Studio (classique) | Oui         | Préversion, RSA 2048 bits | -               |
| Content Moderator                | Oui                | Oui                | -                  |
| Face                             | Oui                | Oui                | -                  |
| Language Understanding           | Oui                | Oui                | -                  |
| Personalizer                     | Oui                | Oui                | -                  |
| QnA Maker                        | Oui                | Oui                | -                  |
| Services Speech                  | Oui                | Oui                | -                  |
| Translator Text                  | Oui                | Oui                | -                  |
| Power BI                         | Oui                | Oui, RSA 4096 bits  | -                  |
| **Analyse**                    |                    |                    |                    |
| Azure Stream Analytics           | Oui                | Oui\*\*            | -                  |
| Event Hubs                       | Oui                | Oui                | -                  |
| Fonctions                        | Oui                | Oui                | -                  |
| Azure Analysis Services          | Oui                | -                  | -                  |
| Azure Data Catalog               | Oui                | -                  | -                  |
| Azure HDInsight                  | Oui                | Tous                | -                  |
| Azure Monitor Application Insights | Oui                | Oui                | -                  |
| Azure Monitor Log Analytics      | Oui                | Oui                | -                  |
| Explorateur de données Azure              | Oui                | Oui                | -                  |
| Azure Data Factory               | Oui                | Oui                | -                  |
| Azure Data Lake Store            | Oui                | Oui, RSA 2048 bits  | -                  |
| **Containers**                   |                    |                    |                    |
| Azure Kubernetes Service         | Oui                | Oui                | -                  |
| Container Instances              | Oui                | Oui                | -                  |
| Container Registry               | Oui                | Oui                | -                  |
| **Calcul**                      |                    |                    |                    |
| Virtual Machines                 | Oui                | Oui                | -                  |
| Groupe de machines virtuelles identiques        | Oui                | Oui                | -                  |
| SAP HANA                         | Oui                | Oui                | -                  |
| App Service                      | Oui                | Oui\*\*            | -                  |
| Automatisation                       | Oui                | Oui\*\*            | -                  |
| Azure Functions                  | Oui                | Oui\*\*            | -                  |
| Portail Azure                     | Oui                | Oui\*\*            | -                  |
| Logic Apps                       | Oui                | Oui                | -                  |
| Applications gérées par Azure       | Oui                | Oui\*\*            | -                  |
| Service Bus                      | Oui                | Oui                | -                  |
| Site Recovery                    | Oui                | Oui                | -                  |
| **Bases de données**                    |                    |                    |                    |
| SQL Server sur machines virtuelles   | Oui                | Oui                | Oui                |
| Azure SQL Database               | Oui                | Oui, RSA 3072 bits  | Oui                |
| Azure SQL Database pour MariaDB   | Oui                | -                  | -                  |
| Azure SQL Database pour MySQL     | Oui                | Oui                | -                  |
| Azure SQL Database pour PostgreSQL | Oui               | Oui                | -                  |
| Azure Synapse Analytics          | Oui                | Oui, RSA 3072 bits  | -                  |
| SQL Server Stretch Database      | Oui                | Oui, RSA 3072 bits  | Oui                |
| Stockage Table                    | Oui                | Oui                | Oui                |
| Azure Cosmos DB                  | Oui                | Oui                | -                  |
| Azure Databricks                 | Oui                | Oui                | -                  |
| Azure Database Migration Service | Oui                | N/A\*              | -                  |
| **DevOps**                       |                    |                    |                    |
| Azure DevOps Services            | Oui                | -                  | Oui                |
| Azure Repos                      | Oui                | -                  | Oui                |
| **Identité**                     |                    |                    |                    |
| Azure Active Directory           | Oui                | -                  | -                  |
| Azure Active Directory Domain Services | Oui          | Oui                | -                  |
| **Intégration**                  |                    |                    |                    |
| Service Bus                      | Oui                | Oui                | Oui                |
| Event Grid                       | Oui                | -                  | -                  |
| Gestion des API                   | Oui                | -                  | -                  |
| **Services IoT**                 |                    |                    |                    |
| IoT Hub                          | Oui                | Oui                | Oui                |
| IoT Hub Device Provisioning      | Oui                | Oui                | -                  |
| **Gestion et gouvernance**    |                    |                    |                    |
| Azure Site Recovery              | Oui                | -                  | -                  |
| Azure Migrate                    | Oui                | Oui                | -                  |
| **Média**                        |                    |                    |                    |
| Media Services                   | Oui                | Oui                | Oui                |
| **Sécurité**                     |                    |                    |                    |
| Azure Security Center pour IoT    | Oui                | Oui                | -                  |
| Azure Sentinel                   | Oui                | Oui                | -                  |
| **Stockage**                      |                    |                    |                    |
| Stockage Blob                     | Oui                | Oui                | Oui                |
| Stockage Blob Premium             | Oui                | Oui                | Oui                |
| Stockage sur disque                     | Oui                | Oui                | -                  |
| Disque Ultra               | Oui                | Oui                | -                  |
| Stockage sur disque managé             | Oui                | Oui                | -                  |
| Stockage Fichier                     | Oui                | Oui                | -                  |
| Stockage Premium de fichiers             | Oui                | Oui                | -                  |
| File Sync                        | Oui                | Oui                | -                  |
| Stockage File d’attente                    | Oui                | Oui                | Oui                |
| Avere vFXT                       | Oui                | -                  | -                  |
| Cache Azure pour Redis            | Oui                | N/A\*              | -                  |
| Azure NetApp Files               | Oui                | Oui                | -                  |
| Stockage archive                  | Oui                | Oui                | -                  |
| StorSimple                       | Oui                | Oui                | Oui                |
| Sauvegarde Azure                     | Oui                | Oui                | Oui                |
| Data Box                         | Oui                | -                  | Oui                |
| Data Box Edge                    | Oui                | Oui                | -                  |

\* Ce service ne rend pas les données persistantes. Les caches temporaires éventuels sont chiffrés avec une clé Microsoft.

\*\* Ce service prend en charge le stockage des données dans votre propre coffre de clés, compte de stockage ou tout autre service de persistance des données qui prend déjà en charge le chiffrement côté serveur avec clé gérée par le client.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser le chiffrement dans Azure](encryption-overview.md).
- Découvrez comment Azure utilise le [double chiffrement](double-encryption.md) pour atténuer les menaces qui accompagnent le chiffrement des données.
