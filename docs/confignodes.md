A ConfigNode is exactly that, a node within/of a configuration. ConfigNodes typically consist of the node itself, the curly boys `{}` and the few, or many, keys and their values within.
For example you have a part config. This is what a layout will resemble. Note that a lot of it is omitted to not needlessly fill this page. Also, comments, for the un-initiated, are denoted by leading `//`. There is no trailing symbol such as `}` that goes with `{`. A comment ends at the EOL (end of line) so each line of comments needs a leading `//`.

```
// This is a comment. Everything after // on a line is ignored by KSP. We'll
// use comments to explain ConfigNodes.

// This is a top-level node. Note the name with curly brackets afterward.
PART
{
    // This is the name value. It's just a standard name/value pair, but any of these named 'name'
    // is treated a bit special by MM
    name = myPart

    // This is another value. Note name = value . You can't have a name without a value,
    // but you can have a value without a name (treated as the empty string for name )
    module = Part

    // This is a node named 'MODULE'. It's a collection of further values and modules
    MODULE
    {
        name = ModuleEngines
    }
}

// This is a patch. Note the prefix - this will be one of  '@' for edit, '+' or '$' for copy,
// '-' or '!' for delete, '%' to edit-or-create. A patch takes some pre-existing top-level node and modifies it
@PART[myPart]
{
    // Edit the value with name 'module'
    @module = PartEnhanced

    // Delete all 'MODULE' nodes.
    -MODULE,* { }
}
```
```
PART // the node, also a root node
{
	name = mk2CrewCabin // a key and value
	module = Part // another key and value
	author = Porkjet // yeah... when pigs fly. oh wait...
	title = MK2 Crew Cabin
	manufacturer = C7 Aerospace Division
	INTERNAL // a node, subnode, not root
	{
		name = MK2_CrewCab_Int
	}
	MODULE // another node, subode
	{
		name = ModuleScienceContainer
		reviewActionName = Review Stored Data
		storeActionName = Store Experiments
		evaOnlyStorage = True
		storageRange = 2.0
	}
}
```

## Operations

A ModuleManager operation is the user-facing and easiest form of action that makes up a KSP mod. From the tiniest thing such as making the stock RTG produce more ElectricCharge to having a tank or structural object present dozens of options in some form to the player, there is always an MM operation involved. And an operation always consists of some combination of the following operators.

### Common Operators
**Prefix operators** are the items that go onto a node or key to tell MM that this object needs to be operated upon, and how.
  * ~~nothing~~, creates a new node.
  * `@` to edit.
  * `%` to edit if it exists or create if it does not exist.
  * `+` or `$` to copy.
  * `-` or `!` to delete.
  
When deleting a key, the operation must take the form of `!keyname = value`. A value needs to be passed (it can be anything at all but it needs to be something) to satisfy the parser's expectation of a key-value structure. When deleting a node, the operation must take the form of `!NODE {}`. Empty curly braces need to be included to satisfy the parser's expectation of a node structure.

MM speaks the universal language of ~~English~~ Math, however, in a basic form, and does not support writings in the forms of quadratics, polynomials and so on. A complex operaion needs to be broken down and written as a `<key> <operator> <value>` setup for each step of the overall operation. If you want to find and declare the volume of a cylinder in MM then your config must take this form:

```
volume = 1 // The formula is Pi*r^2*h
@volume *= 5 // r
@volume ^= 2 // r squared
@volume *= 9 // height
@volume *= 3.142 // pi

// as seen in MM database, the ModuleManager.ConfigCache file
volume = 706.95
```

**Value operators** replace the normal `=` character between name and value and tell MM how you want to change the value. The ones given here are all the Mathematical operators and do not work for strings. Those are explained elsewhere.
  * `+=` Add
  * `-=` Subtract
  * `*=` Multiply
  * `/=` Divide
  * `^=` Raise by n Power

**Filter operators** are prefix operators for keys and nodes, and boolean operators. They are used in the **:HAS** block whose purpose is to restrain the targeting of nodes by what keys or subnodes they do or don't contain, and what values these keys do or don't have.

  * `:HAS[<query>]` for seaching only configs that contain nodes or keys that match the query.
  * `*` for any number of alphanumeric characters.
  * `?` for a single alphanumeric character. This can also be used on a space or special characters.
  * `@` for including nodes.
  * `-` or `!` for excluding nodes.
  * `#` for including keys.
  * `~` for excluding keys.
  * `:NEEDS[<modname>]` for applying a patch only when a given mod is present.
  * `&` or `,` for **AND**
  * `|` for **OR**

Filter operations are used quite extensively in part mods and can do such things as:
* `@PART:HAS[#manufacturer[Jebediah*],@MODULE[ModuleEngines*]]` Single out all parts that have an engine module and a certain manufacturer.
* `@PART:HAS[@MODULE[nameA]:HAS[#keyB[valueC]]]` Single out all parts that have a certain module A with a certain key B with a certain value C

Note that the **OR** boolean is not supported within **:HAS**. Any patch that uses the **HAS** filter and needs to meet this condition needs to be written in the form of 2 or more patches (as necessary) where each possible condition is true. 

This does not work:
```
@PART:HAS[@MODULE[function1]|@MODULE[function3]]
{
	@MODULE[function4]
	{
		@keyA = new value
	}
}
```
But this will work:
```
@PART:HAS[@MODULE[function1]]
{
	@MODULE[function4]
	{
		%keyA = new value
	}
}
@PART:HAS[@MODULE[function3]]
{
	@MODULE[function4]
	{
		%keyA = new value
	}
}
```

When **OR** is/can be used, this is how it behaves: A pair that is bound by **OR** is bound more tightly than (or treated as a single object before) any combination bound by **AND**. It is only known to be used in mod detection, the NEEDS block.
* `:NEEDS[A|B,C|D]` becomes **(A or B) and (C or D)**.
* `:NEEDS[A|B|C,D]` becomes **(A or B or C) and D**.

