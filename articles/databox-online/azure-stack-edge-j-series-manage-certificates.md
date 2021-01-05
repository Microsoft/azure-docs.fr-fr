---
title: Utiliser des certificats avec Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit l’utilisation de certificats avec les appareils Azure Stack Edge Pro avec GPU, y compris la raison pour laquelle les utiliser et les types et la méthode de téléchargement des certificats sur votre appareil.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 09/29/2020
ms.author: alkohli
ms.openlocfilehash: 5cec5eda7cf398949865bf6d3d3e8be5b2d5e840
ms.sourcegitcommit: d9ba60f15aa6eafc3c5ae8d592bacaf21d97a871
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91767366"
---
# <a name="use-certificates-with-azure-stack-edge-pro-gpu-device"></a>Utiliser des certificats avec des appareils Azure Stack Edge Pro avec GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Cet article décrit les types de certificats qui peuvent être installés sur votre appareil Azure Stack Edge Pro. L’article comprend également les détails de chaque type de certificat, ainsi que la procédure d’installation et d’identification de la date d’expiration. 

## <a name="about-certificates"></a>À propos des certificats

Un certificat fournit un lien entre une **clé publique** et une entité (telle qu’un nom de domaine) qui a été **signée** (vérifiée) par un tiers de confiance (par exemple une **autorité de certification**).  Un certificat offre un moyen pratique de distribuer des clés de chiffrement publiques approuvées. Les certificats garantissent que votre communication est approuvée et que vous envoyez des informations chiffrées au serveur approprié. 

Lors de la configuration initiale de votre appareil Azure Stack Edge Pro, les certificats auto-signés sont générés automatiquement. Si vous le souhaitez, vous pouvez apporter vos propres certificats. Vous devez suivre certaines instructions si vous envisagez d’apporter vos propres certificats.

## <a name="types-of-certificates"></a>Types de certificats

Les différents types de certificats utilisés sur votre appareil Azure Stack Edge Pro sont les suivants : 
- Certificats de signature
    - Autorité de certification racine
    - Intermédiaire

- Certificats de point de terminaison
    - Certificats de nœud
    - Certificats de l’interface utilisateur locale
    - Certificats Azure Resource Manager
    - Certificats de stockage d’objets Blob
    - Certificats d’appareils IoT
    <!--- WiFi certificates
    - VPN certificates-->

- Certificats de chiffrement
    - Certificats de session de support

Chacun de ces certificats est décrit en détail dans les sections suivantes.

## <a name="signing-chain-certificates"></a>Certificats de chaîne de signature

Il s’agit des certificats pour l’autorité qui signe les certificats ou l’autorité de certification de signature. 

### <a name="types"></a>Types

Ces certificats peuvent être des certificats racines ou des certificats intermédiaires. Les certificats racines sont toujours auto-signés (ou signés par eux-mêmes). Les certificats intermédiaires ne sont pas auto-signés et sont signés par l’autorité de signature.

### <a name="caveats"></a>Mises en garde

- Les certificats racines doivent être des certificats de chaîne de signature.
- Les certificats racines peuvent être téléchargés sur votre appareil au format suivant : 
    - **DER** : disponibles sous la forme d’une extension de fichier `.cer`.
    - **Encodé en Base64** : également disponibles en tant qu’extension de fichier `.cer`.
    - **P7B** : ce format est utilisé uniquement pour les certificats de chaîne de signature qui incluent les certificats racines et intermédiaires.
- Les certificats de chaîne de signature sont toujours téléchargés avant que vous téléchargiez d’autres certificats.


## <a name="node-certificates"></a>Certificats de nœud

<!--Your Azure Stack Edge Pro device could be a 1-node device or a 4-node device.--> Tous les nœuds de l’appareil communiquent en permanence entre eux et doivent donc avoir une relation d’approbation. Les certificats de nœud permettent d’établir cette approbation. Les certificats de nœud entrent également en jeu lorsque vous vous connectez au nœud de l’appareil à l’aide d’une session PowerShell à distance via https.

### <a name="caveats"></a>Mises en garde

