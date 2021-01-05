---
title: Supprimer en bloc les membres d’un groupe en chargeant un fichier CSV - Azure Active Directory | Microsoft Docs
description: Supprimez en bloc les membres d'un groupe dans le centre d’administration Azure.
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 11/15/2020
ms.topic: how-to
ms.service: active-directory
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 874d434e545555013812714df4d1bd23de9c8543
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95488944"
---
# <a name="bulk-remove-group-members-in-azure-active-directory"></a>Supprimer en bloc les membres d’un groupe dans Azure Active Directory

En utilisant le portail Azure Active Directory (Azure AD), vous pouvez supprimer un grand nombre de membres d’un groupe en utilisant un fichier de valeurs séparées par des virgules (CSV) pour supprimer en bloc les membres du groupe.

## <a name="understand-the-csv-template"></a>Comprendre le modèle CSV

Téléchargez et renseignez le modèle CSV de chargement en bloc pour ajouter des membres du groupe Azure AD en bloc. Votre modèle CSV peut se présenter comme dans l’exemple suivant :

![Feuille de calcul pour le chargement et les appels expliquant l’objectif et les valeurs de chaque ligne et colonne](./media/groups-bulk-remove-members/template-example.png)

### <a name="csv-template-structure"></a>Structure du modèle CSV

Les lignes d’un modèle CSV téléchargé sont les suivantes :

- **Numéro de version** : La première ligne contenant le numéro de version doit être incluse dans le fichier CSV chargé.
- **En-têtes de colonne** : Le format des en-têtes de colonne est &lt;*Nom d’élément*&gt; [PropertyName] &lt;*Obligatoire ou vide*&gt;. Par exemple : `Member object ID or user principal name [memberObjectIdOrUpn] Required`. Certaines anciennes versions du modèle peuvent avoir de légères variations. Pour les modifications d’appartenance à un groupe, vous avez le choix de l’identificateur : ID d’objet membre ou nom d’utilisateur principal.
- **Exemples de lignes** : Nous avons inclus dans le modèle une ligne d’exemples de valeurs acceptables pour chaque colonne. Vous devez supprimer la ligne des exemples et la remplacer par vos propres entrées.

### <a name="additional-guidance"></a>Conseils supplémentaires

- Les deux premières lignes du modèle chargé ne doivent pas être supprimées ni modifiées, sinon, le chargement ne pourra pas être traité.
- Les colonnes obligatoires sont listées en premier.
- Nous vous déconseillons d’ajouter des colonnes au modèle. Les colonnes que vous ajouterez seront ignorées et ne seront pas traitées.
- Nous vous recommandons de télécharger la version la plus récente du modèle CSV aussi souvent que possible.

## <a name="to-bulk-remove-group-members"></a>Supprimer en bloc les membres d’un groupe

1. Connectez-vous [au portail Azure](https://portal.azure.com) avec un compte Utilisateur administrateur dans l’organisation. Les propriétaires de groupes peuvent également supprimer en bloc les membres de groupes qu'ils possèdent.
1. Dans Azure AD, sélectionnez **Groupes** > **Tous les groupes**.
1. Ouvrez le groupe dont vous supprimez des membres, puis sélectionnez **Membres**.
1. Dans la page **Membres**, sélectionnez **Supprimer des membres**.
1. Dans la page **Supprimer en bloc les membres du groupe**, sélectionnez **Télécharger** pour obtenir le modèle de fichier CSV avec les propriétés des membres du groupe requises.

   ![La commande Supprimer des membres se trouve sur la page de profil du groupe](./media/groups-bulk-remove-members/remove-panel.png)

1. Ouvrez le fichier CSV et ajoutez une ligne pour chaque membre du groupe que vous souhaitez supprimer du groupe (les valeurs requises sont ID d’objet de membre ou Nom principal de l’utilisateur). Puis enregistrez le fichier.

    :::image type="content" source="./media/groups-bulk-import-members/csv-file.png" alt-text="Le fichier CSV contient les noms et les ID des membres du groupe à supprimer":::

1. Dans la page **Supprimer en bloc les membres du groupe**, sous **Chargez votre fichier .csv**, accédez au fichier. Quand vous sélectionnez le fichier, la validation du fichier CSV démarre.
1. Quand le contenu du fichier est validé, la page d’importation en bloc affiche **Fichier chargé**. Si des erreurs sont présentes, vous devez les corriger avant de pouvoir envoyer le travail.
1. Une fois votre fichier validé, sélectionnez **Envoyer** pour démarrer l’opération en bloc Azure qui supprime les membres du groupe.
1. Une fois l’opération de suppression terminée, vous recevez une notification indiquant que l’opération en bloc a réussi.

## <a name="check-removal-status"></a>Vérifier l’état de la suppression

Vous pouvez voir l’état de toutes vos demandes d’opération en bloc en attente dans la page **Résultats de l’opération en bloc**.

[![Vérifier l'état sur la page Résultats des opérations en bloc](./media/groups-bulk-remove-members/bulk-center.png)](./media/groups-bulk-remove-members/bulk-center.png#lightbox)

Pour plus d’informations sur chaque élément de ligne au sein de l’opération en bloc, sélectionnez les valeurs sous les colonnes **Nombre de réussites**, **Nombre d’échecs** ou **Nombre total de requêtes**. Si des échecs se sont produits, les raisons sont affichées.

## <a name="bulk-removal-service-limits"></a>Limites du service de suppression en bloc

Chaque opération en bloc de suppression d’une liste d’utilisateurs peut prendre jusqu’à une heure. Ceci permet la suppression d'une liste d'au moins 40 000 membres.

## <a name="next-steps"></a>Étapes suivantes

- [Importer en bloc les membres du groupe](groups-bulk-import-members.md)
- [Télécharger les membres d’un groupe](groups-bulk-download-members.md)
- [Télécharger une liste de tous les groupes](groups-bulk-download.md)
