---
title: Chiffrement des données au repos d’Azure - Sécurité Azure
description: Cet article fournit une vue d’ensemble du chiffrement des données au repos d’Azure, avec ses fonctionnalités globales et des considérations générales.
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
ms.date: 08/13/2020
ms.author: mbaldwin
ms.openlocfilehash: 092320db9b7fe2b1f3fe142f84ad201d40dc6e2e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492282"
---
# <a name="azure-data-encryption-at-rest"></a>Chiffrement des données au repos d’Azure

Microsoft Azure comprend des outils pour protéger les données en fonction des besoins de sécurité et de conformité de votre entreprise. Ce document porte sur les points suivants :

- Protection des données au repos sur Microsoft Azure
- Présentation des différents composants impliqués dans la mise en œuvre de la protection des données
- Examen des avantages et des inconvénients des principales approches de protection de la gestion

Le chiffrement au repos est une exigence de sécurité courante. Dans Azure, les organisations peuvent chiffrer les données au repos, sans assumer les risques et les coûts associés à une solution de gestion des clés personnalisée. Les organisations ont la possibilité de laisser Azure gérer complètement le chiffrement au repos. En outre, les organisations ont différentes options pour gérer étroitement le chiffrement ou les clés de chiffrement.

## <a name="what-is-encryption-at-rest"></a>Qu’est-ce que le chiffrement au repos ?

Le chiffrement au repos fait référence à l’encodage (chiffrement) des données quand elles sont stockées. La conception du chiffrement au repos dans Azure utilise le chiffrement symétrique pour chiffrer et déchiffrer rapidement de grandes quantités de données selon un modèle conceptuel simple :

- Une clé de chiffrement symétrique est utilisée pour chiffrer les données au fil de leur stockage.
- La clé de chiffrement est utilisée pour déchiffrer ces données au fil de leur préparation à une utilisation en mémoire.
- Les données peuvent être partitionnées, et des clés différentes peuvent être utilisées pour chaque partition.
- Les clés doivent être stockées dans un emplacement sécurisé doté du contrôle d’accès basé sur l’identité et de stratégies d’audit. Les clés de chiffrement de données sont souvent chiffrées avec une clé de chiffrement de clé dans Azure Key Vault pour limiter davantage l’accès.

Dans la pratique, les scénarios de gestion et de contrôle des clés, ainsi que les garanties de scalabilité et de disponibilité, nécessitent des mécanismes supplémentaires. Les concepts et les composants du chiffrement des données au repos de Microsoft Azure sont décrits ci-dessous.

## <a name="the-purpose-of-encryption-at-rest"></a>Objectif du chiffrement au repos

Le chiffrement au repos offre une protection des données pour les données stockées (au repos). Les attaques contre les données au repos sont notamment des tentatives d’obtenir un accès physique au matériel sur lequel les données sont stockées puis de compromettre les données qui y sont contenues. Dans une telle attaque, le disque dur d’un serveur peut avoir fait l’objet d’une mauvaise manipulation lors de la maintenance, permettant à un attaquant de retirer le disque dur. Plus tard, cet attaquant peut placer le disque dur dans un ordinateur qu’il contrôle pour tenter d’accéder aux données.

Le chiffrement au repos est conçu pour empêcher l’attaquant d’accéder aux données non chiffrées en garantissant que les données sont chiffrées quand elles sont sur le disque. Si un attaquant récupère un disque dur comprenant des données chiffrées, mais qu’il ne dispose pas des clés de chiffrement, il doit résoudre le chiffrement pour lire les données. Ce type d’attaque est beaucoup plus complexe et laborieux comparé aux attaques de données non chiffrées sur un disque dur. Pour cette raison, le chiffrement au repos est fortement recommandé et constitue une exigence de haute priorité pour de nombreuses organisations.

Le chiffrement au repos peut également être nécessaire pour les besoins de l’organisation en matière de gouvernance et de conformité des données. Les réglementations publiques et de l’industrie, comme HIPAA, PCI et FedRAMP définissent des protections spécifiques quant aux exigences de protection et de chiffrement des données. Le chiffrement au repos est une mesure obligatoire nécessaire à la conformité avec certaines de ces réglementations. Pour plus d’informations sur l’approche de Microsoft en matière de validation FIPS 140-2, consultez la [publication FIPS (Federal Information Processing Standard) 140-2](/microsoft-365/compliance/offering-fips-140-2).

Non seulement le chiffrement au repos répond aux exigences de conformité et aux obligations réglementaires, mais il fournit une défense en profondeur. Microsoft Azure fournit une plateforme conforme destinée aux services, aux applications et aux données. Il fournit également une sécurité complète des équipements et des éléments physiques, un contrôle d’accès aux données et des fonctionnalités d’audit. Toutefois, il est important de mettre en œuvre des mesures de sécurité « superposées » supplémentaires en cas de défaillance de l’une des mesures de sécurité principales. Le chiffrement au repos propose une telle mesure de sécurité.

