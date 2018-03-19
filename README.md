[![Build Status](http://peeeq.de/hudson/job/Frentity/badge/icon)](http://peeeq.de/hudson/job/Frentity/) 
# Frentity
*short fort Frotty's Entity System*

Frentity is a lightweight entity framework which I use as basis for all my maps.
Its main feature are the Entity base classes for Units, Effects and Items, that allow positional and velocital manipulation via 3d vector math.
Additionally it contains some neat wrapper and bootstrap features that make map development more rapid and integrated.

What at first started as just a collection of common snippets has now grown into a fully featured third party library that provides a different, more limited but therefore also simplified workflow.

# Core Features

These features define the basis of the framework and are most frequently used.

## Event Listening

The `EventListener` package provides easy and comfortable listener API for wc3 events.
Example:
```
EventListener.add(someUnit, EVENT_PLAYER_UNIT_DEATH) ->
	print(someUnit.getName())
```

Due to the use of closures, variables can be retained and the callbacks saved, in case you want to stop them manually.
Listeners for specific units are removed once the unit leaves the map.

## Entity base

Unlike modern Entity-Component-Systems, Frentity takes the  monolithic hierarchical approach to encapsulate functionality. Since wc3 are typically smaller and more cohesive, this concept performs fine, as long as the hierarchy stays shallow.

The idea is to encapsulate anything that moves or interacts with other objects inside an entity. Then in your code you use Entities instead of wc3 handles for the most part, which ensures same behaviour in the gameworld.

```
class TestUnitEntity extends UnitEntity

	construct(vec3 pos)
		super(createUnitZ(players[0], 'hfoo', pos, angle(0)), pos)
```

Make sure to run `startEntityLoop()` in your code from `EntityManagement.wurst` to start entity handling by the system.

## FTexts

There are several issues with wc3's default texttag handling, like recycling the oldest texttag when the maximum of 100 is reached.
The `FText` package wraps texttags into FTexts provided via 
```
createFText(...)
createCenterFText(...)
```

`createCenterFText` attempts to center the texttag in default camera distance. You can combine FTexts with Entities by using `TextTagEntity.wurst` which allows dynamic movement and referencing.

With a simple snippet like this
```
doPeriodically(1/32) cb ->
	new ColorChangeTextEntity(t.getPos() - vec3(-64, 0, 0))
    
class ColorChangeTextEntity extends TextTagEntity
	var color = colorA(235,235,125,255)

	construct(vec3 pos)
		super(pos, vec3(GetRandomReal(-6,6), GetRandomReal(-6,6), GetRandomReal(8,14)), "Ðelta", GetRandomReal(4, 12), GetRandomReal(3, 5), color)

	override function update()
		super.update()
		if not done
			if not flying
				addVel(vec3(0,0,10))
				flying = true
			color = color.mix(colorA(226, 100, 4, 255), 0.025)
			ftext.tt.setColor(color)
```
You can easily create the following effect

![](https://media.giphy.com/media/1o1oeeUn3HP64iCIuV/giphy.gif)

Notice the color change, terrain-aware bouncing and the permanent *Item Shop* texttag staying even though the limit was surpassed. Of course you can fully customize this to your needs.

## Buffs

The `Buff.wurst` and related packages provide support for a lazily created list of normal or stacking buffs per entity. Creating dummy buffs via tornado has meanwhile been merged into the standard library, and thus should be the way for you to create Buffs.

Here is an example of a Buff based on ShieldBuff, that prevents a certain amount of damage on the target:

```
public buffTuple buffAbil = compiletime(createDummyBuffObject("Shield Kit", "This unit is protected by a shield that absorbs damage", 
			"BTNDivineIntervention.blp", Abilities.divineShieldTarget))
            
class ShieldKitBuff extends ShieldBuff
	vec3 pos

	construct(real amount)
		super(90., buffAbil, amount)
		Log.debug("ShieldKit created, dur: " + duration.toString() + " amount: " + amount.toString())

	override function apply(UnitEntity ue) 
		super.apply(ue)
		pos = ue.getPos()

	override function defenseModifier()
		super.defenseModifier()
		if blockAmount > 0
			new TextTagEntity(pos, vec3(GetRandomReal(-2, 2),0, GetRandomReal(8,12)), blockAmount.toInt().toString() , 10, 1., colorA(255, 204, 12, 255))

	override function onEnd()
		if duration == 90.
			new TextTagEntity(pos, vec3(0,0, GetRandomReal(9,12)), "Shield applied" , 13, 1.55, colorA(255, 204, 12, 255))
		else
			new TextTagEntity(pos, vec3(0,0, GetRandomReal(9,12)), "Shield collapsed" , 13, 1.55, colorA(255, 204, 12, 255))
```

We can now apply this buff to any unit for the desired effect. If the spell is applied to an already shielded unit, the block amount will be added to the existing shield.
```
new ShieldKitBuff(400.).apply(data)
```


These projects were made with Frentity:
- https://github.com/Frotty/ForestDef
- https://github.com/Frotty/EBR

