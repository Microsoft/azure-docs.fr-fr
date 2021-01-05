---
title: Vue d’ensemble du chiffrement Azure | Documents Microsoft
description: Découvrez les options de chiffrement dans Azure. Consultez les informations sur le chiffrement au repos, le chiffrement en déplacement et la gestion des clés avec Azure Key Vault.
services: security
author: msmbaldwin
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: ff023ad98c7ffa269223b5d0b4a1cecc5fde1feb
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410213"
---
# <a name="azure-encryption-overview"></a>Vue d’ensemble du chiffrement Azure

Cet article fournit une vue d’ensemble de l’utilisation du chiffrement dans Microsoft Azure Document. Il couvre les principales zones de chiffrement, notamment le chiffrement au repos, le chiffrement en vol et la gestion des clés avec Azure Key Vault. Chaque section inclut des liens vers des informations plus détaillées.

## <a name="encryption-of-data-at-rest"></a>Chiffrement des données au repos

Les données au repos incluent des informations qui se trouvent dans un stockage persistant sur un support physique, sous n’importe quel format numérique. Il peut s’agir de fichiers sur un support magnétique ou optique, de données archivées et de sauvegardes de données. Microsoft Azure offre une variété de solutions de stockage de données en fonction des besoins, y compris le stockage sur fichier, disque, objet blob et table. Microsoft fournit également le chiffrement pour protéger [Azure SQL Database](../../azure-sql/database/sql-database-paas-overview.md), [Azure Cosmos DB](../../data-factory/introduction.md) et Azure Data Lake.

Le chiffrement des données au repos est disponible pour les modèles de cloud SaaS (Software-as-a-Service), PaaS (Platform-as-a-Service) et IaaS (Infrastructure-as-a-Service). Cet article résume et fournit des ressources pour vous aider à utiliser les options de chiffrement Azure.

Pour accéder à une discussion plus détaillée du mode de chiffrement de données au repos dans Azure, consultez [Chiffrement au repos des données Azure](encryption-atrest.md).

## <a name="azure-encryption-models"></a>Modèles de chiffrement Azure

Azure prend en charge plusieurs modèles de chiffrement, notamment le chiffrement côté serveur à l’aide de clés gérées par le service, de clés gérées par le client dans Key Vault, ou de clés gérées par le client sur du matériel contrôlé par le client. Avec le chiffrement côté client, vous pouvez gérer et stocker des clés localement ou dans un autre emplacement sûr.

### <a name="client-side-encryption"></a>chiffrement côté client

Le chiffrement côté client est effectué en dehors d’Azure. Il inclut :

- Les données chiffrées par une application qui s’exécute dans le centre de données du client ou par une application de service.
- Les données sont déjà chiffrées lorsqu’elles sont reçues par Azure.

Avec le chiffrement côté client, les fournisseurs de services cloud n’ont pas accès aux clés de chiffrement et ne peuvent pas déchiffrer ces données. Vous conservez un contrôle total des clés.

### <a name="server-side-encryption"></a>Chiffrement côté serveur

Les trois modèles de chiffrement côté serveur offrent différentes caractéristiques de gestion de clés, que vous pouvez choisir en fonction de vos besoins :

- **Clés gérées par le service**  : ce modèle fournit une combinaison de contrôle et de fonctionnalités avec une faible surcharge.

- **Clés gérées par le client**  : ce modèle vous permet de contrôler les clés, avec notamment la prise en charge de BYOK (Bring Your Own Keys), ou d’en générer de nouvelles.

- **Clés gérées par le service sur le matériel contrôlé par le client**  : ce modèle vous permet de gérer les clés dans votre référentiel propriétaire, en dehors du contrôle de Microsoft. Cette caractéristique est appelée HYOK (Host Your Own Key). Toutefois, la configuration est complexe et la plupart des services Azure ne prennent pas en charge ce modèle.

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

Vous pouvez protéger les machines virtuelles Windows et Linux à l’aide de [Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md), qui utilise la technologie [Windows BitLocker](/previous-versions/windows/it-pro/windows-vista/cc766295(v=ws.10)) et Linux [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) pour protéger des disques du système d’exploitation et des disques de données avec un chiffrement de volume complet.

Les clés de chiffrement et les secrets sont sauvegardés dans votre abonnement [Azure Key Vault](../../key-vault/general/overview.md). À l’aide du service Sauvegarde Azure, vous pouvez sauvegarder et restaurer des machines virtuelles chiffrées qui utilisent la configuration de clé de chiffrement à clé (KEK).

### <a name="azure-storage-service-encryption"></a>Chiffrement du service de stockage Azure

