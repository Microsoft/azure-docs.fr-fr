---
title: Fonctionnement des approbations pour Azure AD Domain Services | Microsoft Docs
description: En savoir plus sur le fonctionnement des approbations de forêt avec Azure AD Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 5c72ab7d085de558ee95f3c602ccc6be6160b322
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620203"
---
# <a name="how-trust-relationships-work-for-resource-forests-in-azure-active-directory-domain-services"></a>Fonctionnement des relations d’approbation pour les forêts de ressources dans Azure Active Directory Domain Services

Active Directory Domain Services (AD DS) assure la sécurité sur plusieurs domaines ou forêts par l’intermédiaire de relations d’approbation de domaine et de forêt. Pour que l’authentification puisse s’effectuer entre les approbations, Windows doit d’abord vérifier que le domaine demandé par un utilisateur, un ordinateur ou un service dispose d’une relation d’approbation avec le domaine du compte demandeur.

Pour vérifier cette relation d’approbation, le système de sécurité de Windows calcule un chemin d’approbation entre le contrôleur de domaine du serveur qui reçoit la demande et un contrôleur de domaine dans le domaine du compte demandeur.

Les mécanismes de contrôle d’accès fournis par AD DS et le modèle de sécurité distribué de Windows fournissent un environnement pour le fonctionnement des approbations de domaine et de forêt. Afin que ces approbations fonctionnent correctement, chaque ressource ou ordinateur doit être doté d’un chemin d’approbation direct vers un contrôleur de domaine dans le domaine où il se trouve.

Le chemin d’approbation est implémenté par le service Net Logon, au moyen d’une connexion authentifiée d’appel de procédure distante (RPC) à l’autorité de domaine approuvé. Un canal sécurisé s’étend également à d’autres domaines AD DS par le biais de relations d’approbation entre domaines. Ce canal sécurisé est utilisé pour obtenir et vérifier des informations de sécurité, entre autres les identificateurs de sécurité (SID) pour les utilisateurs et les groupes.

Pour une vue d’ensemble de la manière dont les approbations s’appliquent à Azure AD DS, reportez-vous à [Concepts et fonctionnalités de la forêt de ressources][create-forest-trust].

Pour commencer à utiliser des approbations dans Azure AD DS, [créez un domaine managé qui utilise des approbations de forêt][tutorial-create-advanced].

## <a name="trust-relationship-flows"></a>Flux des relations d’approbation

Le flux des communications sécurisées sur les approbations conditionne l’élasticité d’une approbation. La façon dont vous créez ou configurez une approbation détermine l’étendue de la communication à l’intérieur d’une forêt, ou entre les forêts.

Le flux de la communication sur les approbations est déterminé par le sens de l’approbation. Les approbations peuvent être unidirectionnelles ou bidirectionnelles, transitives ou non transitives.

Le schéma suivant montre que tous les domaines de l’*arborescence 1* et de l’*arborescence 2* présentent des relations d’approbation transitive par défaut. Ainsi, les utilisateurs de l’*arborescence 1* peuvent accéder aux ressources des domaines de l’*arborescence 2*, et les utilisateurs de l’*arborescence 1* peuvent accéder aux ressources de l’*arborescence 2*, lorsque les autorisations appropriées sont affectées au niveau de la ressource.

![Diagramme des relations d’approbation entre deux forêts](./media/concepts-forest-trust/trust-relationships.png)

### <a name="one-way-and-two-way-trusts"></a>Approbations unidirectionnelles et bidirectionnelles

Les relations d’approbation permettant l’accès aux ressources peuvent être unidirectionnelles ou bidirectionnelles.

Une approbation unidirectionnelle est un chemin d’authentification à sens unique, créé entre deux domaines. Dans une approbation unidirectionnelle entre le *domaine A* et le *domaine B*, les utilisateurs du *domaine A* peuvent accéder aux ressources du *domaine B*. En revanche, les utilisateurs du *domaine B* ne peuvent pas accéder aux ressources du *domaine A*.

Certaines approbations unidirectionnelles peuvent être non transitives ou transitives, selon le type d’approbation créé.

