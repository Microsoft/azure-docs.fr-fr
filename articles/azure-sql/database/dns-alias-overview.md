---
title: Alias DNS
description: Vos applications peuvent se connecter à un alias du nom du serveur pour Azure SQL Database. Vous pouvez pendant ce temps modifier la base de données SQL sur laquelle pointe l’alias à tout moment, pour faciliter les tests et autres.
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: seo-lt-2019 sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: genemi, jrasnick, vanto
ms.date: 06/26/2019
ms.openlocfilehash: 25c2157a8de237d0ec66caa72d59c810d419ac76
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94989120"
---
# <a name="dns-alias-for-azure-sql-database"></a>Alias DNS pour Azure SQL Database
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Azure SQL Database dispose d’un serveur DNS (Domain Name System). PowerShell et les API REST acceptent des [appels pour créer et gérer des alias DNS](#anchor-powershell-code-62x) pour le nom de votre [serveur SQL logique](logical-servers.md).

Un *alias DNS* peut être utilisé à la place du nom du serveur. notamment dans la chaîne de connexion des programmes clients. Il représente une couche de traduction capable de rediriger les programmes clients sur différents serveurs. Il évite ainsi d’avoir à rechercher et à modifier tous les clients et leur chaîne de connexion.

Voici quelques-unes des utilisations courantes d’un alias DNS :

- Créer un nom facile à retenir pour un serveur.
- Pendant le développement initial, l’alias peut faire référence à un serveur de test. Lorsque l’application est publiée, vous pouvez le modifier afin qu’il fasse référence au serveur de production. La transition du test à la production n’implique aucune modification des configurations de plusieurs clients qui se connectent au serveur.
- Supposons que l’unique base de données de votre application soit déplacée vers un autre serveur. Vous pouvez modifier l’alias sans avoir à changer les configurations de plusieurs clients.
- Pendant une panne régionale, la restauration géographique vous permet de récupérer votre base de données dans un autre serveur et une autre région. Vous pouvez modifier votre alias existant de façon à pointer vers le nouveau serveur, de sorte que l’application cliente existante puisse s’y connecter à nouveau.

## <a name="domain-name-system-dns-of-the-internet"></a>DNS (Domain Name System) d’Internet

Internet s’appuie sur le serveur DNS. Le serveur DNS traduit les noms conviviaux pour trouver le nom du serveur.

## <a name="scenarios-with-one-dns-alias"></a>Scénarios avec un seul alias DNS

Supposons que vous deviez faire passer votre système à un nouveau serveur. Auparavant, il vous fallait rechercher et mettre à jour la chaîne de connexion de chacun des programmes clients. Maintenant, si les chaînes de connexion utilisent un alias DNS, il n’y a qu’une propriété d’alias à modifier.

La fonctionnalité d’alias DNS d’Azure SQL Database est utile dans les cas suivants :

### <a name="test-to-production"></a>Passage des tests à la production

Lorsque vous commencez à développer les programmes clients, demandez-leur d’utiliser un alias DNS dans leur chaîne de connexion. Faites en sorte que les propriétés de l’alias pointent vers une version de test de votre serveur.

Quand le nouveau système passera en production, vous pourrez les mettre à jour afin qu’elles pointent vers le serveur de production. Aucune modification des programmes clients n’est nécessaire.

### <a name="cross-region-support"></a>Prise en charge interrégionale

Il peut arriver qu’une reprise d’activité après sinistre déplace un serveur dans une autre région géographique. Pour les systèmes utilisant un alias DNS, il n’est pas nécessaire de rechercher et de mettre à jour la chaîne de connexion de tous les clients. Vous pouvez modifier un alias de sorte qu’il fasse référence au nouveau serveur qui héberge maintenant votre base de données Azure SQL.

## <a name="properties-of-a-dns-alias"></a>Propriétés d’un alias DNS

Les propriétés suivantes s’appliquent à tous les alias DNS du serveur :

- *Nom unique :* Chaque nom d’alias créé est unique sur tous les serveurs, tout comme les noms de serveurs.
- *Un serveur est obligatoire :* Vous ne pouvez pas créer d’alias DNS sauf s’il référence un serveur exactement et que celui-ci existe déjà. Un alias mis à jour doit toujours faire référence à un seul serveur existant.
  - En cas de suppression d’un serveur, le système Azure supprime également tous les alias DNS qui y font référence.
- *Pas de liaison à une région :* Les alias DNS ne sont pas liés à une région. Ils peuvent être mis à jour de façon à faire référence à un serveur quelle que soit sa région géographique.
  - Toutefois, en cas de référence à un autre serveur, les deux serveurs doivent se trouver dans le même *abonnement* Azure.
- *Autorisations :* Pour gérer un alias DNS, l’utilisateur doit au moins avoir l’autorisation *Contributeur de serveur*. Pour plus d’informations, consultez [Bien démarrer avec le contrôle d’accès Azure en fonction du rôle dans le portail Azure](../../role-based-access-control/overview.md).

## <a name="manage-your-dns-aliases"></a>Gérer les alias DNS

Les cmdlets PowerShell comme les API REST permettent de gérer les alias DNS par programme.

### <a name="rest-apis-for-managing-your-dns-aliases"></a>API REST pour gérer les alias DNS

La documentation des API REST est disponible près de l’emplacement web suivant :

- [API REST Azure SQL Database](/rest/api/sql/)

Par ailleurs, les API REST peuvent être consultées sur GitHub à l’adresse :

- [Azure SQL Database, alias DNS et API REST](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/sql/resource-manager/Microsoft.Sql/preview/2017-03-01-preview/serverDnsAliases.json)

<a name="anchor-powershell-code-62x"></a>

### <a name="powershell-for-managing-your-dns-aliases"></a>PowerShell pour gérer les alias DNS

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Les cmdlets PowerShell appellent les API REST.

Un exemple de code des cmdlets PowerShell utilisées pour gérer les alias DNS est documenté ici :

- [PowerShell pour les alias DNS sur Azure SQL Database](dns-alias-powershell-create.md)

Voici les cmdlets utilisées dans l’exemple de code :

- [New-AzSqlServerDNSAlias](/powershell/module/az.Sql/New-azSqlServerDnsAlias) : Crée un alias DNS dans le système de service Azure SQL Database. L’alias fait référence au serveur 1.
- [Get-AzSqlServerDNSAlias](/powershell/module/az.Sql/Get-azSqlServerDnsAlias) : Obtient et liste tous les alias DNS attribués au serveur 1.
- [Set-AzSqlServerDNSAlias](/powershell/module/az.Sql/Set-azSqlServerDnsAlias) : modifie le nom du serveur que l’alias doit référencer, de serveur 1 à serveur 2.
- [Remove-AzSqlServerDNSAlias](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias) : supprime l’alias DNS du serveur 2 en utilisant le nom de l’alias.

## <a name="limitations-during-preview"></a>Limitations de la préversion

Actuellement, un alias DNS présente les limitations suivantes :

- *Délai de 2 minutes max. :* La mise à jour ou la suppression d’un alias DNS peut prendre jusqu’à 2 minutes.
  - Indépendamment du bref délai, l’alias arrête immédiatement de faire référence aux connexions clientes au serveur hérité.
- *Recherche DNS :* Pour l’instant, le seul moyen sûr de vérifier quel serveur est référencé par un alias DNS donné est d’effectuer une [recherche DNS](/windows-server/administration/windows-commands/nslookup).
- _L’audit des tables n’est pas pris en charge :_ Vous ne pouvez pas utiliser d’alias DNS sur un serveur si l’*audit des tables* est activé sur une base de données.
  - L’audit des tables est déconseillé.
  - Nous vous recommandons de passer à [l’Audit des objets blob](../../azure-sql/database/auditing-overview.md).

## <a name="related-resources"></a>Ressources associées

- [Vue d’ensemble de la continuité d’activité avec Azure SQL Database](business-continuity-high-availability-disaster-recover-hadr-overview.md), couvrant notamment la récupération d’urgence.
- [Référence de l'API REST Azure](/rest/api/azure/)
- [API des alias DNS du serveur](/rest/api/sql/serverdnsaliases)

## <a name="next-steps"></a>Étapes suivantes

- [PowerShell pour les alias DNS sur Azure SQL Database](dns-alias-powershell-create.md)