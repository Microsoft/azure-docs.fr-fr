---
title: Artefact CreateUiDefinition
description: Montre comment créer l’artefact createUiDefinition pour une application managée Azure. Le fichier est nommé createUiDefinition.json.
ms.topic: conceptual
ms.author: lazinnat
author: lazinnat
ms.date: 07/11/2019
ms.openlocfilehash: 9be1cb02c419314a33b9487bf183ef3659b2b549
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649513"
---
# <a name="reference-user-interface-elements-artifact"></a>Référence : Artefact d’éléments d’interface utilisateur

Cet article est une référence pour un artefact *createUiDefinition.json* dans des applications managées Azure. Pour plus d’informations sur la création d’éléments d’interface utilisateur, consultez [Créer des éléments d’interface utilisateur](create-uidefinition-elements.md).

## <a name="user-interface-elements"></a>Options de l’interface utilisateur

Le fichier JSON suivant montre un exemple de fichier *createUiDefinition.json* pour les applications managées Azure :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "applicationSettings",
        "label": "Application Settings",
        "subLabel": {
          "preValidation": "Configure your application settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Application Settings",
        "elements": [
          {
            "name": "funcname",
            "type": "Microsoft.Common.TextBox",
            "label": "Name of the function to be created",
            "toolTip": "Name of the function to be created",
            "visible": true,
            "constraints": {
              "required": true
            }
          },
          {
            "name": "storagename",
            "type": "Microsoft.Common.TextBox",
            "label": "Name of the storage to be created",
            "toolTip": "Name of the storage to be created",
            "visible": true,
            "constraints": {
              "required": true
            }
          },
          {
            "name": "zipFileBlobUri",
            "type": "Microsoft.Common.TextBox",
            "defaultValue": "https://github.com/Azure/azure-quickstart-templates/tree/master/101-custom-rp-with-function/artifacts/functionzip/functionpackage.zip",
            "label": "The Uri to the uploaded function zip file",
            "toolTip": "The Uri to the uploaded function zip file",
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "funcname": "[steps('applicationSettings').funcname]",
      "storageName": "[steps('applicationSettings').storagename]",
      "zipFileBlobUri": "[steps('applicationSettings').zipFileBlobUri]"
    }
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer une application managée avec des actions et des ressources personnalisées](tutorial-create-managed-app-with-custom-provider.md)
- [Reference : Artefact de modèle de déploiement](reference-main-template-artifact.md)
- [Reference : Afficher un artefact de définition](reference-view-definition-artifact.md)