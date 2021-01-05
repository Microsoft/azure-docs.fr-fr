---
title: Prise en main des certificats Key Vault
description: Les scénarios suivants décrivent plusieurs utilisations principales du service de gestion des certificats Key Vault, notamment les étapes supplémentaires requises pour créer votre premier certificat dans le coffre de clés.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 06/13/2020
ms.author: mbaldwin
ms.openlocfilehash: 85f17897c0e3089a2d2bc5b172e98fa24e8085ff
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920438"
---
# <a name="get-started-with-key-vault-certificates"></a>Prise en main des certificats Key Vault
Les scénarios suivants décrivent plusieurs utilisations principales du service de gestion des certificats Key Vault, notamment les étapes supplémentaires requises pour créer votre premier certificat dans le coffre de clés.

Les points suivants sont présentés :
- Création de votre premier certificat Key Vault
- Création d’un certificat auprès d’une autorité de certification associée à Key Vault
- Création d’un certificat auprès d’une autorité de certification non associée à Key Vault
- Importation d’un certificat

## <a name="certificates-are-complex-objects"></a>Les certificats sont des objets complexes
Les certificats sont composés de trois ressources reliées entre elles en tant que certificat Key Vault, métadonnées de certificat, clé et secret.


![Les certificats sont complexes](../media/azure-key-vault.png)


## <a name="creating-your-first-key-vault-certificate"></a>Création de votre premier certificat Key Vault  
 Pour pouvoir créer un certificat dans Key Vault, vous devez avoir effectué correctement les étapes 1 et 2, et un coffre de clés doit exister pour cet utilisateur/cette organisation.  

