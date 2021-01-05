---
title: Fichier include
description: Fichier include
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 09/10/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 262783e83bdb846a3ea7e3015212c01048a00125
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96152581"
---
Le tableau suivant répertorie les quotas et les limites propres à [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/). Pour plus d’informations sur la tarification des hubs d’événements, consultez la rubrique [Tarification des hubs d’événements](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="common-limits-for-all-tiers"></a>Limites communes à tous les niveaux
Les limites suivantes sont communes à tous les niveaux. 

| Limite |  Notes | Valeur |
| --- |  --- | --- |
| Nombre d’espaces de noms Event Hubs par abonnement |- |100 |
| Nombre d’Event Hubs par espace de noms | Les demandes suivantes de création d’un Event Hub sont rejetées. |10 |
| Nombre de partitions par Event Hub |- |32 |
| Taille du nom d’un Event Hub |- | 256 caractères |
| Taille du nom d’un groupe de consommateurs |- | 256 caractères |
| Nombre de récepteurs non epoch par groupe de consommateurs |- |5 |
| Nombre de règles d’autorisation par espace de noms | Les demandes suivantes pour la création de règle d’autorisation sont rejetées.|12 |
| Nombre d’appels à la méthode GetRuntimeInformation |  - | 50 par seconde | 
| Nombre de règles de réseau virtuel (VNet) et de configuration IP | - | 128 | 


### <a name="basic-vs-standard-tiers"></a>Niveaux de base ou standard
Le tableau suivant indique les limites qui peuvent être différentes pour les niveaux de base et standard. 

| Limite | Notes | De base | Standard |
|---|---|--|---|
| Taille maximale de l’événement Event Hubs| &nbsp; | 256 KB | 1 Mo |
| Nombre de groupes de consommateurs par Event Hub | &nbsp; |1 |20 |
| Nombre de connexions AMQP par espace de noms | Les demandes suivantes de connexions supplémentaires sont rejetées et le code appelant reçoit une exception. |100 |5 000|
| Période de rétention maximale des données d’événement | &nbsp; |1 jour |1 à 7 jours |
| Unités de débit maximales |Le dépassement de cette limite entraîne la limitation de vos données et la génération d’une exception de [serveur occupé](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception). Vous pouvez demander une plus grande quantité d’unités de débit pour le niveau Standard en remplissant une [demande de support](../articles/azure-portal/supportability/how-to-create-azure-support-request.md). Les [unités de débit supplémentaires](../articles/event-hubs/event-hubs-auto-inflate.md) sont disponibles par blocs de 20 sur la base d’un engagement d’achat ferme. |20 | 20 | 

### <a name="dedicated-tier-vs-standard-tier"></a>Niveau dédié ou niveau standard
L’offre Event Hubs Dedicated est facturée à un tarif mensuel fixe, avec un minimum de 4 heures d’utilisation. Le niveau Dedicated offre toutes les fonctionnalités du plan Standard, mais avec la capacité de mise à l’échelle de classe entreprise et les limites pour les clients avec des charges de travail exigeantes. 

Reportez-vous à ce [document](../articles/event-hubs/event-hubs-dedicated-cluster-create-portal.md) sur la création d’un cluster Event Hubs dédié à l’aide du portail Azure.

| Fonctionnalité | standard | Dédié |
| --- |:---|:---|
| Bande passante | 20 unités de débit (jusqu'à 40 unités de débit) | 20 CUS |
| Espaces de noms |  1 | 50 par unité de capacité |
| Event Hubs |  10 par espace de noms | 1 000 par espace de noms |
| Événements d’entrée | Paiement par million d’événements | Inclus |
| Taille des messages | 1 million d’octets | 1 million d’octets |
| Partitions | 32 par hub d’événements | 1 024 par hub d’événements<br/>2 000 par unité de capacité |
| Groupes de consommateurs | 20 par hub d’événements | Aucune limite par unité de capacité, 1 000 par hub d’événements |
| Connexions réparties | 1 000 inclus, 5 000 maximum | 100 000 inclus et maximum |
| Rétention des messages | 7 jours, 84 Go inclus par unité de débit | 90 jours, 10 To inclus per unité de capacité |
| Capture | Paiement par heure | Inclus |


### <a name="schema-registry-limitations"></a>Limitations du registre de schémas

#### <a name="limits-that-are-the-same-for-standard-and-dedicated-tiers"></a>Limites identiques pour les niveaux standard et dedicated 
| Fonctionnalité | Limite | 
|---|---|
| Longueur maximale du nom d’un groupe de schémas | 50 |  
| Longueur maximale du nom d’un schéma | 100 |    
| Taille en octets par schéma | 1 Mo |   
| Nombre de propriétés par groupe de schémas | 1 024 |
| Taille en octets par clé de propriété de groupe | 256 | 
| Taille en octets par valeur de propriété de groupe | 1 024 | 


#### <a name="limits-that-are-different-for-standard-and-dedicated-tiers"></a>Limites différentes pour les niveaux standard et dedicated 

| Limite | standard | Dédié | 
|---|---|--|
| Taille du registre de schémas (espace de noms) en mégaoctets | 25 |  1 024 |
| Nombre de groupes de schémas dans un registre de schémas ou espace de noms | 1 - sans compter le groupe par défaut | 1 000 |
| Nombre de versions de schéma pour tous les groupes de schémas | 25 | 10000 |