---
title: Résoudre des problèmes d’affectation de licence de groupe - Azure Active Directory | Microsoft Docs
description: Identification et résolution des problèmes d’affectation de licences avec la licence basée sur le groupe Azure Active Directory
services: active-directory
keywords: Gestion des licences Azure AD
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: how-to
ms.workload: identity
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: bc3dce8bb51299eb78bd2958eebf7a69594318a4
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546469"
---
# <a name="identify-and-resolve-license-assignment-problems-for-a-group-in-azure-active-directory"></a>Identification et résolution des problèmes d’affectation de licences pour un groupe dans Azure Active Directory

Les licences basées sur le groupe dans Azure Active Directory (Azure AD) introduisent le concept d’utilisateurs en état d’erreur d’attribution de licence. Cet article explique les raisons pour lesquelles les utilisateurs peuvent se trouver dans cet état.

Lorsque vous affectez directement des licences à des utilisateurs individuels, sans recourir à une licence basée sur le groupe, l’opération d’affectation peut échouer. Par exemple, lorsque vous exécutez la cmdlet PowerShell `Set-MsolUserLicense` sur le système d’un utilisateur, cette dernière peut échouer pour diverses raisons, liées à la logique métier. Par exemple, il peut y avoir un nombre insuffisant de licences ou bien un conflit entre deux plans de service qui ne peuvent pas être affectés en même temps. Le problème vous est immédiatement signalé.

Lorsque vous utilisez une licence basée sur le groupe, les mêmes erreurs peuvent se produire, mais elles apparaissent en arrière-plan lorsque le service Azure AD affecte les licences. C’est pourquoi les erreurs ne peuvent pas vous être communiquées immédiatement. Au lieu de cela, elles sont enregistrées sur l’objet utilisateur et signalées par le biais du portail d’administration. L’intention initiale, qui consiste à attribuer une licence à l’utilisateur, est toujours conservée, mais est enregistrée dans un état d’erreur. Elle fera l’objet d’un examen et d’une résolution ultérieurs.

## <a name="find-license-assignment-errors"></a>Rechercher des erreurs d’affectation de licence

### <a name="to-find-users-in-an-error-state-in-a-group"></a>Pour rechercher des utilisateurs en état d’erreur dans un groupe

1. Ouvrez la page de vue d’ensemble du groupe et sélectionnez **Licenses**. Une notification s’affiche si des utilisateurs sont en état d’erreur.

   ![Message de notifications d’erreur et de groupe](./media/licensing-groups-resolve-problems/group-error-notification.png)

1. Sélectionnez la notification pour ouvrir la liste de tous les utilisateurs concernés. Vous pouvez sélectionner chaque utilisateur individuellement pour afficher plus de détails.

   ![liste des utilisateurs en état d’erreur de licences de groupe](./media/licensing-groups-resolve-problems/list-of-users-with-errors.png)

1. Pour rechercher tous les groupes contenant au moins une erreur, dans le panneau **Azure Active Directory**, sélectionnez **Licences**, puis **Vue d’ensemble**. Une fenêtre d’informations s’affiche lorsque des groupes nécessitent votre attention.

   ![Vue d’ensemble et informations sur les groupes en état d’erreur](./media/licensing-groups-resolve-problems/group-errors-widget.png)

1. Cliquez sur la zone pour afficher la liste de tous les groupes contenant des erreurs. Vous pouvez sélectionner chaque groupe pour afficher des détails supplémentaires.

   ![Vue d’ensemble et liste des groupes contenant des erreurs](./media/licensing-groups-resolve-problems/list-of-groups-with-errors.png)

Les sections suivantes décrivent chaque problème potentiel et la manière de le résoudre.

## <a name="not-enough-licenses"></a>Nombre insuffisant de licences

**Problème :** Le nombre de licences disponibles est insuffisant pour un des produits spécifiés dans le groupe. Vous devez acheter des licences produit supplémentaires ou libérer des licences inutilisées par d’autres utilisateurs ou d’autres groupes.

Pour connaître le nombre de licences disponibles, accédez à **Azure Active Directory** > **Licences** > **Tous les produits**.

Pour voir quels utilisateurs et quels groupes utilisent des licences, sélectionnez un produit. Sous **Utilisateurs sous licence**, vous pouvez voir la liste des utilisateurs auxquels des licences ont été affectées directement, ou par le biais d’un ou de plusieurs groupes. Sous **Groupes sous licence**, vous pouvez voir tous les groupes auxquels ce produit est attribué.

**PowerShell :** Les applets de commande PowerShell signalent cette erreur comme _CountViolation_.

## <a name="conflicting-service-plans"></a>Plans de service en conflit

