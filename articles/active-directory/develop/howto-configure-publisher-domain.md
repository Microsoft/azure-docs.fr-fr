---
title: Configurer un domaine d’éditeur d’application | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer un domaine d’éditeur d’application permettant aux utilisateurs de savoir où leurs informations sont envoyées.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/23/2020
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, zachowd
ms.custom: aaddev
ms.openlocfilehash: d62328534157a1e2d9e53830be9d6b87e08c9e40
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87129268"
---
# <a name="how-to-configure-an-applications-publisher-domain"></a>Procédure : Configurer un domaine d’éditeur d’application

Un domaine d’éditeur d’application est présenté aux utilisateurs à l’[invite de consentement de l’application](application-consent-experience.md) et leur permet de savoir où leurs informations sont envoyées. Les applications multilocataires inscrites après le 21 mai 2019 n’ont pas de domaine d’éditeur qui s’affiche comme **non vérifié**. Les applications multilocataires sont des applications qui prennent en charge des comptes en dehors d’un annuaire d’organisation unique, et qui prennent en charge, par exemple, tous les comptes Azure AD ou tous les comptes Azure AD et comptes Microsoft personnels.

## <a name="new-applications"></a>Nouvelles applications

Lorsque vous inscrivez une nouvelle application, le domaine d’éditeur de votre application peut être défini sur une valeur par défaut. La valeur dépend de l’emplacement où l’application est inscrite, en particulier si l’application est inscrite dans un locataire et si le locataire contient des domaines vérifiés par le locataire.

S’il existe des domaines vérifiés par le locataire, le domaine d’éditeur de l’application est défini par défaut sur le domaine vérifié principal du locataire. S’il existe aucun domaine vérifié par le locataire (ce qui est le cas lorsque l’application n’est pas inscrite dans un locataire), le domaine d’éditeur de l’application est défini sur null.

Le tableau suivant récapitule le comportement par défaut de la valeur de domaine d’éditeur.  

| Domaines vérifiés par le locataire | Valeur par défaut du domaine d’éditeur |
|-------------------------|----------------------------|
| null | null |
| *.onmicrosoft.com | *.onmicrosoft.com |
| - *.onmicrosoft.com<br/>- domain1.com<br/>- domain2.com (primary) | domain2.com |

Si le domaine d’éditeur d’une application multilocataires n’est pas défini, ou s’il est défini sur un domaine qui se termine par . onmicrosoft.com, l’invite de consentement de l’application affiche **non vérifié** au lieu du domaine d’éditeur.

## <a name="grandfathered-applications"></a>Applications héritées

Si votre application a été inscrite avant le 21 mai 2019, l’invite de consentement de votre application n’affichera pas **non vérifié** si vous n’avez pas défini de domaine d’éditeur. Nous vous recommandons de définir la valeur de domaine d’éditeur pour que les utilisateurs puissent voir ces informations sur l’invite de consentement de votre application.

## <a name="configure-publisher-domain-using-the-azure-portal"></a>Configurer le domaine d’éditeur à l’aide du Portail Azure

Pour définir le domaine d’éditeur de votre application, procédez comme suit.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.

1. Si votre compte est présent dans plusieurs locataires Azure AD :
   1. Sélectionnez votre profil dans le menu en haut à droite de la page, puis **changez de répertoire**.
   1. Modifiez votre session sur le locataire Azure AD où vous voulez créer votre application.

