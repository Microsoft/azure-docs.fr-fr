---
title: À propos des certificats Azure Key Vault - Azure Key Vault
description: Vue d’ensemble de l’interface REST Azure Key Vault et des certificats.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: overview
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: 66f077028b9f9f7a7644a318d4447eeaaab19e98
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94919928"
---
# <a name="about-azure-key-vault-certificates"></a>À propos des certificats Azure Key Vault

La prise en charge des certificats Key Vault permet de gérer vos certificats x509 et les comportements suivants :  

-   Permet à un propriétaire de certificat de créer un certificat via un processus de création de Key Vault ou l’importation d’un certificat existant. Concerne à la fois les certificats auto-signés et ceux générés par une autorité de certification.
-   Permet à un propriétaire de certificat Key Vault d’implémenter le stockage sécurisé et la gestion des certificats X509 sans interaction avec des éléments de clé privée.  
-   Permet à un propriétaire de certificat de créer une stratégie qui ordonne à Key Vault de gérer le cycle de vie d’un certificat.  
-   Permet aux propriétaires de certificat de fournir des informations de contact pour les notifications concernant des événements d’expiration du cycle de vie et le renouvellement de certificat.  
-   Prend en charge le renouvellement automatique avec des émetteurs sélectionnés : fournisseurs de certificats X509 de partenaire Key Vault/autorités de certification.

>[!Note]
>Les fournisseurs/autorités non partenaires sont également autorisés, mais ils ne prendront pas en charge la fonctionnalité de renouvellement automatique.

## <a name="composition-of-a-certificate"></a>Composition d’un certificat

Lorsqu’un certificat Key Vault est créé, une clé et un secret adressables sont également créés avec le même nom. La clé Key Vault permet d’exécuter des opérations sur les clés et le secret Key Vault permet de récupérer la valeur du certificat en tant que secret. Un certificat Key Vault contient également des métadonnées de certificat x509 publiques.  

L’identificateur et la version de certificats sont similaires à ceux de clés et de secrets. Une version spécifique d’une clé et d’un secret adressables créés avec la version du certificat Key Vault est disponible dans la réponse de certificat Key Vault.
 
![Les certificats sont des objets complexes](../media/azure-key-vault.png)

## <a name="exportable-or-non-exportable-key"></a>Clé exportable ou non exportable

Lorsqu’un certificat Key Vault est créé, il peut être récupéré dans le secret adressable avec la clé privée au format PFX ou PEM. La stratégie utilisée pour créer le certificat doit indiquer que la clé est exportable. Si la stratégie indique qu’elle n’est pas exportable, la clé privée ne fait pas partie de la valeur quand elle est récupérée en tant que secret.  

La clé adressable est plus pertinente avec des certificats KV non exportables. Les opérations de la clé KV adressable sont mappées à partir du champ *keyusage* de la stratégie de certificat KV utilisée pour créer le certificat KV.  

Type de paire de clés pris en charge pour les certificats

 - Types de clés pris en charge : RSA, RSA-HSM, EC, EC-HSM, oct (listés [ici](/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)) : « Exportable » est autorisé seulement avec RSA et EC. Les clés HSM sont non exportables.

|Type de clé|À propos de|Sécurité|
|--|--|--|
|**RSA**| clé RSA « protégée par logiciel »|FIPS 140-2 niveau 1|
|**RSA-HSM**| Clé RSA « protégée par HSM » (référence SKU Premium uniquement)|HSM FIPS 140-2 niveau 2|
|**EC**| clé Elliptic Curve « protégée par logiciel »|FIPS 140-2 niveau 1|
|**EC-HSM**| Clé à courbe elliptique « protégée par HSM » (référence SKU Premium uniquement)|HSM FIPS 140-2 niveau 2|
|||

## <a name="certificate-attributes-and-tags"></a>Attributs et balises de certificat

En plus des métadonnées de certificat, une clé adressable, un secret adressable et un certificat Key Vault contiennent également des attributs et des balises.  

### <a name="attributes"></a>Attributs

Les attributs de certificat sont reproduits dans des attributs de la clé et du secret adressables créés lors de la création du certificat KV.  