Microsoft s’engage à fournir des options de chiffrement au repos sur les services cloud et à donner aux clients le contrôle des clés de chiffrement et la journalisation de l’utilisation des clés. En outre, Microsoft s’emploie à mettre en œuvre par défaut le chiffrement au repos de toutes les données des clients.

## <a name="azure-encryption-at-rest-components"></a>Composants du chiffrement au repos d’Azure

Comme décrit précédemment, l’objectif du chiffrement au repos est que les données stockées sur disque soient chiffrées avec une clé de chiffrement secrète. Pour atteindre cet objectif de clés sécurisées, il est nécessaire de disposer d’un système permettant la création, le stockage, le contrôle d’accès et la gestion des clés de chiffrement. Bien que des détails puissent varier, les implémentations du chiffrement au repos des services Azure peuvent être décrites selon les termes illustrés dans le diagramme suivant.

![Components](./media/encryption-atrest/azure-security-encryption-atrest-fig1.png)

### <a name="azure-key-vault"></a>Azure Key Vault

L’emplacement de stockage des clés de chiffrement et le contrôle d’accès à ces clés sont fondamentaux pour un modèle de chiffrement au repos. Les clés doivent être hautement sécurisées, mais gérables par des utilisateurs spécifiés et disponibles pour des services spécifiques. Pour les services Azure, Azure Key Vault est la solution de stockage des clés recommandée et offre une expérience de gestion commune entre les services. Les clés sont stockées et gérées dans des coffres de clés, et l’accès à un coffre de clés peut être donné à des utilisateurs ou à des services. Azure Key Vault prend en charge la création de clés par les clients ou l’importation de clés des clients pour les utiliser dans des scénarios où les clés de chiffrement sont gérées par ces clients.

### <a name="azure-active-directory"></a>Azure Active Directory

Les autorisations d’utiliser les clés stockées dans Azure Key Vault, pour les gérer ou pour y accéder pour les chiffrer et les déchiffrer dans le cadre du chiffrement au repos, peuvent être données à des comptes Azure Active Directory.

### <a name="key-hierarchy"></a>Hiérarchie des clés

Plusieurs clés de chiffrement sont utilisées dans une implémentation du chiffrement au repos. Le stockage d’une clé de chiffrement dans Azure Key Vault garantit l’accès sécurisé aux clés et la gestion centralisée des clés. Toutefois, l’accès local du service aux clés de chiffrement est plus efficace pour le chiffrement et le déchiffrement en bloc que d’interagir avec Key Vault pour chaque opération de données, ce qui permet un chiffrement renforcé et de meilleures performances. Limiter l’utilisation d’une seule clé de chiffrement réduit le risque que la clé soit compromise et le coût du rechiffrement quand une clé doit être remplacée. Les modèles de chiffrement Azure au repos utilisent une hiérarchie de clé constituée des types de clés suivants afin de répondre à tous ces besoins :

- **Clé de chiffrement des données** : une clé symétrique AES256 utilisée pour chiffrer une partition ou un bloc de données.  Une même ressource peut avoir plusieurs partitions et de nombreuses clés de chiffrement des données. Le chiffrement de chaque bloc de données avec une clé différente rend les attaques d’analyse du chiffrement plus difficiles. L’accès aux clés de chiffrement des données est nécessaire au fournisseur de ressources ou à l’instance d’application qui chiffre et déchiffre un bloc spécifique. Quand une clé de chiffrement des données est remplacée par une nouvelle clé, seules les données du bloc qui y est associé doivent être rechiffrées avec la nouvelle clé.
- **Clé de chiffrement des clés** : une clé de chiffrement utilisée pour chiffrer les clés de chiffrement des données. L’utilisation d’une clé de chiffrement des clés ne quittant jamais le coffre de clés permet le chiffrement et le contrôle des clés de chiffrement des données elles-mêmes. L’entité qui a accès à la clé de chiffrement des clés peut être différente de l’entité qui a besoin de la clé de chiffrement des données. Une entité peut répartir l’accès à la clé de chiffrement des clés pour limiter l’accès de chaque clé de chiffrement des clés vers une partition spécifique. Comme la clé de chiffrement des clés est nécessaire pour déchiffrer les clés de chiffrement des données, la clé de chiffrement des clés est dès lors le point unique par lequel les clés de chiffrement des données peuvent être supprimées en supprimant la clé de chiffrement des clés.

Les clés de chiffrement des données chiffrées avec des clés de chiffrement des clés sont stockées séparément, et seule une entité ayant accès à la clé de chiffrement des clés peut déchiffrer ces clés de chiffrement des données. Différents modèles de stockage des clés sont pris en charge. Pour plus d’informations, consultez les [modèles de chiffrement de données](encryption-models.md).

