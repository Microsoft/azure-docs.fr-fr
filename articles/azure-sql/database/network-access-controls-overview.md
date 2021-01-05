---
title: Contrôles d’accès réseau
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Vue d’ensemble de la gestion et du contrôle d’accès réseau pour Azure SQL Database et Azure Synapse Analytics.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 03/09/2020
ms.openlocfilehash: 90bc57af3aaf0d11cd354bfe7163014f836a72e8
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460003"
---
# <a name="azure-sql-database-and-azure-synapse-analytics-network-access-controls"></a>Contrôles d’accès réseau Azure SQL Database et Azure Synapse Analytics

Lorsque vous créez un serveur SQL logique à partir du [portail Azure](single-database-create-quickstart.md) pour Azure SQL Database et Azure Synapse Analytics, le résultat est un point de terminaison public au format *votrenomdeserveur.basededonnées.windows.net*.

Vous pouvez utiliser les contrôles d'accès réseau suivants pour autoriser de manière sélective l'accès à la base de données via le point de terminaison public :

- Autoriser les services Azure : Si défini sur ACTIVÉ, d’autres ressources dans la limite Azure, par exemple une machine virtuelle Azure, peuvent accéder à SQL Database
- Règles de pare-feu IP : Utilisez cette fonctionnalité pour autoriser explicitement les connexions à partir d’une adresse IP spécifique, par exemple à partir d’ordinateurs locaux

Vous pouvez également autoriser l’accès privé à la base de données à partir des [réseaux virtuels](../../virtual-network/virtual-networks-overview.md) via :

- Règles de pare-feu de réseau virtuel : utilisez cette fonctionnalité pour autoriser le trafic à partir d’un réseau virtuel spécifique au sein de la limite Azure.
- Liaison privée : utilisez cette fonctionnalité pour créer un point de terminaison privé pour le [serveur SQL logique](logical-servers.md) au sein d’un réseau virtuel spécifique.

> [!IMPORTANT]
> Cet article ne s'applique *pas* à **SQL Managed Instance**. Pour plus d’informations sur la configuration réseau, consultez [Connexion à Azure SQL Managed Instance](../managed-instance/connect-application-instance.md).

Pour bien comprendre ces contrôles d’accès et ce qu’ils font, regardez la vidéo ci-dessous :

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Data-Exposed--SQL-Database-Connectivity-Explained/player?WT.mc_id=dataexposed-c9-niner]

## <a name="allow-azure-services"></a>Autoriser les services Azure

Par défaut, lors de la création d’un serveur SQL logique [à partir du portail Azure](single-database-create-quickstart.md), ce paramètre est défini sur **DÉSACTIVÉ**. Ce paramètre s’affiche lorsque la connectivité est autorisée à l’aide d’un point de terminaison de service public.

Vous pouvez aussi changer ce paramètre via le volet du pare-feu une fois le serveur SQL logique créé, comme suit.
  
![Capture d’écran de la gestion du pare-feu du serveur][2]

Lorsque la valeur est définie sur **ACTIVÉ**, votre serveur autorise les communications à partir de toutes les ressources situées dans la limite Azure, qu’elles fassent partie de votre abonnement ou non.

Le paramètre **ACTIVÉ** est souvent plus permissif que ce que souhaitent la plupart des clients. Par conséquent, vous préférerez peut-être définir ce paramètre sur **DÉSACTIVÉ** et le remplacer par des règles de pare-feu IP ou des règles de pare-feu de réseau virtuel plus restrictives. 

Cela étant, ce choix affecte les fonctionnalités suivantes qui s’exécutent sur des machines virtuelles dans Azure qui ne font pas partie de votre réseau virtuel et qui, par conséquent, se connectent à la base de données par le biais d’une adresse IP Azure :

### <a name="import-export-service"></a>Service d’importation/exportation

Le service d’importation/exportation ne fonctionne pas lorsque l’option  **Autoriser l’accès aux services Azure** est **désactivée**. Toutefois, vous pouvez contourner le problème [en exécutant manuellement sqlpackage.exe à partir d’une machine virtuelle Azure ou en effectuant l’exportation](./database-import-export-azure-services-off.md) directement dans votre code à l’aide de l’API DACFx.

### <a name="data-sync"></a>Synchronisation des données

