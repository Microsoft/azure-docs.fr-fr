---
title: Concepts de forêt de ressources pour Azure AD Domain Services | Microsoft Docs
description: Découvrez ce que sont les forêts de ressources dans Azure Active Directory Domain Services et la manière dont elles bénéficient à votre organisation dans un environnement hybride avec des options d’authentification utilisateur limitées ou des problèmes de sécurité.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 79cf408bcb9060c247b97e6a81204c5a5517d384
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619999"
---
# <a name="resource-forest-concepts-and-features-for-azure-active-directory-domain-services"></a>Concepts et fonctionnalités de la forêt de ressources pour Azure Active Directory Domain Services

Azure Active Directory Domain Services (AD DS) fournit une expérience de connexion pour les applications métier héritées locales. Les utilisateurs, groupes et hachages de mot de passe des utilisateurs locaux et du cloud sont synchronisés avec le domaine géré Azure AD DS. Ces hachages de mot de passe synchronisés fournissent aux utilisateurs un ensemble unique d’informations d’identification qu’ils peuvent utiliser localement pour AD DS, Microsoft 365 et Azure Active Directory.

Bien qu’ils soient sécurisés et offrent des avantages supplémentaires en matière de sécurité, certaines organisations ne peuvent pas synchroniser les hachages des mots de passe utilisateur pour Azure AD ou Azure AD DS. Les utilisateurs d’une organisation peuvent ne pas connaître leur mot de passe, car ils utilisent uniquement l’authentification par carte à puce. Ces limitations empêchent certaines organisations d’utiliser Azure AD DS pour déplacer des applications classiques locales vers Azure.

Pour répondre à ces besoins et restrictions, vous pouvez créer un domaine managé qui utilise une forêt de ressources. Cet article conceptuel explique ce que sont les forêts et comment elles font confiance à d’autres ressources pour fournir une méthode d’authentification sécurisée.

## <a name="what-are-forests"></a>Que sont les forêts ?

Une *forêt* est une construction logique utilisée par Active Directory Domain Services (AD DS) pour regrouper un ou plusieurs *domaines*. Les domaines stockent alors les objets pour un utilisateur ou des groupes et fournissent des services d’authentification.

Dans le domaine managé Azure AD DS, la forêt ne contient qu’un seul domaine. Les forêts AD DS locales contiennent souvent de nombreux domaines. Dans les grandes organisations, en particulier après des fusions et acquisitions, vous pouvez vous retrouver avec plusieurs forêts locales qui contiennent chacune plusieurs domaines.

Par défaut, un domaine managé est créé en tant que forêt d’*utilisateurs*. Ce type de forêt synchronise tous les objets d’Azure AD, notamment les comptes d’utilisateur créés dans un environnement AD DS local. Les comptes d’utilisateur peuvent directement s’authentifier auprès du domaine managé, par exemple pour se connecter à une machine virtuelle jointe à un domaine. Une forêt d’utilisateurs fonctionne lorsque les hachages de mot de passe peuvent être synchronisés et que les utilisateurs n’utilisent pas de méthode de connexion exclusive, comme l’authentification par carte à puce.

Dans une forêt de *ressources* de domaine managé, les utilisateurs s’authentifient sur une forêt à *approbation* unique à partir de leur AD DS local. Avec cette approche, les objets utilisateur et les hachages de mot de passe ne sont pas synchronisés avec le domaine managé. Les objets utilisateur et les informations d’identification existent uniquement dans l’instance AD DS locale. Cette approche permet aux entreprises d’héberger des ressources et des plateformes d’application dans Azure qui dépendent de l’authentification classique, par exemple LDAPS, Kerberos ou NTLM, en éliminant les problèmes et craintes en matière d’authentification.

Les forêts de ressources offrent également la possibilité de déplacer vos applications un composant à la fois. De nombreuses applications locales héritées sont multicouches, utilisent souvent un serveur web ou un serveur frontal et de nombreux composants liés aux bases de données. Ces couches compliquent le déplacement de l’ensemble de l’application dans le cloud en une seule étape. Avec les forêts de ressources, vous pouvez déplacer votre application dans le cloud par phases, ce qui simplifie le déplacement de votre application vers Azure.

## <a name="what-are-trusts"></a>Que sont les approbations ?

Les organisations qui ont plusieurs domaines ont souvent besoin que les utilisateurs accèdent à des ressources partagées dans un autre domaine. Pour accéder à ces ressources partagées, les utilisateurs d’un domaine doivent s’authentifier auprès d’un autre domaine. Pour fournir ces fonctionnalités d’authentification et d’autorisation entre les clients et les serveurs dans des domaines différents, il doit y avoir une *approbation* entre les deux domaines.