## <a name="encryption-at-rest-in-microsoft-cloud-services"></a>Chiffrement au repos dans les services cloud Microsoft

Les services Microsoft Cloud sont utilisés dans les trois modèles de cloud : IaaS, PaaS, SaaS. Voici des exemples de la façon dont ils s’adaptent sur chaque modèle :

- Services logiciels, appelés SaaS ou Software as a Server, qui ont des applications fournies par le cloud, comme Microsoft 365.
- Services de plateforme, dont les clients tirent parti sur le cloud dans leurs applications, en utilisant le cloud pour des fonctionnalités comme le stockage, l’analytique et Service Bus.
- Services d’infrastructure, appelés IaaS ou Infrastructure as a Service, dans lesquels le client déploie des systèmes d’exploitation et des applications qui sont hébergés dans le cloud, et tirant éventuellement parti d’autres services cloud.

### <a name="encryption-at-rest-for-saas-customers"></a>Chiffrement au repos pour les clients SaaS

Les clients SaaS (Software as a Service) ont généralement le chiffrement au repos activé ou disponible dans chaque service. Microsoft 365 propose plusieurs options permettant aux clients de vérifier ou d’activer le chiffrement au repos. Pour plus d’informations sur les services Microsoft 365, consultez l’article dédié au [chiffrement dans Microsoft 365](/microsoft-365/compliance/encryption).

### <a name="encryption-at-rest-for-paas-customers"></a>Chiffrement au repos pour les clients PaaS

Les données des clients PaaS (Platform as a Service) résident généralement dans un service de stockage tel que le Stockage Blob, mais peuvent également être mises en cache ou stockées dans l’environnement d’exécution des applications, par exemple, une machine virtuelle. Pour connaître les options de chiffrement au repos disponibles, consultez le tableau ci-dessous pour les plateformes de stockage et d’applications que vous utilisez.

### <a name="encryption-at-rest-for-iaas-customers"></a>Chiffrement au repos pour les clients IaaS

Les clients IaaS (Infrastructure as a Service) peuvent utiliser différents services et applications. Les services IaaS peuvent activer le chiffrement au repos dans leurs machines virtuelles et leurs disques durs virtuels hébergés dans Azure en utilisant Azure Disk Encryption.

#### <a name="encrypted-storage"></a>Stockage chiffré

Comme pour PaaS, les solutions IaaS peuvent tirer parti d’autres services Azure qui stockent les données chiffrées au repos. Dans ce cas, vous pouvez activer la prise en charge du chiffrement au repos telle qu’elle est fournie par chaque service Azure utilisé. Le tableau ci-dessous énumère les principales plateformes de stockage, de services et d’applications, avec le modèle de chiffrement au repos pris en charge.

#### <a name="encrypted-compute"></a>Calcul chiffré

Tous les disques managés, instantanés et images sont chiffrés à l’aide de Storage Service Encryption au moyen d’une clé gérée par le service. Une solution plus complète de chiffrement au repos fait en sorte que les données ne soient jamais stockées sous une forme non chiffrée. Lors de leur traitement sur une machine virtuelle, les données peuvent être conservées dans le fichier d’échange Windows ou Linux, un vidage sur plantage ou le journal des applications. Pour garantir que ces données sont chiffrées au repos, les applications IaaS peuvent utiliser Azure Disk Encryption sur une machine virtuelle (Windows ou Linux) et sur un disque virtuel IaaS Azure.

#### <a name="custom-encryption-at-rest"></a>Chiffrement au repos personnalisé

Il est recommandé que, chaque fois que c’est possible, les applications IaaS tirent parti des options d’Azure Disk Encryption et du chiffrement au repos fournies par les services Azure utilisés. Dans certains cas, comme en présence d’exigences de chiffrement irrégulières ou d’un stockage qui n’est pas basé sur Azure, le développeur d’une application IaaS peut être amené à implémenter lui-même le chiffrement au repos. Les développeurs de solutions IaaS peuvent mieux s’intégrer à la gestion d’Azure et répondre aux attentes des clients en tirant parti de certains composants Azure. En particulier, les développeurs doivent utiliser le service Azure Key Vault pour fournir un stockage des clés sécurisé, ainsi qu’offrir à leurs clients des options de gestion des clés cohérentes avec celles de la plupart des services de plateforme Azure. En outre, les solutions personnalisées doivent utiliser des identités de service gérées par Azure pour permettre aux comptes de service d’accéder aux clés de chiffrement. Pour plus d’informations sur Azure Key Vault et les identités de service gérées, les développeurs peuvent consulter leurs SDK respectifs.

