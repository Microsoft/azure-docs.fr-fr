---
author: georgewallace
ms.service: multiple
ms.topic: include
ms.date: 11/25/2018
ms.author: gwallace
ms.openlocfilehash: 3590b8fdb67b25e0d4389e814aefcc4d18b1193c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015421"
---
Les clients Azure peuvent débloquer 25 000 courriers électroniques gratuits chaque mois. Ces 25 000 courriers électroniques mensuels gratuits vous donnent accès aux rapports avancés, aux analyses et à [toutes les API][all APIs] (Web, SMTP, Event, Parse, etc.). Pour plus d’informations sur les services supplémentaires proposés par SendGrid, visitez la page [Solutions SendGrid][SendGrid Solutions].

### <a name="to-sign-up-for-a-sendgrid-account"></a>Création d’un compte SendGrid
1. Connectez-vous au [portail Azure][Azure portal].
2. Dans le menu du portail Azure ou dans la page d’accueil, sélectionnez **Créer une ressource**.

    ![Capture d’écran du menu du portail Azure avec l’option Créer une ressource sélectionnée.][command-bar-new]
3. Recherchez et sélectionnez **SendGrid**.

    ![Capture d’écran de l’écran Place de marché du portail Azure montrant « SendGr » dans la zone de recherche, et SendGrid sélectionné dans les résultats de la recherche.][sendgrid-store]
4. Remplissez le formulaire d’inscription, puis sélectionnez **Créer**.

    ![Capture d’écran de la boîte de dialogue Créer un compte SendGrid avec les champs Nom, Mot de passe, Abonnement et Groupe de ressources renseignés.][sendgrid-create]
5. Entrez un **nom** pour identifier votre service SendGrid dans vos paramètres Azure. Les noms doivent contenir entre 1 et 100 caractères et ne peuvent inclure que des lettres, des chiffres, des tirets, des points et des traits de soulignement. Le nom doit être unique dans la liste des éléments de l’Azure Store auxquels vous êtes abonné.
6. Entrez et confirmez votre **mot de passe**.
7. Choisissez votre **abonnement**.
8. Créez un **groupe de ressources** ou utilisez un groupe existant.
9. Dans la boîte de dialogue **Niveau tarifaire**, sélectionnez le plan SendGrid de votre choix pour l’inscription.

    ![Capture d’écran de la boîte de dialogue Créer un compte SendGrid avec la section Choisir votre niveau tarifaire ouverte, et le niveau de tarification Gratuit sélectionné.][sendgrid-pricing]
10. Saisissez un **code de promotion**, le cas échéant.
11. Entrez vos **informations de contact**.
12. Lisez et acceptez les **conditions juridiques**.
13. Après avoir confirmé votre achat, une fenêtre contextuelle **Déploiement a réussi** apparaît et votre compte s’affiche.

    ![Capture d’écran de la page Comptes SendGrid montrant le nouveau compte ContosoSendGrid.][all-resources]

    Après avoir effectué votre achat et cliqué sur le bouton **Gérer** pour lancer le processus de vérification par courrier électronique, vous recevrez un e-mail de la part de SendGrid vous demandant de vérifier votre compte. Si vous ne recevez pas cet e-mail ou si vous avez des difficultés pour vérifier votre compte, veuillez consulter notre FAQ.

    ![Capture d’écran de la page du compte ContosoSendGrid avec le bouton Gérer en surbrillance.][manage]

    **Vous ne pouvez envoyer que 100 e-mails/jour jusqu'à ce que vous ayez vérifié votre compte.**

    Pour modifier votre plan d’abonnement ou voir les paramètres de contact de SendGrid, cliquez sur le nom du service SendGrid pour ouvrir le tableau de bord Marketplace SendGrid.

    ![Capture d’écran montrant que la page Paramètres du compte ContosoSendGrid est ouverte en sélectionnant Tous les paramètres dans la page du compte ContosoSendGrid.][settings]

    Pour envoyer un e-mail à l’aide de SendGrid, vous devez fournir votre clé API.

### <a name="to-find-your-sendgrid-api-key"></a>Pour trouver votre clé API SendGrid
1. Cliquez sur **Gérer**.

    ![Capture d’écran de la page du compte ContosoSendGrid avec le bouton Gérer en surbrillance.][manage]
2. Dans votre tableau de bord SendGrid, sélectionnez **Paramètres**, puis **Clés API** dans le menu de gauche.

    ![Capture d’écran du tableau de bord SendGrid avec la liste déroulante Paramètres ouverte et l’option Clés API sélectionnée.][api-keys]

3. Cliquez sur **Créer une clé API**.

    ![Capture d’écran de l’écran Clés API avec le bouton Créer une clé API sélectionné.][general-api-key]
4. Au minimum, fournissez le **nom de cette clé** et un accès complet à **Mail Send** (Envoi de courrier) puis sélectionnez **Enregistrer**.

    ![Capture d’écran de l’écran Ajouter une nouvelle clé API générale avec l’option Envoi de courrier définie sur Accès total, l’option Envois planifiés définie sur Aucun accès, et le bouton Enregistrer en surbrillance.][access]
5. À ce stade, votre API s’affiche. Conservez-la en sécurité.

### <a name="to-find-your-sendgrid-credentials"></a>Recherche de vos informations d’identification SendGrid
1. Cliquez sur l’icône en forme de clé pour trouver votre **nom d’utilisateur**.

    ![Capture d’écran de la page du compte ContosoSendGrid avec l’icône de clé en surbrillance.][key]
2. Le mot de passe est celui que vous avez choisi lors de l’installation. Vous pouvez sélectionner **Modifier le mot de passe** ou **Réinitialiser le mot de passe** pour apporter des modifications.

Pour gérer les paramètres de remise des courriers électroniques, cliquez sur le bouton **Manage** . Cette action vous redirige vers le tableau de bord SendGrid.

![Capture d’écran de la page du compte ContosoSendGrid avec le bouton Gérer en surbrillance.][manage]

Pour plus d’informations sur l’envoi d’e-mails via SendGrid, consultez [Vue d’ensemble des API d’e-mail][Email API Overview].

<!--images-->

[command-bar-new]: ./media/sendgrid-sign-up/new-addon.png
[sendgrid-store]: ./media/sendgrid-sign-up/sendgrid-store.png
[sendgrid-create]: ./media/sendgrid-sign-up/sendgrid-create.png
[sendgrid-pricing]: ./media/sendgrid-sign-up/sendgrid-pricing.png
[all-resources]: ./media/sendgrid-sign-up/all-resources.png
[manage]: ./media/sendgrid-sign-up/manage.png
[settings]: ./media/sendgrid-sign-up/settings.png
[api-keys]: ./media/sendgrid-sign-up/api-keys.png
[general-api-key]: ./media/sendgrid-sign-up/general-api-key.png
[access]: ./media/sendgrid-sign-up/access.png
[key]: ./media/sendgrid-sign-up/key.png

<!--Links-->

[SendGrid Solutions]: https://sendgrid.com/solutions
[Azure portal]: https://portal.azure.com
[SendGrid Getting Started]: http://sendgrid.com/docs
[SendGrid Provisioning Process]: https://support.sendgrid.com/hc/articles/200181628-Why-is-my-account-being-provisioned-
[all APIs]: https://sendgrid.com/docs/API_Reference/index.html
[Email API Overview]: https://sendgrid.com/docs/API_Reference/Web_API_v3/Mail/index.html
