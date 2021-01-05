---
title: Questions fréquentes (FAQ) – Importation de certificat Azure Key Vault
description: Obtenez des réponses aux questions fréquemment posées sur l’importation de certificats Azure Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 07/20/2020
ms.author: sebansal
ms.openlocfilehash: d7d34b61e584b63c517b6c0f8af4cb4adcc7fefe
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289510"
---
# <a name="importing-azure-key-vault-certificates-faq"></a>Questions fréquentes (FAQ) sur l’importation de certificats Azure Key Vault

Cet article offre des réponses aux questions fréquemment posées sur l’importation de certificats Azure Key Vault.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-can-i-import-a-certificate-in-azure-key-vault"></a>Comment importer un certificat dans Azure Key Vault ?

Pour une opération d’importation de certificat, Azure Key Vault accepte deux formats de fichier de certificat : PEM et PFX. Bien qu’il existe des fichiers PEM avec uniquement la partie publique, Key Vault requiert et accepte uniquement un fichier PEM ou PFX avec une clé privée. Pour plus d’informations, consultez [Importer un certificat dans Key Vault](./tutorial-import-certificate.md#import-a-certificate-to-key-vault).

### <a name="after-i-import-a-password-protected-certificate-to-key-vault-and-then-download-it-why-cant-i-see-the-password-thats-associated-with-it"></a>Après avoir importé, puis téléchargé, un certificat protégé par mot de passe dans Key Vault, pourquoi ne puis-je pas voir le mot de passe qui lui est associé ?
    
Une fois qu’un certificat est importé et protégé dans Key Vault, son mot de passe associé n’est pas enregistré. Le mot de passe n’est requis qu’une seule fois pendant l’opération d’importation. Cela est fait exprès, mais vous pouvez toujours obtenir le certificat en tant que secret et le convertir de Base64 en PFX en ajoutant le mot de passe via [Azure PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

### <a name="how-can-i-resolve-a-bad-parameter-error-what-are-the-supported-certificate-formats-for-importing-to-key-vault"></a>Comment résoudre le problème « Erreur de paramètre incorrect » ? Quels sont les formats de certificat pris en charge pour l’importation dans Key Vault ?

Quand vous importez un certificat, vous devez vérifier que la clé est incluse dans le fichier. Si vous avez une clé privée stockée séparément dans un autre format, vous devez associer la clé au certificat. Certaines autorités de certification (CA) fournissent des certificats dans d’autres formats. Par conséquent, avant d’importer le certificat, assurez-vous qu’il est au format de fichier PEM ou PFX et que la clé utilise le chiffrement RSA (Rivest–Shamir–Adleman) ou ECC (Elliptic-Curve Cryptography). 

Pour plus d’informations, consultez les [conditions requises des certificats](./certificate-scenarios.md#formats-of-import-we-support) et les [conditions requises relatives aux clés de certificat](../keys/about-keys.md).

###  <a name="can-i-import-a-certificate-by-using-an-arm-template"></a>Puis-je importer un certificat à l’aide d’un modèle ARM ?

Non, il n’est pas possible d’effectuer des opérations liées au certificat à l’aide d’un modèle Azure Resource Manager (ARM). La solution recommandée consiste à utiliser les méthodes d’importation de certificat dans l’API Azure, l’interface Azure CLI ou PowerShell. Si vous disposez d’un certificat existant, vous pouvez l’importer en tant que secret.

### <a name="when-i-import-a-certificate-via-the-azure-portal-i-get-a-something-went-wrong-error-how-can-i-investigate-further"></a>Lorsque j’importe un certificat via le portail Azure, j’obtiens le message « Une erreur s’est produite ». Comment approfondir les investigations ?
    
Pour voir une erreur plus descriptive, importez le fichier de certificat via [Azure CLI](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-import) ou [PowerShell](/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate?view=azurermps-6.13.0).

### <a name="how-can-i-resolve-error-type-access-denied-or-user-is-unauthorized-to-import-certificate"></a>Comment résoudre le problème « Type d’erreur : Accès refusé ou l’utilisateur n’est pas autorisé à importer un certificat » ?
    
L’opération d’importation nécessite que vous accordiez à l’utilisateur des autorisations pour importer le certificat sous les stratégies d’accès. Pour ce faire, accédez à votre coffre de clés, sélectionnez **Stratégies d’accès** > **Ajouter une stratégie d’accès** > **Sélectionner les autorisations de certificat** > **Principal** , recherchez l’utilisateur, puis ajoutez l’adresse e-mail de l’utilisateur. 

Pour plus d’informations sur les stratégies d’accès liées aux certificats, consultez [À propos des certificats Azure Key Vault](./about-certificates.md#certificate-access-control).


### <a name="how-can-i-resolve-error-type-conflict-when-creating-a-certificate"></a>Comment résoudre le problème « Type d’erreur : Conflit lors de la création d’un certificat » ?
    
Le nom de chaque certificat doit être unique. Un certificat portant le même nom peut être dans un état de suppression réversible. En outre, selon la [composition d’un certificat](./about-certificates.md#composition-of-a-certificate), quand un nouveau certificat est créé, il crée un secret adressable portant le même nom. Ainsi, il existe une autre clé ou un autre secret dans le coffre de clés portant le même nom que la clé ou le secret que vous essayez de spécifier pour votre certificat, l’opération de création du certificat échoue et vous devez supprimer cette clé ou ce secret, ou utiliser un autre nom pour votre certificat. 

Pour plus d’informations, consultez [Opération Obtenir un certificat supprimé](/rest/api/keyvault/getdeletedcertificate/getdeletedcertificate).

### <a name="why-am-i-getting-error-type-char-length-is-too-long"></a>Pourquoi le message « Type d’erreur : la longueur des caractères est trop longue » s’affiche-t-il ?
Cette erreur peut se produire pour deux raisons :    
* Le nom d’objet du certificat est limité à 200 caractères.
* Le mot de passe du certificat est limité à 200 caractères.

### <a name="can-i-import-an-expired-certificate-to-azure-key-vault"></a>Puis-je importer un certificat arrivé à expiration dans Azure Key Vault ?
    
Non, les certificats PFX arrivés à expiration ne peuvent pas être importés dans Key Vault.

### <a name="how-can-i-convert-my-certificate-to-the-proper-format"></a>Comment puis-je convertir mon certificat au format approprié ?

Vous pouvez demander à votre autorité de certification de fournir le certificat dans le format requis. Il existe également des outils tiers qui peuvent vous aider à convertir le certificat dans le format approprié.

### <a name="can-i-import-certificates-from-non-partner-cas"></a>Puis-je importer des certificats d’autorités de certification non-partenaires ?
Oui, vous pouvez importer des certificats à partir de n’importe quelle autorité de certification, mais votre coffre de clés ne pourra pas renouveler automatiquement ces certificats. Vous pouvez définir des rappels pour être informé de l’expiration du certificat.

### <a name="if-i-import-a-certificate-from-a-partner-ca-will-the-autorenewal-feature-still-work"></a>Si j’importe un certificat d’une autorité de certification partenaire, la fonctionnalité de renouvellement automatique fonctionnera-t-elle toujours ?
Oui. Une fois le chargement du certificat effectué, veillez à spécifier la rotation automatique dans la stratégie d’émission du certificat. Vos paramètres restent en vigueur jusqu’au prochain cycle ou la publication de la version de certificat suivante.

### <a name="why-cant-i-see-the-app-service-certificate-that-i-imported-to-key-vault"></a>Pourquoi le certificat App Service que j’ai importé dans Key Vault ne s’affiche-t-il pas ? 
Si vous avez correctement importé le certificat, vous devriez être en mesure de le confirmer en accédant au volet **Secrets**.


## <a name="next-steps"></a>Étapes suivantes

- [Certificats Azure Key Vault](./about-certificates.md)