Avec les approbations de domaine, les mécanismes d’authentification de chaque domaine approuvent les authentifications provenant de l’autre domaine. Les approbations permettent d’obtenir un accès contrôlé aux ressources partagées dans un domaine de ressources (le domaine *approbateur*) en vérifiant que les demandes d’authentification entrantes proviennent d’une autorité approuvée (le domaine *approuvé*). Les approbations agissent comme des ponts qui autorisent uniquement les demandes d’authentification approuvées à circuler entre les domaines.

La façon dont une approbation passe les demandes d’authentification dépend de la façon dont elle est configurée. Les approbations peuvent être configurées d’une des manières suivantes :

* **À sens unique** : fournit un accès à partir du domaine approuvé aux ressources dans le domaine approbateur.
* **Bidirectionnelle** : fournit un accès à partir de chaque domaine aux ressources de l’autre domaine.

Les approbations sont également configurées pour gérer des relations d’approbation supplémentaires d’une des manières suivantes :

* **Non transitive** : l’approbation existe uniquement entre les deux domaines approbateurs.
* **Transitive** : l’approbation s’étend automatiquement aux autres domaines approuvés par un des partenaires.

Dans certains cas, les relations d’approbation sont automatiquement établies lors de la création de domaines. Dans d’autres cas, vous devez choisir un type d’approbation et établir explicitement les relations appropriées. Les types spécifiques d’approbations utilisés et la structure de ces relations d’approbation dépendent de la façon dont le service d’annuaire Active Directory est organisé, et si les différentes versions de Windows coexistent sur le réseau.

## <a name="trusts-between-two-forests"></a>Approbations entre deux forêts

Vous pouvez étendre les approbations de domaine dans une forêt unique à une autre forêt en créant manuellement une approbation de forêt unidirectionnelle ou bidirectionnelle. Une approbation de forêt est une approbation transitive qui existe uniquement entre un domaine racine de forêt et un deuxième domaine racine de forêt.

* Une approbation de forêt unidirectionnelle permet à tous les utilisateurs d’une forêt d’approuver tous les domaines de l’autre forêt.
* Une approbation de forêt bidirectionnelle constitue une relation d’approbation transitive entre chaque domaine des deux forêts.

La transitivité des approbations de forêt est limitée aux deux partenaires de forêt. L’approbation de forêt ne s’étend pas aux forêts supplémentaires approuvées par un des partenaires.

