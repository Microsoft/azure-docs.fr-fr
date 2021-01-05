---
title: 'Synchronisation d’Azure AD Connect : comprendre la configuration par défaut | Microsoft Docs'
description: Cet article décrit la configuration par défaut dans la synchronisation Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ed876f22-6892-4b9d-acbe-6a2d112f1cd1
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5e55526e0a63a0c603e2b62ccb3ac0efed911cff
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996625"
---
# <a name="azure-ad-connect-sync-understanding-the-default-configuration"></a>Synchronisation d’Azure AD Connect : Présentation de la configuration par défaut
Cet article présente les règles de configuration out-of-box. Il décrit les règles et l’impact que celles-ci ont sur la configuration. Il vous guide également tout au long de la configuration par défaut de la synchronisation Azure AD Connect. L’objectif est que le lecteur comprenne comment fonctionne le modèle de configuration, nommé approvisionnement déclaratif, dans un exemple réel. Cet article suppose que vous avez déjà installé et configuré la synchronisation Azure AD Connect à l’aide de l’Assistant d’installation.

Pour comprendre les détails du modèle de configuration, lisez [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).

## <a name="out-of-box-rules-from-on-premises-to-azure-ad"></a>Règles out-of-box du local vers Azure AD
Les expressions suivantes se trouvent dans la configuration out-of-box.

### <a name="user-out-of-box-rules"></a>Règles out-of-box d’utilisateur
Ces règles s’appliquent également au type d’objet iNetOrgPerson.

Un objet utilisateur doit remplir les conditions suivantes pour être synchronisé :

* Il doit disposer d’un sourceAnchor.
* Une fois que l’objet a été créé dans Azure AD, le sourceAnchor ne peut pas être modifié. Si la valeur est modifiée en local, l’objet arrête la synchronisation jusqu’à ce que l’attribut sourceAnchor récupère sa valeur précédente.
* L’attribut accountEnabled (userAccountControl) dont il dispose doit être renseigné. Avec un Active Directory local, cet attribut est toujours présent et renseigné.

Les objets utilisateur suivants ne sont **pas** synchronisés avec Azure AD :

* `IsPresent([isCriticalSystemObject])`. Vérifiez que la plupart des objets out-of-box dans Active Directory, tels que le compte administrateur intégré, ne sont pas synchronisés.
* `IsPresent([sAMAccountName]) = False`. Assurez-vous que les objets utilisateur sans l'attribut sAMAccountName ne sont pas synchronisés. Ce cas se produit pratiquement uniquement dans un domaine mis à niveau à partir d’un système NT4.
* `Left([sAMAccountName], 4) = "AAD_"`, `Left([sAMAccountName], 5) = "MSOL_"`. Ne synchronisez pas le compte de service utilisé par la synchronisation d’Azure AD Connect et de ses versions précédentes.
* Ne synchronisez pas les comptes Exchange qui pourraient ne pas fonctionner dans Exchange Online.
  * `[sAMAccountName] = "SUPPORT_388945a0"`
  * `Left([mailNickname], 14) = "SystemMailbox{"`
  * `(Left([mailNickname], 4) = "CAS_" && (InStr([mailNickname], "}") > 0))`
  * `(Left([sAMAccountName], 4) = "CAS_" && (InStr([sAMAccountName], "}")> 0))`
