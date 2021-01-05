---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 06/26/2019
ms.author: alkohli
ms.openlocfilehash: 09d9b5bbf3f9ca7a4eef37891d03c9c865e7f74b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67448628"
---
Un certificat SSL correct vérifie que vous envoyez des données chiffrées au serveur adéquat. Outre le chiffrement, le certificat permet également l’authentification. Vous pouvez charger votre propre certificat SSL approuvé par l’intermédiaire de l’interface PowerShell de l’appareil.

1. [Connectez-vous à l’interface PowerShell](#connect-to-the-powershell-interface).
2. Utilisez la cmdlet `Set-HcsCertificate` pour charger le certificat. À l’invite, fournissez les paramètres suivants :

   - `CertificateFilePath` : chemin d’accès au partage qui contient le fichier de certificat, au format *.pfx*.
   - `CertificatePassword` : mot de passe permettant de protéger le certificat.
   - `Credentials` : nom d’utilisateur grâce auxquels vous pouvez accéder au partage qui contient le certificat. Renseignez le mot de passe du partage réseau lorsque vous y êtes invité.

     L’exemple suivant montre comment utiliser cette cmdlet :

     ```
     Set-HcsCertificate -Scope LocalWebUI -CertificateFilePath "\\myfileshare\certificates\mycert.pfx" -CertificatePassword "mypassword" -Credential "Username"
     ```

