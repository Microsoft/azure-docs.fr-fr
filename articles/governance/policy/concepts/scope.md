---
title: Comprendre l’étendue dans Azure Policy
description: Décrit le concept d’étendue dans Azure Resource Manager et comment il s’applique à Azure Policy pour contrôler les ressources qu’Azure Policy évalue.
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 83eda330a35b007abfa37046a202b14728726849
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90984428"
---
# <a name="understand-scope-in-azure-policy"></a>Comprendre l’étendue dans Azure Policy

Il existe un certain nombre de paramètres qui déterminent les ressources qui peuvent être évaluées et les ressources qui sont évaluées par Azure Policy. Le concept principal de ces contrôles est l’_étendue_.
Pour une vue d’ensemble de haut niveau, consultez la section [Étendue d’Azure Resource Manager](../../../azure-resource-manager/management/overview.md#understand-scope).
Cet article explique l’impact de l’_étendue_ dans Azure Policy et les objets et propriétés associés.

## <a name="definition-location"></a>Emplacement de la définition

L’étendue utilisée en premier lieu par Azure Policy est la création d’une définition de stratégie, qui doit être enregistrée dans un groupe d’administration ou un abonnement. L’emplacement détermine l’étendue à laquelle l’initiative ou la stratégie peut être affectée. Les ressources doivent faire partie de la hiérarchie de ressources de l’emplacement de la définition à cibler pour l’affectation.

Si l’emplacement de la définition est l’un ou l’autre élément suivant :

- **Abonnement** : seules les ressources au sein de cet abonnement peuvent être affectées à la définition de la stratégie.
- **Groupe d’administration** : seules les ressources au sein des groupes d’administration enfants et des abonnements enfants peuvent être affectées à la définition de la stratégie. Si vous voulez appliquer la définition de stratégie à plusieurs abonnements, l’emplacement doit correspondre à un groupe d’administration comportant chaque abonnement.

L’emplacement doit être le conteneur de ressources partagé par toutes les ressources sur lequel vous souhaitez utiliser la définition de stratégie. Ce conteneur de ressources est généralement un groupe d’administration proche du groupe d’administration racine.

## <a name="assignment-scopes"></a>Étendues d’affectation

Plusieurs propriétés d’une affectation définissent une étendue. L’utilisation de ces propriétés détermine quelles ressources doivent être évaluées par Azure Policy et quelles ressources sont prises en compte pour la conformité. Ces propriétés sont liées aux concepts suivants :

- Inclusion : la conformité d’une hiérarchie de ressources ou d’une ressource individuelle doit être évaluée par la définition. La propriété `properties.scope` sur un objet d’affectation détermine quelles ressources inclure et pour lesquelles la conformité doit être évaluée. Pour plus d’informations, consultez la section [Définition d’affectation](./assignment-structure.md).

- Exclusion : la conformité d’une hiérarchie de ressources ou d’une ressource individuelle ne doit pas être évaluée par la définition. La propriété de _tableau_ `properties.notScopes` sur un objet d’affectation détermine les ressources à exclure. Les ressources comprises dans ces étendues ne sont pas évaluées ou incluses dans le décompte de conformité. Pour plus d’informations, consultez la section [Définition d’affectation : étendues exclues](./assignment-structure.md#excluded-scopes).

En plus des propriétés d'affectation de stratégie figure un objet d'[exemption de stratégie](./exemption-structure.md). Les exemptions améliorent le scénario d'étendue en fournissant une méthode qui permet d'identifier une partie à ne pas évaluer au sein d'une affectation.

- Exemption (**fonctionnalité d’évaluation gratuite**) : la conformité d’une hiérarchie de ressources ou d’une ressource individuelle doit être évaluée par la définition, mais elle ne sera pas évaluée en raison d’une dérogation ou d’une atténuation par une autre méthode. Les ressources dans cet état apparaissent comme **Exemptées** dans les rapports de conformité afin de pouvoir être suivies. L’objet d’exemption est créé sur la hiérarchie de ressources ou la ressource individuelle en tant qu’objet enfant, afin de déterminer l’étendue de l’exemption. Une hiérarchie de ressources ou une ressource individuelle peut être exempte de plusieurs affectations. Il est possible de configurer l’exemption pour qu’elle expire selon un calendrier précis grâce à la propriété `expiresOn`. Pour plus d’informations, consultez la section [Définition d’exemption](./exemption-structure.md).

  > [!NOTE]
  > En raison de l’impact de l’octroi d’une exemption pour une hiérarchie de ressources ou une ressource individuelle, les exemptions comportent des mesures de sécurité supplémentaires. Au-delà de la nécessité de l’opération `Microsoft.Authorization/policyExemptions/write` sur la hiérarchie de ressources ou la ressource individuelle, le créateur d’une exemption doit avoir le verbe `exempt/Action` sur l’affectation cible.

## <a name="scope-comparison"></a>Comparaison d’étendue

Le tableau suivant présente une comparaison des options d’étendue :

| | Inclusion | Exclusion (notScopes) | Exemption |
|---|:---:|:---:|:---:|
|**Les ressource sont évaluées** | &#10004; | - | - |
|**Objet Gestionnaire des ressources** | - | - | &#10004; |
|**Nécessite la modification d’un objet d’affectation de stratégie** | &#10004; | &#10004; | - |

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [structure des définitions de stratégies](./definition-structure.md)
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).