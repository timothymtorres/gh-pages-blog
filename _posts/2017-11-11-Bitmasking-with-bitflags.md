---
layout: post
title: Bitmasking with bitflags
subtitle: The foribidden art of compressing your code into itty-bitty pieces.  Microscope not included!
category : Advanced
tags :  
  - optimize
  - technique
  - network
---

This subject is more advanced and takes time to learn, but as a programmer the knowledge of *bitmasking* is indispensiable.  It is used commonly in networking and performance critical areas of code.  Another benefit is that it takes up little space.  

Before we talk about bitmasking, first we must discuss bitflags.

## What are Bitflags?

A bitflag is a single bit of code in the smallest possible form a `1` or `0`.  One byte of memory is 8 bits, which means eight `1`'s or `0`'s compressed together.  By counting in binary you can set a bitflag to a on/off state.  This is done arthimetically by using a base-2/power of 2/`2^x`.  Regardless of skill level, it is a *neccessary* staple for every programmer to count binary.  If you are not familiar with it, check the [wiki]() about the subject.  

{% highlight lua %}
example_byte =   0 -- bitflag is 0000 0000
example_byte =   1 -- bitflag is 0000 0001 
example_byte =   2 -- bitflag is 0000 0010
example_byte =   4 -- bitflag is 0000 0100
example_byte =   8 -- bitflag is 0000 1000
example_byte =  16 -- bitflag is 0001 0000
example_byte =  32 -- bitflag is 0010 0000
example_byte =  64 -- bitflag is 0100 0000
example_byte = 128 -- bitflag is 1000 0000
{% endhighlight %}  

Since a integer can be a number from 0-255 and contain more than one bitflag, you can combine multiple flags together.  This results in a technique programmers use called **bitmasking**.

---

## How do I Bitmask?

To make code readability easier for this example, I'm going to focus on the 4 bits to the rightest side of a byte.  These four here: 0000 **0000**.

{% highlight lua %}
bitflag_A =   1                          -- bitflag 0001 
bitflag_B =   2                          -- bitflag 0010
                                         --        +____
combined_bitflag = bitflag_A + bitflag_B -- bitflag 0011 (or 1 + 2 = 3) 
{% endhighlight %}

Now this may look like code voodoo mumbo jumbo, but this is a very practical way to store boolean states for variables.  To give you a real life example, we will use vehicles.

{% highlight lua %}
has_dashcam =    1    -- 0001
has_sticker =    2    -- 0010
has_USB_port =   4    -- 0100
has_cupholders = 8    -- 1000
--etc.
{% endhighlight %}

#### Adding and Subtracting Bitflags

Note that these variables are all *optional* and you can have multiple ones *without them being dependent on each other*.  For instance, it's possible to have a car that has both a sticker and a dashcam.