Pour utiliser la fonctionnalité de synchronisation des données avec l'option **Autoriser l'accès aux services Azure** définie sur **DÉSACTIVÉ**, vous devez créer des entrées de règle de pare-feu individuelles afin d'[ajouter des adresses IP](firewall-create-server-level-portal-quickstart.md) à partir de la **balise du service SQL** pour la région qui héberge la base de données **Hub**.
Ajoutez ces règles de pare-feu au niveau du serveur aux serveurs hébergeant à la fois des bases de données **Hub** et **Member** (qui peuvent être dans des régions différentes)

Utilisez le script PowerShell suivant pour générer les adresses IP correspondant à la balise du service SQL pour la région USA Ouest

```powershell
PS C:\>  $serviceTags = Get-AzNetworkServiceTag -Location eastus2
PS C:\>  $sql = $serviceTags.Values | Where-Object { $_.Name -eq "Sql.WestUS" }
PS C:\> $sql.Properties.AddressPrefixes.Count
70
PS C:\> $sql.Properties.AddressPrefixes
13.86.216.0/25
13.86.216.128/26
13.86.216.192/27
13.86.217.0/25
13.86.217.128/26
13.86.217.192/27
```

> [!TIP]
> Get-AzNetworkServiceTag retourne la plage globale de la balise du service SQL malgré la spécification du paramètre Location. Veillez à la filtrer sur la région qui héberge la base de données Hub utilisée par votre groupe de synchronisation.

Notez que la sortie du script PowerShell est en notation CIDR (Classless Inter-Domain Routing). Celle-ci doit être convertie au format d’adresse IP de début et de fin à l’aide de la commande [Get-IPrangeStartEnd.ps1](https://gallery.technet.microsoft.com/scriptcenter/Start-and-End-IP-addresses-bcccc3a9) comme suit :

```powershell
PS C:\> Get-IPrangeStartEnd -ip 52.229.17.93 -cidr 26
start        end
-----        ---
52.229.17.64 52.229.17.127
```

Vous pouvez utiliser ce script PowerShell supplémentaire pour convertir toutes les adresses IP en notation CIDR au format d’adresse IP de début et de fin.

```powershell
PS C:\>foreach( $i in $sql.Properties.AddressPrefixes) {$ip,$cidr= $i.split('/') ; Get-IPrangeStartEnd -ip $ip -cidr $cidr;}
start          end
-----          ---
13.86.216.0    13.86.216.127
13.86.216.128  13.86.216.191
13.86.216.192  13.86.216.223
```

Vous pouvez maintenant les ajouter en tant que règles de pare-feu distinctes, puis définir **Autoriser les services Azure à accéder au serveur** sur OFF (désactivé).

## <a name="ip-firewall-rules"></a>Règles de pare-feu IP

Le pare-feu basé sur IP est une fonctionnalité du serveur SQL logique dans Azure qui empêche tout accès à votre serveur tant que vous n’avez pas [ajouté les adresses IP](firewall-create-server-level-portal-quickstart.md) des ordinateurs clients de manière explicite.

## <a name="virtual-network-firewall-rules"></a>Règles de pare-feu de réseau virtuel

En plus des règles IP, le pare-feu du serveur vous permet de définir des *règles de réseau virtuel*.  
Pour en savoir plus, consultez [Points de terminaison de service de réseau virtuel et règles dans Azure SQL Database](vnet-service-endpoint-rule-overview.md) ou regardez cette vidéo :

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Data-Exposed--Demo--Vnet-Firewall-Rules-for-SQL-Database/player?WT.mc_id=dataexposed-c9-niner]

### <a name="azure-networking-terminology"></a>Terminologie des réseaux Azure

Apprenez à connaître les termes de réseau suivants d’Azure Networking quand vous explorez les règles de pare-feu de réseau virtuel

**Réseau virtuel :** Vous pouvez avoir des réseaux virtuels associés à votre abonnement Azure.

**Sous-réseau :** Un réseau virtuel contient des **sous-réseaux**. Toutes les machines virtuelles Azure que vous avez sont assignées à des sous-réseaux. Un sous-réseau peut contenir plusieurs machines virtuelles ou d’autres nœuds de calcul. Les nœuds de calcul qui se trouvent en dehors de votre réseau virtuel ne peuvent pas accéder à ce dernier, sauf si vous configurez votre sécurité pour leur en donner l’accès.

