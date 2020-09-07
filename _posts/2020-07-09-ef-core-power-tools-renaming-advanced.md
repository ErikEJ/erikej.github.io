---
layout: post
title:  "EF Core Power Tools database reverse engineering: renaming of entities and properties"
date:   2020-09-07 20:28:49 +0100
categories: efcore
---

[Entity Framework Core Power Tools](https://github.com/ErikEJ/EFCorePowerTools) (my free, open source Visual Studio extension, that helps you be more productive with EF Core), includes a feature to rename entities and properties.

This can be useful in several cases, for example if you have "legacy" table and columns names in your database, and would like to be able to use more readable names in your code.

The implementation of this feature required changes to EF Core, but thanks to the open nature of the EF Core project, I got a [PR accepted](https://github.com/dotnet/efcore/pull/11207) to enable exactly that feature.

The feature was implemented with great help from [GitHub user tomzre](https://github.com/tomzre) as you can [see here](https://github.com/ErikEJ/EFCorePowerTools/issues/14). 

### Using the renaming feature

To implement renaming of entities and properties, add a .json file named efpt.renaming.json at the root of your project.

Given the following schema:

```sql
CREATE TABLE dbo.course_sched
(
    ident INT PRIMARY KEY NOT NULL,
    subj_area_cd VARCHAR(30) NOT NULL
)
```
the efpt.renaming.json sample file below will generate these names in the model:

```csharp
public virtual DbSet<CourseOffering> CourseOffering { get; set; }
```
and CourseOffering.cs with SubjectArea and Id string properties.

```json
[
  {
    "UseSchemaName": false,
    "SchemaName": "dbo",
    "Tables": [
      {
        "Name": "course_sched",
        "NewName":  "CourseOffering",
        "Columns": [
          {
            "Name": "subj_area_cd",
            "NewName": "SubjectArea"
          },
          {
            "Name": "identifier",
            "NewName": "Id"
          },
        ]
      }
    ]
  }
]
```

You can also use this to have custom naming of tables with same name in different schemas, [as in this example](https://github.com/ErikEJ/EFCorePowerTools/issues/488#issuecomment-685447772).

In order to do mass re-namings, you can create your own to to populate a class like [this one](https://github.com/ErikEJ/EFCorePowerTools/blob/master/src/GUI/RevEng.Shared/TableRenamer.cs), and the write it to a .json file with code like this:

```csharp
File.WriteAllText("efpt.renaming.json", JsonConvert.SerializeObject(schemas), Encoding.UTF8);
```

> If you manage to create a nice, general purpose tool (maybe simply a command line tool), I will be very happy to receive a PR :-)

[Comments or questions for this blog post?](https://github.com/ErikEJ/erikej.github.io/issues/19)