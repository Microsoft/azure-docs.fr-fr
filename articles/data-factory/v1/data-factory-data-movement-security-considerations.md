---
title: Considérations de sécurité relatives au déplacement des données dans Azure Data Factory
description: Découvrez comment sécuriser les déplacements de données dans Azure Data Factory.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: anandsub
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: c694cf58f4c6b613cbc183753785a34bc15063bd
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093602"
---
# <a name="azure-data-factory---security-considerations-for-data-movement"></a>Azure Data Factory - Considérations de sécurité relatives au déplacement des données

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Considérations de sécurité relatives au déplacement des données dans Azure Data Factory](../data-movement-security-considerations.md).

## <a name="introduction"></a>Introduction
Cet article décrit l’infrastructure de sécurité de base qu’utilisent les services de déplacement des données dans Azure Data Factory pour sécuriser vos données. Les ressources de gestion d’Azure Data Factory reposent sur l’infrastructure de sécurité Azure et utilisent toutes les mesures de sécurité proposées par Azure.

Dans une solution Data Factory, vous créez un ou plusieurs [pipelines](data-factory-create-pipelines.md) de données. Un pipeline constitue un regroupement logique d’activités qui exécutent ensemble une tâche. Ces pipelines se trouvent dans la région où la fabrique de données a été créée. 

