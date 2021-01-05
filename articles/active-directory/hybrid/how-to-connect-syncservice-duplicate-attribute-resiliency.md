---
title: Synchronisation des identités et résilience d’attribut en double | Microsoft Docs
description: Nouveau comportement de gestion des objets présentant des conflits UPN ou ProxyAddress pendant la synchronisation d’annuaires à l’aide d’Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 537a92b7-7a84-4c89-88b0-9bce0eacd931
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/15/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e09dd6a127bd04ae698cb6cad2ffd7f35e3b51c3
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413426"
---
# <a name="identity-synchronization-and-duplicate-attribute-resiliency"></a>Synchronisation des identités et résilience d’attribut en double
La résilience d’attribut en double est une fonctionnalité d’Azure Active Directory qui élimine les problèmes liés aux conflits entre les valeurs **UserPrincipalName** et **ProxyAddress** SMTP lors de l’exécution de l’un des outils de synchronisation de Microsoft.

Ces deux attributs doivent généralement être uniques pour tous les objets **Utilisateur** , **Groupe** , ou **Contact** dans un client Azure Active Directory donné.

> [!NOTE]
> Seuls les Utilisateurs peuvent avoir des noms UPN.
> 
> 

Cette fonctionnalité active un nouveau comportement qui se trouve dans la partie cloud du pipeline de synchronisation. Par conséquent, cette fonctionnalité convient à tout type de client et pour tout produit de synchronisation Microsoft, y compris Azure AD Connect, DirSync et MIM + Connector. Le terme générique « client de synchronisation » désigne ces produits dans le présent document.

## <a name="current-behavior"></a>Comportement actuel
En cas de tentative d’approvisionnement d’un nouvel objet avec une valeur UPN ou ProxyAddress qui enfreint cette contrainte d’unicité, Azure Active Directory bloque la création de l’objet. De même, si un objet est mis à jour avec une valeur UPN ou ProxyAddress qui n’est pas unique, la mise à jour échoue. Le client de synchronisation refait la tentative d’approvisionnement ou la mise à jour à chaque cycle d’exportation ; il échoue à chaque fois jusqu’à la résolution du conflit. Chaque tentative infructueuse génère un e-mail contenant un rapport d’erreur, et une erreur est consignée par le client de synchronisation.

## <a name="behavior-with-duplicate-attribute-resiliency"></a>Comportement avec une résilience d’attribut en double
Au lieu de rejeter l’approvisionnement ou la mise à jour d’un objet comportant un attribut en double, Azure Active Directory met en « quarantaine » l’attribut en double qui enfreint la contrainte d’unicité. Si cet attribut est requis pour l’approvisionnement, comme pour UserPrincipalName, le service affecte une valeur d’espace réservé. Le format de ces valeurs temporaires est  
_**\<OriginalPrefix>+\<4DigitNumber>\@\<InitialTenantDomain>.onmicrosoft.com**_.

Le processus de résilience d’attribut gère uniquement les valeurs **ProxyAddress** UPN et SMTP.

Si l’attribut n’est pas obligatoire, comme **ProxyAddress** , Azure Active Directory met simplement en quarantaine l’attribut à l’origine du conflit et poursuit la création ou la mise à jour de l’objet.

Lorsque l’attribut est mis en quarantaine, des informations sur le conflit sont envoyées dans le même e-mail de rapport d’erreur utilisé avec l’ancien comportement. Toutefois, ces informations n’apparaissent qu’une fois dans le rapport d’erreurs (lors de la mise en quarantaine) ; elles ne sont pas consignées dans les e-mails suivants. En outre, étant donné que l’exportation de cet objet a réussi, le client de synchronisation ne consigne pas d’erreur et ne retente pas la création/la mise à jour lors des cycles de synchronisation suivants.

Pour prendre en charge ce comportement, un nouvel attribut a été ajouté aux classes d’objets Utilisateur, Groupe et Contact :  
**DirSyncProvisioningErrors**

Il s’agit d’un attribut à valeurs multiples utilisé pour stocker les attributs en conflit qui enfreindraient la contrainte d’unicité s’ils étaient ajoutés normalement. Une tâche du minuteur d’arrière-plan a été activée dans Azure Active Directory. Celle-ci s’exécute toutes les heures pour rechercher des conflits d’attributs en double ayant été résolus, puis supprime automatiquement les attributs en question de la quarantaine.

### <a name="enabling-duplicate-attribute-resiliency"></a>Activer la résilience d’attribut en double
La résilience d’attribut en double sera le nouveau comportement par défaut sur tous les locataires Azure Active Directory. Elle sera activée par défaut pour tous les clients qui ont activé la synchronisation pour la première fois le 22 août 2016 ou plus tard. Les clients qui ont activé la synchronisation avant cette date verront cette fonctionnalité activée par lots. Ce déploiement a commencé en septembre 2016, et une notification par courrier électronique sera envoyée au contact de notification technique de chaque client la date d’activation de la fonctionnalité.

