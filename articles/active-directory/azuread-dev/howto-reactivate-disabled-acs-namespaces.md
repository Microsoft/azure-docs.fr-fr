---
title: Réactiver des espaces de noms Azure Access Control Service (ACS) désactivés
description: Recherchez et activez vos espaces de noms Azure Access Control Service (ACS) et demandez une extension de leur activation jusqu’au 4 février 2019.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: how-to
ms.date: 01/21/2019
ms.author: ryanwi
ms.reviewer: jlu
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 7bb572e9e22519491290e54417f5ca350d6c0b5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85383715"
---
# <a name="how-to-reactivate-disabled-access-control-service-namespaces"></a>Procédure : Réactiver des espaces de noms Access Control Service désactivés

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

En novembre 2017, nous avons annoncé la mise hors service de Microsoft Azure Access Control Service (ACS), un service d’Azure Active Directory (Azure AD), le 7 novembre 2018.

Depuis, nous avons envoyé plusieurs e-mails aux administrateurs des abonnements ACS sur cette mise hors service : 12 mois, 9 mois, 6 mois, 3 mois, 1 mois, 2 semaines, 1 semaine et 1 jour avant la date de mise hors service du 7 novembre 2018.

Le 3 octobre 2018, nous avons annoncé (par e-mail et dans [un billet de blog](https://azure.microsoft.com/blog/one-month-retirement-notice-access-control-service/)) une offre d’extension pour les clients n'étant pas en mesure de terminer leur migration avant le 7 novembre 2018. Cette annonce contenait également les instructions relatives à la demande d’extension.

## <a name="why-your-namespace-is-disabled"></a>Pourquoi votre espace de noms est-il désactivé ?

Si vous n’avez pas opté pour l’extension, nous avons commencé à désactiver les espaces de noms ACS à partir du 7 novembre 2018. Vous devez avoir déjà demandé l’extension au 4 février 2019 ; sinon, vous ne serez pas en mesure d’activer les espaces de noms via PowerShell.

> [!NOTE]
> Vous devez être administrateur de service ou coadministrateur de l’abonnement pour exécuter les commandes PowerShell et demander une extension.

## <a name="find-and-enable-your-acs-namespaces"></a>Rechercher et activer vos espaces de noms ACS

Vous pouvez utiliser ACS PowerShell pour répertorier tous vos espaces de noms ACS et réactiver ceux qui ont été désactivés.

1. Télécharger et installer ACS PowerShell :
    1. Accédez à PowerShell Gallery et téléchargez [Acs.Namespaces](https://www.powershellgallery.com/packages/Acs.Namespaces/1.0.2).
    1. Installer le module :

        ```powershell
        Install-Module -Name Acs.Namespaces
        ```

    1. Obtenez la liste de toutes les commandes possibles :

        ```powershell
        Get-Command -Module Acs.Namespaces
        ```

        Pour obtenir de l’aide sur une commande spécifique, exécutez la commande suivante :

        ```powershell
        Get-Help [Command-Name] -Full
        ```
    
        où `[Command-Name]` est le nom de la commande ACS.
1. Connectez-vous à ACS à l’aide de l’applet de commande **Connect-AcsAccount**. 

    Vous devrez peut-être modifier votre stratégie d’exécution en exécutant **Set-ExecutionPolicy** avant de pouvoir exécuter la commande.
1. Obtenez la liste de vos abonnements Azure disponibles à l’aide de l’applet de commande **Get-AcsSubscription**.
1. Obtenez la liste de vos espaces de noms ACS en utilisant l’applet de commande **Get-AcsNamespace**.
1. Assurez-vous que les espaces de noms sont désactivés en vérifiant que `State` est `Disabled`.

    [![Vérifier que les espaces de noms sont désactivés](./media/howto-reactivate-disabled-acs-namespaces/confirm-disabled-namespace.png)](./media/howto-reactivate-disabled-acs-namespaces/confirm-disabled-namespace.png#lightbox)

    Vous pouvez également utiliser `nslookup {your-namespace}.accesscontrol.windows.net` pour vérifier si le domaine est toujours actif.

1. Activez la liste de vos espaces de noms ACS en utilisant l’applet de commande **Enable-AcsNamespace**.

    Après avoir activé vos espaces de noms, vous pouvez demander une extension afin de veiller à ce qu'ils ne soient pas une nouvelle fois désactivés avant le 4 février 2019. Passée cette date, toutes les demandes vers ACS échoueront.

## <a name="request-an-extension"></a>Demander une extension

Nous acceptons les nouvelles demandes d’extension à partir du 21 janvier 2019.

Nous commencerons à désactiver les espaces de noms pour les clients qui ont demandé des extensions le 4 février 2019. Vous pouvez néanmoins réactiver des espaces de noms via PowerShell, mais ils seront à nouveau désactivés après 48 heures.

Après le 4 mars 2019, les clients ne pourront plus réactiver des espaces de noms via PowerShell.

Les extensions supplémentaires ne seront plus automatiquement approuvées. Si vous avez besoin de plus de temps pour la migration, contactez le [support Azure](https://portal.azure.com/#create/Microsoft.Support) pour fournir une chronologie de migration détaillée.

### <a name="to-request-an-extension"></a>Pour demander une extension

1. Connectez-vous au portail Microsoft Azure et créez une [demande de support](https://portal.azure.com/#create/Microsoft.Support).
1. Renseignez le formulaire de demande de support comme indiqué dans l’exemple suivant.

    | Champ de la demande de support | Valeur |
    |-----------------------|--------------------|
    | **Type de problème** | `Technical` |
    | **Abonnement** | Défini sur votre abonnement |
    | **Service** | `All services` |
    | **Ressource** | `General question/Resource not available` |
    | **Type de problème** | `ACS to SAS Migration` |
    | **Subject** | Décrivez le problème |

   ![Affiche un exemple de nouvelle demande de support technique](./media/howto-reactivate-disabled-acs-namespaces/new-technical-support-request.png)

<!--

1. Navigate to your ACS namespace's management portal by going to `https://{your-namespace}.accesscontrol.windows.net`.
1. Select the **Read Terms** button to read the [updated Terms of Use](https://azure.microsoft.com/support/legal/access-control/), which will direct you to a page with the updated Terms of Use.

    [![Select the Read Terms button](./media/howto-reactivate-disabled-acs-namespaces/read-terms-button-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/read-terms-button-expanded.png#lightbox)

1. Select **Request Extension** on the banner at the top of the page. The button will only be enabled after you read the [updated Terms of Use](https://azure.microsoft.com/support/legal/access-control/).

    [![Select the Request Extension button](./media/howto-reactivate-disabled-acs-namespaces/request-extension-button-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/request-extension-button-expanded.png#lightbox)

1. After the extension request is registered, the page will refresh with a new banner at the top of the page.

    [![Updated page with refreshed banner](./media/howto-reactivate-disabled-acs-namespaces/updated-banner-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/updated-banner-expanded.png#lightbox)
-->

## <a name="help-and-support"></a>Aide et support

- Si vous rencontrez des problèmes après avoir suivi cette procédure, contactez le [support Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).
- Si vous avez des questions ou des commentaires sur la mise hors service d’ACS, contactez-nous à l’adresse acsfeedback@microsoft.com.

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les informations sur le retrait d’ACS dans [Guide pratique pour migrer depuis Azure Access Control Service](active-directory-acs-migration.md).