1. Accédez à [Azure Active Directory > Inscriptions d’applications](https://go.microsoft.com/fwlink/?linkid=2083908) pour rechercher et sélectionner l’application que vous souhaitez configurer.

   Une fois l’application sélectionnée, vous pourrez voir sa page **Vue d’ensemble**.

1. Dans la page **Vue d’ensemble** de l’application, sélectionner la section **Personnalisation**.

1. Recherchez le champ **Domaine d’éditeur** et sélectionnez une des options suivantes :

   - Sélectionnez **Configurer un domaine** si vous n’avez pas déjà configuré de domaine.
   - Sélectionnez **Mettre à jour le domaine** si un domaine a déjà été configuré.

Si votre application est inscrite dans un locataire, vous verrez deux onglets dans lesquels sélectionner : **Sélectionner un domaine vérifié** et **Vérifier un nouveau domaine**.

Si votre application n’est pas inscrite dans un locataire, vous ne verrez que l’option de vérification d’un nouveau domaine pour votre application.

### <a name="to-verify-a-new-domain-for-your-app"></a>Pour vérifier un nouveau domaine pour votre application

1. Créez un fichier nommé `microsoft-identity-association.json` et collez l’extrait de code JSON suivant.

   ```json
   {
      "associatedApplications": [
         {
            "applicationId": "{YOUR-APP-ID-HERE}"
         },
         {
            "applicationId": "{YOUR-OTHER-APP-ID-HERE}"
         }
      ]
    }
   ```

1. Remplacez l’espace réservé *{YOUR-APP-ID-HERE}* par l’ID d’application (client) qui correspond à votre application.

1. Hébergez le fichier dans : `https://{YOUR-DOMAIN-HERE}.com/.well-known/microsoft-identity-association.json`. Remplacez l’espace réservé *{YOUR-DOMAIN-HERE}* pour qu’il corresponde au domaine vérifié.

1. Cliquez sur le bouton **Vérifier et enregistrer le domaine**.

### <a name="to-select-a-verified-domain"></a>Pour sélectionner un domaine vérifié

- Si votre client contient des domaines vérifiés, sélectionnez un des domaines dans la liste déroulante **Sélectionner un domaine vérifié**.

>[!Note]
> L’en-tête « Content-type » attendu qui doit être retourné est `application/json`. Vous pouvez recevoir une erreur comme indiqué ci-dessous si vous utilisez autre chose, tel que `application/json; charset=utf-8`. 
> 
>``` "Verification of publisher domain failed. Error getting JSON file from https:///.well-known/microsoft-identity-association. The server returned an unexpected content type header value. " ```
>

## <a name="implications-on-the-app-consent-prompt"></a>Conséquences sur l’invite de consentement d’application

La configuration du domaine d’éditeur a un impact sur ce que les utilisateurs voient à l’invite de consentement de l’application. Pour bien comprendre les composants de l’invite de consentement, consultez [Comprendre les expériences du consentement d’application](application-consent-experience.md).

Le tableau suivant décrit le comportement pour les applications créées avant le 21 mai 2019.

![Invite de consentement pour les applications créées avant le 21 mai 2019](./media/howto-configure-publisher-domain/old-app-behavior-table.png)

Le comportement pour les nouvelles applications créées après le 21 mai 2019 varie en fonction du domaine d’éditeur et du type d’application. Le tableau suivant décrit les modifications que vous devriez voir les différentes combinaisons de configurations.

![Invite de consentement pour les applications créées après le 21 mai 2019](./media/howto-configure-publisher-domain/new-app-behavior-table.png)

## <a name="implications-on-redirect-uris"></a>Conséquences sur les URI de redirection

Les applications qui connectent les utilisateurs à un compte professionnel ou scolaire ou des comptes Microsoft personnels ([multilocataire](single-and-multi-tenant-apps.md)) sont soumises à quelques restrictions lors de la spécification des URI de redirection.

### <a name="single-root-domain-restriction"></a>Restriction de domaine racine unique

Lorsque la valeur de domaine d’éditeur pour les applications multilocataires est définie sur null, les applications sont limitées au partage d’un domaine racine unique pour les URI de redirection. Par exemple, la combinaison de valeurs suivante n’est pas autorisée, car le domaine racine, contoso.com, ne correspond pas à fabrikam.com.

```
"https://contoso.com",
"https://fabrikam.com",
```

### <a name="subdomain-restrictions"></a>Restrictions de sous-domaine

Les sous-domaines sont autorisés, mais vous devez inscrire explicitement le domaine racine. Par exemple, bien que les URI suivants partagent un domaine racine unique, la combinaison n’est pas autorisée.

```
"https://app1.contoso.com",
"https://app2.contoso.com",
```

Toutefois, si le développeur ajoute explicitement le domaine racine, la combinaison sera autorisée.

```
"https://contoso.com",
"https://app1.contoso.com",
"https://app2.contoso.com",
```

### <a name="exceptions"></a>Exceptions

Les cas suivants ne sont pas soumis à la restriction de domaine racine unique :

- Applications à locataire unique, ou applications ciblant des comptes dans un répertoire unique
- Utilisation de localhost comme URI de redirection
- URI de redirection avec des schémas personnalisés (non-HTTP ou HTTPS)

## <a name="configure-publisher-domain-programmatically"></a>Configurer un domaine d’éditeur par programmation

Il n’existe actuellement aucune prise en charge de l’API REST ou de PowerShell pour configurer un domaine d’éditeur par programmation.
