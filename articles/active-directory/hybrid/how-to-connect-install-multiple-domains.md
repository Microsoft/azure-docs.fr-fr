---
title: Domaines multiples Azure AD Connect
description: Ce document décrit la définition et la configuration de plusieurs domaines de premier niveau avec Microsoft 365 et Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 5595fb2f-2131-4304-8a31-c52559128ea4
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/31/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53a0da5b5db21c9a543d39d1b252b0b4c64e2a56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91306359"
---
# <a name="multiple-domain-support-for-federating-with-azure-ad"></a>Prise en charge de plusieurs domaines pour la fédération avec Azure AD
La documentation suivante fournit des conseils sur l’utilisation de plusieurs domaines de niveau supérieur et sous-domaines lors de la fédération avec des domaines Microsoft 365 ou Azure AD.

## <a name="multiple-top-level-domain-support"></a>Prise en charge de plusieurs domaines de niveau supérieur
La fédération de plusieurs domaines de niveau supérieur avec Azure AD nécessite une configuration supplémentaire qui n’est pas requise lors de la fédération avec un domaine de niveau supérieur.

Lorsqu’un domaine est fédéré avec Azure AD, plusieurs propriétés sont définies sur le domaine dans Azure.  L’une des plus importantes est IssuerUri.  Cette propriété est un URI qui est utilisé par Azure AD pour identifier le domaine auquel le jeton est associé.  L’URI n’a pas besoin de résoudre quoi que ce soit, mais il doit s’agir d’un URI valide.  Par défaut, AD Azure affecte l’URI à la valeur de l’identificateur du service de fédération dans votre configuration AD FS locale.

> [!NOTE]
> L’identificateur du service de fédération est un URI qui identifie de façon unique un service de fédération.  Le service de fédération est une instance d’AD FS qui fonctionne en tant que service du jeton de sécurité.
>
>

Vous pouvez afficher IssuerUri à l’aide de la commande PowerShell `Get-MsolDomainFederationSettings -DomainName <your domain>`.

