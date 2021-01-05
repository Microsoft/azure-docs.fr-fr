---
title: Intégration de Key Vault à l’autorité de certification DigiCert
description: Guide pratique pour intégrer Key Vault à l’autorité de certification DigiCert
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 06/02/2020
ms.author: sebansal
ms.openlocfilehash: 4635bcc51d2ab626b16ddcf02e772bf3df3cad4d
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763738"
---
# <a name="integrating-key-vault-with-digicert-certificate-authority"></a>Intégration de Key Vault à l’autorité de certification DigiCert

Azure Key Vault vous permet de provisionner, gérer et déployer facilement des certificats numériques pour votre réseau et de sécuriser les communications pour les applications. Un certificat numérique correspond à des informations d’identification électroniques qui visent à établir une preuve d’identité dans une transaction électronique. 

Les utilisateurs d’Azure Key Vault peuvent générer des certificats DigiCert directement à partir de leur coffre de clé. Key Vault assure une gestion du cycle de vie des certificats de bout en bout pour les certificats émis par DigiCert à la faveur du partenariat de confiance établi entre Key Vault et l’autorité de certification DigiCert.

Pour plus d’informations générales sur les certificats, consultez [Certificats Azure Key Vault](./about-certificates.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide, vous avez besoin des ressources suivantes.
* Un coffre de clés. Vous pouvez utiliser un coffre de clés existant ou en créer un nouveau en suivant les étapes d’un des guides de démarrage rapide suivants :
   - [Créer un coffre de clés avec Azure CLI](../general/quick-create-cli.md)
   - [Créer un coffre de clés avec Azure PowerShell](../general/quick-create-powershell.md)
   - [Créer un coffre de clés avec le portail Azure](../general/quick-create-portal.md).
*   Vous devez activer le compte DigiCert CertCentral. [Inscrivez-vous](https://www.digicert.com/account/signup/) pour obtenir un compte CertCentral.
*   Des autorisations de niveau administrateur dans vos comptes.


### <a name="before-you-begin"></a>Avant de commencer

Veillez à garder les informations suivantes à portée de main pour votre compte DigiCert CertCentral :
-   ID de compte CertCentral
-   ID d’organisation
-   Clé API

## <a name="adding-certificate-authority-in-key-vault"></a>Ajout d’une autorité de certification dans Key Vault 
Après avoir récupéré les informations ci-dessus du compte DigiCert CertCentral, vous pouvez ajouter DigiCert à la liste des autorités de certification du coffre de clés.

### <a name="azure-portal"></a>Portail Azure

1.  Pour ajouter l’autorité de certification DigiCert, accédez au coffre de clés auquel vous voulez ajouter DigiCert. 
2.  Dans les pages des propriétés Key Vault, sélectionnez **Certificats**.
3.  Sélectionnez l’onglet **Autorités de certification**. ![sélectionner Autorités de certification](../media/certificates/how-to-integrate-certificate-authority/select-certificate-authorities.png)
4.  Sélectionnez l’option **Ajouter**.
 ![ajouter des autorités de certification](../media/certificates/how-to-integrate-certificate-authority/add-certificate-authority.png)
5.  Dans l’écran **Créer une autorité de certification**, choisissez les valeurs suivantes :
    -   **Name** : ajoutez un nom d’émetteur identifiable. Par exemple : DigicertCA
    -   **Fournisseur** : sélectionnez DigiCert dans le menu.
    -   **Account ID** (ID de compte) : entrez votre ID de compte DigiCert CertCentral.
    -   **Mot de passe du compte** : entrez la clé d’API que vous avez générée dans votre compte DigiCert CertCentral.
    -   **ID d’organisation** : entrez l’OrgID récupéré à partir du compte DigiCert CertCentral. 
    -   Cliquez sur **Créer**.
   
6.  Vous pouvez alors observer que DigicertCA a bien été ajouté à la liste des autorités de certification.


### <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell vous permet de créer et gérer des ressources Azure à l’aide de commandes ou de scripts. Azure héberge Azure Cloud Shell, environnement d’interpréteur de commandes interactif que vous pouvez utiliser via le portail Azure dans le navigateur proprement dit.

Si vous choisissez d’installer et d’utiliser PowerShell en local, ce tutoriel nécessite le module Azure PowerShell version 1.0.0 ou ultérieure. Tapez `$PSVersionTable.PSVersion` pour connaître la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Login-AzAccount` pour créer une connexion avec Azure.

```azurepowershell-interactive
Login-AzAccount
```

1.  Créer un **groupe de ressources**

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. 

```azurepowershell-interactive
New-AzResourceGroup -Name ContosoResourceGroup -Location EastUS
```

2. Créer un **coffre de clés**

Vous devez utiliser un nom unique pour votre coffre de clés. Ici, « Contoso-Vaultname » est le nom du coffre de clés utilisé dans ce guide.

- **Nom du coffre** Contoso-Vaultname.
- **Nom du groupe de ressources** ContosoResourceGroup.
- **Emplacement** USA Est.

```azurepowershell-interactive
New-AzKeyVault -Name 'Contoso-Vaultname' -ResourceGroupName 'ContosoResourceGroup' -Location 'EastUS'
```

3. Définissez des variables pour les informations récupérées à partir du compte DigiCert CertCentral.

- Définissez la variable **Account ID** (ID de compte).
- Définissez la variable **Org ID** (ID d’organisation).
- Définissez la variable **API Key** (Clé d’API).

```azurepowershell-interactive
$accountId = "myDigiCertCertCentralAccountID"
$org = New-AzKeyVaultCertificateOrganizationDetail -Id OrganizationIDfromDigiCertAccount
$secureApiKey = ConvertTo-SecureString DigiCertCertCentralAPIKey -AsPlainText –Force
```

4. Définissez l’émetteur (**Issuer**). Digicert est alors ajouté en tant qu’autorité de certification dans le coffre de clés. Pour en savoir plus sur les paramètres [consultez les informations disponibles ici](/powershell/module/az.keyvault/Set-AzKeyVaultCertificateIssuer).
```azurepowershell-interactive
Set-AzKeyVaultCertificateIssuer -VaultName "Contoso-Vaultname" -Name "TestIssuer01" -IssuerProvider DigiCert -AccountId $accountId -ApiKey $secureApiKey -OrganizationDetails $org -PassThru
```

5. **Définissez une stratégie pour le certificat et émettez le certificat** dans le coffre de clés directement à partir de DigiCert.

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "TestIssuer01" -ValidityInMonths 12 -RenewAtNumberOfDaysBeforeExpiry 60
Add-AzKeyVaultCertificate -VaultName "Contoso-Vaultname" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

Du fait de cette intégration, le certificat est maintenant correctement émis par l’autorité de certification DigiCert dans le coffre de clés spécifié.

## <a name="troubleshoot"></a>Dépanner

Si le certificat émis est à l’état « désactivé » sur le portail Azure, examinez le message d’erreur DigiCert pour ce certificat dans **Opération de certificat** .

 ![Opération de certificat](../media/certificates/how-to-integrate-certificate-authority/certificate-operation-select.png)

Message d’erreur « Veuillez effectuer une fusion pour exécuter cette demande de certificat. »
Vous devez fusionner la demande de signature de certificat signée par l’autorité de certification pour effectuer cette demande. Apprenez-en davantage [ici](https://docs.microsoft.com/azure/key-vault/certificates/create-certificate-signing-request).

Pour plus d’informations, voir [Informations de référence sur les opérations liées aux certificats dans l’API REST Key Vault](/rest/api/keyvault). Pour plus d’informations sur l’établissement d’autorisations, consultez [Coffres : créer ou mettre à jour](/rest/api/keyvault/vaults/createorupdate) et [Coffres : mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="frequently-asked-questions"></a>Forum aux questions

- Puis-je générer un certificat avec caractères génériques DigiCert par le biais de KeyVault ? 
   Oui. Cela dépend de la façon dont vous avez configuré votre compte DigiCert.
- Comment créer un certificat **OV-SSL ou EV-SSL** avec DigiCert ? 
   Key Vault prend en charge la création de certificats SSL OV et EV. Quand vous créez un certificat, cliquez sur Configuration de stratégie avancée, puis spécifiez le type de certificat. Les valeurs prises en charge sont les suivantes : OV-SSL, EV-SSL
   
   Vous pouvez créer ce type de certificat dans un coffre de clés si votre compte DigiCert le permet. Pour ce type de certificat, la validation est effectuée par DigiCert, et son équipe de support peut vous accompagner au mieux avec la solution en cas d’échec de la validation. Lors de la création d’un certificat, vous pouvez ajouter des informations supplémentaires en les définissant dans subjectName.

Exemple
    ```SubjectName="CN = docs.microsoft.com, OU = Microsoft Corporation, O = Microsoft Corporation, L = Redmond, S = WA, C = US"
    ```
   
- Y a-t-il un délai dans la création d’un certificat DigiCert à l’aide de l’intégration par rapport à l’acquisition d’un certificat directement par le biais de DigiCert ?
   Non. Lors de la création d’un certificat, c’est le processus de vérification qui peut prendre du temps, et cette vérification dépend du processus suivi par DigiCert.


## <a name="next-steps"></a>Étapes suivantes

- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