- Le certificat de nœud doit être fourni au format `.pfx` avec une clé privée qui peut être exportée. 
- Vous pouvez créer et charger 1 certificat de nœud générique ou 4 certificats de nœuds individuels. 
- Un certificat de nœud doit être modifié si le domaine DNS change mais que le nom de l’appareil n’est pas modifié. Si vous apportez votre propre certificat de nœud, vous ne pouvez pas modifier le numéro de série de l’appareil, vous pouvez uniquement modifier le nom de domaine.
- Utilisez le tableau suivant pour vous guider lors de la création d’un certificat de nœud.
   
    |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom de sujet |
    |---------|---------|---------|---------|
    |Nœud|`<NodeSerialNo>.<DnsDomain>`|`*.<DnsDomain>`<br><br>`<NodeSerialNo>.<DnsDomain>`|`mydevice1.microsoftdatabox.com` |
   

## <a name="endpoint-certificates"></a>Certificats de point de terminaison

Pour tous les points de terminaison exposés par l’appareil, un certificat est requis pour la communication approuvée. Les certificats de point de terminaison incluent ceux qui sont requis lors de l’accès à Azure Resource Manager et au stockage d’objets Blob via les API REST. 

Lorsque vous apportez votre propre certificat signé, vous avez également besoin de la chaîne de signature correspondante du certificat. Pour la chaîne de signature, Azure Resource Manager et les certificats Blob sur l’appareil, vous aurez besoin des certificats correspondants sur l’ordinateur client pour vous authentifier et communiquer avec l’appareil.

### <a name="caveats"></a>Mises en garde

- Les certificats de point de terminaison doivent être au format `.pfx` avec une clé privée. La chaîne de signature doit être au format DER (extension de fichier `.cer`). 
- Lorsque vous apportez vos propres certificats de point de terminaison, il peut s’agir de certificats individuels ou de certificats multidomaines. 
- Si vous importez une chaîne de signature, le certificat de la chaîne de signature doit être téléchargé avant de télécharger un certificat de point de terminaison.
- Ces certificats doivent être modifiés si le nom de l’appareil ou les noms de domaine DNS sont modifiés.
- Vous pouvez utiliser un certificat de point de terminaison générique.
- Les propriétés des certificats de point de terminaison sont similaires à celles d’un certificat SSL standard. 
- Utilisez le tableau suivant lors de la création d’un certificat de point de terminaison :

    |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom de sujet |
    |---------|---------|---------|---------|
    |Azure Resource Manager|`management.<Device name>.<Dns Domain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`|`management.mydevice1.microsoftdatabox.com` |
    |Stockage d'objets blob|`*.blob.<Device name>.<Dns Domain>`|`*.blob.< Device name>.<Dns Domain>`|`*.blob.mydevice1.microsoftdatabox.com` |
    |Certificat unique à plusieurs SAN pour les deux points de terminaison|`<Device name>.<dnsdomain>`|`<Device name>.<dnsdomain>`<br>`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`<br>`*.blob.<Device name>.<Dns Domain>`|`mydevice1.microsoftdatabox.com` |


## <a name="local-ui-certificates"></a>Certificats de l’interface utilisateur locale

Vous pouvez accéder à l’interface utilisateur web locale de votre appareil via un navigateur. Pour vous assurer que cette communication est sécurisée, vous pouvez télécharger votre propre certificat. 

### <a name="caveats"></a>Mises en garde

- Le certificat de l’interface utilisateur locale est également téléchargé dans un format `.pfx` avec une clé privée qui peut être exportée.
- Après avoir téléchargé le certificat de l’interface utilisateur locale, vous devrez redémarrer le navigateur et effacer le cache. Reportez-vous aux instructions spécifiques à votre navigateur.

    |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom de sujet |
    |---------|---------|---------|---------|
    |Interface utilisateur locale| `<Device name>.<DnsDomain>`|`<Device name>.<DnsDomain>`| `mydevice1.microsoftdatabox.com` |
   

## <a name="iot-edge-device-certificates"></a>Certificats d’appareil IoT Edge

Votre appareil Azure Stack Edge Pro est également un appareil IoT avec le calcul activé par un appareil IoT Edge connecté. Pour permettre une connexion sécurisée entre votre appareil IoT Edge et les appareils en aval qui peuvent s’y connecter, vous pouvez également télécharger des certificats IoT Edge. 