Dans une approbation bidirectionnelle, le *domaine A* approuve le *domaine B*, et le *domaine B* approuve le *domaine A*. Cette configuration signifie que les demandes d’authentification peuvent être transmises entre les deux domaines, dans les deux sens. Certaines relations bidirectionnelles peuvent être non transitives ou transitives, selon le type d’approbation créé.

Toutes les approbations de domaine dans une forêt AD DS sont des approbations transitives bidirectionnelles. Lorsqu’un domaine enfant est créé, une approbation transitive bidirectionnelle est automatiquement créée entre le nouveau domaine enfant et le domaine parent.

### <a name="transitive-and-non-transitive-trusts"></a>Approbations transitives et non transitives

La transitivité détermine si une approbation peut être étendue en dehors des deux domaines avec lesquels elle a été formée.

* Une approbation transitive peut être utilisée pour étendre les relations d’approbation à d’autres domaines.
* Une approbation non transitive peut être utilisée pour refuser des relations d’approbation à d’autres domaines.

Chaque fois que vous créez un domaine dans une forêt, une relation d’approbation transitive bidirectionnelle est automatiquement créée entre le nouveau domaine et son domaine parent. Si des domaines enfants sont ajoutés au nouveau domaine, le chemin d’approbation remonte la hiérarchie de domaine, étendant le chemin d’approbation initial créé entre le nouveau domaine et son domaine parent. Les relations d’approbation transitive remontent dans une arborescence de domaine au fur et à mesure qu’elle se forme, créant ainsi des approbations transitives entre tous les domaines de l’arborescence de domaine.

Les demandes d’authentification suivent ces chemins d’approbation, si bien que les comptes de n’importe quel domaine de la forêt peuvent être authentifiés par n’importe quel autre domaine de la forêt. Avec un processus d’authentification unique, les comptes disposant des autorisations appropriées peuvent accéder aux ressources de n’importe quel domaine de la forêt.

## <a name="forest-trusts"></a>Approbations de forêts

Les approbations de forêt vous aident à gérer les infrastructures AD DS segmentées, et à prendre en charge l’accès aux ressources et à d’autres objets dans plusieurs forêts. Les approbations de forêt sont utiles pour les fournisseurs de services, les sociétés expérimentant fusions ou acquisitions, les extranets collaboratifs d’entreprise et les sociétés cherchant une solution d’autonomie administrative.

En vous appuyant sur les approbations de forêt, vous pouvez lier deux forêts différentes pour former une relation d’approbation transitive unidirectionnelle ou bidirectionnelle. Une approbation de forêt permet aux administrateurs de connecter deux forêts AD DS au moyen d’une seule relation d’approbation, pour fournir une expérience d’authentification et d’autorisation transparente dans les forêts.

Une approbation de forêt peut uniquement être créée entre un domaine racine de forêt dans une forêt et un domaine racine de forêt dans une autre forêt. Il n’est possible de créer des approbations de forêt qu’entre deux forêts, et celles-ci ne peuvent pas être étendues implicitement à une troisième forêt. Ce comportement signifie que si une approbation de forêt est créée entre la *forêt 1* et la *forêt 2*, et qu’une autre approbation de forêt est créée entre la *forêt 2* et la *forêt 3*, la *forêt 1* n’entretient aucune relation d’approbation implicite avec la *forêt 3*.

Le schéma suivant illustre deux relations d’approbation de forêt distinctes entre trois forêts AD DS d’une même organisation.

![Diagramme des relations d’approbation de forêt au sein d’une même organisation](./media/concepts-forest-trust/forest-trusts-diagram.png)

Cet exemple de configuration fournit l’accès suivant :

* Les utilisateurs de la *forêt 2* peuvent accéder aux ressources de n’importe quel domaine de la *forêt 1* ou de la *forêt 3*
* Les utilisateurs de la *forêt 3* peuvent accéder aux ressources de n’importe quel domaine de la *forêt 2*
* Les utilisateurs de la *forêt 1* peuvent accéder aux ressources de n’importe quel domaine de la *forêt 2*

