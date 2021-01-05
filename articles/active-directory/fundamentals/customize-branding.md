---
title: Personnaliser la page de connexion de votre organisation - Azure AD
description: Instructions pour personnaliser la page de connexion Azure Active Directory avec les informations de votre organisation.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 06/24/2020
ms.author: ajburnle
ms.reviewer: kexia
ms.custom: it-pro, seodec18, fasttrack-edit
ms.collection: M365-identity-device-management
ms.openlocfilehash: bb138ade0f579a7545c0910646b6adfb7d5ac02a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996659"
---
# <a name="add-branding-to-your-organizations-azure-active-directory-sign-in-page"></a>Personnaliser la page de connexion Azure Active Directory de votre organisation
Utilisez le logo et la palette de couleurs personnalisée de votre organisation pour offrir une apparence cohérente à vos pages de connexion Azure Active Directory (Azure AD). Vos pages de connexion s’affichent quand les utilisateurs se connectent aux applications web de votre organisation, comme Microsoft 365, qui utilisent Azure AD comme fournisseur d’identité.

>[!NOTE]
>La personnalisation nécessite l’utilisation des éditions Azure Active Directory Premium 1 ou Premium 2. Pour plus d’informations sur les licences et les éditions, consultez [S’inscrire à Azure AD Premium](active-directory-get-started-premium.md).<br><br>Les clients vivant en Chine peuvent accéder aux éditions Premium d’Azure Active Directory à l’aide de l’instance mondiale d’Azure Active Directory. Actuellement, les éditions Premium d’Azure AD ne sont pas prises en charge dans le service Azure géré par 21Vianet en Chine. Pour plus d’informations, contactez-nous sur le [forum Azure Active Directory](https://feedback.azure.com/forums/169401-azure-active-directory/).

## <a name="customize-your-azure-ad-sign-in-page"></a>Personnaliser votre page de connexion Azure AD
Vous pouvez personnaliser les pages de connexion Azure AD qui s’affichent quand les utilisateurs se connectent à des applications propres aux locataires de votre organisation, comme `https://outlook.com/contoso.com`, ou lors de l’envoi d’une variable de domaine, comme `https://passwordreset.microsoftonline.com/?whr=contoso.com`.

Votre personnalisation ne s’affiche pas immédiatement quand vos utilisateurs accèdent à des sites tels que www\.office.com. L’utilisateur doit se connecter avant que votre personnalisation n’apparaisse. Une fois que l’utilisateur est connecté, la personnalisation peut prendre une quinzaine de minutes pour s’afficher. 

> [!NOTE]
> Tous les éléments de personnalisation sont facultatifs. Par exemple, si vous spécifiez un logo de bannière sans image d’arrière-plan, la page de connexion affiche votre logo avec l’image d’arrière-plan par défaut du site de destination (par exemple Microsoft 365).<br><br>De plus, la personnalisation de la page de connexion ne s’étend pas aux comptes Microsoft personnels. Si les utilisateurs ou des invités professionnels se connectent avec un compte Microsoft personnel, leur page de connexion ne reflète pas la personnalisation de votre organisation.

### <a name="to-customize-your-branding"></a>Pour personnaliser votre marque
1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez **Azure Active Directory**, puis sélectionnez **Marque de société** et **Configurer**.

    ![Contoso - Page Marque de société, avec option Configurer mise en surbrillance](media/customize-branding/company-branding-configure-button.png)

3. Dans la page **Configurer la marque de société**, fournissez tout ou partie des informations suivantes.

    >[!IMPORTANT]
    >Toutes les images personnalisées que vous ajoutez sur cette page présentent des restrictions en termes de taille d’image (pixels), et éventuellement de taille de fichier (Ko). En raison de ces restrictions, vous devrez probablement utiliser un éditeur de photos pour créer des images à la bonne taille.

    - **Paramètres généraux :**

        ![Page Configurer la marque de société, avec paramètres généraux complétés](media/customize-branding/configure-company-branding-general-settings.png)

        - **Langue.** La langue est définie automatiquement par défaut et ne peut pas être modifiée.
        
        - **Image d’arrière-plan de la page de connexion.** Sélectionnez un fichier image .png ou .jpg qui apparaîtra sous la forme d’un arrière-plan sur vos pages de connexion. L’image est ancrée au centre du navigateur et s’adapte à la taille de l’espace affichable. Vous ne pouvez pas sélectionner une image dont la taille est supérieure à 1920 x 1080 pixels ou dont la taille de fichier est supérieure à 300 Ko.
        
            Il est recommandé d’utiliser des images qui n’attireront pas l’attention. Par exemple, si une zone blanche opaque apparaît au centre de l’écran, elle peut couvrir n’importe quelle partie de l’image en fonction des dimensions de l’espace affichable.

        - **Logo de bannière.** Sélectionnez une version .png ou .jpg de votre logo, qui apparaît sur la page de connexion une fois que l’utilisateur a entré un nom d’utilisateur et sur la page du portail **Mes applications**.
            
            L’image ne peut pas faire plus de 60 pixels de haut ou 280 pixels de large. Nous vous recommandons d’utiliser une image transparente dans la mesure où l’arrière-plan peut ne pas correspondre à l’arrière-plan de votre logo. Nous vous recommandons également de ne pas ajouter de marge intérieure autour de l’image afin que votre logo ne semble pas trop petit.

        - **Indication sur le nom d’utilisateur.** Entrez le texte d’indication qui s’affiche pour les utilisateurs ayant oublié leur nom d’utilisateur. Ce texte doit être au format Unicode, ne comporter aucun lien ni code, et ne pas dépasser 64 caractères. Si des invités se connectent à votre application, nous vous suggérons de ne pas ajouter cet indicateur.

        - **Texte de la page de connexion et mise en forme.** Entrez le texte qui apparaît au bas de la page de connexion. Vous pouvez utiliser ce texte pour communiquer des informations supplémentaires telles que le numéro de téléphone à votre support technique ou une mention légale. Ce texte doit être au format Unicode et ne doit pas dépasser 1 024 caractères.

           Vous pouvez personnaliser le texte de la page de connexion que vous avez entré. Pour commencer un nouveau paragraphe, utilisez deux fois la touche Entrée. Vous pouvez également modifier la mise en forme du texte pour y inclure le gras, l’italique, le soulignement ou un lien hypertexte. Pour ajouter une mise en forme au texte, utilisez la syntaxe suivante : 

          > Lien hypertexte : ```[text](link)``` 
          
          > Gras : ``` **text** ``` ou ``` __text__ ``` 
          
          > Italique : ``` *text* ``` ou ``` _text_ ``` 
          
          > Souligné : ``` ++text++ ``` 

    - **Paramètres avancés**
            
        ![Page Configurer la marque de société, avec paramètres avancés renseignés](media/customize-branding/configure-company-branding-advanced-settings.png)   

        - **Couleur d’arrière-plan de la page de connexion.** Spécifiez la couleur hexadécimale (par exemple, #FFFFFF pour blanc) qui s’affiche à la place de votre image d’arrière-plan en cas de faible bande passante. Nous vous recommandons d’utiliser la couleur principale de votre logo de bannière ou la couleur de votre organisation.

        - **Logo carré.** Sélectionnez une image .png (recommandé) ou .jpg du logo de votre organisation pour qu’elle soit présentée aux utilisateurs pendant le processus d’installation de nouveaux appareils Windows 10 Entreprise. Cette image est utilisée uniquement pour l’authentification Windows et s’affiche uniquement sur les locataires qui utilisent [Windows Autopilot]( /windows/deployment/windows-autopilot/windows-10-autopilot) pour les pages de déploiement ou de saisie de mot de passe dans d’autres expériences Windows 10. Dans certains cas, elle peut également apparaître dans la boîte de dialogue de consentement.
        
            La taille de l’image ne peut pas dépasser 240 x 240 pixels et doit être inférieure à 10 Ko. Nous vous recommandons d’utiliser une image transparente dans la mesure où l’arrière-plan peut ne pas correspondre à l’arrière-plan de votre logo. Nous vous recommandons également de ne pas ajouter de marge intérieure autour de l’image afin que votre logo ne semble pas trop petit.
    
        - **Logo carré, thème foncé.** Identique à l’image de logo carré ci-dessus. Cette image de logo prend la place de l’image de logo carré dans le cas d’un arrière-plan foncé, comme avec les écrans « Windows 10 Azure AD Joined » de l’expérience OOBE (out-of-box experience).  Si votre logo ressort bien sur des arrière-plans blancs, bleu foncé et noirs, vous n’avez pas besoin d’ajouter cette image. 
        
        - **Afficher l’option permettant de rester connecté.** Vous pouvez autoriser les utilisateurs à rester connecté à Azure AD jusqu’à ce qu’ils se déconnectent de manière explicite. Si vous sélectionnez **Non**, cette option est masquée et les utilisateurs doivent se connecter chaque fois que le navigateur est fermé puis ouvert.

            Cette fonctionnalité est disponible uniquement sur l’objet de personnalisation par défaut, non sur un objet spécifique d’une langue. Pour en savoir plus sur la configuration et la résolution des problèmes de l’option permettant de rester connecté, consultez [Configurer l’invite « Rester connecté ? » pour les comptes Azure AD](keep-me-signed-in.md)
        
            >[!NOTE]
            >Certaines fonctionnalités de SharePoint Online et Office 2010 dépendent du choix des utilisateurs de rester connecté. Si vous définissez cette option sur **Non**, il se peut que vos utilisateurs voient des invites de connexion supplémentaires et inattendues.
   

3. Une fois que vous avez terminé votre personnalisation, sélectionnez **Enregistrer**.

    Si ce processus crée votre première configuration de personnalisation, il devient le processus par défaut pour votre locataire. Si vous avez d’autres configurations, vous pouvez choisir votre configuration par défaut.
    
    >[!IMPORTANT]
    >Si vous souhaitez ajouter d’autres configurations de personnalisation d’entreprise pour votre locataire, vous devez sélectionnez **Nouvelle langue** dans la page **Contoso - Marque de société**. Cette opération ouvre la page **Configurer la marque de société**, où vous pouvez suivre les mêmes étapes que ci-dessus.

## <a name="update-your-custom-branding"></a>Mettre à jour votre personnalisation
Une fois que vous avez créé votre personnalisation, vous pouvez revenir en arrière et la modifier à votre gré.

### <a name="to-edit-your-custom-branding"></a>Modifier votre personnalisation
1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez **Azure Active Directory**, puis sélectionnez **Marque de société** et **Configurer**.

    ![Page Contoso - Marque de société, avec configuration par défaut](media/customize-branding/company-branding-default-config.png)

3. Dans la page **Configurer la marque de société**, ajoutez, supprimez ou modifiez les informations en fonction des descriptions disponibles dans la section [Personnaliser votre page de connexion Azure AD](#customize-your-azure-ad-sign-in-page) de cet article.

4. Sélectionnez **Enregistrer**.

   Il peut s’écouler jusqu’à une heure avant que les modifications que vous avez apportées à la page de connexion soient visibles.

## <a name="add-language-specific-company-branding-to-your-directory"></a>Ajouter une marque de société spécifique à une langue à votre répertoire
Vous ne pouvez pas changer la langue par défaut de votre configuration d’origine. Toutefois, si vous avez besoin d’une configuration dans une autre langue, vous pouvez créer une autre configuration.

### <a name="to-add-a-language-specific-branding-configuration"></a>Pour ajouter une configuration de personnalisation spécifique à une langue

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez **Azure Active Directory**, puis **Marque de société** et **Nouvelle langue**.

    ![Page Contoso - Marque de société, avec option Nouvelle langue mise en surbrillance](media/customize-branding/company-branding-new-language.png)

3. Dans la page **Configurer la marque de société**, sélectionnez votre langue (par exemple, français), puis ajoutez vos informations traduites en fonction des descriptions disponibles dans la section [Personnaliser votre page de connexion Azure AD](#customize-your-azure-ad-sign-in-page) de cet article.

4. Sélectionnez **Enregistrer**.

    La page **Contoso - Marque de société** est mise à jour pour afficher la nouvelle configuration en français.

    ![Page de personnalisation de la société Contoso, avec la nouvelle configuration de langue affichée](media/customize-branding/company-branding-french-config.png)

## <a name="add-your-custom-branding-to-pages"></a>Ajouter votre personnalisation aux pages
Pour ajouter votre personnalisation aux pages, modifiez la fin de l’URL avec le texte `?whr=yourdomainname`. Cette modification fonctionne sur plusieurs pages, notamment la page de configuration de l’authentification multifacteur (MFA), la page de configuration de réinitialisation de mot de passe en libre-service (SSPR) et la page de connexion.

**Exemples :**

**URL d’origine :** https://aka.ms/MFASetup<br>
**URL personnalisée :** `https://account.activedirectory.windowsazure.com/proofup.aspx?whr=contoso.com`

**URL d’origine :** https://aka.ms/SSPR<br>
**URL personnalisée :** `https://passwordreset.microsoftonline.com/?whr=contoso.com`