**Problème :** Un des produits spécifiés dans le groupe contient un plan de service en conflit avec un autre plan de service déjà attribué à l’utilisateur par le biais d’un autre produit. Certains plans de service sont configurés de sorte qu’ils ne puissent pas être attribués à l’utilisateur d’un autre plan de service lié.

Considérez l'exemple suivant. Un utilisateur possède une licence Office 365 Entreprise *E1* qui lui a été attribuée directement, avec tous les plans activés. L’utilisateur a été ajouté à un groupe auquel le produit Office 365 Entreprise *E3* est attribué. Ce produit contient des plans de service incompatibles avec les plans contenus dans E1. Par conséquent, l’attribution de la licence de groupe échoue et l’erreur « Plans de service en conflit » s’affiche. Dans cet exemple, les plans de service en conflit sont les suivants :

- SharePoint Online (Plan 2) est en conflit avec SharePoint Online (Plan 1).
- Exchange Online (Plan 2) est en conflit avec Exchange Online (Plan 1).

Pour résoudre ce conflit, vous devez désactiver deux des plans. Vous pouvez désactiver la licence E1 directement attribuée à l’utilisateur. Sinon, vous devez modifier l’attribution de licence de l’ensemble du groupe et désactiver les plans dans la licence E3. Vous pouvez également supprimer la licence E1 à partir de l’utilisateur si elle est redondante dans le cadre de la licence E3.

Seul l’administrateur peut décider de la méthode à utiliser pour résoudre les problèmes de conflit de licences produit. Azure AD ne résout pas automatiquement les conflits de licences.

**PowerShell :** Les applets de commande PowerShell signalent cette erreur comme _MutuallyExclusiveViolation_.

## <a name="other-products-depend-on-this-license"></a>D’autres produits dépendent de cette licence

**Problème :** Un des produits spécifiés dans le groupe contient un plan de service qui, pour fonctionner, doit être activé pour un autre plan de service dans un autre produit. Cette erreur se produit lorsqu’Azure AD tente de supprimer le plan de service sous-jacent. Par exemple, cela peut se produire lorsque vous supprimez l’utilisateur du groupe.

Pour résoudre ce problème, vous devez vérifier que le plan requis est toujours attribué aux utilisateurs par une autre méthode, ou que les services dépendants sont désactivés pour ces utilisateurs. Après cela, vous pouvez supprimer la licence de groupe sur ces utilisateurs.

**PowerShell :** Les applets de commande PowerShell signalent cette erreur comme _DependencyViolation_.

## <a name="usage-location-isnt-allowed"></a>L’emplacement d’utilisation n’est pas autorisé

**Problème :** Certains services Microsoft ne sont pas disponibles partout en raison de lois et réglementations locales. Avant de pouvoir attribuer une licence à un utilisateur, vous devez spécifier la propriété **Emplacement d’utilisation** pour l’utilisateur. Vous pouvez spécifier l’emplacement sous la section **Utilisateur** > **Profil** > **Modifier** du portail Azure.

Si Azure AD tente d’attribuer une licence de groupe à un utilisateur dont l’emplacement d’utilisation n’est pas pris en charge, il échoue et enregistre une erreur pour cet utilisateur.

Pour résoudre ce problème, retirez du groupe sous licence les utilisateurs associés à des emplacements non pris en charge. Autrement, si les valeurs d’emplacement d’utilisation actuelles ne représentent pas l’emplacement réel des utilisateurs, vous pouvez modifier ceux-ci de sorte que les licences soient attribuées correctement la prochaine fois (à condition que le nouvel emplacement soit pris en charge).

**PowerShell :** Les applets de commande PowerShell signalent cette erreur comme _ProhibitedInUsageLocationViolation_.

> [!NOTE]
> Lorsqu’Azure AD attribue des licences de groupe, tous les utilisateurs sans emplacement d’utilisation spécifié héritent de l’emplacement du répertoire. Nous recommandons aux administrateurs de définir des valeurs d’emplacement d’utilisation correctes pour les utilisateurs avant d’utiliser la licence groupée afin de se conformer aux lois et réglementations locales.

## <a name="duplicate-proxy-addresses"></a>Adresses proxy en double

Si vous utilisez Exchange Online, certains comptes d’utilisateur de votre organisation risquent d’être erronément configurés avec la même valeur d’adresse proxy. Lorsque la fonction de licence basée sur un groupe tente d’affecter une licence à un tel utilisateur, l’opération échoue et le message « L'adresse proxy est déjà utilisée » s’affiche.

