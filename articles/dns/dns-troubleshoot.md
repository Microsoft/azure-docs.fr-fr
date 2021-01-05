---
title: Guide de résolution des problèmes – Azure DNS
description: Dans ce parcours d’apprentissage, prenez en main la résolution des problèmes courants liés à Azure DNS
services: dns
author: rohinkoul
ms.service: dns
ms.topic: troubleshooting
ms.date: 09/20/2019
ms.author: rohink
ms.openlocfilehash: fae63c61949302e25c9dee2899577fa4f0d2a975
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965576"
---
# <a name="azure-dns-troubleshooting-guide"></a>Guide de résolution des problèmes d’Azure DNS

Cet article fournit des solutions aux problèmes fréquemment rencontrés avec Azure DNS.

Si cette procédure ne résout pas votre problème, vous pouvez également rechercher ou publier votre problème sur notre [page de questions Microsoft Q&A pour obtenir l’aide de la communauté](/answers/topics/azure-virtual-network.html). Vous pouvez également effectuer une demande de support Azure.


## <a name="i-cant-create-a-dns-zone"></a>Impossible de créer une zone DNS

Pour résoudre les problèmes courants, essayez une ou plusieurs des étapes suivantes :

1.  Consultez les journaux d’audit Azure DNS pour déterminer la cause de l’échec.
2.  Chaque nom de zone DNS doit être unique au sein de son groupe de ressources. Autrement dit, deux zones DNS portant le même nom ne peuvent pas partager un même groupe de ressources. Essayez d’utiliser un autre nom de zone ou un autre groupe de ressources.
3.  Il se peut que l'erreur « Vous avez atteint ou dépassé le nombre maximal de zones dans l'abonnement {subscription id} » s'affiche. Utilisez un autre abonnement Azure, supprimez certaines zones ou contactez le support Azure pour augmenter votre limite d’abonnement.
4.  Il se peut que l'erreur « La zone {Nom de la zone} n’est pas disponible » s'affiche. Cette erreur signifie qu’Azure DNS n’a pas pu allouer des serveurs de noms de cette zone DNS. Essayez d’utiliser un autre nom de zone. Sinon, si vous êtes le propriétaire du nom de domaine, vous pouvez contacter le support Azure qui peut vous allouer des serveurs de noms.


### <a name="recommended-articles"></a>Articles recommandés

* [Enregistrements et zones DNS](dns-zones-records.md)
* [Création d’une zone DNS](./dns-getstarted-portal.md)

## <a name="i-cant-create-a-dns-record"></a>Impossible de créer un enregistrement DNS

Pour résoudre les problèmes courants, essayez une ou plusieurs des étapes suivantes :

1.  Consultez les journaux d’audit Azure DNS pour déterminer la cause de l’échec.
2.  Le jeu d’enregistrements existe-t-il déjà ?  Azure DNS gère les enregistrements à l’aide de *jeux* d’enregistrement, qui sont une collection d’enregistrements ayant le même nom et le même type. Si un enregistrement ayant le même nom et le même type existe déjà, pour ajouter un autre enregistrement vous devez modifier le jeu d’enregistrements existant.
3.  Essayez-vous de créer un enregistrement à l’extrémité de la zone DNS (la « racine » de la zone) ? Si c’est le cas, la convention DNS exige d’utiliser le caractère ’@’ comme nom d’enregistrement. Notez également que les normes DNS n’autorisent pas les enregistrements CNAME au niveau de l’extrémité de la zone.
4.  Constatez-vous un conflit d’enregistrement CNAME ?  Les normes DNS n’autorisent pas l’utilisation d’un même nom pour deux enregistrements CNAME de type différent. Si vous avez un enregistrement CNAME existant, la création d’un enregistrement ayant le même nom mais de type différent échoue.  De même, la création d’un enregistrement CNAME échoue si le nom correspond à un enregistrement existant d’un autre type. Supprimez le conflit en supprimant l’autre enregistrement ou en choisissant un nom d’enregistrement différent.
5.  Vous avez atteint la limite du nombre de jeux d’enregistrements autorisé dans une zone DNS ? Le nombre actuel de jeux d’enregistrement et le nombre maximal de jeux d’enregistrements sont affichés dans le portail Azure, sous les « Propriétés » de la zone. Si vous avez atteint cette limite, supprimez des jeux d’enregistrements ou contactez le support Azure pour augmenter votre limite pour cette zone, puis réessayez. 