* Ne synchronisez pas les objets qui pourraient ne pas fonctionner dans Exchange Online.
  `CBool(IIF(IsPresent([msExchRecipientTypeDetails]),BitAnd([msExchRecipientTypeDetails],&H21C07000) > 0,NULL))`  
  Ce masque de bits (&H21C07000) pourrait exclure les objets suivants :
  * Dossier public à extension messagerie (en préversion à partir de la version 1.1.524.0)
  * Boîte aux lettres de surveillance du système
  * Boîte aux lettres de base de données de boîtes aux lettres (Boîte aux lettres système)
  * Groupe de sécurité universel (ne s'applique pas à un utilisateur, mais est présent pour des raisons d'héritage)
  * Groupe non universel (ne s'applique pas à un utilisateur, mais est présent pour des raisons d'héritage)
  * Plan de boîte aux lettres
  * Boîte aux lettres de découverte
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`. Ne synchronisez aucun objet victime de réplication.

Les règles d’attribut suivantes s’appliquent :

* `sourceAnchor <- IIF([msExchRecipientTypeDetails]=2,NULL,..)`. L’attribut sourceAnchor n’est pas partagé à partir d’une boîte aux lettres liée. On part du principe que si une boîte aux lettres liée a été trouvée, le compte actuel sera associé ultérieurement.
* Les attributs liés à Exchange sont uniquement synchronisés si l'attribut **mailNickName** a une valeur.
* Lorsqu’il existe plusieurs forêts, les attributs sont consommés dans l’ordre suivant :
  1. Les attributs liés à l’authentification (par exemple, userPrincipalName) sont obtenus à partir de la forêt avec un compte activé.
  2. Les attributs qui se trouvent dans la LAG (liste d’adresses globale) Exchange sont synchronisés à partir de la forêt avec la boîte aux lettres Exchange.
  3. Si aucune boîte aux lettres ne peut être trouvée, ces attributs peuvent provenir de n'importe quelle forêt.
  4. Les attributs liés à Exchange (attributs techniques non visibles dans la liste d’adresses globale) sont fournis à partir de la forêt où `mailNickname ISNOTNULL`.
  5. S’il existe plusieurs forêts qui répondent à une de ces règles, alors l’ordre de création (date/heure) des connecteurs (forêts) est utilisé pour déterminer quelle forêt fournit les attributs. La première forêt connectée sera la première forêt à être synchronisée. 

### <a name="contact-out-of-box-rules"></a>Règles out-of-box de contact
Un objet contact doit remplir les conditions suivantes pour être synchronisé :

* Le contact doit être à extension messagerie. Il est vérifié par les règles suivantes :
  * `IsPresent([proxyAddresses]) = True)`. L'attribut proxyAddresses doit être renseigné.
  * L'attribut proxyAddresses ou l'attribut de messagerie peuvent comporter une adresse de messagerie principale. La présence d’un \@ est utilisée pour vérifier que le contenu est une adresse de messagerie. L’une de ces deux règles doit prendre la valeur True.
    * `(Contains([proxyAddresses], "SMTP:") > 0) && (InStr(Item([proxyAddresses], Contains([proxyAddresses], "SMTP:")), "@") > 0))`. Existe-t-il une entrée avec « SMTP: », et si tel est le cas, est-il possible de trouver un \@ dans la chaîne ?
    * `(IsPresent([mail]) = True && (InStr([mail], "@") > 0)`. L’attribut de messagerie est-il renseigné, et si tel est le cas, est-il possible de trouver un \@ dans la chaîne ?

Les objets contact suivants ne sont **pas** synchronisés avec Azure AD :

* `IsPresent([isCriticalSystemObject])`. Vérifiez qu’aucun objet contact marqué comme critique n’est synchronisé. Cet objet ne devrait pas avoir de configuration par défaut.
* `((InStr([displayName], "(MSOL)") > 0) && (CBool([msExchHideFromAddressLists])))`.
* `(Left([mailNickname], 4) = "CAS_" && (InStr([mailNickname], "}") > 0))`. Ces objets ne fonctionneraient pas avec Exchange Online.
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`. Ne synchronisez aucun objet victime de réplication.

### <a name="group-out-of-box-rules"></a>Règles out-of-box de groupe
Un objet groupe doit remplir les conditions suivantes pour être synchronisé :

* Il doit compter moins de 50 000 membres. Ce chiffre correspond au nombre de membres du groupe local.
  * S’il compte plus de membres avant sa toute première synchronisation, le groupe n’est pas synchronisé.
  * Si le nombre de membres augmente à partir du moment où il a été créé, alors sa synchronisation est interrompue lorsqu’il atteint 50 000 membres, jusqu’à ce que le nombre redevienne inférieur à 50 000 membres.
  * Remarque : La limite de 50 000 membres s’applique également avec Azure AD. Vous ne pouvez pas synchroniser les groupes d’un nombre de membres supérieur même si vous modifiez ou supprimez cette règle.
* Si le groupe est un **groupe de distribution**, il doit alors également être activé pour la messagerie. Consultez la page [Règles out-of-box de contact](#contact-out-of-box-rules) pour l’application de cette règle.

Les objets groupe suivants ne sont **pas** synchronisés avec Azure AD :

* `IsPresent([isCriticalSystemObject])`. Vérifiez que la plupart des objets out-of-box dans Active Directory, tels que le groupe administrateur intégré, ne sont pas synchronisés.
* `[sAMAccountName] = "MSOL_AD_Sync_RichCoexistence"`. Groupe hérité utilisé par DirSync.
* `BitAnd([msExchRecipientTypeDetails],&amp;H40000000)`. Groupe de rôles.
* `CBool(InStr(DNComponent(CRef([dn]),1),"\\0ACNF:")>0)`. Ne synchronisez aucun objet victime de réplication.

### <a name="foreignsecurityprincipal-out-of-box-rules"></a>Règles out-of-box ForeignSecurityPrincipal
Les FSP sont joints à « tout » (\*) objet dans le métaverse. En réalité, cela se produit uniquement pour les utilisateurs et les groupes de sécurité. Cette configuration garantit que les appartenances inter-forêts sont correctement résolues et représentées dans Azure AD.

### <a name="computer-out-of-box-rules"></a>Règles out-of-box d’ordinateur
Un objet ordinateur doit remplir les conditions suivantes pour être synchronisé :

* `userCertificate ISNOTNULL`. Seuls les objets ordinateur Windows 10 renseignent cet attribut. Tous les objets ordinateur avec une valeur de cet attribut sont synchronisés.

## <a name="understanding-the-out-of-box-rules-scenario"></a>Présentation du scénario de règles out-of-box
Dans cet exemple, nous utilisons un déploiement comportant une forêt de comptes (A), une forêt de ressources (R) et un répertoire Azure AD.

![Vue d’ensemble avec description du scénario](./media/concept-azure-ad-connect-sync-default-configuration/scenario.png)

Dans cette configuration, il est supposé exister un compte activé dans la forêt de comptes et un compte désactivé dans la forêt de ressources avec une boîte aux lettres liée.

Nos objectifs avec la configuration par défaut sont les suivants :

* Les attributs liés à la connexion sont synchronisés à partir de la forêt avec le compte activé.
* Les attributs qui se trouvent dans la LAG (liste d’adresses globale) sont synchronisés à partir de la forêt avec la boîte aux lettres. Si aucune boîte aux lettres n’est trouvée, toute autre forêt est utilisée.
* Si une boîte aux lettres liée est trouvée, le compte activé lié doit être trouvé pour l’objet à exporter vers Azure AD.

### <a name="synchronization-rule-editor"></a>Éditeur de règles de synchronisation
La configuration peut être affichée et modifiée avec l’outil Éditeur de règles de synchronisation (SRE), dont le raccourci se trouve dans le menu Démarrer.

![Icône de l’Éditeur de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/sre.png)

Le SRE est un outil du kit de ressources installé avec la synchronisation Azure AD Connect. Pour être en mesure de le démarrer, vous devez être membre du groupe ADSyncAdmins. Lorsqu’il démarre, vous voyez quelque chose comme suit :

![Règles de synchronisation entrante](./media/concept-azure-ad-connect-sync-default-configuration/syncrulesinbound.png)

Dans ce volet, vous voyez toutes les règles de synchronisation créées pour votre configuration. Chaque ligne du tableau correspond à une règle de synchronisation. À gauche, sous Types de règles, deux types sont présentés : Entrante et Sortante. Les termes entrante et sortante sont à prendre du point de vue du métaverse. Vous allez principalement vous concentrer sur les règles de trafic entrant dans cette vue d’ensemble. La liste actuelle des règles de synchronisation dépend du schéma détecté dans Active Directory. Dans l’image ci-dessus, la forêt de comptes (fabrikamonline.com) ne dispose d’aucun service (par exemple, Exchange ou Lync), et aucune règle de synchronisation n’a été créée pour ces services. Toutefois, dans la forêt de ressources (res.fabrikamonline.com), vous trouvez des règles de synchronisation pour ces services. Le contenu des règles diffère selon la version détectée. Par exemple, dans un déploiement avec Exchange 2013, plus de flux d’attributs sont configurés que dans Exchange 2010/2007.

### <a name="synchronization-rule"></a>Règle de synchronisation
Une règle de synchronisation est un objet de configuration avec un jeu d’attributs qui circule quand une condition est remplie. Elle sert aussi à décrire la relation entre un objet dans un espace de connecteur et un objet dans le métaverse, relation appelée **jointure** ou **correspondance**. Les règles de synchronisation ont une valeur de précédence qui indique leurs relations les unes par rapport aux autres. Une règle de synchronisation avec une valeur plus basse possède une précédence plus élevée et, dans un conflit de flux d’attributs, c’est la précédence la plus élevée qui l’emporte.

En guise d’exemple, examinez la règle de synchronisation **Entrant depuis AD – Utilisateur AccountEnabled**. Marquez cette ligne dans l’outil SRE et sélectionnez **Modifier**.

Dans la mesure où il s’agit d’une règle out-of-box, vous recevez un avertissement à l’ouverture de la règle. Vous ne devez pas apporter de [modifications aux règles out-of-box](how-to-connect-sync-best-practices-changing-default-configuration.md)vous êtes donc invité à indiquer vos intentions. Dans ce cas, vous souhaitez uniquement afficher la règle. Sélectionnez **Non**.

![Avertissement de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/warningeditrule.png)

Une règle de synchronisation comporte quatre sections de configuration : Description, Filtre d’étendue, Règles de jointure et Transformations.

#### <a name="description"></a>Description
La première section fournit des informations de base telles que le nom et la description.

![Onglet Description dans l’Éditeur de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/syncruledescription.png)

Vous y trouverez également des informations concernant le système connecté associé à cette règle, le type d’objet dans le système connecté auquel il s’applique et le type d’objet de métaverse. Le type d’objet du métaverse est toujours « person », que le type d’objet source soit un utilisateur, iNetOrgPerson ou un contact. Comme il ne doit jamais changer, il est créé en tant que type générique. Le type de lien peut être Join, StickyJoin ou Provision. Ce paramètre fonctionne avec la section Règles de jointure et il est traité plus loin dans ce document.

Vous pouvez également voir que cette règle de synchronisation est utilisée pour la synchronisation de mot de passe. Si un utilisateur est dans la portée pour cette règle de synchronisation, le mot de passe est synchronisé du local vers le cloud (en supposant que vous avez activé la fonctionnalité de synchronisation de mot de passe).

#### <a name="scoping-filter"></a>Filtre d’étendue
La section Filtre d’étendue sert à configurer à quel moment une règle de synchronisation doit s’appliquer. Comme le nom de la règle de synchronisation que vous examinez indique qu’elle ne doit être appliquée qu’aux utilisateurs activés, l’étendue est configurée de sorte que l’attribut AD **userAccountControl** ne doive pas avoir le bit 2 défini. Lorsque le moteur de synchronisation détecte un utilisateur dans Active Directory, il applique la synchronisation lorsque **userAccountControl** est défini sur la valeur décimale 512 (utilisateur normal activé). Il n’applique pas la règle lorsque l’utilisateur a **userAccountControl** défini sur 514 (utilisateur normal désactivé).

![Capture d’écran montrant la section « Filtre d’étendue » de la fenêtre « Modifier une règle de synchronisation entrante ».](./media/concept-azure-ad-connect-sync-default-configuration/syncrulescopingfilter.png)

Le filtre d’étendue possède des groupes et des clauses qui peuvent être imbriqués. Toutes les clauses à l’intérieur d’un groupe doivent être satisfaites pour qu’une règle de synchronisation s’applique. Quand plusieurs groupes sont définis, au moins l’un d’entre eux doit être satisfait pour que la règle s’applique. C’est-à-dire qu’une opération OR logique est évaluée entre les groupes et qu’une opération AND logique est évaluée à l’intérieur d’un groupe. On trouve un exemple de cette configuration dans la règle de synchronisation sortante **Sortant vers AAD – Group Join**, indiquée ci-dessous. Il existe plusieurs groupes de filtres de synchronisation, par exemple, un pour les groupes de sécurité (`securityEnabled EQUAL True`) et un autre pour les groupes de distribution (`securityEnabled EQUAL False`).

![Onglet Étendue dans l’Éditeur de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/syncrulescopingfilterout.png)

Cette règle sert à définir les groupes qui doivent être approvisionnés dans Azure AD. Les groupes de distribution doivent être activés pour la messagerie électronique pour pouvoir être synchronisés avec Azure AD, mais pour les groupes de sécurité aucune adresse de messagerie n’est nécessaire.

#### <a name="join-rules"></a>Règles de jointure
La troisième section sert à configurer la relation entre les objets dans l’espace de connecteur et les objets dans le métaverse. La règle que vous avez examinée plus haut n’ayant aucune configuration pour Join Rules, vous allez plutôt examiner **Entrant depuis AD – User Join**.

![Onglet Règles de jointure dans l’Éditeur de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/syncrulejoinrules.png)

Le contenu de la règle de jointure dépend de l’option de correspondance sélectionnée dans l’Assistant Installation. Pour une règle entrante, l’évaluation commence avec un objet dans l’espace de connecteur source et chaque groupe dans les règles de jointure est évalué de manière séquentielle. Si un objet source évalué correspond exactement à un objet dans le métaverse selon l’une des règles de jointure, les deux objets sont joints. Si toutes les règles ont été évaluées et qu’il n’existe aucune correspondance, le type de lien indiqué dans la page de description est utilisé. Si cette configuration est définie sur **Provisionner**, un nouvel objet est créé dans la cible, le métaverse, si au moins un attribut dans les critères de jonction est présent (a une valeur). L’approvisionnement d’un nouvel objet dans le métaverse porte également le nom de **projection** .

Les règles de jointure ne sont évaluées qu’une seule fois. Quand un objet d’espace de connecteur et un objet de métaverse sont joints, ils le restent tant que l’étendue de la règle de synchronisation est encore satisfaite.

Lors de l’évaluation de plusieurs règles de synchronisation, une seule d’entre elles avec des règles de jointure définies doit être dans l’étendue. Si plusieurs règles de synchronisation avec des règles de jointure sont détectées pour un objet, une erreur est levée. Pour cette raison, la meilleure pratique consiste à n’avoir qu’une seule règle de synchronisation avec jointure définie quand plusieurs règles de synchronisation sont dans l’étendue pour un objet. Dans la configuration par défaut de la synchronisation d’Azure AD Connect, vous pouvez identifier facilement ces règles, car leur nom se termine par le mot **Join** . Une règle de synchronisation sans aucune règle de jointure définie applique les flux d’attributs lorsqu’une autre règle de synchronisation a joint les objets ensemble ou a approvisionné un nouvel objet dans la cible.

Si vous examinez l’illustration ci-dessus, vous pouvez voir que la règle essaie de joindre **objectSID** avec **msExchMasterAccountSid** (Exchange) et **msRTCSIP-OriginatorSid** (Lync), ce qui est le résultat que nous attendons dans une topologie de forêt de comptes/ressources. Vous trouverez la même règle sur toutes les forêts. L’hypothèse est que chaque forêt peut être une forêt de comptes ou de ressources. Cette configuration fonctionne également si vous disposez de comptes qui résident dans une forêt unique et n’ont pas à être joints.

#### <a name="transformations"></a>Transformations
La section Transformation définit tous les flux d’attributs qui s’appliquent à l’objet cible quand les objets sont joints et quand le filtre d’étendue est satisfait. Si vous revenez à la règle de synchronisation **Entrant depuis AD – Utilisateur AccountEnabled** , vous trouvez les transformations suivantes :

![Onglet Transformations dans l’Éditeur de règles de synchronisation](./media/concept-azure-ad-connect-sync-default-configuration/syncruletransformations.png)

Pour mettre cette configuration en contexte, dans un déploiement de forêt Account-Resource, il devrait y avoir un compte activé dans la forêt de comptes et un compte désactivé dans la forêt de ressources avec des paramètres Exchange et Lync. La règle de synchronisation que vous examinez contient les attributs nécessaires pour la connexion et ceux-ci devraient circuler à partir de la forêt dans laquelle se trouve un compte activé. Tous ces flux d’attributs sont regroupés dans une règle de synchronisation.

Une transformation peut avoir différents types : Constant, Direct et Expression.

* Un flux constant transmet toujours une valeur codée en dur. Dans le cas ci-dessus, il définit toujours la valeur **True** dans l’attribut du métaverse nommé **accountEnabled**.
* Un flux direct transmet toujours la valeur de l’attribut source telle quelle vers l’attribut cible.
* Le troisième type de flux, Expression, permet d’effectuer des configurations avancées.

Le langage d’expression étant VBA (Visual Basic pour Applications), les utilisateurs ayant une expérience de Microsoft Office ou VBScript reconnaîtront le format. Les attributs sont placés entre crochets : [nom_attribut]. Les noms des attributs et des fonctions respectent la casse, mais l’Éditeur de règles de synchronisation évalue les expressions et affiche un avertissement si elles ne sont pas valides. Toutes les expressions sont placées sur une seule ligne avec les fonctions imbriquées. Pour illustrer toute la puissance du langage de configuration, voici le flux pour pwdLastSet, mais avec des commentaires supplémentaires insérés :

```
// If-then-else
IIF(
// (The evaluation for IIF) Is the attribute pwdLastSet present in AD?
IsPresent([pwdLastSet]),
// (The True part of IIF) If it is, then from right to left, convert the AD time format to a .NET datetime, change it to the time format used by Azure AD, and finally convert it to a string.
CStr(FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")),
// (The False part of IIF) Nothing to contribute
NULL
)
```

Pour plus d’informations sur le langage d’expression pour les flux d’attributs, consultez [Présentation des expressions de l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) .

### <a name="precedence"></a>Priorité
Vous avez maintenant vu certaines règles de synchronisation individuelles, mais les règles fonctionnent de concert dans la configuration. Dans certains cas, une valeur d’attribut est partagée à partir de plusieurs règles de synchronisation vers le même attribut cible. Dans ce cas, la précédence d’attribut est utilisée pour déterminer quel attribut l’emporte. Par exemple, examinez l’attribut sourceAnchor. Cet attribut est important pour pouvoir se connecter à Azure AD. Vous pouvez trouver un flux d’attributs pour cet attribut dans deux règles de synchronisation différentes : **Entrant depuis AD – Utilisateur AccountEnabled** et **Entrant depuis AD – Utilisateur Common**. En raison de la précédence de la règle de synchronisation, l’attribut sourceAnchor est partagé à partir de la forêt avec un compte activé en premier lieu lorsqu’il existe plusieurs objets joints à l’objet métaverse. S’il n’y a aucun compte activé, le moteur de synchronisation utilise la règle de synchronisation fourre-tout **Entrant depuis AD – Utilisateur Common**. Cette configuration garantit que même pour les comptes qui sont désactivés, il existe toujours un sourceAnchor.

![Règles de synchronisation entrante](./media/concept-azure-ad-connect-sync-default-configuration/syncrulesinbound.png)

La précédence des règles de synchronisation est définie dans les groupes par l’Assistant d’installation. Toutes les règles d’un groupe ont le même nom, mais elles sont connectées à différents répertoires connectés. L’Assistant d’installation donne à la règle **Entrant depuis AD – User Join** la précédence la plus élevée et effectue une itération sur tous les répertoires AD connectés. Il continue ensuite avec les groupes de règles suivants dans un ordre prédéfini. À l’intérieur d’un groupe, les règles sont ajoutées dans l’ordre dans lequel les connecteurs ont été ajoutés à l’Assistant. Si un autre connecteur est ajouté via l’Assistant, les règles de synchronisation sont réordonnées et les règles du nouveau connecteur sont insérées en dernier dans chaque groupe.

### <a name="putting-it-all-together"></a>Exemple complet
Nous en savons maintenant assez sur les règles de synchronisation pour comprendre le fonctionnement de la configuration avec les différentes règles de synchronisation. Si vous regardez un utilisateur et les attributs qui sont partagés avec le métaverse, les règles sont appliquées dans l’ordre suivant :

| Nom | Commentaire |
|:--- |:--- |
| Entrant depuis AD – User Join |Règle pour joindre les objets de l’espace de connecteur avec métaverse. |
| Entrant depuis AD – Utilisateur AccountEnabled |Attributs requis pour la connexion à Azure AD et Microsoft 365. Nous voulons ces attributs à partir du compte activé. |
| Entrant depuis AD – Utilisateur Common à partir d’Exchange |Attributs trouvés dans la liste d’adresses globale. Nous supposons que la qualité des données est meilleure dans la forêt où nous avons trouvé la boîte aux lettres de l’utilisateur. |
| Entrant depuis AD – Utilisateur Common |Attributs trouvés dans la liste d’adresses globale. Dans le cas où nous ne trouvons pas de boîte aux lettres, tout autre objet joint peut contribuer à la valeur d’attribut. |
| Entrant depuis AD – Utilisateur Exchange |Existe seulement si Exchange a été détecté. Transfère tous les attributs Exchange d’infrastructure. |
| Entrant depuis AD – Utilisateur Lync |Existe seulement si Lync a été détecté. Transfère tous les attributs Lync d’infrastructure. |

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur le modèle de configuration dans [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).
* En savoir plus sur le langage d’expression dans [Comprendre les expressions d’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md).
* Poursuivre la lecture sur le fonctionnement de la configuration out-of-box dans [Présentation des utilisateurs et des contacts](concept-azure-ad-connect-sync-user-and-contacts.md)
* Apprendre à effectuer une modification pratique à l’aide de l’approvisionnement déclaratif dans [Comment modifier la configuration par défaut](how-to-connect-sync-change-the-configuration.md).

**Rubriques de présentation**

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)

