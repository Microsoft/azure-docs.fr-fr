---
title: Exigences concernant le service Azure Import/Export | Microsoft Docs
description: L’objectif est ici de comprendre la configuration matérielle et logicielle requise pour le service Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 08/12/2019
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: d17b9cdebf06e7d754a23a7224bbe3dba1a2832c
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783604"
---
# <a name="azure-importexport-system-requirements"></a>Configuration système requise du service Azure Import/Export

Cet article décrit les principales exigences de votre service Azure Import/Export. Nous vous recommandons de lire attentivement les informations suivantes avant d’utiliser le service Import/Export, puis d’y revenir par la suite si nécessaire.

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Pour préparer les disques durs à l’aide de l’outil WAImportExport, un certain nombre de **systèmes d’exploitation 64 bits autorisant le chiffrement de lecteur BitLocker** sont pris en charge.


|Plateforme |Version |
|---------|---------|
|Windows     | Windows 7 Entreprise, Windows 7 Édition intégrale <br> Windows 8 Pro, Windows 8 Entreprise, Windows 8.1 Professionnel, Windows 8.1 Entreprise <br> Windows 10        |
|Windows Server     |Windows Server 2008 R2 <br> Windows Server 2012 et Windows Server 2012 R2         |

## <a name="other-required-software-for-windows-client"></a>Autres logiciels nécessaires pour le client Windows

|Plateforme |Version |
|---------|---------|
|.NET Framework    | 4.5.1       |
| BitLocker        |  _          |


## <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

Le service Azure Import/Export prend en charge les types de comptes de stockage suivants :

- Comptes de stockage v2 à usage général standard (recommandés pour la plupart des scénarios)
- Comptes de stockage d’objets blob
- Comptes de stockage v1 à usage général (déploiements Classic ou Azure Resource Manager)

Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](storage-account-overview.md).

Chaque tâche peut servir à transférer des données vers ou à partir d'un seul compte de stockage. Autrement dit, une même tâche d’importation/exportation ne peut pas englober plusieurs comptes de stockage. Pour plus d'informations sur la création d'un compte de stockage, consultez la page [Création d'un compte de stockage](storage-account-create.md).

> [!IMPORTANT]
> Pour les comptes de stockage sur lesquels la fonctionnalité [Points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md) a été activée, utilisez le paramètre **Autoriser les services Microsoft approuvés…** pour [activer le service Import/Export](./storage-network-security.md) afin d’importer/exporter des données vers ou depuis Azure.

## <a name="supported-storage-types"></a>Types de stockage pris en charge

Les types de stockage suivants sont pris en charge avec le service Azure Import/Export.


|Travail  |Service de stockage |Prise en charge  |Non pris en charge  |
|---------|---------|---------|---------|
|Importer     |  Stockage Blob Azure <br><br> Présentation du stockage de fichiers       | Objets blob de blocs et de pages pris en charge <br><br> Fichiers pris en charge          |
|Exporter     |   Stockage Blob Azure       | Objets blob de blocs, de pages et d’ajout pris en charge         | Fichiers Azure non pris en charge


## <a name="supported-hardware"></a>Matériel pris en charge

Dans le cadre du service Azure Import/Export, vous avez besoin de disques pris en charge pour copier les données.

### <a name="supported-disks"></a>Disques pris en charge

Les types de disques suivants sont pris en charge avec le service Azure Import/Export.


|Type de disque  |Size  |Prise en charge |
|---------|---------|---------|
|SSD    |   2,5"      |SATA III          |
|HDD     |  2,5"<br>3,5"       |SATA II, SATA III         |

Les types de disque suivants ne sont pas pris en charge :

- USB.
- Disque dur externe avec adaptateur USB intégré.
- Disques situés à l’intérieur du boîtier d’un disque dur externe.

Un travail d’importation/exportation peut avoir à lui seul :

- Un maximum de 10 disques HHD/SSD
- Un mélange de disques HDD/SSD de n’importe quelle taille

Il est possible de répartir un grand nombre de disques entre plusieurs tâches, et il n’existe aucune limite quant au nombre de tâches pouvant être créées. Dans le cas des tâches d’importation, seul le premier volume de données du lecteur est traité. Il doit être formaté avec NTFS.

Au moment de préparer les disques durs et de copier les données avec l’outil WAImportExport, vous pouvez utiliser les adaptateurs USB externes. La plupart des adaptateurs USB 3.0 ou version ultérieure du commerce doivent fonctionner.

## <a name="next-steps"></a>Étapes suivantes

* [Transfert de données avec l’utilitaire de ligne de commande AzCopy](./storage-use-azcopy-v10.md)