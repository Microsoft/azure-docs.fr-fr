---
title: Résoudre les problèmes d’inscription combinée - Azure Active Directory
description: Résoudre les erreurs d’authentification multifacteur et d’inscription combinée à la réinitialisation de mot de passe en libre-service Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 04/15/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5ba1e280e3c14a24e33246799ca0d7ef8221294f
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741692"
---
# <a name="troubleshooting-combined-security-information-registration"></a>Résoudre les problèmes d’inscription des informations de sécurité combinée

Les informations contenues dans cet article sont destinées à guider les administrateurs qui cherchent à résoudre les problèmes signalés par des utilisateurs de l’expérience d’inscription combinée.

## <a name="audit-logs"></a>Journaux d’audit

Les événements consignés pour l’inscription combinée se trouvent dans la catégorie Méthodes d’authentification dans les journaux d’audit Azure AD.

![L’interface des journaux d’audit Azure AD affichant les événements d’inscription](media/howto-registration-mfa-sspr-combined-troubleshoot/combined-security-info-audit-log.png)

Le tableau suivant répertorie tous les événements d’audit générés par l’inscription combinée :

| Activité | Statut | Motif | Description |
| --- | --- | --- | --- |
| L’utilisateur a inscrit toutes les informations de sécurité requises | Succès | Utilisateur a inscrit toutes les informations de sécurité requises. | Cet événement se produit lorsqu’un utilisateur a terminé l’inscription.|
| L’utilisateur a inscrit toutes les informations de sécurité requises | Échec | L’utilisateur a annulé l’inscription des informations de sécurité. | Cet événement se produit lorsqu’un utilisateur annule l’enregistrement à partir du mode d’interruption.|
| Informations de sécurité d’utilisateur inscrit | Succès | L’utilisateur a inscrit une *méthode*. | Cet événement se produit lorsqu’un utilisateur inscrit une méthode individuelle. La *méthode* peut être une application Authenticator, un téléphone, un e-mail, des questions de sécurité, un mot de passe d’application, un autre numéro de téléphone, etc.| 
| L’utilisateur a examiné ses informations de sécurité | Succès | L’utilisateur a correctement examiné ses informations de sécurité. | Cet événement se produit lorsqu’un utilisateur sélectionne **Les informations semblent correctes**  sur la page d’examen des informations de sécurité.|
| L’utilisateur a examiné ses informations de sécurité | Échec | Échec de l’examen des informations de sécurité de l’utilisateur. | Cet événement se produit lorsqu’un utilisateur sélectionne **Les informations semblent correctes** sur la page d’examen des informations de sécurité, mais qu’une erreur survient sur le serveur principal.|
| L’utilisateur a supprimé ses informations de sécurité | Succès | L’utilisateur a supprimé une *méthode*. | Cet événement se produit lorsqu’un utilisateur supprime une méthode individuelle. La *méthode* peut être une application Authenticator, un téléphone, un e-mail, des questions de sécurité, un mot de passe d’application, un autre numéro de téléphone, etc.|
| L’utilisateur a supprimé ses informations de sécurité | Échec | Échec de la suppression de la *méthode* par l’utilisateur. | Cet événement se produit lorsqu’un utilisateur tente de supprimer une méthode, mais que sa tentative échoue. La *méthode* peut être une application Authenticator, un téléphone, un e-mail, des questions de sécurité, un mot de passe d’application, un autre numéro de téléphone, etc.|
| L’utilisateur a modifié ses informations de sécurité par défaut | Succès | Utilisateur a modifié sa *méthode* d’informations de sécurité par défaut. | Cet événement se produit lorsqu’un utilisateur modifie sa méthode d’identification par défaut. Sa *méthode* peut consister en une notification de l’application Authenticator, un code ou un jeton provenant de mon application d’authentification, un appel au + X XXXXXXXXXX, un SMS au + X XXXXXXXXX, etc.|
| L’utilisateur a modifié ses informations de sécurité par défaut | Échec | Échec de la modification de la *méthode* des informations de sécurité par défaut de l’utilisateur. | Cet événement se produit lorsqu’un utilisateur tente de modifier sa méthode d’authentification par défaut, mais que sa tentative échoue. Sa *méthode* peut consister en une notification de l’application Authenticator, un code ou un jeton provenant de mon application d’authentification, un appel au + X XXXXXXXXXX, un SMS au + X XXXXXXXXX, etc.|

