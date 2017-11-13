---
layout: post
title: When to rewrite and when to hack code
subtitle: Sometimes your code needs to be rebuilt.  Sometimes your code needs to be hacked with a chainsaw!  Know which to use and when.
category : Intermediate
tags :  
  - API
  - refactor
  - design
---

When a developer is adding code to their project, a common scenario that will occur is a feature needs to *expand the functionality* of a piece of code beyond the scope of the inital implementation when it was first created. There are two ways to address this.

**Hack or Rewrite?**

---

Over a few months, I had hacked a bunch of quick-fixes for a module that permated across my entire project.  It ballooned into nasty code that was a nightmare to maintain and add to.  I gritted my teeth, and ultimately decided I needed to perform some open heart code surgery to get to the root of the problem.  Two weeks later and presto, all finished!  After my rewrite was done, I pondered the 'why' of my actions.  What affected my decision to rewrite this code, and what were the tradeoffs?  

Thus I have determined an outline relating to the issue.

## Why does something need to be rewritten?

Perhaps due to bad code design, maybe just to have a cleaner syntax, or even performance issues, but a developer needs to write down all the reasons for doing so and the perceived benefits obtained from the rewrite.  A big factor to consider - rewritting code can be quite time consuming compared to a quick hack which brings me to my next point.

## What is a hack?  

Quite simply, it is a piece of code that is not a *formal* part of the design.  Often hacks are coded in a quick manner that is (often) sloppy and akin to duct taping something to fix a problem.  A good example would be hardcoding variables, instead of passing them through parameters.  

## When to use one or the other?

* Time should be a important factor
* Code maintainability and readability
* Depth and breadth of the problem 
* Number of developers working on project

### Time

*How long will it take to rewrite the code vs adding a hack?  
How critical is time to you as a developer?  
How much time will this save me in the future?*

If you need to push out a fix for a problem ASAP, a hack will get the job done while time can be spent getting a formal rewrite finished.  Hacks serve as *shortcuts* in contrast to rewrites which are more time consuming.

### Maintence

*How negatively or postively will this affect your codebase?*

Hacks tend to bloat the code in a nasty way that makes it hard to read and maintain.  In this regard, a developer should seriously aim to *minimize* using hacks in their code base.  Use them sparringly is a good rule of thumb.  

Rewrites do just the opposite, it leaves your codebase with a pristine shine and the result is generally easier code use and maintence.

### Scope of the Issue

*Is one module affected?  
How much of that module?  
How big of a module is it?  
If multiple modules are affected, then how many and how deep?*

If the problem is big enough, a rewrite is often neccessary.  But if the issue is limited to a few lines of code in a single module, a hack should suffice.

### Dev Quanitiy  

*What is the number of developers working on the project?*

The more developers, the higher magnitude of issues hacks can cause.  If there is a sole developer, he or she will know what the hacks do, and where they are.  Add another developer into the mix and the 'self-awareness' for the hacks disappears.  Now imagine, *multiple developers*, each with their own unique hacks, that others are not aware of.  **This will magnify confusion tremendously** due to having code that is not a documented part of the design or included as a method.  This can snowball into significant problems if not addressed properly!

---

Hacks generally have a reputation for being quick and dirty.  They also *excel* at testing new things before you invest time into a design that may get scrapped.  They are also great for short term fixes, but sacrifice code readability in most cases.  This is further amplified by the number of developers working on the same project.  

Code rewrites are no angels either!  They are time consuming and can cause some serious bugs!  Instead of using the old code as hacks do, you are literally tearing out the old code and rebuilding it, while trying to incorporate more features on top of it.  If not planned properly, this can break a project. Old code that once worked has been altered (usually) that can cause unexpected behaviour if not accounted for.  

## Conclusion

Both methods serve useful purposes, but with different tradeoffs.  Knowing when to apply one over the other can save you lots of development time if done properly.

## Code Sample

