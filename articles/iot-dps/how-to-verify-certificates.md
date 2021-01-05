---
title: Vérifier des certificats d’autorité de certification X.509 auprès du Service IoT Hub Device Provisioning
description: Guide pratique visant à effectuer une preuve de possession pour les certificats d’autorité de certification X.509 avec Azure IoT Hub Device Provisioning (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 02/26/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: 50e8dfd2998b50bfff7341e49ac4d0770c115166
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90530860"
---
# <a name="how-to-do-proof-of-possession-for-x509-ca-certificates-with-your-device-provisioning-service"></a>Effectuer une preuve de possession pour les certificats d’autorité de certification X.509 avec votre service Device Provisioning

Un certificat d’autorité de certification X.509 vérifié est un certificat d’autorité de certification qui a été chargé et inscrit auprès de votre service d’approvisionnement, et a été soumis à une preuve de possession par le service. 

La preuve de possession implique les étapes suivantes :
1. Obtenir un code de vérification unique généré par le service d’approvisionnement pour votre certificat d’autorité de certification X.509. Vous pouvez effectuer cette opération à partir du portail Azure.
2. Créer un certificat de vérification X.509 avec le code de vérification en tant que sujet et signer le certificat avec la clé privée associée à votre certificat d’autorité de certification X.509.
3. Charger le certificat de vérification signé dans le service. Le service valide le certificat de vérification à l’aide de la partie publique du certificat d’autorité de certification à vérifier, prouvant ainsi que vous êtes en possession de la clé privée du certificat de l’autorité de certification.

Les certificats vérifiés jouent un rôle important lors de l’utilisation de groupes d’inscription. La vérification de la propriété du certificat fournit un niveau de sécurité supplémentaire en permettant de s’assurer que l’utilisateur qui a chargé le certificat est en possession de la clé privée associée. La vérification empêche tout acteur malveillant qui intercepte votre trafic d’extraire un certificat intermédiaire et de l’utiliser pour créer un groupe d’inscription dans son propre service d’approvisionnement à des fins de détournement de vos appareils. En prouvant que vous êtes le propriétaire du certificat racine ou d’un certificat intermédiaire dans une chaîne d’approbation, vous prouvez que vous êtes autorisé à générer des certificats feuilles pour les appareils qui seront inscrits dans le cadre de ce groupe d’inscription. Par conséquent, le certificat racine ou intermédiaire configuré dans un groupe d’inscription doit être un certificat vérifié ou être associé à un certificat vérifié dans la chaîne d’approbation présentée par un appareil lors de son authentification auprès du service. Pour en savoir plus sur l’attestation de certificat X.509, consultez [Certificats X.509](concepts-x509-attestation.md) et [Contrôle de l’accès des appareils au service de provisionnement avec des certificats X.509](concepts-x509-attestation.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates).

## <a name="register-the-public-part-of-an-x509-certificate-and-get-a-verification-code"></a>Inscrire la partie publique d’un certificat X.509 et obtenir un code de vérification

Pour inscrire un certificat d’autorité de certification auprès de votre service d’approvisionnement et obtenir un code de vérification que vous pouvez utiliser lors de la preuve de possession, procédez comme suit. 

1. Dans le portail Azure, accédez à votre service d’approvisionnement et ouvrez **Certificats** dans le menu de gauche. 
2. Cliquez sur **Ajouter** pour ajouter un nouveau certificat.
3. Entrez un nom d’affichage convivial pour votre certificat. Accédez au fichier .cer ou .pem représentant la partie publique de votre certificat X.509. Cliquez sur **Télécharger**.
4. Une fois que vous avez obtenu une notification vous informant que votre certificat a été correctement chargé, cliquez sur **Enregistrer**.

    ![Téléchargement d’un certificat](./media/how-to-verify-certificates/add-new-cert.png)  

   Votre certificat apparaît maintenant dans la liste **Explorateur de certificats**. Notez que la colonne **ÉTAT** de ce certificat présente la valeur *Non vérifié*.

5. Cliquez sur le certificat que vous avez ajouté à l’étape précédente.

6. Dans **Détails du certificat**, cliquez sur **Générer le code de vérification**.

7. Le service d’approvisionnement crée un **code de vérification** que vous pouvez utiliser pour valider la propriété du certificat. Copiez ce code dans le Presse-papiers. 

   ![Vérifier le certificat](./media/how-to-verify-certificates/verify-cert.png)  

## <a name="digitally-sign-the-verification-code-to-create-a-verification-certificate"></a>Signer numériquement le code de vérification pour créer un certificat de vérification

Vous devez à présent signer ce *code de vérification* avec la clé privée associée à votre certificat d’autorité de certification X.509, ce qui génère une signature. Appelée [preuve de possession](https://tools.ietf.org/html/rfc5280#section-3.1), cette opération permet d’obtenir un certificat de vérification signé.

Microsoft propose des outils et des exemples conçus pour simplifier la création d’un certificat de vérification signé : 

- Le **Kit de développement logiciel (SDK) C Azure IoT Hub** fournit des scripts PowerShell (Windows) et Bash (Linux) pour vous aider à créer des certificats d’autorité de certification et des certificats feuilles pour le développement, et à effectuer la preuve de possession à l’aide d’un code de vérification. Vous pouvez télécharger les [fichiers](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates) appropriés pour votre système dans un dossier de travail et suivre les instructions du [fichier readme de l’exemple de gestion de certificats d’autorité de certification](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) pour effectuer la preuve de possession sur un certificat d’autorité de certification. 
- Le **Kit de développement logiciel (SDK) C# Azure IoT Hub** contient l’[exemple de vérification de certificat de groupe](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/service/GroupCertificateVerificationSample), que vous pouvez utiliser pour effectuer la preuve de possession.
 
> [!IMPORTANT]
> En plus d’effectuer la preuve de possession, les scripts PowerShell et Bash mentionnés précédemment permettent de créer des certificats racines, des certificats intermédiaires et des certificats feuilles qui peuvent servir à authentifier et à approvisionner des appareils. Ces certificats doivent être utilisés uniquement pour le développement. Ils ne doivent jamais être utilisés dans un environnement de production. 

Les scripts PowerShell et Bash fournis dans la documentation et les Kits de développement logiciel (SDK) s’appuient sur [OpenSSL](https://www.openssl.org/). Vous pouvez également utiliser OpenSSL ou d’autres outils tiers pour vous aider à effectuer la preuve de possession. Pour plus d’informations sur les outils fournis avec les Kits de développement logiciel (SDK), consultez [Comment utiliser les outils fournis dans les SDK pour simplifier le développement pour l’approvisionnement](how-to-use-sdk-tools.md). 


## <a name="upload-the-signed-verification-certificate"></a>Charger le certificat de vérification signé

1. Chargez la signature obtenue en tant que certificat de vérification dans votre service d’approvisionnement dans le portail. Dans le panneau **Détails du certificat** du portail Azure, utilisez l’icône _Explorateur de fichiers_ en regard du champ **Fichier .pem ou .cer du certificat de vérification** pour charger le certificat de vérification signé à partir de votre système.

2. Une fois le chargement du certificat terminé, cliquez sur **Vérifier**. Dans la liste **Explorateur de certificats**, la colonne **ÉTAT** prend la valeur **_Vérifié_**. Si le panneau ne se met pas à jour automatiquement, cliquez sur **Actualiser**.

   ![Charger la vérification du certificat](./media/how-to-verify-certificates/upload-cert-verification.png)  

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment utiliser le portail pour créer un groupe d’inscription, consultez [Gérer les inscriptions d’appareils avec le portail Azure](how-to-manage-enrollments.md).
- Pour savoir comment utiliser les Kits de développement logiciel (SDK) pour créer un groupe d’inscription, consultez [Gérer les inscriptions d’appareils avec les Kits de développement logiciel (SDK) Service](how-to-manage-enrollments-sdks.md).










