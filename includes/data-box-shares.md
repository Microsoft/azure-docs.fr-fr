---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 06/05/2020
ms.author: alkohli
ms.openlocfilehash: 5aaf0ce747b14b2fa9f2fcd9a65b774aa7d2db3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87102642"
---
Selon le compte de stockage sélectionné, Data Box crée jusqu’à :

* Trois partages pour chaque compte de stockage associé pour GPv1 et GPv2.
* Un partage pour le stockage Premium.
* Un partage pour le compte de stockage Blob.

Sous les partages d’objet blob de blocs et d’objet blob de pages, les entités de premier niveau sont des conteneurs et les entités de second niveau sont des objets blob. Sous les partages Azure Files, les entités de premier niveau sont des partages et les entités de second niveau sont des fichiers.

Le tableau suivant montre le chemin UNC aux partages sur votre Data Box et l’URL de chemin Stockage Azure où les données sont chargées. La dernière URL de chemin de Stockage Azure peut être dérivée à partir du chemin de partage UNC.
 
| Blobs et fichiers | Chemins et URL |
| --------------- | -------------- |
| Objets blob de blocs Azure | <li>Chemin UNC aux partages : `\\<DeviceIPAddress>\<StorageAccountName_BlockBlob>\<ContainerName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Objets blob de pages Azure  | <li>Chemin UNC aux partages : `\\<DeviceIPAddres>\<StorageAccountName_PageBlob>\<ContainerName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Chemin UNC aux partages : `\\<DeviceIPAddres>\<StorageAccountName_AzFile>\<ShareName>\files\a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |      