**Étape 1** : fournisseurs d’autorités de certification  
-   L’embarquement en tant qu’administrateur informatique, administrateur PKI ou toute personne assurant la gestion des comptes auprès des autorités de certification, pour une société donnée (par exemple Contoso) est une condition préalable requise pour utiliser des certificats Key Vault.  
    Les autorités de certification suivantes sont les fournisseurs actuels associés à Key Vault. Apprenez-en davantage [ici](./create-certificate.md#partnered-ca-providers).   
    -   DigiCert : Key Vault propose des certificats TSL/SSL OV avec DigiCert.  
    -   GlobalSign : Key Vault propose des certificats TSL/SSL OV avec GlobalSign.  

**Étape 2** : un administrateur de compte d’un fournisseur d’autorité de certification crée des informations d’identification pouvant être utilisées par Key Vault pour inscrire, renouveler et utiliser des certificats TSL/SSL via Key Vault.

**Étape 3** : un administrateur Contoso, ainsi qu’un employé Contoso (utilisateur Key Vault) qui possède des certificats, dépendant de l’autorité de certification, peuvent obtenir un certificat auprès de l’administrateur ou directement à partir du compte avec l’autorité de certification.  

- Commencez par ajouter des informations d’identification dans un coffre de clés en [configurant une ressource émettrice de certificat](/rest/api/keyvault/setcertificateissuer/setcertificateissuer). L’émetteur de certificat est une entité représentée dans Azure Key Vault (KV) en tant que ressource CertificateIssuer. Il fournit des informations sur la source d’un certificat KV, ainsi que le nom de l’émetteur, le fournisseur, les informations d’identification et d’autres détails administratifs.
  - Ex. MyDigiCertIssuer  
    -   Fournisseur  
    -   Informations d’identification : informations d’identification du compte d’autorité de certification. Chaque autorité de certification possède ses propres données spécifiques.  

    Pour plus d’informations sur la création de comptes avec des fournisseurs d’autorités de certification, consultez le billet associé sur le [blog Key Vault](/archive/blogs/kv/manage-certificates-via-azure-key-vault).  

**Étape 3.1** : configurez un [contact de certificat](/rest/api/keyvault/setcertificatecontacts/setcertificatecontacts) pour les notifications. Il s’agit du contact de l’utilisateur Key Vault. Key Vault n’applique pas cette étape.  

Remarque : cette procédure (jusqu’à la fin de l’étape 3.1) est une opération unique.  

## <a name="creating-a-certificate-with-a-ca-partnered-with-key-vault"></a>Création d’un certificat auprès d’une autorité de certification associée à Key Vault

![Créer un certificat auprès d’une autorité de certification associée à Key Vault](../media/certificate-authority-2.png)

**Étape 4** : les descriptions suivantes correspondent aux étapes numérotées en vert dans le diagramme précédent.  
  (1) : dans le diagramme ci-dessus, votre application crée un certificat qui commence en interne par la création d’une clé dans votre coffre de clés.  
  (2) : Key Vault envoie une demande de certificat TSL/SSL à l’autorité de certification.  
  (3) : votre application interroge, dans un processus boucle et attente, votre instance Key Vault pour savoir si la création du certificat est terminée. La création de certificats est terminée lorsque Key Vault reçoit la réponse de l’autorité de certification avec un certificat X509.  
  (4) : l’autorité de certification répond à la demande de certificat TSL/SSL de Key Vault avec un certificat TSL/SSL X509.  
  (5) : la création de votre nouveau certificat se termine avec la fusion du certificat X509 pour l’autorité de certification.  

  Utilisateur Key Vault : crée un certificat en spécifiant une stratégie.

  -   Répétition au besoin  
  -   Contraintes de stratégie  
      -   Propriétés X509  
      -   Propriétés de la clé  
      -   Référence du fournisseur - > par exemple MyDigiCertIssure  
      -   Informations de renouvellement - > par exemple 90 jours avant l’expiration  

  - La création de certificat est généralement un processus asynchrone. Elle implique l’interrogation de votre coffre de clés pour connaître l’état de l’opération de création d’un certificat.  
[Opération d’obtention du certificat](/rest/api/keyvault/getcertificateoperation/getcertificateoperation)  
      -   État : terminé, en échec avec informations sur l’erreur ou annulé.  
      -   Une opération d’annulation peut être lancée en raison du délai de création. L’annulation peut ou non être effective.  

### <a name="network-security-and-access-policies-associated-with-integrated-ca"></a>Stratégies de sécurité et d’accès réseau associées à l’autorité de certification intégrée
Le service Key Vault envoie des demandes à l’autorité de certification (trafic sortant). Par conséquent, il est entièrement compatible avec les coffres de clés qui se trouvent derrière un pare-feu. Il ne partage pas de stratégies d’accès avec l’autorité de certification. L’autorité de certification doit être configurée de façon à accepter indépendamment les demandes de signature. [Guide d’intégration de l’autorité de certification approuvée](./how-to-integrate-certificate-authority.md)

## <a name="import-a-certificate"></a>Importation d’un certificat  
 Vous pouvez également importer un certificat dans Key Vault : PFX ou PEM.  

 L’importation d’un certificat nécessite la présence d’un certificat PEM ou PFX sur le disque ainsi qu’une clé privée. 
-   Vous devez spécifier les noms du coffre et du certificat (la stratégie est facultative).

-   Les fichiers PEM/PFX contiennent des attributs que Key Vault peut analyser et utiliser pour remplir la stratégie de certificat. Si une stratégie de certificat est déjà spécifiée, Key Vault essaie de faire correspondre les données des fichiers PEM/PFX.  

-   Une fois l’importation finalisée, les opérations ultérieures utilisent la nouvelle stratégie (nouvelles versions).  

-   Si aucune opération ultérieure n’a lieu, l’instance Key Vault commence par envoyer un avis d’expiration. 

-   L’utilisateur peut également modifier la stratégie qui est fonctionnelle au moment de l’importation, mais qui contient des erreurs, car aucune information n’a été spécifiée lors de l’importation. Ex. Aucune information sur l’émetteur.  

### <a name="formats-of-import-we-support"></a>Formats d’importation que nous prenons en charge
Azure Key Vault prend en charge les fichiers de certificat .pem et .pfx pour l’importation de certificats dans le coffre de clés.
Nous prenons en charge le type d’importation suivant pour le format de fichier PEM. Un seul certificat encodé en PEM avec une clé non chiffrée encodée en PKCS #8, avec les éléments suivants

-----BEGIN CERTIFICATE----- -----END CERTIFICATE-----

-----BEGIN PRIVATE KEY----- -----END PRIVATE KEY-----

Quand vous importez le certificat, vous devez vérifier que la clé est incluse dans le fichier. Si vous avez la clé privée ailleurs et dans un autre format, vous devez associer la clé au certificat. Certaines autorités de certification fournissent des certificats dans différents formats. Avant d’importer le certificat, vous devez donc vérifier qu’il est au format .pem ou .pfx. 

### <a name="formats-of-merge-csr-we-support"></a>Formats de fusion de CSR que nous prenons en charge
AKV prend en charge 2 formats basés sur PEM. Vous pouvez fusionner un seul certificat encodé en PKCS#8 ou un fichier P7B encodé en Base64 (chaîne de certificats signés par l’autorité de certification). 

-----BEGIN CERTIFICATE----- -----END CERTIFICATE-----


## <a name="creating-a-certificate-with-a-ca-not-partnered-with-key-vault"></a>Création d’un certificat auprès d’une autorité de certification non associée à Key Vault  
 Cette méthode permet d’avoir recours à d’autres autorités de certification que les fournisseurs associés à Key Vault. Autrement dit, votre organisation peut utiliser une autorité de certification de son choix.  

![Créer un certificat avec votre propre autorité de certification](../media/certificate-authority-1.png)  

 Les descriptions suivantes correspondent aux étapes écrites en vert dans le diagramme précédent.  

  (1) : dans le diagramme ci-dessus, votre application crée un certificat qui commence en interne par la création d’une clé dans votre coffre de clés.  

  (2 ) : Key Vault retourne une demande de signature de certificat (CSR) à votre application.  

  (3) : votre application transmet la demande CSR à l’autorité de certification que vous avez choisie.  

  (4) : l’autorité de certification que vous avez choisie répond avec un certificat X509.  

  (5) : votre application termine la création du certificat par le biais d’une fusion avec le certificat X509 provenant de votre autorité de certification.
