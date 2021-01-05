---
title: Vue d’ensemble des enregistrements et des zones DNS - Azure DNS
description: Vue d’ensemble de la prise en charge de l’hébergement d’enregistrements et zones DNS dans le DNS Microsoft Azure.
author: rohinkoul
ms.assetid: be4580d7-aa1b-4b6b-89a3-0991c0cda897
ms.service: dns
ms.topic: conceptual
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 12/18/2017
ms.author: rohink
ms.openlocfilehash: 41eb15a38c97532951723f12d1ac74c90c838eb1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968194"
---
# <a name="overview-of-dns-zones-and-records"></a>Vue d’ensemble des enregistrements et des zones DNS

Cette page explique les concepts clés des domaines, zones DNS, enregistrements DNS et jeux d’enregistrements, et la manière dont ces éléments sont pris en charge dans le DNS Azure.

## <a name="domain-names"></a>Noms de domaine

DNS est une hiérarchie de domaines. Celle-ci démarre à partir du domaine « racine », dont le nom est simplement «  **.**  ».  Puis viennent les domaines de niveau supérieur, tels que « com », « net », « org », « uk » ou « jp ».  Vous trouvez ensuite les domaines de second niveau, comme « org.uk » ou « co.jp ». Dans une hiérarchie DNS, les domaines sont distribués et hébergés par des serveurs de noms situés dans le monde entier.

Un bureau d’enregistrement de noms de domaine est une organisation permettant d’acheter un nom de domaine, par exemple `contoso.com`.  L’achat d’un nom de domaine vous autorise à contrôler la hiérarchie DNS sous ce nom, par exemple à diriger le nom `www.contoso.com` vers le site web de votre entreprise. Le bureau d’enregistrement peut héberger le domaine sur ses propres serveurs de noms en votre nom, ou vous autoriser À spécifier d’autres serveurs de noms.

Le DNS Azure fournit une infrastructure de serveurs de noms à haute disponibilité répartie dans le monde entier, que vous pouvez utiliser pour héberger votre domaine. En hébergeant vos domaines dans le DNS Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes éléments que vos autres services Azure : informations d’identification, API, outils, facturation et support.

Azure DNS ne prend actuellement pas en charge l'achat de noms de domaine. Si vous voulez acheter des domaines, vous devez faire appel à un bureau d’enregistrement de noms de domaine. Le bureau d’enregistrement facture généralement des frais annuels peu élevés. Les domaines peuvent être hébergés dans Azure DNS pour la gestion des enregistrements DNS. Pour plus d’informations, consultez [Déléguer un domaine à Azure DNS](dns-domain-delegation.md) .

## <a name="dns-zones"></a>Zones DNS

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

## <a name="dns-records"></a>Enregistrements DNS

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

### <a name="time-to-live"></a>Durée de vie

La durée de vie (TTL) spécifie la durée pendant laquelle chaque enregistrement est mis en cache par les clients avant d’être réinterrogé. Dans l’exemple ci-dessus, la durée de vie est de 3 600 secondes ou 1 heure.

Dans le DNS Azure, la durée de vie est spécifiée pour le jeu d’enregistrements, pas pour chaque enregistrement. La même valeur est donc utilisée pour tous les enregistrements au sein de ce jeu d’enregistrements.  Vous pouvez spécifier une valeur de durée de vie quelconque comprise entre 1 et 2 147 483 647 secondes.

### <a name="wildcard-records"></a>Enregistrements génériques