### <a name="recommended-articles"></a>Articles recommandés

* [Enregistrements et zones DNS](dns-zones-records.md)
* [Création d’une zone DNS](./dns-getstarted-portal.md)



## <a name="i-cant-resolve-my-dns-record"></a>Impossible de résoudre mon enregistrement DNS

La résolution de noms DNS est un processus en plusieurs étapes. Elle peut échouer pour de nombreuses raisons. Les étapes suivantes vous permettent de déterminer quelle résolution DNS échoue pour un enregistrement DNS dans une zone hébergée au sein d’Azure DNS.

1.  Vérifiez que les enregistrements DNS ont été configurés correctement dans Azure DNS. Passez en revue les enregistrements DNS dans le portail Azure, et vérifiez que le nom de la zone, le nom de l’enregistrement et le type d’enregistrement sont corrects.
2.  Vérifiez que les enregistrements DNS sont résolus correctement sur les serveurs de noms Azure DNS.
    - Si vous exécutez des requêtes DNS à partir de votre ordinateur local, vous voyez peut-être les résultats mis en cache qui ne reflètent pas l’état actuel des serveurs de noms.  En outre, les réseaux d’entreprise utilisent des serveurs proxy DNS qui empêchent de diriger les requêtes DNS vers des serveurs de noms spécifique.  Pour éviter ces problèmes, utilisez un service de résolution de noms basé sur le web tel que [digwebinterface](https://digwebinterface.com).
    - Veillez à spécifier les serveurs de noms corrects pour votre zone DNS, comme affiché dans le portail Azure.
    - Vérifiez que le nom DNS est correct (vous devez spécifier le nom complet, notamment le nom de la zone) et que le type d’enregistrement est correct
3.  Vérifiez que le nom de domaine DNS a été correctement [délégué aux serveurs de noms Azure DNS](dns-domain-delegation.md). [De nombreux sites web tiers proposent la validation de délégation DNS](https://www.bing.com/search?q=dns+check+tool). Il s’agit d’un test de délégation de *zone*. Vous devez donc uniquement entrer le nom de zone DNS et pas le nom complet d’enregistrement.
4.  Après avoir suivi les étapes ci-dessus, votre enregistrement DNS devrait se résoudre correctement. Pour vérifier, vous pouvez à nouveau utiliser [digwebinterface](https://digwebinterface.com), cette fois en vous servant des paramètres de serveur de noms par défaut.


### <a name="recommended-articles"></a>Articles recommandés

* [Délégation de domaine à Azure DNS](dns-domain-delegation.md)



## <a name="how-do-i-specify-the-service-and-protocol-for-an-srv-record"></a>Comment spécifier le « service » et le « protocole » pour un enregistrement SRV ?

Azure DNS gère les enregistrements DNS en tant que jeux d’enregistrement, la collection d’enregistrements ayant le même nom et le même type. Pour le jeu d’enregistrements SRV, le « service » et le « protocole » doivent être spécifiés dans le nom du jeu d’enregistrements. Les autres paramètres SRV (« priorité », « poids », « port » et « cible ») sont spécifiés pour chaque enregistrement dans le jeu d’enregistrements.

Exemples de noms d’enregistrement SRV (nom de service « sip », protocole « tcp ») :

- \_sip.\_tcp (crée un jeu d’enregistrements à l’extrémité de la zone)
- \_sip.\_tcp.sipservice (crée un jeu d’enregistrements nommé « sipservice »)

### <a name="recommended-articles"></a>Articles recommandés

* [Enregistrements et zones DNS](dns-zones-records.md)
* [Création de jeux d’enregistrements et d’enregistrements DNS à l’aide du portail Azure](./dns-getstarted-portal.md)
* [Type d’enregistrement SRV (Wikipedia)](https://en.wikipedia.org/wiki/SRV_record)


## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [Enregistrements et zones DNS](dns-zones-records.md)
* Pour commencer à utiliser le DNS Azure, découvrez comment [créer une zone DNS](./dns-getstarted-portal.md) et [créer des enregistrements DNS](./dns-getstarted-portal.md).
* Pour migrer une zone DNS, découvrez comment [importer et exporter un fichier de zone DNS](dns-import-export.md).