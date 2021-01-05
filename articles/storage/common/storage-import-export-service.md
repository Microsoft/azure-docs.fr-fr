---
title: Utilisation du service Azure Import/Export pour transférer des données vers et à partir de Stockage Azure | Microsoft Docs
description: Découvrez comment créer des tâches d’importation et d’exportation dans le portail Azure pour transférer des données vers et à partir de Stockage Azure.
author: alkohli
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: b2918c5842d6470e634518ff9c12f6f04aefc920
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88185381"
---
# <a name="what-is-azure-importexport-service"></a>Qu’est-ce que le service Azure Import/Export ?

Le service Azure Import/Export est utilisé pour importer en toute sécurité des volumes importants de données dans Stockage Blob Azure et Azure Files en expédiant des lecteurs de disque vers un centre de données Azure. Vous pouvez également utiliser ce service pour transférer des données de Stockage Blob Azure vers des lecteurs de disque et les expédier vers vos sites locaux. Les données d’un ou plusieurs lecteurs de disque peuvent être importées dans le Stockage Blob Azure ou Azure Files.

Utilisez vos propres lecteurs de disque et transférez des données avec le service Azure Import/Export. Vous pouvez également utiliser les lecteurs de disque fournis par Microsoft.

Si vous souhaitez transférer des données à l’aide des lecteurs de disque fournis par Microsoft, vous pouvez utiliser un [disque Azure Data Box](../../databox/data-box-disk-overview.md) pour importer des données dans Azure. Microsoft envoie jusqu’à 5 lecteurs de disque SSD chiffrés dotés d’une capacité totale de 40 To par commande à votre centre de données via un transporteur régional. Vous pouvez rapidement configurer les lecteurs de disque, y copiez des données via une connexion USB 3.0, puis les réexpédier à Azure. Pour plus d’informations, accédez à la [vue d’ensemble d’Azure Data Box Disk](../../databox/data-box-disk-overview.md).

## <a name="azure-importexport-use-cases"></a>Cas d’utilisation du service Azure Import/Export

Envisagez d’utiliser le service Azure Import/Export lorsque le chargement ou le téléchargement de données sur le réseau est trop lent ou lorsque l’obtention d’une bande passante réseau supplémentaire est coûteuse. Utilisez ce service dans les situations suivantes :

* **Migration des données vers le cloud** : déplacez rapidement de grandes quantités de données vers Azure et à moindre coût.
* **Distribution de contenu** : envoyez rapidement des données aux sites de vos clients.
* **Sauvegarder** : sauvegardez vos données locales dans Stockage Azure.
* **Récupération de données** : récupérez de grandes quantités de données stockées pour les transférer vers votre site local.

## <a name="importexport-components"></a>Composants du service Azure Import/Export

Le service Azure Import/Export utilise les composants suivants :

* **Service d’importation/exportation** : ce service disponible dans le portail Azure permet à l’utilisateur de créer des travaux d’importation (chargement) et d’exportation (téléchargement) de données et d’en effectuer le suivi.  

* **Outil WAImportExport** : outil en ligne de commande qui effectue les opérations suivantes :
  * prépare vos lecteurs de disque expédiés à l’importation ;
  * facilite la copie de vos données sur le lecteur ;
  * Chiffre les données sur le lecteur avec BitLocker AES 256 bits. Vous pouvez utiliser un protecteur de clé externe pour protéger votre clé BitLocker.
  * génère les fichiers journaux de lecteur utilisés lors de la création de l’importation ;
  * permet d’identifier le nombre de disques requis pour les travaux d’exportation.