Azure DNS prend en charge les [enregistrements génériques](https://en.wikipedia.org/wiki/Wildcard_DNS_record). Ces derniers sont retournés en réponse à toute requête contenant un nom correspondant (sauf s’il existe une correspondance plus proche d’un jeu d’enregistrements non génériques). Azure DNS prend en charge des jeux d'enregistrements génériques pour tous les types d'enregistrements, hormis NS et SOA.

Pour créer un jeu d’enregistrements génériques, utilisez le nom de jeu d’enregistrements « \* ». Vous pouvez également utiliser un nom avec « \* » comme étiquette à gauche, par exemple, « \*.foo ».

### <a name="caa-records"></a>Enregistrements CAA

Un enregistrement CAA permet aux propriétaires de domaine de spécifier les autorités de certification autorisées à émettre des certificats pour leur domaine. Ainsi, les autorités de certification peuvent éviter d’émettre des certificats incorrects dans certains cas. Les enregistrements CAA présentent trois propriétés :
* **Flags** : il s’agit d’un entier compris entre 0 et 255, utilisé pour représenter l’indicateur critique qui a une signification particulière d’après la [RFC](https://tools.ietf.org/html/rfc6844#section-3).
* **Tag** : une chaîne ASCII qui peut être l’un des éléments suivants :
    * **issue** : utilisez cette option si vous souhaitez spécifier quelles autorités de certification sont autorisées à émettre des certificats (de tous types).
    * **issuewild** : utilisez cette option si vous souhaitez spécifier quelles autorités de certification sont autorisées à émettre des certificats (certificats avec caractères génériques uniquement).
    * **iodef**: spécifiez une adresse e-mail ou le nom d’hôte auquel les autorités de certification peuvent envoyer des notifications pour les demandes portant sur des certificats non autorisés.
* **Value** : valeur de la balise choisie.

### <a name="cname-records"></a>Enregistrements CNAME

Les jeux d’enregistrements CNAME ne peuvent pas coexister avec d’autres jeux d’enregistrements portant le même nom. Par exemple, vous ne pouvez pas créer un jeu d’enregistrements CNAME avec le nom relatif « www » et un enregistrement A avec le nom relatif « www » en même temps.

Étant donné que l’extrémité (apex) de la zone (nom = « \@ ») contient toujours les jeux d’enregistrements NS et SOA créés lors de la création de la zone, vous ne pouvez pas créer un jeu d’enregistrements CNAME au niveau de l’extrémité (apex) de la zone.

Ces contraintes résultent des normes DNS. Il ne s’agit pas de limites posées par le DNS Azure.

### <a name="ns-records"></a>Enregistrements NS

Le jeu d’enregistrements NS à l’extrémité (apex) de la zone (nom = « \@ ») est créé automatiquement avec chaque zone DNS, et est automatiquement supprimé lorsque la zone est supprimée (il ne peut pas être supprimé séparément).

Ce jeu d’enregistrements contient les noms des serveurs de noms Azure DNS attribués à la zone. Vous pouvez ajouter des serveurs de noms supplémentaires à ce jeu d’enregistrements NS, pour prendre en charge le co-hébergement de domaines avec plusieurs fournisseurs DNS. Vous pouvez également modifier la durée de vie et les métadonnées pour ce jeu d’enregistrements. Toutefois, vous ne pouvez pas supprimer ni modifier les serveurs de noms Azure DNS préremplis. 

Cela s’applique uniquement au jeu d’enregistrements NS défini à l’extrémité de la zone. Les autres jeux d’enregistrements NS dans votre zone (tels que ceux utilisés pour déléguer des zones enfants) peuvent être créés, modifiés et supprimés sans contrainte.

### <a name="soa-records"></a>Enregistrements SOA

Un jeu d’enregistrements SOA est créé automatiquement au sommet (apex) de chaque zone (nom = « \@ »), et est automatiquement supprimé lors de la suppression de la zone.  Il n’est pas possible de créer ou supprimer séparément des enregistrements SOA.

Vous pouvez modifier toutes les propriétés de l’enregistrement SOA, sauf la propriété « host » qui est préconfigurée pour faire référence au nom du serveur de noms principal fourni par le DNS Azure.

Le numéro de série de la zone dans l’enregistrement SOA n’est pas mis à jour automatiquement lorsque des modifications sont apportées aux enregistrements de la zone. Vous pouvez le mettre à jour manuellement en modifiant l’enregistrement SOA, si nécessaire.

### <a name="spf-records"></a>Enregistrements SPF

[!INCLUDE [dns-spf-include](../../includes/dns-spf-include.md)]

### <a name="srv-records"></a>Enregistrements SRV

Les [enregistrements SRV](https://en.wikipedia.org/wiki/SRV_record) sont utilisés par différents services pour spécifier les emplacements de serveur. Lorsque vous spécifiez un enregistrement SRV dans le DNS Azure :

* Le *service* et le *protocole* doivent être spécifiés dans le nom du jeu d’enregistrements, préfixés avec des traits de soulignement.  Par exemple, « \_sip.\_TCP.Name ».  Pour un enregistrement à l’extrémité (apex) de la zone, il est inutile de spécifier « \@ » dans son nom. Utilisez simplement le service et le protocole, par exemple, « \_sip.\_tcp ».
* La *priorité*, le *poids*, le *port* et la *cible* sont spécifiés en tant que paramètres de chaque enregistrement dans le jeu d’enregistrements.

### <a name="txt-records"></a>Enregistrements TXT

Les enregistrements TXT sont utilisés pour mapper des noms de domaine sur des chaînes de texte arbitraires. Ils sont utilisés dans plusieurs applications, en particulier celles liées à la configuration des e-mails, telles [Sender Policy Framework (SPF)](https://en.wikipedia.org/wiki/Sender_Policy_Framework) et [DomainKeys Identified Mail (DKIM)](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail).

Les normes DNS autorisent un seul enregistrement TXT à contenir plusieurs chaînes, chacune d’entre elles pouvant comprendre jusqu’à 254 caractères. Lorsque plusieurs chaînes sont utilisées, elles sont concaténées par des clients et traitées comme une chaîne unique.

Lorsque vous appelez l’API REST DNS Azure, vous devez spécifier chaque chaîne TXT séparément.  Lorsque vous utilisez le portail Azure, PowerShell ou des interfaces CLI, vous devez spécifier une chaîne unique par enregistrement, qui est automatiquement divisée en segments de 254 caractères si nécessaire.

Les chaînes multiples dans un enregistrement DNS ne doivent pas être confondues avec les enregistrements TXT multiples dans un jeu d’enregistrements TXT.  Un jeu d’enregistrements TXT peut contenir plusieurs enregistrements, *chacun d'entre eux* pouvant contenir plusieurs chaînes.  Azure DNS prend en charge une longueur totale pouvant atteindre 1 024 caractères dans chaque jeu d’enregistrements TXT (sur tous les enregistrements combinés).

## <a name="tags-and-metadata"></a>Balises et métadonnées

### <a name="tags"></a>Balises

Les balises sont une liste de paires nom-valeur. Azure Resource Manager les utilise pour étiqueter des ressources.  Azure Resource Manager utilise des balises pour vous permettre de générer des vues filtrées de votre facture Azure, et de définir une stratégie pour laquelle des balises sont requises. Pour plus d’informations sur les balises, voir [Organisation des ressources Azure à l’aide de balises](../azure-resource-manager/management/tag-resources.md).

Le DNS Azure prend en charge l’utilisation de balises Azure Resource Manager sur des ressources de zone DNS.  Il ne prend pas en charge les balises sur les jeux d’enregistrements DNS, bien que l’alternative « métadonnées » soit prise en charge sur les jeux d’enregistrements DNS comme expliqué ci-dessous.

### <a name="metadata"></a>Métadonnées

À la place de balises de jeu d’enregistrements, le DNS Azure prend en charge l’annotation des jeux d’enregistrements à l’aide de « métadonnées ».  Comme des balises, les métadonnées permettent d’associer des paires nom-valeur à chaque jeu d’enregistrements.  Cela peut être utile, par exemple, pour indiquer l’objectif de chaque jeu d’enregistrements.  Contrairement aux balises, vous ne pouvez pas utiliser les métadonnées pour produire une vue filtrée de votre facture Azure. Et vous ne pouvez pas non plus en spécifier dans une stratégie Azure Resource Manager.

## <a name="etags"></a>Etags

Supposons que deux personnes ou deux processus tentent de modifier un enregistrement DNS en même temps. Lequel gagne ? Et le gagnant sait-il qu’il a remplacé les modifications créées par quelqu’un d’autre ?

Azure DNS utilise les Etags pour gérer les modifications simultanées de la même ressource en toute sécurité. Les Etags sont différents des [« Balises » Azure Resource Manager](#tags). Chaque ressource DNS (zone ou jeu d’enregistrements) est associée à un Etag. Chaque fois qu’une ressource est récupérée, son Etag l’est également. Lors de la mise à jour d’une ressource, vous pouvez transmettre l’Etag afin qu’Azure DNS vérifie que l’Etag du serveur correspond. Étant donné que chaque mise à jour d’une ressource entraîne la régénération de l’Etag, l’absence de concordance entre les Etags indique qu’une modification simultanée a eu lieu. Les Etags sont également utilisés lorsque vous créez une ressource pour vous assurer que la ressource n’existe pas déjà.

Par défaut, Azure DNS PowerShell utilise les Etags pour bloquer les modifications simultanées apportées à des zones et des jeux d’enregistrements. Le commutateur facultatif *-Overwrite* peut être utilisé pour supprimer les vérifications d’Etags, auquel cas toutes les modifications simultanées qui se sont produites sont remplacées.

Au niveau de l’API REST Azure DNS, les Etags sont spécifiés à l’aide d’en-têtes HTTP.  Leur comportement est indiqué dans le tableau suivant :

| En-tête | Comportement |
| --- | --- |
| None |PUT réussit toujours (aucune vérification Etag) |
| If-match \<etag> |PUT ne réussit que si la ressource existe et que l’Etag correspond |
| If-match * |PUT réussit seulement si la ressource existe |
| If-none-match * |PUT réussit seulement si la ressource n’existe pas |


## <a name="limits"></a>limites

Les limites par défaut suivantes s’appliquent lors de l’utilisation du DNS Azure :

[!INCLUDE [dns-limits](../../includes/dns-limits.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour commencer à utiliser le DNS Azure, découvrez comment [créer une zone DNS](./dns-getstarted-portal.md) et [créer des enregistrements DNS](./dns-getstarted-portal.md).
* Pour migrer une zone DNS, découvrez comment [importer et exporter un fichier de zone DNS](dns-import-export.md).