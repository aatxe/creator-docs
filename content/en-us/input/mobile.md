---
title: Mobile input
description: Explains Roblox support for mobile devices.
---

Over half of all Roblox sessions are played on mobile devices, so it's important to consider cross-platform accessibility when designing an experience for a wide audience. You should aim to support a variety of input devices, including [mouse and keyboard inputs](../input/mouse-and-keyboard.md) and [gamepad](../input/gamepad.md).

When designing a mobile experience, consider the [device orientation](#device-orientation) that you intend user's to use in your experience, then implement your inputs with `Class.ContextActionService` to perform the following mobile-related input tasks:

- [Create on-screen buttons](#custom-mobile-buttons) visible only on mobile devices.
- [Setup context dependent inputs](#context-dependent-inputs) that allows the same button or input to perform a different action depending on the situation.
- [Detect other input devices](#detect-other-devices), such as a mouse or keyboard connected to a mobile tablet, to provide the correct on-screen prompts to the user.

## Device orientation

On phones and tablets, the device orientation majorly affects the user experience and interaction. For example, landscape mode is best operated with two thumbs while portrait mode may lend itself to one-finger interface.

By default, Roblox experiences run in landscape mode, allowing the experience to switch between landscape "left" and landscape "right" as the user's device rotates. However, experiences can be locked to a particular orientation if desired.

### Orientation modes

There are five different orientation modes, including two sensor-based modes and three locked modes.

<table>
<thead>
  <tr>
    <th colspan="2">Sensor modes</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Landscape sensor</td>
    <td>The default Roblox setting in which the experience always appears in landscape mode (no portrait mode) and the device detects its physical orientation to ensure the experience view is always oriented upward.</td>
  </tr>
  <tr>
    <td>Sensor</td>
    <td>The device detects its physical orientation to ensure the experience view is always oriented upward, switching between landscape and portrait mode as needed.</td>
  </tr>
</tbody>
</table>

<table>
<thead>
  <tr>
    <th colspan="2">Locked modes</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Landscape left</td>
    <td>On devices with a physical home button, the home button is to the left of the display. On devices with a virtual home/nav bar, its touch region is at the bottom of the display.</td>
  </tr>
  <tr>
    <td>Landscape right</td>
    <td>On devices with a physical home button, the home button is to the right of the display. On devices with a virtual home/nav bar, its touch region is at the bottom of the display.</td>
  </tr>
  <tr>
    <td>Portrait</td>
    <td>On devices with a physical home button, the home button is below the display. On devices with a virtual home/nav bar, its touch region is at the bottom of the display.</td>
  </tr>
</tbody>
</table>

<Alert severity="warning">
Roblox does not include a "portrait upside-down" mode since many devices do not natively support that orientation.
</Alert>

### Orientation properties

When setting an orientation, you can set the [starting orientation](#starting-orientation), the [in-experience orientation](#in-experience-orientation), and the [current orientation](#current-orientation).

#### Starting orientation

`Class.StarterGui.ScreenOrientation` sets the default orientation for a place. Acceptable values include:

- `Enum.ScreenOrientation|ScreenOrientation.LandscapeSensor`
- `Enum.ScreenOrientation|ScreenOrientation.Sensor`
- `Enum.ScreenOrientation|ScreenOrientation.LandscapeLeft`
- `Enum.ScreenOrientation|ScreenOrientation.LandscapeRight`
- `Enum.ScreenOrientation|ScreenOrientation.Portrait`

Because this property affects all new users who join the experience, you can set its value in `Class.StarterGui` → `Enum.ScreenOrientation` within Studio.

#### In-experience orientation

`Class.PlayerGui.ScreenOrientation` explicitly changes the experience's orientation for a user. When this property is set to one of the `Enum.ScreenOrientation` enums in a `Class.LocalScript`, the experience will immediately orient itself to match the setting. This can be useful when an experience needs to provide a particular experience like locking the view to portrait for a minigame.

The following code sample in a `Class.LocalScript` sets the screen orientation to portrait:

```lua
local Players = game:GetService("Players")
local playerGUI = Players.LocalPlayer:WaitForChild("PlayerGui")

task.wait(2)

playerGUI.ScreenOrientation = Enum.ScreenOrientation.Portrait
```

#### Current orientation

`Class.PlayerGui.CurrentScreenOrientation` gets the current device orientation. Possible values include:

- `Enum.ScreenOrientation|ScreenOrientation.LandscapeLeft`
- `Enum.ScreenOrientation|ScreenOrientation.LandscapeRight`
- `Enum.ScreenOrientation|ScreenOrientation.Portrait`

The following code prints the user's current screen orientation:

```lua
local Players = game:GetService("Players")
local playerGUI = Players.LocalPlayer:WaitForChild("PlayerGui")

print(playerGUI.CurrentScreenOrientation)
```

## Character movement modes

Roblox offers several `Class.StarterPlayer` properties you can set to change how users on mobile devices can move through your experience.

You can set mobile movement control schemes for Roblox experiences by changing the values of `Class.StarterPlayer.DevTouchMovementMode` to one of the following:

<table>
<thead>
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>`ClickToMove`</td>
    <td>Users can only move through the experience by tapping a target location. This mode includes a jump button in the lower-right region of the screen. [Automatic jumping](#automatic-jumping) is always active in this movement mode.</td>
  </tr>
  <tr>
    <td>`DPad`</td>
    <td><Alert severity="error">This option has been removed from the Roblox mobile app and should not be used for production-ready experiences. </Alert></td>
  </tr>
  <tr>
    <td>`DynamicThumbstick`</td>
    <td>A dynamic thumbstick appears where the user initially presses down. This mode includes a jump button in the lower-right region of the screen. This is the default user setting for mobile users if `UserChoice` is set.</td>
  </tr>
  <tr>
    <td>`Scriptable`</td>
    <td>Disables all default controls and allows you to [script your own control scheme](#adding-mobile-buttons).</td>
  </tr>
  <tr>
    <td>`Thumbpad`</td>
    <td><Alert severity="error">This option has been removed from the Roblox mobile app and should not be used for production-ready experiences. </Alert></td>
  </tr>
  <tr>
    <td>`Thumbstick`</td>
    <td>A mobile thumbstick located in the lower-left region of the screen. Unlike `DynamicThumbstick`, the thumbstick position is static and doesn't change position when the user touches on the screen.</td>
  </tr>
  <tr>
    <td>`UserChoice`</td>
    <td>Allows users to choose their desired control scheme from the in-experience Settings menu. This is the default movement mode for experiences.</td>
  </tr>
</tbody>
</table>

### Automatic jumping

When `Class.StarterPlayer.AutoJumpEnabled` is enabled, the user's character automatically jumps across gaps when approaching the edge of a platform. `Class.StarterPlayer.AutoJumpEnabled` is enabled by default for mobile devices.

Disable `Class.StarterPlayer.AutoJumpEnabled` to disable this feature and force users to jump only using their key bindings.

## Custom Mobile Buttons

To add custom mobile buttons, use the `Class.ContextActionService:BindAction()` method which takes the following parameters:

<table>
<thead>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>`actionName`</td>
    <td>string</td>
    <td>An identifier string for the action you are binding. You can use the actionName with other functions in `Class.ContextActionService` to edit the binding.</td>
  </tr>
  <tr>
    <td>`functionToBind`</td>
    <td>function</td>
    <td>The function to call when the specified input is triggered. This function receives three arguments:
    <ul><li> A string equal to the actionName.</li>
    <li> A `Enum.UserInputState` which defines the input state when it called the function.</li>
    <li> The `Class.InputObject` used in the function call.</li></ul></td>
  </tr>
  <tr>
    <td>`createTouchButton`</td>
    <td>boolean</td>
    <td>When true, creates an on-screen button when the game is running on a mobile device.</td>
  </tr>
  <tr>
    <td>`inputTypes`</td>
    <td>tuple</td>
    <td>The inputs you intend to bind to the function, such as enum values from a `Enum.KeyCode`.</td>
  </tr>
</tbody>
</table>

You can use the following code sample to create an Interact action that creates an on-screen button and also accepts a [keyboard](./mouse-and-keyboard.md#keyboard-input) and [gamepad](./gamepad.md) input:

```lua
local ContextActionService = game:GetService("ContextActionService")

local function handleAction(actionName, inputState, inputObject)
	if inputState == Enum.UserInputState.Begin then
		print(actionName, inputObject)
	end
end

-- Bind action to function
ContextActionService:BindAction("Interact", handleAction, true, Enum.KeyCode.T, Enum.KeyCode.ButtonR1)
```

<Alert severity="info">
To remove a mobile button from the screen, call `Class.ContextActionService:UnbindAction()|UnbindAction()` using the `actionName` string you passed to `Class.ContextActionService:BindAction()|BindAction()`.
</Alert>

Once a custom button is added, you can use one of the several functions from `Class.ContextActionService` to customize the on-screen buttons that are created by `Class.ContextActionService:BindAction()|BindAction()`.

- To change the text label for a mobile button, call `Class.ContextActionService:SetTitle()|SetTitle()` with the `actionName` string and a title string.
- To use a custom image just like other GUI buttons, call `Class.ContextActionService:SetImage()|SetImage()` method, replacing the example asset ID below with an image of your choice.
- To set a button's position, call `Class.ContextActionService:SetPosition()|SetPosition()` with a `Datatype.UDim2` position value.

```lua
-- Set button label to "Talk"
ContextActionService:SetTitle("Interact", "Talk")
-- Set button image
ContextActionService:SetImage("Interact", "rbxassetid://104919049969988")
-- Set button position
ContextActionService:SetPosition("Interact", UDim2.new(1, -70, 0, 10))
```

### Context-Dependent Inputs

When developing for mobile devices you may often want to change what a single button does based on the context. Since screen space on mobile devices is limited, use contextual buttons that perform different actions based on what the character is able to do.

For example, you can display an active "Collect" button when the user is standing near a chest of gold, bound to the function `collectTreasure()`:

```lua
local ContextActionService = game:GetService("ContextActionService")

local function collectTreasure(actionName, inputState, inputObject)
	if inputState == Enum.UserInputState.Begin then
		print("Collect treasure")
	end
end

ContextActionService:BindAction("Interact", collectTreasure, true, Enum.KeyCode.T, Enum.KeyCode.ButtonR1)
ContextActionService:SetTitle("Interact", "Collect")
ContextActionService:SetPosition("Interact", UDim2.new(1, -70, 0, 10))
```

At another point during gameplay, you can change the button to "Talk" when the user is standing near an NPC. Instead of removing the existing button to place another, you can simply call `Class.ContextActionService:BindAction()|BindAction()` on the existing `"Interact"` action, changing the target function and button title:

```lua
ContextActionService:BindAction("Interact", talkToNPC, true, Enum.KeyCode.T, Enum.KeyCode.ButtonR1)
ContextActionService:SetTitle("Interact", "Talk")
```

## Detect other devices

In cross-platform experiences, it's important to reference the user's preferred input options by displaying input options for the actively used device. For example, a mobile device can have a [mouse and keyboard](./mouse-and-keyboard.md) or [gamepad](./gamepad.md) connected, or it's possible that a desktop has a touchscreen enabled. If multiple input sources are enabled, you can use `Class.UserInputService:GetLastInputType()|GetLastInputType()` to get the user's last used input device.

As a foundation, you can use the following `Class.ModuleScript`, placed within `Class.ReplicatedStorage` and renamed to **UserInputModule**, to fetch the user's input type, after which you can adapt the UI layout or context to your experience's specific needs.

```lua
local UserInputService = game:GetService("UserInputService")

local UserInput = {}

local inputTypeString
-- If device has active keyboard and mouse, assume those inputs
if UserInputService.KeyboardEnabled and UserInputService.MouseEnabled then
	inputTypeString = "Keyboard/Mouse"
-- Else if device has touch capability but no keyboard and mouse, assume touch input
elseif UserInputService.TouchEnabled then
	inputTypeString = "Touch"
-- Else if device has an active gamepad, assume gamepad input
elseif UserInputService.GamepadEnabled then
	inputTypeString = "Gamepad"
end

function UserInput.getInputType()
	local lastInputEnum = UserInputService:GetLastInputType()

	if lastInputEnum == Enum.UserInputType.Keyboard or string.find(tostring(lastInputEnum.Name), "MouseButton") or lastInputEnum == Enum.UserInputType.MouseWheel then
		inputTypeString = "Keyboard/Mouse"
	elseif lastInputEnum == Enum.UserInputType.Touch then
		inputTypeString = "Touch"
	elseif string.find(tostring(lastInputEnum.Name), "Gamepad") then
		inputTypeString = "Gamepad"
	end
	return inputTypeString, lastInputEnum
end

return UserInput
```

Once the **UserInputModule** script is in place, use the following code sample in a `Class.LocalScript` to get the user's last input type:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Require module
local UserInputModule = require(ReplicatedStorage:WaitForChild("UserInputModule"))

local currentUserInput, inputEnum = UserInputModule.getInputType()
print(currentUserInput, inputEnum)
```
