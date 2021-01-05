---
title: 'Azure AD Connect : principes de conception | Microsoft Docs'
description: Cette rubrique détaille certains aspects de la conception de l’implémentation
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 4114a6c0-f96a-493c-be74-1153666ce6c9
ms.service: active-directory
ms.custom: azure-ad-connect
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 08/10/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: baa03499cc11bda24ead986dd64621572484cbb1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89279650"
---
# <a name="azure-ad-connect-design-concepts"></a>Principes de conception Azure AD Connect
L’objectif de ce document est de décrire les principes qui doivent présider à la conception de l’implémentation d’Azure AD Connect. Il s’agit d’une exploration approfondie de certains aspects. Ces concepts sont également décrits brièvement dans d’autres documents.

## <a name="sourceanchor"></a>sourceAnchor
L’attribut sourceAnchor est défini en tant qu’ *attribut immuable pendant la durée de vie d’un objet*. Il identifie de façon univoque un objet comme étant le même objet local et dans Azure AD. L’attribut est également appelé **immutableId** et les deux noms sont interchangeables.

Le mot « immuable », signifiant « qui ne peut pas être changé », est important dans ce document. Étant donné que la valeur de cet attribut ne peut pas être changée une fois qu’elle a été définie, il est important de choisir une conception qui prend en charge votre scénario.

L’attribut est utilisé pour les scénarios suivants :

* Quand un nouveau serveur de moteur de synchronisation est créé ou recréé après un scénario de récupération d’urgence, cet attribut lie les objets existants dans Azure AD à des objets locaux.
* Si vous passez d’une identité de cloud uniquement à un modèle d’identité synchronisé, alors cet attribut permet une correspondance exacte et concrète des objets existants dans Azure AD avec des objets locaux.
* Si vous utilisez la fédération, alors cet attribut est utilisé avec **userPrincipalName** dans la revendication pour identifier un utilisateur de façon univoque.

Cette rubrique traite de sourceAnchor seulement par rapport aux utilisateurs. Les mêmes règles s’appliquent à tous les types d’objets, mais le problème ne se pose généralement que pour des utilisateurs.

### <a name="selecting-a-good-sourceanchor-attribute"></a>Sélection d’un attribut sourceAnchor approprié
La valeur de l’attribut doit respecter les règles suivantes :

* sa longueur doit être inférieure à 60 caractères
  * les caractères autres que a-z, A-Z ou 0-9 sont codés et comptabilisés comme 3 caractères
* elle ne doit pas contenir les caractères spéciaux suivants : &#92; ! # $ % & * + / = ? ^ &#96; { } | ~ < > ( ) ' ; : , [ ] " \@ _
* elle doit être globalement unique
* elle doit être une chaîne, un entier ou une valeur binaire
* elle ne doit pas être basée sur le nom de l’utilisateur, car celui-ci est susceptible de changer
* elle ne doit pas respecter la casse et doit éviter les valeurs qui peuvent varier selon la casse
* elle doit être assignée lorsque l’objet est créé.

Si le sourceAnchor sélectionné n’est pas de type chaîne, alors Azure AD Connect encode la valeur de l’attribut en base 64 pour garantir qu’aucun caractère spécial n’apparaît. Si vous utilisez un serveur de fédération autre qu’ADFS, assurez-vous que votre serveur a également la capacité d’encoder la valeur de l’attribut en base 64.

L’attribut sourceAnchor respecte la casse. La valeur « JohnDoe » n’est pas le même que « johndoe ». Mais vous ne devez pas avoir deux objets différents avec uniquement une différence de casse.

Si vous avez une seule forêt locale, alors l’attribut que vous devez utiliser est **objectGUID**. C’est également l’attribut utilisé quand vous utilisez la configuration rapide dans Azure AD Connect, ainsi que l’attribut utilisé par DirSync.

Si vous avez plusieurs forêts et que vous ne déplacez pas d’utilisateurs entre les forêts et les domaines , alors **objectGUID** est un attribut approprié à utiliser même dans ce cas.

Si vous déplacez des utilisateurs entre des forêts et des domaines, alors vous devez trouver un attribut qui ne change pas ou qui peut être déplacé en même temps que les utilisateurs. Une approche recommandée est d'introduire un attribut synthétique. Un attribut qui peut contenir quelque chose qui ressemble à un GUID serait approprié. Lors de la création d’un objet, un nouveau GUID est créé et affecté à l’utilisateur. Une règle de synchronisation personnalisée peut être créée dans le serveur du moteur de synchronisation pour générer cette valeur selon **objectGUID** et mettre à jour l’attribut sélectionné dans ADDS. Quand vous déplacez l’objet, veillez à copier également le contenu de cette valeur.

