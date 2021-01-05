---
title: Charger des données en bloc pour ajouter ou créer des membres dans un groupe - Azure Active Directory | Microsoft Docs
description: Ajoutez en bloc les membres d’un groupe dans le centre d’administration Azure Active Directory.
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 12/02/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 22d39a2ee66f2c63612ad2cb3cf9ae61f1660de3
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96547744"
---
# <a name="bulk-add-group-members-in-azure-active-directory"></a>Ajouter en bloc les membres d’un groupe dans Azure Active Directory

En utilisant le portail Azure Active Directory (Azure AD), vous pouvez ajouter un grand nombre de membres à un groupe en utilisant un fichier de valeurs séparées par des virgules (CSV) pour importer en bloc les membres du groupe.

## <a name="understand-the-csv-template"></a>Comprendre le modèle CSV

Téléchargez et renseignez le modèle CSV de chargement en bloc pour ajouter des membres du groupe Azure AD en bloc. Votre modèle CSV peut se présenter comme dans l’exemple suivant :

![Feuille de calcul pour le chargement et les appels expliquant l’objectif et les valeurs de chaque ligne et colonne](./media/groups-bulk-import-members/template-with-callouts.png)

### <a name="csv-template-structure"></a>Structure du modèle CSV

Les lignes d’un modèle CSV téléchargé sont les suivantes :

- **Numéro de version** : La première ligne contenant le numéro de version doit être incluse dans le fichier CSV chargé.
- **En-têtes de colonne** : Le format des en-têtes de colonne est &lt;*Nom d’élément*&gt; [PropertyName] &lt;*Obligatoire ou vide*&gt;. Par exemple : `Member object ID or user principal name [memberObjectIdOrUpn] Required`. Certaines anciennes versions du modèle peuvent avoir de légères variations. Pour les modifications d’appartenance à un groupe, vous avez le choix de l’identificateur : ID d’objet membre ou nom d’utilisateur principal.
- **Exemples de lignes** : Nous avons inclus dans le modèle une ligne d’exemples de valeurs acceptables pour chaque colonne. Vous devez supprimer la ligne des exemples et la remplacer par vos propres entrées.

### <a name="additional-guidance"></a>Conseils supplémentaires

- Les deux premières lignes du modèle chargé ne doivent pas être supprimées ni modifiées, sinon, le chargement ne pourra pas être traité.
- Les colonnes obligatoires sont listées en premier.
- Nous vous déconseillons d’ajouter des colonnes au modèle. Les colonnes que vous ajouterez seront ignorées et ne seront pas traitées.
- Nous vous recommandons de télécharger la version la plus récente du modèle CSV aussi souvent que possible.
- Ajoutez au moins deux UPN ou ID d’objet pour télécharger le fichier avec succès.

## <a name="to-bulk-import-group-members"></a>Pour importer en bloc les membres du groupe

1. Connectez-vous [au portail Azure](https://portal.azure.com) avec un compte Utilisateur administrateur dans l’organisation. Les propriétaires de groupes peuvent également importer en bloc les membres de groupes qu'ils possèdent.
1. Dans Azure AD, sélectionnez **Groupes** > **Tous les groupes**.
1. Ouvrez le groupe auquel vous ajoutez des membres, puis sélectionnez **Membres**.
1. Dans la page **Membres**, sélectionnez **Importer des membres**.
1. Dans la page **Importer en bloc les membres du groupe**, sélectionnez **Télécharger** pour obtenir le modèle de fichier CSV avec les propriétés des membres du groupe requises.

    ![La commande Importer des membres se trouve sur la page de profil du groupe](./media/groups-bulk-import-members/import-panel.png)

1. Ouvrez le fichier CSV et ajoutez une ligne pour chaque membre du groupe que vous souhaitez importer dans le groupe (les valeurs requises sont **ID d’objet de membre** ou **Nom principal de l’utilisateur**). Puis enregistrez le fichier.

    :::image type="content" source="./media/groups-bulk-import-members/csv-file.png" alt-text="Le fichier CSV contient les noms et les ID des membres à importer":::

1. Dans la page **Importer en bloc les membres du groupe**, sous **Chargez votre fichier .csv**, accédez au fichier. Quand vous sélectionnez le fichier, la validation du fichier CSV démarre.
1. Quand le contenu du fichier est validé, la page d’importation en bloc affiche **Fichier chargé**. Si des erreurs sont présentes, vous devez les corriger avant de pouvoir envoyer le travail.
1. Une fois votre fichier validé, sélectionnez **Envoyer** pour démarrer l’opération en bloc Azure qui importe les membres du groupe dans le groupe.
1. Une fois l’opération d’importation terminée, vous recevez une notification indiquant que l’opération en bloc a réussi.

## <a name="check-import-status"></a>Vérifier l'état de l'importation

Vous pouvez voir l’état de toutes vos demandes d’opération en bloc en attente dans la page **Résultats de l’opération en bloc**.

[![Vérifier l'état sur la page Résultats des opérations en bloc.](./media/groups-bulk-import-members/bulk-center.png)](./media/groups-bulk-import-members/bulk-center.png#lightbox)

Pour plus d’informations sur chaque élément de ligne au sein de l’opération en bloc, sélectionnez les valeurs sous les colonnes **Nombre de réussites**, **Nombre d’échecs** ou **Nombre total de requêtes**. Si des échecs se sont produits, les raisons sont affichées.

## <a name="bulk-import-service-limits"></a>Limites du service d’importation en bloc

Chaque opération en bloc pour importer une liste des membres d’un groupe peut prendre jusqu’à une heure. Ceci permet l’importation d’une liste d’au plus 40 000 membres.

## <a name="next-steps"></a>Étapes suivantes

- [Supprimer en bloc les membres d’un groupe](groups-bulk-remove-members.md)
- [Télécharger les membres d’un groupe](groups-bulk-download-members.md)
- [Télécharger une liste de tous les groupes](groups-bulk-download.md)
