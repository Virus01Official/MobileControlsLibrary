# MobileControls

`MobileControls` is a Lua library for adding virtual joystick and button controls to Love2D games, making it easier to test mobile-like controls in a desktop environment. This library emulates mobile touchscreen input, offering simple, interactive controls for movement and button actions. It is ideal for game prototypes or mobile games developed with Love2D.

## Features

- Virtual Joystick: Control movement with a touch-based joystick.
- Buttons: Simulate touch-based button presses.
- Mouse Simulation: On desktop, mouse input simulates touch events.
- Joystick Input: Retrieve normalized joystick values for player movement or other controls.

## Installation

Simply copy the `mobilecontrols.lua` file from 'src/mobilecontrols.lua' into your Love2D project directory.

```plaintext
your_project/
├── main.lua
├── mobilecontrols.lua
```

In your `main.lua`, you can require the library like so:

```lua
local MobileControls = require("mobilecontrols")
```

## Example Usage

The following code demonstrates how to integrate the `MobileControls` library in a basic Love2D project to move a player object using the virtual joystick.

### `main.lua`

```lua
local MobileControls = require("mobilecontrols")

-- Player object
local player = {
    x = love.graphics.getWidth() / 2,
    y = love.graphics.getHeight() / 2,
    speed = 200, -- Speed in pixels per second
    radius = 20
}

function love.load()
    love.graphics.setBackgroundColor(0.1, 0.1, 0.1)
end

function love.update(dt)
    local touches = {}
    
    if love.touch then
        -- Handle mobile touch input (if touch is available)
        for i = 1, love.touch.getTouchCount() do
            local id, x, y = love.touch.getTouch(i)
            -- Convert touch coordinates to match the screen resolution
            table.insert(touches, {id = id, x = x * love.graphics.getWidth(), y = y * love.graphics.getHeight()})
        end
    else
        -- Fallback for desktop (use mouse as touch input)
        local id = 1
        local x, y = love.mouse.getPosition()
        table.insert(touches, {id = id, x = x, y = y})
    end

    MobileControls:updateTouches(touches)
    
    -- Get joystick input
    local dx, dy = MobileControls:getJoystick()

    -- Move the player based on joystick input
    player.x = player.x + dx * player.speed * dt
    player.y = player.y + dy * player.speed * dt

    -- Clamp player position to screen bounds
    player.x = math.max(player.radius, math.min(love.graphics.getWidth() - player.radius, player.x))
    player.y = math.max(player.radius, math.min(love.graphics.getHeight() - player.radius, player.y))
end

function love.draw()
    -- Draw mobile controls
    MobileControls:draw()

    -- Draw player
    love.graphics.setColor(0.8, 0.2, 0.2)
    love.graphics.circle("fill", player.x, player.y, player.radius)

    -- Display joystick values
    local dx, dy = MobileControls:getJoystick()
    love.graphics.setColor(1, 1, 1)
    love.graphics.print(string.format("Joystick: dx=%.2f, dy=%.2f", dx, dy), 10, 10)
end

function love.touchreleased(id)
    MobileControls:touchReleased(id)
end

function love.mousepressed(x, y, button, istouch, presses)
    -- Simulate touch press for desktop
    MobileControls:updateTouches({
        {id = 1, x = x, y = y}
    })
end

function love.mousereleased(x, y, button, istouch, presses)
    -- Simulate touch release for desktop
    MobileControls:touchReleased(1)
end
```

## API

### `MobileControls:updateTouches(touches)`

- **touches**: A list of touch events, where each touch is a table with `id`, `x`, and `y` coordinates.
- Updates joystick and button states based on the current touch inputs.

### `MobileControls:draw()`

- Draws the virtual joystick and buttons on the screen.

### `MobileControls:getJoystick()`

- Returns the normalized joystick values (`dx`, `dy`), where both values range from -1 to 1.

### `MobileControls:isButtonPressed(label)`

- **label**: The label of the button you want to check.
- Returns `true` if the specified button is pressed, otherwise `false`.

### `MobileControls:touchReleased(id)`

- **id**: The touch ID that is released.
- Updates control states when a touch event is released.

## License

This library is released under the MIT License. See `LICENSE` for more details.
