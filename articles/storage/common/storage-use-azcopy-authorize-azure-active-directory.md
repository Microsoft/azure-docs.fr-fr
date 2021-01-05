---
title: Autoriser l’accès aux objets blob avec AzCopy et Azure Active Directory | Microsoft Docs
description: Vous pouvez fournir des informations d’identification d’autorisation pour les opérations AzCopy à l’aide d’Azure Active Directory (Azure AD).
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 11/03/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: b13b5e1e27e9717066ff8f1aa8e245e8d9f54bbb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498113"
---
# <a name="authorize-access-to-blobs-with-azcopy-and-azure-active-directory-azure-ad"></a>Autoriser l’accès aux objets blob avec AzCopy et Azure Active Directory (Azure AD)

Vous pouvez fournir à AzCopy des informations d’identification d’autorisation à l’aide d’Azure AD. De cette façon, vous n’aurez pas à ajouter un jeton de signature d’accès partagé (SAP) à chaque commande. 

Commencez par vérifier vos attributions de rôles. Ensuite, choisissez le type de _principal de sécurité_ que vous souhaitez autoriser. Il peut s’agir d’une [identité d’utilisateur](../../active-directory/fundamentals/add-users-azure-active-directory.md), d’une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) ou d’un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md).

Une identité d’utilisateur est tout utilisateur ayant une identité dans Azure AD. Il s’agit du principal de sécurité le plus facile à autoriser. Les identités managées et les principaux de service sont d’excellentes options si vous envisagez d’utiliser AzCopy à l’intérieur d’un script qui s’exécute sans intervention de l’utilisateur. Une identité managée est mieux adaptée pour les scripts qui s’exécutent à partir d’une machine virtuelle Azure, et un principal de service est mieux adapté pour les scripts qui s’exécutent localement. 

Pour plus d’informations sur AzCopy, consultez [Prise en main d’AzCopy](storage-use-azcopy-v10.md).

## <a name="verify-role-assignments"></a>Vérifier les attributions de rôles

Le niveau d’autorisation dont vous avez besoin repose sur votre volonté de planifier le chargement des fichiers, ou juste leur téléchargement.

Si vous voulez simplement télécharger les fichiers, vérifiez que le rôle [Lecteur des données blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) a été attribué à votre identité d’utilisateur, identité managée ou principal de service.

Si vous souhaitez télécharger des fichiers, vérifiez qu’un de ces rôles a été attribué à votre principal de sécurité :

- [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)

Ces rôles peuvent être attribués à votre principal de service sur l’une de ces étendues :

- Conteneur (système de fichiers)
- Compte de stockage
- Resource group
- Abonnement

