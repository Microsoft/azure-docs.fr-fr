---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 11/21/2019
ms.author: alkohli
ms.openlocfilehash: 0c6845f081ccbe42e70964eaa939d58597e3b69b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185534"
---
1. Accédez à l’interface utilisateur web locale de votre appareil et connectez-vous à votre appareil. Assurez-vous que l’appareil est déverrouillé.

2. Accédez à la page **Paramètres réseau**. Prenez note de l’adresse IP de l’appareil pour l’interface réseau servant à se connecter au client.

3. Si vous utilisez un client Windows distant, démarrez le **Bloc-notes** en tant qu’administrateur, puis ouvrez le fichier hosts situé dans `C:\Windows\System32\Drivers\etc`.

4. Ajoutez l’entrée suivante à votre fichier hosts : `<Device IP address> <Blob service endpoint>`

    Vous avez obtenu le point de terminaison du service Blob à partir du compte de stockage Edge créé dans le portail Azure. Vous allez utiliser le suffixe du point de terminaison du service Blob uniquement.

    Pour référence, utilisez l’image suivante. Enregistrez le fichier `hosts`.

    ![Modifier le fichier hosts sur le client Windows](media/azure-stack-edge-gateway-add-device-ip-address-blob-service-endpoint/hosts-file-1.png)