Une autre solution consiste à choisir un attribut existant, dont vous êtes sûr qu’il ne changera pas. **employeeID**est un des attributs couramment utilisés. Si vous envisagez d’opter pour un attribut contenant des lettres, assurez-vous qu’il n’y a aucun risque de changement de la casse (majuscule ou minuscule) pour la valeur de l’attribut. Des attributs inappropriés qui ne doivent pas être utilisés sont notamment ceux qui ont le nom de l’utilisateur. En cas de mariage ou de divorce, le nom risque de changer, ce qui n’est pas autorisé pour cet attribut. C'est également une raison pour laquelle il n'est même pas possible de sélectionner des attributs comme **userPrincipalName**, **mail** et **targetAddress** dans l'Assistant Installation d'Azure AD Connect. Ces attributs contiennent également le caractère « \@ », qui n’est pas autorisé dans sourceAnchor.

### <a name="changing-the-sourceanchor-attribute"></a>Changement de l’attribut sourceAnchor
La valeur de l’attribut sourceAnchor ne peut pas être changée une fois que l’objet a été créé dans Azure AD et que l’identité est synchronisée.

Pour cette raison, les restrictions suivantes s’appliquent à Azure AD Connect :

* L’attribut sourceAnchor peut être défini seulement lors de l’installation initiale. Si vous exécutez une nouvelle fois l’Assistant Installation, cette option est en lecture seule. Si vous devez changer ce paramètre, alors vous devez effectuer une désinstallation et une réinstallation.
* Si vous installez un autre serveur Azure AD Connect, vous devez sélectionner le même attribut sourceAnchor que celui qui a été précédemment utilisé. Si vous avez précédemment utilisé DirSync et que vous passez à Azure AD Connect, vous devez utiliser **objectGUID** , car il s'agit de l’attribut utilisé par DirSync.
* Si la valeur de sourceAnchor est changée après que l’objet a été exporté vers Azure AD, alors Azure AD Connect Sync génère une erreur et n’autorise plus de modifications sur cet objet avant que le problème ait été résolu et que sourceAnchor soit replacé à sa valeur précédente dans le répertoire source.

