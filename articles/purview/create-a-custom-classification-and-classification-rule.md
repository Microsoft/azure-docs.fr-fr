---
title: Création d’une classification et d’une règle de classification personnalisées (préversion)
description: Cet article vous explique comment créer des classifications personnalisées pour définir des types de données propres à votre organisation dans votre patrimoine de données. Il décrit également la création de règles de classification personnalisées permettant de trouver les données spécifiées dans l’ensemble du patrimoine de données.
author: animukherjee
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/1/2020
ms.openlocfilehash: 53c71eec4c913c606669b7d36ca9a71ac8f3e2f1
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693459"
---
# <a name="custom-classifications-in-azure-purview"></a>Classifications personnalisées dans Azure Purview 

Cet article vous explique comment créer des classifications personnalisées pour définir des types de données propres à votre organisation dans votre patrimoine de données. Il décrit également la création de règles de classification personnalisées permettant de trouver les données spécifiées dans l’ensemble du patrimoine de données.

## <a name="default-classifications"></a>Classifications par défaut

Le catalogue de données Azure Purview propose un grand nombre de classifications par défaut qui représentent des types de données personnelles typiques, que vous possédez peut-être dans votre patrimoine de données.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/classification.png" alt-text="Sélection de Classifications" border="true":::

Vous avez également la possibilité de créer des classifications personnalisées, si aucune des classifications par défaut ne répond à vos besoins.

## <a name="steps-to-create-a-custom-classification"></a>Procédure de création d’une classification personnalisée

Pour créer une classification personnalisée, procédez comme suit :

1. Dans votre catalogue, sélectionnez **Centre de gestion** dans le menu de gauche.

2. Sélectionnez **Classifications** sous **Gestion des métadonnées**.

3. Sélectionnez **+Nouveau**.

    :::image type="content" source="media/create-a-custom-classification-and-classification-rule/new-classification.png" alt-text="Nouvelle classification" border="true":::

Le volet **Ajouter une nouvelle classification** s’ouvre. Il vous permet d’attribuer un nom et une description à votre classification. Il est conseillé d’utiliser une convention d’espacement des noms du type `your company name.classification name`.
Les classifications système de Microsoft sont regroupées sous l’espace de noms réservé `MS.`, par exemple **MS.GOVERNMENT.US.SOCIAL\_SECURITY\_NUMBER**.

Le nom de la classification doit commencer par une lettre, suivie d’une séquence de lettres, de chiffres, de points (.) et de traits de soulignement.
Les espaces ne sont pas autorisés. À mesure que vous tapez, l’expérience utilisateur génère automatiquement un nom convivial. Le nom convivial est celui que voient les utilisateurs quand vous l’appliquez à une ressource dans le catalogue.

Pour que le nom convivial reste concis, le système le crée suivant la logique ci-dessous :

- Tous les segments de l’espace de noms sont découpés, à l’exception des deux derniers.

- La casse est ajustée de sorte que la première lettre de chaque mot soit en majuscule. Toutes les autres lettres sont converties en minuscules.

- Tous les traits de soulignement (\_) sont remplacés par des espaces.

Par exemple, si vous avez nommé votre classification **CONTOSO.HR.EMPLOYEE\_ID**, le nom convivial stocké dans le système est **Hr.Employee ID**.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/contoso-hr-employee-id.png" alt-text="CONTOSO.HR.EMPLOYEE_ID" border="true":::

Sélectionnez **OK** : votre nouvelle classification est ajoutée à la liste de vos classifications.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/custom-classification.png" alt-text="Classification personnalisée" border="true":::

Si vous sélectionnez la classification dans la liste, la page Détails de la classification s’ouvre. Vous y trouverez tous les détails de la classification :
nombre d’instances, nom formel, règles de classification associées (le cas échéant) et nom du propriétaire.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/select-classification.png" alt-text="Sélection de la classification" border="true":::

## <a name="custom-classification-rules"></a>Règles de classification personnalisées

Le service de catalogue fournit un ensemble de règles de classification par défaut, utilisées par l’analyseur pour détecter automatiquement certains types de données. Vous pouvez également ajouter vos propres règles de classification personnalisées pour détecter d’autres types de données qui vous intéressent. Cette fonctionnalité peut être très puissante si vous recherchez certaines données dans votre patrimoine de données.

Prenons par exemple une société nommée Contoso. Les ID employés sont standardisés dans l’ensemble de l’entreprise avec le mot \"Employee\" suivi d’un GUID, soit EMPLOYEE{GUID}. En voici une instance possible : EMPLOYEE9c55c474-9996-420c-a285-0d0fc23f1f55.

