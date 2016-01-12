---
layout:     post
title:      "Ruby - The elevator"
subtitle:   "Designing the Object Oriented Elevator with TDD"
header-img: "img/tag.jpg"
thumb-img:  "img/tag-thumb.jpg"
date:       2015-12-06 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
---


a Class Has a Single Responsibility
Defining the classes
How can you determine if the Gear class contains behavior that belongs somewhere else?
How would you describe the responsibility of the Gear class?
Writing loosely coupled code to remove interdependency between classes.
Represent floors like an array

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring      
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def gear_inches
    ratio * Wheel.new(rim, tire).diameter
  end

# ...
end

Gear.new(52, 11, 26, 1.5).gear_inches
```

instead declare an instance variable wheel that hold
This change is so small it is almost invisible, but coding in this style has huge benefits. Moving the creation of the new Wheel instance outside of Gear decouples the two classes. Gear can now collaborate with any object that implements diameter.

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel   
  def initialize(chainring, cog, wheel)
    @chainring = chainring
    @cog       = cog
    @wheel     = wheel
  end

  def gear_inches
    ratio * wheel.diameter
  end
  # ...
end
      # Gear expects a ‘Duck’ that knows ‘diameter’
Gear.new(52, 11, Wheel.new(26, 1.5)).gear_inches