Cette configuration n’autorise pas les utilisateurs de la *forêt 1* à accéder aux ressources de la *forêt 3*, ou inversement. Pour permettre aux utilisateurs de la *forêt 1* et de la *forêt 3* de partager des ressources, vous devez créer une approbation transitive bidirectionnelle entre ces deux forêts.

Si une approbation de forêt unidirectionnelle est créée entre deux forêts, les membres de la forêt approuvée peuvent utiliser les ressources situées dans la forêt d’approbation. Par contre, l’approbation ne fonctionne que dans un seul sens.

Par exemple, lorsqu’une approbation de forêt unidirectionnelle est créée entre la *forêt 1* (la forêt approuvée) et la *forêt 2* (la forêt d’approbation) :

* Les membres de la *forêt 1* peuvent accéder aux ressources situées dans la *forêt 2*.
* Les membres de la *forêt 2* ne peuvent pas accéder aux ressources situées dans la *forêt 1* au moyen de la même approbation.

> [!IMPORTANT]
> La forêt de ressources Azure AD Domain Services ne prend en charge qu’une approbation de forêt unidirectionnelle vers Active Directory local.

### <a name="forest-trust-requirements"></a>Conditions de l’approbation de forêt

Avant de pouvoir créer une approbation de forêt, vous devez vérifier que vous disposez de l’infrastructure DNS (Domain Name System) appropriée. Les approbations de forêt ne peuvent être créées que si l’une des configurations DNS suivantes est disponible :

* Un serveur DNS racine unique est le serveur DNS racine pour les deux espaces de noms DNS de forêt : la zone racine contient les délégations de chaque espace de noms DNS, et les indications de racine de tous les serveurs DNS incluent le serveur DNS racine.
* En l’absence de serveur DNS racine partagé, les serveurs DNS racines de chaque espace de noms DNS de forêt utilisent des redirecteurs conditionnels DNS, afin que chaque espace de noms DNS route les requêtes de noms dans l’autre espace de noms.

    > [!IMPORTANT]
    > La forêt de ressources Azure AD Domain Services doit utiliser cette configuration DNS. L’hébergement d’un espace de noms DNS autre que l’espace de noms DNS de la forêt de ressources n’est pas une fonctionnalité d’Azure AD Domain Services. L’utilisation de redirecteurs conditionnels constitue la configuration appropriée.

* En l’absence de serveur DNS racine partagé, les serveurs DNS racines de chaque espace de noms DNS de forêt utilisent des zones secondaires DNS, configurées dans chaque espace de noms DNS pour router les requêtes de noms dans l’autre espace de noms.

Pour créer une approbation de forêt, vous devez être membre du groupe Admins du domaine (dans le domaine racine de la forêt), ou du groupe Administrateurs de l’entreprise dans Active Directory. Chaque approbation se voit attribuer un mot de passe que les administrateurs des deux forêts doivent connaître. Les membres administrateurs de l’entreprise dans les deux forêts peuvent créer les approbations dans les deux forêts à la fois et, dans ce scénario, un mot de passe aléatoire par chiffrement est généré et écrit automatiquement pour les deux forêts.

L’approbation de forêt sortante pour Azure AD Domain Services est créée dans le portail Azure. Vous ne créez pas l’approbation manuellement avec le domaine managé proprement dit. L’approbation de forêt entrante doit être configurée par un utilisateur disposant des privilèges précédemment indiqués dans l’instance Active Directory locale.

## <a name="trust-processes-and-interactions"></a>Interactions et processus d’approbation

De nombreuses transactions inter-domaines et inter-forêts dépendent des approbations de domaine ou de forêt pour effectuer diverses tâches. Cette section décrit les processus et les interactions qui se produisent lorsque des ressources sont accessibles dans les approbations et que les références d’authentification sont évaluées.

### <a name="overview-of-authentication-referral-processing"></a>Vue d’ensemble du traitement de la référence d’authentification

