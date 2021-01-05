---
title: 'Azure AD Connect : Résolution des erreurs lors de la synchronisation | Microsoft Docs'
description: Explique comment résoudre les erreurs rencontrées lors de la synchronisation avec Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 2209d5ce-0a64-447b-be3a-6f06d47995f8
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 10/29/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a668fa9bf0ef4fd3b5451ff4c815b676fe237e51
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410621"
---
# <a name="troubleshooting-errors-during-synchronization"></a>Résolution des erreurs lors de la synchronisation
Des erreurs peuvent se produire lorsque les données d’identité sont synchronisées à partir de Windows Server Active Directory (AD DS) vers Azure Active Directory (Azure AD). Cet article fournit une vue d’ensemble des différents types d’erreurs de synchronisation, certains des scénarios qui provoquent ces erreurs et les méthodes possibles pour les résoudre. Cet article inclut les types d’erreur courants et peut ne pas couvrir toutes les erreurs possibles.

 Cet article suppose que le lecteur est familiarisé avec les [principes de conception d’Azure AD et Azure AD Connect](plan-connect-design-concepts.md).

Avec la version la plus récente d’Azure AD Connect (\(août 2016 ou une version ultérieure\)), un rapport d’erreurs de synchronisation est disponible dans le [portail Azure](https://aka.ms/aadconnecthealth) dans le cadre d’Azure AD Connect Health pour synchronisation.

À compter du 1er septembre 2016, la fonctionnalité [Résilience d’attribut en double Azure Active Directory](how-to-connect-syncservice-duplicate-attribute-resiliency.md) sera activée par défaut pour tous les *nouveaux* clients Azure Active Directory. Cette fonctionnalité sera automatiquement activée pour les clients existants dans les mois à venir.

Azure AD Connect effectue trois types d'opérations à partir des répertoires dont il assure la synchronisation : importation, synchronisation et exportation. Les erreurs peuvent se produire dans toutes les opérations. Cet article se concentre principalement sur les erreurs lors de l’exportation vers Azure AD.

## <a name="errors-during-export-to-azure-ad"></a>Erreurs lors de l’exportation vers Azure AD
La section suivante décrit les différents types d’erreurs de synchronisation pouvant se produire lors de l’opération d’exportation vers Azure AD avec le connecteur Azure AD. Ce connecteur peut être identifié par le format de nom « contoso. *onmicrosoft.com* ».
Les erreurs lors de l’exportation vers Azure AD indiquent que l’opération \(Ajouter, mettre à jour, supprimer, etc.\) tentée par le \(moteur de synchronisation\) d’Azure AD Connect sur Azure Active Directory a échoué.

![Vue d’ensemble des erreurs d’exportation](./media/tshoot-connect-sync-errors/Export_Errors_Overview_01.png)

## <a name="data-mismatch-errors"></a>Erreurs d’incohérence de données
### <a name="invalidsoftmatch"></a>InvalidSoftMatch
#### <a name="description"></a>Description
* Lorsque le \(moteur de synchronisation\) d’Azure AD Connect indique à Azure Active Directory d’ajouter ou mettre à jour des objets, Azure AD fait correspondre l’objet entrant en utilisant l’attribut **sourceAnchor** sur l’attribut **immutableId** des objets dans Azure AD. Cette correspondance est appelée une **Correspondance exacte**.
* Lorsqu’Azure AD **ne trouve pas** d’objet correspondant à l’attribut **immutableId** avec l’attribut **sourceAnchor** de l’objet entrant, il revient pour utiliser les attributs ProxyAddresses et UserPrincipalName pour rechercher une correspondance avant de configurer un nouvel objet. Cette correspondance est appelée une **Correspondance souple**. La correspondance souple est conçue pour faire correspondre des objets déjà présents dans Azure AD (qui sont générés dans Azure AD) avec les nouveaux objets ajoutés ou mis à jour lors de la synchronisation représentant la même entité (utilisateurs, groupes) en local.
* L’erreur **InvalidSoftMatch** se produit lorsque la correspondance exacte ne trouve aucun objet correspondant **ET** que la correspondance souple trouve un objet correspondant, mais que cet objet a une valeur d’attribut *immutableId* différente de l’attribut *SourceAnchor* de l’objet entrant, suggérant que l’objet correspondant a été synchronisé avec un autre sur l’Active Directory local.

En d’autres termes, afin que la correspondance souple fonctionne, l’objet mis en correspondance ne doit avoir aucune valeur pour *immutableId*. Si un objet avec l’attribut *immutableId* défini avec une valeur ne réussit pas la correspondance exacte mais satisfait aux critères de correspondance souple, l’opération entraîne une erreur de synchronisation InvalidSoftMatch.

Le schéma Azure Active Directory n’autorise de donner à deux objets ou plus la même valeur pour les attributs suivants. \(Cette liste est non exhaustive :\)

* ProxyAddresses
* UserPrincipalName
* onPremisesSecurityIdentifier
* ObjectId

> [!NOTE]
> La fonctionnalité [Résilience d’attribut en double Azure AD](how-to-connect-syncservice-duplicate-attribute-resiliency.md) est également déployée en tant que comportement par défaut d’Azure Active Directory.  Cela réduira le nombre d’erreurs de synchronisation vue par Azure AD Connect (ainsi que d’autres clients de synchronisation) en rendant Azure AD Azure résilient dans sa façon de traiter les attributs ProxyAddresses et UserPrincipalName en double présents dans les environnements AD. Cette fonctionnalité ne corrige pas les erreurs de duplication. Les données doivent donc être corrigées. Mais elle permet la configuration de nouveaux objets dont le provisionnement serait autrement bloqué en raison de valeurs en double dans Azure AD. Cela permet également de réduire le nombre d’erreurs de synchronisation renvoyées au client de synchronisation.
> Si cette fonctionnalité est activée pour votre client, vous ne verrez pas les erreurs de synchronisation InvalidSoftMatch lors de la configuration de nouveaux objets.
>
>

#### <a name="example-scenarios-for-invalidsoftmatch"></a>Exemples de scénarios pour InvalidSoftMatch
1. Deux objets ou plus ayant la même valeur d’attribut ProxyAddresses existent dans l’instance Active Directory locale. Un seul est provisionné dans Azure AD.
2. Deux objets ou plus ayant la même valeur d’attribut userPrincipalName existent dans l’instance Active Directory locale. Un seul est provisionné dans Azure AD.
3. Un objet a été ajouté dans l’Active Directory sur site avec la même valeur d’attribut ProxyAddresses que celle d’un objet existant dans Azure Active Directory. L’objet ajouté en local n’est pas provisionné dans Azure Active Directory.
4. Un objet a été ajouté dans l’Active Directory sur site avec la même valeur d’attribut userPrincipalName que celle d’un compte dans Azure Active Directory. L’objet n’est pas provisionné dans Azure Active Directory.
5. Un compte synchronisé a été déplacé de la forêt A vers la forêt B. Azure AD Connect (le moteur de synchronisation) utilisait l’attribut ObjectGUID pour calculer l’attribut SourceAnchor. Après le déplacement de la forêt, la valeur de l’attribut SourceAnchor est différente. Le nouvel objet (à partir de la forêt B) ne peut pas se synchroniser avec l’objet existant dans Azure AD.
6. Un objet synchronisé a été supprimé accidentellement à partir d’un Active Directory local et un nouvel objet a été créé dans Active Directory pour la même entité (par exemple, utilisateur) sans supprimer le compte dans Azure Active Directory. Le nouveau compte ne parvient pas à se synchroniser avec l’objet Azure Active Directory existant.
7. Azure AD Connect a été désinstallé et réinstallé. Pendant la réinstallation, un autre attribut a été choisi comme attribut SourceAnchor. Tous les objets qui avaient été précédemment synchronisés ont arrêté la synchronisation avec l’erreur InvalidSoftMatch.

#### <a name="example-case"></a>Exemple de scénario :
1. **Bob Smith** est un utilisateur synchronisé dans Azure Active Directory à partir d’un Active Directory local de *contoso.com*
2. La valeur **UserPrincipalName** de Bob Smith est définie sur **bobs\@contoso.com**.
3. **"abcdefghijklmnopqrstuv=="** est l’attribut **SourceAnchor** calculé par Azure AD Connect avec l’attribut **objectGUID** de Bob Smith à partir de l’Active Directory local, qui est **immutableId** pour Bob Smith dans Azure Active Directory.
4. Robert dispose des valeurs suivantes pour l’attribut **proxyAddresses** :
   * smtp : bobs@contoso.com
   * smtp : bob.smith@contoso.com
   * **smtp : bob\@contoso.com**
5. Un nouvel utilisateur, **Bob Taylor** , est ajouté à l’Active Directory local.
6. La valeur **UserPrincipalName** de Bob Taylor est définie sur **bobt\@contoso.com**.
7. **"abcdefghijkl0123456789==""** est l’attribut **sourceAnchor** calculé par Azure AD Connect avec l’attribut **objectGUID** de Bob Taylor à partir de l’Active Directory local. L’objet de Bob Taylor n’est PAS encore synchronisé sur Azure Active Directory.
8. Bob dispose des valeurs suivantes pour l’attribut proxyAddresses
   * smtp : bobt@contoso.com
   * smtp : bob.taylor@contoso.com
   * **smtp : bob\@contoso.com**
9. Lors de la synchronisation, Azure Connect AD reconnaît l’ajout de Bob Taylor dans l’Active Directory local et demande à Azure AD d’apporter la même modification.
10. Azure AD effectue d’abord la correspondance exacte. Autrement dit, il cherche si des objets avec l’attribut immutableId égal à "abcdefghijkl0123456789==". La correspondance exacte échoue car aucun autre objet dans Azure AD n’a cette valeur d’attribut immutableId.
11. Azure AD tente alors de chercher Bob Taylor par correspondance souple. Autrement dit, il cherche des objets avec l’attribut proxyAddresses égal aux trois valeurs, notamment smtp : bob@contoso.com
12. Azure AD trouver l’objet de Bob Smith correspondant aux critères de correspondance souple. Mais cet objet a la valeur d’attribut immutableId = "abcdefghijklmnopqrstuv==". qui indique cet objet a été synchronisé à partir d’un autre objet sur un Active Directory local. Par conséquent, Azure AD ne peut pas établir de correspondance souple sur ces objets et cause une erreur de synchronisation **InvalidSoftMatch**.

#### <a name="how-to-fix-invalidsoftmatch-error"></a>Procédure de correction des erreurs InvalidSoftMatch
La raison la plus courante pour l’erreur InvalidSoftMatch est que deux objets avec des valeurs \(immutableId\) SourceAnchor différentes ont la même valeur pour les attributs ProxyAddresses et/ou UserPrincipalName, qui sont utilisés pendant le processus de correspondance souple sur Azure AD. Correction de la correspondance souple invalide

1. Identifiez les valeurs dupliquées pour les attributs proxyAddresses, userPrincipalName ou autres causant l’erreur. Identifiez également les deux objets \(ou plus\) impliqués dans le conflit. Le rapport généré par [Azure AD Connect Health pour synchronisation](./how-to-connect-health-sync.md) peut vous aider à identifier les deux objets.
2. Identifiez l’objet qui doit conserver la valeur dupliquée et celui qui ne doit pas le faire.
3. Supprimez la valeur dupliquée de l’objet ne devant PAS avoir cette valeur. Vous devez apporter la modification dans le répertoire de provenance de l’objet. Dans certains cas, vous devrez peut-être supprimer un des objets en conflit.
4. Si vous avez apporté la modification dans l’Active Directory local, laissez Azure AD Connect synchroniser la modification.

Les rapports d’erreurs de synchronisation dans Azure AD Connect Health pour la synchronisation sont mis à jour toutes les 30 minutes et incluent les erreurs de la dernière tentative de synchronisation.

> [!NOTE]
> ImmutableId, par définition, ne doit pas changer tout au long de la durée de vie de l’objet. Si Azure AD Connect n’a pas été configuré avec certains des scénarios de la liste ci-dessus à l’esprit, vous pouvez vous retrouver dans une situation où Azure AD Connect calcule une valeur différente pour l’attribut SourceAnchor de l’objet Active Directory qui représente la même entité (même utilisateur/groupe/contact, etc.) ayant un objet Azure Active Directory existant que vous souhaitez continuer à utiliser.
>
>

#### <a name="related-articles"></a>Articles connexes
* [Les attributs en double ou non valides empêchent la synchronisation d’annuaires dans Microsoft 365](https://support.microsoft.com/kb/2647098)

### <a name="objecttypemismatch"></a>ObjectTypeMismatch
#### <a name="description"></a>Description
Azure AD tente d’effectuer une correspondance souple pour deux objets. Il est possible que deux objets avec des valeurs de « type d’objet » différentes (par exemple, utilisateur, groupe, contact etc.), aient les mêmes valeurs pour les attributs utilisés pour effectuer la correspondance souple. Comme la duplication de ces attributs n’est pas autorisée dans Azure AD, l’opération peut entraîner une erreur de synchronisation « ObjectTypeMismatch ».

#### <a name="example-scenarios-for-objecttypemismatch-error"></a>Exemples de scénarios pour l’erreur ObjectTypeMismatch
* Un groupe de sécurité activé pour la messagerie est créé dans Microsoft 365. L’administrateur ajoute un nouvel utilisateur ou un nouveau contact à l’Active Directory local (qui n’est pas encore synchronisé à Azure AD) avec la même valeur pour l’attribut ProxyAddresses que pour le groupe Microsoft 365.

#### <a name="example-case"></a>Exemple de scénario
1. L’administrateur crée un nouveau groupe de sécurité activé pour la messagerie dans Microsoft 365 pour le service des impôts et fournit une adresse de messagerie en tant que tax@contoso.com. Ce groupe est assigné à la valeur d’attribut ProxyAddresses de **smtp : tax\@contoso.com**
2. Un nouvel utilisateur rejoint Contoso.com et un compte est créé pour l’utilisateur local avec l’attribut proxyAddress en tant que **smtp : tax\@contoso.com**
3. Lorsqu’Azure AD Connect synchronise le nouveau compte d’utilisateur, il obtient l’erreur « ObjectTypeMismatch ».

#### <a name="how-to-fix-objecttypemismatch-error"></a>Procédure de correction des erreurs ObjectTypeMismatch
La raison la plus courante pour l’erreur ObjectTypeMismatch est que deux objets de types différents (utilisateur, groupe, contact, etc.) ont la même valeur pour l’attribut ProxyAddresses. Pour corriger l’erreur ObjectTypeMismatch :

1. Identifiez les valeurs dupliquées pour les attributs proxyAddresses (ou autres attributs) causant l’erreur. Identifiez également les deux objets \(ou plus\) impliqués dans le conflit. Le rapport généré par [Azure AD Connect Health pour synchronisation](./how-to-connect-health-sync.md) peut vous aider à identifier les deux objets.
2. Identifiez l’objet qui doit conserver la valeur dupliquée et celui qui ne doit pas le faire.
3. Supprimez la valeur dupliquée de l’objet ne devant PAS avoir cette valeur. Notez que vous devez apporter la modification dans le répertoire de provenance de l’objet. Dans certains cas, vous devrez peut-être supprimer un des objets en conflit.
4. Si vous avez apporté la modification dans l’Active Directory local, laissez Azure AD Connect synchroniser la modification. Le rapport d’erreurs de synchronisation dans Azure AD Connect Health pour synchronisation est mis à jour toutes les 30 minutes et inclut les erreurs de la dernière tentative de synchronisation.

## <a name="duplicate-attributes"></a>Attributs en double
### <a name="attributevaluemustbeunique"></a>AttributeValueMustBeUnique
#### <a name="description"></a>Description
Le schéma Azure Active Directory n’autorise de donner à deux objets ou plus la même valeur pour les attributs suivants. Chaque objet dans Azure AD est obligé d’avoir une valeur unique de ces attributs sur une instance donnée.

* ProxyAddresses
* UserPrincipalName

Si Azure AD Connect tente d’ajouter un nouvel objet ou de mettre à jour un objet existant avec une valeur qui est déjà affectée à un autre objet dans Azure Active Directory pour les attributs ci-dessus, l’opération entraîne l’erreur de synchronisation « AttributeValueMustBeUnique ».

#### <a name="possible-scenarios"></a>Scénarios possibles :
1. La valeur en double est affectée à un objet déjà synchronisé, ce qui crée un conflit avec un autre objet synchronisé.

#### <a name="example-case"></a>Exemple de scénario :
1. **Bob Smith** est un utilisateur synchronisé dans Azure Active Directory à partir d’un Active Directory local de contoso.com
2. La valeur **UserPrincipalName** locale de Bob Smith est définie sur **bobs\@contoso.com**.
3. Robert dispose des valeurs suivantes pour l’attribut **proxyAddresses** :
   * smtp : bobs@contoso.com
   * smtp : bob.smith@contoso.com
   * **smtp : bob\@contoso.com**
4. Un nouvel utilisateur, **Bob Taylor** , est ajouté à l’Active Directory local.
5. La valeur **UserPrincipalName** de Bob Taylor est définie sur **bobt\@contoso.com**.
6. **Bob Taylor** dispose des valeurs suivantes pour l’attribut **proxyAddresses** i. smtp : bobt@contoso.com ii. smtp : bob.taylor@contoso.com
7. L’objet de Bob Taylor est synchronisé avec Azure AD avec succès.
8. L’administrateur a décidé de mettre à jour l’attribut **ProxyAddresses** de Bob Taylor avec la valeur suivante : i. **smtp : bob\@contoso.com**
9. Azure AD tente de mettre à jour l’objet de Bob Taylor dans Azure AD avec la valeur ci-dessus, mais cette opération échoue car cette valeur ProxyAddresses est déjà attribuée à Bob Smith, ce qui provoque une erreur « AttributeValueMustBeUnique ».

#### <a name="how-to-fix-attributevaluemustbeunique-error"></a>Correction des erreurs AttributeValueMustBeUnique
La raison la plus courante pour l’erreur AttributeValueMustBeUnique est que deux objets avec des valeurs \(immutableId\) SourceAnchor différentes ont la même valeur pour les attributs ProxyAddresses et/ou UserPrincipalName. Correction de l’erreur AttributeValueMustBeUnique

1. Identifiez les valeurs dupliquées pour les attributs proxyAddresses, userPrincipalName ou autres causant l’erreur. Identifiez également les deux objets \(ou plus\) impliqués dans le conflit. Le rapport généré par [Azure AD Connect Health pour synchronisation](./how-to-connect-health-sync.md) peut vous aider à identifier les deux objets.
2. Identifiez l’objet qui doit conserver la valeur dupliquée et celui qui ne doit pas le faire.
3. Supprimez la valeur dupliquée de l’objet ne devant PAS avoir cette valeur. Notez que vous devez apporter la modification dans le répertoire de provenance de l’objet. Dans certains cas, vous devrez peut-être supprimer un des objets en conflit.
4. Si vous avez apporté la modification dans l’Active Directory local, laissez Azure AD Connect synchroniser la modification pour corriger l’erreur.

#### <a name="related-articles"></a>Articles connexes
-[Les attributs en double ou non valides empêchent la synchronisation d’annuaires dans Microsoft 365](https://support.microsoft.com/kb/2647098)

## <a name="data-validation-failures"></a>Échecs de validation de données
### <a name="identitydatavalidationfailed"></a>IdentityDataValidationFailed
#### <a name="description"></a>Description
Azure Active Directory applique diverses restrictions sur les données elles-mêmes avant d’autoriser les données à être écrites dans le répertoire. Ces restrictions permettent de garantir que les utilisateurs finaux bénéficient de la meilleure expérience possible lors de l’utilisation des applications qui dépendent de ces données.

#### <a name="scenarios"></a>Scénarios
a. La valeur de l’attribut UserPrincipalName comporte des caractères non valides/non pris en charge.
b. L’attribut UserPrincipalName ne suit pas le format requis.

#### <a name="how-to-fix-identitydatavalidationfailed-error"></a>Correction des erreurs IdentityDataValidationFailed
a. Assurez-vous que l’attribut userPrincipalName utilise des caractères pris en charge ainsi que le format requis.

#### <a name="related-articles"></a>Articles connexes
* [Préparation de l’approvisionnement d’utilisateurs via la synchronisation d’annuaires sur Microsoft 365](https://support.office.com/article/Prepare-to-provision-users-through-directory-synchronization-to-Office-365-01920974-9e6f-4331-a370-13aea4e82b3e)

### <a name="federateddomainchangeerror"></a>FederatedDomainChangeError
#### <a name="description"></a>Description
Cette situation entraîne l’apparition d’une erreur de synchronisation **« FederatedDomainChangeError »** lorsque le suffixe de l’attribut UserPrincipalName d’un utilisateur est modifié en remplaçant un domaine fédéré par un autre domaine fédéré.

#### <a name="scenarios"></a>Scénarios
Pour un utilisateur synchronisé, le suffixe UserPrincipalName a été modifié d’un domaine fédéré à un autre domaine fédéré en local. Par exemple, *UserPrincipalName = bob\@contoso.com* a été remplacé par *UserPrincipalName = bob\@fabrikam.com*.

#### <a name="example"></a>Exemple
1. Bob Smith, un compte pour Contoso.com, est ajouté en tant qu’un nouvel utilisateur dans Active Directory avec UserPrincipalName bob@contoso.com
2. Bob passe à une autre division Contoso.com appelée Fabrikam.com et son UserPrincipalName est modifié en bob@fabrikam.com
3. Les domaines contoso.com et fabrikam.com sont des domaines fédérés avec Azure Active Directory.
4. L’attribut UserPrincipalName n’est pas mis à jour et cause une erreur de synchronisation « FederatedDomainChangeError ».

#### <a name="how-to-fix"></a>Procédure de résolution
Si le suffixe UserPrincipalName d’un utilisateur a été mis à jour en remplaçant bob@ **contoso.com** par bob\@**fabrikam.com** , où **contoso.com** et **fabrikam.com** sont des **domaines fédérés** , procédez comme suit pour corriger l’erreur de synchronisation

1. Replacez le UserPrincipalName de l’utilisateur dans Azure AD bob@contoso.com par bob@contoso.onmicrosoft.com. Vous pouvez utiliser la commande PowerShell suivante avec le Module Azure AD PowerShell : `Set-MsolUserPrincipalName -UserPrincipalName bob@contoso.com -NewUserPrincipalName bob@contoso.onmicrosoft.com`
2. Permettez au cycle de synchronisation suivant de tenter de synchronisation. Cette fois la synchronisation sera réussie et mettra à jour l’attribut UserPrincipalName de Bob sur bob@fabrikam.com, comme prévu.

#### <a name="related-articles"></a>Articles connexes
* [Les modifications ne sont pas synchronisées par l’outil de synchronisation Azure Active Directory une fois que vous avez modifié l’UPN d’un compte utilisateur afin qu’il utilise un autre domaine fédéré](https://support.microsoft.com/help/2669550/changes-aren-t-synced-by-the-azure-active-directory-sync-tool-after-you-change-the-upn-of-a-user-account-to-use-a-different-federated-domain)

## <a name="largeobject"></a>LargeObject
### <a name="description"></a>Description
Lorsqu’un attribut dépasse la limite de taille autorisée, la longueur maximale ou le nombre défini par le schéma Azure Active Directory, l’opération de synchronisation cause l’erreur de synchronisation **LargeObject** ou **ExceededAllowedLength**. Cette erreur se produit généralement pour les attributs suivants

* userCertificate
* userSMIMECertificate
* thumbnailPhoto
* proxyAddresses

### <a name="possible-scenarios"></a>Scénarios possibles
1. L’attribut userCertificate de Bob stocke de trop nombreux certificats affectés à Bob. Ceux-ci peuvent inclure des certificats plus anciens, expirés. La limite matérielle est de 15 certificats. Pour plus d’informations sur la gestion des erreurs LargeObject avec l’attribut userCertificate, consultez à l’article [Gérer les erreurs LargeObject provoquées par l’attribut userCertificate](tshoot-connect-largeobjecterror-usercertificate.md).
2. L’attribut userSMIMECertificate de Bob stocke un trop grand nombre de certificats affectés à Bob. Ceux-ci peuvent inclure des certificats plus anciens, expirés. La limite matérielle est de 15 certificats.
3. L’attribut thumbnailPhoto de Bob défini dans Active Directory est trop volumineux pour être synchronisé dans Azure AD.
4. Lors du remplissage automatique de l’attribut ProxyAddresses dans Active Directory, un objet s’est vu affecter trop de ProxyAddresses.

### <a name="how-to-fix"></a>Procédure de résolution
1. Assurez-vous que l’attribut à l’origine de l’erreur est dans la limite autorisée.

## <a name="existing-admin-role-conflict"></a>Conflit de rôle Administrateur existant

### <a name="description"></a>Description
Un **conflit de rôle Administrateur existant** se produit sur un objet utilisateur pendant la synchronisation lorsque cet objet utilisateur a :

- des autorisations administratives et
- le même UserPrincipalName qu’un objet Azure AD existant

Azure AD Connect n’est pas autorisé à établir une correspondance souple à partir d’AD local avec un objet utilisateur dans Azure AD qui possède un rôle administratif qui lui est assigné.  Pour plus d’informations, consultez [Remplissage de UserPrincipalName dans Azure AD](plan-connect-userprincipalname.md)

![Administrateur existant](media/tshoot-connect-sync-errors/existingadmin.png)


### <a name="how-to-fix"></a>Procédure de résolution
Pour corriger ce problème, effectuez les étapes suivantes :

1. Supprimez le compte Azure AD (propriétaire) de tous les rôles d’administrateur. 
2. **Supprimez définitivement** l’objet mis en quarantaine dans le cloud. 
3. Le prochain cycle de synchronisation s’occupe de la mise en correspondance logicielle de l’utilisateur local avec le compte cloud (dans la mesure où l’utilisateur cloud n’est plus administrateur général). 
4. Restaurez les appartenances aux rôles pour le propriétaire. 

>[!NOTE]
>Vous pouvez réaffecter le rôle administratif à l’objet utilisateur existant une fois la correspondance souple entre l’objet utilisateur local et l’objet utilisateur Azure AD effectuée.

## <a name="related-links"></a>Liens connexes
* [Recherche d’objets Active Directory dans le centre d’administration Active Directory](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560661(v=ws.10))
* [Comment faire pour interroger Azure Active Directory pour un objet à l’aide d’Azure Active Directory PowerShell](/previous-versions/azure/jj151815(v=azure.100))