Les données au repos dans le stockage Blob Azure et les partages de fichiers Azure peuvent être chiffrées dans les scénarios côté serveur et côté client.

[Azure Storage Service Encryption](../../storage/common/storage-service-encryption.md) (SSE) peut chiffrer automatiquement les données avant qu’elles ne soient stockées, et les déchiffre automatiquement quand vous les récupérez. Le processus est entièrement transparent pour les utilisateurs. Storage Service Encryption utilise le [chiffrement AES (Advanced Encryption Standard)](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256 bits, qui est l’un des chiffrements par blocs les plus forts disponibles. AES gère le chiffrement, le déchiffrement et la gestion des clés en toute transparence.

### <a name="client-side-encryption-of-azure-blobs"></a>Chiffrement côté client des objets blob Azure

Vous pouvez effectuer le chiffrement côté client des objets blob Azure de différentes manières.

Vous pouvez utiliser la bibliothèque cliente de stockage Azure pour le package NuGet .NET afin de chiffrer les données dans vos applications clientes avant de les charger vers votre stockage Azure.

Pour en savoir plus et télécharger la bibliothèque cliente de stockage Azure pour le package NuGet .NET, consultez [Stockage Microsoft Azure 8.3.0](https://www.nuget.org/packages/WindowsAzure.Storage).

Quand vous utilisez le chiffrement côté client avec Key Vault, vos données sont chiffrées à l’aide d’une clé de chiffrement de contenu (CEK) symétrique à usage unique qui est générée par le SDK client de stockage Azure. La clé CEK est chiffrée à l’aide d’une clé de chiffrement de clé (KEK), qui peut être une clé symétrique ou une paire de clés asymétriques. Vous pouvez la gérer localement ou la stocker dans Key Vault. Les données chiffrées sont ensuite chargées vers Stockage Azure.

Pour en savoir plus sur le chiffrement côté client avec Key Vault et obtenir des instructions de démarrage pas à pas, consultez [Didacticiel : Chiffrement et déchiffrement d’objets blob dans Microsoft Azure Storage à l’aide d’Azure Key Vault](../../storage/blobs/storage-encrypt-decrypt-blobs-key-vault.md).

Pour finir, vous pouvez également utiliser la bibliothèque cliente de stockage Azure pour Java afin d’effectuer un chiffrement côté client avant de charger des données vers le stockage Azure et de déchiffrer les données lors de leur téléchargement vers le client. La bibliothèque prend également en charge l’intégration à [Key Vault](https://azure.microsoft.com/services/key-vault/) pour la gestion des clés de compte de stockage.

### <a name="encryption-of-data-at-rest-with-azure-sql-database"></a>Chiffrement des données au repos avec Azure SQL Database

[Azure SQL Database](../../azure-sql/database/sql-database-paas-overview.md) est une base de données relationnelle à usage général dans Azure qui prend en charge des structures telles que les données relationnelles, JSON, les données spatiales et XML. SQL Database prend en charge le chiffrement côté serveur grâce à la fonctionnalité de chiffrement transparent des données (TDE) et le chiffrement côté client grâce à la fonction Always Encrypted.

#### <a name="transparent-data-encryption"></a>chiffrement transparent des données

[TDE](/sql/relational-databases/security/encryption/transparent-data-encryption-tde) est utilisé pour chiffrer les fichiers de données [SQL Server](https://www.microsoft.com/sql-server/sql-server-2016), [Azure SQL Database](../../azure-sql/database/sql-database-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) en temps réel à l’aide d’une clé de chiffrement de base de données (DEK) stockée dans l’enregistrement de démarrage de la base de données pour être disponible lors de la récupération.

TDE protège les données et les fichiers journaux, à l’aide d’algorithmes de chiffrement AES et 3DES (Triple Data Encryption Standard). Le chiffrement du fichier de base de données est effectué au niveau de la page. Les pages dans une base de données chiffrée sont chiffrées avant d’être écrites sur le disque, et sont déchiffrées quand elles sont lues en mémoire. TDE est désormais activé par défaut sur les nouvelles bases de données Azure SQL.

#### <a name="always-encrypted-feature"></a>Fonctionnalité Always Encrypted

Avec la fonctionnalité [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) dans Azure SQL, vous pouvez chiffrer les données dans les applications clientes avant de les stocker dans Azure SQL Database. Vous pouvez également activer la délégation de l’administration locale de base de données à des tiers, et maintenir la séparation entre ceux qui possèdent et peuvent afficher les données et ceux qui les gèrent, mais qui ne doivent pas pouvoir y accéder.

#### <a name="cell-level-or-column-level-encryption"></a>Chiffrement au niveau des cellules ou au niveau des colonnes

Avec Azure SQL Database, vous pouvez appliquer un chiffrement symétrique à une colonne de données à l’aide de Transact-SQL. Cette approche porte le nom de [chiffrement au niveau des cellules ou chiffrement au niveau des colonnes](/sql/relational-databases/security/encryption/encrypt-a-column-of-data) (CLE), car vous pouvez l’utiliser pour chiffrer les colonnes ou même des cellules de données spécifiques avec différentes clés de chiffrement. Cela vous offre la possibilité d’un chiffrement plus granulaire que le chiffrement transparent des données, qui chiffre les données dans les pages.

Le CLE dispose de fonctions intégrées que vous pouvez utiliser pour chiffrer les données à l’aide de clés symétriques ou asymétriques, de la clé publique d’un certificat ou d’une phrase secrète à l’aide de 3DES.

### <a name="cosmos-db-database-encryption"></a>Chiffrement de base de données Azure Cosmos DB

[Azure Cosmos DB](../../cosmos-db/database-encryption-at-rest.md) est une base de données multi-modèles distribuée par Microsoft au niveau mondial. Les données utilisateur stockées dans le stockage non volatile (disques SSD) Cosmos DB sont chiffrées par défaut. Il n’existe aucun contrôle pour activer ou désactiver le chiffrement. Le chiffrement au repos est implémenté à l’aide d’un certain nombre de technologies de sécurité, notamment des systèmes de stockage de clés sécurisés, des réseaux chiffrés et des API de chiffrement. Les clés de chiffrement sont gérées par Microsoft et font l’objet d’une rotation par le biais de directives internes de Microsoft.

### <a name="at-rest-encryption-in-data-lake"></a>Chiffrement au repos dans Data Lake

[Azure Data Lake](../../data-lake-store/data-lake-store-encryption.md) est un référentiel de l’entreprise pour tous les types de données collectées dans un seul emplacement avant toute définition formelle de spécifications ou schémas. Data Lake Store prend en charge « par défaut » un chiffrement transparent de données au repos défini lors de la création de votre compte. Par défaut, Azure Data Lake Store gère les clés pour vous, mais vous avez la possibilité de les gérer vous-même.

Trois types de clés sont utilisées dans le chiffrement et le déchiffrement des données : la clé de chiffrement principale (MEK), la clé de chiffrement de données (DEK) et la clé de chiffrement de blocs (BEK). La clé MEK permet de chiffrer la clé DEK, stockée sur un support persistant et a clé BEK est dérivée de la clé DEK et du bloc de données. Si vous gérez vos propres clés, vous pouvez procéder à la rotation de la clé MEK.

## <a name="encryption-of-data-in-transit"></a>Chiffrement des données en transit

Azure offre plusieurs mécanismes pour protéger la confidentialité des données lorsqu’elles transitent d’un emplacement à un autre.

### <a name="data-link-layer-encryption-in-azure"></a>Chiffrement de la couche de liaison de données dans Azure

Chaque fois que le trafic du client Azure s'effectue entre différents centres de données - en dehors des limites physiques non contrôlées par Microsoft (ou pour le compte de Microsoft) - une méthode de chiffrement de la couche de liaison de données utilisant les [normes de sécurité MAC IEEE 802.1AE](https://1.ieee802.org/security/802-1ae/) (également appelées MACsec) est appliquée de point à point sur le matériel réseau sous-jacent. Les paquets sont chiffrés et déchiffrés sur les appareils avant d'être envoyés, ce qui permet d'éviter les attaques physiques de l'intercepteur ou les attaques par snooping/écoutes téléphoniques. Étant donné que cette technologie est intégrée au matériel réseau, elle fournit un chiffrement de débit de ligne sur le matériel réseau sans augmentation mesurable de la latence de la liaison. Ce chiffrement MACsec est activé par défaut pour tout le trafic Azure au sein d’une région ou entre des régions, et aucune intervention des clients n’est nécessaire pour l’activer. 

### <a name="tls-encryption-in-azure"></a>Chiffrement TLS dans Azure

Microsoft permet aux clients d'utiliser le protocole [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS) pour protéger les données lorsqu'elles circulent entre les services cloud et les clients. Les centres de données Microsoft négocient une connexion TLS avec les systèmes clients qui se connectent aux services Azure. TLS fournit une authentification forte, la confidentialité et l’intégrité des messages (activation de la détection de falsification et d’interception des messages), l’interopérabilité, la flexibilité des algorithmes, ainsi que la facilité de déploiement et d’utilisation.

[Perfect Forward Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy) (PFS) protège les connexions entre les systèmes clients des clients et les services cloud de Microsoft par des clés uniques. Les connexions utilisent également les longueurs de clés de chiffrement RSA de 2 048 bits. Cette combinaison rend difficile pour une personne l’interception et l’accès aux données en transit.

### <a name="azure-storage-transactions"></a>Transactions de stockage Azure

Si vous interagissez avec le stockage Azure via le portail Azure, toutes les transactions se produisent via HTTPS. L’API de stockage REST par le biais de HTTPS peut également être utilisée pour interagir avec le stockage Azure. Vous pouvez appliquer l’utilisation du protocole HTTPS quand vous appelez les API REST pour accéder aux objets dans les comptes de stockage en activant le transfert sécurisé requis pour le compte de stockage.

Les signatures d’accès partagé ([SAP](../../storage/common/storage-sas-overview.md)), qui peuvent être utilisées pour déléguer l’accès aux objets de stockage Azure, incluent une option pour spécifier que seul le protocole HTTPS est autorisé quand vous utilisez des signatures d’accès partagé. Cette approche garantit que toute personne envoyant des liens avec des jetons SAP utilise le protocole approprié.

[SMB 3.0](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn551363(v=ws.11)#BKMK_SMBEncryption), qui est utilisé pour accéder à Azure File Shares, prend en charge le chiffrement et est disponible dans Windows Server 2012 R2, Windows 8, Windows 8.1 et Windows 10. Cela rend possible l’accès entre les régions et même l’accès sur le bureau.

Le chiffrement côté client chiffre les données avant qu’elles soient envoyées à votre instance Stockage Azure, afin qu’elles soient chiffrées quand elles transitent sur le réseau.

### <a name="smb-encryption-over-azure-virtual-networks"></a>Chiffrement SMB sur les réseaux virtuels Azure 

En utilisant [SMB 3.0](https://support.microsoft.com/help/2709568/new-smb-3-0-features-in-the-windows-server-2012-file-server) sur des machines virtuelles qui exécutent Windows Server 2012 ou version ultérieure, vous pouvez sécuriser les transferts de données en chiffrant les données en transit sur des réseaux virtuels Azure. Le chiffrement des données offre une protection contre la falsification et les attaques d’écoute. Les administrateurs peuvent activer le chiffrement SMB pour l’ensemble du serveur ou juste des partages spécifiques.

Par défaut, une fois le chiffrement SMB activé pour un partage ou un serveur, seuls les clients SMB 3.0 sont autorisés à accéder aux partages chiffrés.

## <a name="in-transit-encryption-in-vms"></a>Chiffrement en transit sur des machines virtuelles

Les données en transit vers, à partir de et entre les machines virtuelles exécutant Windows peuvent être chiffrées de différentes manières, en fonction de la nature de la connexion.

### <a name="rdp-sessions"></a>Sessions RDP

Vous pouvez vous connecter et ouvrir une session sur une machine virtuelle à l’aide du [protocole RDP (Remote Desktop Protocol)](/windows/win32/termserv/remote-desktop-protocol) à partir d’un ordinateur client Windows ou d’un Mac avec un client RDP installé. Les données en transit sur le réseau dans les sessions RDP peuvent être protégées par le TLS.

Vous pouvez également utiliser le Bureau à distance pour vous connecter à une machine virtuelle Linux dans Azure.

### <a name="secure-access-to-linux-vms-with-ssh"></a>Sécuriser l’accès aux machines virtuelles Linux avec SSH

Pour la gestion à distance, vous pouvez utiliser [Secure Shell](../../virtual-machines/linux/ssh-from-windows.md) (SSH) afin de vous connecter aux machines virtuelles Linux exécutées dans Azure. SSH est un protocole de connexion chiffré qui permet d’ouvrir des sessions en toute sécurité à travers des connexions non sécurisées. Il s’agit du protocole de connexion par défaut pour les machines virtuelles Linux hébergées dans Azure. En faisant appel à des clés SSH pour l’authentification, vous éliminez le besoin de mots de passe pour vous connecter. SSH utilise une paire de clés publique/privée (chiffrement asymétrique) pour l’authentification.

## <a name="azure-vpn-encryption"></a>Chiffrement VPN Azure

Vous pouvez vous connecter à Azure via un réseau privé virtuel qui crée un tunnel sécurisé pour protéger la confidentialité des données envoyées sur le réseau.

### <a name="azure-vpn-gateways"></a>Passerelles VPN Azure

Vous pouvez utiliser la [passerelle VPN Azure](../../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md) pour envoyer un trafic chiffré entre votre réseau virtuel et votre emplacement local sur une connexion publique, ou pour envoyer un trafic entre des réseaux virtuels.

Les VPN de site à site utilisent [IPsec](https://en.wikipedia.org/wiki/IPsec) pour le chiffrement du transport. Les passerelles VPN Azure utilisent un ensemble de propositions par défaut. Vous pouvez configurer des passerelles VPN Azure pour utiliser une stratégie IPsec/IKE personnalisée avec des algorithmes de chiffrement spécifiques et des avantages clés, plutôt que des ensembles de stratégies Azure par défaut.

### <a name="point-to-site-vpns"></a>VPN point à site

Les VPN point à site permettent à des ordinateurs clients d’accéder à un réseau virtuel Azure. Le [protocole SSTP (Secure Socket Tunneling Protocol)](/previous-versions/technet-magazine/cc162322(v=msdn.10)) est utilisé pour créer le tunnel VPN. Il peut traverser des pare-feu (le tunnel apparaît en tant que connexion HTTPS). Vous pouvez utiliser votre propre autorité de certification racine interne d’infrastructure à clé publique pour la connectivité point à site.

Vous pouvez configurer une connexion VPN point à site à un réseau virtuel à l’aide du portail Azure avec l’authentification par certificat ou PowerShell.

Pour en savoir plus sur les connexions VPN de point à site à des réseaux virtuels Azure, consultez :

[Configurer une connexion point à site sur un réseau virtuel à l'aide d'une authentification par certification : Portail Azure](../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md) 

[Configurer une connexion point à site sur un réseau virtuel à l'aide d'une authentification par certificat : PowerShell](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

### <a name="site-to-site-vpns"></a>VPN site à site 

Vous pouvez utiliser une connexion de passerelle VPN de site à site pour connecter votre réseau local à un réseau virtuel Azure par le biais d’un tunnel VPN IPsec/IKE (IKEv1 ou IKEv2). Ce type de connexion nécessite un périphérique VPN local disposant d’une adresse IP publique exposée en externe.

Vous pouvez configurer une connexion VPN de site à site à un réseau virtuel à l’aide du portail Azure, de PowerShell ou d’Azure CLI.

Pour plus d'informations, consultez les pages suivantes :

[Créer une connexion de site à site dans le portail Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)

[Créer une connexion de site à site à l’aide de PowerShell](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)

[Créer un réseau virtuel avec une connexion VPN de site à site à l’aide de l’interface de ligne de commande](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)

## <a name="in-transit-encryption-in-data-lake"></a>Chiffrement en transit dans Data Lake

Les données en transit (ou données en mouvement) sont également toujours chiffrées dans Data Lake Store. Outre le chiffrement des données avant leur stockage sur un support permanent, les données sont également toujours sécurisées en transit à l’aide du protocole HTTPS. HTTPS est le seul protocole pris en charge pour les interfaces REST Data Lake Store.

Pour en savoir plus sur le chiffrement des données en transit dans Data Lake, consultez [Chiffrement des données dans Azure Data Lake Store.](../../data-lake-store/data-lake-store-encryption.md)

## <a name="key-management-with-key-vault"></a>Gestion des clés dans Key Vault

Sans protection appropriée et la gestion des clés, le chiffrement est rendu inutilisable. Key Vault est la solution recommandée de Microsoft qui permet de gérer et de contrôler l’accès aux clés de chiffrement utilisées par les services cloud. Les autorisations d’accès aux clés peuvent être attribuées aux services ou aux utilisateurs via des comptes Azure Active Directory.

Key Vault soulage les entreprises de la nécessité de configurer, de corriger et de tenir à jour des modules de sécurité matériels (HSM) et des logiciels de gestion de clés. Quand vous utilisez Key Vault, vous conservez le contrôle. Microsoft ne voit jamais vos clés, et les applications n’y ont pas accès directement. Vous pouvez également importer ou générer des clés dans les modules HSM.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de la sécurité d’Azure](./overview.md)
- [Vue d’ensemble de la sécurité réseau d’Azure](network-overview.md)
- [Vue d’ensemble de la sécurité des bases de données d’Azure](../../azure-sql/database/security-overview.md)
- [Vue d’ensemble de la sécurité des machines virtuelles d’Azure](virtual-machines-overview.md)
- [Chiffrement des données au repos](encryption-atrest.md)
- [Meilleures pratiques en matière de chiffrement et de sécurité des données](data-encryption-best-practices.md)