---
layout: post
title:  "Create a Custom Static Code Analysis Rule for Azure SQL Database / SQL Server with .NET"
date:   2024-04-04 18:28:49 +0100
categories: dacfx dotnet
---

This walkthrough demonstrates the steps used to create a SQL Server Code Analysis rule. The rule created in this walkthrough is used to avoid WAITFOR DELAY statements in stored procedures, triggers, and functions.


In this walkthrough, you will create a custom rule for Transact-SQL static code analysis by using the following processes:  
  
1. Create a class library, enable signing for that project, and add the necessary NuGet package references.  
  
2. Create a Visual C\# custom rule class.  
  
3. Create two helper Visual C\# classes.  
  
4. To use the rule with either Visual Studio or other tools/pipelines, see my previous blog post [here](https://erikej.github.io/dacfx/codeanalysis/sqlserver/2024/04/02/dacfx-codeanalysis.html)

**Prerequisites**
  
You need the following components to complete this walkthrough:  
  
- You must have installed a version of Visual Studio 2022 that includes SQL Server Data Tools from the Data Storage and Processing workload and supports Visual C\# development.
  
- You must have a SQL Server Database project or [MSBuild.SDK.Sqlproj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) project that contains SQL Server objects.  

> This walkthrough is intended for users who are already familiar with the SQL Server features of SQL Server Data Tools. You are also expected to be familiar with Visual Studio concepts, such as how to create a class library, add NuGet packages and how to use the code editor to add code to a class.  
  
## Creating a Custom Code Analysis Rule for SQL Server  

First create a class library. To create a class library project:  
  
1. Create a Visual C\# class library project named SampleRules. Target .NET 8.
  
2. Rename the file Class1.cs to AvoidWaitForDelayRule.cs.  
  
3. In Solution Explorer, right-click the project node and then click **Manage NuGet Packages**. Locate and install the `System.Composition` NuGet package.
  
4. In Solution Explorer, right-click the project node and then click **Manage NuGet Packages**. Locate and install the `Microsoft.SqlServer.DacFx` NuGet package - the selected version must be `162.x.x`(for example `162.2.111`) with Visual Studio 2022.

Next you will add supporting classes that will be used by the rule.  
  
## Creating the Custom Code Analysis Rule Supporting Classes

Before you create the class for the rule itself, you will add a visitor class. This class might be useful for creating additional custom rules.  
  
The class that you must define is the WaitForDelayVisitor class, derived from [TSqlConcreteFragmentVisitor](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom.tsqlconcretefragmentvisitor). This class provides access to the WAITFOR DELAY statements in the model. Visitor classes make use of the [ScriptDom](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom) APIs provided by SQL Server. In this API, Transact-SQL code is represented as an abstract syntax tree (AST) and visitor classes can be useful when you wish to find specific syntax objects such as WAITFORDELAY statements. These might be difficult to find using the object model since they're not associated to a specific object property or relationship, but it's easy to find them using the visitor pattern and the [ScriptDom](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom) API.  
  
### Defining the WaitForDelayVisitor Class  
  
1. In **Solution Explorer**, select the SampleRules project.  
  
2. On the **Project** menu, select **Add** > **Class**. The **Add New Item** dialog box appears.
  
3. In the **Name** text box, type WaitForDelayVisitor.cs and then click the **Add** button. The WaitForDelayVisitor.cs file is added to the project in **Solution Explorer**.  
  
4. Open the WaitForDelayVisitor.cs file and add the following usings to the file:
  
    ```  
    using System.Collections.Generic;  
    using Microsoft.SqlServer.TransactSql.ScriptDom;  
    ```  
  
5. In the class declaration, derive the class from TSqlConcreteFragmentVisitor:  
  
    ```  
    internal class WaitForDelayVisitor : TSqlConcreteFragmentVisitor
    ```  
  
6. Add the following code to define the List member variable:  
  
    ```  
    public IList<WaitForStatement> WaitForDelayStatements { get; private set; }  
    ```  
  
7. Define the class constructor by adding the following code:  
  
    ```  
    public WaitForDelayVisitor() {  
       WaitForDelayStatements = new List<WaitForStatement>();  
    }  
    ```  
  
8. Override the ExplicitVisit method by adding the following code:  
  
    ```  
    public override void ExplicitVisit(WaitForStatement node) {  
       // We are only interested in WAITFOR DELAY occurrences  
       if (node.WaitForOption == WaitForOption.Delay)  
          WaitForDelayStatements.Add(node);  
    }  
    ```  
  
    This method visits the WAITFOR statements in the model, and adds those that have the DELAY option specified to the list of WAITFOR DELAY statements. The key class referenced here is [WaitForStatement](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom.waitforstatement).  
  
9. On the **File** menu, click **Save**.  

## Creating the Custom Code Analysis Rule Class

Now that you have added the helper class that the custom Code Analysis rule will use, you will create a custom rule class and name it AvoidWaitForDelayRule. The AvoidWaitForDelayRule custom rule will be used to help database developers avoid WAITFOR DELAY statements in stored procedures, triggers, and functions.  
  
### Creating the AvoidWaitForDelayRule Class  
  
1. Open the AvoidWaitForDelayRule.cs file and add the following using statements to the file:  
  
    ```  
    using Microsoft.SqlServer.Dac.CodeAnalysis;  
    using Microsoft.SqlServer.Dac.Model;  
    using Microsoft.SqlServer.TransactSql.ScriptDom;  
    using System;  
    using System.Collections.Generic;  
    using System.Globalization; 
    ```  
  
5. In the AvoidWaitForDelayRule class declaration, change the access modifier to public:  
  
    ```  
    /// <summary>  
    /// This is a rule that returns a warning message   
    /// whenever there is a WAITFOR DELAY statement appears inside a subroutine body.  
    /// This rule only applies to stored procedures, functions and triggers.  
    /// </summary>  
    public sealed class AvoidWaitForDelayRule  
    ```  
  
6. Derive the AvoidWaitForDelayRule class from the Microsoft.SqlServer.Dac.CodeAnalysis.SqlCodeAnalysisRule base class:  
  
    ```  
    public sealed class AvoidWaitForDelayRule : SqlCodeAnalysisRule  
    ```  
  
7. Add the ExportCodeAnalysisRuleAttribute to your class.  
  
    ExportCodeAnalysisRuleAttribute allows the code analysis service to discover custom code analysis rules. Only classes marked with an ExportCodeAnalysisRuleAttribute (or an attribute that inherits from this) can be used in code analysis.  
  
    ExportCodeAnalysisRuleAttribute provides some required metadata used by the service. This includes a unique ID for this rule, a display name that will be shown in the Visual Studio user interface, and a Description that can be used by your rule when identifying problems.  

    ```  
     [ExportCodeAnalysisRule(AvoidWaitForDelayRule.RuleId,
        "Avoid using WaitFor Delay statements in stored procedures, functions and triggers.",
        Description = "WAITFOR DELAY statement was found in {0}.",
        Category = "Performance",
        RuleScope = SqlRuleScope.Element)]   
    public sealed class AvoidWaitForDelayRule : SqlCodeAnalysisRule  
    {  
       /// <summary>  
       /// The Rule ID should resemble a fully-qualified class name. In the Visual Studio UI  
       /// rules are grouped by "Namespace + Category", and each rule is shown using "Short ID: DisplayName".  
       /// For this rule, that means the grouping will be "SampleRules.Performance", with the rule  
       /// shown as "SR1004: Avoid using WaitFor Delay statements in stored procedures, functions and triggers."  
       /// </summary>  
       public const string RuleId = "SR1004";  
    }  
    ```  
    The RuleScope property should be Microsoft.SqlServer.Dac.CodeAnalysis.SqlRuleScope.Element as this rule will analyze specific elements. The rule will be called once for each matching element in the model. If you wish to analyze an entire model then Microsoft.SqlServer.Dac.CodeAnalysis.SqlRuleScope.Model can be used instead.  
  
8. Add a constructor that sets up the Microsoft.SqlServer.Dac.CodeAnalysis.SqlAnalysisRule.SupportedElementTypes. This is required for element-scoped rules. It defines the types of elements to which this rule will be applied. In this case, the rule will be applied to stored procedures, triggers, and functions. Note that the Microsoft.SqlServer.Dac.Model.ModelSchema class lists all available element types that can be analyzed.  
  
    ```  
    public AvoidWaitForDelayRule()  
    {  
       // This rule supports Procedures, Functions and Triggers. Only those objects will be passed to the Analyze method  
       SupportedElementTypes = new[]  
       {  
          // Note: can use the ModelSchema definitions, or access the TypeClass for any of these types  
          ModelSchema.ExtendedProcedure,  
          ModelSchema.Procedure,  
          ModelSchema.TableValuedFunction,  
          ModelSchema.ScalarFunction,  
  
          ModelSchema.DatabaseDdlTrigger,  
          ModelSchema.DmlTrigger,  
          ModelSchema.ServerDdlTrigger  
       };  
    }  
    ```  
  
9. Add an override for the Microsoft.SqlServer.Dac.CodeAnalysis.SqlAnalysisRule.Analyze(Microsoft.SqlServer.Dac.CodeAnalysis.SqlRuleExecutionContext) method, which uses Microsoft.SqlServer.Dac.CodeAnalysis.SqlRuleExecutionContext as input parameters. This method returns a list of potential problems.  
  
    The method obtains the Microsoft.SqlServer.Dac.Model.TSqlModel, Microsoft.SqlServer.Dac.Model.TSqlObject, and [TSqlFragment](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom.tsqlfragment) from the context parameter. The WaitForDelayVisitor class is then used to obtain a list of all WAITFOR DELAY statements in the model.  
  
    For each [WaitForStatement](/dotnet/api/microsoft.sqlserver.transactsql.scriptdom.waitforstatement) in that list, a Microsoft.SqlServer.Dac.CodeAnalysis.SqlRuleProblem is created.  
  
    ```  
    /// <summary>  
    /// For element-scoped rules the Analyze method is executed once for every matching   
    /// object in the model.   
    /// </summary>  
    /// <param name="ruleExecutionContext">The context object contains the TSqlObject being   
    /// analyzed, a TSqlFragment  
    /// that's the AST representation of the object, the current rule's descriptor, and a   
    /// reference to the model being  
    /// analyzed.  
    /// </param>  
    /// <returns>A list of problems should be returned. These will be displayed in the Visual   
    /// Studio error list</returns>  
    public override IList<SqlRuleProblem> Analyze(  
        SqlRuleExecutionContext ruleExecutionContext)  
    {  
         List<SqlRuleProblem> problems = new List<SqlRuleProblem>();  
  
         TSqlObject modelElement = ruleExecutionContext.ModelElement;  
  
         // this rule does not apply to inline table-valued function  
         // we simply do not return any problem in that case.  
         if (IsInlineTableValuedFunction(modelElement))  
         {  
             return problems;  
         }  
  
         string elementName = GetElementName(ruleExecutionContext, modelElement);  
  
         // The rule execution context has all the objects we'll need, including the   
         // fragment representing the object,  
         // and a descriptor that lets us access rule metadata  
         TSqlFragment fragment = ruleExecutionContext.ScriptFragment;  
         RuleDescriptor ruleDescriptor = ruleExecutionContext.RuleDescriptor;  
  
         // To process the fragment and identify WAITFOR DELAY statements we will use a   
         // visitor   
         WaitForDelayVisitor visitor = new WaitForDelayVisitor();  
         fragment.Accept(visitor);  
         IList<WaitForStatement> waitforDelayStatements = visitor.WaitForDelayStatements;  
  
         // Create problems for each WAITFOR DELAY statement found   
         // When creating a rule problem, always include the TSqlObject being analyzed. This   
         // is used to determine  
         // the name of the source this problem was found in and a best guess as to the   
         // line/column the problem was found at.  
         //  
         // In addition if you have a specific TSqlFragment that is related to the problem   
         //also include this  
         // since the most accurate source position information (start line and column) will   
         // be read from the fragment  
         foreach (WaitForStatement waitForStatement in waitforDelayStatements)  
         {  
            SqlRuleProblem problem = new SqlRuleProblem(  
                String.Format(CultureInfo.CurrentCulture,   
                    ruleDescriptor.DisplayDescription, elementName),  
                modelElement,  
                waitForStatement);  
            problems.Add(problem);  
        }  
        return problems;  
    }  
  
    private static string GetElementName(  
        SqlRuleExecutionContext ruleExecutionContext,   
        TSqlObject modelElement)  
    {  
        // Get the element name using the built in DisplayServices. This provides a number of   
        // useful formatting options to  
        // make a name user-readable  
        var displayServices = ruleExecutionContext.SchemaModel.DisplayServices;  
        string elementName = displayServices.GetElementName(  
            modelElement, ElementNameStyle.EscapedFullyQualifiedName);  
        return elementName;  
    }  
  
    private static bool IsInlineTableValuedFunction(TSqlObject modelElement)  
    {  
        return TableValuedFunction.TypeClass.Equals(modelElement.ObjectType)  
                       && FunctionType.InlineTableValuedFunction ==   
            modelElement.GetMetadata<FunctionType>(TableValuedFunction.FunctionType);  
    }  
    ```  
  
10. Click **File** > **Save**.

### Add multiple target frameworks

To use the .dll with Visual Studio, you must build a .NET Framework 4.7.2 file, and to use it with .NET cross platform tools, it must be a .NET 8.0 file.

In order to build both .NET and a .NET Framework .dll files, you can update the target framworks in your project file. Open the SampleRules project file (.csproj) and change:

```xml
<TargetFramework>net8.0</TargetFramework>
```
to

```xml
<TargetFrameworks>net8.0;net472</TargetFrameworks>
```

To build for .NET 4.7.2, **remove** these two lines from your .csproj:

```xml
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
```

Your project file should now look like this:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFrameworks>net8.0;net472</TargetFrameworks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.SqlServer.DacFx" Version="162.2.111" />
    <PackageReference Include="System.Composition" Version="8.0.0" />
  </ItemGroup>

</Project>
```

### Building the Class Library  
  
1. On the **Project** menu, click **SampleRules Properties**.  
  
2. Click the **Signing** tab.  
  
3. Click **Sign the assembly**.  
  
4. In **Choose a strong name key file**, click **\<New\>**.  
  
5. In the **Create Strong Name Key** dialog box, in **Key file name**, type MyRefKey.  
   
6. Click **OK**.  
  
7. On the **File** menu, click **Save All**.  
  
8. On the **Build** menu, click **Build Solution**.

The rules SampleRules.dll assemblies will be located in the `bin/Debug/net8.0` and `bin/Debug/net472`folder under your project file.

### Using the assembly files

Next, you must install the assembly so that it will be loaded when you build and deploy SQL Server projects. Refer to my previous blog post [here](https://erikej.github.io/dacfx/codeanalysis/sqlserver/2024/04/02/dacfx-codeanalysis.html).


> This is a simplified and updated version of the walkthrough in the official docs [here](https://learn.microsoft.com/sql/ssdt/walkthrough-author-custom-static-code-analysis-rule-assembly)