> [!NOTE]
> Une fois que la résilience des attributs en double a été activée, elle ne peut pas être désactivée.

Vous pouvez vérifier si cette fonctionnalité est activée sur votre client en téléchargeant la dernière version du module Azure Active Directory PowerShell, puis en exécutant ce qui suit :

`Get-MsolDirSyncFeatures -Feature DuplicateUPNResiliency`

`Get-MsolDirSyncFeatures -Feature DuplicateProxyAddressResiliency`

> [!NOTE]
> Vous pouvez n’est plus utiliser l’applet de commande Set-MsolDirSyncFeature pour activer de manière proactive la fonctionnalité de résilience des attributs en double avant qu’elle ne soit activée pour votre locataire. Pour pouvoir tester la fonctionnalité, vous devez créer un nouveau locataire Azure Active Directory.

## <a name="identifying-objects-with-dirsyncprovisioningerrors"></a>Identification des objets avec DirSyncProvisioningErrors
Il existe actuellement deux méthodes pour identifier les objets qui comportent ces erreurs en raison de conflits de propriété dupliquée : Azure Active Directory PowerShell et le [centre d’administration Microsoft 365](https://admin.microsoft.com). Il est prévu d’augmenter la capacité de génération de rapports dans le portail.

### <a name="azure-active-directory-powershell"></a>Azure Active Directory PowerShell
Pour les applets de commande PowerShell dans cette rubrique, les conditions suivantes sont vérifiées :

* Toutes les applets de commande suivantes sont sensibles à la casse.
* L’indicateur **–ErrorCategory PropertyConflict** doit toujours être inclus. Il n’existe actuellement pas d’autres types de **ErrorCategory** , mais cela pourrait changer.

Commencez par exécuter **Connect-MsolService** et entrer les informations d’identification d’un administrateur client.

Utilisez ensuite les applets de commande et les opérateurs suivants pour afficher les erreurs de différentes manières :

1. [Afficher tout](#see-all)
2. [Par type de propriété](#by-property-type)
3. [Par valeur en conflit](#by-conflicting-value)
4. [À l’aide d’une recherche de chaîne](#using-a-string-search)
5. Triées
6. [Par quantité limitée ou l’ensemble des erreurs](#in-a-limited-quantity-or-all)

#### <a name="see-all"></a>Afficher tout
Une fois connecté, exécutez ce qui suit pour voir une liste générale d’erreurs d’approvisionnement d’attribut du client :

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict`

Le résultat ressemble à ce qui suit :  
 ![Get-MsolDirSyncProvisioningError](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/1.png "Get-MsolDirSyncProvisioningError")  

#### <a name="by-property-type"></a>Par type de propriété
Pour voir les erreurs par type de propriété, ajoutez l’indicateur **-PropertyName** à l’argument **UserPrincipalName** ou **ProxyAddresses** :

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyName UserPrincipalName`

ou

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyName ProxyAddresses`

#### <a name="by-conflicting-value"></a>Par valeur en conflit
Pour afficher les erreurs concernant une propriété spécifique, ajoutez l’indicateur **-PropertyValue** ( **-PropertyName** doit également être utilisé avec cet indicateur) :

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyValue User@domain.com -PropertyName UserPrincipalName`

#### <a name="using-a-string-search"></a>À l’aide d’une recherche de chaîne
Pour effectuer une recherche de chaîne élargie, utilisez l’indicateur **-SearchString** . Il peut s’utiliser indépendamment des autres indicateurs ci-dessus, à l’exception de **-ErrorCategory PropertyConflict** , qui est toujours requis :

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -SearchString User`

#### <a name="in-a-limited-quantity-or-all"></a>Par quantité limitée ou l’ensemble des erreurs
1. **MaxResults \<Int>** peut être utilisé pour limiter la requête à un nombre spécifique de valeurs.
2. **All** permet de vérifier que tous les résultats sont récupérés, notamment si le nombre d’erreurs est élevé.

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -MaxResults 5`

## <a name="microsoft-365-admin-center"></a>Centre d’administration Microsoft 365
Vous pouvez afficher les erreurs de synchronisation d’annuaires dans le Centre d’administration Microsoft 365. Le rapport dans le centre d’administration Microsoft 365 n’affiche que les objets **Utilisateur** qui présentent ces erreurs. Il n’indique pas d’informations sur les conflits entre **Groupes** et **Contacts**.

![Screenshot that shows directory synchronization errors in the Microsoft 365 admin center.](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/1234.png "Active users")

Pour obtenir des instructions sur l’affichage des erreurs de synchronisation d’annuaires dans le Centre d’administration Microsoft 365, consultez [Identifier les erreurs de synchronisation d’annuaires dans Microsoft 365](https://support.office.com/article/Identify-directory-synchronization-errors-in-Office-365-b4fc07a5-97ea-4ca6-9692-108acab74067).

### <a name="identity-synchronization-error-report"></a>Rapport d’erreur de synchronisation d’identité
Lorsqu’un objet présentant un conflit d’attribut en double est traité avec ce nouveau comportement, une notification afférente est incluse dans l’e-mail standard contenant le rapport d’erreur de synchronisation d’identité. Ce dernier est envoyé au contact du client en charge des notifications techniques. Toutefois, ce comportement présente un changement majeur. Auparavant, les informations de conflit d’attribut en double apparaissaient dans chaque rapport d’erreurs généré jusqu’à la résolution du conflit. Avec ce nouveau comportement, la notification d’erreur pour un conflit donné n’apparaît qu’une fois : au moment où l’attribut en conflit est mis en quarantaine.

Voici un exemple de notification par e-mail d’un conflit ProxyAddress :  
    ![Screenshot that shows an example of an email notification for a ProxyAddress conflict.](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/6.png "Active users")  

## <a name="resolving-conflicts"></a>Résolution des conflits
Les stratégies et tactiques de résolution des problèmes pour ces erreurs ne doivent pas différer de la façon dont les erreurs d’attribut en double ont été traitées par le passé. La seule différence est que la tâche du minuteur effectue un balayage du client côté serveur afin d’ajouter automatiquement l’attribut en question à l’objet concerné lorsque le conflit est résolu.

L’article suivant décrit les différentes stratégies de dépannage et de résolution : [Les attributs en double ou non valides empêchent la synchronisation d’annuaires dans Office 365](https://support.microsoft.com/kb/2647098).

## <a name="known-issues"></a>Problèmes connus
Aucun de ces problèmes connus n’entraîne une dégradation du service ou une perte des données. Plusieurs d’entre eux relèvent de l’esthétisme ; d’autres génèrent des erreurs d’attribut en double (« *pré-résilience* »), au lieu de mettre en quarantaine l’attribut à l’origine du conflit ; un dernier requiert un travail de correction manuelle supplémentaire pour certaines erreurs.

**Comportement de base :**

1. Les objets ayant une configuration d’attribut spécifique continuent à recevoir des erreurs d’exportation ; les attributs dupliqués ne sont pas mis en quarantaine.  
   Par exemple :
   
    a. Un nouvel utilisateur est créé dans AD avec un UPN **Joe\@contoso.com** et une ProxyAddress **smtp:Joe\@contoso.com**
   
    b. Les propriétés de cet objet sont en conflit avec un Groupe existant, où ProxyAddress est **SMTP:Joe\@contoso.com**.
   
    c. Lors de l’exportation, une erreur de **conflit ProxyAddress** est générée au lieu de la mise en quarantaine des attributs à l’origine du conflit. L’opération est retentée à chaque cycle de synchronisation, comme cela était le cas avant l’activation de la fonction de résilience.
2. Si deux Groupes sont créés en local avec la même adresse SMTP, l’approvisionnement de l’un d’entre eux échoue à la première tentative, ce qui génère une erreur standard d’attribut **ProxyAddress** en double. Toutefois, la valeur en double est bien mise en quarantaine lors du prochain cycle de synchronisation.

**Rapport du portail Office** :

1. Le message d’erreur détaillé pour deux objets dans un ensemble de conflit UPN est le même. Cela indique que l’UPN des deux objets a changé/été mis en quarantaine, alors que seules les données de l’un d’entre eux ont changé.
2. Le message d’erreur détaillé d’un conflit UPN affiche une propriété displayName incorrecte pour un utilisateur dont l’UPN a changé/été mis en quarantaine. Par exemple :
   
    a. **L’utilisateur A** est d’abord synchronisé avec **UPN = User\@contoso.com**.
   
    b. **L’utilisateur B** fait ensuite l’objet d’une tentative de synchronisation avec **UPN = User\@contoso.com**.
   
    c. **L’UPN de l’utilisateur B** est remplacée par **User1234\@contoso.onmicrosoft.com** et **User\@contoso.com** est ajouté à **DirSyncProvisioningErrors** .
   
    d. Le message d’erreur de l’ **utilisateur B** doit indiquer que l’ **utilisateur A** a déjà **User\@contoso.com** comme UPN, mais il affiche le paramètre displayName de l’ **utilisateur B**.

**Rapport d’erreur de synchronisation d’identité** :

Le lien pour la *procédure à suivre pour résoudre ce problème* est incorrect :  
    ![Utilisateurs actifs](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/6.png "Active users")  

Il doit pointer vers [https://aka.ms/duplicateattributeresiliency]().

## <a name="see-also"></a>Voir aussi
* [Synchronisation d’Azure AD Connect](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
* [Identifier les erreurs de synchronisation d’annuaires dans Microsoft 365](https://support.office.com/article/Identify-directory-synchronization-errors-in-Office-365-b4fc07a5-97ea-4ca6-9692-108acab74067)