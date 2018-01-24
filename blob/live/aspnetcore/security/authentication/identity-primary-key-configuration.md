---
title: "Configurer le type de données de clé primaire d’identité"
author: AdrienTorris
description: "Cet article présente les étapes de configuration du type de données utilisé pour la clé primaire ASP.NET Core Identity."
ms.author: scaddie
manager: wpickett
ms.date: 09/28/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authentication/identity-primary-key-configuration
ms.openlocfilehash: da46aa90a434a978a55467da982d746eb2d959ee
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/19/2018
---
# <a name="configure-the-aspnet-core-identity-primary-key-data-type"></a><span data-ttu-id="1e077-103">Configurer le type de données de clé primaire ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="1e077-103">Configure the ASP.NET Core Identity primary key data type</span></span>

<span data-ttu-id="1e077-104">Identité de ASP.NET Core permet de configurer le type de données utilisé pour représenter une clé primaire.</span><span class="sxs-lookup"><span data-stu-id="1e077-104">ASP.NET Core Identity allows you to configure the data type used to represent a primary key.</span></span> <span data-ttu-id="1e077-105">Identité utilise le `string` type de données par défaut.</span><span class="sxs-lookup"><span data-stu-id="1e077-105">Identity uses the `string` data type by default.</span></span> <span data-ttu-id="1e077-106">Vous pouvez substituer ce comportement.</span><span class="sxs-lookup"><span data-stu-id="1e077-106">You can override this behavior.</span></span>

## <a name="customize-the-primary-key-data-type"></a><span data-ttu-id="1e077-107">Personnaliser le type de données de clé primaire</span><span class="sxs-lookup"><span data-stu-id="1e077-107">Customize the primary key data type</span></span>

1. <span data-ttu-id="1e077-108">Créer une implémentation personnalisée de la [IdentityUser](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuser-1) classe.</span><span class="sxs-lookup"><span data-stu-id="1e077-108">Create a custom implementation of the [IdentityUser](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuser-1) class.</span></span> <span data-ttu-id="1e077-109">Il représente le type à utiliser pour la création d’objets utilisateur.</span><span class="sxs-lookup"><span data-stu-id="1e077-109">It represents the type to be used for creating user objects.</span></span> <span data-ttu-id="1e077-110">Dans l’exemple suivant, la valeur par défaut `string` type est remplacé par `Guid`.</span><span class="sxs-lookup"><span data-stu-id="1e077-110">In the following example, the default `string` type is replaced with `Guid`.</span></span>

    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo-PrimaryKeysConfig/Models/ApplicationUser.cs?highlight=4&range=7-13)]

1. <span data-ttu-id="1e077-111">Créer une implémentation personnalisée de la [IdentityRole](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.identity.entityframeworkcore.identityrole-1) classe.</span><span class="sxs-lookup"><span data-stu-id="1e077-111">Create a custom implementation of the [IdentityRole](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.identity.entityframeworkcore.identityrole-1) class.</span></span> <span data-ttu-id="1e077-112">Il représente le type à utiliser pour créer des objets de rôle.</span><span class="sxs-lookup"><span data-stu-id="1e077-112">It represents the type to be used for creating role objects.</span></span> <span data-ttu-id="1e077-113">Dans l’exemple suivant, la valeur par défaut `string` type est remplacé par `Guid`.</span><span class="sxs-lookup"><span data-stu-id="1e077-113">In the following example, the default `string` type is replaced with `Guid`.</span></span>
    
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo-PrimaryKeysConfig/Models/ApplicationRole.cs?highlight=3&range=7-12)]
    
1. <span data-ttu-id="1e077-114">Créer une classe de contexte de base de données personnalisée.</span><span class="sxs-lookup"><span data-stu-id="1e077-114">Create a custom database context class.</span></span> <span data-ttu-id="1e077-115">Il hérite de la classe de contexte de base de données Entity Framework utilisée pour l’identité.</span><span class="sxs-lookup"><span data-stu-id="1e077-115">It inherits from the Entity Framework database context class used for Identity.</span></span> <span data-ttu-id="1e077-116">Le `TUser` et `TRole` arguments de référencent aux classes d’utilisateur et le rôle personnalisés créés à l’étape précédente, respectivement.</span><span class="sxs-lookup"><span data-stu-id="1e077-116">The `TUser` and `TRole` arguments reference the custom user and role classes created in the previous step, respectively.</span></span> <span data-ttu-id="1e077-117">Le `Guid` type de données est défini pour la clé primaire.</span><span class="sxs-lookup"><span data-stu-id="1e077-117">The `Guid` data type is defined for the primary key.</span></span>

    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo-PrimaryKeysConfig/Data/ApplicationDbContext.cs?highlight=3&range=9-26)]
    
1. <span data-ttu-id="1e077-118">Inscrire la classe de contexte de base de données personnalisées lors de l’ajout du service d’identité dans la classe de démarrage de l’application.</span><span class="sxs-lookup"><span data-stu-id="1e077-118">Register the custom database context class when adding the Identity service in the app's startup class.</span></span>

    # <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="1e077-119">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="1e077-119">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)
    
    <span data-ttu-id="1e077-120">Le `AddEntityFrameworkStores` méthode n’accepte pas un `TKey` argument comme il le faisiez dans ASP.NET Core 1.x.</span><span class="sxs-lookup"><span data-stu-id="1e077-120">The `AddEntityFrameworkStores` method doesn't accept a `TKey` argument as it did in ASP.NET Core 1.x.</span></span> <span data-ttu-id="1e077-121">Type de données de la clé primaire est déduit en analysant le `DbContext` objet.</span><span class="sxs-lookup"><span data-stu-id="1e077-121">The primary key's data type is inferred by analyzing the `DbContext` object.</span></span>
    
    [!code-csharp[Main](identity/sample/src/ASPNETv2-IdentityDemo-PrimaryKeysConfig/Startup.cs?highlight=6-8&range=25-37)]
    
    # <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="1e077-122">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="1e077-122">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)
    
    <span data-ttu-id="1e077-123">Le `AddEntityFrameworkStores` méthode accepte un `TKey` argument indiquant le type de données de la clé primaire.</span><span class="sxs-lookup"><span data-stu-id="1e077-123">The `AddEntityFrameworkStores` method accepts a `TKey` argument indicating the primary key's data type.</span></span>
    
    [!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo-PrimaryKeysConfig/Startup.cs?highlight=9-11&range=39-55)]
    
    ---

## <a name="test-the-changes"></a><span data-ttu-id="1e077-124">Tester les modifications</span><span class="sxs-lookup"><span data-stu-id="1e077-124">Test the changes</span></span>

<span data-ttu-id="1e077-125">Une fois les modifications de configuration, la propriété qui représente la clé primaire reflète le nouveau type de données.</span><span class="sxs-lookup"><span data-stu-id="1e077-125">Upon completion of the configuration changes, the property representing the primary key reflects the new data type.</span></span> <span data-ttu-id="1e077-126">L’exemple suivant montre comment accéder à la propriété dans un contrôleur MVC.</span><span class="sxs-lookup"><span data-stu-id="1e077-126">The following example demonstrates accessing the property in an MVC controller.</span></span>

[!code-csharp[Main](identity/sample/src/ASPNET-IdentityDemo-PrimaryKeysConfig/Controllers/AccountController.cs?name=snippet_GetCurrentUserId&highlight=6)]