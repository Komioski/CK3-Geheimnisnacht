Coding Style Guideline 
======================

Intro
-----
This guideline is being created with the goal to standardise the coding style throughout the mod as a whole, increasing both readability and ease of future modification. *This is a work in progress and currently nothing in this document is to be considered the final iteration.*

**Important Note:** Please avoid full file tidying when you are changing other things within a file, as this can make identifying errors much more difficult.

That said, let's begin.

General Rules for Maintaining Ease of Patch Updating
----------------------------------------------------
If you are working with overwriting something from vanilla, outside of four exceptions we shall get to shortly, the Jomini Engine will allow you to simply overwrite the key by adding it to any file in the correct folder within the mod's hierarchy. 
These exceptions are as follows:
- Events. Events will not overwrite by key, unfortunately, I presume this is something to do with them technically being nested under a namespace function in the memory. Either way, you must overwrite the entire file. Unfortunate, but true.
- Religions. Both Faiths and Tenents/Doctrines are unable to overwrite by key - once again, probably because they're nested children to Religions.
- GUI. This I don't have an explanation for, it just doesn't work that way.
This functionally means that unless you are working with three prior mentioned you should never have to add a vanilla script file to the mod. If you are in doubt, you can ask a higher ranking (Warlock Engineer, or Grey Seer) developer.
Avoiding adding vanilla files allows for much higher ease in maintaining parity on a patch level, preventing the horrendous situation we faced trying to get the mod to boot in the past, and allows for energy to be focused primarily on continued development rather than simply keeping up with CK3's base changes.

On the matter of GUI, if you are adding a GUI file from vanilla CK3, please place a comment that reads `#CfV` (This stands for Changed from Vanilla) before and after your changes/additions. This helps prevent your work from being overwritten in any update, and helps to mitigate avoidable conflicts when integrating.

You may have noticed that I mentioned four exceptions, but thus far have only spoken of three (Religion, Events, GUI) of them. The fourth is Localization, in which overwriting by key has a specific method. To overwrite a vanilla localization key, you MUST place your .yaml file within the english/replace/ folder. Outside of the folder, it shall simply register it as a clashing key, and throw an error up in the log.


General Code Structure
-----------------------
### Curly Braces
Your opening brace `{` should always be on the same line as the code it is associated with, as in the following example:
```
this_is_an_example = {
```
If your code is a simple trigger, effect, or statement, rather than a more complex (or multiple), you should keep close the brace on the same line as in the following example:
```
this_is_an_example = { example_trigger = yes }
```
Otherwise, it should be on its own line, for readability purposes, as follows:
```
this_is_an_example = {
	example_trigger = yes
	example_trigger_two = yes
}
```

### Indenting and Aligning Code
Indentation should always be done according to these four rules, to allow for easier changes in future, and for much better readability for anyone that has to work with your code in future.

**Rule One:** Each opening brace `{` increases the line indent by 1, unless followed by a single line of code.

**Rule Two:** Each closing brace `}` decreases the line indent by 1, unless following a single line of code.

**Rule Three:** Line Indentation begins at 0

**Rule Four:** Indentation should always be done using Tabs rather than Spaces, ask someone if you are unsure as to how to set this up using your text editor.

### Logic Operators

#### If/Else_If/Else vs Switch

First I shall introduce you to the switch operator, because I have noted a lack of people using it, and instead relying on a long series of if/else_if/else statements.
The switch operator is functionally a more performant if/else_if series, with the fallback entry functioning as the else. It can only take simple assigns, which limits it somewhat, but it should still be robust enough for use in most cases.

A simple assign is one that can be reduced to A = B. This includes, and primarily consists of, your boolean triggers. has_trait, culture, and so on. If you simply use yes/no, it shall attempt to evaluate the following cases as a boolean, so this is useful for checking for solely the presence of a flag, for example. This can also be used to check simplistic variables, such as checking if var:temporary_signature_weapon results in `flag:axe`, `flag:hammer`, or `flag:spear`.

```
switch = {
	trigger = simple_assign 

	case_1 = { action_1 = yes }
	case_2 = { action_2 = yes }
	fallback = { action_3 = yes }
}
```

From this, you may have deduced when to use a switch. Namely, in all cases that you have more than two simple assign conditions that or of the same kind (has_trait for example). If/Else_If are very powerful operators, due to their ability to take complex assigns, but can be performance heavy in a series unless optimised perfectly.

#### Negating Logic Operators
For which negatory logic operator to use, please consult the following table:

| **Operator**	| **When to Use**                                                         |
|---------------|------------------------------------------------------------------------|
| NOT			| Use when you are only negating a single trigger, such as in the following `NOT = { prestige = 0 }.|
| NOR			| Use when you wish to list multiple triggers, in which only one must be negated to count.			|
| NAND			| Use when you wish to list multiple triggers, in which all must be negated to count.				|


Encoding
--------

## .yml Files
All .yml files must be saved in UTF8-BOM encoding. To make this much easier on yourself, as near all text editors will open a UTF8-BOM file *in* UTF8-BOM, you can copy and paste another .yml file, rename it, and change the interior localisation, whilst maintaining the desired encoding.

Optimisation
------------

## Triggers
CK3, much like every Paradox Game, relies on ticks, or calculations that your CPU has to go through before it can advance a day. For this guide, these matter predominantly through their generation from Checks,.

As an absolute rule, put your most exclusive check first. For an explanation as to why, contine reading.

The goal with optimising triggers is to reduce these ticks as much as possible, and thus reduce the amount of impact on performance that our code has on the base game. The easiest way to do this is through the order of our checks, for an example let us assume that there is one player, 10 000 characters, and some random value of prestige for each of those characters.

```
trigger = {
	is_ai = yes
	prestige = 0
	this = character:dame_100
	piety = 0
}
```
We have four checks in this event, and we start with 10 000 eligible characters. We have one player, so we remove it from the eligible pool, and go to the next check, with 9 999 eligible characters. Let us assume that around half have more than 0 prestige, and so we go onto the next check with about 5 000 eligible characters. From all these eligible characters, we are looking for character 790117, so the list of eligible characters drops to 1. This one character is then subject to another check, as to if they have more than 0 piety.

So four checks, generating 25 001 ticks. Whereas, if we rewrite this code to, instead, resemble this:

```
trigger = {
	this = character:dame_100
	prestige = 0
	piety = 0
	is_ai = yes
}
```

Here we go from having 10 000 eligible characters, to having 1 eligible character in a single check. Which then compounds with only having one tick per check following, as it must *only* check that one character. Hence we go from 25 001 ticks, to only 10 004 ticks, for the same result. Which is a rather massive improvement of 60% performance wise, for the same result, with the same amount of checks.

Naming conventions
------------------
| **What**                              | **Convention**                                                         | **Examples**                            |
|---------------------------------------|------------------------------------------------------------------------|-----------------------------------------|
| Scripted Triggers                     | Start with WH_ and end with _trigger                                   | `WH_is_vampire_trigger`   		       |
| Scripted Effects                      | Start with WH_ and end with _effect                                    | `WH_add_too_much_prowess_effect`         |
| Event File Names                      | Event Files should be titled WH_(Event_File_Name) to prevent conflict  | `WH_artifacts_events.txt`               |
| Event Namespaces                      | Event Namespaces should either be WH_(Namespace) or GH(Namespace)      | `WH_artifacts`, `WHfancyshit`              |
| Localisation File Names               | Localisation Files MUST be saved in the format (Name)l_<language>.yml  |  `dialog_WH_l_english.yml`              |
