---
title: Grammaire des règles de revendication Azure Attestation
description: Détails sur les revendications de stratégie et les règles de revendication Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 3ed5c3f8232047787c6f05628f1eef35a7533999
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91285416"
---
# <a name="claim-and-claim-rules"></a>Revendication et règles de revendication

Pour comprendre la grammaire des règles de revendication, vous devez d’abord comprendre les revendications de stratégie d’attestation.

## <a name="claim"></a>Revendication

Une revendication est un ensemble de propriétés regroupées pour fournir des informations pertinentes. Pour Azure Attestation, une revendication contient les propriétés suivantes :

- **type** : valeur de chaîne qui représente le type de la revendication.
- **value** : valeur booléenne, entière ou de chaîne qui représente la valeur de la revendication.
- **valueType** : type de données des informations stockées dans la propriété value. Les types pris en charge sont String, Integer et Boolean. Si cette propriété n’est pas définie, la valeur par défaut est « String ».
- **issuer** : informations relatives à l’émetteur de la revendication. L’émetteur a l’un des types suivants :
  - **AttestationService** : Azure Attestation met à la disposition de l’auteur de la stratégie d’attestation certaines revendications dont il peut se servir pour élaborer la stratégie appropriée.
  - **AttestationPolicy** : la stratégie elle-même (telle que définie par l’administrateur) peut ajouter des revendications à la preuve entrante pendant le traitement. Dans ce cas, l’émetteur est défini sur « AttestationPolicy ».
  - **CustomClaim** : l’attesteur (client) peut également ajouter des revendications à la preuve d’attestation. Dans ce cas, l’émetteur est défini sur « CustomClaim ».

Si cette propriété n’est pas définie, la valeur par défaut est « CustomClaim ».

## <a name="claim-rule"></a>Règle de revendication

Le jeu de revendications entrantes est utilisé par le moteur de stratégie pour calculer le résultat de l’attestation. Une règle de revendication est un ensemble de conditions utilisées pour valider les revendications entrantes et effectuer l’action définie.

```
Conditions list => Action (Claim)
```

L’évaluation par Azure Attestation d’une règle de revendication implique les étapes suivantes :

- Si la liste des conditions n’est pas présente, exécuter l’action avec la revendication spécifiée 
- Dans le cas contraire, évaluer les conditions indiquées dans la liste des conditions.
- Si l’évaluation de la liste des conditions retourne false, arrêter. Sinon, continuer.

Les conditions d’une règle de revendication permettent de déterminer si l’action doit être exécutée. La liste des conditions est une séquence de conditions séparées par l’opérateur « && ».

La liste des conditions est structurée comme suit :

```
Condition && Condition && ...
```

La condition est structurée comme suit :

```
Identifier:[ClaimPropertyCondition, ClaimPropertyCondition,…]
```

La liste de conditions est composée de conditions individuelles associées aux différentes propriétés d’une revendication. Une condition peut avoir un identificateur facultatif, qui peut être utilisé pour faire référence aux revendications qui satisfont à la condition. Cette référence peut être utilisée dans les autres conditions ou l’action de la même règle.

Par exemple

```
F1:[type=="OSName" , issuer=="CustomClaim"] && 
[type=="OSName" , issuer=="AttestationService", value== F1.value ] 
=> issueproperty(type="report_validity_in_minutes", value=1440);

F1:[type=="OSName" , issuer=="CustomClaim"] && 
C2:[type=="OSName" , issuer=="AttestationService", value== F1.value ] 
=> issue(claim = C2);
```

Voici les opérateurs qui peuvent être utilisés pour vérifier les conditions :

| Type de valeur | Opérations prises en charge |
|--|--|
| Integer | == (égal à), \!= (différent de), <= (inférieur ou égal à), < (inférieur à), >= (supérieur ou égal à), > (supérieur à) |
| String | == (égal à), \!= (différent de) |
| Boolean | == (égal à), \!= (différent de) |

Évaluation de la liste des conditions

- La présence de l’opérateur « && » implique qu’une liste de conditions n’est évaluée comme vraie que si l’évaluation de toutes les conditions de la liste retourne true.
- Une condition représente des critères de filtrage sur le jeu de revendications. La condition elle-même est réputée avoir la valeur true si au moins une revendication satisfait à la condition.
- Une revendication est considérée comme satisfaisant au critère de filtrage représenté par la condition si chacune de ses propriétés satisfait aux conditions de propriété de revendication correspondantes présentes dans la condition.  

L’ensemble d’actions autorisées dans une stratégie est décrit ci-dessous.

| Verbe de l’action | Description | Sections de la stratégie auxquelles elles s’appliquent |
|--|--|--|
| permit() | L’ensemble de revendications entrantes peut être utilisé pour calculer **issuancerules**. Ne prend pas de revendication comme paramètre. | **authorizationrules** |
| deny() | Le jeu de revendications entrantes ne doit pas être utilisé pour calculer **issuancerules**. Ne prend pas de revendication comme paramètre. | **authorizationrules** |
| add(revendication) | Ajoute la revendication au jeu de revendications entrantes. Toute revendication ajoutée au jeu de revendications entrantes est disponible pour les règles de revendication suivantes. |**authorizationrules**, **issuancerules** |
| issue(revendication) | Ajoute la revendication au jeu de revendications entrantes et sortantes. | **issuancerules** |
| issueproperty(revendication) | Ajoute la revendication au jeu de revendications entrantes et de propriétés. | **issuancerules**

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)

