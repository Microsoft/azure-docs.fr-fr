---
title: Élément d’interface utilisateur CredentialsCombo
description: Décrit l’élément d’interface utilisateur Microsoft.Compute.CredentialsCombo pour le Portail Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 09/29/2018
ms.author: tomfitz
ms.openlocfilehash: 47c88e08e5d2eac09fbcd5b60a8ccd73b46c9616
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063762"
---
# <a name="microsoftcomputecredentialscombo-ui-element"></a>Élément d’interface utilisateur Microsoft.Compute.CredentialsCombo

Un groupe de contrôles avec validation intégrée pour les clés publiques SSH et les mots de passe Windows et Linux.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

Pour Windows, les utilisateurs voient :

![Microsoft.Compute.CredentialsCombo Windows](./media/managed-application-elements/microsoft-compute-credentialscombo-windows.png)

Pour Linux avec mot de passe sélectionné, les utilisateurs voient :

![Microsoft.Compute.CredentialsCombo mot de passe Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-password.png)

Pour Linux avec une clé publique SSH sélectionnée, les utilisateurs voient :

![Microsoft.Compute.CredentialsCombo clé Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-key.png)

## <a name="schema"></a>schéma

Pour Windows, utilisez le schéma suivant :

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "password": "Password",
    "confirmPassword": "Confirm password"
  },
  "toolTip": {
    "password": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false
  },
  "osPlatform": "Windows",
  "visible": true
}
```

Pour **Linux**, utilisez le schéma suivant :

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "authenticationType": "Authentication type",
    "password": "Password",
    "confirmPassword": "Confirm password",
    "sshPublicKey": "SSH public key"
  },
  "toolTip": {
    "authenticationType": "",
    "password": "",
    "sshPublicKey": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false,
    "hidePassword": false
  },
  "osPlatform": "Linux",
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

Si `osPlatform` est **Windows**, ou `osPlatform` est **Linux** et si l’utilisateur a fourni un mot de passe au lieu d’une clé publique SSH, le contrôle renvoie la sortie suivante :

```json
{
  "authenticationType": "password",
  "password": "p4ssw0rddem0",
}
```

Si `osPlatform` est **Linux** et si l’utilisateur a fourni une clé publique SSH, le contrôle renvoie la sortie suivante :

```json
{
  "authenticationType": "sshPublicKey",
  "sshPublicKey": "AAAAB3NzaC1yc2EAAAABIwAAAIEA1on8gxCGJJWSRT4uOrR13mUaUk0hRf4RzxSZ1zRbYYFw8pfGesIFoEuVth4HKyF8k1y4mRUnYHP1XNMNMJl1JcEArC2asV8sHf6zSPVffozZ5TT4SfsUu/iKy9lUcCfXzwre4WWZSXXcPff+EHtWshahu3WzBdnGxm5Xoi89zcE=",
}
```

## <a name="remarks"></a>Notes

- `osPlatform` doit être spécifié. Il peut s’agir de **Windows** ou de **Linux**.
- Si `constraints.required` a la valeur **true**, le mot de passe ou les zones de texte de la clé publique SSH doivent contenir des valeurs pour pouvoir effectuer la validation correctement. La valeur par défaut est **true**.
- Si `options.hideConfirmation` est défini sur **true**, la deuxième zone de texte de confirmation du mot de passe utilisateur est masquée. La valeur par défaut est **false**.
- Si `options.hidePassword` est défini sur **true**, l’option permettant d’utiliser l’authentification par mot de passe est masquée. Elle peut être utilisée uniquement lorsque `osPlatform` est **Linux**. La valeur par défaut est **false**.
- Il est possible d’implémenter des contraintes supplémentaires sur les mots de passe autorisés à l’aide de la propriété `customPasswordRegex`. La chaîne de `customValidationMessage` s’affiche lorsque la validation personnalisée du mot de passe échoue. La valeur par défaut pour les deux propriétés est **null**.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