**Point de terminaison de service de réseau virtuel :** Un [point de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md) est un sous-réseau dont les valeurs de propriétés incluent un ou plusieurs noms de type de service Azure formels. Dans cet article, nous nous intéressons au nom de type de **Microsoft.Sql**, qui fait référence au service Azure nommé SQL Database.

**Règle de réseau virtuel :** une règle de réseau virtuel pour votre serveur est un sous-réseau qui figure dans la liste de contrôle d’accès (ACL) de votre serveur. Pour figurer dans l’ACL pour votre base de données dans SQL Database, le sous-réseau doit contenir le nom de type **Microsoft.Sql**. Une règle de réseau virtuel donne l’instruction au serveur d’accepter les communications provenant de tout nœud se trouvant sur le sous-réseau.

## <a name="ip-vs-virtual-network-firewall-rules"></a>IP ou Règles de pare-feu de réseau virtuel

Le pare-feu Azure SQL Database permet de spécifier des plages d’adresses IP à partir desquelles les communications sont acceptées dans SQL Database. Cette approche est indiquée pour les adresses IP stables qui se trouvent en dehors du réseau privé Azure. Toutefois, les machines virtuelles à l’intérieur du réseau privé Azure sont configurées avec des adresses IP *dynamiques*. Les adresses IP dynamiques peuvent changer lorsque votre machine virtuelle est redémarrée et ainsi invalider la règle de pare-feu basée sur IP. Spécifier une adresse IP dynamique dans une règle de pare-feu au sein d’un environnement de production serait inimaginable.

Vous pouvez contourner cette limitation en obtenant une adresse IP *statique* pour votre machine virtuelle. Pour plus de détails, consultez [Créer une machine virtuelle avec une adresse IP publique statique à partir du portail Azure](../../virtual-network/virtual-network-deploy-static-pip-arm-portal.md). L’approche des IP statiques peut toutefois devenir difficile à gérer, et s’avère coûteuse quand elle est appliquée à grande échelle.

Les règles de réseau virtuel constituent une alternative plus facile pour établir et gérer l’accès à partir d’un sous-réseau spécifique qui contient vos machines virtuelles.

> [!NOTE]
> Vous ne pouvez pas encore avoir SQL Database dans un sous-réseau. Si votre serveur était un nœud sur un sous-réseau de votre réseau virtuel, tous les nœuds situés dans le réseau virtuel pourraient communiquer avec votre instance SQL Database. Dans ce cas, vos machines virtuelles pourraient communiquer avec le serveur SQL Database sans avoir à utiliser de règles de réseau virtuel ni de règles IP.

## <a name="private-link"></a>Private Link

Private Link vous permet de vous connecter à un serveur par le biais d’un **point de terminaison privé**. Un point de terminaison privé est une adresse IP privée au sein d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) et d’un sous-réseau spécifiques.

## <a name="next-steps"></a>Étapes suivantes

- Pour un guide de démarrage rapide sur la création d’une règle de pare-feu IP au niveau du serveur, consultez [Créer une base de données dans SQL Database](single-database-create-quickstart.md).

- Pour un guide de démarrage rapide sur la création d’une règle de pare-feu de réseau virtuel au niveau du serveur, consultez [Points de terminaison et règles de service de réseau virtuel pour Azure SQL Database](vnet-service-endpoint-rule-overview.md).

- Pour obtenir de l’aide sur la connexion à une base de données dans SQL Database à partir d’applications open source ou tierces, consultez [Exemples de code de démarrage rapide client pour SQL Database](/previous-versions/azure/ee336282(v=azure.100)).

- Pour plus d’informations sur les autres ports que vous devrez peut-être ouvrir, consultez la section **SQL Database : exécution externe ou exécution interne** de [Ports au-delà de 1433 pour ADO.NET 4.5 et SQL Database](adonet-v12-develop-direct-route-ports.md).

- Pour une vue d’ensemble de la connectivité Azure SQL Database, consultez [Architecture de connectivité Azure SQL](connectivity-architecture.md).

- Pour obtenir une vue d’ensemble de la sécurité Azure SQL Database, consultez [Sécurisation de votre base de données](security-overview.md)

<!--Image references-->
[1]: media/quickstart-create-single-database/new-server2.png
[2]: media/quickstart-create-single-database/manage-server-firewall.png
