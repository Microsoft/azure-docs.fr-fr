---
title: azcopy login | Microsoft Docs
description: Cet article fournit des informations de référence sur la commande azcopy login.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 98f8554d6313147c03d4a0bec74e36043cdce342
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87285269"
---
# <a name="azcopy-login"></a>azcopy login

Se connecte à Azure Active Directory pour accéder aux ressources du stockage Azure.

## <a name="synopsis"></a>Synopsis

Connectez-vous à Azure Active Directory pour accéder aux ressources du stockage Azure.

Pour être autorisé à utiliser votre compte de stockage Azure, vous devez attribuer le rôle **Contributeur aux données Blob du stockage** à votre compte d’utilisateur, dans le contexte du compte de stockage, du groupe de ressources parent ou de l’abonnement parent.

Cette commande met en cache les informations de connexion chiffrées pour l’utilisateur actuel à l’aide de mécanismes intégrés au système d’exploitation.

> [!IMPORTANT]
> Si vous définissez une variable d’environnement à l’aide de la ligne de commande, la variable sera lisible dans votre historique de ligne de commande. Vous pouvez supprimer de l’historique de la ligne de commande les variables qui contiennent des informations d’identification. Pour empêcher l’affichage des variables dans votre historique, vous pouvez utiliser un script qui invite l’utilisateur à entrer ses informations d’identification et qui définit la variable d’environnement.

```azcopy
azcopy login [flags]
```

## <a name="related-conceptual-articles"></a>Articles conceptuels associés

- [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)
- [Transférer des données avec AzCopy et le Stockage Blob](storage-use-azcopy-blobs.md)
- [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Exemples

Connectez-vous de manière interactive avec l’ID de locataire AAD par défaut qui est défini sur « common » :

```azcopy
azcopy login
```

Connectez-vous de manière interactive avec un ID de locataire spécifié :

```azcopy
azcopy login --tenant-id "[TenantID]"
```

Se connecter à l’aide de l’identité attribuée par le système d’une machine virtuelle :

```azcopy
azcopy login --identity
```

Se connecter à l’aide de l’identité attribuée par l’utilisateur d’une machine virtuelle et un ID de client de l’identité du service :
  
```azcopy
azcopy login --identity --identity-client-id "[ServiceIdentityClientID]"
```
 
Se connecter à l’aide de l’identité attribuée par l’utilisateur d’une machine virtuelle et un ID d’objet de l’identité du service :

```azcopy
azcopy login --identity --identity-object-id "[ServiceIdentityObjectID]"
```

Se connecter à l’aide de l’identité attribuée par l’utilisateur d’une machine virtuelle et un ID de ressource de l’identité du service :
 
```azcopy
azcopy login --identity --identity-resource-id "/subscriptions/<subscriptionId>/resourcegroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myID"
```

Connectez-vous en tant que principal de service à l’aide d’une clé secrète client : Définissez la variable d’environnement AZCOPY_SPA_CLIENT_SECRET sur le secret client pour une authentification du principal de service basée sur les secrets.

```azcopy
azcopy login --service-principal --application-id <your service principal's application ID>
```

Connectez-vous en tant que principal de service à l’aide d’un certificat et de son mot de passe :

Définissez la variable d’environnement AZCOPY_SPA_CERT_PASSWORD sur le mot de passe du certificat pour une autorisation du principal de service par certificat :

```azcopy
azcopy login --service-principal --certificate-path /path/to/my/cert --application-id <your service principal's application ID>
```

Traitez `/path/to/my/cert` comme un chemin d’accès à un fichier PEM ou PKCS12. AzCopy n’accède pas au magasin de certificats du système pour obtenir votre certificat.

`--certificate-path` est obligatoire lors d’une authentification du principal de service par certificat.

## <a name="options"></a>Options

Chaîne **--aad-endpoint** : point de terminaison Azure Active Directory à utiliser. La valeur par défaut (https://login.microsoftonline.com) ) est correcte pour le cloud Azure public. Définissez ce paramètre lors de l'authentification dans un cloud national. Non nécessaire pour Managed Service Identity.

Chaîne **--application-id** : ID d’application de l’identité affectée par l’utilisateur. Obligatoire pour l’authentification du principal de service.

Chaîne **--certificate-path** : chemin du certificat pour l’authentification SPN. Obligatoire pour l’authentification du principal de service basée sur les certificats.

**--help** : aide pour la commande `azcopy login`.

**--identity** : connexion à l’aide de l’identité de la machine virtuelle, également appelée « Managed Service Identity » (MSI).

Chaîne **--identity-client-id** : ID client de l’identité affectée par l’utilisateur.

Chaîne **--identity-object-id** : ID d’objet de l’identité affectée par l’utilisateur.

Chaîne **--identity-resource-id** : ID de ressource de l’identité affectée par l’utilisateur.

**--service-principal** : connexion via le nom de principal du service (SPN) à l’aide d’un certificat ou d’un secret. Le secret client ou le mot de passe du certificat doit être placé dans la variable d’environnement appropriée. Saisissez « AzCopy » pour afficher les noms et les descriptions des variables d’environnement.

Chaîne **--tenant-id** : ID de locataire Azure Active Directory à utiliser pour la connexion interactive de l’appareil OAuth.

## <a name="options-inherited-from-parent-commands"></a>Options héritées des commandes parentes

|Option|Description|
|---|---|
|--cap-mbps float|Limite la vitesse de transfert, en mégabits par seconde. Par moment, le débit peut dépasser légèrement cette limite. Si cette option est définie sur zéro ou si elle est omise, le débit n’est pas limité.|
|--output-type (chaîne)|Met en forme la sortie de la commande. Les formats possibles sont « text » et « JSON ». La valeur par défaut est « text ».|
|--trusted-microsoft-suffixes (chaîne)   |Spécifie des suffixes de domaine supplémentaires où des jetons de connexion Azure Active Directory peuvent être envoyés.  La valeur par défaut est «  *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net ». Tous les éléments répertoriés ici sont ajoutés à la valeur par défaut. Pour la sécurité, vous devez placer uniquement des domaines Microsoft Azure ici. Séparez plusieurs entrées par des points-virgules.|

## <a name="see-also"></a>Voir aussi

- [azcopy](storage-ref-azcopy.md)
