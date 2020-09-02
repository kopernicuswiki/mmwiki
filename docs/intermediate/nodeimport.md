Node/Key Import (not the official name) is an immensely powerful and yet unknown feature of ModuleManager. This is the ability to call the value of a key or the contents of a node and all its subnodes without having to explicitly copy-paste it all.

## Import Commands
There are a few ways and contexts for this. Note whether the end of the calling string or segment ends with a `$` (meaning you want a key's value) or `{}` meaning you want a node and all its keys.

### Same level
A same-level (or higher level, that is, reaching upward within the node hierarchy) call may not seem useful but it is. It can reach anywhere within the same root node (for example: the individual `PART{}` for parts or the individual `Body{}` for Kopernicus planets.) Note the use of `../` which, for none-too-savvy, denotes "go up one level" which occurs in such examples as Windows Explorer and Mac OS Finder.

The keys to consider:
* `#NODE/subnode {}`
* `#../NODE/subnode {}`
* `#$key$`
* `#$NODE/key$`
* `#$../NODE/key$`


### Relative top level
The use of `/` immediately after # or #$ tells ModuleManager to treat as the top level, the node given in the MM pass (such as each `PART` targetted by `@PART:HAS[NODE]:NEEDS[mod248]`).

The keys to consider:
* `#/NODE/subnode {}`
* `#$/NODE/key$`
* `#$/key$`

The following, somewhat intense, config snippet #2 shows how to apply a performance boost via B9 Part Switch to all Kerbodyne engines, and easily import and use values from various places within a string type value (the subtype description for display to the player). The description's line starts with `#` and has a few instances of `$/key` but can also accept `$/NODE/subnode/key`. The actual numbers and words of the keys called will be rendered in-game and can be confirmed and observed in the ConfigCache. A simpler sample is given in snippet #1.

#### Snippet #1 
This sample...
 ```
 PART
 {
	name = this.test
	mass = 3.124
	cost = 6464
	description = #abc $/mass$ def $/cost$ ghi
 }
 ``` 
 
 will translate to 
  ```
 PART
 {
	name = this.test
	mass = 3.124
	cost = 6464
	description = abc 3.124 def 6464 ghi
 }
 ``` 

#### Snippet #2 
```
@PART:HAS[#manufacturer[Kerbodyne],@MODULE[ModuleEnginesFX]]
{
	StandardIspVac = #$MODULE[ModuleEnginesFX]/atmosphereCurve/key,1[0, ]$
	Standard1IspSL = #$MODULE[ModuleEnginesFX]/atmosphereCurve/key,1[1, ]$
	Standard1Thrust = #$MODULE[ModuleEnginesFX]/maxThrust$
	
	Overclock1IspVac = #$MODULE[ModuleEnginesFX]/atmosphereCurve/key,1[0, ]$
	Overclock1IspSL = #$MODULE[ModuleEnginesFX]/atmosphereCurve/key,1[1, ]$
	Overclock1Thrust = #$MODULE[ModuleEnginesFX]/maxThrust$
	
	@Overclock1IspVac *= 1.1
	@Overclock1IspSL *= 1.1
	@Overclock1Thrust *= 1.25
	
	MODULE
	{
		name = ModuleB9PartSwitch
		moduleID = Turbopump
		switcherDescription = Turbopump Mode
		switcherDescriptionPlural = Turbopump Modes
		switchInFlight = True
		SUBTYPE
		{
			name = Standard
			descriptionSummary = #Standard performance.<br><b>Thrust</b>: $/Standard1Thrust$ <br><b>Isp (Vac)</b>: $/StandardIspVac$<br><b>Isp (ASL)</b>: $/Standard1IspSL$
			MODULE
			{
				IDENTIFIER
				{
					name = ModuleEnginesFX
				}
				DATA
				{
					maxThrust = #$/MODULE[ModuleEnginesFX]/maxThrust$
					#/MODULE[ModuleEnginesFX]/atmosphereCurve {}
				}
			}
		}
		SUBTYPE
		{
			name = Overclock1
			descriptionSummary = #Enhanced performance.<br><b>Thrust</b>: $/Overclock1Thrust$ <br><b>Isp (Vac)</b>: $/Overclock1IspVac$<br><b>Isp (ASL)</b>: $/Overclock1IspSL$
			MODULE
			{
				IDENTIFIER
				{
					name = ModuleEnginesFX
				}
				DATA
				{
					maxThrust = #$/Overclock1Thrust$
					#/MODULE[ModuleEnginesFX]/atmosphereCurve {}
					@atmosphereCurve
					{
						@key,*[1, ] *= 1.1
					}
				}
			}
		}
	}
```

### Root node level
The use of `@` immediately after # or #$ tells ModuleManager to treat the NODE as a(nother) root node, to enter it and use its contents in this node. Some root nodes are `Kopernicus{}`, each and every `PART{}`, `EVE_CLOUDS{}` and `Scatterer_planetsList{}`.
* `#@NODE/subnode {}`
* `#$@NODE/key$`

The following config snippet #3 is used by Near Future Propulsion, to easily tune all of its ion engines for your gameplay balance needs, allowing you to make them more powerful in the way(s) you feel appropriate so that you can save on part count (because primarily, ion engines really suck at TWR). This is the technique also used by planet mods such as Galaxies Unbound and Whirligig World to provide a means to toggle certain features within them.

#### Snippet #3
```
NearFutureCustomSettings
{
	nearFuturePowerMultiplier = 1.0
	nearFutureThrustMultiplier = 1.0
	nearFutureIspMultiplier = 1.0
}

@PART[ionEngine,ionArgon-*,ionXenon-*,mpdt-*,pit-*,vasimr-*]:AFTER[NearFuturePropulsion]
{
	@MODULE[ModuleEnginesFX],*
	{
		@maxThrust *= #$@NearFutureCustomSettings/nearFutureThrustMultiplier$
		@PROPELLANT[ElectricCharge]
		{
			@ratio *= #$@NearFutureCustomSettings/nearFuturePowerMultiplier$
			@ratio /= #$@NearFutureCustomSettings/nearFutureThrustMultiplier$
			@ratio *= #$@NearFutureCustomSettings/nearFutureIspMultiplier$
		}
		@atmosphereCurve
		{
			@key,*[1, ] *= #$@NearFutureCustomSettings/nearFutureIspMultiplier$
		}
	}
}
```

The following config snippet #4 is used by SunflaresOfMaar, a scatterer sunflare pack which promised to deliver sunflares of different colors and sizes according to individual stars' types.

```
// Create root node for MM sub-node insertion if scatterer exists
Sunflare_Canaan:NEEDS[SunflaresOfMaar/Canaan]
{
	ghost1SettingsList1
	{
		Item = 0.3,0.56,0.92,0.95
	}
	ghost2SettingsList1
	{
		Item = 0.4,1,36,-0.3
		Item = 0.4,1,48,0.3
		Item = 0.1,1,24,0.4
		Item = 0.15,1,6,0.6
	}
	...
}

// change the sunflare color, size and brightness but import the ghost lists to save on bulk and avoid bulk edits in file
@Scatterer_sunflare:AFTER[scatterer]:NEEDS[SunflaresOfMaar/Canaan]
{
	%author = JadeOfMaar
	@Sun:NEEDS[!KSS|!BeyondHome|!InterstellarConsotium]
	{
		%assetPath = SunflaresOfMaar/Canaan/G
		%flareSettings = 0.6,1,0.8
		%spikesSettings = 0.5,1,0.4
		%sunGlareFadeDistance = 500000
		
		!ghost1SettingsList1 {}
		!ghost2SettingsList1 {}
		...
		
		#@Sunflare_Canaan/ghost1SettingsList1 {}
		#@Sunflare_Canaan/ghost2SettingsList1 {}
		...
	}
	@Cercani:NEEDS[OtherWorldsReboot]
	{
		%assetPath = SunflaresOfMaar/Canaan/K
		%flareSettings = 0.55,1,1.2
		%spikesSettings = 0.45,1,0.6
		%sunGlareFadeDistance = 500000
		
		!ghost1SettingsList1 {}
		!ghost2SettingsList1 {}
		...
		
		#@Sunflare_Canaan/ghost1SettingsList1 {}
		#@Sunflare_Canaan/ghost2SettingsList1 {}
		...
	}
	...
}
```