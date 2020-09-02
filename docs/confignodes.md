A ConfigNode is exactly that, a node within/of a configuration. ConfigNodes typically consist of the node itself, the curly boys `{}` and the few, or many, keys and their values within.
For example you have a part config. This is what a layout will resemble. Note that a lot of it is omitted to not needlessly fill this page.
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

### Common Syntax
* Operators:
  * ~~nothing~~, creates a new node.
  * `@` to edit.
  * `%` to edit if it exists or create if it does not exist.
  * `+` or `$` to copy.
  * `-` or `!` to delete.
  
Whn deleting a key, the operation must take the form of `!keyname = value`. A value needs to be passed (it can be anything at all but it needs to be something) to satisfy the parser's expectation of a key-value structure. When deleting a node, the operation must take the form of `!NODE {}`. Empty curly braces need to be included to satisfy the parser's expectation of a node structure.

* Filters:
  * `*` for any number of alphanumeric characters.
  * `?` for a single calphanumeric character. This can also be used on a space or special characters.
  * `@` for including nodes.
  * `-` or `!` for excluding nodes.
  * `#` for including keys.
  * `~` for excluding keys.
  * `:HAS[<query>]` for seaching only configs that contain nodes or keys that match the query.
  * `:NEEDS[<modname>]` for applying a patch only when a given mod is present.
  * `&` or `,` for **AND**
  * `|` for **OR**
  
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

When **OR** is/can be used, this is how it behaves: A pair that is bound by **OR** is bound more tightly than (or treated as a single object before) any combination bound by **AND**.
* `:NEEDS[A|B,C|D]` becomes **(A or B) and (C or D)**.
* `:NEEDS[A|B|C,D]` becomes **(A or B or C) and D**.

### Maths

MM speaks the universal language of ~~English~~ Math, however, in a basic form, and does not support writings in the forms of quadratics, polynomials and so on. A complex operaion needs to be broken down and written as a `<key> <operator> <value>` setup for each step of the overall operation:

* Operators
  * `+=` Add
  * `-=` Subtract
  * `*=` Multiply
  * `/=` Divide
  * `^=` Raise by n Power