Même si Data Factory est disponible uniquement dans **USA Ouest**, **USA Est** et **Europe Nord**, le service de déplacement des données est [mondialement disponible dans plusieurs régions](data-factory-data-movement-activities.md#global). Le service Data Factory s’assure que les données ne quittent pas une zone/région géographique donnée, sauf si vous demandez explicitement au service d’utiliser une autre région dans le cas où le service de déplacement des données n’est pas encore déployé dans cette région. 

Azure Data Factory ne stocke aucune donnée à l’exception des informations d’identification du service lié pour les banques de données cloud, qui sont chiffrées à l’aide de certificats. Il vous permet de créer des workflows pilotés par les données afin d’orchestrer le déplacement de données entre les [banques de données prises en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), ainsi que le traitement des données à l’aide des [services de calcul](data-factory-compute-linked-services.md) situés dans d’autres régions ou dans un environnement local. Il vous permet également de [surveiller et gérer des workflows](data-factory-monitor-manage-pipelines.md) au moyen de programmes et à l’aide des mécanismes de l’interface utilisateur.

Le déplacement des données à l’aide d’Azure Data Factory a obtenu les **certifications** suivantes :
-   [HIPAA/HITECH](/compliance/regulatory/offering-hipaa-hitech)  
-   [ISO/IEC 27001](https://www.microsoft.com/en-us/trustcenter/Compliance/ISO-IEC-27001)  
-   [ISO/IEC 27018](https://www.microsoft.com/en-us/trustcenter/Compliance/ISO-IEC-27018) 
-   [CSA STAR](https://www.microsoft.com/en-us/trustcenter/Compliance/CSA-STAR-Certification)
     
Si la conformité Azure vous intéresse et que vous désirez savoir comment Azure protège sa propre infrastructure, consultez le [Centre de confidentialité Microsoft](https://microsoft.com/en-us/trustcenter/default.aspx). 

Cet article présente les principes de sécurité à prendre en compte dans les deux scénarios de déplacement de données suivants : 

- **Scénario cloud** : dans ce scénario, votre source et votre destination sont toutes deux accessibles publiquement via Internet. Cela inclut les services de stockage cloud managés comme Stockage Azure, Azure Synapse Analytics, Azure SQL Database, Azure Data Lake Store, Amazon S3, Amazon Redshift, les services SaaS tels que Salesforce et les protocoles web tels que FTP et OData. Vous trouverez [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) une liste complète des sources de données prises en charge.
- **Scénario hybride** : dans ce scénario, votre source ou votre destination se trouve derrière un pare-feu ou au sein d’un réseau d’entreprise local, ou la banque de données se trouve dans un réseau privé/virtuel réseau (le plus souvent la source) et n’est pas accessible publiquement. Les serveurs de base de données hébergés sur des machines virtuelles sont également inclus dans ce scénario.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="cloud-scenarios"></a>Scénarios cloud
### <a name="securing-data-store-credentials"></a>Sécurisation des informations d’identification des banques de données
Azure Data Factory protège les informations d’identification de vos banques de données en les **chiffrant** à l’aide de **certificats gérés par Microsoft**. Ces certificats sont remplacés tous les **deux ans** (avec renouvellement des certificats et migration des informations d’identification). Ces informations d’identification chiffrées sont stockées de manière sécurisée dans un **stockage Azure géré par les services de gestion d’Azure Data Factory**. Pour plus d’informations sur la sécurité du stockage Azure, consultez l’article [Vue d’ensemble des fonctionnalités de sécurité du stockage Azure](../../storage/blobs/security-recommendations.md).

### <a name="data-encryption-in-transit"></a>Chiffrement des données en transit
Tous les transferts de données entre les services de déplacement des données dans Data Factory et une banque de données cloud s’effectuent via un canal HTTPS ou TLS sécurisé, si la banque de données cloud prend en charge HTTPS ou TLS.

> [!NOTE]
> Toutes les connexions à **Azure SQL Database** et à **Azure Synapse Analytics** doivent toujours être chiffrées (via SSL/TLS) lorsque les données sont en transit depuis et vers la base de données. Lorsque vous créez un pipeline à l’aide d’un éditeur JSON, ajoutez la propriété **encryption** et définissez sa valeur sur **true** dans la **chaîne de connexion**. Si vous utilisez [l’Assistant de copie](data-factory-azure-copy-wizard.md), celui-ci définit cette propriété par défaut. Pour le **stockage Azure**, vous pouvez utiliser **HTTPS** dans la chaîne de connexion.

### <a name="data-encryption-at-rest"></a>Chiffrement des données au repos
Certaines banques de données prennent en charge le chiffrement des données au repos. Nous vous suggérons d’activer le mécanisme de chiffrement des données pour ces banques de données. 

#### <a name="azure-synapse-analytics"></a>Azure Synapse Analytics
Transparent Data Encryption (TDE) dans Azure Synapse Analytics vous aide à vous protéger contre les menaces d’activités malveillantes, par le biais d’un chiffrement et déchiffrement en temps réel de vos données au repos. Ce comportement est transparent pour le client. Pour plus d’informations, consultez [Sécuriser une base de données dans Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md).

#### <a name="azure-sql-database"></a>Azure SQL Database
Azure SQL Database prend également en charge TDE (Transparent Data Encryption) qui vous protège contre toute menace d’activité malveillante, en effectuant un chiffrement et un déchiffrement en temps réel des données, sans qu’il soit nécessaire de modifier l’application. Ce comportement est transparent pour le client. Pour plus d’informations, consultez [Transparent Data Encryption avec Azure SQL Database](/sql/relational-databases/security/encryption/transparent-data-encryption-with-azure-sql-database). 

#### <a name="azure-data-lake-store"></a>Azure Data Lake Store
Azure Data Lake Store assure également le chiffrement des données stockées dans le compte. Dans ce cas, il chiffre automatiquement les données avant qu’elles soient rendues persistantes, il les déchiffre avant qu’elles soient récupérées. Ainsi, ces opérations sont totalement transparentes vis-à-vis du client qui accède aux données. Pour plus d’informations, consultez [Sécurité dans Azure Data Lake Store](../../data-lake-store/data-lake-store-security-overview.md). 

#### <a name="azure-blob-storage-and-azure-table-storage"></a>Stockage Blob Azure et Stockage de tables Azure
Stockage Blob Azure et le stockage de tables Azure prennent en charge SSE (Storage Service Encryption), qui chiffre vos données avant qu’elles ne soient persistantes dans le stockage et qui les déchiffre avant leur récupération. Pour plus d’informations, consultez [Azure Storage Service Encryption pour les données au repos](../../storage/common/storage-service-encryption.md).

#### <a name="amazon-s3"></a>Amazon S3
Amazon S3 prend en charge le chiffrement des données au repos côté client et côté serveur. Pour plus d’informations, consultez [Protection des données à l’aide du chiffrement](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html). Actuellement, Data Factory ne prend pas en charge Amazon S3 dans un cloud privé virtuel (VPC).

#### <a name="amazon-redshift"></a>Amazon Redshift
Amazon Redshift prend en charge le chiffrement du cluster pour les données au repos. Pour plus d’informations, consultez [Amazon Redshift Database Encryption (Chiffrement de base de données Amazon Redshift)](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html). Actuellement, Data Factory ne prend pas en charge Amazon Redshift dans un cloud privé virtuel (VPC). 

#### <a name="salesforce"></a>Salesforce
Salesforce prend en charge Shield Platform qui permet de chiffrer tous les fichiers, pièces jointes et champs personnalisés. Pour plus d’informations, consultez [Understanding the Web Server OAuth Authentication Flow (Comprendre le flux d’authentification Web Server OAuth)](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_web_server_oauth_flow.htm).  

## <a name="hybrid-scenarios-using-data-management-gateway"></a>Scénarios hybrides (à l’aide de la passerelle de gestion des données)
Pour les scénarios hybrides, la passerelle de gestion des données doit être installée au sein d’un réseau local, d’un réseau virtuel (Azure) ou d’un cloud privé virtuel (Amazon). La passerelle doit pouvoir accéder aux banques de données locales. Pour obtenir des informations détaillées sur la passerelle, consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md). 

![Canaux de passerelle de gestion de données](media/data-factory-data-movement-security-considerations/data-management-gateway-channels.png)

Le **canal de commande** établit la communication entre les services de déplacement des données dans Data Factory et la passerelle de gestion des données. La communication contient des informations relatives à l’activité. Le canal de données est utilisé pour transférer des données entre les banques de données locales et les banques de données cloud.    

### <a name="on-premises-data-store-credentials"></a>Informations d’identification des banques de données locales
Les informations d’identification associées à vos banques de données locales sont stockées en local (et non dans le cloud). Elles peuvent être définies de trois façons différentes. 

- À l’aide de **texte brut** (moins sécurisé) via HTTPS depuis le portail Azure ou l’Assistant de copie. Les informations d’identification sont transmises en texte brut à la passerelle locale.
- À l’aide de la **bibliothèque de chiffrement JavaScript depuis l’Assistant de copie**.
- À l’aide de **l’application Gestionnaire des informations d’identification ClickOnce**. L’application ClickOnce s’exécute sur l’ordinateur local ayant accès à la passerelle et définit les informations d’identification pour la banque de données. Cette option et l’option suivante sont les plus sécurisées. L’application Gestionnaire des informations d’identification utilise par défaut le port 8050 de l’ordinateur hébergeant la passerelle pour établir une communication sécurisée.  
- Utilisez la cmdlet PowerShell [New-AzureRmDataFactoryEncryptValue](/powershell/module/az.datafactory/New-azDataFactoryEncryptValue) pour chiffrer les informations d’identification. L'applet de commande utilise le certificat qui a servi à configurer la passerelle pour chiffrer les informations d'identification. Vous pouvez utiliser les informations d’identification chiffrées retournées par cette cmdlet et les ajouter à l’élément **EncryptedCredential** de la **chaîne de connexion** dans le fichier JSON que vous utilisez avec la cmdlet [New-AzDataFactoryLinkedService](/powershell/module/az.datafactory/new-azdatafactorylinkedservice) ou dans l’extrait de code JSON de Data Factory Editor dans le portail. Cette option et l’application ClickOnce sont les options les plus sécurisées. 

#### <a name="javascript-cryptography-library-based-encryption"></a>Chiffrement à partir de la bibliothèque de chiffrement JavaScript
Vous pouvez chiffrer les informations d’identification des banques de données à l’aide de la [bibliothèque de chiffrement JavaScript](https://www.microsoft.com/download/details.aspx?id=52439) depuis [l’Assistant de copie](data-factory-copy-wizard.md). Lorsque vous sélectionnez cette option, l’Assistant de copie récupère la clé publique de la passerelle et l’utilise pour chiffrer les informations d’identification des banques de données. Les informations d’identification sont déchiffrées par l’ordinateur de la passerelle et protégées par l’API de protection des données Windows [(DPAPI)](/previous-versions/ms995355(v=msdn.10)).

**Navigateurs pris en charge :** IE8, IE9, IE10, IE11, Microsoft Edge, ainsi que les derniers navigateurs Firefox, Chrome, Opera et Safari. 

#### <a name="click-once-credentials-manager-app"></a>Application Gestionnaire des informations d’identification ClickOnce
Vous pouvez lancer l’application Gestionnaire des informations d’identification ClickOnce depuis le portail Azure ou l’Assistant de copie lors de la création de pipelines. Cette application s’assure que les informations d’identification ne sont pas transférées en texte brut sur le réseau. Elle utilise par défaut le port **8050** de l’ordinateur hébergeant la passerelle pour établir une communication sécurisée. Ce port peut être modifié en cas de besoin.  
  
![Port HTTPS pour la passerelle](media/data-factory-data-movement-security-considerations/https-port-for-gateway.png)

La passerelle de gestion des données utilise actuellement un seul **certificat**. Ce certificat est créé lors de l’installation de la passerelle (pour toutes les passerelles de gestion des données créées après novembre 2016 et ayant une version 2.4.xxxx.x ou ultérieure). Vous pouvez remplacer ce certificat par votre propre certificat SSL/TLS. Ce certificat est utilisé par l’application Gestionnaire des informations d’identification ClickOnce pour se connecter en toute sécurité à l’ordinateur de la passerelle et définir les informations d’identification des banques de données. Il stocke les informations d’identification des banques de données en local et de manière sécurisée à l’aide de l’API Windows [DPAPI](/previous-versions/ms995355(v=msdn.10)) sur l’ordinateur qui héberge la passerelle. 

> [!NOTE]
> Les passerelles plus anciennes qui ont été installées avant novembre 2016 ou ayant une version 2.3.xxxx.x continuent à utiliser les informations d’identification chiffrées et stockées dans le cloud. Même si vous mettez à niveau la passerelle vers la dernière version, les informations d’identification ne sont pas migrées vers un ordinateur local.    
  
| Version de la passerelle (lors de la création) | Informations d’identification stockées | Chiffrement/sécurité des informations d’identification | 
| --------------------------------- | ------------------ | --------- |  
| < = 2.3.xxxx.x | Dans le cloud | Chiffrement à l’aide d’un certificat (différent de celui utilisé par l’application Gestionnaire des informations d’identification) | 
| > = 2.4.xxxx.x | Localement | Sécurisation à l’aide de DPAPI | 
  

### <a name="encryption-in-transit"></a>Chiffrement en transit
Tous les transferts de données s’effectuent via un canal sécurisé **HTTPS** et **TLS via TCP** pour empêcher les attaques de l’intercepteur pendant la communication avec les services Azure.
 
Vous pouvez également utiliser un [VPN IPSec](../../vpn-gateway/vpn-gateway-about-vpn-devices.md) ou [ExpressRoute](../../expressroute/expressroute-introduction.md) pour renforcer la sécurité du canal de communication entre votre réseau local et Azure.

Un réseau virtuel est une représentation logique de votre réseau dans le cloud. Vous pouvez connecter un réseau local à votre réseau virtuel Azure (VNet) en configurant un VPN IPSec (de site à site) ou ExpressRoute (peering privé).     

Le tableau suivant récapitule les recommandations de configuration réseau et de la passerelle selon différentes combinaisons d’emplacements source et de destination pour le déplacement de données hybrides.

| Source | Destination | Configuration réseau | Configuration de la passerelle |
| ------ | ----------- | --------------------- | ------------- | 
| Local | Machines virtuelles et services cloud déployés au sein de réseaux virtuels | VPN IPSec (de point à site ou de site à site) | La passerelle peut être installée en local ou sur une machine virtuelle Azure au sein du réseau virtuel | 
| Local | Machines virtuelles et services cloud déployés au sein de réseaux virtuels | ExpressRoute (peering privé) | La passerelle peut être installée en local ou sur une machine virtuelle Azure au sein du réseau virtuel | 
| Local | Services Azure disposant d’un point de terminaison public | ExpressRoute (peering public) | La passerelle doit être installée en local | 

Les illustrations suivantes décrivent l’utilisation de la passerelle de gestion des données pour le déplacement de données entre une base de données locale et les services Azure à l’aide d’ExpressRoute et d’un VPN IPSec (avec un réseau virtuel) :

**Express Route :**
 
![ExpressRoute avec passerelle](media/data-factory-data-movement-security-considerations/express-route-for-gateway.png) 

**IPSec VPN :**

![VPN IPSec avec passerelle](media/data-factory-data-movement-security-considerations/ipsec-vpn-for-gateway.png)

### <a name="firewall-configurations-and-filtering-ip-address-of-gateway"></a>Configurations de pare-feu et filtrage des adresses IP de la passerelle

#### <a name="firewall-requirements-for-on-premisesprivate-network"></a>Configuration requise du pare-feu pour un réseau local/privé  
Dans une entreprise, le **pare-feu d’entreprise** s’exécute sur le routeur central. Le **pare-feu Windows** s’exécute quant à lui en tant que démon sur l’ordinateur local sur lequel est installée la passerelle. 

Le tableau suivant indique les paramètres de **port sortant** et de domaine requis pour le **pare-feu d’entreprise**.

| Noms de domaine | Ports sortants | Description |
| ------------ | -------------- | ----------- | 
| `*.servicebus.windows.net` | 443, 80 | Requis par la passerelle pour se connecter aux services de déplacement des données dans Data Factory |
| `*.core.windows.net` | 443 | Utilisé par la passerelle pour se connecter au compte de stockage Azure lorsque vous utilisez la fonctionnalité de [copie intermédiaire](data-factory-copy-activity-performance.md#staged-copy). | 
| `*.frontend.clouddatahub.net` | 443 | Requis par la passerelle pour se connecter au service Azure Data Factory. | 
| `*.database.windows.net` | 1433   | (FACULTATIF) Requis lorsque votre destination est Azure SQL Database ou Azure Synapse Analytics. Utilisez la fonctionnalité de copie intermédiaire pour copier des données vers Azure SQL Database/Azure Synapse Analytics sans ouvrir le port 1433. | 
| `*.azuredatalakestore.net` | 443 | (Facultatif) nécessaire lorsque votre destination est Azure Data Lake Store. | 

> [!NOTE] 
> Vous serez peut-être amené à gérer les ports/domaines de filtrage au niveau du pare-feu de l'entreprise, comme l'exigent les sources de données correspondantes. Ce tableau utilise uniquement Azure SQL Database, Azure Synapse Analytics et Azure Data Lake Store comme exemples.   

Le tableau suivant indique les paramètres de **port entrant** requis pour le **pare-feu Windows**.

| Ports entrants | Description | 
| ------------- | ----------- | 
| 8050 (TCP) | Requis par l’application Gestionnaire des informations d’identification pour définir en toute sécurité les informations d’identification des banques de données locales sur la passerelle. | 

![Configuration requise des ports de la passerelle](media/data-factory-data-movement-security-considerations/gateway-port-requirements.png)

#### <a name="ip-configurationsfiltering-in-data-store"></a>Configuration/filtrage des adresses IP dans la banque de données
Pour certaines banques de données hébergées dans le cloud, vous devez également approuver l'adresse IP de l'ordinateur qui y accède. Vérifiez que l'adresse IP de l'ordinateur de passerelle est approuvée/correctement configurée dans le pare-feu.

Les banques de données cloud suivantes requièrent l'approbation de l'adresse IP de l'ordinateur de passerelle. Certaines de ces banques de données peuvent ne pas nécessiter l'approbation des adresses IP par défaut. 

- [Azure SQL Database](../../azure-sql/database/firewall-configure.md) 
- [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md)
- [Azure Data Lake Store](../../data-lake-store/data-lake-store-secure-data.md#set-ip-address-range-for-data-access)
- [Azure Cosmos DB](../../cosmos-db/how-to-configure-firewall.md)
- [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) 

## <a name="frequently-asked-questions"></a>Forum aux questions

**Question :** La passerelle peut-elle être partagée entre plusieurs fabriques de données ?
**Réponse :** Nous ne prenons pas encore en charge cette fonctionnalité. mais y travaillons activement.

**Question :** Quels sont les paramètres de port permettant de garantir le bon fonctionnement de la passerelle ?
**Réponse :** La passerelle établit des connexions HTTP pour l’accès à Internet. Les **ports sortants 443 et 80** doivent être ouverts pour que la passerelle puisse établir cette connexion. Ouvrez le **port entrant 8050** uniquement au niveau de l’ordinateur (et non au niveau du pare-feu d’entreprise) pour l’application Gestionnaire des informations d’identification. Si vous utilisez Azure SQL Database ou Azure Synapse Analytics comme source/destination, vous devez également ouvrir le port **1433**. Pour plus d'informations, consultez la section [Configurations de pare-feu et filtrage d'adresses IP](#firewall-configurations-and-filtering-ip-address-of gateway). 

**Question :** De quels certificats la passerelle a-t-elle besoin ?
**Réponse :** La passerelle actuelle requiert un certificat utilisé par l’application Gestionnaire des informations d’identification pour configurer en toute sécurité les informations d’identification des banques de données. Ce certificat est un certificat auto-signé créé et configuré lors de l’installation de la passerelle. Vous pouvez également utiliser votre propre certificat TLS/SSL. Pour en savoir plus, consultez la section [Application Gestionnaire des informations d’identification ClickOnce](#click-once-credentials-manager-app). 

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les performances de l’activité de copie, consultez le [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md).