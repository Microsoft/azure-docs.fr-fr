---
title: Utiliser vos propres certificats avec des appareils Azure Data Box/Azure Data Box Heavy
description: Comment utiliser vos propres certificats pour accéder à l’interface utilisateur web locale et au stockage BLOB sur votre appareil Data Box ou Data Box Heavy.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 12/11/2020
ms.author: alkohli
ms.openlocfilehash: deb52c8716f97874beae4accbf6f34f72e20ca04
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516465"
---
# <a name="use-your-own-certificates-with-data-box-and-data-box-heavy-devices"></a>Utiliser vos propres certificats avec des appareils Data Box et Data Box Heavy

Lors du traitement de la commande, des certificats auto-signés sont générés pour l’accès à l’interface utilisateur web locale et au stockage BLOB d’un appareil Data Box ou Data Box Heavy. Si vous préférez communiquer avec votre appareil via un canal approuvé, vous pouvez utiliser vos propres certificats.

Cet article explique comment installer vos propres certificats et comment rétablir les certificats par défaut avant de renvoyer votre appareil au centre de données. Il fournit également un résumé des exigences relatives aux certificats.

## <a name="about-certificates"></a>À propos des certificats

Un certificat fournit un lien entre une **clé publique** et une entité (telle qu’un nom de domaine) qui a été **signée** (vérifiée) par un tiers de confiance (par exemple, une **autorité de certification**).  Un certificat offre un moyen pratique de distribuer des clés de chiffrement publiques approuvées. De cette façon, les certificats garantissent que votre communication est approuvée et que vous envoyez des informations chiffrées au serveur approprié.

Lors de la configuration initiale de votre appareil Data Box, les certificats auto-signés sont générés automatiquement. Si vous le souhaitez, vous pouvez apporter vos propres certificats. Vous devez suivre certaines instructions si vous envisagez d’apporter vos propres certificats.

Sur un appareil Data Box ou Data Box Heavy, deux types de certificats de point de terminaison sont utilisés :

- Certificats du stockage BLOB
- Certificat de l’interface utilisateur locale

### <a name="certificate-requirements"></a>Configuration requise des certificats

Les certificats doivent respecter les critères suivants :

- Le certificat de point de terminaison doit être au format `.pfx` avec une clé privée qui peut être exportée.
- Vous pouvez utiliser un certificat individuel pour chaque point de terminaison, un certificat multidomaine pour plusieurs points de terminaison ou un certificat de point de terminaison générique.
- Les propriétés d’un certificat de point de terminaison sont similaires à celles d’un certificat SSL standard.
- Un certificat correspondant au format DER (extension de nom de fichier `.cer`) est requis sur l’ordinateur client.
- Après avoir chargé le certificat de l’interface utilisateur locale, vous devrez redémarrer le navigateur et effacer le cache. Reportez-vous aux instructions spécifiques à votre navigateur.
- Les certificats doivent être modifiés si le nom de l’appareil ou le nom de domaine DNS est modifié.
- Utilisez le tableau suivant lors de la création de certificats de point de terminaison :

  |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom de sujet |
  |---------|---------|---------|---------|
  |Interface utilisateur locale| `<DeviceName>.<DNSdomain>`|`<DeviceName>.<DNSdomain>`| `mydevice1.microsoftdatabox.com` |
  |Stockage d'objets Blob|`*.blob.<DeviceName>.<DNSdomain>`|`*.blob.< DeviceName>.<DNSdomain>`|`*.blob.mydevice1.microsoftdatabox.com` |
  |Certificat unique à plusieurs SAN|`<DeviceName>.<DNSdomain>`|`<DeviceName>.<DNSdomain>`<br>`*.blob.<DeviceName>.<DNSdomain>`|`mydevice1.microsoftdatabox.com` |

Pour plus d’informations, consultez [Spécifications du certificat](../../articles/databox-online/azure-stack-edge-j-series-certificate-requirements.md).

## <a name="add-certificates-to-device"></a>Ajouter des certificats à l’appareil

Vous pouvez utiliser vos propres certificats pour accéder à l’interface utilisateur web locale et pour accéder au stockage BLOB.

> [!IMPORTANT]
> Si le nom de l’appareil ou le domaine DNS est modifié, de nouveaux certificats doivent être créés. Les certificats clients et les certificats d’appareil doivent ensuite être mis à jour avec les nouveaux nom d’appareil et domaine DNS.

Pour ajouter votre propre certificat à votre appareil, procédez comme suit :

1. Accédez à **Gérer** > **Certificats**.

   **Nom** indique le nom de l’appareil. **Domaine DNS** indique le nom de domaine du serveur DNS.

   Le bas de l’écran affiche les certificats en cours d’utilisation. Pour un nouvel appareil, vous verrez les certificats auto-signés qui ont été générés pendant le traitement de la commande.

   ![Page Certificats d’un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-manage-certs.png)

