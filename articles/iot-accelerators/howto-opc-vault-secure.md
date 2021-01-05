---
title: Sécuriser la communication d’appareils OPC UA avec OPC Vault - Azure | Microsoft Docs
description: Guide pratique pour inscrire des applications OPC UA et émettre des certificats d’application signés pour vos appareils OPC UA avec OPC Vault.
author: mregen
ms.author: mregen
ms.date: 8/16/2018
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: a1b7564988c8a4d63a37b53d18ed3a7359e65d72
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92926408"
---
# <a name="use-the-opc-vault-certificate-management-service"></a>Utiliser le service de gestion de certificats OPC Vault

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article explique comment inscrire des applications et émettre des certificats d’application signés pour vos appareils OPC UA.

## <a name="prerequisites"></a>Prérequis

### <a name="deploy-the-certificate-management-service"></a>Déployer le service de gestion de certificats

Tout d’abord, déployez le service dans le cloud Azure. Pour plus d’informations, consultez [Déployer le service de gestion de certificats OPC Vault](howto-opc-vault-deploy.md).

### <a name="create-the-issuer-ca-certificate"></a>Créer le certificat d’autorité de certification d’émetteur

Si vous ne l’avez pas encore fait, créez le certificat de l’autorité de certification émettrice. Pour plus d’informations, consultez [Créer et gérer le certificat émetteur pour OPC Vault](howto-opc-vault-manage.md).

## <a name="secure-opc-ua-applications"></a>Sécuriser des applications OPC UA

### <a name="step-1-register-your-opc-ua-application"></a>Étape 1 : Inscrire votre application OPC UA 

> [!IMPORTANT]
> Le rôle Rédacteur est nécessaire pour inscrire une application.