![Capture d’écran montrant des résultats après l’entrée de la commande « Get-MsolDomainFederationSettings » dans PowerShell.](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

Un problème survient lorsque vous ajoutez plusieurs domaines de niveau supérieur.  Par exemple, supposons que vous avez configuré la fédération entre Azure AD et votre environnement local.  Pour ce document, le domaine bmcontoso.com est utilisé.  Un second domaine de premier niveau, bmfabrikam.com, est alors ajouté.

![Capture d’écran montrant plusieurs domaines de niveau supérieur](./media/how-to-connect-install-multiple-domains/domains.png)

Lorsque vous essayez de convertir le domaine bmfabrikam.com pour qu’il soit fédéré, une erreur se produit.  La raison de cette erreur tient à une contrainte dans Azure AD qui n’autorise pas IssuerURI à avoir la même valeur pour plusieurs domaines.  

![Capture d’écran montrant une erreur de fédération dans PowerShell.](./media/how-to-connect-install-multiple-domains/error.png)

### <a name="supportmultipledomain-parameter"></a>Paramètre SupportMultipleDomain
Pour contourner cette contrainte, nous devons ajouter un IssuerUri différent, ce qui peut être effectué à l’aide du paramètre `-SupportMultipleDomain`.  Ce paramètre est utilisé avec les applets de commande suivantes :

* `New-MsolFederatedDomain`
* `Convert-MsolDomaintoFederated`
* `Update-MsolFederatedDomain`

Ce paramètre permet à Azure AD de configurer l’IssuerUri afin qu’il soit basé sur le nom du domaine.  IssuerUri sera unique au sein des annuaires dans Azure AD.  L’utilisation du paramètre permet à la commande PowerShell de s’exécuter correctement.

![Capture d’écran montrant la réussite de l’exécution de la commande PowerShell.](./media/how-to-connect-install-multiple-domains/convert.png)

Les paramètres du domaine bmfabrikam.com ressemblent à ce qui suit :

![Capture d’écran montrant les paramètres du domaine « bmfabrikam.com ».](./media/how-to-connect-install-multiple-domains/settings.png)

`-SupportMultipleDomain` ne modifie pas les autres points de terminaison qui sont toujours configurés pour pointer vers le service de fédération adfs.bmcontoso.com.

De plus, `-SupportMultipleDomain` s’assure que le système AD FS inclut la valeur Issuer appropriée dans les jetons émis pour Azure AD. Pour définir cette valeur, la partie domaine de l’UPN des utilisateurs est prise et définie en tant que domaine dans issuerURI, c’est-à-dire https://{upn suffix}/adfs/services/trust.

Ainsi, pendant l’authentification auprès d’Azure AD ou de Microsoft 365, l’élément IssuerUri du jeton de l’utilisateur est employé pour localiser le domaine dans Azure AD. Si aucune correspondance ne peut être trouvée, l’authentification échoue.

Par exemple, si l’UPN d’un utilisateur est bsimon@bmcontoso.com, l’élément IssuerUri dans les émissions AD FS du jeton a la valeur `http://bmcontoso.com/adfs/services/trust`. Cet élément correspond à la configuration Azure AD et donc l’authentification réussit.

Vous trouverez ci-dessous la règle de revendication personnalisée qui implémente cette logique :

```
c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)", "http://${domain}/adfs/services/trust/"));
```


> [!IMPORTANT]
> Afin d’utiliser le commutateur -SupportMultipleDomain lors de la tentative d’ajout de nouveaux domaines ou la conversion de domaines déjà existants, votre approbation fédérée doit déjà avoir été configurée pour les prendre en charge.
>
>

## <a name="how-to-update-the-trust-between-ad-fs-and-azure-ad"></a>Mise à jour de l’approbation entre AD FS et Azure AD
Si vous n’avez pas configuré l’approbation fédérée entre AD FS et votre instance Azure AD, vous devrez peut-être recréer cette approbation.  En effet, lorsqu’elle est configurée sans le paramètre `-SupportMultipleDomain`, l’élément IssuerUri est défini sur la valeur par défaut.  Dans la capture d’écran ci-dessous, vous pouvez voir que IssuerUri a la valeur `https://adfs.bmcontoso.com/adfs/services/trust`.

Si vous avez correctement ajouté un nouveau domaine dans le portail Azure AD, puis que vous essayez de le convertir à l’aide de `Convert-MsolDomaintoFederated -DomainName <your domain>`, vous obtenez l’erreur suivante.

![Capture d’écran montrant une erreur de fédération dans PowerShell après une tentative de conversion d’un nouveau domaine à l’aide de la commande « Convert-MsolDomaintoFederated ».](./media/how-to-connect-install-multiple-domains/trust1.png)

Si vous essayez d’ajouter le commutateur `-SupportMultipleDomain`, vous recevez l’erreur suivante :

![Capture d’écran montrant une erreur de fédération après l’ajout du commutateur « -SupportMultipleDomain ».](./media/how-to-connect-install-multiple-domains/trust2.png)

Tenter d’exécuter `Update-MsolFederatedDomain -DomainName <your domain> -SupportMultipleDomain` sur le domaine d’origine entraîne également une erreur.

![Erreur de fédération](./media/how-to-connect-install-multiple-domains/trust3.png)

Utilisez les étapes ci-dessous pour ajouter un domaine de niveau supérieur supplémentaire.  Si vous avez déjà ajouté un domaine sans utiliser le paramètre `-SupportMultipleDomain`, commencez par supprimer et mettre à jour votre domaine d’origine.  Si vous n’avez pas encore ajouté un domaine de niveau supérieur, vous pouvez commencer par ajouter un domaine à l’aide de PowerShell d’Azure AD Connect.

Utilisez les étapes suivantes pour supprimer l’approbation de Microsoft Online et mettre à jour votre domaine d’origine.

1. Sur votre serveur de fédération AD FS, ouvrez **Gestion AD FS.**
2. Dans le volet gauche, développez **Relations d’approbation** et **Approbations de la partie de confiance**
3. Sur la droite, supprimez l’entrée **Plateforme d’identité Microsoft Office 365** .
   ![Supprimer Microsoft en ligne](./media/how-to-connect-install-multiple-domains/trust4.png)
4. Sur un ordinateur avec [Module Azure Active Directory pour Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)) installé, exécutez la commande suivante : `$cred=Get-Credential`.  
5. Entrez le nom d’utilisateur et le mot de passe d’un administrateur global pour le domaine Azure AD avec lequel vous voulez fédérer.
6. Dans PowerShell, entrez `Connect-MsolService -Credential $cred`
7. Dans PowerShell, entrez `Update-MSOLFederatedDomain -DomainName <Federated Domain Name> -SupportMultipleDomain`.  Cette mise à jour concerne le domaine d’origine.  Donc, en utilisant les domaines ci-dessus, cela donnerait : `Update-MsolFederatedDomain -DomainName bmcontoso.com -SupportMultipleDomain`

Procédez comme suit pour ajouter le nouveau domaine de niveau supérieur à l’aide de PowerShell

1. Sur un ordinateur avec [Module Azure Active Directory pour Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)) installé, exécutez la commande suivante : `$cred=Get-Credential`.  
2. Entrez le nom d’utilisateur et le mot de passe d’un administrateur global pour le domaine Azure AD avec lequel vous voulez fédérer
3. Dans PowerShell, entrez `Connect-MsolService -Credential $cred`
4. Dans PowerShell, entrez `New-MsolFederatedDomain –SupportMultipleDomain –DomainName`

