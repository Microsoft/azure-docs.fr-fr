---
title: Activer le déploiement échelonné des fonctionnalités pour des audiences ciblées
titleSuffix: Azure App Configuration
description: Découvrez comment activer le déploiement échelonné des fonctionnalités pour des audiences ciblées
ms.service: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.topic: conceptual
ms.date: 11/20/2020
ms.openlocfilehash: c415eaeab2edd0a1b324bba4266266201cb50cbf
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929682"
---
# <a name="enable-staged-rollout-of-features-for-targeted-audiences"></a>Activer le déploiement échelonné des fonctionnalités pour des audiences ciblées

Les indicateurs de fonctionnalités vous permettent d’activer ou de désactiver de manière dynamique des fonctionnalités dans votre application. Les filtres de fonctionnalités déterminent l’état d’un indicateur de fonctionnalité chaque fois qu’il est évalué. La bibliothèque `Microsoft.FeatureManagement` comprend `TargetingFilter`, qui permet d’activer un indicateur de fonctionnalité pour une liste spécifiée d’utilisateurs et de groupes ou pour un pourcentage donné d’utilisateurs. `TargetingFilter` est « rémanent ». Cela signifie qu’une fois qu’un utilisateur reçoit une fonctionnalité, il continue à voir cette fonctionnalité sur toutes les demandes ultérieures. Vous pouvez utiliser `TargetingFilter` pour activer une fonctionnalité pour un compte spécifique pendant une version de démonstration, pour déployer progressivement de nouvelles fonctionnalités aux utilisateurs de différents groupes ou « anneaux », et bien plus encore.

Dans cet article, vous allez apprendre à déployer une nouvelle fonctionnalité dans une application web ASP.NET Core aux utilisateurs et groupes spécifiés, en utilisant `TargetingFilter` avec Azure App Configuration.

## <a name="create-a-web-application-with-feature-flags-and-authentication"></a>Créer une application web avec des indicateurs de fonctionnalité et une authentification

Pour déployer des fonctionnalités en fonction des utilisateurs et des groupes, vous avez besoin d’une application web qui permet aux utilisateurs de se connecter.

1. Créez une application web qui s’authentifie auprès d’une base de données locale à l’aide de la commande suivante :

   ```dotnetcli
   dotnet new mvc --auth Individual -o TestFeatureFlags
   ```

1. Générez-la et exécutez-la, puis sélectionnez le lien **S’inscrire** dans le coin supérieur droit pour créer un nouveau compte d’utilisateur. Utilisez une adresse e-mail de `test@contoso.com`. Dans l’écran **Confirmation de l’inscription**, sélectionnez **Cliquez ici pour confirmer votre compte**.

1. Suivez les instructions données dans [Démarrage rapide : Ajouter des indicateurs de fonctionnalités à une application ASP.NET Core](./quickstart-feature-flag-aspnet-core.md) pour ajouter un indicateur de fonctionnalité à votre nouvelle application web.

1. Activez/désactivez l’indicateur de fonctionnalité dans App Configuration. Vérifiez que cette action contrôle la visibilité de l’élément **Bêta** dans la barre de navigation.

## <a name="update-the-web-application-code-to-use-targetingfilter"></a>Mettre à jour le code de l’application web pour utiliser TargetingFilter

À ce stade, vous pouvez utiliser l’indicateur de fonctionnalité pour activer ou désactiver la fonctionnalité `Beta` pour tous les utilisateurs. Pour activer l’indicateur de fonctionnalité pour certains utilisateurs tout en le désactivant pour d’autres, mettez à jour votre code afin d’utiliser `TargetingFilter`. Dans cet exemple, vous allez utiliser l’adresse e-mail de l’utilisateur connecté comme identifiant utilisateur et la partie nom de domaine de l’adresse e-mail comme groupe. Vous ajouterez l’utilisateur et le groupe à `TargetingContext`. `TargetingFilter` utilise ce contexte pour déterminer l’état de l’indicateur de fonctionnalité pour chaque requête.

1. Effectuez une mise à jour vers la version la plus récente du package `Microsoft.FeatureManagement.AspNetCore`.

   ```dotnetcli
   dotnet add package Microsoft.FeatureManagement.AspNetCore
   ```

1. Ajoutez un fichier *TestTargetingContextAccessor.cs* :

    ```csharp
    using Microsoft.AspNetCore.Http;
    using Microsoft.FeatureManagement.FeatureFilters;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;

    namespace TestFeatureFlags
    {
        public class TestTargetingContextAccessor : ITargetingContextAccessor
        {
            private const string TargetingContextLookup = "TestTargetingContextAccessor.TargetingContext";
            private readonly IHttpContextAccessor _httpContextAccessor;

            public TestTargetingContextAccessor(IHttpContextAccessor httpContextAccessor)
            {
                _httpContextAccessor = httpContextAccessor ?? throw new ArgumentNullException(nameof(httpContextAccessor));
            }

            public ValueTask<TargetingContext> GetContextAsync()
            {
                HttpContext httpContext = _httpContextAccessor.HttpContext;
                if (httpContext.Items.TryGetValue(TargetingContextLookup, out object value))
                {
                    return new ValueTask<TargetingContext>((TargetingContext)value);
                }
                List<string> groups = new List<string>();
                if (httpContext.User.Identity.Name != null)
                {
                    groups.Add(httpContext.User.Identity.Name.Split("@", StringSplitOptions.None)[1]);
                }
                TargetingContext targetingContext = new TargetingContext
                {
                    UserId = httpContext.User.Identity.Name,
                    Groups = groups
                };
                httpContext.Items[TargetingContextLookup] = targetingContext;
                return new ValueTask<TargetingContext>(targetingContext);
            }
        }
    }
    ```

