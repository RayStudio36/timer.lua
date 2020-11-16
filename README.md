# timer.lua

Timer module for lua.

Delayed and time-limited function calls and tweening.

\***This script is from the [Hump](https://github.com/vrld/hump) repository**

## Category

* [Installation](#installation)
* [Example](#example)
* [Documentation](#documentation)
    * [List of Functions](#list-of-functions)
    * [Function Reference](#function-reference)
        * [Timer.new()](#timernew)
        * [Timer.after(delay, func)](#timerafterdelay-func)
        * [Timer.script(func)](#timerscriptfunc)
        * [Timer.every(delay, func, count)](#timereverydelay-func-count)
        * [Timer.during(delay, func, after)](#timerduringdelay-func-after)
        * [Timer.cancel(handle)](#timercancelhandle)
        * [Timer.clear()](#timerclear)
        * [Timer.update(dt)](#timerupdatedt)
        * [Timer.tween(duration, subject, target, method, after, ...)](#timertweenduration-subject-target-method-after-)
    * [Tweening methods](#tweening-methods)

## Installation

The `timer.lua` file should be dropped into an existing project and required by it.

```lua
Timer = require 'timer'
```

[↑Back to category](#category)

## Example

```lua
Timer = require 'timer'

------------------
--- static use ---
------------------
function start()
    Timer.after(1, function() print("Hello, world!") end)
end

function update(dt)
    Timer.update(dt)
end

------------------
--- object use ---
------------------
local timer = Timer()

function start()
    timer:after(1, function() print("Hello, world!") end)
end

function update(dt)
    timer:update(dt)
end
```

[↑Back to category](#category)

## Documentation

See origin [doc](https://hump.readthedocs.io/en/latest/timer.html#)

```lua
Timer = require "timer"
```

`timer` offers a simple interface to schedule the execution of functions. It is possible to run functions after and for some amount of time. For example, a timer could be set to move critters every 5 seconds or to make the player invincible for a short amount of time.

In addition to that, `timer` offers various [tweening](http://en.wikipedia.org/wiki/Inbetweening) functions that make it easier to produce [juicy games](http://www.youtube.com/watch?v=Fy0aCDmgnxg).

[↑Back to category](#category)

### List of Functions

- [`Timer.new()`](#`Timer.new()`)
- [`Timer.after(delay, func)`](#`Timer.after(delay,-func)`)
- [`Timer.script(func)`](#`Timer.script(func)`)
- [`Timer.every(delay, func, count)`](#`Timer.every(delay,-func,-count)`)
- [`Timer.during(delay, func, after)`](#`Timer.during(delay,-func,-after)`)
- [`Timer.cancel(handle)`](#`Timer.cancel(handle)`)
- [`Timer.clear()`](#`Timer.clear()`)
- [`Timer.update(dt)`](#`Timer.update(dt)`)
- [`Timer.tween(duration, subject, target, method, after, ...)`](#`Timer.tween(duration,-subject,-target,-method,-after,-...)`)

[↑Back to category](#category)

### Function Reference

#### `Timer.new()`

- Returns: A timer instance.

Creates a new timer instance that is independent of the global timer: It will manage it’s own list of scheduled functions and does not in any way affect the the global timer. Likewise, the global timer does not affect timer instances.

**Note**

*If you don’t need multiple independent schedulers, you can use the global/default timer (see examples).*

**Note**

*Unlike the default timer, timer instances use the colon-syntax, i.e., you need to call `instance:after(1, foo)` instead of `Timer.after(1, foo)`.*

**Example**

```lua
menuTimer = Timer.new()
```

[↑Back to category](#category)

#### `Timer.after(delay, func)`

- Arguments: 
    - **delay** (number) – Number of seconds the function will be delayed.
    - **func** (function) – The function to be delayed.
- Returns: The timer handle. See also [`Timer.cancel(handle)`](#`Timer.cancel(handle)`).

Schedule a function. The function will be executed after `delay` seconds have elapsed, given that `update(dt)` is called every frame.

**Note**

*There is no guarantee that the delay will not be exceeded, it is only guaranteed that the function will not be executed before the delay has passed.*

`func` will receive itself as only parameter. This is useful to implement periodic behavior (see the example).

**Example**

```lua
-- grant the player 5 seconds of immortality
player.isInvincible = true
Timer.after(5, function() player.isInvincible = false end)
```

```lua
-- print "foo" every second. See also every()
Timer.after(1, function(func) print("foo") Timer.after(1, func) end)
```

```lua
--Using a timer instance:
menuTimer:after(1, finishAnimation)
```

[↑Back to category](#category)

#### `Timer.script(func)`

- Arguments:
    - **func** (function) – Script to execute.

Execute a function that can be paused without causing the rest of the program to be suspended. `func` will receive a function - `wait` - to do interrupt the script (but not the whole program) as only argument. The function prototype of wait is: `wait(delay)`.

**Example**

```lua
Timer.script(function(wait)
    print("Now")
    wait(1)
    print("After one second")
    wait(1)
    print("Bye!")
end)
```

```lua
-- useful for splash screens
Timer.script(function(wait)
    Timer.tween(0.5, splash.pos, {x = 300}, 'in-out-quad')
    wait(5) -- show the splash for 5 seconds
    Timer.tween(0.5, slpash.pos, {x = 800}, 'in-out-quad')
end)
```

```lua
-- repeat something with a varying delay
Timer.script(function(wait)
    while true do
        spawn_ship()
        wait(1 / (1-production_speed))
    end
end)
```

```lua
-- jumping with timer.script
self.timers:script(function(wait)
    local w = 1/12
    self.jumping = true
    Timer.tween(w*2, self, {z = -8}, "out-cubic", function()
        Timer.tween(w*2, self, {z = 0},"in-cubic")
    end)

    self.quad = self.quads.jump[1]
    wait(w)

    self.quad = self.quads.jump[2]
    wait(w)

    self.quad = self.quads.jump[3]
    wait(w)

    self.quad = self.quads.jump[4]
    wait(w)

    self.jumping = false
    self.z = 0
end)
```

[↑Back to category](#category)

#### `Timer.every(delay, func, count)`

- Arguments:
    - **delay** (number) – Number of seconds between two consecutive function calls.
    - **func** (function) – The function to be called periodically.
    - **count** (number) – Number of times the function is to be called (optional).
- Returns: The timer handle. See also Timer.cancel().

Add a function that will be called `count` times every `delay` seconds.

If `count` is omitted, the function will be called until it returns `false` or Timer.cancel() or Timer.clear() is called on the timer instance.

**Example**

```lua
-- toggle light on and off every second
Timer.every(1, function() lamp:toggleLight() end)
```

```lua
-- launch 5 fighters in quick succession (using a timer instance)
mothership_timer:every(0.3, function() self:launchFighter() end, 5)
```

```lua
-- flicker player's image as long as he is invincible
Timer.every(0.1, function()
    player:flipImage()
    return player.isInvincible
end)
```

[↑Back to category](#category)

#### `Timer.during(delay, func, after)`

- Arguments:
    - **delay** (number) – Number of seconds the func will be called.
    - **func** (function) – The function to be called on `update(dt)`.
    - **after** (function) – A function to be called after delay seconds (optional).
- Returns: The timer handle. See also [`Timer.cancel(handle)`](#`Timer.cancel(handle)`).

Run `func(dt)` for the next `delay` seconds. The function is called every time `update(dt)` is called. Optionally run `after()` once `delay` seconds have passed.

`after()` will receive itself as only parameter.

**Note**

*You should not add new timers in `func(dt)`, as this can lead to random crashes.*

**Example**

```lua
-- play an animation for 5 seconds
Timer.during(5, function(dt) animation:update(dt) end)
```

```lua
-- shake the camera for one second
local orig_x, orig_y = camera:position()
Timer.during(1, function()
    camera:lookAt(orig_x + math.random(-2,2), orig_y + math.random(-2,2))
end, function()
    -- reset camera position
    camera:lookAt(orig_x, orig_y)
end)
```

```lua
player.isInvincible = true
-- flash player for 3 seconds
local t = 0
player.timer:during(3, function(dt)
    t = t + dt
    player.visible = (t % .2) < .1
end, function()
    -- make sure the player is visible after three seconds
    player.visible = true
    player.isInvincible = false
end)
```

[↑Back to category](#category)

#### `Timer.cancel(handle)`

- Arguments:
    - **handle** (table) – The function to be canceled.

Prevent a timer from being executed in the future.

**Example**

```lua
function tick()
    print('tick... tock...')
end
handle = Timer.every(1, tick)
-- later
Timer.cancel(handle) -- NOT: Timer.cancel(tick)
```

```lua
-- using a timer instance
function tick()
    print('tick... tock...')
end
handle = menuTimer:every(1, tick)
-- later
menuTimer:cancel(handle)
```

[↑Back to category](#category)

#### `Timer.clear()`

Remove all timed and periodic functions. Functions that have not yet been executed will discarded.

**Example**

```lua
Timer.clear()
```

```lua
menuTimer:clear()
```

[↑Back to category](#category)

#### `Timer.update(dt)`

- Arguments:
    - **dt** (number) – Time that has passed since the last `update()`.

Update timers and execute functions if the deadline is reached. Call in game `update(dt)`(such as [love2d](https://http://love2d.org/)'s `love.update(dt)`).

```lua
function love.update(dt)
    do_stuff()
    Timer.update(dt)
end
```

```lua
-- using hump.gamestate and a timer instance
function menuState:update(dt)
    self.timers:update(dt)
end
```

[↑Back to category](#category)

#### `Timer.tween(duration, subject, target, method, after, ...)`

- Arguments:	
    - **duration** (number) – Duration of the tween.
    - **subject** (table) – Object to be tweened.
    - **target** (table) – Target values.
    - **method** (string) – Tweening method, defaults to ‘linear’ (see here, optional).
    - **after** (function) – Function to execute after the tween has finished (optional).
    - **...** (mixed) – Additional arguments to the tweening function.

- Returns: A timer handle.

[Tweening](http://en.wikipedia.org/wiki/Inbetweening) (short for in-betweening) is the process that happens between two defined states. For example, a tween can be used to gradually fade out a graphic or move a text message to the center of the screen. For more information why tweening should be important to you, check out this great talk on [juicy games](http://www.youtube.com/watch?v=Fy0aCDmgnxg).

`timer` offers two interfaces for tweening: the low-level `Timer.during()` and the higher level interface `Timer.tween()`.

To see which tweening methods hump offers, see [below](#tweeningmethods).

**Example**

```lua
function love.load()
    color = {0, 0, 0}
    Timer.tween(10, color, {255, 255, 255}, 'in-out-quad')
end

function love.update(dt)
    Timer.update(dt)
end

function love.draw()
    love.graphics.setBackgroundColor(color)
end
```

```lua
function love.load()
    circle = {rad = 10, pos = {x = 400, y = 300}}
    -- multiple tweens can work on the same subject
    -- and nested values can be tweened, too
    Timer.tween(5, circle, {rad = 50}, 'in-out-quad')
    Timer.tween(2, circle, {pos = {y = 550}}, 'out-bounce')
end

function love.update(dt)
    Timer.update(dt)
end

function love.draw()
    love.graphics.circle('fill', circle.pos.x, circle.pos.y, circle.rad)
end
```

```lua
function love.load()
    -- repeated tweening

    circle = {rad = 10, x = 100, y = 100}
    local grow, shrink, move_down, move_up
    grow = function()
        Timer.tween(1, circle, {rad = 50}, 'in-out-quad', shrink)
    end
    shrink = function()
        Timer.tween(2, circle, {rad = 10}, 'in-out-quad', grow)
    end

    move_down = function()
        Timer.tween(3, circle, {x = 700, y = 500}, 'bounce', move_up)
    end
    move_up = function()
        Timer.tween(5, circle, {x = 200, y = 200}, 'out-elastic', move_down)
    end

    grow()
    move_down()
end

function love.update(dt)
    Timer.update(dt)
end

function love.draw()
    love.graphics.circle('fill', circle.x, circle.y, circle.rad)
end
```

[↑Back to category](#category)

### Tweening methods

At the core of tweening lie interpolation methods. These methods define how the output should look depending on how much time has passed. For example, consider the following tween:

```lua
-- now: player.x = 0, player.y = 0
Timer.tween(2, player, {x = 2})
Timer.tween(4, player, {y = 8})
```

At the beginning of the tweens (no time passed), the interpolation method would place the player at `x = 0, y = 0`. After one second, the player should be at `x = 1, y = 2`, and after two seconds the output is `x = 2, y = 4`.

The actual duration of and time since starting the tween is not important, only the fraction of the two. Similarly, the starting value and output are not important to the interpolation method, since it can be calculated from the start and end point. Thus an interpolation method can be fully characterized by a function that takes a number between 0 and 1 and returns a number that defines the output (usually also between 0 and 1). The interpolation function must hold that the output is 0 for input 0 and 1 for input 1.

timer predefines several commonly used interpolation methods, which are generalized versions of [Robert Penner’s easing functions](http://www.robertpenner.com/easing/). Those are:

`'linear'`, `'quad'`, `'cubic'`, `'quart'`, `'quint'`, `'sine'`, `'expo'`, `'circ'`, `'back'`, `'bounce'`, and `'elastic'`.

**Custom interpolators**

You can add custom interpolation methods by adding them to the tween table:

```lua
Timer.tween.sqrt = function(t) return math.sqrt(t) end
-- or just Timer.tween.sqrt = math.sqrt
```

Access the your method like you would the predefined ones. You can even use the modyfing prefixes:

```lua
Timer.tween(5, circle, {radius = 50}, 'in-out-sqrt')
```

You can also invert and chain functions:

```lua
outsqrt = Timer.tween.out(math.sqrt)
inoutsqrt = Timer.tween.chain(math.sqrt, outsqrt)
```

[↑Back to category](#category)