1. Ouvrez votre service de certificats à l’adresse `https://myResourceGroup-app.azurewebsites.net`, puis connectez-vous.
2. Accédez à **Register New** (Inscrire nouveau). Pour un inscrire une application, un utilisateur doit disposer au moins du rôle Rédacteur.
2. Le formulaire d’entrée suit les conventions de nommage d’OPC UA. Par exemple, dans la capture d’écran suivante, les paramètres de l’exemple [Serveur de référence OPC UA](https://github.com/OPCFoundation/UA-.NETStandard/tree/master/Applications/ReferenceServer) de la pile .NET Standard sont présentés :

   ![Capture d’écran de l’inscription de serveur de référence UA](media/howto-opc-vault-secure/reference-server-registration.png "Inscription de serveur de référence UA")

5. Sélectionnez **Register** pour inscrire l’application dans la base de données d’application de service de certificats. Le workflow guide l’utilisateur directement vers l’étape suivante destinée à demander un certificat signé pour l’application.

### <a name="step-2-secure-your-application-with-a-ca-signed-application-certificate"></a>Étape 2 : Sécuriser votre application avec un certificat d’application signé par une autorité de certification

Sécurisez votre application OPC UA en émettant un certificat signé basé sur une demande de signature de certificat. Vous pouvez aussi demander une nouvelle paire de clés, qui comprend une nouvelle clé privée au format PFX ou PEM. Pour savoir quelle méthode est prise en charge pour votre application, consultez la documentation de votre appareil OPC UA. En général, la méthode CSR est recommandée car elle ne nécessite pas le transfert d’une clé privée sur un réseau.

#### <a name="request-a-new-certificate-with-a-new-keypair"></a>Demander un nouveau certificat avec une nouvelle paire de clés

1. Accédez à **Applications**.
3. Sélectionnez **New Request** (Nouvelle demande) pour une application listée.

   ![Capture d’écran d’une demande de nouveau certificat](media/howto-opc-vault-secure/request-new-certificate.png "Demander un nouveau certificat")

3. Sélectionnez **Request new KeyPair and Certificate** (Demander une nouvelle paire de clés et un nouveau certificat) pour demander une clé privée et un nouveau certificat signé avec la clé publique pour votre application.

   ![Capture d’écran de génération d’une nouvelle paire de clés et d’un nouveau certificat](media/howto-opc-vault-secure/generate-new-key-pair.png "Générer une nouvelle paire de clés")

4. Complétez le formulaire avec un sujet et les noms de domaine. Pour la clé privée, choisissez PEM ou PFX avec mot de passe. Sélectionnez **Generate New KeyPair** (Générer une nouvelle paire de clés) pour créer la demande de certificat.

   ![Capture d’écran de l’écran Afficher les détails de la demande de certificat et du bouton Générer une nouvelle paire de clés](media/howto-opc-vault-secure/approve-reject.png "Approuver un certificat")

5. L’approbation exige que l’utilisateur possède le rôle Approbateur et des autorisations de signature dans Azure Key Vault. Dans le workflow classique, les rôles Approbateur et Demandeur doivent être attribués à des utilisateurs distincts. Sélectionnez **Approve** (Approuver) ou **Reject** (Rejeter) pour lancer ou annuler la création effective de la paire de clés et l’opération de signature. La nouvelle paire de clés est créée et stockée de manière sécurisé dans Azure Key Vault jusqu’à ce qu’elle soit téléchargée par le demandeur de certificat. Le certificat obtenu avec la clé publique est signé par l’autorité de certification. Ces opérations peuvent prendre quelques secondes.

   ![Capture d’écran d’affichage des détails d’une demande de certificat, avec message d’approbation au bas](media/howto-opc-vault-secure/view-key-pair.png "Afficher une paire de clés")

7. La clé privée (PFX ou PEM) et le certificat (DER) obtenus peuvent être téléchargés à partir de là au format sélectionné en tant que téléchargement de fichiers binaires. Une version encodée en base64 est aussi disponible, par exemple, pour copier et coller le certificat dans une ligne de commande ou une entrée de texte. 
8. Après avoir téléchargé et stocké de manière sécurisée la clé privée, vous pouvez sélectionner **Delete Private Key** (Supprimer la clé privée). Le certificat avec la clé publique reste disponible pour une utilisation ultérieure.
9. En raison de l’utilisation d’un certificat signé par une autorité de certification, le certificat et la liste de révocation de certificats de l’autorité de certification doivent également être téléchargés ici.

À présent, la façon dont la nouvelle paire de clés est appliquée dépend de l’appareil OPC UA. En général, le certificat et la liste de révocation de certificats de l’autorité de certification sont copiés dans un dossier `trusted`, tandis que la clé publique et la clé privée du certificat d’application sont appliquées à un dossier `own` dans le magasin de certificats. Certains appareils peuvent déjà prendre en charge la transmission (de type push) par le serveur pour les mises à jour des certificats. Reportez-vous à la documentation de votre appareil OPC UA.

#### <a name="request-a-new-certificate-with-a-csr"></a>Demander un nouveau certificat avec une demande de signature de certificat (CSR) 

1. Accédez à **Applications**.
3. Sélectionnez **New Request** (Nouvelle demande) pour une application listée.

   ![Capture d’écran d’une demande de nouveau certificat](media/howto-opc-vault-secure/request-new-certificate.png "Demander un nouveau certificat")

3. Sélectionnez **Request new Certificate with Signing Request** (Demander un nouveau certificat avec demande de signature) pour demander un nouveau certificat signé pour votre application.

   ![Capture d’écran de génération d’un nouveau certificat](media/howto-opc-vault-secure/generate-new-certificate.png "Générer un nouveau certificat")

4. Chargez la demande de signature de certificat (CSR) en sélectionnant un fichier local ou en collant une demande de signature de certificat encodée en base64 dans le formulaire. Sélectionnez **Generate New Certificate** (Générer un nouveau certificat).

   ![Capture d’écran d’affichage des détails de la demande de certificat](media/howto-opc-vault-secure/approve-reject-csr.png "Approuver une demande de signature de certificat")

5. L’approbation exige que l’utilisateur possède le rôle Approbateur et des autorisations de signature dans Azure Key Vault. Sélectionnez **Approve** (Approuver) ou **Reject** (Rejeter) pour lancer ou annuler la l’opération de signature effective. Le certificat obtenu avec la clé publique est signé par l’autorité de certification. Cette opération peut prendre quelques secondes.

   ![Capture de l’option Afficher les détails de la demande de certificat, comportant un message d’approbation en bas](media/howto-opc-vault-secure/view-cert-csr.png "Afficher un certificat")

6. Le certificat (DER) obtenu peut être téléchargé à partir de là en tant que fichier binaire. Une version encodée en base64 est aussi disponible, par exemple, pour copier et coller le certificat dans une ligne de commande ou une entrée de texte. 
10. Après avoir téléchargé et stocké de manière sécurisée le certificat, vous pouvez sélectionner **Delete Certificate** (Supprimer le certificat).
11. En raison de l’utilisation d’un certificat signé par une autorité de certification, le certificat et la liste de révocation de certificats de l’autorité de certification doivent également être téléchargés ici.

À présent, la façon dont le nouveau certificat est appliqué dépend de l’appareil OPC UA. En général, le certificat et la liste de révocation de certificats de l’autorité de certification sont copiés dans un dossier `trusted`, tandis que le certificat d’application est appliqué à un dossier `own` dans le magasin de certificats. Certains appareils peuvent déjà prendre en charge la transmission (de type push) par le serveur pour les mises à jour des certificats. Reportez-vous à la documentation de votre appareil OPC UA.

### <a name="step-3-device-secured"></a>Étape 3 : Appareil sécurisé

L’appareil OPC UA est maintenant prêt à communiquer avec d’autres appareils OPC UA sécurisés par des certificats signés par l’autorité de certification sans configuration supplémentaire.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment sécuriser des appareils OPC UA, vous pouvez :

> [!div class="nextstepaction"]
> [Exécuter un service de gestion de certificats sécurisé](howto-opc-vault-secure-ca.md)
