---
title: Connecter un compte Amazon Web Services à Cloudyn dans Azure
description: Connectez un compte Amazon Web Services pour afficher les données de coût et d’utilisation dans les rapports Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: de0fd1a27ca97545ccdd534908eb802993514017
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690168"
---
# <a name="connect-an-amazon-web-services-account"></a>Connecter un compte Amazon Web Services

Vous avez deux possibilités pour connecter votre compte Amazon Web Services (AWS) à Cloudyn. Vous pouvez le connecter avec un rôle IAM ou avec un compte utilisateur IAM en lecture seule. Le rôle IAM est recommandé, car il vous permet de déléguer l’accès avec des autorisations définies pour les entités approuvées. Le rôle IAM ne vous oblige pas à partager les clés d’accès sur le long terme. Une fois que vous avez connecté un compte AWS à Cloudyn, les données de coût et d’utilisation sont disponibles dans les rapports Cloudyn. Ce document vous présente les deux options disponibles.

Pour plus d’informations sur les identités AWS IAM, consultez [Identités (utilisateurs, groupes et rôles)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html).

En outre, vous activez les rapports de facturation détaillée AWS et vous stockez les informations dans un compartiment AWS Simple Storage Service (S3). Les rapports de facturation détaillée incluent les frais de facturation avec des informations de balisage et de ressource sur une base horaire. Le stockage des rapports permet à Cloudyn de récupérer ces derniers à partir de votre compartiment et d’afficher les informations dans ses rapports.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]
## <a name="aws-role-based-access"></a>Accès en fonction du rôle AWS

Les sections suivantes vous guident durant la création d’un rôle IAM en lecture seule pour fournir l’accès à Cloudyn.

### <a name="get-your-cloudyn-account-external-id"></a>Obtenir l’ID externe de votre compte Cloudyn

La première étape consiste à obtenir la phrase secrète de connexion unique à partir du portail Cloudyn. Il est utilisé dans AWS comme **ID externe**.