## <a name="azure-resource-providers-encryption-model-support"></a>Prise en charge du modèle de chiffrement des fournisseurs de ressources Azure

Les services Microsoft Azure prennent chacun en charge un ou plusieurs modèles de chiffrement au repos. Cependant, pour certains services, un ou plusieurs des modèles de chiffrement peuvent ne pas être applicables. Pour les services qui prennent en charge les scénarios de clé gérés par le client, ils peuvent prendre en charge uniquement un sous-ensemble des types de clés pris en charge par Azure Key Vault pour les clés de chiffrement à clé. En outre, les services peuvent prendre en charge ces scénarios et les types de clé selon des planifications différentes. Cette section décrit la prise en charge du chiffrement au repos au moment de la rédaction de ce document pour chacun des principaux services de stockage de données Azure.

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

Tout client utilisant les fonctionnalités IaaS d’Azure peut effectuer le chiffrement au repos pour ses machines virtuelles et ses disques IaaS via Azure Disk Encryption. Pour plus d’informations sur Azure Disk Encryption, consultez la [documentation d’Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md).

#### <a name="azure-storage"></a>Stockage Azure

Tous les services Stockage Azure (Stockage Blob, Stockage File d’attente, Stockage Table et Azure Files) prennent en charge le chiffrement au repos côté serveur ; certains services prennent également en charge les clés gérées par le client et le chiffrement côté client.

- Côté serveur : Tous les services Stockage Azure permettent par défaut le chiffrement côté serveur avec des clés gérées par le service, une opération transparente pour l’application. Pour plus d’informations, consultez [Azure Storage Service Encryption pour les données au repos](../../storage/common/storage-service-encryption.md). Azure Stockage Blob Azure et Azure Files prennent également en charge les clés RSA 2048 bits gérées par le client dans Azure Key Vault. Pour plus d’informations, consultez [Chiffrement du service de stockage à l’aide de clés gérées par le client dans Azure Key Vault](../../storage/common/customer-managed-keys-configure-key-vault.md).
- Côté client : Les objets blob, les tables et les files d’attente Azure prennent en charge le chiffrement côté client. Lors de l’utilisation du chiffrement côté client, les clients chiffrent les données et les chargent sous la forme d’un objet blob chiffré. La gestion des clés est effectuée par le client. Pour plus d’informations, consultez [Chiffrement côté client et Azure Key Vault pour le stockage Microsoft Azure](../../storage/common/storage-client-side-encryption.md).

#### <a name="azure-sql-database"></a>Azure SQL Database

Azure SQL Database prend actuellement en charge le chiffrement au repos pour les scénarios de chiffrement côté service géré par Microsoft et côté client.

La prise en charge du chiffrement côté serveur est actuellement fournie par la fonctionnalité SQL nommée Transparent Data Encryption. Une fois qu’un client Azure SQL Database active Transparent Data Encryption, les clés sont créées et gérées automatiquement pour lui. Le chiffrement au repos peut être activé au niveau de la base de données et au niveau du serveur. À compter de juin 2017, [Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) est activé par défaut sur les bases de données nouvellement créées. Azure SQL Database prend également en charge les clés RSA 2048 bits gérées par le client dans Azure Key Vault. Pour plus d’informations, voir [Transparent Data Encryption avec prise en charge de BYOK pour Azure SQL Database et Data Warehouse](/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql).

Le chiffrement côté client des données Azure SQL Database est pris en charge via la fonctionnalité [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine). Always Encrypted utilise une clé qui est créée et stockée par le client. Les clients peuvent stocker la clé principale dans un magasin de certificats Windows, dans Azure Key Vault ou dans un module de sécurité matériel local. Avec SQL Server Management Studio, les utilisateurs SQL choisissent quelle clé ils veulent utiliser pour quelle colonne.

## <a name="conclusion"></a>Conclusion

La protection des données des clients stockées au sein des services Azure est d’une importance capitale pour Microsoft. Tous les services hébergés par Azure doivent à terme fournir des options de chiffrement au repos. Les services Azure prennent en charge les clés managées par le service, les clés gérées par le client ou le chiffrement côté client. Les services Azure étendent considérablement la disponibilité du chiffrement au repos, et la disponibilité en préversion puis générale de nouvelles options est planifiée dans les mois à venir.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur les clés gérées par le service et les clés gérées par le client, consultez les [modèles de chiffrement de données](encryption-models.md).
- Découvrez comment Azure utilise le [double chiffrement](double-encryption.md) pour atténuer les menaces qui accompagnent le chiffrement des données.
- Découvrez ce que Microsoft fait pour garantir [l’intégrité et la sécurité de la plateforme](platform.md) des hôtes parcourant les pipelines du matériel et des microprogrammes, des intégrations, des opérationnalisations et des réparations.