Un certificat Key Vault comprend les attributs suivants :  

-   *enabled* : booléen, facultatif, **true** par défaut. Peut être spécifié pour indiquer si les données du certificat peuvent être récupérées en tant que secret ou utilisables en tant que clé. Également utilisé avec *nbf* et *exp*. Quand une opération se produit entre *nbf* et *exp*, elle est autorisée seulement si enabled a la valeur true. Les opérations en dehors de la fenêtre *nbf* et *exp* sont automatiquement interdites.  

Des attributs supplémentaires en lecture seule sont inclus dans la réponse :

-   *created* : IntDate, indique quand cette version du certificat a été créée.  
-   *updated* : IntDate, indique quand cette version du certificat a été mise à jour.  
-   *exp* : IntDate, contient la valeur de la date d’expiration du certificat X.509.  
-   *nbf* : IntDate, contient la valeur de la date du certificat X.509.  

> [!Note] 
> Si un certificat Key Vault expire, sa clé et son secret adressables ne sont plus utilisables.  

### <a name="tags"></a>Balises

 Dictionnaire de paires clé/valeur spécifié par le client, similaire aux balises dans les clés et les secrets.  

 > [!Note]
> Les balises peuvent être lues par un appelant s’il dispose de l’autorisation *list* ou *get* sur ce type d’objet (clés, secrets ou certificats).

## <a name="certificate-policy"></a>Stratégie de certificat

Une stratégie de certificat contient des informations sur la création et la gestion du cycle de vie d’un certificat Key Vault. Lorsqu’un certificat avec une clé privée est importé dans le coffre de clés, une stratégie par défaut est créée en lisant le certificat x509.  

Quand un certificat Key Vault est créé de zéro, une stratégie doit être fournie. La stratégie spécifie comment créer cette version du certificat Key Vault ou la prochaine version. Lorsqu’une stratégie a été définie, des opérations de création successives ne sont pas nécessaires pour les prochaines versions. Il n’existe qu’une seule instance d’une stratégie pour toutes les versions d’un certificat Key Vault.  

Globalement, une stratégie de certificat contient les informations suivantes (les définitions correspondantes se trouvent [ici](/powershell/module/az.keyvault/set-azkeyvaultcertificatepolicy?view=azps-4.4.0)) :  

-   Propriétés du certificat X.509 : contient le nom du sujet, les autres noms du sujet et d’autres propriétés utilisées pour créer une demande de certificat X.509.  
-   Propriétés des clés : contient les champs type de clé, longueur de clé, exportable et ReuseKeyOnRenewal. Ces champs indiquent au coffre de clés comment générer une clé. 
     - Types de clés pris en charge : RSA, RSA-HSM, EC, EC-HSM, Oct (répertorié [ici](/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)) 
-   Propriétés du secret : propriétés du secret comme le type de contenu de secret adressable pour générer la valeur du secret, pour récupérer le certificat en tant que secret.  
-   Actions de la durée de vie : actions de la durée de vie du certificat KV. Chaque action de la durée de vie contient :  

     - Déclencheur : spécifié en jours avant l’expiration ou en pourcentage de la durée de vie  

     - Action : spécifie le type d’action, *emailContacts* ou *autoRenew*  

-   Émetteur : paramètres relatifs à l’émetteur de certificat à utiliser pour émettre des certificats X.509.  
-   Attributs de stratégie : attributs liés à la stratégie  

### <a name="x509-to-key-vault-usage-mapping"></a>Mappage d’utilisation de clé X509 avec Key Vault

Le tableau suivant représente le mappage de la stratégie d’utilisation de la clé x509 avec des opérations sur les clés effectives d’une clé créée lors de la création d’un certificat Key Vault.

|**Indicateurs d’utilisation de la clé X509**|**Opérations sur la clé Key Vault**|**Comportement par défaut**|
|----------|--------|--------|
|DataEncipherment|encrypt, decrypt| N/A |
|DecipherOnly|decrypt| N/A  |
|DigitalSignature|sign, verify| Valeur par défaut de Key Vault sans spécification de l’utilisation au moment de la création du certificat | 
|EncipherOnly|encrypt| N/A |
|KeyCertSign|sign, verify|N/A|
|KeyEncipherment|wrapKey, unwrapKey| Valeur par défaut de Key Vault sans spécification de l’utilisation au moment de la création du certificat | 
|NonRepudiation|sign, verify| N/A |
|crlsign|sign, verify| N/A |