1. Ouvrez le portail Cloudyn à partir du portail Azure ou accédez à [https://azure.cloudyn.com](https://azure.cloudyn.com) et connectez-vous.
2. Cliquez sur le symbole de roue dentée, puis sélectionnez **Cloud Accounts** (Comptes cloud).
3. Dans Gestion de comptes, sélectionnez l’onglet **Comptes AWS**, puis cliquez sur **Ajouter un nouveau +** .
4. Dans la boîte de dialogue **Add AWS Account** (Ajouter un compte AWS), copiez la valeur **External ID** (ID externe) et enregistrez-la pour la procédure de création du rôle AWS décrite à la section suivante. L’ID externe est propre à votre compte. Dans l’image ci-après, l’exemple d’ID externe est _Contoso_ suivi d’un nombre. Votre ID est différent.  
    ![ID externe indiquée dans la boîte de dialogue Ajouter un compte AWS](./media/connect-aws-account/external-id.png)

### <a name="add-aws-read-only-role-based-access"></a>Ajouter un accès en lecture seule en fonction du rôle AWS

1. Connectez-vous à la console AWS sur [https://console.aws.amazon.com/iam/home](https://console.aws.amazon.com/iam/home), puis sélectionnez **Rôles**.
2. Cliquez sur **Créer un rôle**, puis sélectionnez **Autre compte AWS**.
3. Dans la zone **Account ID** (ID de compte), collez `432263259397`. Cet ID de compte est le compte de collecteur de données Cloudyn attribué par AWS au service Cloudyn. Utilisez exactement l’ID de compte indiqué.
4. En regard du libellé **Options** (Options), sélectionnez **Require external ID** (Nécessite un ID externe). Collez la valeur propre à votre compte que vous avez copiée à partir du champ **External ID** (ID externe) de Cloudyn. Cliquez ensuite sur **Suivant : Autorisations**.  
    ![Coller l’ID externe depuis Cloudyn sur la page Créer un rôle](./media/connect-aws-account/create-role01.png)
5. Sous **Joindre des stratégies d’autorisation**, dans la recherche de filtre **Type de stratégie**, tapez `ReadOnlyAccess`, sélectionnez **ReadOnlyAccess**, puis cliquez sur **Suivant : Révision**.  
    ![Sélectionner l’accès en lecture seule dans la liste des noms de stratégie](./media/connect-aws-account/readonlyaccess.png)
6. Sur la page Révision, vérifiez que vos sélections sont correctes, puis tapez un **nom de rôle**. Par exemple, *Azure-Cost-Mgt*. Entrez une **description du rôle**. Par exemple, _Attribution de rôle pour Cloudyn_, puis cliquez sur **Créer un rôle**.
7. Dans la liste **Rôles**, cliquez sur le rôle que vous avez créé et copiez la valeur du champ **Role ARN** à partir de la page de résumé. Utilisez la valeur d’ARN (Amazon Resource Name, nom de ressource Amazon) de rôle ultérieurement quand vous enregistrerez votre configuration dans Cloudyn.  
    ![Copier l’ARN de rôle à partir de la page Résumé](./media/connect-aws-account/role-arn.png)

### <a name="configure-aws-iam-role-access-in-cloudyn"></a>Configurer l’accès en fonction du rôle AWS IAM dans Cloudyn

1. Ouvrez le portail Cloudyn à partir du portail Azure ou accédez à https://azure.cloudyn.com/ et connectez-vous.
2. Cliquez sur le symbole de roue dentée, puis sélectionnez **Cloud Accounts** (Comptes cloud).
3. Dans Gestion de comptes, sélectionnez l’onglet **Comptes AWS**, puis cliquez sur **Ajouter un nouveau +** .
4. Dans **Nom de compte**, saisissez un nom pour le compte.
5. En regard de **Type d’accès**, sélectionnez **Rôle IAM**.
6. Dans le champ **Role ARN**, collez la valeur que vous avez copiée précédemment, puis cliquez sur **Enregistrer**.  
    ![Coller l’ARN de rôle dans la boîte de dialogue Ajouter un compte AWS](./media/connect-aws-account/add-aws-account-box.png)


Votre compte AWS apparaît dans la liste des comptes. L’**ID de propriétaire** répertorié correspond à la valeur du champ Role ARN. La colonne **Account Status** (État du compte) doit présenter une coche verte indiquant que Cloudyn peut accéder à votre compte AWS. Jusqu’à ce que vous activiez la facturation AWS détaillée, votre état de consolidation présente la valeur **Standalone** (Autonome).

![État du compte AWS indiqué sur la page de gestion des comptes](./media/connect-aws-account/aws-account-status01.png)

Cloudyn commence la collecte des données et le remplissage des rapports. Ensuite, [activez la facturation AWS détaillée](#enable-detailed-aws-billing).


## <a name="aws-user-based-access"></a>Accès en fonction de l’utilisateur AWS

Les sections suivantes vous guident durant la création d’un utilisateur en lecture seule pour fournir l’accès à Cloudyn.

### <a name="add-aws-read-only-user-based-access"></a>Ajouter un accès en lecture seule en fonction de l’utilisateur AWS

1. Connectez-vous à la console AWS sur [https://console.aws.amazon.com/iam/home](https://console.aws.amazon.com/iam/home), puis sélectionnez **Utilisateurs**.
2. Cliquez sur **Add User**.
3. Dans le champ **Nom d’utilisateur**, tapez un nom d’utilisateur.
4. Sous **Type d’accès**, sélectionnez **Accès par programme** et cliquez sur **Suivant : Autorisations**.  
    ![Entrer un nom d’utilisateur sur la page Ajouter un utilisateur](./media/connect-aws-account/add-user01.png)
5. Pour les autorisations, sélectionnez **Attacher directement les stratégies existantes**.
6. Sous **Joindre des stratégies d’autorisation**, dans la recherche de filtre **Type de stratégie**, tapez `ReadOnlyAccess`, sélectionnez **ReadOnlyAccess**, puis cliquez sur **Suivant : Révision**.  
    ![Sélectionner ReadOnlyAccess pour définir les autorisations de l’utilisateur](./media/connect-aws-account/set-permission-for-user.png)
7. Sur la page Révision, vérifiez que vos sélections sont correctes, puis cliquez sur **Créer un utilisateur**.
8. Sur la page Terminé, votre ID de clé d’accès et la clé d’accès secrète sont affichés. Ces informations vous permettent de configurer l’inscription dans Cloudyn.
9. Cliquez sur **Télécharger .csv** et enregistrez le fichier credentials.csv dans un emplacement sécurisé.  
    ![Cliquer sur Télécharger le fichier .csv pour enregistrer les informations d’identification](./media/connect-aws-account/download-csv.png)

### <a name="configure-aws-iam-user-based-access-in-cloudyn"></a>Configurer l’accès en fonction de l’utilisateur AWS IAM dans Cloudyn

1. Ouvrez le portail Cloudyn à partir du portail Azure ou accédez à https://azure.cloudyn.com/ et connectez-vous.
2. Cliquez sur le symbole de roue dentée, puis sélectionnez **Cloud Accounts** (Comptes cloud).
3. Dans Gestion de comptes, sélectionnez l’onglet **Comptes AWS**, puis cliquez sur **Ajouter un nouveau +** .
4. Sous **Nom de compte**, tapez un nom de compte.
5. En regard de **Type d’accès**, sélectionnez **Utilisateur IAM**.
6. Sous **Clé d’accès**, collez la valeur du champ **Access key ID** à partir du fichier credentials.csv.
7. Sous **Clé secrète**, collez la valeur du champ **Secret access key** à partir du fichier credentials.csv, puis cliquez sur **Enregistrer**.  

Votre compte AWS apparaît dans la liste des comptes. L’**état de votre compte** doit être accompagné d’une coche verte.

Cloudyn commence la collecte des données et le remplissage des rapports. Ensuite, [activez la facturation AWS détaillée](#enable-detailed-aws-billing).

## <a name="enable-detailed-aws-billing"></a>Activer la facturation AWS détaillée

Suivez la procédure ci-après pour obtenir votre ARN de rôle AWS. L’ARN de rôle vous permet d’accorder des autorisations de lecture à un compartiment de facturation.

1. Connectez-vous à la console AWS sur [https://console.aws.amazon.com](https://console.aws.amazon.com), puis sélectionnez **Services**.
2. Dans la zone de recherche de service, tapez *IAM*, puis sélectionnez cette option.
3. Dans le menu de gauche, sélectionnez **Roles** (Rôles).
4. Dans la liste des rôles, sélectionnez le rôle que vous avez créé pour l’accès Cloudyn.
5. Dans la page récapitulative des rôles, cliquez pour copier la valeur **Role ARN** (ARN de rôle). Conservez l’ARN de rôle à portée de main pour les étapes ultérieures.

### <a name="create-an-s3-bucket"></a>Créer un compartiment S3

Vous créez un compartiment S3 pour stocker les informations de facturation détaillée.

1. Connectez-vous à la console AWS sur [https://console.aws.amazon.com](https://console.aws.amazon.com), puis sélectionnez **Services**.
2. Dans la zone de recherche de service, tapez *S3*, puis sélectionnez **S3**.
3. Dans la page Amazon S3, cliquez sur **Create bucket** (Créer un compartiment).
4. Dans l’Assistant de création d’un compartiment, choisissez un nom de compartiment et une région, puis cliquez sur **Next** (Suivant).  
    ![Exemple d’informations sur la page Créer un compartiment](./media/connect-aws-account/create-bucket.png)
5. Dans la page **Set properties** (Définir les propriétés), conservez les valeurs par défaut, puis cliquez sur **Next** (Suivant).
6. Dans la page de révision, cliquez sur **Create bucket** (Créer un compartiment). Votre liste de compartiments s’affiche.
7. Cliquez sur le compartiment que vous avez créé, sélectionnez l’onglet **Permissions** (Autorisations), puis sélectionnez **Bucket Policy** (Stratégie de compartiment). L’éditeur de stratégie de compartiment s’affiche.
8. Copiez l’exemple JSON ci-après et collez-le dans l’éditeur de stratégie de compartiment.
   - Remplacez `<BillingBucketName>` par le nom de votre compartiment S3.
   - Remplacez `<ReadOnlyUserOrRole>` par l’ARN de rôle ou d’utilisateur que vous avez précédemment copié.

   ```json
   {
      "Version": "2012-10-17",
      "Id": "Policy1426774604000",
      "Statement": [
          {
              "Sid": "Stmt1426774604000",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::386209384616:root"
              },
              "Action": [
                  "s3:GetBucketAcl",
                  "s3:GetBucketPolicy"
              ],
              "Resource": "arn:aws:s3:::<BillingBucketName>"
          },
          {
              "Sid": "Stmt1426774604001",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::386209384616:root"
              },
              "Action": "s3:PutObject",
              "Resource": "arn:aws:s3:::<BillingBucketName>/*"
          },
          {
              "Sid": "Stmt1426774604002",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "<ReadOnlyUserOrRole>"
              },
              "Action": [
                  "s3:List*",
                  "s3:Get*"
              ],
              "Resource": "arn:aws:s3:::<BillingBucketName>/*"
          }
      ]
   }
   ```

9. Cliquez sur **Enregistrer**.  
    ![Cliquer sur Enregistrer dans l’éditeur de stratégie de compartiment](./media/connect-aws-account/bucket-policy-editor.png)


### <a name="enable-aws-billing-reports"></a>Activer les rapports de facturation AWS

Après avoir créé et configuré le compartiment S3, accédez à [Billing Preferences](https://console.aws.amazon.com/billing/home?#/preference) (Préférences de facturation) dans la console AWS.

1. Dans la page des préférences, sélectionnez **Receive Billing Reports** (Recevoir les rapports de facturation).
2. Sous **Receive Billing Reports** (Recevoir les rapports de facturation), entrez le nom du compartiment que vous avez créé, puis cliquez sur **Verify** (Vérifier).  
3. Sélectionnez les quatre options de granularité de rapports, puis cliquez sur **Save preferences** (Enregistrer les préférences).  
    ![Sélectionner le niveau de granularité pour activer les rapports](./media/connect-aws-account/enable-reports.png)

Cloudyn récupère les informations de facturation détaillée à partir de votre compartiment S3 et remplit les rapports après l’activation de la facturation détaillée. L’apparition des données de facturation détaillée dans la console Cloudyn peut prendre jusqu’à 24 heures. Lorsque les données de facturation détaillée sont disponibles, l’état de consolidation de votre compte présente la valeur **Consolidated** (Consolidé). L’état du compte affiche la valeur **Completed** (Terminé).

![État de la consolidation affichée sous l’onglet Comptes AWS](./media/connect-aws-account/consolidated-status.png)

Certains des rapports d’optimisation peuvent nécessiter quelques jours de données pour obtenir une taille d’échantillon de données adaptée à la fourniture de recommandations précises.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur Cloudyn, suivez le tutoriel [Vérifier l’utilisation et les coûts](tutorial-review-usage.md) sur Cloudyn.
