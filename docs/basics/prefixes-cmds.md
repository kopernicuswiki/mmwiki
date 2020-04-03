<h1>Prefixes and Commands</h1>

ModuleManager works by reading in ConfigNodes, performing specified operations on them, then geiving KSP the modified nodes as if they had been loaded normally. These operations are expressed in a simple (if slightly unintuitive) syntax, consisting of commands (usually referred to as prefixes), indices, and passes. This page will discuss prefixes.

The syntax of a ModuleManager Patch (relating to prefixes at least) is as follows:
```
<prefix>NODE_NAME
{
    <prefix>key_name = value
}
```
The prefix can be any one of the following:

|Prefix|Function|
|------|--------|
|No Prefix|Insert|
|@|Edit|
|%|Edit or Create (if the target exists, edit it, but if not create it)|
|+ or $|Copy|
|- or !|Delete*|

*Note that when deleting a node you must provide an empty set of curly braces.

`!Body[Vall] { }`

When deleting a value, you must still set a value. `DELETE` is the most common.

`!TechRequired = DELETE`