2. Si vous avez besoin de modifier le **nom** (nom de l’appareil) ou le **domaine DNS** (domaine du serveur DNS pour l’appareil), faites-le maintenant avant d’ajouter le certificat. Ensuite, sélectionnez **Appliquer**.

   Le certificat doit être modifié si le nom de l’appareil ou le nom du domaine DNS est modifié.

   ![Appliquer un nouveau nom d’appareil et de domaine DNS pour un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-device-name-dns.png)

3. Pour ajouter un certificat, sélectionnez **Ajouter un certificat** pour ouvrir le panneau **Ajouter un certificat**. Sélectionnez ensuite le **type de certificat** : **Stockage BLOB** ou **Interface utilisateur web locale**.

   ![Panneau Ajouter des certificats sur la page Certificats d’un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-add-certificate-cert-type.png)

4. Choisissez le fichier du certificat (au format `.pfx`), puis entrez le mot de passe qui a été défini lors de l’exportation du certificat. Ensuite, sélectionnez **Valider et ajouter**.

   ![Paramètres pour l’ajout d’un certificat de point de terminaison BLOB à un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-add-blob-cert.png)

   Une fois que le certificat a été ajouté, l’écran **Certificats** affiche l’empreinte du nouveau certificat. L’état du certificat est **Valide**.

   ![Un nouveau certificat valide a été ajouté](media/data-box-bring-your-own-certificates/certificates-view-new-certificate.png)

5. Pour afficher les détails du certificat, cliquez sur le nom du certificat. Le certificat expirera au bout d’un an.

   ![Afficher les détails du certificat d’un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-cert-details.png)

   <!--If you changed the local web UI certificate, you'll see the following error. This error will go away when you install the new certificate on the client computer.

   ![Error after a new Local web UI certificate is added to a Data Box device](media/data-box-bring-your-own-certificates/certificates-unable-to-communicate-error.png) TEST. RESTORE IF ERROR IS REPRODUCED.-->

6. Si vous avez modifié le certificat pour l’interface utilisateur web locale, vous devez redémarrer le navigateur, puis l’interface utilisateur web locale. Cette étape est nécessaire pour éviter tout problème de cache SSL.

  <!-- TESTING THIS - The communication error should be gone from the **Certificates** screen.-->

7. Installez le nouveau certificat sur l’ordinateur client que vous utilisez pour accéder à l’interface utilisateur web locale. Pour obtenir des instructions, consultez [Importer des certificats sur le client](#import-certificates-to-client) ci-dessous.


## <a name="import-certificates-to-client"></a>Importer des certificats sur le client

Après avoir ajouté un certificat à votre appareil Data Box, vous devez importer le certificat sur l’ordinateur client que vous utilisez pour accéder à l’appareil. Vous allez importer le certificat dans le magasin de l’autorité de certification racine de confiance pour l’ordinateur local.

Pour importer un certificat sur un client Windows, procédez comme suit :

1. Dans Explorateur de fichiers, cliquez avec le bouton droit sur le fichier du certificat (au format `.cer`), puis sélectionnez **Installer le certificat**. Cette action démarre l’Assistant Importation de certificat.

    ![Importation du certificat 1](media/data-box-bring-your-own-certificates/import-cert-01.png)

2. Pour **Emplacement du magasin**, sélectionnez **Ordinateur local**, puis sélectionnez **Suivant**.

    ![Sélectionner l’ordinateur local comme emplacement du magasin dans l’Assistant Importation de certificat](media/data-box-bring-your-own-certificates/import-cert-02.png)

3. Sélectionnez successivement **Placer tous les certificats dans le magasin suivant**, **Autorité de certification racine de confiance**, puis **Suivant**.

   ![Sélectionner le magasin Autorités de certification racines de confiance dans l’Assistant Importation de certificat](media/data-box-bring-your-own-certificates/import-cert-03.png)

4. Passez en revue vos paramètres, puis sélectionnez **Terminer**. Un message vous indique que l’importation a réussi.

   ![Vérifier les paramètres de votre certificat, puis terminer l’Assistant Importation de certificat.](media/data-box-bring-your-own-certificates/import-cert-04.png)

## <a name="revert-to-default-certificates"></a>Rétablir les certificats par défaut

Avant de retourner votre appareil au centre de données Azure, vous devez rétablir les certificats d’origine qui ont été générés lors du traitement de la commande.

Pour rétablir les certificats générés lors du traitement de la commande, procédez comme suit :

1. Accédez à **Gérer** > **Certificats**, puis sélectionnez **Rétablir les certificats**.

   Le rétablissement des certificats revient à utiliser les certificats auto-signés qui ont été générés lors du traitement de la commande. Vos propres certificats sont supprimés de l’appareil.

   ![Option Rétablir les certificats dans Gérer les certificats d’un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-revert-certificates.png)

2. Une fois le rétablissement des certificats terminé, accédez à **Arrêter ou redémarrer**, puis sélectionnez **Redémarrer**. Cette étape est nécessaire pour éviter tout problème de cache SSL.

   ![Arrêter ou redémarrer l’interface utilisateur web locale après avoir rétabli les certificats sur un appareil Data Box](media/data-box-bring-your-own-certificates/certificates-restart-ui.png)

   Patientez quelques minutes, puis reconnectez-vous à l’interface utilisateur web locale.