![Diagramme d'approbation de forêt entre Azure AD DS et les instances AD DS locales](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

Vous pouvez créer différentes configurations d’approbation de domaine et de forêt en fonction de la structure Active Directory de l’organisation. Azure AD DS prend en charge uniquement une approbation de forêt unidirectionnelle. Dans cette configuration, les ressources du domaine managé peuvent approuver tous les domaines d’une forêt locale.

## <a name="supporting-technology-for-trusts"></a>Technologies de soutien pour les approbations

Les approbations utilisent différents services et fonctionnalités, comme le DNS, pour localiser les contrôleurs de domaine dans les forêts partenaires. Les approbations dépendent également des protocoles d’authentification NTLM et Kerberos, ainsi que des mécanismes d’autorisation et de contrôle d’accès basés sur Windows pour fournir une infrastructure de communication sécurisée sur les domaines et forêts Active Directory. Les services et fonctionnalités qui suivent vous aident à créer des relations d’approbation réussies.

### <a name="dns"></a>DNS

AD DS a besoin du DNS pour l’emplacement et du nom du contrôleur de domaine (DC). La prise en charge suivante de DNS est fournie pour qu’AD DS fonctionne correctement :

* Un service de résolution de noms qui permet aux hôtes et services réseau de localiser les contrôleurs réseau.
* Une structure d’affectation de noms qui permet à une entreprise de refléter sa structure organisationnelle dans les noms de ses domaines de service d’annuaire.

Un espace de noms de domaine DNS est généralement déployé et reflète l’espace de noms de domaine AD DS. S’il existe un espace de noms DNS avant le déploiement AD DS, l’espace de noms DNS est généralement partitionné pour Active Directory, et un sous-domaine DNS et une délégation pour la racine de forêt Active Directory sont créés. Des noms de domaine DNS supplémentaires sont ensuite ajoutés pour chaque domaine Active Directory enfant.

DNS est également utilisé pour prendre en charge l’emplacement des contrôleurs de domaine Active Directory. Les zones DNS sont remplies avec des enregistrements de ressources DNS qui permettent aux hôtes et services réseau de localiser les contrôleurs de domaine Active Directory.

### <a name="applications-and-net-logon"></a>Applications et ouverture de session réseau

Les applications et le service d’ouverture de session réseau sont des composants du modèle de canal de sécurité distribuée Windows. Les applications intégrées à Windows Server et à Active Directory utilisent des protocoles d’authentification pour communiquer avec le service d’ouverture de session réseau afin d’établir un chemin sécurisé sur lequel l’authentification peut se produire.

### <a name="authentication-protocols"></a>Protocoles d’authentification

Les contrôleurs Active Directory authentifient les utilisateurs et les applications à l’aide d’un des protocoles suivants :

* **Protocole d’authentification Kerberos version 5**
    * Le protocole Kerberos version 5 est le protocole d’authentification par défaut utilisé par les ordinateurs locaux exécutant Windows et prenant en charge les systèmes d’exploitation tiers. Ce protocole est spécifié dans le RFC 1510 et est entièrement intégré à Active Directory, au protocole SMB (Server Message Block), au protocole HTTP et à l’appel de procédure distante (RPC), ainsi qu’aux applications client et serveur qui utilisent ces protocoles.
    * Lorsque le protocole Kerberos est utilisé, le serveur n’a pas besoin de contacter le contrôleur de domaine. Au lieu de cela, le client obtient un ticket pour un serveur en en demandant un à partir d’un contrôleur de domaine dans le domaine du compte de serveur. Le serveur valide ensuite le ticket sans consulter aucune autre autorité.
    * Si un ordinateur impliqué dans une transaction ne prend pas en charge le protocole Kerberos version 5, le protocole NTLM est utilisé.

* **Protocole d’authentification NTLM**
    * Le protocole NTLM est un protocole d’authentification réseau classique utilisé par les systèmes d’exploitation plus anciens. Pour des raisons de compatibilité, il est utilisé par les domaines Active Directory pour traiter les demandes d’authentification réseau qui proviennent d’applications conçues pour des clients et serveurs d’anciennes versions de Windows et de systèmes d’exploitation tiers.
    * Lorsque le protocole NTLM est utilisé entre un client et un serveur, le serveur doit contacter un service d’authentification de domaine sur un contrôleur de domaine pour vérifier les informations d’identification du client. Le serveur authentifie le client en transférant les informations d’identification du client à un contrôleur de domaine dans le domaine du compte client.
    * Lorsque deux domaines ou forêts Active Directory sont connectés par une approbation, les demandes d’authentification effectuées à l’aide de ces protocoles peuvent être acheminées pour fournir un accès aux ressources dans les deux forêts.

## <a name="authorization-and-access-control"></a>Autorisation et contrôle d’accès

Les technologies d’autorisation et d’approbation fonctionnent ensemble pour fournir une infrastructure de communication sécurisée sur les domaines ou forêts Active Directory. L’autorisation détermine le niveau d’accès d’un utilisateur aux ressources d’un domaine. Les approbations facilitent l’autorisation inter-domaines des utilisateurs en fournissant un chemin d’accès pour authentifier les utilisateurs dans d’autres domaines, de sorte que leurs demandes de ressources partagées dans ces domaines peuvent être autorisées.

Lorsqu’une demande d’authentification effectuée dans un domaine approbateur est validée par le domaine approuvé, elle est transmise à la ressource cible. La ressource cible détermine ensuite s’il faut autoriser la demande spécifique faite par l’utilisateur, le service ou l’ordinateur dans le domaine approuvé en fonction de sa configuration de contrôle d’accès.

Les approbations fournissent ce mécanisme pour valider les demandes d’authentification qui sont transmises à un domaine approbateur. Les mécanismes de contrôle d’accès sur l’ordinateur de la ressource déterminent le niveau d’accès final accordé au demandeur dans le domaine approuvé.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les approbations, consultez [Comment fonctionnent les approbations de forêt dans Azure AD DS ?][concepts-trust]

Pour commencer la création d’un domaine managé Azure AD DS avec une forêt de ressources, consultez [Créer et configurer un domaine managé Azure AD][tutorial-create-advanced]. Vous pouvez ensuite [créer une approbation de forêt sortante vers un domaine local][create-forest-trust].

<!-- LINKS - INTERNAL -->
[concepts-trust]: concepts-forest-trust.md
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-forest-trust]: tutorial-create-forest-trust.md
