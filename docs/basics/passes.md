## NEEDS
Needs, not necessarily a pass but the conditioner for patches. If a particular patch's needs are not met then the patch is not executed. This is the most visible aspect of the power of ModuleManager.

```
@ConfigNode[this]:NEEDS[mod1,mod2|mod3] { 
	// actions
}

@ConfigNode[this]:NEEDS[mod1/subdir] { 
	// actions
}
``` 
NEEDS can accept any number of values (mod names, directories/subdirectories) and can do logic gates. How the logic gates work is: requisites bound by OR are treated with more priority than ( treated as a single object before) any combination bound by **AND**. The following examples apply:
* `:NEEDS[A|B,C|D]` becomes **(A or B) and (C or D)**.
* `:NEEDS[A|B|C,D]` becomes **(A or B or C) and D**.
* `:NEEDS[mod1,mod2/subfolder]` becomes **This mod and this subfolder of this other mod**.

The ability to specify subfolders becomes important where a single mod maker may group all of his or her mods under one root level folder in GameData. Prime examples of such mods or mod suites are Umbra Space Industries, Wild Blue Industries, and SpaceTux Recycled Parts.
A mod not made by RoverDude, the owner of USI, but which requires Karbonite would need to do the following to validate the presence of Karbonite before running its patch:
` @ConfigNode:NEEDS[UmbraSpaceIndustries/Karbonite]`

NEEDS can be applied to each and every key,not just to nodes. This comes in handy for such cases as baked-in tech tree detection for feature-rich parts (engines, experiments...)
```
PART:NEEDS[myMod/Engines]
{
	name = thedevice
	category = Engine
	TechRequired = aviation
	%TechRequired:NEEDS[CommunityTechTree] = subsonicFlight
	%TechRequired:NEEDS[AngleCanMods/TETRIXTechTree] = advancedEngineering
}
```

## BEFORE and AFTER

The `:BEFORE` and `:AFTER` passes inherit from `:NEEDS` but they only accept __one value__ at a time. These simply say: "If this needed mod is present, run this patch before or after that needed mod." If you specify more than one and try to do logic gates then your operation is invalid. MM will likely only ackowledge the first value given or give a negative response and tell you that you're doing madness.

## FOR

The `:FOR` pass accepts only one value. It represents a mod and its position in the sequence of patch processing. It is incredibly powerful and can easily give you or your fanbase (if you are a popular mod maker) a very bad day. The FOR pass does two things:
* It declares that the given mod is present in the KSP install.
* It sets a patch to run within the sequence of the declared mod regardless of where it physically is within GameData.

> :warning: Big warning. Several patches have been observed that contain `:NEEDS[x]:FOR[y]`. When FOR is given, NEEDS is ignored.


Where the FOR pass becomes a nightmare is if you use this for a mod that you do not own. Some mods still contain RemoteTech patches that contain `:FOR[RemoteTech]`. These patches announce that RemoteTech is present regardless of whether it's actually installed, and while it is not installed (by players who are not fans of RemoteTech) related patches in other mods will be executed and bad things will happen such as all antenna parts targeted by these other patches will get stripped of their stock modules, making them useless and causing all unmanned vessels using them to be uncontrollable. Nobody wants this kind of thing to happen.

Where the FOR pass shines brightly is if you need to create a pseudo-mod/placeholder mod/virtual mod so that certain patches within a mod you own can be bunched together into their own sequence and executed before or after the patches that run normally as part of your actual mod. If your mod, **MyTestMod** is one where it needs to change itself and behave differently if another mod is present, then to do so it must run patches before it runs its patches. ;) Such patches would need to have and use these setups in the necessary ways:
* `@ConfigNode[this]:FOR[MyTestCondition]` where **MyTestCondition** alphanumerically preceeds **MyTestMod**.
* `@ConfigNode[this]:NEEDS[MyTestCondition]`

A prime example of this is the mod "Rational Resources". This mod sets resource placements and B9 tank definitions. Any patch that requires it would need to have `:NEEDS[RationalResources]`. It contains an optional patch containing `:FOR[RationalResourcesSquad]` which converts all stock tanks to use B9 Part Switch. The actions of **RationalResourcesSquad** all require its parent mod to be present and to have its parents' actions executed beforehand.

## LEGACY

The default, general pass. This is where all patches are executed where the **pass operators**: NEEDS, FOR, FIRST, BEFORE, FOR, AFTER, LAST, and FINAL are not given. Patches are run recursively and alphanumerically from the top level of GameData downward. Patches that sit directly in GameData and don't have a pass operator get to run the earliest in the LEGACY pass.

All of the MM passes (execution phases) are (in this order): FIRST, LEGACY, BEFORE, FOR, AFTER, LAST, and FINAL. 

## FINAL and LAST

The `:FINAL` pass is exactly that. It's a separate pass where every patch that's intended to execute at the very end of MM's operations are executed. Unfortunately this pass has been abused to hell and back due to mass lack of knowledge of MM, and causes a lot of trouble for mods ahead in time. FINAL is okay to use for personal patches for testing, and is okay to be released/published only by mods with immense scale or impact such as Kopernicus (the one mod by which all other planet mods work), but is not okay to be released by every other mod. Always assume that someone might make a mod that will want to patch the things that your mod patches, after your mod does. In many cases this actually happens, and it causes a huge gridlock in extreme cases where a popular mod with ARR license is abandoned and players want to change something after that mod.

The `:LAST` pass is semifinal. It occurs after LEGACY, before FINAL, and is teated like the FOR pass. LAST accepts one value, a mod name. Where many mods avoid explicitly using `:FINAL` and instead do `:FOR[zzzzMyMod]` (with lots of Z's) the mod makers can opt instead to do:
* `:FOR[zMyMod]` with just one Z (How many actual regular mods have names that start with Z? ....Practically none.)
* `:LAST[MyMod]`. Note that the LAST operation only works when the mod's presence is prior acknowledged by MM.

Exercise caution even when using LAST, as otherwise the same abuse of FINAL can happen all over again here.