This is the old code that I was using to broadcast events to objects in my game.

{% highlight lua %}
local function event(zone, action, data)
  local z_type, z_tile, z_stage, z_range = zone.type, zone.tile, zone.stage, zone.range  
  local z_player, z_target = zone.player, zone.target
  local date = os.time()

  if z_type == 'self' then
    z_player.log:event(date, action, data, 1)
  elseif z_type == 'pair' then
    z_player.log:event(date, action, data, 1)
    z_target.log:event(date, action, data, 2)
  elseif z_type == 'stage' then
    local players = z_tile:getPlayers(z_stage)
    -- if player destroyed equipment or revived, isolate player for 1st person
    for _, player in pairs(players) do 
      if z_player and z_player == player then player.log:event(date, action, data, 1)
      else player.log:event(date, action, data, 3) 
      end
    end
  elseif z_type == 'tile' then  
    local tile_players = z_tile:getPlayers('all')
    for _, player in pairs(tile_players) do  
--print('z_player is '..z_player, 'player is '..player)
      if z_player and z_player == player then player.log:event(date, action, data, 1)
      else player.log:event(date, action, data, 3) 
      end      
    end
  elseif z_type == 'area' then
    local map = z_tile:getMap()
    local y_pos, x_pos = z_tile:getPos()
    for y = y_pos - z_range, y_pos + z_range do
      for x = x_pos - z_range, x_pos + z_range do
        if map[y] and map[y][x] then
          -- setting up a new localized zone thats params will be overwritten
          -- z_type is now 'tile' for the next broadcastEvent() instead of 'area'
          -- this avoids an infinite broadcastEvent() loop
          local zone = zone
          zone.type, zone.tile = 'tile', map[y][x]
          event(zone, action, data)
        end
      end
    end
  end
end

return event
{% endhighlight %}   

And this is the rewritten new code that has improved functionality!
  
{% highlight lua %}
local broadcastEvent = {}

function broadcastEvent.player(player, msg, self_msg, event)  --this broadcasts specifically to exclude player and give player their own self_msg 
  local tile = player:getTile()
  local stage = player:getStage()

  local players = tile:getPlayers(stage)
  for _, player_INST in pairs(players) do
    if player_INST:isStanding() and player_INST ~= player then 
      -- plug in map[y][x] coords into msg with string.gsub()
      player_INST.log:insert(msg, event)
    end
  end

  player.log:insert(self_msg, event)
end

--settings={stage=inside/outside, range=num, mob_type=zombie/human, exclude={player=true, ...}}
function broadcastEvent.zone(zone, msg, event, setting)
  if zone:isClass('tile') then
    local tile = zone
    local range, stage, mob_type, exclude = setting.range, setting.stage, setting.mob_type, setting.exclude

    -- delivers msg to tile
    local players = tile:getPlayers(stage)
    for _, player_INST in pairs(players) do
      if (not mob_type or player_INST:isMobType(mob_type) ) and player_INST:isStanding() and (not exclude or not exclude[player_INST]) then 
        -- plug in map[y][x] coords into msg with string.gsub()
        player_INST.log:insert(msg, event)
      end
    end        

    if range then -- delivers msg to area of tiles (in the shape of a square that is [range] x sized)
      local map = zone:getMap()
      local y_pos, x_pos = tile:getPos()
      for y = y_pos - range, y_pos + range do
        for x = x_pos - range, x_pos + range do
          if map[y] and map[y][x] and not (y_pos == y and x_pos == x) then -- avoid broadcast on map[y_pos][x_pos] since the msg has already been delievered to that tile 
            broadcastEvent(map[y][x], msg, event, {stage=stage, mob_type=mob_type, exclude=exclude}) 
          end
        end
      end
    end      
  elseif zone:isClass('map') then
    --broadcast msg to all players on that map
  --elseif zone:isType('server') then   broadcast msg to all players on server?
  end
end

return broadcastEvent
{% endhighlight %}  