{% highlight lua %}
fancy_car = 0                          -- 0000 (the car is bare bones raw)
fancy_car = fancy_car + has_dashcam    -- 0001 (the car now has a dashcam attachted)
fancy_car = fancy_car + has_sticker    -- 0011 (and we even threw on a sticker for bling bling!)
-- and if later we wanted to, we could even remove those properties from the variable!
fancy_car = fancy_car - has_sticker    -- 0010 (our fancy_car lost it's luster since we removed the sticker)
{% endhighlight %}

But what if you added the same property twice?  

{% highlight lua %}
fancy_car = 0                       -- 0000
fancy_car = fancy_car + has_dashcam -- 0001
fancy_car = fancy_car + has_dashcam -- 0010 (Uh oh, not good)
{% endhighlight %}

Yikes!  The bitflag has changed so that now our `fancy_car` has morphed into having a sticker unintentionally.  This is not what we want!  To resolve this you need to use [logic switches]() in your code.  

Depending on the language this may be the `&` `|` `~` operations or bit operations `bit.band` `bit.bor` `bit.bnot` in Lua.  The operation to set the bitflags we want needs to be the `or` bit operation for a logic switch.  Let's try this again!

#### Setting a bitflag

{% highlight lua %}
local bit = require('bit')
fancy_car = 0                               -- 0000
fancy_car = bit.bor(fancy_car, has_dashcam) -- 0000 | 0001 = 0001
fancy_car = bit.bor(fancy_car, has_dashcam) -- 0001 | 0001 = 0001 (presto, it hasn't changed!)
{% endhighlight %}

Success!  Using logic switches to add bitflags together no longer causes unintended bugs.

#### Checking a bitflag

To get a bitflags boolean state you need to check if it is set to `1` or `0`. The `and` logic switch will be useful.

{% highlight lua %}
local bit = require('bit')
fancy_car = 0                                            -- 0000
fancy_car = bit.bor(fancy_car, has_dashcam, has_sticker) -- 0000 | 0001 | 0010 = 0011
isolated_bit = bit.band(fancy_car, has_USB_port)         -- 0011 & 0100 = 0000 (isolates the single bit from the rest)
print(isolated_bit == has_USB_port)                      -- 0000 == 0100 (false, it does not have a USB_port)
{% endhighlight %}

Here we compared the `has_USB_port` flag against the bits set in our `fancy_car`.  Because fancy_car lacks a `0100` bitflag, it will return false when checked.  This is due to the `and` logic switch *isolating* the specific bit that is needed.  This bit is checked against the bitflag, which returns false in this case. 

#### Removing a bitflag

If we use the minus operation on the variable, it will run into the same problem as before if we try to remove a bitflag twice or one that isn't there.  To resolve this we must use our logic switches again.  This time it will be trickier, since we must use both the `and` and `not` bit operations.

{% highlight lua %}
local bit = require('bit')
fancy_car = 0                                            -- 0000
fancy_car = bit.bor(fancy_car, has_dashcam, has_sticker) -- 0000 | 0001 | 0010 = 0011
fancy_car = band(fancy_car, bnot(has_dashcam))           -- 0011 & (~0001) = 0010 & 1110 = 0010
{% endhighlight %}

Our `not` bit operator inverts the bitflag we need to remove and the `and` pairs the `1`'s together and anything with a `0` gets set to 0 regardless if it is paired with a `1`.

---

## Why do I Bitmask?

The main uses are simple: 
- *Speed*
- *Compression*
- *Readability*

Instead of looping through multiple booleans in a table like so:

{% highlight lua %}
car1_properties = {has_dashcam=true, has_sticker=false, has_USB_port=true, has_cupholders=false}
car2_properties = {has_dashcam=true, has_sticker=false, has_USB_port=true, has_cupholders=true}
car3_properties = {has_dashcam=false, has_sticker=false, has_USB_port=false, has_cupholders=false}

-- maybe do this for each car?
for prop, state in pairs(car1_properties) do
  -- set each property to true/false/whatever
end

-- or something like
car1_property_has_dashcam = true
car1_property_has_sticker = false
car1_property_has_USB_port = true
car1_property_has_cupholders = false

car2_property_has_dashcam = true
car2_property_has_sticker = true
-- etc. etc. etc. you get the idea
{% endhighlight %}

Doing code like this is slow and ineffecient!  Not to mention it is very bloated!!!  We want to minimize our code footprint as much as possible and have it run quickly by using a *single* variable to hold all the property states.  

Another very good reason to use bitmasking is when you are sending packets of data over a network.  The data needs to be as compressed as possible otherwise you end up with large packets that take longer to send and receive.  By using bitmasking the data is crammed into the smallest possible form that makes delievery a blitz.

## Caveats 

Every language has it's own niche when it comes to the amount of bits stored in a variable.  Make sure you are familiar with the max amount and don't go past it.  If the limit is bypassed, then the bitflags will need to be broken into smaller groups otherwise there will be a bug involving bits overflowing. 

Another important thing that I encountered is *number readability*.  This is **especially important** with larger bitflags as the numbers swell up and it's easy to make mistakes.  It looks like this:

{% highlight lua %}
-- properties for food stored in a house in this example
has_ramen =        1
has_beans =        2
has_soup =         4
has_carrot =       8
has_potato =      16
has_cabbage =     32
has_letteuce =    64
has_milk =       128
has_tomato =     256
has_eggs =       512
has_pickles =   1024
has_onion =     2048
has_bread =     4096
has_butter =    8192
has_biscut =   16384
has_ham =      32768
has_olive =    65536
has_turkey =  131072
--etc. etc.  it keeps getting bigger and bigger
{% endhighlight %}

Doing the first several bitflags are no problem, but once the numbers get past the 12th bit having to do the math in your head can lead to mistakes.  Not to mention the code looks ugly.  If you need to remove a flag from the set, it is a big pain to have to rewrite all the numbers.  

A simpler way to manage this is to use `2^bitflag`. 

{% highlight lua %}
-- properties for food stored in a house in this example
has_ramen =        2^ 0
has_beans =        2^ 1
has_soup =         2^ 2
has_carrot =       2^ 3
has_potato =       2^ 4
has_cabbage =      2^ 5
has_letteuce =     2^ 6
has_milk =         2^ 7
has_tomato =       2^ 8
has_eggs =         2^ 9
has_pickles =      2^10
has_onion =        2^11
has_bread =        2^12
has_butter =       2^13
has_biscut =       2^14
has_ham =          2^15
has_olive =        2^16
has_turkey =       2^17
-- much neater!
{% endhighlight %}

The numbers are the same, but they are stored in a neater and compact fashion.

---

Not every programmer is going to be familiar with bitmasking.  Don't be surprised if you attempt to add it to a project and your fellow developers start scratching their heads in confusion.  If that is the case, just refer them to this post and they can observe the benefits associated with the technique.  

When using bitmasking for the first few times it may seem tough, but once you become accostumed it becomes a natural programming habit just like any other tool.