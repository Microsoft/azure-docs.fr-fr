---
title: Fichier include
description: Fichier include
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 01/22/2019
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: ebc23ce4238c736442fbc4507e858876f9192fd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76020899"
---
## <a name="associate-an-azure-storage-account-to-iot-hub"></a>Associer un compte Azure Storage à IoT Hub

Étant donné que l’application d’appareil simulé charge un fichier dans un objet Blob, vous devez disposer d’un compte [Azure Storage](../articles/storage/common/storage-account-create.md) associé à votre IoT Hub. Lorsque vous associez un compte de stockage Azure avec un IoT Hub, celui-ci génère un URI SAS. Un appareil peut utiliser cet URI SAS pour charger en toute sécurité un fichier vers un conteneur d’objets blob. Le service IoT Hub et les Kits de développement logiciel (SDK) d’appareil coordonnent le processus qui génère l’URI SAS et le rend disponible pour un appareil à utiliser pour charger un fichier.

Suivez les instructions de [Configurer les téléchargements de fichiers à l’aide du portail Azure](../articles/iot-hub/iot-hub-configure-file-upload.md). Assurez-vous qu’un conteneur de blobs est associé à votre IoT Hub et que les notifications de fichier sont activées.

![Activer les notifications de fichier dans le portail](./media/iot-hub-associate-storage/file-notifications-vs2019.png)