Lorsqu’une demande d’authentification est référencée dans un domaine, le contrôleur de domaine de ce domaine doit déterminer si une relation d’approbation existe avec le domaine à partir duquel la demande est envoyée. Le sens de l’approbation et son type, transitif ou non transitif, doivent également être déterminés avant d’authentifier l’utilisateur pour l’accès aux ressources du domaine. Le processus d’authentification qui se produit entre des domaines approuvés varie en fonction du protocole d’authentification utilisé. Les protocoles Kerberos V5 et NTLM traitent différemment les références de l’authentification sur un domaine.

### <a name="kerberos-v5-referral-processing"></a>Traitement de la référence Kerberos V5

Le protocole d’authentification Kerberos V5 dépend du service Net Logon sur les contrôleurs de domaine pour les informations d’authentification et d’autorisation du client. Le protocole Kerberos se connecte à un centre de distribution de clés (KDC) en ligne, et au magasin de comptes Active Directory pour les tickets de session.

Le protocole Kerberos utilise également des approbations pour les services d’accord de tickets (TGS) inter-domaines, et pour valider des certificats PAC (Privilege Attribute Certificate) sur un canal sécurisé. Le protocole Kerberos procède à une authentification inter-domaine uniquement auprès des domaines Kerberos dotés de systèmes d’exploitation non Windows, tels qu’un domaine Kerberos MIT ; il n’a pas besoin d’interagir avec le service Net Logon.

Si le client utilise Kerberos V5 pour l’authentification, il demande un ticket au serveur dans le domaine cible, à partir d’un contrôleur de domaine dans son domaine de compte. Le KDC Kerberos agit comme intermédiaire approuvé entre le client et le serveur, et fournit une clé de session qui permet aux deux parties de s’authentifier mutuellement. Si le domaine cible est différent du domaine actuel, le KDC suit un processus logique permettant de déterminer si une demande d’authentification peut être référencée :

1. Le domaine actuel est-il approuvé directement par le domaine du serveur qui est demandé ?
    * Si c’est le cas, envoyez au client une référence au domaine demandé.
    * Sinon, passez à l’étape suivante.

2. Existe-t-il une relation d’approbation transitive entre le domaine actuel et le prochain domaine sur le chemin d’approbation ?
    * Si c’est le cas, envoyez au client une référence au prochain domaine sur le chemin d’approbation.
    * Sinon, envoyez un message d’authentification refusée au client.

### <a name="ntlm-referral-processing"></a>Traitement de la référence NTLM

Le protocole d’authentification NTLM dépend du service Net Logon sur les contrôleurs de domaine pour les informations d’authentification et d’autorisation du client. Ce protocole authentifie les clients qui n’utilisent pas l’authentification Kerberos. NTLM utilise des approbations pour passer les demandes d’authentification entre les domaines.

Si le client utilise NTLM pour l’authentification, la demande initiale d’authentification est directement envoyée, du client au serveur de ressources dans le domaine cible. Ce serveur crée un défi auquel le client répond. Le serveur envoie ensuite la réponse de l’utilisateur à un contrôleur de domaine dans son domaine de comptes d’ordinateur. Ce contrôleur de domaine vérifie le compte d’utilisateur par rapport à sa base de données de comptes de sécurité.

Si le compte n’existe pas dans la base de données, le contrôleur de domaine détermine s’il faut procéder à l’authentification directe, transférer la demande ou la refuser à l’aide de la logique suivante :

1. Le domaine actuel a-t-il une relation d’approbation directe avec le domaine de l’utilisateur ?
    * Si c’est le cas, le contrôleur de domaine envoie les informations d’identification du client à un contrôleur de domaine dans le domaine de l’utilisateur pour l’authentification directe.
    * Sinon, passez à l’étape suivante.

2. Le domaine actuel a-t-il une relation d’approbation transitive avec le domaine de l’utilisateur ?
    * Si c’est le cas, transmettez la demande d’authentification au domaine suivant dans le chemin d’approbation. Ce contrôleur de domaine répète le processus en vérifiant les informations d’identification de l’utilisateur par rapport à sa propre base de données de comptes de sécurité.
    * Sinon, envoyez un message de connexion refusée au client.

