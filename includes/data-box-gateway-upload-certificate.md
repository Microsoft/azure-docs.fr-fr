---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 27a4d775b8d88e02be69655e5adfc6155f867ef6
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580919"
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
