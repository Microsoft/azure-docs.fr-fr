---
title: Fichier include
description: Fichier include
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 02/14/2020
ms.topic: include
ms.openlocfilehash: 5eb2517b4b71b61dd70406f345c005d007d9e3ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91535953"
---
L’entité pattern.any vous permet de rechercher des données en forme libre où le libellé de l’entité ne permet pas de distinguer la fin de l’entité du reste de l’énoncé.

Une application Ressources humaines permet aux employés de trouver les formulaires de l’entreprise.

|Énoncé|
|--|
|Où se trouve **HRF-123456** ?|
|Qui a créé **HRF-123234** ?|
|**HRF-456098** est-il publié en français ?|

Toutefois, chaque formulaire comporte à la fois un nom mis en forme utilisé dans le tableau précédent, ainsi qu’un nom convivial, tel que `Request relocation from employee new to the company 2018 version 5`.

Les énoncés avec le nom du formulaire convivial ressemblent à ceci :

|Énoncé|
|--|
|Où se trouve le formulaire **Request relocation from employee new to the company 2018 version 5** (demande de réaffectation d'un nouvel employé) ?|
|Qui a écrit le formulaire **Request relocation from employee new to the company 2018 version 5** (demande de réaffectation d'un nouvel employé - 2018 version 5) ?|
|Le formulaire **Request relocation from employee new to the company 2018 version 5** (demande de réaffectation d'un nouvel employé - 2018 version 5) ? est-t disponible en français ?|

Certains mots peuvent être source de confusion pour LUIS, qui ne sait pas où se termine l’entité. Dans un modèle, l’entité Pattern.any permet de spécifier le début et la fin du nom du formulaire afin que LUIS puisse extraire correctement son nom.

|Exemple de modèle d’énoncé|
|--|
|Où se trouve {FormName}[?]|
|Qui a créé {FormName}[?]|
|{nom_formulaire} est-il publié en Français[?]|

Vérifier les informations de [référence](../reference-entity-pattern-any.md) Pattern.any