### <a name="kerberos-based-processing-of-authentication-requests-over-forest-trusts"></a>Traitement Kerberos des demandes d’authentification sur les approbations de forêt

Lorsque deux forêts sont connectées par une approbation de forêt, les demandes d’authentification effectuées à l’aide des protocoles NTLM ou Kerberos V5 peuvent être acheminées entre les forêts pour fournir un accès aux ressources dans les deux forêts.

Lorsqu’une approbation de forêt est établie pour la première fois, chaque forêt collecte tous les espaces de noms approuvés dans sa forêt partenaire, et stocke les informations dans un [objet domaine approuvé](#trusted-domain-object). Les espaces de noms approuvés incluent les noms d’arborescence de domaine, les suffixes de nom d’utilisateur principal (UPN), les suffixes de nom de principal du service (SPN) et les espaces de noms d’ID de sécurité (SID) utilisés dans l’autre forêt. Les objets domaine approuvé (TDO) sont répliqués dans le catalogue global.

Avant que les protocoles d’authentification puissent suivre le chemin d’approbation de la forêt, le nom de principal du service (SPN) de l’ordinateur de la ressource doit être résolu en un emplacement dans l’autre forêt. Un principal du service peut porter l’un des noms suivants :

* le nom DNS d’un hôte ;
* le nom DNS d’un domaine ;
* le nom unique d’un objet point de connexion de service.

Quand une station de travail dans une forêt tente d’accéder à des données sur un ordinateur de ressource dans une autre forêt, le processus d’authentification Kerberos contacte le contrôleur de domaine pour obtenir un ticket de service vers le SPN de l’ordinateur de la ressource. Une fois que le contrôleur de domaine a interrogé le catalogue global et déterminé que le SPN n’est pas dans la même forêt que le contrôleur de domaine, le contrôleur de domaine renvoie une référence pour son domaine parent à la station de travail. À ce stade, la station de travail interroge le domaine parent pour obtenir le ticket de service, et continue de suivre la chaîne de référence jusqu’à atteindre le domaine dans lequel se trouve la ressource.

Le schéma et les étapes ci-dessous fournissent une description détaillée du processus d’authentification Kerberos qui est utilisé lorsque des ordinateurs exécutant Windows essaient d’accéder aux ressources depuis un ordinateur situé dans une autre forêt.

![Diagramme du processus Kerberos sur une approbation de forêt](media/concepts-forest-trust/kerberos-over-forest-trust-process-diagram.png)

1. *Utilisateur1* s’authentifie sur *StationDeTravail1* à l’aide des informations d’identification depuis le domaine *europe.tailspintoys.com*. L’utilisateur tente ensuite d’accéder à une ressource partagée sur le *ServeurDeFichiers1* situé dans la forêt *usa.wingtiptoys.com*.

2. La *StationDeTravail1* contacte le KDC Kerberos sur un contrôleur de domaine de son domaine, le *CDEnfant1*, et demande un ticket de service pour obtenir le SPN du *ServeurDeFichiers1*.

3. Le *CDEnfant1* ne trouve pas le SPN dans sa base de données de domaine et interroge le catalogue global pour voir si des domaines de la forêt *tailspintoys.com* contiennent ce SPN. Étant donné qu’un catalogue global est limité à sa propre forêt, le SPN est introuvable.

    Le catalogue global consulte ensuite sa base de données pour obtenir des informations sur les approbations de forêt établies avec sa forêt. S’il en trouve, il compare les suffixes de nom figurant dans l’objet TDO de l’approbation de forêt au suffixe du SPN cible pour trouver une correspondance. Lorsqu’une correspondance est trouvée, le catalogue global retourne une indication de routage au *CDEnfant1*.

    Les indications de routage permettent de diriger les demandes d’authentification vers la forêt de destination. Les indications sont utilisées uniquement lorsque tous les canaux d’authentification classiques, tels que le contrôleur de domaine local, puis le catalogue global, ne parviennent pas à localiser un SPN.

4. Le *CDEnfant1* renvoie une référence pour son domaine parent à la *StationDeTravail1*.

5. La *StationDeTravail1* contacte un contrôleur de domaine dans le *CDRacineDeForêt1* (son domaine parent) pour obtenir une référence à un contrôleur de domaine (*CDRacineDeForêt2*) dans le domaine racine de forêt de la forêt *wingtiptoys.com*.

6. La *StationDeTravail1* contacte le *CDRacineDeForêt2* dans la forêt *wingtiptoys.com* pour obtenir un ticket de service vers le service demandé.

7. Le *CDRacineDeForêt2* contacte son catalogue global à la recherche du SPN ; le catalogue global trouve une correspondance pour le SPN et la renvoie au *CDRacineDeForêt2*.

8. Le *CDRacineDeForêt2* retourne ensuite la référence vers *usa.wingtiptoys.com* à la *StationDeTravail1*.

9. La *StationDeTravail1* contacte le KDC sur le *CDEnfant2* et négocie le ticket pour que l’*Utilisateur1* puisse accéder au *ServeurDeFichiers1*.

10. Une fois que la *StationDeTravail1* dispose d’un ticket de service, elle envoie ce ticket de service au *ServeurDeFichiers1* qui lit les informations d’identification de sécurité de l’*Utilisateur1* et élabore un jeton d’accès en conséquence.

## <a name="trusted-domain-object"></a>Objet domaine approuvé

Chaque approbation de domaine ou de forêt au sein d’une organisation est représentée par un objet domaine approuvé (TDO) stocké dans le conteneur *Système* au sein de son domaine.

### <a name="tdo-contents"></a>Contenu TDO

Les informations contenues dans un objet TDO varient selon que cet objet a été créé par une approbation de domaine ou par une approbation de forêt.

Lors de la création d’une approbation de domaine, des attributs, tels que le nom de domaine DNS, le SID de domaine, le type d’approbation, la transitivité de l’approbation et le nom de domaine réciproque sont représentés dans l’objet TDO. Les objets TDO d’approbation de forêt stockent des attributs supplémentaires pour identifier tous les espaces de noms approuvés à partir de la forêt partenaire. Ces attributs incluent les noms d’arborescence de domaine, les suffixes de nom d’utilisateur principal (UPN), les suffixes de nom de principal du service (SPN) et les espaces de noms d’ID de sécurité (SID).

Étant donné que les approbations sont stockées dans Active Directory sous la forme d’objets TDO, tous les domaines d’une forêt ont connaissance des relations d’approbation en place dans l’ensemble de la forêt. De même, lorsque deux ou plusieurs forêts sont jointes par l’intermédiaire d’approbations de forêt, les domaines racines de forêt dans chaque forêt ont connaissance des relations d’approbation qui sont en place dans tous les domaines des forêts approuvées.

### <a name="tdo-password-changes"></a>Modification du mot de passe TDO

Les deux domaines d’une relation d’approbation partagent un mot de passe, celui-ci est stocké dans l’objet TDO, dans Active Directory. Dans le cadre du processus de maintenance des comptes, tous les 30 jours, le contrôleur du domaine d’approbation modifie le mot de passe stocké dans l’objet TDO. Dans la mesure où toutes les approbations bidirectionnelles sont en fait deux approbations unidirectionnelles allant dans des directions opposées, le processus se produit deux fois pour les approbations bidirectionnelles.

Une approbation comprend une partie d’approbation et une partie approuvée. Dans la partie approuvée, tout contrôleur de domaine accessible en écriture peut être utilisé pour le processus. Dans la partie d’approbation, l’émulateur de contrôleur de domaine principal procède à la modification du mot de passe.

Pour modifier un mot de passe, les contrôleurs de domaine suivent le processus décrit ici :

1. L’émulateur de contrôleur de domaine principal dans le domaine d’approbation crée un nouveau mot de passe. Un contrôleur de domaine dans le domaine approuvé ne lance jamais la modification du mot de passe. Elle est toujours initiée par l’émulateur de contrôleur de domaine principal du domaine d’approbation.

2. L’émulateur de contrôleur de domaine principal dans le domaine d’approbation définit le champ *OldPassword* de l’objet TDO sur le champ *NewPassword* actuel.

3. L’émulateur de contrôleur de domaine principal dans le domaine d’approbation affecte le nouveau mot de passe au champ *NewPassword* de l’objet TDO. En conservant une copie du mot de passe précédent, vous pouvez revenir à l’ancien mot de passe si jamais le contrôleur de domaine du domaine approuvé ne parvient pas à recevoir la modification, ou si la modification n’est pas répliquée avant qu’une demande utilisant le nouveau mot de passe d’approbation soit effectuée.

4. L’émulateur de contrôleur de domaine principal dans le domaine d’approbation effectue un appel distant à un contrôleur de domaine dans le domaine approuvé, afin de lui demander de définir le mot de passe du compte d’approbation sur le nouveau mot de passe.

5. Le contrôleur de domaine dans le domaine approuvé remplace le mot de passe d’approbation par le nouveau mot de passe.

6. Dans chaque partie de l’approbation, les mises à jour sont répliquées sur les autres contrôleurs de domaine dans le domaine. Dans le domaine d’approbation, la modification déclenche une réplication urgente de l’objet domaine approuvé.

Le mot de passe est désormais modifié sur les deux contrôleurs de domaine. La réplication normale distribue les objets TDO aux autres contrôleurs de domaine dans le domaine. Toutefois, il est possible que le contrôleur de domaine dans le domaine d’approbation modifie le mot de passe sans parvenir à correctement mettre à jour un contrôleur de domaine dans le domaine approuvé. Ce scénario peut se produire à cause d’un canal sécurisé, exigé dans le processus de modification du mot de passe, et qui n’a pas pu être établi. Il est également possible que le contrôleur de domaine dans le domaine approuvé ne soit pas disponible à un moment donné dans le processus, et qu’il ne reçoive pas le mot de passe mis à jour.

Pour gérer les situations dans lesquelles la modification du mot de passe n’est pas convenablement communiquée, le contrôleur de domaine dans le domaine d’approbation ne modifie jamais le nouveau mot de passe, sauf s’il a été dûment authentifié (configuration d’un canal sécurisé) à l’aide du nouveau mot de passe. Ce comportement explique pourquoi l’ancien et le nouveau mot de passe sont conservés dans l’objet TDO du domaine d’approbation.

Une modification de mot de passe n’est pas finalisée tant que l’authentification utilisant le mot de passe n’est pas réussie. L’ancien mot de passe stocké peut être utilisé sur le canal sécurisé jusqu’à ce que le contrôleur de domaine du domaine approuvé reçoive le nouveau mot de passe, ce qui permet de bénéficier d’un service sans interruption.

Si l’authentification avec le nouveau mot de passe échoue, à cause du mot de passe non valide, le contrôleur de domaine d’approbation tente de procéder à l’authentification à l’aide de l’ancien mot de passe. S’il s’authentifie correctement avec l’ancien mot de passe, il reprend le processus de modification du mot de passe dans un délai de 15 minutes.

Les mises à jour des mots de passe d’approbation doivent être répliquées dans un délai de 30 jours sur les contrôleurs de domaine des deux parties de l’approbation. Si le mot de passe d’approbation est modifié après ce délai de 30 jours, et qu’un contrôleur de domaine ne possède que le mot de passe N-2, il ne peut pas utiliser l’approbation depuis la partie d’approbation, et il ne peut pas créer de canal sécurisé dans la partie approuvée.

## <a name="network-ports-used-by-trusts"></a>Ports réseau utilisés par les approbations

Dans la mesure où les approbations doivent être déployées sur plusieurs limites du réseau, elles peuvent être amenées à s’étendre sur un ou plusieurs pare-feu. Lorsque c’est le cas, vous pouvez transmettre par tunnel le trafic d’approbation à travers un pare-feu, ou ouvrir des ports spécifiques dans le pare-feu pour autoriser le trafic.

> [!IMPORTANT]
> Active Directory Domain Services ne prend pas en charge la restriction du trafic RPC Active Directory sur des ports spécifiques.

Pour en savoir plus sur les ports nécessaires dans une approbation de forêt, consultez la section **Windows Server 2008 et versions ultérieures** de l’article du Support Microsoft [Guide pratique de configuration d’un pare-feu pour les domaines et les approbations Active Directory](https://support.microsoft.com/help/179442/how-to-configure-a-firewall-for-domains-and-trusts).

## <a name="supporting-services-and-tools"></a>Services et outils associés

Pour prendre en charge les approbations et l’authentification, quelques fonctionnalités et outils de gestion supplémentaires sont utilisés.

### <a name="net-logon"></a>Net Logon

Le service Net Logon gère un canal sécurisé, depuis un ordinateur Windows vers un contrôleur de domaine. Il est également utilisé dans les processus suivants, liés à l’approbation :

* Gestion et configuration de l’approbation - Net Logon permet de conserver les mots de passe d’approbation, de collecter les informations d’approbation et de vérifier les approbations en interagissant avec le processus LSA et l’objet domaine approuvé.

    Pour les approbations de forêt, les informations d’approbation incluent l’enregistrement des informations d’approbation de forêt (*FTInfo*) ; celui-ci comprend l’ensemble des espaces de noms qu’une forêt approuvée entend gérer, annoté à l’aide d’un champ indiquant si chaque revendication est approuvée par la forêt d’approbation.

* Authentification - Fournit sur un canal sécurisé les informations d’identification de l’utilisateur à un contrôleur de domaine, et retourne pour cet utilisateur les SID de domaine et les droits d’utilisateur.

* Emplacement du contrôleur de domaine - Permet de rechercher ou de localiser les contrôleurs de domaine dans un ou plusieurs domaines.

* Validation directe - Les informations d’identification des utilisateurs dans d’autres domaines sont traitées par Net Logon. Lorsqu’un domaine d’approbation doit vérifier l’identité d’un utilisateur, il transmet pour contrôle, via Net Logon, les informations d’identification de l’utilisateur au domaine approuvé.

* Vérification du certificat PAC (Privilege Attribute Certificate) - Lorsqu’un serveur utilisant le protocole Kerberos pour l’authentification doit vérifier le certificat PAC dans un ticket de service, il envoie, pour contrôle, ce certificat à son contrôleur de domaine sur le canal sécurisé.

### <a name="local-security-authority"></a>Autorité de sécurité locale

L’autorité de sécurité locale (LSA) est un sous-système protégé qui conserve des informations se rapportant à tous les aspects de la sécurité locale sur un système. Regroupés sous l’appellation de « stratégie de sécurité locale », l’autorité LSA fournit différents services de traduction entre les noms et les identificateurs.

Le sous-système de sécurité LSA fournit des services en mode noyau et en mode utilisateur pour valider l’accès aux objets, vérifier les privilèges de l’utilisateur et générer des messages d’audit. LSA est chargé de vérifier la validité de tous les tickets de session présentés par les services, dans des domaines approuvés ou non approuvés.

### <a name="management-tools"></a>Outils d'administration

Les administrateurs peuvent utiliser *Domaines et approbations Active Directory*, *Netdom* et *Nltest* pour exposer, créer, supprimer ou modifier des approbations.

* *Domaines et approbations Active Directory* est la console MMC (Microsoft Management Console) qui est utilisée pour gérer les approbations de domaine, les niveaux fonctionnels de domaine et de forêt et les suffixes de nom d’utilisateur principal.
* Les outils en ligne de commande *Netdom* et *Nltest* peuvent être utilisés pour rechercher, afficher, créer et gérer les approbations. Ces outils communiquent directement avec l’autorité LSA sur un contrôleur de domaine.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les forêts de ressources, consultez [Fonctionnement des approbations de forêt dans Azure AD DS][concepts-trust]

Pour prendre en main la création d’un domaine managé avec une forêt de ressources, consultez [Créer et configurer un domaine managé Azure AD DS][tutorial-create-advanced]. Vous pouvez ensuite [créer une approbation de forêt sortante vers un domaine local][create-forest-trust].

<!-- LINKS - INTERNAL -->
[concepts-trust]: concepts-forest-trust.md
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-forest-trust]: tutorial-create-forest-trust.md