## <a name="using-ms-ds-consistencyguid-as-sourceanchor"></a>Utilisation de ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor
Par défaut, Azure AD Connect (version 1.1.486.0 ou antérieure) utilise objectGUID comme attribut sourceAnchor. ObjectGUID est généré par le système. Vous ne pouvez pas spécifier sa valeur lors de la création d’objets Active Directory locaux. Comme expliqué dans la section [sourceAnchor](#sourceanchor), il existe des scénarios où vous devez spécifier la valeur sourceAnchor. Si les scénarios ne s’appliquent pas à vous, vous devez utiliser un attribut AD configurable (par exemple, msDS-ConsistencyGuid) en tant qu’attribut sourceAnchor.

Azure AD Connect (version 1.1.524.0 ou ultérieure) facilite l’utilisation de l’attribut ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor. Lorsque vous utilisez cette fonctionnalité, Azure AD Connect configure automatiquement les règles de synchronisation pour :

1. Utiliser ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor pour les objets utilisateur. ObjectGUID est utilisé pour d’autres types d’objets.

2. Pour tout objet utilisateur AD local dont l’attribut ms-DS-ConsistencyGuid n’est pas spécifié, Azure AD Connect réécrit sa valeur objectGUID dans l’attribut ms-DS-ConsistencyGuid dans l’annuaire Active Directory local. Une fois l’attribut ms-DS-ConsistencyGuid spécifié, Azure AD Connect exporte l’objet vers Azure AD.

>[!NOTE]
> Une fois qu’un objet Active Directory local est importé dans Azure AD Connect (c’est-à-dire, importé dans l’espace de connecteur AD et projeté dans le Metaverse), vous ne pouvez plus modifier sa valeur sourceAnchor. Pour spécifier la valeur sourceAnchor pour un objet AD local donné, configurez son attribut ms-DS-ConsistencyGuid avant son importation dans Azure AD Connect.

### <a name="permission-required"></a>Autorisation requise
Pour que cette fonctionnalité opère, le compte AD DS utilisé pour effectuer la synchronisation avec l’annuaire Active Directory local doit disposer d’une autorisation en écriture sur l’attribut ms-DS-ConsistencyGuid dans l’annuaire Active Directory local.

### <a name="how-to-enable-the-consistencyguid-feature---new-installation"></a>Comment activer la fonctionnalité ConsistencyGuid - nouvelle installation
Vous pouvez activer l’utilisation de ConsistencyGuid en tant que sourceAnchor pendant une nouvelle installation. Cette section décrit en détail l’installation rapide et l’installation personnalisée.

  >[!NOTE]
  > Seules les versions plus récentes d’Azure AD Connect (1.1.524.0 et ultérieures) prennent en charge l’utilisation de ConsistencyGuid en tant que sourceAnchor lors d’une nouvelle installation.

### <a name="how-to-enable-the-consistencyguid-feature"></a>Comment activer la fonctionnalité ConsistencyGuid

#### <a name="express-installation"></a>Installation rapide
Lorsque vous installez Azure AD Connect en mode Express, l’Assistant Azure AD Connect détermine automatiquement l’attribut AD le plus approprié à utiliser en tant qu’attribut sourceAnchor à l’aide de la logique suivante :

* Tout d’abord, l’Assistant Azure AD Connect interroge votre client Azure AD pour extraire l’attribut Active Directory utilisé comme attribut sourceAnchor lors de la précédente installation d’Azure AD Connect (le cas échéant). Si cette information est disponible, Azure AD Connect utilise le même attribut AD.

  >[!NOTE]
  > Seules les versions plus récentes d’Azure AD Connect (1.1.524.0 et ultérieures) stockent des informations dans votre locataire Azure AD concernant l’attribut sourceAnchor utilisé pendant l’installation. Les versions antérieures d’Azure AD Connect ne le font pas.

* Si aucune information n’est disponible sur l’attribut sourceAnchor utilisé, l’Assistant vérifie l’état de l’attribut ms-DS-ConsistencyGuid dans votre annuaire Active Directory local. Si l’attribut n’est configuré sur aucun objet dans l’annuaire, l’Assistant utilise l’attribut ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor. Si l’attribut est configuré sur un ou plusieurs objets dans l’annuaire, l’Assistant en déduit que l’attribut est utilisé par d’autres applications et n’est pas adapté en tant qu’attribut sourceAnchor...

* Dans ce cas, l’Assistant revient à l’utilisation d’objectGUID comme attribut sourceAnchor.

* Une fois l’attribut sourceAnchor choisi, l’Assistant stocke les informations dans votre client Azure AD. Les informations seront utilisées dans le cadre d’une installation future d’Azure AD Connect.

Une fois l’installation rapide terminée, l’Assistant vous informe concernant l’attribut sélectionné comme attribut sourceAnchor.

![L’Assistant informe concernant attribut AD choisi pour sourceAnchor](./media/plan-connect-design-concepts/consistencyGuid-01.png)

#### <a name="custom-installation"></a>Installation personnalisée
Lorsque vous installez Azure AD Connect en mode personnalisé, l’Assistant Azure AD Connect fournit deux options lorsque vous configurez l’attribut sourceAnchor :

![Installation personnalisée - Configuration de sourceAnchor](./media/plan-connect-design-concepts/consistencyGuid-02.png)

| Paramètre | Description |
| --- | --- |
| Let Azure manage the source anchor for me (Laisser Azure gérer l’ancre source pour moi) | Sélectionnez cette option si vous souhaitez qu’Azure AD sélectionne l’attribut pour vous. Si vous sélectionnez cette option, l’Assistant Azure AD Connect applique la même [logique de sélection de l’attribut sourceAnchor que celle utilisée lors de l’installation rapide](#express-installation). Comme pour l’installation rapide, L’Assistant vous indique quel attribut a été sélectionné comme attribut sourcAnchor une fois l’installation personnalisée terminée. |
| Un attribut spécifique | Sélectionnez cette option si vous souhaitez spécifier un attribut AD existant comme attribut sourceAnchor. |

### <a name="how-to-enable-the-consistencyguid-feature---existing-deployment"></a>Comment activer la fonctionnalité ConsistencyGuid - déploiement existant
Si vous avez un déploiement Azure AD Connect existant qui utilise objectGUID comme attribut Source Anchor, vous pouvez faire en sorte qu’il utilise ConsistencyGuid à la place.

>[!NOTE]
> Seules les versions plus récentes d’Azure AD Connect (1.1.552.0 et ultérieures) prennent en charge le passage d’ObjectGuid à ConsistencyGuid en tant qu’attribut Ancre source.

Pour passer d’objectGUID à ConsistencyGuid en tant qu’attribut Ancre source :

1. Démarrez l’Assistant Azure AD Connect et cliquez sur **Configurer** pour accéder à l’écran Tâches.

2. Sélectionnez l’option de tâche **Configurer l’ancre source** et cliquez sur **Suivant**.

   ![Activer ConsistencyGuid pour un déploiement existant - Étape 2](./media/plan-connect-design-concepts/consistencyguidexistingdeployment01.png)

3. Entrez vos informations d’identification d’administrateur Azure AD et cliquez sur **Suivant**.

4. L’Assistant Azure AD Connect vérifie l’état de l’attribut ms-DS-ConsistencyGuid dans votre annuaire Active Directory local. Si l’attribut n’est configuré sur aucun objet de l’annuaire, Azure AD Connect conclut qu’aucune autre application n’utilise actuellement l’attribut et qu’il peut être utilisé comme attribut Ancre Source en toute sécurité. Cliquez sur **Suivant** pour continuer.

   ![Activer ConsistencyGuid pour un déploiement existant - Étape 4](./media/plan-connect-design-concepts/consistencyguidexistingdeployment02.png)

5. Dans l’écran **Prêt à configurer**, cliquez sur **Configurer** pour modifier la configuration.

   ![Activer ConsistencyGuid pour un déploiement existant - Étape 5](./media/plan-connect-design-concepts/consistencyguidexistingdeployment03.png)

6. Une fois la configuration terminée, l’Assistant indique que ms-DS-ConsistencyGuid est maintenant utilisé en tant qu’attribut sourceAnchor.

   ![Activer ConsistencyGuid pour un déploiement existant - Étape 6](./media/plan-connect-design-concepts/consistencyguidexistingdeployment04.png)

Pendant l’analyse (étape 4), si l’attribut est configuré sur un ou plusieurs objets de l’annuaire, l’Assistant en déduit que l’attribut est utilisé par une autre application et retourne une erreur, comme illustré dans le diagramme ci-dessous. Cette erreur peut également se produire quand vous avez déjà activé la fonctionnalité ConsistencyGuid sur votre serveur principal Azure AD Connect et que vous essayez de faire de même sur votre serveur de préproduction.

![Activer ConsistencyGuid pour un déploiement existant - Erreur](./media/plan-connect-design-concepts/consistencyguidexistingdeploymenterror.png)

 Si vous êtes certain que l'attribut n'est pas utilisé par d'autres applications, vous pouvez éviter cette erreur en redémarrant l'Assistant Azure AD Connect avec le commutateur **/SkipLdapSearch** spécifié. Pour cela, exécutez la commande suivante dans l’invite de commandes :

```
"c:\Program Files\Microsoft Azure Active Directory Connect\AzureADConnect.exe" /SkipLdapSearch
```

### <a name="impact-on-ad-fs-or-third-party-federation-configuration"></a>Impact sur AD FS d’une configuration de fédération tierce
Si vous utilisez Azure AD Connect pour gérer un déploiement d’AD FS local, Azure AD Connect met automatiquement à jour les règles de revendication pour utiliser le même attribut AD en tant qu’attribut sourceAnchor. Cela garantit que la revendication ImmutableID générée par ADFS est cohérente avec les valeurs de sourceAnchor exportées vers Azure AD.

Si vous gérez AD FS en dehors d’Azure AD Connect ou si vous utilisez des serveurs de fédération tiers pour l’authentification, vous devez mettre à jour manuellement les règles de revendication pour que la revendication ImmutableID soit cohérente avec les valeurs de sourceAnchor exportées vers Azure AD, comme décrit dans la section [Modifier les règles de revendication AD FS](./how-to-connect-fed-management.md#modclaims) de l’article. Une fois l’installation terminée, L’Assistant renvoie l’avertissement suivant :

![Configuration de fédération tierce](./media/plan-connect-design-concepts/consistencyGuid-03.png)

### <a name="adding-new-directories-to-existing-deployment"></a>Ajout d’annuaires à un déploiement existant
Supposons que vous avez déployé Azure AD Connect avec la fonctionnalité ConsistencyGuid activée, et que vous souhaitez ajouter un annuaire au déploiement. Lorsque vous tentez d’ajouter l’annuaire, l’Assistant Azure AD Connect vérifie l’état de l’attribut ms-DS-ConsistencyGuid dans l’annuaire. Si l’attribut est configuré sur un ou plusieurs objets dans l’annuaire, l’Assistant en déduit que l’attribut est utilisé par d’autres applications et retourne une erreur, comme illustré dans la diagramme ci-dessous. Si vous êtes certain que l'attribut n'est pas utilisé par d'autres applications, vous pouvez éviter cette erreur en redémarrant l'Assistant Azure AD Connect avec le commutateur **/SkipLdapSearch** spécifié, comme décrit ci-dessus, ou en contactant le support pour plus d'informations.

![Ajout d’annuaires à un déploiement existant](./media/plan-connect-design-concepts/consistencyGuid-04.png)

## <a name="azure-ad-sign-in"></a>Authentification dans Azure AD
Lorsque vous intégrez votre annuaire local à Azure AD, il est important de bien comprendre comment les paramètres de synchronisation peuvent affecter la manière dont les utilisateurs s’authentifient. Azure AD utilise userPrincipalName (UPN) pour authentifier l’utilisateur. Toutefois, lorsque vous synchronisez vos utilisateurs, vous devez choisir avec soin l’attribut à utiliser pour la valeur de userPrincipalName.

### <a name="choosing-the-attribute-for-userprincipalname"></a>Choix de l’attribut de userPrincipalName
Lorsque vous sélectionnez l’attribut fournissant la valeur d’UPN à utiliser dans Azure, vous devez vous assurer que :

* Les valeurs de l’attribut sont conformes à la syntaxe UPN (RFC 822), c’est-à-dire au format nom_utilisateur\@domaine.
* Le suffixe des valeurs correspond à l’un des domaines personnalisés vérifiés dans Azure AD

Dans la configuration rapide, le choix supposé de l’attribut est userPrincipalName. Si l’attribut userPrincipalName ne contient pas la valeur que vous souhaitez que vos utilisateurs utilisent pour se connecter à Azure, alors choisissez **Installation personnalisée**.

### <a name="custom-domain-state-and-upn"></a>État du domaine personnalisé et UPN
Il est important de s’assurer qu’il existe un domaine vérifié pour le suffixe UPN.

John est un utilisateur de contoso.com. Vous souhaitez que John utilise l’UPN local john\@contoso.com pour se connecter à Azure une fois que vous avez synchronisé les utilisateurs vers votre annuaire Azure AD contoso.onmicrosoft.com. Pour ce faire, vous devez ajouter et vérifier contoso.com comme domaine personnalisé dans Azure AD avant de commencer la synchronisation des utilisateurs. Si le suffixe UPN de John, par exemple contoso.com, ne correspond pas à un domaine vérifié dans Azure AD, alors Azure AD remplace le suffixe UPN par contoso.onmicrosoft.com.

### <a name="non-routable-on-premises-domains-and-upn-for-azure-ad"></a>Domaines locaux non routables et UPN pour Azure AD
Certaines organisations ont des domaines non routables, comme contoso.local, ou de simples domaines à étiquette unique, comme contoso. Dans Azure AD, vous n’êtes pas en mesure de vérifier un domaine non routable. Azure AD Connect peut uniquement se synchroniser sur un domaine vérifié dans Azure AD. Lorsque vous créez un répertoire Azure AD, il crée un domaine routable qui devient le domaine par défaut de votre Azure AD, par exemple contoso.onmicrosoft.com. Par conséquent, il devient nécessaire de vérifier tous les autres domaines routables dans un scénario de ce type, si vous ne souhaitez pas effectuer de synchronisation avec le domaine par défaut onmicrosoft.com.

Pour plus d’informations sur l’ajout et la vérification de domaines, consultez [Ajouter un nom de domaine personnalisé à Azure Active Directory](../fundamentals/add-custom-domain.md) .

Azure AD Connect détecte si vous exécutez un environnement de domaine non routable et vous avertit en temps utile si vous tentez de poursuivre la configuration rapide. Si votre domaine n’est pas routable, il est probable que les UPN des utilisateurs aient également des suffixes non routables. Par exemple, si votre domaine est contoso.local, Azure AD Connect vous propose d’utiliser des paramètres personnalisés plutôt que la configuration rapide. Avec les paramètres personnalisés, vous êtes en mesure de spécifier l’attribut à utiliser comme UPN pour la connexion à Azure une fois les utilisateurs synchronisés avec Azure AD.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).