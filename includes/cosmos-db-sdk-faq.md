---
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 8/12/2020
ms.author: sngun
ms.openlocfilehash: 42d1e0f056457ba54e0102a4d23f42a81ebea08d
ms.sourcegitcommit: ac7029597b54419ca13238f36f48c053a4492cb6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/29/2020
ms.locfileid: "90068731"
---
**Comment serai-je informé du retrait du kit SDK ?**

Microsoft vous avertit 12 mois à l’avance de la fin de la prise en charge d’un kit SDK mis hors service afin de favoriser une transition en douceur vers un kit SDK pris en charge. Nous vous informons via différents canaux de communication : le portail Azure, les mises à jour Azure et une communication directe avec les administrateurs de service affectés.

**Pendant cette période de 12 mois, puis-je créer des applications à l’aide d’un kit SDK Azure Cosmos DB destiné à être mis hors service ?** 

Oui, au cours de la période de préavis de 12 mois, vous pouvez créer, déployer et modifier des applications à l’aide du kit SDK Azure Cosmos DB destiné à être mis hors service. Nous vous conseillons de migrer vers une version prise en charge plus récente du kit SDK Azure Cosmos DB pendant cette période de 12 mois, le cas échéant. 

**Après la date de mise hors service, qu’advient-il des applications qui utilisent le kit SDK Azure Cosmos DB non pris en charge ?** 

Après la date de mise hors service, Azure Cosmos DB n’apporte plus de correctifs de bogues, n’ajoute plus de nouvelles fonctionnalités et ne fournit plus de support aux versions mises hors service du kit SDK. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis les versions mises hors service du Kit de développement logiciel (SDK) continueront d’être traitées par le service Azure Cosmos DB. 

**Quelles versions du kit SDK disposent des dernières fonctionnalités et mises à jour ?**

Les nouvelles fonctionnalités et mises à jour ne sont ajoutées qu’à la dernière version mineure de la dernière version majeure prise en charge du kit SDK. Nous vous recommandons de toujours utiliser la dernière version pour tirer parti des nouvelles fonctionnalités, des améliorations des performances et des correctifs de bogues. Si vous utilisez une ancienne version du kit SDK encore en service, vos requêtes vers Azure Cosmos DB continuent de fonctionner, mais vous n’avez accès à aucune des nouvelles fonctionnalités.  

**Que faire si je ne parviens pas à mettre à jour mon application avant la date limite ?**

Nous vous recommandons de mettre à niveau vers la dernière version du kit de développement logiciel dès que possible. Une fois qu’un kit SDK est marqué pour la mise hors service, vous avez 12 mois pour mettre à jour votre application. Si vous n’êtes pas en mesure de procéder à une mise à jour avant la date de mise hors service, les requêtes envoyées à partir des versions mises hors service du kit SDK continuent d’être traitées par Azure Cosmos DB. Vos applications continuent donc de fonctionner. Toutefois, Azure Cosmos DB n’apporte plus de correctifs de bogues, n’ajoute plus de nouvelles fonctionnalités et ne fournit plus de support aux versions mises hors service du kit SDK. 

Si vous disposez d’un plan de support et avez besoin d’assistance technique, veuillez [nous contacter](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) en remplissant un ticket de support.
    


