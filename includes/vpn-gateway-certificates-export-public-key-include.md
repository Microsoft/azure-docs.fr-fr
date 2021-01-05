---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/19/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 12e9bec0c560f1b068b07a1b6afe218a112e439f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94553130"
---
Après avoir créé un certificat racine autosigné, exportez le fichier .cer de clé publique du certificat racine (et non celui de la clé privée). Vous chargerez ce fichier plus tard sur Azure. Les étapes suivantes vous aideront à exporter le fichier .cer pour votre certificat racine auto-signé :

1. Pour obtenir un fichier .cer du certificat, ouvrez **Gérer les certificats utilisateur**. Localisez le certificat racine auto-signé, généralement dans « Certificates - Curent User\Personal\Certificates » et cliquez avec le bouton droit. Cliquez sur **Toutes les tâches**, puis cliquez sur **Exporter**. Cette opération ouvre **l’Assistant Exportation de certificat**. Si vous ne trouvez pas le certificat sous Current User\Personal\Certificates, il est possible que vous ayez ouvert par erreur « Certificats – Ordinateur local » et non « Certificats – Utilisateur actuel ». Si vous voulez ouvrir le Gestionnaire de certificats dans le champ d’application utilisateur actuel en utilisant PowerShell, tapez *certmgr* dans la fenêtre de la console.

   ![Capture d’écran montrant la fenêtre Certificats pour l’utilisateur actuel avec les Certificats sélectionnés et un menu contextuel avec Exporter la sélection de Toutes les tâches.](./media/vpn-gateway-certificates-export-public-key-include/export.png)
2. Dans l’assistant, cliquez sur **Suivant**.

   ![Exportation du certificat](./media/vpn-gateway-certificates-export-public-key-include/exportwizard.png)
3. Sélectionnez **Non, ne pas exporter la clé privée**, puis cliquez sur **Suivant**.

   ![N’exportez pas la clé privée](./media/vpn-gateway-certificates-export-public-key-include/notprivatekey.png)
4. Sur la page **Format de fichier d’exportation**, sélectionnez **Codé à base 64 X.509 (.cer).** , puis cliquez sur **Suivant**.

   ![Encodage en base 64](./media/vpn-gateway-certificates-export-public-key-include/base64.png)
5. Dans **Fichier à exporter**, cliquez sur **Parcourir** pour accéder à l’emplacement vers lequel vous souhaitez exporter le certificat. Pour la zone **Nom de fichier**, nommez le fichier de certificat. Cliquez ensuite sur **Suivant**.

   ![Capture d’écran montrant l’assistant Exportation de certificat avec une zone de texte de nom de fichier et une option Parcourir.](./media/vpn-gateway-certificates-export-public-key-include/browse.png)
6. Cliquez sur **Terminer** pour exporter le certificat.

   ![Capture d’écran montrant l’Assistant Exportation de certificat affichant les paramètres sélectionnés.](./media/vpn-gateway-certificates-export-public-key-include/finish.png)
7. Votre certificat est correctement exporté.

   ![Capture d’écran montrant un message indiquant que l’exportation a réussi.](./media/vpn-gateway-certificates-export-public-key-include/success.png)
8. Le certificat exporté ressemble à ceci :

   ![Capture d’écran montrant une icône de certificat et un nom de fichier avec l’extension CER.](./media/vpn-gateway-certificates-export-public-key-include/exported.png)
9. Si vous ouvrez le certificat exporté à l’aide du Bloc-notes, vous verrez quelque chose de similaire à cet exemple. La section en bleu contient les informations qui sont chargées sur Azure. Si vous ouvrez votre certificat avec le Bloc-notes et qu’il ne ressemble pas à celui de l’exemple, cela signifie généralement que vous ne l’avez pas exporté au format X.509 de base 64 (.cer). De plus, si vous voulez utiliser un autre éditeur de texte, notez que certains éditeurs peuvent ajouter une mise en forme non souhaitée en arrière-plan. Cela peut créer des problèmes quand vous chargez le texte de ce certificat sur Azure.

   ![Ouverture du fichier avec le Bloc-notes](./media/vpn-gateway-certificates-export-public-key-include/notepad.png)
