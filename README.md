# XTL - Xrm Templating Language [![Build status](https://ci.appveyor.com/api/projects/status/skqv53ykh62587qp?svg=true)](https://ci.appveyor.com/project/DigitalFlow/xrm-templating-language)

|Line Coverage|Branch Coverage|
|-----|-----------------|
|[![Line coverage](https://cdn.rawgit.com/digitalflow/xrm-templating-language/master/reports/badge_linecoverage.svg)](https://cdn.rawgit.com/digitalflow/xrm-templating-language/master/reports/index.htm)|[![Branch coverage](https://cdn.rawgit.com/digitalflow/xrm-templating-language/master/reports/badge_branchcoverage.svg)](https://cdn.rawgit.com/digitalflow/xrm-templating-language/master/reports/index.htm)|

A domain specific language for Dynamics CRM allowing for easy text template processing

## Purpose
XTL is a domain specific language created for easing text processing inside Dynamics CRM.
It is an interpreted programming language, with easy syntax for allowing everyone to use it.

The parsing and interpreting is done using a custom recursive descent parser implemented in C#.
It is embedded inside a plugin and does not need any external references, so that execution works in CRM online and on-premises environments.

## Where to get it
You can always download the latest release from the [releases page](https://github.com/DigitalFlow/Xrm-Templating-Language/releases).
Beware that the release solutions only target CRM >= v9.0 currently.
The code supports CRM v8.0 as well, I just don't have a development organization for creating the solutions there.
This will be done shortly.

As alternative:
Build it yourself by running `build.cmd`, or simply download from [AppVeyor](https://ci.appveyor.com/project/DigitalFlow/xrm-templating-language/build/artifacts).

## Requirements
XTL itself does not use any specific CRM features and is compatible with Dynamics CRM 2011 and higher.
Currently the Plugin is built against Dynamics 365 SDK however. Future releases may target specific CRM versions.
The template editor is only available in CRM 2016 and later, as it uses the Web Api.
The solution which can be downloaded from the releases site currently targets Dynamics 365 aka 9.0.
This is due to the fact that I don't have access to other CRM versions for developing right now.
For XTL itself you only need the DLL which you can download on AppVeyor. The solution just adds the template editor.

## Examples
Examples of how to use XTL can be found in our [Wiki](https://github.com/DigitalFlow/Xrm-Templating-Language/wiki).

## How To Register
### Using XTL Editor
Inside the solution you imported, you'll find that there is a configuration page.
You can use this configuration page for testing templates as well as managing existing ones.
So creating of new template handlers inside your organization can be done completely using the editor.
It creates SDK message processing steps in the background, applying your custom settings combined with the needed default configs.

Impression:
![xtleditor](https://user-images.githubusercontent.com/4287938/38904112-b887b490-42a8-11e8-8b0b-1ccce115728e.png)

### Manual Way
Register the assemblies using the Plugin Registration Tool.
You can then create steps in the pre operation stage. If you're on update message, be sure to register a preimg with all attributes needed for generating your texts.

You'll need an unsecure json configuration per step.

Properties in there:
- target: The target field for the generated text
- templateField: The source field for the template (for "per record" templates, for example if replacing place holders inside emails)
- template: A constant template that will be used for all records (for example when formatting addresses)
- executionCriteria: An XTL expression that should return true if the template should be applied, or false otherwise. If not set, default is to apply the logic. Comparison Operators such as IsEqual automatically return booleans.

Target always has to be set, in addition to either template or templateField.

Sample (For formatting emails):
```JSON
{
    "target": "description",
    "templateField": "description",
    "executionCriteria": "IsEqual(Value(\"directioncode\"), true)"
}
```

## Benefits
When dealing with the default e-mail editor of Dynamics CRM, the borders of what's possible are reached fast.
XTL aims to integrate flawlessly into Dynamics CRM, to enhance the text processing capabilities. It is not limited to any specific CRM entity.
Using XTL provides the following benefits:

- Using of the primary entity's related entity values, no matter how "far" they are away (i.e. regardingobject.parentaccountid.ownerid.fullname for using the full name of the owner of the company that the contact receiving an email belongs to).
- Using of child entity values, where the primary record does not hold the lookup (i.e. all tasks associated to an account)
- Conditional Branching (If-Then-Else constructs)
- Generating Record URLs for all records reachable using above expressions
- Easy to learn syntax (I admit that there are currently quite a few brackets needed for complex expressions)

## General Information
- Make sure that your functions only return strings (i.e. string constants) or values generated by Value() at the top level.
- XTL placeholders are identified using ${{placeholder}}  patterns currently. Please take care that your placeholder does not contain any }} patterns, as this would break the placeholder.
- Be careful when pasting text into records from this page or other formatted content. The text will contain format instructions, which break the placeholders. It is advised to copy the examples in here to NotePad++ or similar before inserting in e-mail HTML editor.

## Types
Native Types:
- String Constants (Alpha numeric text inside quotes)
- Integers (Digit expression)
- Booleans (true or false)
- Functions (Identifiers without quotes followed by parenthesis with parameters)

In addition, null is also a reserved keyword.

## Functions
### If
Takes 3 Parameters: A condition to check, a true action and a false action.
If the condition resolves to true, the true action is executed, otherwise the false action.

Example:
```
If( IsNull ( Value("subject") ), "No subject passed", Value("subject") )
```

### Or
Takes 2 ... n parameters and checks if any of them resolves to true.
If yes, then true is returned, otherwise false

Example:
```
Or ( IsNull( Value ("parentaccountid") ), IsNull( Value ("parentcontactid") ) )  
```

### And
Takes 2 ... n parameters and checks if all of them resolve to true.
If yes, then true is returned, otherwise false

Example:
```
And ( IsNull( Value ("parentaccountid") ), IsNull( Value ("parentcontactid") ) )  
```

### Not
Takes a parameter that resolves to a boolean and reverts its value.

Example:
```
Not ( IsNull( Value ("parentaccountid") ) )  
```

### IsNull
Checks if a parameter is null. If yes, true is returned, otherwise false.

Example:
```
IsNull( Value ("parentaccountid") )  
```

### IsEqual
Checks if two parameters are equal. If yes, true is returned, otherwise false.
The parameters are required to be of the same type.
OptionSetValues are compared with their respective integer values.

Example:
```
IsEqual ( Value ( "gendercode" ), 1 )
```

### Value
Returns the object value of the given property. If it is used as top level function, the matching string representation is returned.
The text function which was present in early releases was replaced by this, as the Value function hosts both the string representation and the actual value now.
You can jump to entities that are connected by a lookup by concatenating field names with a '.' as separator.
Default base entity for all operations is the primary entity. You can override this behaviour by passing an entity object as second parameter.

Example:
```
Value ("regardingobjectid.firstname")
```

### RecordUrl
Returns the record urls of all entities or entity references that are passed as paramters.
For this to work, you have to set a secure json configuration with property organizationUrl set to your organization's url.
 
Example:
```
RecordUrl ( Value ( "regardingobjectid") )
```

### Fetch
Returns a list of records, which were retrieved using supplied query.
First parameter is the fetch xml. You can optionally pass a list of further parameters, such as entity references, entities or option set values as references.
Inside the fetch you can then reference them by {0}, {1}, ... and so on.
The primary entity is always {0}, so additional references start at {1}.
 
Example:
```
Fetch ( "<fetch no-lock='true'><entity name='task'><attribute name='description' /><attribute name='subject' /><filter><condition attribute='regardingobjectid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) )
```

If there is the possibility of one of the references being null, you'll have to wrap the fetch inside an if-clause that only executes the fetch, if your reference is not null. Otherwise the function will fail and replace the whole placeholder with an empty string.

Example(Please note that in recent releases (>= v1.0.31) the Text function is removed. You can replace it by the value function):
![email_table](https://user-images.githubusercontent.com/4287938/36945291-e5173fa0-1fab-11e8-86e6-3007eac254c9.gif)
Source Text:
```
${{If(IsNull(Value("regardingobjectid" ) ), "No tasks", RecordTable ( Fetch ( "<fetch no-lock='true'><entity name='task'><attribute name='description' /><attribute name='subject' /><filter><condition attribute='regardingobjectid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) ), "task", true, "subject", "description"))}}
```

### First
Receives a list and returns the first object found in it.

Example:
```
First ( Fetch ( "<fetch no-lock='true'><entity name='task'><attribute name='description' /><attribute name='subject' /><filter><condition attribute='regardingobjectid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) ) )
```

### Last
Receives a list and returns the last object found in it.

Example:
```
Last ( Fetch ( "<fetch no-lock='true'><entity name='task'><attribute name='description' /><attribute name='subject' /><filter><condition attribute='regardingobjectid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) ) )
```

### RecordTable
Returns a table of records with specified columns and record url if wanted.
First parameter is a list of records, for displaying inside the table. Consider retrieving them using the Fetch function.
Second is the name of the sub entity, third whether to include URL or not.
All subsequent parameters are treated as columns to retrieve and will be added in the same order.

Below you can find an example, which executes on e-mail creation and searches for tasks associated to the mails regarding contact.
It will then print the task subject and description, including an url, into the mail.
 
Example:
```
RecordTable ( Fetch ( "<fetch no-lock='true'><entity name='task'><attribute name='description' /><attribute name='subject' /><filter><condition attribute='regardingobjectid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) ), "task", true, "subject", "description")
```

### PrimaryRecord
Returns the current primary entity as Entity object.
No parameters are needed.

Example:
```
PrimaryRecord ()
```

### Concat
Concatenates all parameters that were passed into one string.

Example:
```
Concat(Value("lastname"), ", ", Value("firstname"))
```
Above example could return something like 'Baggins, Frodo'.

### Substring
Takes the substring of your input starting from a given index. Length of substring can be passed optionally as well.

Example:
```
Substring(Value("firstname"), 1, 2 )
```
Above example returns 'ro' when input is 'Frodo'.

### Replace
Replaces text in an input string using your pattern and replacement regexes.
You can use the whole .NET regex syntax for your pattern and replacement regex.

Example:
```
Replace(Value("firstname"), "o", "a" )
```
Above example returns 'Frada' when input is 'Frodo'.

## Sample
Consider the following e-mail template content:
```
Hello ${{Value("regardingobjectid.parentcustomerid.ownerid.firstname")}},

a new case was associated with your account ${{Value("regardingobjectid.parentcustomerid.name")}}, you can open it using the following URL:
${{RecordUrl(Value("regardingobjectid"))}} 
```

When creating an e-mail from above template or even just an outgoing e-mail with the template content as text it will resolve to:
```
Hello Frodo,

a new case was associated with your account TheShire Limited, you can open it using the following URL:
https://imagine-creative-url.local
```

## Template Editor
The solution that you can find inside the releases section also contains a live editor that can be used for creating, testing and serializing configurations for XTL. In addition to that, existing XTL plugin execution steps can be loaded, tested and updated.
You just need to import the xtl solution, open it and select the configuration page.
Inside this page, you can select which entity you want your template to be executed on, which record it should use for previews and set execution criteria and your template. When selecting to load an existing plugin execution step, this information is parsed from the existing step.
After clicking "Preview", the template will be processed and you'll be presented the result and the interpreter trace for debugging purposes.

The example below targets an E-mail, which has an opportunity as regarding object:
![xtl_template_editor](https://user-images.githubusercontent.com/4287938/37870319-d9e516dc-2fca-11e8-9e78-6c52d842d535.gif)

Source snippet of the template being used:
```
Hi ${{Value("regardingobjectid.parentaccountid.ownerid")}},

${{Value("regardingobjectid.createdby")}} created a new opportunity for your account ${{Value("regardingobjectid.parentaccountid.name")}}.

You can open it using the following link: ${{RecordUrl(Value("regardingobjectid"))}}.

Below products have been configured:
${{RecordTable ( Fetch ( "<fetch no-lock='true' ><entity name='opportunityproduct' ><attribute name='productdescription' /><attribute name='priceperunit' /><attribute name='quantity' /><attribute name='extendedamount' /><filter type='and' ><condition attribute='opportunityid' operator='eq' value='{1}' /></filter></entity></fetch>", Value ( "regardingobjectid" ) ), "opportunityproduct", true, "productdescription", "quantity", "priceperunit", "extendedamount")}}

In case any questions are left, please give ${{Value("regardingobjectid.createdby")}} a call: ${{Value("regardingobjectid.createdby.homephone")}}.

This E-Mail was automatically generated by the system.
```

## License
Licensed using the MIT license, enjoy!

## Credits
I learnt writing the interpreter using the [Let's Build a Compiler Tutorial](https://compilers.iecc.com/crenshaw/) by Jack Crenshaw.
Although being quite old, the technics are still the same and the advice is invaluable, it's a great resource for learning.
