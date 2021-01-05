---
title: Utiliser le portail Azure pour gérer des clés gérées par le client pour Azure Data Box
description: Découvrez comment utiliser le portail Azure afin de créer et de gérer des clés gérées par le client avec Azure Key Vault pour Azure Data Box. Les clés gérées par le client vous permettent de créer, faire pivoter, désactiver et révoquer des contrôles d’accès.
services: databox
author: alkohli
ms.service: databox
ms.topic: how-to
ms.date: 11/19/2020
ms.author: alkohli
ms.subservice: pod
ms.openlocfilehash: f75907dc1fa079cebb3b80874090c658fd7b8174
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302831"
---
# <a name="use-customer-managed-keys-in-azure-key-vault-for-azure-data-box"></a>Utiliser des clés gérées par le client dans Azure Key Vault pour Azure Data Box

Azure Data Box protège la clé de déverrouillage de l’appareil (également appelée mot de passe de l’appareil). Cette dernière est utilisée pour verrouiller un appareil via une clé de chiffrement. Par défaut, cette clé de chiffrement est une clé gérée par Microsoft. Pour plus de contrôle, vous pouvez utiliser une clé gérée par le client.

L’utilisation d’une clé gérée par le client n’affecte pas la manière dont les données de l’appareil sont chiffrées. Elle affecte uniquement le mode de chiffrement de la clé de déverrouillage de l’appareil.

Pour conserver ce niveau de contrôle tout au long du processus de commande, utilisez une clé gérée par le client lorsque vous créez votre commande. Pour plus d’informations, consultez [Didacticiel : Commander Azure Data Box](data-box-deploy-ordered.md).