> [!TIP]
> Pour cela, exécutez la cmdlet PowerShell suivante par rapport à Exchange Online :
> ```
> Get-Recipient -ResultSize unlimited | where {$_.EmailAddresses -match "user@contoso.onmicrosoft.com"} | fL Name, RecipientType,emailaddresses
> ```
> Pour en savoir plus sur ce problème, consultez le [message d’erreur indiquant que l’adresse de proxy est déjà utilisée dans Exchange Online](https://support.microsoft.com/help/3042584/-proxy-address-address-is-already-being-used-error-message-in-exchange-online). L’article inclut également des informations sur [la connexion à Exchange Online à l’aide de PowerShell à distance](/powershell/exchange/connect-to-exchange-online-powershell?view=exchange-ps).

Une fois les problèmes d’adresse de proxy résolus pour les utilisateurs concernés, veillez à forcer le traitement des licences sur le groupe, pour vous assurer que les licences peuvent désormais être appliquées.

## <a name="azure-ad-mail-and-proxyaddresses-attribute-change"></a>Modification d’attribut Azure AD Mail et ProxyAddresses

**Problème :** Lors de la mise à jour de l’attribution de licence sur un utilisateur ou un groupe, vous pouvez voir que les attributs Azure AD Mail et ProxyAddresses de certains utilisateurs sont modifiés.

La mise à jour d’attribution de licence sur un utilisateur entraîne le déclenchement du calcul de l’adresse proxy, ce qui peut modifier les attributs de l’utilisateur. Pour comprendre la raison exacte de la modification et résoudre le problème, consultez cet article [Comment l’attribut proxyAddresses est renseigné dans Azure AD](https://support.microsoft.com/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad).

## <a name="licenseassignmentattributeconcurrencyexception-in-audit-logs"></a>LicenseAssignmentAttributeConcurrencyException dans les journaux d’audit

**Problème :** L’utilisateur dispose de LicenseAssignmentAttributeConcurrencyException pour l’affectation de licence dans les journaux d’audit.
Lorsque la gestion des licences basée sur les groupes tente d’affecter simultanément la même licence à un utilisateur, cette exception est enregistrée sur l’utilisateur. Cela se produit généralement lorsqu’un utilisateur est membre de plusieurs groupes avec la même licence affectée. Azure AD va réessayer de traiter la licence d'utilisation et va résoudre le problème. Aucune action du client n’est requise pour résoudre ce problème.

## <a name="more-than-one-product-license-assigned-to-a-group"></a>Plusieurs licences de produit affectées à un groupe

Vous pouvez attribuer plusieurs licences produit à un même groupe. Par exemple, vous pouvez attribuer Office 365 Entreprise E3 et Enterprise Mobility + Security à un groupe afin de faciliter l’activation de tous les services inclus pour les utilisateurs.

Azure AD tente d’attribuer toutes les licences spécifiées dans le groupe à chaque utilisateur. Si Azure AD ne peut pas affecter l’un des produits en raison de problèmes liés à la logique métier, il ne pourra pas non plus affecter les autres licences du groupe. Exemple : si le nombre de licences est insuffisant, ou en cas de conflit avec les autres services activés pour l’utilisateur.

Vous pouvez voir les utilisateurs pour lesquels l’attribution a échoué et vérifier les produits concernés.

## <a name="when-a-licensed-group-is-deleted"></a>Quand un groupe sous licence est supprimé

Vous devez supprimer toutes les licences attribuées à un groupe pour pouvoir le supprimer. Toutefois, la suppression des licences de tous les utilisateurs dans le groupe peut prendre du temps. La suppression d’attributions de licence d’un groupe peut échouer si l’utilisateur a une licence dépendante attribuée ou en cas de conflit d’adresse proxy qui interdit la suppression de la licence. Si un utilisateur a une licence qui dépend d’une licence en cours de suppression dans le cadre d’une suppression de groupe, l’attribution de licence héritée de l’utilisateur est convertie en attribution directe.

Par exemple, prenons un groupe avec une licence Office 365 E3/E5 attribuée avec un plan de service Skype Entreprise activé. Imaginez que certains membres du groupe ont des licences d’audioconférence attribuées directement. Quand le groupe est supprimé, la gestion des licences attribuées au groupe tente de supprimer Office 365 E3/E5 pour tous les utilisateurs. Comme l’audioconférence dépend de Skype Entreprise, pour tous les utilisateurs bénéficiant de l’audioconférence, la gestion des licences attribuées au groupe convertit les licences Office 365 E3/E5 en attributions de licence directes.

## <a name="manage-licenses-for-products-with-prerequisites"></a>Gérer les licences des produits avec des prérequis

Certains produits Microsoft Online que vous possédez peut-être sont des *modules complémentaires*. Ils nécessitent l’activation d’un plan de service requis pour un utilisateur ou un groupe pour qu’une licence puisse leur être attribuée. Avec les licences basées sur le groupe, le système impose que les plans de service requis et du module complémentaire soient présents pour le même groupe. Cela permet de veiller à ce que tous les utilisateurs ajoutés au groupe puissent recevoir le produit de travail complet. Considérez l’exemple suivant :

Microsoft Workplace Analytics est un produit additionnel. Il contient un plan de service unique portant le même nom. Ce plan de service peut uniquement être affecté à un utilisateur ou à un groupe, lorsque l’une des conditions requises suivantes est également assignée :

- Exchange Online (plan 1)
- Exchange Online (plan 2)

Si nous essayons d’affecter ce produit seul à un groupe, le portail renvoie un message de notification. Si nous sélectionnons les détails, le message d’erreur suivant s’affiche :

  « Échec de l’opération de licence. Assurez-vous que le groupe possède les services nécessaires avant d’ajouter ou de retirer un service dépendant. **Le service Microsoft Workplace Analytics nécessite Exchange Online (plan 2) pour être activé également.**  »

Pour attribuer cette licence de module complémentaire à un groupe, nous devons nous assurer que le groupe contient également le plan de service requis. Par exemple, nous pouvons mettre à jour un groupe existant qui contient déjà le produit Office 365 E3 complet, et lui ajouter le produit additionnel.

Il est également possible de créer un groupe autonome contenant uniquement les produits requis pour que le module additionnel fonctionne. Il peut être utilisé pour fournir une licence uniquement aux utilisateurs sélectionnés pour le produit additionnel. Selon l’exemple précédent, vous devez attribuer les produits suivants au même groupe :

- Office 365 Enterprise E3, avec uniquement le plan de service Exchange Online (plan 2) activé
- Microsoft Workplace Analytics

Dorénavant, tout utilisateur ajouté à ce groupe utilise une licence de produit E3 et une licence de produit Workplace Analytics. Dans le même temps, ces utilisateurs peuvent être membres d’un autre groupe leur fournissant le produit E3 complet ; ils ne consommeront alors qu’une seule licence de ce produit.

> [!TIP]
> Vous pouvez créer plusieurs groupes pour chaque plan de service requis. Par exemple, si vous utilisez Office 365 Enterprise E1 et Office 365 Enterprise E3 pour vos utilisateurs, vous pouvez créer deux groupes pour accorder une licence Microsoft Workplace Analytics : un groupe demandant E1 comme condition préalable et un autre demandant E3. Cela vous permettra de distribuer le module complémentaire aux utilisateurs de E1 et de E3 sans consommer de licences supplémentaires.

## <a name="force-group-license-processing-to-resolve-errors"></a>Forcer le traitement des licences de groupe pour résoudre des erreurs

Selon les actions entreprises pour résoudre les erreurs, il peut être nécessaire de déclencher manuellement le traitement d’un groupe pour mettre à jour l’état de l’utilisateur.

Par exemple, si vous avez libéré des licences en supprimant leurs affectations directes à des utilisateurs, vous devez déclencher le traitement des groupes ayant échoué précédemment afin d’attribuer des licences complètes à tous les utilisateurs membres. Pour traiter à nouveau un groupe, accédez au volet correspondant, ouvrez **Licences**, puis sélectionnez le bouton **Retraiter** dans la barre d’outils.

## <a name="force-user-license-processing-to-resolve-errors"></a>Forcer le traitement des licences utilisateur pour résoudre des erreurs

Selon les mesures que vous avez prises pour résoudre les erreurs, il peut être nécessaire de déclencher manuellement le traitement d’un utilisateur pour mettre à jour l’état de l’utilisateur.

Par exemple, après avoir résolu le problème d’adresse proxy en double pour un utilisateur affecté, vous devez déclencher le traitement de l’utilisateur. Pour retraiter un utilisateur , accédez au volet correspondant, ouvrez **Licences**, puis sélectionnez le bouton **Retraiter** dans la barre d’outils.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres scénarios de gestion des licences par le biais des groupes, consultez les articles suivants :

* [What is group-based licensing in Azure Active Directory? (Présentation des licences basées sur le groupe dans Azure Active Directory)](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [Affectation de licences à un groupe dans Azure Active Directory](./licensing-groups-assign.md)
* [Migration des utilisateurs individuels sous licence vers les licences basées sur les groupes dans Azure Active Directory](licensing-groups-migrate-users.md)
* [Guide pratique pour migrer des utilisateurs entre des licences de produit à l’aide de la gestion de licences basée sur des groupes dans Azure Active Directory](licensing-groups-change-licenses.md)
* [Azure Active Directory group-based licensing additional scenarios (Autres scénarios de licence basée sur le groupe Azure Active Directory)](./licensing-group-advanced.md)
* [Exemples PowerShell pour les licences basées sur les groupes dans Azure Active Directory](licensing-ps-examples.md)