Procédez comme suit pour ajouter le nouveau domaine de niveau supérieur à l’aide d’Azure AD Connect.

1. Lancez Azure AD Connect à partir du bureau ou du menu Démarrer
2. Choisissez « Ajouter un domaine Azure AD supplémentaire » ![Capture d’écran montrant la page « Tâches supplémentaires » avec l’option « Ajouter un domaine Azure AD supplémentaire » sélectionnée.](./media/how-to-connect-install-multiple-domains/add1.png)
3. Entrez votre informations d’identification Azure AD et Active Directory
4. Sélectionnez le second domaine que vous souhaitez configurer pour la fédération.
   ![Ajouter un domaine Azure AD supplémentaire](./media/how-to-connect-install-multiple-domains/add2.png)
5. Cliquez sur Installer

### <a name="verify-the-new-top-level-domain"></a>Vérifiez le nouveau domaine de niveau supérieur
À l’aide de la commande PowerShell `Get-MsolDomainFederationSettings -DomainName <your domain>`vous pouvez afficher l’IssuerUri mis à jour.  La capture d’écran ci-dessous montre les paramètres de la fédération mis à jour sur le domaine d’origine `http://bmcontoso.com/adfs/services/trust`

![Capture d’écran montrant les paramètres de fédération mis à jour sur le domaine d’origine.](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

Et IssuerUri sur le nouveau domaine a été défini sur `https://bmfabrikam.com/adfs/services/trust`

![Get-MsolDomainFederationSettings](./media/how-to-connect-install-multiple-domains/settings2.png)

## <a name="support-for-subdomains"></a>Prise en charge des sous-domaines
Lorsque vous ajoutez un sous-domaine, en raison de la façon dont Azure AD a géré les domaines, il héritera des paramètres du parent.  Donc, IssuerUri doit correspondre aux parents.

Donc, supposons que j’ai bmcontoso.com et que j’ajoute ensuite corp.bmcontoso.com.  Pour un nouvel utilisateur de corp.bmcontoso.com, l’IssuerUri doit être défini sur **`http://bmcontoso.com/adfs/services/trust`** .  Toutefois, la règle standard implémentée ci-dessus pour Azure AD génère un jeton avec un émetteur **`http://corp.bmcontoso.com/adfs/services/trust`** . ce qui ne correspondra pas à la valeur requise du domaine et l’authentification échouera.

### <a name="how-to-enable-support-for-subdomains"></a>Activation de la prise en charge des sous-domaines
Pour contourner ce problème, l’approbation de la partie de confiance AD FS de Microsoft Online doit être mise à jour.  Pour cela, vous devez configurer une règle de revendication personnalisée afin qu’elle retire tous les sous-domaines du suffixe UPN de l’utilisateur pendant la construction de la valeur Issuer.

La revendication suivante fait cela :

```    
c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));
```

[!NOTE]
Le dernier numéro de l’expression régulière définit le nombre de domaines parents dans votre domaine racine. Dans le cas présent, bmcontoso.com est utilisé. Donc, deux domaines parents sont nécessaires. Si trois domaines parents doivent être conservés (par exemple corp.bmcontoso.com), le nombre indiqué est « trois ». Vous pouvez si vous le souhaitez indiquer une plage. Dans ce cas, la correspondance est toujours effectuée avec un maximum de domaines. « {2,3} » correspond à deux ou trois domaines (par exemple bmfabrikam.com et corp.bmcontoso.com).

Utilisez les étapes suivantes pour ajouter une revendication personnalisée pour prendre en charge des sous-domaines.

1. Ouvrez Gestion AD FS
2. Cliquez avec le bouton droit sur l’approbation de la partie de confiance Microsoft Online et choisissez Modifier les règles de revendication
3. Sélectionnez la troisième règle de revendication et remplacez ![Modifier la revendication](./media/how-to-connect-install-multiple-domains/sub1.png)
4. Remplacez la revendication en cours :

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)","http://${domain}/adfs/services/trust/"));
   ```
    par

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));
   ```

    ![Remplacer la revendication](./media/how-to-connect-install-multiple-domains/sub2.png)

5. Cliquez sur OK.  Cliquez sur Appliquer.  Cliquez sur OK.  Fermez Gestion AD FS.

## <a name="next-steps"></a>Étapes suivantes
Azure AD Connect étant installé, vous pouvez passer à [Vérification de l’installation et affectation des licences](how-to-connect-post-installation.md).

Pour en savoir plus sur ces fonctionnalités activées lors de l’installation, consultez les pages : [Mise à niveau automatique](how-to-connect-install-automatic-upgrade.md), [Prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) et [Azure AD Connect Health](how-to-connect-health-sync.md).

Pour en savoir plus sur ces sujets courants, consultez l’article [Planificateur Azure AD Connect Sync](how-to-connect-sync-feature-scheduler.md).

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).