Contoso est en mesure de configurer le système d’analyse de façon à rechercher des instances de ces ID en créant une règle de classification personnalisée. L’entreprise peut fournir une expression régulière qui correspond au modèle de données, en l’occurrence `\^Employee\[A-Za-z0-9\]{8}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{12}\$`. Éventuellement, si les données se trouvent généralement dans une colonne dont le nom est connu, comme Employee\_ID ou EmployeeID, il est possible d’ajouter une expression régulière de modèle de colonne pour rendre l’analyse encore plus précise. Voici un exemple d’expression régulière : Employee\_ID\|EmployeeID.

Le système d’analyse peut alors, avec cette règle, examiner les données réelles et le nom de la colonne pour tenter d’identifier chacune des instances où le modèle d’ID d’employé est trouvé.

## <a name="steps-to-create-a-custom-classification-rule"></a>Procédure de création d’une règle de classification personnalisée

Pour créer une règle de classification personnalisée, procédez comme suit :

1. Créez une classification personnalisée en suivant les instructions de la section ci-dessus. Vous l’ajouterez dans la configuration de la règle de classification afin que le système l’applique lorsqu’il trouve une correspondance dans la colonne.

2. Sélectionnez l’icône **Centre de gestion**.

3. Sélectionnez la section **Règles de classification**.

    :::image type="content" source="media/create-a-custom-classification-and-classification-rule/classificationrules.png" alt-text="Vignette Règles de classification" border="true":::

4. Sélectionnez **Nouveau**.

    :::image type="content" source="media/create-a-custom-classification-and-classification-rule/newclassificationrule.png" alt-text="Ajout d’une nouvelle règle de classification" border="true":::

5. La boîte de dialogue **Nouvelle règle de classification** s’ouvre. Renseignez les informations de configuration de la nouvelle règle.

    :::image type="content" source="media/create-a-custom-classification-and-classification-rule/createclassificationrule.png" alt-text="Création d’une règle de classification" border="true":::

|Champ     |Description  |
|---------|---------|
|Nom   |    Obligatoire. 100 caractères maximum.    |
|Description      |facultatif. 256 caractères maximum.    |
|Nom de la classification    | Obligatoire. Sélectionnez le nom de la classification dans la liste déroulante pour indiquer à l’analyseur de l’appliquer si une correspondance est trouvée.        |
|État   |  Obligatoire. Les options sont activées ou désactivées. Elles sont par défaut activées.    |
|Modèle de données    |facultatif. Expression régulière représentant les données stockées dans le champ de données. La limite est très élevée. Dans l’exemple précédent, les modèles de données testent un ID d’employé correspondant littéralement au mot `Employee{GUID}`.  |
|Modèle de colonne    |facultatif. Expression régulière représentant les noms de colonnes à faire correspondre. La limite est très élevée.          |

Sous **Modèle de données** se trouvent deux options :

- **Seuil de correspondances distinctes** : nombre total de valeurs de données distinctes qui doivent se trouver dans une colonne pour que l’analyseur exécute le modèle de données dessus. La valeur suggérée est 8. Elle peut être ajustée manuellement dans une plage comprise entre 2 et 32. Le système a besoin de cette valeur afin de veiller à ce que la colonne contienne suffisamment de données pour que l’analyseur puisse les classer avec précision. Par exemple, une colonne qui comporte plusieurs lignes contenant toutes la valeur 1 ne sera pas classée. Les colonnes dont une ligne comprend une valeur et les autres lignes des valeurs Null ne sont pas non plus classées. Si vous spécifiez plusieurs modèles, cette valeur s’applique à chacun d’eux.

- **Seuil de correspondances minimales** : paramètre permettant de définir le pourcentage minimal de correspondances de valeurs de données dans une colonne qui doivent être trouvées par l’analyseur pour que la classification soit appliquée. La valeur suggérée est 60 %. Vous devez faire attention à ce paramètre. Si vous réduisez le niveau au-dessous de 60 %, vous risquez d’introduire des classifications faussement positives dans votre catalogue. Si vous spécifiez plusieurs modèles de données, ce paramètre est désactivé et la valeur fixée à 60 %.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous l’avez créé, votre règle de classification est prête à être ajoutée à un ensemble de règles d’analyse afin d’être utilisée par votre analyse. Pour plus d’informations, consultez [Création d’un ensemble de règles d’analyse](create-a-scan-rule-set.md).
