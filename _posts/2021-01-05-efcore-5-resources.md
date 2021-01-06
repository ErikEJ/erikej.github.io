---
layout: post
title:  "Entity Framework Core 5 free resources"
date:   2021-01-05 18:28:49 +0100
categories: efcore
---

It is now almost two months since EF Core 5 was launched at the [.NET Conf 2020](https://www.dotnetconf.net/) online event on November 10, 2020.

In the meantime, an update to both .NET 5 and an updated [version 5.0.1](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) of EF Core has been released.

Below I will point you to some of the online resources that cover new features and introductions to EF Core 5, so you can get a good start with this version of EF Core.

# Documentation, blog posts, articles

For a **good overview** of the release, read the [blog post on the .NET blog](https://devblogs.microsoft.com/dotnet/announcing-the-release-of-ef-core-5-0/) by Program Manager - .NET Data Jeremy Likness.

For a very long list of all the **new features** in EF Core 5, you can scroll through the [documentation here](https://docs.microsoft.com/ef/core/what-is-new/ef-core-5.0/whatsnew?WT.mc_id=DT-MVP-4025156).

To get a list of the relatively few **breaking changes**, you can find them well [documented here](https://docs.microsoft.com/ef/core/what-is-new/ef-core-5.0/breaking-changes?WT.mc_id=DT-MVP-4025156).

If you use **third party extensions** or tools with EF Core, have a look at the [list here](https://docs.microsoft.com/ef/core/extensions?WT.mc_id=DT-MVP-4025156) to check if they have been updated for EF Core 5.

Finally, Julie Lerman, top authority on everything EF Core, has published an **article for CODE Magazine**, [EF Core 5: Building on the Foundation](https://www.codemag.com/Article/2010042/EF-Core-5-Building-on-the-Foundation), where she walks you through some of the new, major features in EF Core 5. 

# Videos

The EF Core team has also produced a number of videos, that highlight one or more EF Core 5 features.

## .NET Conf 2020

[Entity Framework Core 5.0: The Next Generation for Data Access](https://youtu.be/BIImyq8qaD4) - Jeremy and Shay from the EF Core team gives a brief introduction to EF Core in general, and demonstrates latest features in action like many-to-many, table-per-type and filtered includes.

[Get a Head Start with Entity Framework Core 5.0 with EF Core Power Tools](https://youtu.be/uph-AGyOd8c) - You have an existing database, and would really like to take advantage of EF Core 5.0, but you are not familiar with the dotnet command line and the EF Core commands. See how the "EF Core Power Tools" for Visual Studio 2019 comes to your rescue!

## Entity Framework Community Standup

[EF Core 5.0 Demo Extravaganza](https://youtu.be/5Oow3LlFjTQ) - Arthur from the EF Core team shows a lot of demos!

[Many-to-Many in EF Core 5.0](https://youtu.be/W1sxepfIMRM) - Join the team as they explore the latest many-to-many mapping features implemented for EF Core 5.0 including skip navigations and more!

[What's New with Migrations in EF Core 5.0](https://youtu.be/mSsGERmrhnE) - The Entity Framework Core team focused on major improvements to migrations for the EF Core 5.0 release. Learn about what's new and explore different migrations scenarios in this demo-heavy session, headed by Brice from the EF Core team.

[EF Core 5.0 Collations](https://youtu.be/OgMhLVa_VfA) - In this community standup, we'll be showing new features around case sensitivity and collations in 5.0. We'll also provide a glimpse into how these features were designed, and what considerations and constraints guide the EF team - performance, cross-database support, usability and more. Come see how we design EF features under the hood!

[Special EF Core 5.0 Community Panel](https://youtu.be/AkqRn2vr1lc) - In this special edition of the EF Core Community Standup, we celebrate the release of EF Core 5.0 with a community panel. We'll welcome Entity Framework luminaries Diego Vega, Erik E Jensen, Jon P Smith and Julie Lerman to discuss their favorite features and answer your questions live.

## On .NET Live

[Modern Entity Framework: A Tour of EF Core 5.0 pt 1](https://youtu.be/p0UJdoBj-Lc) - EF Core 5.0 includes support for many-to-many relationships and TPT mapping, two sorely missed features from EF6. Join us for a whirlwind tour where we compare EF Core 5.0 features with those from classic EF6. In this episode, Jeremy chats with his teammate Arthur Vickers about some of the new and very useful features that are available in Entity Framework Core 5.

[Deep Dive into Many-to-Many: A Tour of EF Core 5.0 pt. 2](https://youtu.be/b2klBzcALJc) - In this episode, Arthur Vickers returns to chat some more with Jeremy about some of the new  features in Entity Framework Core 5. In particular, they’ll be diving deep into the building blocks of many-to-many. You’ll see how they can be configured to allow flexible mapping and access to the underlying join table.

Hope you find these resources useful, and let me know if you think I missed anything.

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/24)