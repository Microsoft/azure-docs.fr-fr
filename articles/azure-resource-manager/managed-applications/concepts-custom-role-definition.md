---
title: Vue d’ensemble des définitions de rôle personnalisées
description: Décrit le concept de création des définitions de rôle personnalisées pour les applications managées.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 09/16/2019
ms.openlocfilehash: 7b7fff913c177703f959bfa103c8e310d01059e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81391824"
---
# <a name="custom-role-definition-artifact-in-azure-managed-applications"></a>Artefact de définition de rôle personnalisée dans les applications managées Azure

La définition de rôle personnalisée est un artefact facultatif dans les applications managées. Il est utilisé pour déterminer les autorisations dont l’application managée a besoin pour exécuter ses fonctions.

Cet article présente l’artefact de définition de rôle personnalisée et ses fonctionnalités.

## <a name="custom-role-definition-artifact"></a>Artefact de définition de rôle personnalisée

Vous devez nommer l’artefact de définition de rôle personnalisée customRoleDefinition.json. Placez-le au même niveau que createUiDefinition.json et mainTemplate.json dans le package .zip qui crée une définition d’application managée. Pour savoir comment créer le package .zip et publier une définition d’application managée, consultez [Publier une définition d’application managée.](publish-service-catalog-app.md)

## <a name="custom-role-definition-schema"></a>Schéma de définition de rôle personnalisée

Le fichier customRoleDefinition.json a une propriété `roles` de niveau supérieur qui est un tableau de rôles. Ces rôles sont les autorisations dont l’application managée a besoin pour fonctionner. Actuellement, seuls les rôles intégrés sont autorisés, mais vous pouvez spécifier plusieurs rôles. Un rôle peut être référencé par l’ID de la définition de rôle ou par le nom de rôle.

Exemple de code JSON pour la définition de rôle personnalisée :

```json
{
    "contentVersion": "0.0.0.1",
    "roles": [
        {
            "properties": {
                "roleName": "Contributor"
            }
        },
        {
            "id": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
        },
        {
            "id": "/providers/Microsoft.Authorization/roledefinitions/9980e02c-c2be-4d73-94e8-173b1dc7cf3c"
        }
    ]
}
```

## <a name="roles"></a>Rôles

Un rôle est composé d’un `$.properties.roleName` ou d’un `id` :

```json
{
    "id": null,
    "properties": {
        "roleName": "Contributor"
    }
}
```

> [!NOTE]
> Vous pouvez utiliser le champ `id` ou `roleName`. Un seul est requis. Ces champs sont utilisés pour rechercher la définition de rôle qui doit être appliquée. Si les deux sont fournis, le champ `id` sera utilisé.

|Propriété|Requis ?|Description|
|---------|---------|---------|
|id|Oui|ID du rôle intégré. Vous pouvez utiliser l’ID complet ou simplement le GUID.|
|roleName|Oui|Le nom du rôle intégré.|