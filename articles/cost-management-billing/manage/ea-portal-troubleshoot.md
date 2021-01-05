---
title: Résoudre les problèmes d’accès au portail Azure EA
description: Cet article décrit certains problèmes courants qui peuvent se produire avec un Contrat Entreprise Azure (EA) dans le portail Azure EA.
author: bandersmsft
ms.author: banders
ms.date: 08/20/2020
ms.topic: troubleshooting
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: 5402ee3283e5ae11f43884b4002486e8bdefae2c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294405"
---
# <a name="troubleshoot-azure-ea-portal-access"></a>Résoudre les problèmes d’accès au portail Azure EA

Cet article décrit certains problèmes courants qui peuvent se produire avec un Contrat Entreprise Azure (EA). Le portail Azure EA permet de gérer les utilisateurs et les coûts d’un contrat Entreprise. Vous pouvez rencontrer ces problèmes lors de la configuration ou de la mise à jour de l’accès au portail Azure EA.

## <a name="issues-adding-an-admin-to-an-enrollment"></a>Problèmes lors de l’ajout d’un administrateur à une inscription

Il existe différents types de niveaux d’authentification pour les inscriptions Entreprise. Lorsque les niveaux d’authentification sont appliqués de manière incorrecte, vous pouvez rencontrer des problèmes lorsque vous essayez de vous connecter au portail Azure EA.

Vous utilisez le portail Azure EA pour accorder l’accès aux utilisateurs avec différents niveaux d’authentification. Un administrateur d’entreprise peut mettre à jour le niveau d’authentification pour répondre aux exigences de sécurité de son organisation.

### <a name="authentication-level-types"></a>Types de niveau d’authentification

- Compte Microsoft uniquement - Pour les organisations qui souhaitent utiliser, créer et gérer des utilisateurs via des comptes Microsoft.
- Compte professionnel ou scolaire - Pour les organisations qui ont configuré Active Directory avec la fédération dans le cloud et qui ont tous les comptes sur un seul locataire.
- Compte professionnel ou scolaire inter-locataires - Pour les organisations qui ont configuré Active Directory avec la fédération dans le cloud et qui auront des comptes sur plusieurs locataires.
- Compte mixte - Vous permet d’ajouter des utilisateurs avec un compte Microsoft et/ou avec un compte professionnel ou scolaire.

Le premier compte professionnel ou scolaire ajouté à l’inscription détermine le domaine _par défaut_. Pour ajouter un compte professionnel ou scolaire avec un autre locataire, vous devez modifier le niveau d’authentification sous l’inscription en spécifiant l’authentification inter-locataires.

Pour mettre à jour le niveau d’authentification :

1. Connectez-vous au portail Azure EA en tant qu’administrateur d’entreprise.
2. Cliquez sur **Gérer** dans le volet de navigation de gauche.
3. Cliquez sur l’onglet **Inscription**.
4. Sous **Détails de l’inscription**, sélectionnez **Niveau d’authentification**.
5. Cliquez sur le symbole de crayon.
6. Cliquez sur **Enregistrer**.

![Exemple illustrant les niveaux d’authentification ](./media/ea-portal-troubleshoot/create-ea-authentication-level-types.png)