## <a name="certificate-issuer"></a>Émetteur de certificat

Un objet certificat Key Vault conserve une configuration utilisée pour communiquer avec un fournisseur de l’émetteur de certificat sélectionné pour demander des certificats x509.  

-   Partenaires Key Vault avec les fournisseurs de l’émetteur de certificat suivants pour les certificats TLS/SSL

|**Nom du fournisseur**|**Emplacements**|
|----------|--------|
|DigiCert|Pris en charge dans tous les emplacements de service de coffre de clés dans le cloud public et Azure Government|
|GlobalSign|Pris en charge dans tous les emplacements de service de coffre de clés dans le cloud public et Azure Government|

Avant de pouvoir créer un émetteur de certificat dans un coffre de clés, les étapes préliminaires 1 et 2 suivantes doivent être exécutées.  

1. Intégrer aux fournisseurs d’autorités de certification  

    -   Un administrateur de l’organisation doit intégrer sa société (par ex. Contoso) à au moins un fournisseur d’autorité de certification.  

2. L’administrateur crée des informations d’identification du demandeur pour Key Vault afin d’inscrire (et de renouveler) des certificats TLS/SSL  

    -   Fournit la configuration à utiliser pour créer un objet émetteur du fournisseur dans le coffre de clés  

Pour plus d’informations sur la création d’objets Émetteur à partir du portail Certificats, consultez le [blog de certificats Key Vault](/archive/blogs/kv/manage-certificates-via-azure-key-vault)  

Key Vault permet de créer plusieurs objets émetteurs avec une configuration de fournisseur de l’émetteur différente. Lorsqu’un objet émetteur est créé, son nom peut être référencé dans une ou plusieurs stratégies de certificat. Le référencement de l’objet émetteur indique à Key Vault d’utiliser la configuration telle que spécifiée dans l’objet émetteur lors de la demande du certificat x509 à partir du fournisseur d’autorité de certification lors de la création ou du renouvellement du certificat.  

Les objets émetteur sont créés dans le coffre et ne peuvent être utilisés qu’avec des certificats KV dans le même coffre.  

## <a name="certificate-contacts"></a>Contacts du certificat

Les contacts du certificat contiennent des informations de contact pour l’envoi de notifications déclenchées par des événements de durée de vie de certificat. Les informations de contact sont partagées par tous les certificats dans le coffre de clés. Une notification est envoyée à tous les contacts spécifiés pour un événement pour n’importe quel certificat dans le coffre de clés. Pour obtenir des informations sur la façon de définir un contact de certificat, reportez-vous [ici](overview-renew-certificate.md#steps-to-set-certificate-notifications).  

## <a name="certificate-access-control"></a>Contrôle d’accès aux certificats

 Le contrôle d’accès pour les certificats est géré par Key Vault et fourni par le coffre de clés qui contient ces certificats. La stratégie de contrôle d’accès pour les certificats est différente des stratégies de contrôle d’accès pour les clés et les secrets dans un même coffre de clés. Les utilisateurs peuvent créer un ou plusieurs coffres pour stocker les certificats afin de maintenir une segmentation et une gestion des certificats appropriées au scénario.  Pour plus d’informations sur le contrôle d’accès aux certificats, reportez-vous [ici](certificate-access-control.md).

## <a name="next-steps"></a>Étapes suivantes

- [À propos de Key Vault](../general/overview.md)
- [Présentation des clés, des secrets et des certificats](../general/about-keys-secrets-certificates.md)
- [À propos des clés](../keys/about-keys.md)
- [À propos des secrets](../secrets/about-secrets.md)
- [Authentification, requêtes et réponses](../general/authentication-requests-and-responses.md)
- [Guide du développeur Key Vault](../general/developers-guide.md)
