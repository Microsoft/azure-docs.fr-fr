---
title: Erreurs de nom de compte de stockage
description: Décrit les erreurs que vous pouvez rencontrer lorsque vous spécifiez un nom de compte de stockage.
ms.topic: troubleshooting
ms.date: 03/09/2018
ms.openlocfilehash: 5b2706d8540ea38ef08bf7ca0f804e6811a93085
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76153970"
---
# <a name="resolve-errors-for-storage-account-names"></a>Résoudre les erreurs liées aux noms de compte de stockage

Cet article décrit les erreurs de dénomination que vous risquez de rencontrer lorsque vous déployez un compte de stockage.

## <a name="symptom"></a>Symptôme

Si le nom de votre compte de stockage inclut des caractères interdits, vous recevez une erreur similaire à ce qui suit :

```
Code=AccountNameInvalid
Message=S!torageckrexph7isnoc is not a valid storage account name. Storage account name must be
between 3 and 24 characters in length and use numbers and lower-case letters only.
```

Pour les comptes de stockage, vous devez fournir un nom pour la ressource qui est unique dans Azure. Si vous ne fournissez pas un nom unique, une erreur de ce type s’affiche :

```
Code=StorageAccountAlreadyTaken
Message=The storage account named mystorage is already taken.
```

Si vous déployez un compte de stockage portant le même nom qu’un compte de stockage existant dans votre abonnement, mais que vous indiquez un emplacement différent, vous recevez une erreur indiquant que le compte de stockage existe déjà dans un autre emplacement. Supprimez le compte de stockage existant ou indiquez le même emplacement que le compte de stockage existant.

## <a name="cause"></a>Cause :

Ce nom doit comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres. Le nom doit être unique.

## <a name="solution"></a>Solution

Assurez-vous que le nom du compte de stockage est unique. Vous pouvez créer un nom unique en concaténant votre nommage avec le résultat de la fonction [uniqueString](template-functions-string.md#uniquestring) .

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]",
"type": "Microsoft.Storage/storageAccounts",
```

Vérifiez que le nom de votre compte de stockage ne présente pas plus de 24 caractères. La fonction [uniqueString](template-functions-string.md#uniquestring) renvoie 13 caractères. Si vous concaténez un préfixe ou un suffixe au résultat de la fonction **uniqueString**, fournissez une valeur de 11 caractères maximum.

```json
"parameters": {
  "storageNamePrefix": {
    "type": "string",
    "maxLength": 11,
    "defaultValue": "storage",
    "metadata": {
    "description": "The value to use for starting the storage account name."
    }
  }
}
```

Assurez-vous que le nom de votre compte de stockage n’inclut aucune majuscule, ni aucun caractère spécial.