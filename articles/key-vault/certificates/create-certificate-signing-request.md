---
title: Création et fusion d’une demande de signature de certificat dans Azure Key Vault
description: Création et fusion d’une demande de signature de certificat dans Azure Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.date: 06/17/2020
ms.author: sebansal
ms.openlocfilehash: 6d66648680aa14baa53372732df52a6c247a0117
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483761"
---
# <a name="creating-and-merging-csr-in-key-vault"></a>Création et fusion d’une demande de signature de certificat dans Key Vault

Azure Key Vault prend en charge le stockage d’un certificat numérique émis par toute autorité de certification de votre choix dans votre coffre de clés. Il prend en charge la création de la demande de signature de certificat avec une paire de clés publique/privée pouvant être signée par toute autorité de certification de votre choix. Il peut s’agir d’une autorité de certification interne de l’entreprise ou d’une autorité de certification publique externe. Une demande de signature de certificat (également appelée CSR ou demande de certification) est un message envoyé par l’utilisateur à une autorité de certification afin de demander l’émission d’un certificat numérique.

Pour plus d’informations générales sur les certificats, consultez [Certificats Azure Key Vault](./about-certificates.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="adding-certificate-in-key-vault-issued-by-partnered-ca"></a>Ajout d’un certificat dans Key Vault émis par une autorité de certification associée
Partenaires Key Vault avec les deux autorités de certification suivantes pour simplifier la création de certificats. 

|Fournisseur|Type de certificat|Configuration  
|--------------|----------------------|------------------|  
|DigiCert|Key Vault propose des certificats SSL OV ou EV avec DigiCert| [Guide d’intégration](./how-to-integrate-certificate-authority.md)
|GlobalSign|Key Vault propose des certificats SSL OV ou EV avec GlobalSign| [Guide d’intégration](https://support.globalsign.com/digital-certificates/digital-certificate-installation/generating-and-importing-certificate-microsoft-azure-key-vault)

## <a name="adding-certificate-in-key-vault-issued-by-non-partnered-ca"></a>Ajout d’un certificat dans Key Vault émis par une autorité de certification non associée

Les étapes suivantes vous aideront à créer un certificat à partir d’autorités de certification qui ne sont pas partenaires de Key Vault (par exemple, GoDaddy n’est pas une autorité de certification de coffre de clés approuvée). 


### <a name="azure-powershell"></a>Azure PowerShell



1. Tout d’abord, **créez la stratégie de certificat**. Key Vault n’inscrit pas ni ne renouvelle le certificat issu par l’émetteur au nom de l’utilisateur, puisque l’autorité de certification choisie dans ce scénario n’est pas prise en charge et, donc, IssuerName a la valeur Inconnu.

   ```azurepowershell
   $policy = New-AzKeyVaultCertificatePolicy -SubjectName "CN=www.contosoHRApp.com" -ValidityInMonths 1  -IssuerName Unknown
   ```
    
   > [!NOTE]
   > Si vous utilisez un nom unique relatif qui comporte une virgule (,) dans la valeur, utilisez des guillemets simples et placez la valeur qui contient le caractère spécial entre guillemets doubles. Exemple : `$policy = New-AzKeyVaultCertificatePolicy -SubjectName 'OU="Docs,Contoso",DC=Contoso,CN=www.contosoHRApp.com' -ValidityInMonths 1  -IssuerName Unknown`. Dans cet exemple, la valeur `OU` est **Docs, Contoso**. Ce format fonctionne pour toutes les valeurs qui contiennent une virgule.

2. Créez une **demande de signature de certificat**.

   ```azurepowershell
   $csr = Add-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -CertificatePolicy $policy
   $csr.CertificateSigningRequest
   ```

3. Obtenez la **demande CSR signée par l’autorité de certification**. `$csr.CertificateSigningRequest` correspond à la demande de signature de certificat codée en base4. Vous pouvez prendre cet objet blob et le vider dans le site web de la demande de certificat de l’émetteur. Cette étape varie d’une autorité de certification à l’autre. Le meilleur moyen consiste à rechercher les instructions de votre autorité de certification quant à l’exécution de cette étape. Vous pouvez également utiliser des outils comme certreq ou openssl pour faire signer la demande de certificat et terminer le processus de génération d’un certificat.


4. **Fusionnez la demande signée** dans Key Vault. Une fois que la demande de certificat a été signée par l’émetteur, vous pouvez reprendre le certificat signé pour le fusionner avec la paire de clés publique/privée initiale créée dans Azure Key Vault.

    ```azurepowershell-interactive
    Import-AzKeyVaultCertificate -VaultName ContosoKV -Name ContosoManualCSRCertificate -FilePath C:\test\OutputCertificateFile.cer
    ```

    La demande de certificat est maintenant correctement fusionnée.

### <a name="azure-portal"></a>Portail Azure

1.  Pour générer la CSR pour l’autorité de certification de votre choix, accédez au coffre de clés auquel vous voulez ajouter le certificat.
2.  Dans les pages des propriétés Key Vault, sélectionnez **Certificats**.
3.  Sélectionnez l’onglet **Générer/Importer**.
4.  Dans l’écran **Créer un certificat**, choisissez les valeurs suivantes :
    - **Méthode de création de certificat :** Générer.
    - **Nom du certificat :** ContosoManualCSRCertificate.
    - **Type d’autorité de certification :** Certificat délivré par une autorité de certification non intégrée
    - **Objet :** `"CN=www.contosoHRApp.com"`
    - Sélectionnez les autres valeurs si besoin. Cliquez sur **Créer**.

    ![Propriétés du certificat](../media/certificates/create-csr-merge-csr/create-certificate.png)  


6.  Vous verrez que le certificat est maintenant ajouté à la liste des certificats. Sélectionnez ce nouveau certificat que vous venez de créer. L’état actuel du certificat est « désactivé », car il n’a pas encore été émis par l’autorité de certification.
7. Cliquez sur l’onglet **Opération de certificat** et sélectionnez **Télécharger CSR**.

   ![Capture d’écran mettant en évidence le bouton Télécharger CSR.](../media/certificates/create-csr-merge-csr/download-csr.png)
 
8.  Présentez le fichier .csr à l’autorité de certification pour faire signer la demande.
9.  Une fois que la demande est signée par l’autorité de certification, reprenez le fichier de certificat pour **fusionner la demande signée** dans le même écran d’opération de certificat.

La demande de certificat est maintenant correctement fusionnée.

> [!NOTE]
> Si vos valeurs de nom unique relatif comportent des virgules, vous pouvez également les ajouter au champ **Objet** en les mettant entre guillemets doubles, comme indiqué à l’étape 4.
> Exemple d’entrée pour « Objet » : `DC=Contoso,OU="Docs,Contoso",CN=www.contosoHRApp.com` Dans cet exemple, le nom unique relatif `OU` contient une valeur qui comporte une virgule dans son nom. La sortie obtenue pour `OU` est **Docs, Contoso**.


## <a name="adding-more-information-to-csr"></a>Ajout d’informations supplémentaires à la CSR

Si vous voulez ajouter des informations supplémentaires lors de la création de la CSR, par exemple : 
    - Pays :
    - Ville / Localité :
    - Département / Province :
    - Organisation :
    - Unité d’organisation : Vous pouvez ajouter toutes ces informations lors de la création d’une CSR en les définissant dans subjectName.

Exemple
    ```SubjectName="CN = docs.microsoft.com, OU = Microsoft Corporation, O = Microsoft Corporation, L = Redmond, S = WA, C = US"
    ```

> [!NOTE]
> Si vous demandez un certificat DV avec tous ces détails dans la CSR, l’autorité de certification peut rejeter la demande, car elle peut ne pas être en mesure de valider toutes les informations dans la demande. Si vous demandez un certificat OV, il est plus approprié d’ajouter toutes ces informations dans la CSR.


## <a name="troubleshoot"></a>Dépanner

- **Type d’erreur « La clé publique du certificat d’entité finale dans le contenu du certificat X.509 spécifié ne correspond pas à la partie publique de la clé privée spécifiée. Vérifiez que le certificat est valide. »** Cette erreur peut se produire si vous ne fusionnez pas la demande de signature de certificat avec celle qui a été lancée. Chaque fois qu’une demande de signature de certificat est créée, elle crée une clé privée qui doit être mise en correspondance lors de la fusion de la demande signée.
    
- Quand la CSR est fusionnée, est-ce que la chaîne entière est fusionnée ?
    Oui, l’ensemble de la chaîne sera fusionné, à condition que l’utilisateur ait remis le fichier p7b à fusionner.

- Si le certificat émis est à l’état « désactivé » dans le portail Azure, examinez le message d’erreur pour ce certificat dans **Opération de certificat**.

Pour plus d’informations, voir [Informations de référence sur les opérations liées aux certificats dans l’API REST Key Vault](/rest/api/keyvault). Pour plus d’informations sur l’établissement d’autorisations, consultez [Coffres : créer ou mettre à jour](/rest/api/keyvault/vaults/createorupdate) et [Coffres : mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy).

- **Type d’erreur « Le nom d’objet fourni n’est pas un nom X500 valide. »** Cette erreur peut se produire si vous avez inclus des « caractères spéciaux » dans les valeurs de SubjectName. Consultez les notes dans les instructions du portail Azure et de PowerShell, respectivement. 

## <a name="next-steps"></a>Étapes suivantes

- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