> [!NOTE]
> L’outil WAImportExport est disponible dans deux versions, la version 1 et la version 2. Nous vous recommandons d’utiliser :
>
> * la version 1 pour l’importation/exportation dans Stockage Blob Azure ;
> * la version 2 pour l’importation de données dans des fichiers Azure Files.
>
> L’outil WAImportExport est compatible uniquement avec le système d’exploitation Windows 64 bits. Pour connaître les versions de système d’exploitation prises en charge, consultez [Conditions requises pour le service Azure Import/Export](storage-import-export-requirements.md#supported-operating-systems).

* **Lecteurs de disque** : vous pouvez expédier des disques SSD (Solid-state drive) ou des disques HDD (Hard disk drive) au centre de données Azure. Lorsque vous créez un travail d’importation, vous expédiez les lecteurs de disque contenant vos données. Lorsque vous créez un travail d’exportation, vous expédiez des lecteurs vides au centre de données Azure. Pour les types de disques, consultez [Types de disques pris en charge](storage-import-export-requirements.md#supported-hardware).

## <a name="how-does-importexport-work"></a>Comment fonctionne le service Import/Export ?

Le service Azure Import/Export permet le transfert de données dans des objets blob Azure ou des fichiers Azure Files en créant des travaux. Utilisez le Portail Azure ou l’API REST Azure Resource Manager pour créer des travaux. Chaque travail est associé à un seul compte de stockage.

Il peut s’agir d’un travail d’importation ou d’exportation. Un travail d’importation vous permet d’importer des données dans des objets blob Azure ou des fichiers Azure Files, tandis que le travail d’exportation permet d’exporter des données à partir d’objets blob Azure. Dans le cas d’un travail d’importation, vous expédiez des disques contenant vos données. Lorsque vous créez un travail d’exportation, vous expédiez des lecteurs vides à un centre de données Azure. Dans les deux cas, vous pouvez expédier jusqu’à 10 lecteurs de disque par travail.

### <a name="inside-an-import-job"></a>Dans un travail d’importation

Globalement, un travail d’importation comprend les opérations suivantes :

1. Déterminer les données à importer, le nombre de disques dont vous avez besoin, l’emplacement de l’objet blob de destination de vos données dans Stockage Azure.
2. Utiliser l’outil WAImportExport pour copier les données sur les lecteurs de disque. Chiffrer les lecteurs de disque avec BitLocker.
3. Créer un travail d’importation dans votre compte de stockage cible dans le Portail Azure. Charger les fichiers journaux du lecteur.
4. Indiquer l’adresse de retour et le numéro de compte transporteur à utiliser pour le retour des disques.
5. Expédier les lecteurs de disque à l’adresse d’expédition indiquée lors de la création du travail.
6. Mettez à jour le numéro de suivi et envoyez le travail d’importation.
7. Les disques sont réceptionnés et traités dans le centre de données Azure.
8. Les disques sont expédiés à l’aide de votre compte de transporteur à l’adresse de retour indiquée dans le travail d’importation.

> [!NOTE]
> Pour les expéditions à l’échelle nationale (dans le pays/la région du centre de données), veuillez partager un compte de transporteur domestique.
>
> Pour les expéditions à l’étranger (en dehors du pays/de la région du centre de données), veuillez partager un compte de transporteur international.

 ![Figure 1 : flux d’importation de travail](./media/storage-import-export-service/importjob.png)

Pour obtenir des instructions pas à pas sur l’importation de données, consultez :

* [Importer des données dans des objets blob Azure](storage-import-export-data-to-blobs.md)
* [Importer des données dans des fichiers Azure Files](storage-import-export-data-to-files.md)

### <a name="inside-an-export-job"></a>Dans un travail d’exportation

> [!IMPORTANT]
> Le service prend uniquement en charge l’exportation d’objets blob Azure. L’exportation de fichiers Azure n’est pas prise en charge.

Globalement, un travail d’exportation comprend les opérations suivantes :

1. Déterminer les données à exporter, le nombre de lecteurs vous besoin, les objets blob sources ou les chemins de conteneur de vos données dans le stockage blob.
2. Créer un travail d’importation dans votre compte de stockage source dans le Portail Azure.
3. Spécifier les objets blob sources ou les chemins de conteneur de vos données à exporter.
4. Indiquer l’adresse de retour et le numéro de compte transporteur à utiliser pour le retour des disques.
5. Expédier les lecteurs de disque à l’adresse d’expédition indiquée lors de la création du travail.
6. Mettez à jour le numéro de suivi et envoyez le travail d’exportation.
7. Les disques sont réceptionnés et traités dans le centre de données Azure.
8. Les disques sont chiffrés à l’aide de BitLocker et les clés sont disponibles via le Portail Azure.  
9. Les disques sont expédiés à l’aide de votre compte de transporteur à l’adresse de retour indiquée dans le travail d’importation.

> [!NOTE]
> Pour les expéditions à l’échelle nationale (dans le pays/la région du centre de données), veuillez partager un compte de transporteur domestique.
>
> Pour les expéditions à l’étranger (en dehors du pays/de la région du centre de données), veuillez partager un compte de transporteur international.
  
 ![Figure 2 : flux d’exportation de travail](./media/storage-import-export-service/exportjob.png)

Pour obtenir des instructions pas à pas sur l’exportation de données, consultez[Exporter des données à partir d’objets blob Azure](storage-import-export-data-from-blobs.md).

## <a name="region-availability"></a>Disponibilité des régions

Le service Azure Import/Export prend en charge la copie des données vers et depuis tous les comptes de stockage Azure. Vous pouvez expédier des lecteurs de disque aux emplacements suivants. Si votre compte de stockage se trouve dans un emplacement Azure qui n’est pas spécifié ici, un emplacement d’expédition de remplacement est proposé lorsque vous créez le travail.

### <a name="supported-shipping-locations"></a>Emplacements d’expédition pris en charge :

|Pays/région  |Pays/région  |Pays/région  |Pays/région  |
|---------|---------|---------|---------|
|USA Est    | Europe Nord        | Inde centrale        |US Gov Iowa         |
|USA Ouest     |Europe Ouest         | Inde Sud        | Est des États-Unis – US DoD        |
|USA Est 2    | Asie Est        |  Inde Ouest        | Centre des États-Unis – US DoD        |
|USA Ouest 2     | Asie Sud-Est        | Centre du Canada        | Chine orientale         |
|USA Centre     | Australie Est        | Est du Canada        | Chine du Nord        |
|Centre-Nord des États-Unis     |  Sud-Australie Est       | Brésil Sud        | Sud du Royaume-Uni        |
|États-Unis - partie centrale méridionale     | OuJapon Est        |Centre de la Corée         | Centre de l’Allemagne        |
|Centre-USA Ouest     |  Japon Est       | Gouvernement américain - Virginie        | Nord-Est de l’Allemagne        |
|Afrique du Sud Ouest   |  Afrique du Sud Nord |

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Les données présentes sur le lecteur doivent être chiffrées à l’aide du chiffrement de lecteur BitLocker AES 256 bits. Ce chiffrement permet de protéger vos données en transit.

Pour les travaux d’importation, les lecteurs sont chiffrés de deux manières.  

* Spécifiez l’option lors de l’utilisation du fichier *dataset.csv* pendant l’exécution de l’outil WAImportExport pour préparer le disque.

* Activez le chiffrement BitLocker manuellement sur le disque. Spécifiez la clé de chiffrement dans le fichier *driveset.csv* lors de l’exécution de la ligne de commande de l’outil WAImportExport pendant la préparation du disque. La clé de chiffrement BitLocker peut être davantage protégée à l’aide d’un protecteur de clé externe (également appelé « clé gérée par Microsoft ») ou d’une clé gérée par le client. Pour plus d’informations, consultez le guide indiquant comment [Utiliser une clé gérée par le client pour protéger votre clé BitLocker](storage-import-export-encryption-key-portal.md).

Pour les travaux d’exportation, une fois vos données copiées sur le disque, le service chiffre le disque à l’aide de BitLocker avant de vous le renvoyer. La clé de chiffrement vous est fournie par le biais du Portail Azure. Le lecteur doit être déverrouillé à l’aide de l’outil WAImporExport en utilisant la clé.

[!INCLUDE [storage-import-export-delete-personal-info.md](../../../includes/storage-import-export-delete-personal-info.md)]

### <a name="pricing"></a>Tarifs

**Frais de manipulation de disque**

Des frais de manipulation sont appliqués pour chaque disque traité dans le cadre de votre travail d’importation/exportation. Pour plus d’informations, consultez [Tarification d’Azure Import/Export](https://azure.microsoft.com/pricing/details/storage-import-export/).

**Frais d’expédition**

Lorsque vous envoyez des disques à Azure, vous payez le coût d’expédition au transporteur. Lorsque Microsoft vous renvoie les disques, les frais d’expédition sont facturés au compte de transporteur que vous avez spécifié lors de la création du travail.

**Frais de transaction**

[Les frais de transaction de stockage standard](https://azure.microsoft.com/pricing/details/storage/) s’appliquent pour l’importation et l’exportation des données. Des frais de sortie standard sont également applicables avec les frais de transaction de stockage lorsque les données sont exportées depuis le stockage Azure. Pour plus d’informations sur les frais de sortie, consultez [Tarification du transfert de données](https://azure.microsoft.com/pricing/details/data-transfers/).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser le service Azure Import/Export pour :

* [Importer des données dans des objets blob Azure](storage-import-export-data-to-blobs.md)
* [Exporter des données à partir d’objets blob Azure](storage-import-export-data-from-blobs.md)
* [Importer des données dans des fichiers Azure Files](storage-import-export-data-to-files.md)