Pour savoir comment vérifier et attribuer des rôles, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux objets blob et données de file d’attente](./storage-auth-aad-rbac-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

> [!NOTE]
> Gardez à l’esprit que la propagation des affectations de rôles Azure peut prendre cinq minutes.

Vous n’avez pas besoin qu’un de ces rôles soit assigné à votre responsable de la sécurité si votre responsable de la sécurité est ajouté à la liste de contrôle d’accès du conteneur ou du répertoire cible. Dans la liste de contrôle d’accès, votre responsable de la sécurité a besoin de droits d’écriture sur le répertoire cible et d’autorisation d’exécution sur le conteneur et chaque répertoire parent.

Pour en savoir plus, voir [Modèle de contrôle d’accès dans Azure Data Lake Storage Gen2](../blobs/data-lake-storage-access-control-model.md).

## <a name="authorize-a-user-identity"></a>Autoriser une identité d’utilisateur

Une fois que vous avez vérifié que votre identité d’utilisateur est associée au niveau d’autorisation nécessaire, ouvrez une invite de commandes, saisissez la commande suivante, puis appuyez sur la touche ENTRÉE.

```azcopy
azcopy login
```

Si vous recevez une erreur, essayez d’inclure l’ID de locataire de l’organisation à laquelle appartient le compte de stockage.

```azcopy
azcopy login --tenant-id=<tenant-id>
```

Remplacez l’espace réservé `<tenant-id>` par l’ID de locataire de l’organisation à laquelle appartient le compte de stockage. Dans le Portail Microsoft Azure, sélectionnez **Azure Active Directory > Propriétés > ID du répertoire**.

Cette commande retourne un code d’authentification et l’URL d’un site web. Ouvrez le site web, indiquez le code, puis choisissez le bouton **Suivant**.

![Créez un conteneur.](media/storage-use-azcopy-v10/azcopy-login.png)

Une fenêtre de connexion s’affiche. Dans cette fenêtre, connectez-vous à votre compte Azure à l’aide de vos informations d’identification de compte Azure. Une fois connecté, vous pouvez fermer la fenêtre du navigateur et commencer à utiliser AzCopy.

<a id="service-principal"></a>

## <a name="authorize-a-managed-identity"></a>Autoriser une identité managée

C’est une excellente option si vous prévoyez d’utiliser AzCopy à l’intérieur d’un script qui s'exécute sans interaction de l’utilisateur et depuis une machine virtuelle Azure. Lorsque vous utilisez cette option, vous n’êtes pas tenu de stocker les informations d’identification sur la machine virtuelle.

Vous pouvez vous connecter à votre compte à l’aide d’une identité managée à l’échelle du système que vous avez activée sur votre machine virtuelle, ou à l’aide de l’ID client, de l’ID objet ou de l’ID ressource d’une identité managée attribuée par l’utilisateur que vous avez attribuée à votre machine virtuelle.

Pour en savoir plus sur l’activation d’une identité managée à l'échelle du système ou la création d'une identité managée attribuée par l'utilisateur, consultez [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#enable-system-assigned-managed-identity-on-an-existing-vm).

#### <a name="authorize-by-using-a-system-wide-managed-identity"></a>Autoriser à l’aide d’une identité managée à l’échelle du système

Commencez par vérifier que vous avez activé une identité managée à l'échelle du système sur votre machine virtuelle. Consultez [Identité managée attribuée par le système](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

Ensuite, dans votre console de commande, entrez la commande suivante et appuyez sur la touche ENTRÉE.

```azcopy
azcopy login --identity
```

#### <a name="authorize-by-using-a-user-assigned-managed-identity"></a>Autoriser à l’aide d’une identité managée attribuée par l’utilisateur

Commencez par vérifier que vous avez activé une identité managée attribuée par l'utilisateur sur votre machine virtuelle. Consultez [Identité managée attribuée par l'utilisateur](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#user-assigned-managed-identity).

Ensuite, dans votre console de commande, entrez une des commandes suivantes et appuyez sur la touche ENTRÉE.

```azcopy
azcopy login --identity --identity-client-id "<client-id>"
```

Remplacez l'espace réservé `<client-id>` par l'ID client de l’identité managée attribuée par l’utilisateur.

```azcopy
azcopy login --identity --identity-object-id "<object-id>"
```

Remplacez l'espace réservé `<object-id>` par l'ID objet de l’identité managée attribuée par l’utilisateur.

```azcopy
azcopy login --identity --identity-resource-id "<resource-id>"
```

Remplacez l'espace réservé `<resource-id>` par l'ID ressource de l’identité managée attribuée par l’utilisateur.

## <a name="authorize-a-service-principal"></a>Autoriser un principal de service

C’est une excellente option si vous prévoyez d’utiliser AzCopy à l’intérieur d’un script qui s'exécute sans interaction de l’utilisateur, en local notamment. Si vous envisagez d’exécuter AzCopy sur des machines virtuelles s’exécutant dans Azure, une identité de service managée est plus facile à administrer. Pour plus d’informations, consultez la section [Autoriser une identité managée](#authorize-a-managed-identity) dans cet article.

Avant d’exécuter un script, vous devez vous connecter de manière interactive au moins une fois afin de pouvoir fournir à AzCopy les informations d’identification de votre fournisseur de services.  Ces informations d’identification sont stockées dans un fichier sécurisé et chiffré afin que votre script n’ait pas à fournir ces informations sensibles.

Vous pouvez vous connecter à votre compte en utilisant un secret client ou en utilisant le mot de passe d’un certificat qui est associé à l’enregistrement de l’application de votre directeur de service.

Pour en savoir plus sur la création du principal du service, consultez la page [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

Pour en savoir plus sur les principaux de service, consultez [Objets application et principal de service dans Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md).

#### <a name="authorize-a-service-principal-by-using-a-client-secret"></a>Autoriser un principal de service à l’aide d’une clé secrète client

Commencez par définir la variable d’environnement `AZCOPY_SPA_CLIENT_SECRET` pour la clé secrète client de votre inscription d’application de principal de service.

> [!NOTE]
> Assurez-vous de définir cette valeur à partir de votre invite de commande, et non dans les paramètres des variables d’environnement de votre système d’exploitation. De cette façon, la valeur n’est disponible que pour la session en cours.

Cet exemple montre comment vous pouvez le faire dans PowerShell.

```azcopy
$env:AZCOPY_SPA_CLIENT_SECRET="$(Read-Host -prompt "Enter key")"
```

> [!NOTE]
> Envisagez d’utiliser une invite comme dans cet exemple. De cette façon, votre mot de passe n’apparaîtra pas dans l’historique des commandes de votre console.  

Ensuite, tapez la commande suivante, puis appuyez sur la touche ENTRÉE.

```azcopy
azcopy login --service-principal  --application-id application-id --tenant-id=tenant-id
```

Remplacez l’espace réservé `<application-id>` par l’ID d’application de votre enregistrement d’application de principal de service. Remplacez l’espace réservé `<tenant-id>` par l’ID de locataire de l’organisation à laquelle appartient le compte de stockage. Dans le Portail Microsoft Azure, sélectionnez **Azure Active Directory > Propriétés > ID du répertoire**. 

#### <a name="authorize-a-service-principal-by-using-a-certificate"></a>Autoriser un principal de service à l’aide d’un certificat

Si vous préférez utiliser vos propres informations d’identification pour l’autorisation, vous pouvez télécharger un certificat à l’enregistrement de votre application, puis utiliser ce certificat pour vous connecter.

En plus de télécharger votre certificat dans votre application, vous devrez également avoir une copie du certificat enregistrée sur la machine ou la VM sur laquelle AzCopy sera exécuté. Cette copie du certificat doit être au format.PFX ou.PEM et doit inclure la clé privée. La clé privée doit être protégée par un mot de passe. Si vous utilisez Windows et que votre certificat n’existe que dans une liste de certificats, assurez-vous d’exporter ce certificat vers un fichier PFX (y compris la clé privée). Pour obtenir des instructions, consultez la page sur [la cmdlet Export-PfxCertificate](/powershell/module/pkiclient/export-pfxcertificate)

Ensuite, définissez la variable d’environnement `AZCOPY_SPA_CERT_PASSWORD` pour le mot de passe du certificat.

> [!NOTE]
> Assurez-vous de définir cette valeur à partir de votre invite de commande, et non dans les paramètres des variables d’environnement de votre système d’exploitation. De cette façon, la valeur n’est disponible que pour la session en cours.

Cet exemple montre comment vous pouvez effectuer cette tâche dans PowerShell.

```azcopy
$env:AZCOPY_SPA_CERT_PASSWORD="$(Read-Host -prompt "Enter key")"
```

Ensuite, tapez la commande suivante, puis appuyez sur la touche ENTRÉE.

```azcopy
azcopy login --service-principal --certificate-path <path-to-certificate-file> --tenant-id=<tenant-id>
```

Remplacez l’espace réservé `<path-to-certificate-file>` par le chemin d’accès complet ou relatif du fichier de certificat. AzCopy enregistre le chemin d’accès à ce certificat, mais il n’enregistre pas une copie du certificat, alors assurez-vous de garder ce certificat en place. Remplacez l’espace réservé `<tenant-id>` par l’ID de locataire de l’organisation à laquelle appartient le compte de stockage. Dans le Portail Microsoft Azure, sélectionnez **Azure Active Directory > Propriétés > ID du répertoire**.

> [!NOTE]
> Envisagez d’utiliser une invite comme dans cet exemple. De cette façon, votre mot de passe n’apparaîtra pas dans l’historique des commandes de votre console. 

<a id="managed-identity"></a>


## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur AzCopy, consultez [Prise en main d’AzCopy](storage-use-azcopy-v10.md).

- Si vous avez des questions, des problèmes ou des commentaires d’ordre général, saisissez-les sur la page [GitHub](https://github.com/Azure/azure-storage-azcopy).