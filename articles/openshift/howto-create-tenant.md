---
title: Créer un locataire Azure AD pour Azure Red Hat OpenShift
description: Voici comment créer un locataire Azure Active Directory (Azure AD) pour héberger votre cluster Microsoft Azure Red Hat OpenShift.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: b57bafdc3f4748bf634955334424941b55e02fd8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492367"
---
# <a name="create-an-azure-ad-tenant-for-azure-red-hat-openshift"></a>Créer un locataire Azure AD pour Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Microsoft Azure Red Hat OpenShift nécessite un locataire [Azure Active Directory (Azure AD)](../active-directory/develop/quickstart-create-new-tenant.md) pour créer votre cluster. Un *locataire* est une instance Azure AD dédiée reçue par une organisation ou un développeur d’applications lorsque l’un ou l’autre crée une relation avec Microsoft en s’inscrivant à Azure, Microsoft Intune ou Microsoft 365. Chaque locataire Azure AD est distinct et séparé des autres locataires Azure AD. Il dispose de ses propres identités professionnelles et scolaires et de ses propres inscriptions d’application.

Si vous ne disposez pas d’un locataire Azure AD, suivez ces instructions pour en créer un.

## <a name="create-a-new-azure-ad-tenant"></a>Créer un nouveau locataire Azure AD

Pour créer un locataire :

1. Connectez-vous au [Portail Azure](https://portal.azure.com/)en utilisant le compte que vous souhaitez associer à votre cluster Azure Red Hat OpenShift.
2. Ouvrez le [panneau Azure Active Directory](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory) pour créer un locataire (aussi connu en tant que nouveau *Azure Active Directory*).
3. Fournissez un **nom d’organisation**.
4. Fournissez un **nom de domaine initial**. *onmicrosoft.com* lui est adjoint. Ici, vous pouvez reprendre la valeur de *Nom de l’organisation*.
5. Choisissez un pays ou une région où le locataire sera créé.
6. Cliquez sur **Créer**.
7. Après avoir créé votre locataire Azure AD, cliquez sur le lien **Cliquez ici pour gérer votre nouveau répertoire**. Le nom de votre nouveau locataire doit figurer dans le coin supérieur droit du Portail Azure :  

    ![Capture d’écran du portail montrant le nom du locataire en haut à droite][tenantcallout]  

8. Prenez note de l’*ID locataire* afin de pouvoir spécifier ultérieurement où créer votre cluster Azure Red Hat OpenShift. Dans le portail, vous devez maintenant voir le panneau de vue d’ensemble d’Azure Active Directory pour votre nouveau locataire. Sélectionnez **Propriétés**, puis copiez la valeur de votre **ID de répertoire**. Nous nous référerons à cette valeur en tant que `TENANT` dans le didacticiel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

[tenantcallout]: ./media/howto-create-tenant/tenant-callout.png

## <a name="resources"></a>Ressources

Consultez la [documentation Azure Active Directory](../active-directory/index.yml) pour plus d’informations sur les [locataires Azure AD](../active-directory/develop/quickstart-create-new-tenant.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer un principal de service, générer une URL de rappel de clé secrète client et d’authentification, et créer un utilisateur Active Directory pour tester des applications sur votre cluster Azure Red Hat OpenShift.

[Créer un utilisateur et un objet d’application Azure AD](howto-aad-app-configuration.md)