## <a name="troubleshooting-interrupt-mode"></a>Résolution des problèmes de mode d’interruption

| Symptôme | Étapes de dépannage |
| --- | --- |
| Les méthodes que je m’attendais à voir ne s’affichent pas. | 1. Veuillez vérifier si l’utilisateur possède un rôle d’administrateur Azure AD. Si oui, veuillez afficher les différences de stratégie d’administrateur SSPR. <br> 2. Déterminez si l’utilisateur est interrompu en raison de la mise en œuvre de l’inscription de l’authentification multifacteur ou de la mise en œuvre de l’inscription SSPR. Consultez l’[organigramme](../../active-directory/authentication/concept-registration-mfa-sspr-combined.md#combined-registration-modes) sous « Modes d’inscription combinée » pour déterminer quelles méthodes doivent être indiquées. <br> 3. Déterminer comment la stratégie d’authentification multifacteur ou SSPR a été modifiée. Si la modification est récente, la propagation de la stratégie mise à jour peut prendre un peu de temps.|

## <a name="troubleshooting-manage-mode"></a>Résolution des problèmes du mode de gestion

| Symptôme | Étapes de dépannage |
| --- | --- |
| Je n’ai pas la possibilité d’ajouter une méthode particulière. | 1. Déterminez si la méthode est activée pour l’authentification multifacteur ou pour le SSPR. <br> 2. Si la méthode est activée, veuillez réenregistrer les stratégies, puis patientez 1 à 2 heures avant de les tester de nouveau. <br> 3. Si la méthode est activée, assurez-vous que l’utilisateur n’a pas déjà configuré le nombre maximum de cette méthode qu’il possible de configurer.|

## <a name="disable-combined-registration"></a>Activer une inscription combinée

Quand un utilisateur inscrit un numéro de téléphone ou une application mobile dans la nouvelle expérience combinée, notre service estampille un jeu d’indicateurs (StrongAuthenticationMethods) pour ces méthodes sur cet utilisateur. Cette fonctionnalité permet à l’utilisateur effectuer une authentification Microsoft Azure Multi-Factor Authentication (MFA) avec ces méthodes, chaque fois qu’une telle authentification multifacteur est requise.

Si un administrateur active la préversion, les utilisateurs s’inscrivent via la nouvelle expérience, puis quand l’administrateur désactive la préversion, les utilisateurs peuvent également être inscrits pour l’authentification multifacteur à leur insu.

Si un utilisateur qui a mené à bien l’enregistrement combiné accède à la page d’inscription à la réinitialisation du mot de passe libre-service (SSPR) sur [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup), il sera invité à effectuer une authentification multifacteur avant de pouvoir accéder à cette page. Cette étape est normale d'un point de vue technique, mais elle est inhabituelle pour les utilisateurs qui étaient auparavant uniquement inscrits pour la SSPR. Bien que cette étape supplémentaire améliore la situation de sécurité de l’utilisateur en fournissant un niveau supplémentaire de sécurité, il arrive que les administrateurs souhaitent restaurer leurs utilisateurs, de façon à ce qu’ils ne puissent plus effectuer d’authentification multifacteur.  

### <a name="how-to-roll-back-users"></a>Comment réinitialiser des utilisateurs

Si, en tant qu’administrateur, vous voulez réinitialiser les paramètres de l’authentification multifacteur d’un utilisateur, vous pouvez utiliser le script PowerShell fourni dans la section suivante. Le script efface la propriété StrongAuthenticationMethods pour l’application mobile et/ou le numéro de téléphone d’un utilisateur. Si vous exécutez ce script pour vos utilisateurs, ils devront se réinscrire pour l’authentification multifacteur s’ils en ont besoin. Nous vous recommandons de tester la réinitialisation avec un ou deux utilisateurs avant de réinitialiser tous les utilisateurs affectés.

La procédure suivante vous guidera lors de la restauration d’un utilisateur ou d’un groupe d’utilisateurs.

#### <a name="prerequisites"></a>Prérequis

1. Installez les modules Azure AD PowerShell appropriés. Dans une fenêtre de PowerShell, exécutez les commandes suivantes pour installer les modules :

   ```powershell
   Install-Module -Name MSOnline
   Import-Module MSOnline
   ```

1. Enregistrez la liste des ID d’objet de l’utilisateur affecté sur votre ordinateur en tant que fichier texte contenant un ID par ligne. Prenez note de l’emplacement du fichier.
1. Enregistrez le script suivant sur votre ordinateur, et notez l’emplacement du script :

   ```powershell
   <# 
   //********************************************************
   //*                                                      *
   //*   Copyright (C) Microsoft. All rights reserved.      *
   //*                                                      *
   //********************************************************
   #>

   param($path)

   # Define Remediation Fn
   function RemediateUser {

       param  
       (
           $ObjectId
       )

       $user = Get-MsolUser -ObjectId $ObjectId

       Write-Host "Checking if user is eligible for rollback: UPN: "  $user.UserPrincipalName  " ObjectId: "  $user.ObjectId -ForegroundColor Yellow

       $hasMfaRelyingParty = $false
       foreach($p in $user.StrongAuthenticationRequirements)
       {
           if ($p.RelyingParty -eq "*")
           {
               $hasMfaRelyingParty = $true
               Write-Host "User was enabled for per-user MFA." -ForegroundColor Yellow
           }
       }

       if ($user.StrongAuthenticationMethods.Count -gt 0 -and -not $hasMfaRelyingParty)
       {
           Write-Host $user.UserPrincipalName " is eligible for rollback" -ForegroundColor Yellow
           Write-Host "Rolling back user ..." -ForegroundColor Yellow
           Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName $user.UserPrincipalName
           Write-Host "Successfully rolled back user " $user.UserPrincipalName -ForegroundColor Green
       }
       else
       {
           Write-Host $user.UserPrincipalName " is not eligible for rollback. No action required."
       }

       Write-Host ""
       Start-Sleep -Milliseconds 750
   }

   # Connect
   Import-Module MSOnline
   Connect-MsolService

   foreach($line in Get-Content $path)
   {
       RemediateUser -ObjectId $line
   }
   ```

#### <a name="rollback"></a>Restauration

Dans une fenêtre PowerShell, exécutez la commande suivante en fournissant les emplacements du script et du fichier utilisateur. Entrez les informations d’identification de l’administrateur général quand vous y êtes invité. Le script génère le résultat de chaque opération de mise à jour d’utilisateur.

`<script location> -path <user file location>`

### <a name="disable-the-updated-experience"></a>Désactiver l’expérience mise à jour

Pour désactiver l’expérience mise à jour pour vos utilisateurs, procédez comme suit :

1. Connectez-vous au portail Microsoft Azure en tant qu’administrateur d’utilisateurs.
2. Accédez à **Azure Active Directory** > **Paramètres utilisateur** > **Gérer les paramètres des fonctionnalités préliminaires du volet d’accès**.
3. Sous **Users can use preview features for registering and managing security info** (les utilisateurs peuvent utiliser les fonctionnalités en préversion pour inscrire et gérer les informations de sécurité), positionnez le sélecteur sur **None** (Aucune), puis sélectionnez **Enregistrer**.

Les utilisateurs ne seront plus invités à s’inscrire à l’aide de l’expérience mise à jour.

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur l'inscription combinée de la réinitialisation de mot de passe en libre-service (SSPR) et d'Azure AD Multi-Factor Authentication](concept-registration-mfa-sspr-combined.md)
