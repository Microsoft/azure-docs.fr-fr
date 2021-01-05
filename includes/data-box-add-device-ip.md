---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/07/2018
ms.author: alkohli
ms.openlocfilehash: d01cf90c42efbe611748027d0ea47ff8af3e5621
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94553166"
---
1. Connectez-vous à l’appareil Data Box. Assurez-vous qu’il est déverrouillé.

    ![La capture d’écran montre votre tableau de bord avec l’appareil affiché comme étant déverrouillé.](media/data-box-add-device-ip/data-box-connect-via-rest-1.png)

2. Accédez à **Définir des interfaces réseau**. Prenez note de l’adresse IP de l’appareil pour l’interface réseau servant à se connecter au client.

    ![La capture d’écran montre les paramètres réseau dans lesquels vous pouvez voir l’adresse IP.](media/data-box-add-device-ip/data-box-connect-via-rest-2.png)

3. Accédez à **Connexion et copie**, puis cliquez sur **REST**.

    ![La capture d’écran montre le volet Connexion et copie dans lequel vous pouvez sélectionner REST comme paramètre d’accès.](media/data-box-add-device-ip/data-box-connect-via-rest-3.png)

4. Dans la boîte de dialogue **Accéder au compte de stockage et charger des données**, copiez le **Point de terminaison du service BLOB**.

    ![La capture d’écran montre la boîte de dialogue Accéder au compte de stockage et charger des données dans laquelle vous pouvez copier le point de terminaison du service BLOB.](media/data-box-add-device-ip/data-box-connect-via-rest-4.png)

5. Démarrez le **Bloc-notes** en tant qu’administrateur, puis ouvrez le fichier **hosts** qui se trouve dans `C:\Windows\System32\Drivers\etc`.
6. Ajoutez l’entrée suivante à votre fichier **hosts** : `<device IP address> <Blob service endpoint>`
7. Pour référence, utilisez l’image suivante. Enregistrez le fichier **hosts**.

    ![La capture d’écran montre un document Bloc-notes avec l’adresse IP et le point de terminaison du service BLOB ajoutés.](media/data-box-add-device-ip/data-box-connect-via-rest-5.png)