Cet article explique comment activer une clé gérée par le client pour votre commande Data Box existante dans le [portail Azure](https://portal.azure.com/). Vous y découvrirez comment modifier le coffre de clés, la clé, la version ou l’identité de votre clé gérée par le client actuelle, ou rebasculer vers une clé gérée par Microsoft.

Cet article s’applique aux appareils Azure Data Box et Azure Data Box Heavy.

## <a name="requirements"></a>Spécifications

Dans le cadre d’une commande Data Box, la clé gérée par le client doit respecter les conditions suivantes :

- La clé doit être créée et stockée dans une instance Azure Key Vault pour laquelle les fonctionnalités **Suppression réversible** et **Ne pas vider** sont activées. Pour plus d’informations, consultez la page [Qu’est-ce qu’Azure Key Vault ?](../key-vault/general/overview.md) Vous pouvez créer un coffre de clés et une clé lors de la création ou de la mise à jour de votre commande.

- La clé doit correspondre à une clé RSA d’une taille de 2048 ou supérieure.

## <a name="enable-key"></a>Activer la clé

Afin d’activer un clé gérée par le client pour votre commande Data Box existante dans le portail Azure, procédez comme suit :

1. Accédez à l’écran **Vue d’ensemble** de votre commande Data Box.

    ![Écran Vue d’ensemble d’une commande Data Box - 1](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-1.png)

2. Accédez à **Paramètres > Chiffrement**, puis sélectionnez **Clé gérée par le client**. Sélectionnez ensuite **Sélectionner une clé et un coffre de clés**.

    ![Sélectionner l’option de chiffrement de clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-3.png)

   Dans l’écran **Sélectionner une clé dans Azure Key Vault**, votre abonnement est automatiquement renseigné.

 3. Pour **Coffre de clés**, vous pouvez sélectionner un coffre de clés existant dans la liste déroulante ou sélectionner **Créer nouveau** pour créer un nouveau coffre de clés.

     ![Options de coffre de clés lors de la sélection d’une clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-3-a.png)

     Pour créer un coffre de clés, entrez l’abonnement, le groupe de ressources, le nom du coffre de clés et autres informations dans l’écran **Créer un coffre de clés**. Dans **Options de récupération**, assurez-vous que les options **Suppression réversible** et **Protection contre le vidage** sont activées. Sélectionnez ensuite **Vérifier + créer**.

      ![Examiner et créer un Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-4.png)

      Passez en revue les informations correspondant à votre coffre de clés, puis sélectionnez **Créer**. Patientez quelques minutes avant la fin de la création du coffre de clés.

       ![Créer un Azure Key Vault avec vos paramètres](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-5.png)

4. Dans l’écran **Sélectionner une clé dans Azure Key Vault**, vous pouvez sélectionner une clé existante dans le coffre de clés ou en créer une nouvelle.

    ![Sélectionner une clé dans Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-6.png)

   Si vous souhaiter créer une nouvelle clé, sélectionnez **Créer**. Vous devez utiliser une clé RSA. La taille peut être supérieure ou égale à 2048.

    ![Créer une nouvelle clé dans Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-6-a.png)

    Entrez un nom pour votre nouvelle clé, acceptez les autres valeurs par défaut, puis sélectionnez **Créer**. Vous serez informé qu’une clé a été créée dans votre coffre de clés.

    ![Nommer la nouvelle clé](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-7.png)

5. Pour **Version**, vous pouvez sélectionner un version de clé existante dans la liste déroulante.

    ![Sélectionner la version pour la nouvelle clé](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-8.png)

    Si vous souhaitez générer une nouvelle version de clé, sélectionnez **Créer**.

    ![Ouvrir une boîte de dialogue permettant de créer une version de clé](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-8-a.png)

    Choisissez les paramètres de la nouvelle version de clé, puis sélectionnez **Créer**.

    ![Créer une nouvelle version de la clé](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-8-b.png)

6. Après avoir sélectionné un coffre de clés, une clé et une version de clé, choisissez **Sélectionner**.

    ![Clé d’une instance Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-8-c.png)

    Les paramètres **Type de chiffrement** affichent le coffre de clés et la clé que vous avez choisis.

    ![Clé et coffre de clés pour une clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-9.png)

7. Sélectionnez le type d’identité à utiliser pour gérer la clé gérée par le client pour cette ressource. Vous pouvez utiliser l’identité **attribuée par le système** générée lors de la création de la commande ou choisir une identité attribuée par l’utilisateur.

    Une identité attribuée par l’utilisateur est une ressource indépendante que vous pouvez utiliser pour gérer l’accès aux ressources. Pour plus d’informations, consultez [Types d’identités managées](/azure/active-directory/managed-identities-azure-resources/overview).

    ![Sélectionner le type d’identité](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-13.png)

    Pour attribuer une identité d’utilisateur, sélectionnez **Attribué par l’utilisateur**. Sélectionnez ensuite **Sélectionner une identité d’utilisateur**, puis l’identité managée que vous souhaitez utiliser.

    ![Sélectionner une identité à utiliser](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-14.png)

    Vous ne pouvez pas créer d’identité d’utilisateur ici. Pour savoir comment procéder, consultez [Créer, répertorier, supprimer ou affecter un rôle à une identité managée attribuée par l’utilisateur à l’aide du portail Azure](/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal).

    L’identité d’utilisateur sélectionnée s’affiche dans les paramètres **Type de chiffrement**.

    ![Identité d’utilisateur sélectionnée dans les paramètres Type de chiffrement](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-15.png)

 9. Sélectionnez **Enregistrer** pour enregistrer les paramètres **Type de chiffrement** mis à jour.

     ![Enregistrer votre clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-10.png)

    L’URL de la clé est affichée sous **Type de chiffrement**.

    ![URL de la clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-11.png)<!--Probably need new screen from recent order. Can you provide one? I can't create an order using CMK with the subscription I'm using.-->

## <a name="change-key"></a>Modifier la clé

Pour modifier le coffre de clés, la clé et/ou la version de clé pour la clé gérée par le client que vous utilisez actuellement, procédez comme suit :

1. Dans l’écran **Vue d’ensemble** de votre commande Data Box, accédez à **Paramètres** > **Chiffrement**, puis cliquez sur **Modifier la clé**.

    ![Écran Vue d’ensemble d’une commande Data Box avec clé gérée par le client - 1](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-16.png)

2. Choisissez **Sélectionner un autre coffre de clés et une autre clé**.

    ![Écran Vue d’ensemble d’une commande Data Box, option Sélectionner une autre clé et un autre coffre de clés](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-16-a.png)

3. L’écran **Sélectionner une clé dans le coffre de clés** affiche l’abonnement, mais pas de coffre de clés, clé ou version de clé. Vous pouvez procéder aux modifications suivantes :

   - Sélectionnez une autre clé dans le même coffre de clés. Vous devez sélectionner le coffre de clés avant de sélectionner la clé et la version.

   - Sélectionnez un autre coffre de clés et attribuez une nouvelle clé.

   - Modifiez la version de la clé actuelle.
   
    Une fois vos modifications apportées, choisissez **Sélectionner**.

    ![Choisir l’option de chiffrement - 2](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-17.png)

4. Sélectionnez **Enregistrer**.

    ![Enregistrer les paramètres de chiffrement mis à jour - 1](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-17-a.png)

## <a name="change-identity"></a>Modifier l'identité

Pour modifier l’identité utilisée pour gérer l’accès à la clé gérée par le client dans le cadre de cette commande, procédez comme suit :

1. Dans l’écran **Vue d’ensemble** de votre commande Data Box, accédez à **Paramètres** > **Chiffrement**.

2. Apportez l’une des modifications suivantes :

     - Pour modifier l’identité d’un autre utilisateur, cliquez sur **Sélectionnez une autre identité d’utilisateur**. Sélectionnez ensuite une autre identité dans le panneau situé à droite de l’écran, puis choisissez **Sélectionner**.

       ![Option de modification de l’identité attribuée par l’utilisateur pour une clé gérée par le client](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-18.png)

   - Pour basculer vers l’identité attribuée par le système générée lors de la création de la commande, sélectionnez **Attribuée par le système** sous **Sélectionner le type d’identité**.

     ![Option permettant de passer à une clé gérée par l’utilisateur attribuée par le système](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-19.png)

3. Sélectionnez **Enregistrer**.

    ![Enregistrer les paramètres de chiffrement mis à jour - 2](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-17-a.png)

## <a name="use-microsoft-managed-key"></a>Utiliser une clé gérée par Microsoft

Pour passer de l’utilisation d’une clé gérée par le client à une clé gérée par Microsoft pour votre commande, procédez comme suit :

1. Dans l’écran **Vue d’ensemble** de votre commande Data Box, accédez à **Paramètres** > **Chiffrement**.

2. Pour **Sélectionner le type**, sélectionnez **Clé gérée par Microsoft**.

    ![Écran Vue d’ensemble d’une commande Data Box - 5](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-20.png)

3. Sélectionnez **Enregistrer**.

    ![Enregistrer les paramètres de chiffrement mis à jour pour une clé gérée par Microsoft](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-21.png)

## <a name="troubleshoot-errors"></a>Résoudre les erreurs

Si vous recevez des erreurs liées à votre clé gérée par le client, utilisez le tableau suivant pour résoudre les problèmes.

| Code d’erreur| Détails de l’erreur| Récupérable ?|
|-------------|--------------|---------|
| SsemUserErrorEncryptionKeyDisabled| Impossible de récupérer la clé d’accès, car la clé gérée par le client est désactivée.| Oui, en activant la version de clé.|
| SsemUserErrorEncryptionKeyExpired| Impossible de récupérer la clé d’accès, car la clé gérée par le client a expiré.| Oui, en activant la version de clé.|
| SsemUserErrorKeyDetailsNotFound| Impossible de récupérer la clé d’accès, car la clé gérée par le client est introuvable.| Si vous avez supprimé le coffre de clés, vous ne pouvez pas récupérer la clé gérée par le client.  Si vous avez migré le coffre de clés vers un autre locataire, consultez [Modifier l’ID de client du coffre de clés après un déplacement d’abonnement](../key-vault/general/move-subscription.md). Si vous avez supprimé le coffre de clés :<ol><li>Oui, s’il s’agit de la durée de la protection contre le vidage, à l’aide des étapes de [Récupérer un coffre de clés](../key-vault/general/soft-delete-powershell.md#recovering-a-key-vault).</li><li>Non au-delà de la durée de la protection contre le vidage.</li></ol><br>Sinon, si le coffre de clés a subi une migration de locataire, oui, il peut être récupéré à l’aide d’une des étapes ci-dessous : <ol><li>Restaurez le coffre de clés sur l’ancien locataire.</li><li>Définissez `Identity = None`, puis redéfinissez la valeur sur `Identity = SystemAssigned`. Cela supprime et recrée l’identité une fois que la nouvelle identité a été créée. Activez les autorisations `Get`, `Wrap` et `Unwrap` sur la nouvelle identité dans la stratégie d’accès du coffre de clés.</li></ol> |
| SsemUserErrorKeyVaultBadRequestException | Une clé gérée par le client a été appliquée, mais l'accès de la clé n'a pas été accordé ou a été révoqué, ou l'accès au coffre de clé est impossible en raison de l'activation du pare-feu. | Ajoutez l'identité sélectionnée à votre coffre de clés pour permettre l'accès à la clé gérée par le client. Si le pare-feu est activé pour le coffre de clés, optez pour une identité attribuée par le système, puis ajoutez une clé gérée par le client. Pour plus d’informations, consultez [Activer la clé](#enable-key). |
| SsemUserErrorKeyVaultDetailsNotFound| Impossible de récupérer la clé d’accès, car le coffre de clés associé à la clé gérée par le client est introuvable. | Si vous avez supprimé le coffre de clés, vous ne pouvez pas récupérer la clé gérée par le client.  Si vous avez migré le coffre de clés vers un autre locataire, consultez [Modifier l’ID de client du coffre de clés après un déplacement d’abonnement](../key-vault/general/move-subscription.md). Si vous avez supprimé le coffre de clés :<ol><li>Oui, s’il s’agit de la durée de la protection contre le vidage, à l’aide des étapes de [Récupérer un coffre de clés](../key-vault/general/soft-delete-powershell.md#recovering-a-key-vault).</li><li>Non au-delà de la durée de la protection contre le vidage.</li></ol><br>Sinon, si le coffre de clés a subi une migration de locataire, oui, il peut être récupéré à l’aide d’une des étapes ci-dessous : <ol><li>Restaurez le coffre de clés sur l’ancien locataire.</li><li>Définissez `Identity = None`, puis redéfinissez la valeur sur `Identity = SystemAssigned`. Cela supprime et recrée l’identité une fois que la nouvelle identité a été créée. Activez les autorisations `Get`, `Wrap` et `Unwrap` sur la nouvelle identité dans la stratégie d’accès du coffre de clés.</li></ol> |
| SsemUserErrorSystemAssignedIdentityAbsent  | Impossible de récupérer la clé d’accès, car la clé gérée par le client est introuvable.| Oui, vérifier si : <ol><li>Le coffre de clés contient toujours le fichier MSI dans la stratégie d’accès.</li><li>L’identité est de type System assigned (Attribuée par le système).</li><li>Activez les autorisations Get, Wrap et Unwrap sur l’identité dans la stratégie d’accès du coffre de clés.</li></ol>|
| SsemUserErrorUserAssignedLimitReached | L'ajout d'une nouvelle identité attribuée par l'utilisateur a échoué, car vous avez atteint la limite d'identités attribuées par l'utilisateur pouvant être ajoutées. | Recommencez l’opération avec moins d’identités d’utilisateur ou supprimez plusieurs identités attribuées par l’utilisateur de la ressources avant de réessayer. |
| SsemUserErrorCrossTenantIdentityAccessForbidden | L'opération d'accès à l'identité managée a échoué. <br> Remarque : cela se produit lorsque l’abonnement est déplacé vers un autre locataire. Le client doit manuellement déplacer l’identité vers le nouveau locataire. E-mail PFA pour plus d’informations. | Déplacez l’identité sélectionnée vers le nouveau locataire sous lequel l’abonnement est présent. Pour plus d’informations, consultez [Activer la clé](#enable-key). |
| SsemUserErrorKekUserIdentityNotFound | Une clé gérée par le client a été appliquée, mais l'identité affectée par l'utilisateur qui a accès à la clé est introuvable dans Active Directory. <br> Remarque : cela se produit lorsque l’identité d’utilisateur est supprimée d’Azure.| Essayez d'ajouter une autre identité attribuée par l'utilisateur à votre coffre de clés pour permettre l'accès à la clé gérée par le client. Pour plus d’informations, consultez [Activer la clé](#enable-key). |
| SsemUserErrorUserAssignedIdentityAbsent | Impossible de récupérer la clé d’accès, car la clé gérée par le client est introuvable. | Impossible d’accéder à la clé gérée par le client. L’identité attribuée par l’utilisateur (UAI) associée à la clé est supprimée ou le type UAI a changé. |
| SsemUserErrorCrossTenantIdentityAccessForbidden | L'opération d'accès à l'identité managée a échoué. <br> Remarque : cela se produit lorsque l’abonnement est déplacé vers un autre locataire. Le client doit manuellement déplacer l’identité vers le nouveau locataire. E-mail PFA pour plus d’informations. | Essayez d'ajouter une autre identité attribuée par l'utilisateur à votre coffre de clés pour permettre l'accès à la clé gérée par le client. Pour plus d’informations, consultez [Activer la clé](#enable-key).|
| SsemUserErrorKeyVaultBadRequestException | Une clé gérée par le client a été appliquée, mais l'accès de la clé n'a pas été accordé ou a été révoqué, ou l'accès au coffre de clé est impossible en raison de l'activation du pare-feu. | Ajoutez l'identité sélectionnée à votre coffre de clés pour permettre l'accès à la clé gérée par le client. Si le pare-feu est activé pour le coffre de clés, optez pour une identité attribuée par le système, puis ajoutez une clé gérée par le client. Pour plus d’informations, consultez [Activer la clé](#enable-key). |
| Erreur générique  | Impossible de récupérer la clé d’accès.| Il s'agit d'une erreur générique. Contactez le Support Microsoft pour résoudre l’erreur et déterminer les étapes suivantes.|

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’Azure Key Vault ?](../key-vault/general/overview.md)
- [Démarrage rapide : Définir et récupérer un secret depuis Azure Key Vault à l’aide du portail Azure](../key-vault/secrets/quick-create-portal.md)