Les comptes Microsoft doit avoir un ID associé, créé à l’adresse [https://signup.live.com](https://signup.live.com/).

Des comptes professionnels ou scolaires sont disponibles pour les organisations qui ont configuré Active Directory avec la fédération et dont tous les comptes figurent sur un seul locataire. Les utilisateurs peuvent être ajoutés avec l’authentification des utilisateurs fédérés professionnels ou scolaires si le service Active Directory interne de l’entreprise est fédéré.

Si votre organisation n’utilise pas la fédération Active Directory, vous ne pouvez pas utiliser votre adresse e-mail professionnelle ou scolaire. Au lieu de cela, inscrivez ou créez une nouvelle adresse e-mail et inscrivez-la en tant que compte Microsoft.

## <a name="unable-to-access-the-azure-ea-portal"></a>Impossible d’accéder au portail Azure EA

Si vous recevez un message d’erreur lorsque vous essayez de vous connecter au portail Azure EA, suivez les étapes de dépannage suivantes :

- Assurez-vous d’utiliser l’URL correcte du portail Azure EA, à savoir https://ea.azure.com.
- Déterminez si votre accès au portail Azure EA a été ajouté en tant que compte professionnel ou scolaire ou en tant que compte Microsoft.
  - Si vous utilisez votre compte professionnel, entrez votre adresse e-mail professionnelle et votre mot de passe professionnel. Votre mot de passe professionnel vous est fourni par votre organisation. Vous pouvez vérifier auprès de votre service informatique comment réinitialiser ce mot de passe en cas de problème.
  - Si vous utilisez un compte Microsoft, entrez l’adresse e-mail et le mot de passe de votre compte Microsoft. Si vous avez oublié le mot de passe de votre compte Microsoft, vous pouvez le réinitialiser à l’adresse [https://account.live.com/password/reset](https://account.live.com/password/reset).
- Utilisez une session de navigateur InPrivate ou Incognito pour vous connecter afin de ne conserver aucun cookie ni aucune information en cache des sessions précédentes ou existantes. Effacez le cache de votre navigateur et utilisez une fenêtre InPrivate ou Incognito pour ouvrir https://ea.azure.com.
- Si vous recevez une erreur _Utilisateur non valide_ lors de l’utilisation d’un compte Microsoft, cela peut être dû au fait que vous possédez plusieurs comptes Microsoft. Celui avec lequel vous essayez de vous connecter ne correspond pas à l’adresse e-mail principale.
Ou, si vous recevez une erreur _Utilisateur non valide_, cela peut être dû au fait qu’un type de compte incorrect a été utilisé lorsque l’utilisateur a été ajouté à l’inscription. Par exemple, un compte professionnel ou scolaire à la place d’un compte Microsoft. Dans cet exemple, un autre administrateur EA doit ajouter le compte correct ou vous devez contacter le [support technique](https://support.microsoft.com/supportforbusiness/productselection?sapId=cf791efa-485b-95a3-6fad-3daf9cd4027c).
  - Si vous devez vérifier l’alias principal, accédez à [https://account.live.com](https://account.live.com). Cliquez ensuite sur **Vos informations**, puis sur **Gérer la manière dont vous vous connectez à Microsoft**. Suivez les invites pour vérifier une autre adresse e-mail et obtenir un code pour accéder aux informations sensibles. Entrez le code de sécurité. Sélectionnez **Configurer ultérieurement** si vous ne souhaitez pas configurer l’authentification à deux facteurs.
  - Vous verrez la page **Gérer la manière dont vous vous connectez à Microsoft**, dans laquelle vous pouvez consulter vos alias de compte. Vérifiez que l’alias principal est celui que vous utilisez pour vous connecter au portail Azure EA. Si ce n’est pas le cas, vous pouvez en faire votre alias principal. Ou, vous pouvez utiliser l’alias principal pour le portail Azure EA à la place.

## <a name="azure-ea-activation-faq"></a>FAQ sur l’activation d’Azure EA

Cette section de l’article décrit les solutions aux problèmes courants liés à l’activation d’Azure EA.

### <a name="i-didnt-receive-an-activation-email"></a>Je n’ai pas reçu d’e-mail d’activation

Un e-mail d’activation provenant du portail Azure EA est envoyé à partir de *waep@microsoft.com* . Si vous n’avez pas reçu l’e-mail d’activation, recherchez-le dans votre dossier de courrier indésirable. Il est envoyé avec l’objet _Invitation à voir/gérer Microsoft Azure service_subject_. Il est envoyé à chaque administrateur EA nouvellement ajouté.

Si vous êtes sûr d’avoir été configuré comme administrateur EA, vous n’êtes pas obligé d’attendre de recevoir l’e-mail d’activation pour vous connecter au portail Azure EA. Au lieu de cela, vous pouvez accéder à https://ea.azure.com et vous connecter avec votre adresse e-mail (compte professionnel, scolaire ou Microsoft) et votre mot de passe.

### <a name="i-would-like-to-add-a-new-ea-administrator-to-my-enrollment"></a>Je souhaite ajouter un nouvel administrateur EA à mon inscription

Un nouvel administrateur d’entreprise peut être ajouté par des administrateurs d’entreprise existants. Si vous êtes l’administrateur EA, connectez-vous au portail EA > cliquez sur **Gérer** > sur **+ Ajouter un administrateur** dans le coin supérieur droit pour ajouter un nouvel administrateur EA. Vérifiez que vous disposez de son adresse e-mail et de la méthode de connexion préférée, via l’authentification professionnelle/scolaire ou Microsoft Live ID par exemple, pour que les utilisateurs soient ajoutés.

Si vous n’êtes pas l’administrateur EA, contactez les administrateurs EA de votre entreprise pour leur demander de vous ajouter à l’inscription. Une fois qu’ils vous ont ajouté à l’inscription, vous recevez un e-mail d’activation.

Toutefois, si les administrateurs EA ne sont pas en mesure de vous aider, nous pouvons vous ajouter en leur nom si vous pouvez nous fournir les éléments suivants :
- Numéro d’inscription.
- Adresse e-mail à ajouter et type d’authentification (professionnel/scolaire/MS).
- Approbation reçue par e-mail de l’administrateur EA.

Une fois que vous disposez de toutes les informations requises, envoyez une requête à [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport).

### <a name="i-would-like-to-update-the-first-ea-admin-on-the-enrollment"></a>Je souhaite mettre à jour le premier administrateur EA sur l’inscription

Le premier administrateur EA peut être mis à jour dans le centre de gestion des licences en volume en mettant à jour le contact de notification et l’administrateur en ligne sur le portail. La mise à jour du portail EA prendra environ 24 heures. Une fois la mise à jour terminée, le nouvel administrateur EA reçoit l’e-mail d’activation.

Si vous n’avez pas accès au portail VLSC ou si votre administrateur EA initial ne peut plus gérer l’inscription et n’a pas accès au portail EA, envoyez une requête à [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport) pour demander une mise à jour et fournissez les informations suivantes :
- Numéro d’inscription
- Adresse e-mail à ajouter et type d’authentification (professionnel/scolaire/MS)
- Raison du changement d’administrateur EA initial
- Approbation reçue par e-mail de l’administrateur EA initial

### <a name="my-current-ea-admin-is-no-longer-with-the-company"></a>Mon administrateur EA ne fait plus partie de l’entreprise

Une inscription EA peut avoir plusieurs administrateurs EA, vous pouvez contacter un autre administrateur EA pour qu’il ajoute un nouvel administrateur EA/de nouveaux propriétaires de compte/un nouvel administrateur de service. Toutefois, si vous n’êtes pas certain de savoir qui est l’administrateur EA dans votre entreprise ou si aucun autre administrateur EA n’est disponible sur l’inscription, fournissez-nous les informations suivantes :
- Numéro d’inscription
- Adresse e-mail à ajouter et type d’authentification (professionnel/scolaire/MS)
- Informations telles que l’administrateur EA actuel ne fait plus partie de l’entreprise

Notez que s’il existe d’autres administrateurs EA sur l’inscription, nous contacterons les administrateurs EA pour demander leur approbation sur les modifications administratives apportées à l’inscription.

### <a name="my-enrollment-is-showing-in-pending-status-how-do-i-activate-my-enrollment"></a>L’état affiché de mon inscription est En attente. Comment activer mon inscription ?

Les inscriptions sont à l’état En attente si l’administrateur EA initial ne s’est pas connecté à l’inscription auparavant. Si vous êtes l’administrateur EA, connectez-vous au portail Azure EA. Sur la page d’accueil avec tous vos numéros d’inscription, vous ne verrez peut-être pas votre inscription en attente. Décochez la case « actif » en haut à droite de votre portail EA. Cette action affiche l’inscription en attente. Cliquez sur l’inscription pour accéder aux informations puis, une fois que vous avez atteint la page Gérer de l’inscription, l’état est mis à jour de En attente à Actif.

### <a name="why-is-my-account-stuck-in-pending-status"></a>Pourquoi mon compte est-il bloqué à l’état En attente ?

Quand de nouveaux propriétaires de comptes sont ajoutés pour la première fois à l’inscription, « En attente » s’affiche toujours sous l’état. Lors de la réception de l’e-mail de bienvenue et d’activation, l’AO peut se connecter pour activer son compte. La connexion met à jour l’état du compte de « en attente » à « actif ».

### <a name="i-received-an-error-when-signing-in-to-azure-ea-portal"></a>J’ai reçu une erreur lors de la connexion au portail Azure EA

Plusieurs raisons sont possibles pour un message d’erreur sur le portail Azure EA lors de la connexion. Veuillez suivre les étapes de résolution des problèmes suivantes :

 1. Vérifiez que vous utilisez l’URL de portail EA correcte sur [https://ea.azure.com](https://ea.azure.com).
 1. Déterminez si votre accès au portail Azure EA a été ajouté en tant que compte professionnel ou scolaire ou Microsoft Live ID. Si vous utilisez votre compte professionnel, veuillez entrer votre adresse e-mail professionnelle et votre mot de passe professionnel. Si vous utilisez Microsoft Live ID, veuillez entrer votre adresse e-mail Live ID et votre mot de passe Microsoft Live ID. Si vous avez oublié votre mot de passe Microsoft Live ID, réinitialisez-le à [https://account.live.com/password/reset](https://account.live.com/password/reset).
 1. Il est recommandé d’utiliser un navigateur privé pour vous connecter afin qu’aucun cookie ou cache des sessions précédentes/existantes ne soit conservé. Effacez le cache, puis utilisez une fenêtre de mode de navigation privée/incognito pour ouvrir [https://ea.azure.com](https://ea.azure.com).
 1. Si vous obtenez une erreur « Utilisateur non valide » lors de l’utilisation d’un compte Microsoft, cela peut être dû au fait que vous avez plusieurs comptes Microsoft et que celui avec lequel vous tentez de vous connecter n’est pas l’alias principal. Pour vérifier l’alias principal, accédez à account.live.com :
    - Accédez à « Vos informations » > « Gérer votre adresse e-mail de connexion ou numéro de téléphone ».
    - Suivez les invites à l’écran pour vérifier une autre adresse e-mail et obtenir un code pour accéder aux informations sensibles.
    - Entrez le code de sécurité.
    - Si vous préférez configurer l’authentification en deux étapes ultérieurement, sélectionnez « Configurer ultérieurement ».
    - Vous allez accéder à la page « Gérer vos alias de compte » dans laquelle vous verrez les alias de compte dont vous disposez. Vérifiez que l’alias principal est celui que vous utilisez pour vous connecter au portail Azure EA. Si ce n’est pas le cas, vous pouvez en faire votre alias principal ou utiliser l’alias principal pour le portail Azure EA à la place.

En cas d’échec des étapes de résolution des problèmes ci-dessus, envoyez une requête à [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport) en fournissant des informations telles que :
- Les navigateurs et la version utilisés.
- Capture d’écran du message d’erreur.
- URL de la page indiquant une erreur.  
- Date, heure et fuseau horaire lorsque l’erreur s’est produite.
- En outre, un fichier journal, si vous en obtenez un, sera utile. Voici les étapes à suivre pour capturer une trace réseau à l’aide des informations ci-dessous :
  1. Ouvrez Internet Explorer.
  1. Appuyez sur F12, une zone en bas d’IE s’ouvre.
  1. Sélectionnez l’onglet **Réseau**.
  1. Cliquez sur **Démarrer la capture**.
  1. Effectuez l’action qui est à l’origine de l’erreur.
  1. Une fois que vous avez obtenu l’erreur, cliquez sur **Arrêter la capture**.
  1. Enregistrez le fichier et incluez les informations dans la demande de support.
  1. Veillez à fournir votre numéro d’inscription et adresse e-mail dans la demande de support.

### <a name="what-is-the-difference-between-a-workschool-account-and-microsoft-account"></a>Quelle est la différence entre un compte professionnel/scolaire et un compte Microsoft ?

**Compte Microsoft :** Comptes qui ont été associés à Live ID sur [https://signup.live.com](https://signup.live.com).

**Compte professionnel/scolaire :** Disponible uniquement pour les organisations qui ont configuré Active Directory avec une fédération dans le cloud, et dont tous les comptes se trouvent sur un seul locataire. Les utilisateurs peuvent être ajoutés avec le type d’autorisation professionnel/scolaire si le service Active Directory interne de l’entreprise est fédéré sur le cloud.

  Depuis septembre 2016, Microsoft n’autorise plus l’inscription d’adresse e-mail professionnelle ou scolaire en tant que compte Microsoft. Pour plus de détails, consultez les documents suivants : [https://blogs.technet.microsoft.com/enterprisemobility/2016/09/15/cleaning-up-the-azure-ad-and-microsoft-account-overlap/](https://blogs.technet.microsoft.com/enterprisemobility/2016/09/15/cleaning-up-the-azure-ad-and-microsoft-account-overlap/).

  Si votre organisation n’est pas fédérée sur le cloud, vous ne pourrez pas utiliser votre adresse e-mail professionnelle ou scolaire. Inscrivez ou créez une nouvelle adresse e-mail et inscrivez-la en tant que compte Microsoft à la place.

### <a name="i-forgot-my-password-to-azure-ea-portal"></a>J’ai oublié mon mot de passe du portail Azure EA

Si vous avez oublié votre mot de passe Microsoft Live ID, réinitialisez-le à [https://account.live.com/password/reset](https://account.live.com/password/reset).

Si vous avez oublié votre mot de passe professionnel, contactez l’administrateur informatique de votre entreprise.

### <a name="i-have-a-valid-work-or-school-account-but-i-cant-add-it-to-the-ea-portal"></a>Je dispose d’un compte professionnel ou scolaire valide, mais je ne peux pas l’ajouter au portail EA

Si vous disposez d’un compte professionnel ou scolaire sous un autre locataire, définissez « compte professionnel ou scolaire inter-locataire » comme niveau d’autorisation dans la page des détails de l’inscription. Vous pourrez alors ajouter le compte.

## <a name="next-steps"></a>Étapes suivantes

- Les administrateurs du portail Azure EA doivent lire la [documentation consacrée à l’administration du portail Azure EA](ea-portal-administration.md) pour découvrir les tâches d’administration courantes.