L’appareil dispose de certificats auto-signés qui peuvent être utilisés si vous souhaitez utiliser uniquement le scénario de calcul avec l’appareil. Cependant, si l’appareil Azure Stack Edge Pro est connecté à des appareils en aval, vous devez apporter vos propres certificats.

Il y a trois certificats IoT Edge que vous devez installer pour activer cette relation d’approbation :

- **Autorité de certification racine ou autorité de certification propriétaire**
- **Autorité de certification de l’appareil** 
- **Certificat de clé d’appareil**

### <a name="caveats"></a>Mises en garde

- Les certificats IoT Edge sont téléchargés au format `.pem`. 


Pour plus d’informations sur les certificats de IoT Edge, consultez [Détails des certificats Azure IoT Edge](../iot-edge/iot-edge-certs.md#iot-edge-certificates).

## <a name="support-session-certificates"></a>Certificats de session de support

Si votre appareil Azure Stack Edge Pro rencontre des problèmes, pour résoudre ces derniers, vous pouvez ouvrir une session de support PowerShell distante sur l’appareil. Pour activer une communication sécurisée et chiffrée sur cette session de support, vous pouvez télécharger un certificat.

### <a name="caveats"></a>Mises en garde

- Assurez-vous que le certificat `.pfx` correspondant avec la clé privée est installé sur l’ordinateur client à l’aide de l’outil de déchiffrement.
- Vérifiez que le champ **Utilisation de la clé** pour le certificat n’est pas défini sur **Signature du certificat**. Pour vérifier cela, cliquez avec le bouton droit sur le certificat, choisissez **Ouvrir** et dans l’onglet **Détails**, recherchez **Utilisation de la clé**. 


### <a name="caveats"></a>Mises en garde

- Le certificat de session de support doit être fourni au format DER avec une extension `.cer`.


<!--## VPN certificates

If VPN is configured on your Azure Stack Edge Pro device, then you will also need a certificate for any communication that occurs over the VPN channel. You can bring your own VPN certificate to ensure the communication is trusted.

### Caveats

- The VPN certificate must be uploaded as a pfx format with a  private key.
- The VPN certificate is not dependant on the device name, device serial number, or device configuration. It only requires the external FQDN.
- Make sure that the client OID is set.-->

<!--## WiFi certificates

If your device is configured to operate on a wireless network, then you will also need a WiFi certificate for any communication that occurs over the wireless network. 

### Caveats

- The WiFi certificate must be uploaded as a pfx format with a private key.
- Make sure that the client OID is set.-->


## <a name="create-certificates-optional"></a>Créer des certificats (facultatif)

La section suivante décrit la procédure de création de la chaîne de signature et des certificats de point de terminaison.

### <a name="certificate-workflow"></a>Workflow de certificat

Vous disposerez d’une méthode définie pour créer les certificats pour les appareils qui fonctionnent dans votre environnement. Vous pouvez utiliser les certificats qui vous sont fournis par votre administrateur informatique. 

**À des fins de développement ou de test uniquement, vous pouvez également utiliser Windows PowerShell pour créer des certificats sur votre système local.** Lors de la création des certificats pour le client, suivez ces instructions :

1. Vous pouvez créer un des types de certificats suivants :

    - Créez un certificat valide unique pour une utilisation avec un nom de domaine complet (FQDN) unique. Par exemple *mydomain.com*.
    - Créez un certificat générique pour sécuriser le nom de domaine principal ainsi que plusieurs sous-domaines. Par exemple * *.mydomain.com*.
    - Créez un certificat SAN (autre nom de l’objet) qui couvre plusieurs noms de domaine dans un seul certificat. 

2. Si vous apportez votre propre certificat, vous aurez besoin d’un certificat racine pour la chaîne de signature. Consultez les étapes pour [Créer des certificats de chaîne de signature](#create-signing-chain-certificate).

3. Vous pouvez ensuite créer les certificats de point de terminaison pour l’interface utilisateur locale de l’appliance, l’objet Blob et Azure Resource Manager. Vous pouvez créer 3 certificats distincts pour l’appliance, l’objet Blob et Azure Resource Manager, ou vous pouvez créer un seul certificat pour les 3 points de terminaison. Pour obtenir des instructions détaillées, consultez [Créer des certificats de signature et de point de terminaison](#create-signed-endpoint-certificates).

4. Que créiez 3 certificats séparés ou un seul certificat, spécifiez les noms d’objet (SN) et les autres noms de l’objet (SAN) conformément aux instructions fournies pour chaque type de certificat. 

### <a name="create-signing-chain-certificate"></a>Créer un certificat de chaîne de signature

Créez ces certificats via Windows PowerShell exécuté en mode administrateur. **Les certificats créés de cette manière doivent être utilisés à des fins de développement ou de test uniquement.**

Le certificat de chaîne de signature ne doit être créé qu’une seule fois. Les autres certificats de point de terminaison feront référence à ce certificat pour la signature.
 

```azurepowershell
$cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature -Subject "CN=RootCert" -HashAlgorithm sha256 -KeyLength 2048 -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsageProperty Sign -KeyUsage CertSign
```


### <a name="create-signed-endpoint-certificates"></a>Créer des certificats de point de terminaison signés

Créez ces certificats via Windows PowerShell exécuté en mode administrateur.

Dans ces exemples, les certificats des points de terminaison sont créés pour un appareil avec :
    - Nom de l’appareil : `DBE-HWDC1T2`
    - Domaine DNS : `microsoftdatabox.com`

Remplacez avec le nom et le domaine DNS de votre appareil pour créer des certificats pour votre appareil.
 
**Certificat de point de terminaison Blob**

Créez un certificat pour le point de terminaison d’objet Blob dans votre magasin personnel.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "*.blob.$AppName.$domain" -Subject "CN=*.blob.$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificat de point de terminaison Azure Resource Manager**

Créez un certificat pour les points de terminaison Azure Resource Manager dans votre magasin personnel.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "management.$AppName.$domain","login.$AppName.$domain" -Subject "CN=management.$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificat de l’interface utilisateur web locale de l’appareil**

Créez un certificat pour l’interface utilisateur web locale de l’appareil dans votre magasin personnel.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "$AppName.$domain" -Subject "CN=$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificat multi-SAN unique pour tous les points de terminaison**

Créez un seul certificat pour tous les points de terminaison de votre magasin personnel.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"
$DeviceSerial = "HWDC1T2"

New-SelfSignedCertificate -Type Custom -DnsName "$AppName.$domain","$DeviceSerial.$domain","management.$AppName.$domain","login.$AppName.$domain","*.blob.$AppName.$domain" -Subject "CN=$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

Une fois les certificats créés, l’étape suivante consiste à télécharger les certificats sur votre appareil Azure Stack Edge Pro


## <a name="upload-certificates"></a>Charger des certificats 

Les certificats que vous avez créés pour votre appareil résident par défaut dans le **magasin personnel** sur votre client. Ces certificats doivent être exportés sur votre client dans des fichiers au format appropriés qui peuvent ensuite être téléchargés sur votre appareil.

1. Le certificat racine doit être exporté au format DER avec l’extension `.cer`. Pour obtenir des instructions détaillées, consultez [Exporter des certificats au format DER](#export-certificates-as-der-format).
2. Les certificats de point de terminaison doivent être exportés en tant que fichiers *.pfx* avec des clés privées. Pour obtenir des instructions détaillées, consultez [Exporter des certificats en tant que fichiers *.pfx* avec des clés privées](#export-certificates-as-pfx-format-with-private-key). 
3. Les certificats racines et de point de terminaison sont ensuite téléchargés sur l’appareil à l’aide de l’option **+ Ajouter un certificat** dans la page Certificats de l’interface utilisateur web locale. 

    1. Téléchargez d’abord le certificat racine. Dans l’interface utilisateur web locale, accédez à **Certificats > + Ajouter un certificat**.

        ![Ajouter un certificat de chaîne de signature 1](media/azure-stack-edge-series-manage-certificates/add-cert-1.png)

    2. Ensuite, téléchargez les certificats de point de terminaison. 

        ![Ajouter un certificat de chaîne de signature 2](media/azure-stack-edge-series-manage-certificates/add-cert-2.png)

        Choisissez les fichiers de certificat au format *.pfx* et entrez le mot de passe que vous avez fourni lors de l’exportation du certificat. L’application du certificat Azure Resource Manager peut prendre quelques minutes.

        Si la chaîne de signature n’est pas mise à jour d’abord et que vous essayez de charger les certificats de point de terminaison, vous obtiendrez une erreur.

        ![Erreur d’application du certificat](media/azure-stack-edge-series-manage-certificates/apply-cert-error-1.png)

        Revenez en arrière et chargez le certificat de chaîne de signature, puis chargez et appliquez les certificats de point de terminaison.

> [!IMPORTANT]
> Si le nom de l’appareil ou le domaine DNS est modifié, de nouveaux certificats doivent être créés. Les certificats clients et les certificats d’appareil doivent ensuite être mis à jour avec les nouveaux nom d’appareil et domaine DNS. 

## <a name="import-certificates-on-the-client-accessing-the-device"></a>Importer des certificats sur le client qui accède à l’appareil

Les certificats que vous avez créés et téléchargés sur votre appareil doivent être importés sur votre client Windows (qui accède à l’appareil) dans le magasin de certificats approprié.

1. Le certificat racine que vous avez exporté en tant que DER doit maintenant être importé dans les **Autorités de certification racines de confiance** sur votre système client. Pour obtenir des instructions détaillées, consultez [Importer des certificats dans le magasin d’autorités de certification racine approuvées](#import-certificates-as-der-format).

2. Les certificats de point de terminaison que vous avez exportés en tant que `.pfx` doivent être exportés au format DER avec l’extension `.cer`. Ce `.cer` est ensuite importé dans le **magasin de certificats personnel** sur votre système. Pour obtenir des instructions détaillées, consultez [Importer des certificats dans le magasin de certificats personnel](#import-certificates-as-der-format).

### <a name="import-certificates-as-der-format"></a>Importer des certificats au format DER

Pour importer des certificats sur un client Windows, procédez comme suit :

1. Cliquez avec le bouton droit sur le fichier et sélectionnez **Installer le certificat**. Cette action démarre l’Assistant Importation de certificat.

    ![Importation du certificat 1](media/azure-stack-edge-series-manage-certificates/import-cert-1.png)

2. Pour **Emplacement du magasin**, sélectionnez **Ordinateur local**, puis sélectionnez **Suivant**.

    ![Importation du certificat 2](media/azure-stack-edge-series-manage-certificates/import-cert-2.png)

3. Sélectionnez **Placer tous les certificats dans le magasin suivant**, puis sélectionnez **Parcourir**. 

    - Pour effectuer l’importation dans le magasin personnel, accédez au magasin personnel de votre hôte distant, puis sélectionnez **Suivant**.

        ![Importer le certificat 4](media/azure-stack-edge-series-manage-certificates/import-cert-4.png)


    - Pour effectuer l’importation dans le magasin de confiance, accédez à l’autorité de certification racine approuvée, puis sélectionnez **Suivant**.

        ![Importer le certificat 3](media/azure-stack-edge-series-manage-certificates/import-cert-3.png)

 
4. Sélectionnez **Terminer**. Un message indiquant que l’importation a réussi s’affiche.

### <a name="export-certificates-as-pfx-format-with-private-key"></a>Exporter les certificats au format .pfx avec une clé privée

Procédez comme suit pour exporter un certificat SSL avec une clé privée sur un ordinateur Windows. 

> [!IMPORTANT]
> Effectuez ces étapes sur le même ordinateur que celui que vous avez utilisé pour créer le certificat. 

1. Exécutez *certlm.msc* pour lancer le magasin de certificats de l’ordinateur local.

1. Double-cliquez sur le dossier **Personnel**, puis sur **Certificats**.

    ![Exporter le certificat 1](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-1.png)
 
2. Cliquez avec le bouton droit sur le certificat que vous souhaitez sauvegarder, puis choisissez **Toutes les tâches > Exporter...**

    ![Exporter le certificat 2](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-2.png)

3. Suivez l’Assistant d’exportation de certificat pour sauvegarder votre certificat dans un fichier .pfx.

    ![Exporter le certificat 3](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-3.png)

4. Choisissez **Oui, exporter la clé privée**.

    ![Exporter le certificat 4](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-4.png)

5. Choisissez **Inclure tous les certificats dans le chemin d’accès du certificat si possible**, **Exporter toutes les propriétés étendues** et **Activer la confidentialité du certificat**. 

    > [!IMPORTANT]
    > NE sélectionnez PAS l’option **Supprimer la clé privée si l’exportation est réussie**.

    ![Exporter le certificat 5](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-5.png)

6. Entrez un mot de passe dont vous vous souviendrez. Confirmez le mot de passe. Le mot de passe protège la clé privée.

    ![Exporter le certificat 6](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-6.png)

7. Choisissez d’enregistrer le fichier à un emplacement défini.

    ![Exporter le certificat 7](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-7.png)
  
8. Sélectionnez **Terminer**.

    ![Exporter le certificat 8](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-8.png)

9. Vous recevez un message indiquant que l’exportation a réussi. Sélectionnez **OK**.

    ![Exporter le certificat 9](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-9.png)

La sauvegarde du fichier .pfx est maintenant enregistrée à l’emplacement que vous avez sélectionné et est prête à être déplacée ou stockée pour une conservation sûre.


### <a name="export-certificates-as-der-format"></a>Exporter les certificats au format DER

1. Exécutez *certlm.msc* pour lancer le magasin de certificats de l’ordinateur local.

1. Dans le magasin de certificats personnel, sélectionnez le certificat racine. Cliquez avec le bouton droit, puis sélectionnez **All Tasks (Toutes les tâches) -> Exporter...** .

    ![Exporter le certificat DER 1](media/azure-stack-edge-series-manage-certificates/export-cert-cer-1.png)

2. L’assistant de certificat s’ouvre. Sélectionnez le format **X.509 binaire encodé DER (.cer)** . Sélectionnez **Suivant**.

    ![Exporter le certificat DER 2](media/azure-stack-edge-series-manage-certificates/export-cert-cer-2.png)

3. Recherchez et sélectionnez l’emplacement où vous souhaitez exporter le fichier au format .cer.

    ![Exporter le certificat DER 3](media/azure-stack-edge-series-manage-certificates/export-cert-cer-3.png)

4. Sélectionnez **Terminer**.

    ![Exporter le certificat DER 4](media/azure-stack-edge-series-manage-certificates/export-cert-cer-4.png)


## <a name="supported-certificate-algorithms"></a>Algorithmes de certificat pris en charge

 Seuls les certificats RSA (Rivest-Shamir-Adleman) sont pris en charge avec votre appareil Azure Stack Edge Pro. Si des certificats ECDSA (Elliptic Curve Digital Signature Algorithm) sont utilisés, le comportement de l’appareil est indéterminé.

 Les certificats qui contiennent une clé publique RSA sont appelés certificats RSA. Les certificats qui contiennent une clé publique ECC (Elliptic Curve Cryptographic) sont appelés certificats ECDSA (Elliptic Curve Digital Signature Algorithm). 


## <a name="view-certificate-expiry"></a>Afficher l’expiration du certificat

Si vous apportez vos propres certificats, les certificats expirent généralement après 1 an ou 6 mois. Pour afficher la date d’expiration de votre certificat, accédez à la page **Certificats** dans l’interface utilisateur web locale de votre appareil. Si vous sélectionnez un certificat spécifique, vous pouvez afficher la date d’expiration de votre certificat.

<!--## Rotate certificates

Rotation of certificates is not implemented in this release. You are also not notified of the pending expiration date on your certificate. 

View the certificate expiration date on the **Certificates** page in the local web UI of your device. Once the certificate expiration is approaching, create and upload new certificates as per the detailed instructions in [Create and upload certificates](azure-stack-edge-j-series-manage-certificates.md).-->

## <a name="next-steps"></a>Étapes suivantes

[Déployer votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)