1. Dans *Startup.cs*, ajoutez une référence à l’espace de noms *Microsoft.FeatureManagement.FeatureFilters* :

    ```csharp
    using Microsoft.FeatureManagement.FeatureFilters;
    ```

1. Mettez à jour la méthode *ConfigureServices* pour inscrire `TargetingFilter`, suite à l’appel à `AddFeatureManagement()` :

    ```csharp
    services.AddFeatureManagement()
            .AddFeatureFilter<TargetingFilter>();
    ```

1. Mettez à jour la méthode *ConfigureServices* pour ajouter `TestTargetingContextAccessor`, créé à l’étape précédente, à la collection de services. *TargetingFilter* l’utilise pour déterminer le contexte de ciblage chaque fois que l’indicateur de fonctionnalité est évalué.

    ```csharp
      services.AddSingleton<ITargetingContextAccessor, TestTargetingContextAccessor>();
    ```

L’intégralité de la méthode *ConfigureServices* se présente comme suit :

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));
    services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
            .AddEntityFrameworkStores<ApplicationDbContext>();
    services.AddControllersWithViews();
    services.AddRazorPages();

    // Add feature management, targeting filter, and ITargetingContextAccessor to service collection
    services.AddFeatureManagement().AddFeatureFilter<TargetingFilter>();
    services.AddSingleton<ITargetingContextAccessor, TestTargetingContextAccessor>();
    }
```

## <a name="update-the-feature-flag-to-use-targetingfilter"></a>Mettre à jour l’indicateur de fonctionnalité pour utiliser TargetingFilter

1. Sur le portail Azure, accédez à votre magasin App Configuration et sélectionnez **Gestionnaire de fonctionnalités**.

1. Sélectionnez le menu contextuel de l’indicateur de fonctionnalité *Bêta* que vous avez créé dans le démarrage rapide. Sélectionnez **Edit** (Modifier).

    > [!div class="mx-imgBorder"]
    > ![Modifier l'indicateur de fonctionnalité Beta](./media/edit-beta-feature-flag.png)

1. Dans l’écran **Modifier**, cochez la case **Activer l’indicateur de fonctionnalité**, si elle n’est pas déjà sélectionnée. Cochez ensuite la case **Utiliser le filtre de fonctionnalités**.

1. Sélectionnez la case d’option **Ciblage**.

1. Sélectionnez les options suivantes :

    - **Pourcentage par défaut** : 0
    - **Groupes** : Entrez le **nom** _contoso.com_ et un **pourcentage** de _50_
    - **Utilisateurs** : `test@contoso.com`

    L’écran de filtrage des fonctionnalités se présente comme suit :

    > [!div class="mx-imgBorder"]
    > ![Indicateur de fonctionnalité conditionnel](./media/feature-flag-filter-enabled.png)

    Ces paramètres entraînent le comportement suivant :

    - L’indicateur de fonctionnalité est toujours activé pour l’utilisateur `test@contoso.com`, car `test@contoso.com` est listé dans la section _Utilisateurs_.
    - L’indicateur de fonctionnalité est activé pour 50 % des autres utilisateurs dans le groupe _contoso.com_, car _contoso.com_ est listé dans la section _Groupes_ avec un _pourcentage_ de _50_.
    - La fonctionnalité est toujours désactivée pour tous les autres utilisateurs, car le _pourcentage par défaut_ est défini sur _0_.

1. Sélectionnez **Appliquer** pour enregistrer ces paramètres et revenir à l’écran du **gestionnaire de fonctionnalités**.

1. Le **filtre de fonctionnalités** pour l’indicateur de fonctionnalité s’affiche maintenant comme *Ciblage*. Cet état indique que l’indicateur de fonctionnalité sera activé ou désactivé à la demande, en fonction des critères appliqués par le filtre de fonctionnalités *Ciblage*.

## <a name="targetingfilter-in-action"></a>TargetingFilter en action

Pour voir les effets de cet indicateur de fonctionnalité, générez et exécutez l’application. Au départ, l’élément *Bêta* n’apparaît pas dans la barre d’outils, car l’option _Pourcentage par défaut_ est définie sur 0.

Connectez-vous à présent en tant que `test@contoso.com`, en utilisant le mot de passe que vous avez défini lors de l’inscription. L’élément *Bêta* apparaît désormais dans la barre d’outils, car `test@contoso.com` est spécifié en tant qu’utilisateur ciblé.

La vidéo suivante illustre ce comportement.

> [!div class="mx-imgBorder"]
> ![TargetingFilter en action](./media/feature-flags-targetingfilter.gif)

Vous pouvez créer des utilisateurs supplémentaires avec des adresses e-mail `@contoso.com` pour voir le comportement des paramètres de groupe. La moitié de ces utilisateurs verront l’élément *Bêta*. L’autre moitié ne verra pas l’élément *Bêta*.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vue d’ensemble de la gestion des fonctionnalités](./concept-feature-management.md)