---
title: Haute disponibilité et récupération d’urgence
description: Apprenez à concevoir votre application Azure Batch afin de faire face à une panne régionale.
ms.topic: how-to
ms.date: 01/29/2019
ms.openlocfilehash: c77e6fa672d26545af47b262a982a7d0d7303eec
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92058010"
---
# <a name="design-your-application-for-high-availability"></a>Concevoir votre application à des fins de haute disponibilité

Azure Batch est un service régional. Batch est disponible dans toutes les régions Azure, mais lorsqu'un compte Batch est créé, il doit être associé à une région. Toutes les opérations associées au compte Batch s'appliquent alors à cette région. Par exemple, les pools et les machines virtuelles associées à ceux-ci sont créés dans la même région que le compte Batch.

Lorsque vous concevez une application qui utilise Batch, vous devez prendre en compte la possibilité que Batch ne soit pas disponible dans une région. Même si ce cas de figure est rare, il peut arriver qu'un problème touche la région toute entière ou l'ensemble du service Batch de celle-ci. Un problème peut également survenir au niveau de votre compte Batch proprement dit.

Si l'application ou la solution qui utilise Batch doit toujours être disponible, elle doit être conçue de manière à pouvoir basculer vers une autre région ou à répartir la charge de travail entre deux ou plusieurs régions. Les deux approches nécessitent au moins deux comptes Batch situés dans des régions distinctes.

## <a name="multiple-batch-accounts-in-multiple-regions"></a>Plusieurs comptes Batch dans plusieurs régions

L'utilisation de plusieurs comptes Batch situés dans des régions distinctes permet à votre application de continuer à s'exécuter si un compte Batch d'une autre région devient indisponible. L'utilisation de plusieurs comptes est particulièrement importante si votre application doit être hautement disponible.

Dans certains cas, une application peut être conçue pour utiliser en permanence deux régions ou plus. Par exemple, si vous avez besoin d'une très grande capacité, l'utilisation de plusieurs régions peut être nécessaire pour gérer une application à grande échelle ou pour faire face à une future croissance.

## <a name="design-considerations-for-providing-failover"></a>Considérations de conception pour le basculement

Pour basculer une solution vers une autre région, tous ses composants doivent être pris en compte ; un deuxième compte Batch ne suffit pas. Par exemple, dans la plupart des applications Batch, un compte de stockage Azure est requis ; et pour obtenir des performances acceptables, le compte de stockage et le compte Batch doivent se trouver dans la même région.

Lors de la conception d'une solution capable de basculer, tenez compte des points suivants :

- Créez au préalable tous les comptes requis dans les différentes régions, comme le compte Batch et le compte de stockage. Généralement, la création d'un compte est gratuite. En revanche, le stockage de données et l'utilisation du compte sont payants.
- Veillez à ce que des quotas soient définis à l'avance sur les comptes afin de pouvoir allouer le nombre requis de cœurs à l'aide du compte Batch.
- Utilisez des modèles et/ou des scripts pour automatiser le déploiement de l'application dans une région.
- Maintenez à jour les données de référence et les fichiers binaires de l'application dans toutes les régions. Une région à jour pourra être mise en ligne rapidement sans avoir à attendre le téléchargement et le déploiement des fichiers. Par exemple, si une application personnalisée à installer sur des nœuds de pool est stockée et référencée à l'aide de packages d'application Batch, lorsqu'une nouvelle version de l'application est produite, elle doit être téléchargée sur chaque compte Batch et référencée par la configuration du pool (ou la nouvelle version peut devenir la version par défaut).
- Dans l'application qui appelle Batch, Stockage ou tout autre service, basculez facilement les clients ou la charge vers l'autre région.
- Pour garantir le succès d'un basculement, il est recommandé de basculer fréquemment vers une autre région dans le cadre du fonctionnement normal. Par exemple, avec deux déploiements dans des régions distinctes, basculez une fois par mois vers l'autre région.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur la création de comptes Batch avec le [portail Azure](batch-account-create-portal.md), l’interface [Azure CLI](./scripts/batch-cli-sample-create-account.md), [PowerShell](batch-powershell-cmdlets-get-started.md) ou l’[API Gestion du service Batch](batch-management-dotnet.md).
- Des quotas par défaut sont associés à un compte Batch ; [cet article](batch-quota-limit.md) détaille les valeurs des quotas par défaut et explique comment augmenter les quotas.
