---
title: Élément d’interface utilisateur ServicePrincipalSelector
description: Décrit l’élément d’interface utilisateur Microsoft.Common.ServicePrincipalSelector pour le portail Azure. Fournit un contrôle permettant de choisir une application, et une zone de texte pour entrer un mot de passe ou une empreinte de certificat.
author: tfitzmac
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: tomfitz
ms.openlocfilehash: 2fdbbad467d8c762db485fc7935e9cef78313fd0
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184448"
---
# <a name="microsoftcommonserviceprincipalselector-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.ServicePrincipalSelector

Contrôle qui permet aux utilisateurs de sélectionner un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) existant ou d’inscrire une nouvelle application. Quand vous sélectionnez **Créer nouveau**, vous suivez les étapes pour inscrire une nouvelle application. Lorsque vous sélectionnez une application existante, le contrôle fournit une zone de texte pour entrer un mot de passe ou une empreinte de certificat.

## <a name="ui-samples"></a>Exemples d’interface utilisateur

Vous pouvez utiliser une application par défaut, créer une nouvelle application ou utiliser une application existante.

### <a name="use-default-application-or-create-new"></a>Utiliser une application par défaut ou en créer une nouvelle

La vue par défaut est déterminée par les valeurs de la propriété `defaultValue` et le **Type de principal du service** est défini sur **Créer nouveau**. Si la propriété `principalId` contient un identificateur global unique (GUID) valide, le contrôle recherche le `objectId` de l’application. La valeur par défaut s’applique si l’utilisateur n’effectue pas de sélection dans le contrôle.

Si vous voulez inscrire une nouvelle application, sélectionnez **Modifier la sélection** : la boîte de dialogue **Inscrire une application** est alors affichée. Entrez le **Nom** et le **Type de compte pris en charge**, puis sélectionnez le bouton **Inscrire**.

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-default.png" alt-text="Vue initiale de Microsoft.Common.ServicePrincipalSelector.":::

Une fois que vous avez inscrit une nouvelle application, utilisez le **Type d’authentification** pour entrer un mot de passe ou une empreinte numérique de certificat.

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-authenticate.png" alt-text="Authentification de Microsoft.Common.ServicePrincipalSelector.":::

### <a name="use-existing-application"></a>Utiliser une application existante

Pour utiliser une application existante, choisissez **Sélectionner**, puis sélectionnez **Effectuer une sélection**. Utilisez la boîte de dialogue **Sélectionner une application** pour rechercher le nom de l’application. Dans les résultats, sélectionnez l’application, puis cliquez sur le bouton **Sélectionner**. Une fois que vous avez sélectionné une application, le contrôle affiche le **Type d’authentification** pour vous permettre d’entrer un mot de passe ou une empreinte numérique de certificat.

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-existing.png" alt-text="Microsoft.Common.ServicePrincipalSelector - Sélectionner une application existante.":::

## <a name="schema"></a>schéma

```json
{
  "name": "ServicePrincipal",
  "type": "Microsoft.Common.ServicePrincipalSelector",
  "label": {
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type",
    "sectionHeader": "Service Principal"
  },
  "toolTip": {
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type"
  },
  "defaultValue": {
    "principalId": "<default guid>",
    "name": "(New) default App Id"
  },
  "constraints": {
    "required": true,
    "regex": "^[a-zA-Z0-9]{8,}$",
    "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
  },
  "options": {
    "hideCertificate": false
  },
  "visible": true
}
```

## <a name="remarks"></a>Remarques

- Les propriétés obligatoires sont les suivantes :
  - `name`
  - `type`
  - `label`
  - `defaultValue` : Spécifie les valeurs `principalId` et `name` par défaut.

- Les propriétés facultatives sont les suivantes :
  - `toolTip` : Attache une info-bulle `infoBalloon` à chaque étiquette.
  - `visible` : Masquez ou affichez le contrôle.
  - `options` : Spécifie si l’option d’empreinte de certificat doit être proposée ou non.
  - `constraints` : Contraintes Regex pour la validation du mot de passe.

## <a name="example"></a>Exemple

Un exemple du contrôle `Microsoft.Common.ServicePrincipalSelector` est fourni ci-dessous. La propriété `defaultValue` définit `principalId` sur `<default guid>` en tant qu’espace réservé pour un GUID d’identificateur d’application par défaut.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "SPNcontrol",
        "label": "SPNcontrol",
        "elements": [
          {
            "name": "ServicePrincipal",
            "type": "Microsoft.Common.ServicePrincipalSelector",
            "label": {
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type",
              "sectionHeader": "Service Principal"
            },
            "toolTip": {
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type"
            },
            "defaultValue": {
              "principalId": "<default guid>",
              "name": "(New) default App Id"
            },
            "constraints": {
              "required": true,
              "regex": "^[a-zA-Z0-9]{8,}$",
              "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
            },
            "options": {
              "hideCertificate": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "appId": "[steps('SPNcontrol').ServicePrincipal.appId]",
      "objectId": "[steps('SPNcontrol').ServicePrincipal.objectId]",
      "password": "[steps('SPNcontrol').ServicePrincipal.password]",
      "certificateThumbprint": "[steps('SPNcontrol').ServicePrincipal.certificateThumbprint]",
      "newOrExisting": "[steps('SPNcontrol').ServicePrincipal.newOrExisting]",
      "authenticationType": "[steps('SPNcontrol').ServicePrincipal.authenticationType]"
    }
  }
}
```

## <a name="example-output"></a>Exemple de sortie

L’`appId` est l’ID de l’inscription d’application que vous avez sélectionnée ou créée. L’`objectId` est un tableau d’ID d’objet pour les principaux de service configurés pour l’inscription de l’application sélectionnée.

Quand aucune sélection n’est effectuée à partir du contrôle, la valeur de la propriété `newOrExisting` est **new** :

```json
{
  "appId": {
    "value": "<default guid>"
  },
  "objectId": {
    "value": ["<default guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "new"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

Quand vous sélectionnez **Créer nouveau** ou une application existante à partir du contrôle, la valeur de la propriété `newOrExisting` est **existing** :

```json
{
  "appId": {
    "value": "<guid>"
  },
  "objectId": {
    "value": ["<guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "existing"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
- Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).