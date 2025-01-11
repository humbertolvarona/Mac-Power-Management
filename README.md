# Mac Power Management Script

This AppleScript allows you to toggle between minimal and normal power consumption modes on a macOS system. It provides options to minimize power usage by disabling Bluetooth and Wi-Fi and turning off the display, as well as to restore the system to normal power consumption settings.

---

## Features

- **Minimal Power Consumption Mode**:
  - Disables Bluetooth.
  - Disables Wi-Fi.
  - Turns off the display.
  - Saves the system's state as `minimal`.

- **Normal Power Consumption Mode**:
  - Restores Bluetooth.
  - Restores Wi-Fi.
  - Saves the system's state as `normal`.

- State persistence using a hidden file in the user's home directory (`~/.mac_power_state`).

---

## How It Works

### States
The script maintains two states:
1. **`normal`**: Represents the default power consumption mode.
2. **`minimal`**: Represents the minimal power consumption mode.

### Workflow
1. On execution, the script checks the current state from the `~/.mac_power_state` file.
2. It displays a dialog with the appropriate options based on the current state:
   - If in `normal` mode, it offers to switch to `minimal` mode.
   - If in `minimal` mode, it offers to restore `normal` mode.
3. Based on the user's choice:
   - Applies minimal power settings (disables Bluetooth, Wi-Fi, and turns off the display).
   - Restores normal settings (enables Bluetooth and Wi-Fi).

---

## Requirements

1. macOS with AppleScript support.
2. Admin privileges are required for:
   - Disabling/enabling Bluetooth.
   - Disabling/enabling Wi-Fi.

---

## Script

Here is the script:

```applescript
--------------------------------------------------------------------------------
-- Function to get the path of the state file
--------------------------------------------------------------------------------
on getStateFilePath()
	return (POSIX path of (path to home folder)) & ".mac_power_state"
end getStateFilePath

--------------------------------------------------------------------------------
-- Function to read the current state from the file
--------------------------------------------------------------------------------
on readCurrentState()
	set thePath to getStateFilePath()
	try
		-- Check if the file exists
		set currentState to do shell script "test -f " & quoted form of thePath & " && cat " & quoted form of thePath
		-- Validate content
		if currentState is not "minimal" and currentState is not "normal" then error
	on error
		-- Assume "normal" if the file doesn't exist or is corrupted
		set currentState to "normal"
	end try
	return currentState
end readCurrentState

--------------------------------------------------------------------------------
-- Function to save the state to the file
--------------------------------------------------------------------------------
on saveState(newState)
	set thePath to getStateFilePath()
	do shell script "echo " & quoted form of newState & " > " & quoted form of thePath
end saveState

--------------------------------------------------------------------------------
-- Main script logic
--------------------------------------------------------------------------------
-- Read the current state
set currentState to readCurrentState()

-- Decide which dialog to show based on the current state
if currentState is "minimal" then
	set dialogMessage to "Your Mac is currently in minimal power consumption mode. What do you want to do?"
	set dialogButtons to {"Restore", "Cancel"}
	set defaultButton to "Restore"
else
	set dialogMessage to "Your Mac is currently in normal power consumption mode. What do you want to do?"
	-- Rename the "OK" button to "Minimize"
	set dialogButtons to {"Minimize", "Cancel"}
	set defaultButton to "Minimize"
end if

-- Display the dialog
set userChoice to display dialog dialogMessage buttons dialogButtons default button defaultButton

-- Determine the user's action
set userAction to button returned of userChoice

if userAction is "Cancel" then
	display dialog "The script has been canceled." buttons {"OK"} default button "OK"
	return
	
else if userAction is "Minimize" then
	----------------------------------------------------------------------------
	-- Apply minimal power consumption settings
	----------------------------------------------------------------------------
	
	-- Turn off the display (suspend backlight)
	do shell script "pmset displaysleepnow"
	
	-- Disable Bluetooth (requires admin privileges)
	do shell script "defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0; killall -HUP bluetoothd" with administrator privileges
	
	-- Disable Wi-Fi (requires admin privileges)
	-- Replace 'en0' with the correct interface name if needed
	do shell script "networksetup -setairportpower en0 off" with administrator privileges
	
	-- Save the new state
	saveState("minimal")
	
	-- Final message
	display dialog "Your Mac is now in minimal power consumption mode. The display will turn on again when you press a key or move the mouse." buttons {"OK"} default button "OK"
	
else if userAction is "Restore" then
	----------------------------------------------------------------------------
	-- Restore normal power consumption settings
	----------------------------------------------------------------------------
	
	-- Enable Bluetooth (requires admin privileges)
	do shell script "defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 1; killall -HUP bluetoothd" with administrator privileges
	
	-- Enable Wi-Fi (requires admin privileges)
	-- Replace 'en0' with the correct interface name if needed
	do shell script "networksetup -setairportpower en0 on" with administrator privileges
	
	-- Save the new state
	saveState("normal")
	
	-- Final message
	display dialog "Your Mac has been restored to normal power consumption mode." buttons {"OK"} default button "OK"
end if
```

---

## How to Use

1. Open the **Script Editor** application on your Mac.
2. Copy and paste the script into a new document.
3. Save the script as an **Application**:
   - Select `File > Export`.
   - Choose `File Format: Application`.
4. Run the application.
5. Follow the prompts to toggle between power consumption modes.

---

## Notes

1. **Admin Privileges**: The script will prompt for your administrator password when enabling/disabling Wi-Fi or Bluetooth.
2. **Wi-Fi Interface**: Ensure `en0` is the correct Wi-Fi interface for your Mac. If not, update the script with the correct interface name.
3. **State File**: The script saves the current state in a hidden file `~/.mac_power_state` in your home directory.
4. **Customizable**: You can modify the script to include additional power-saving actions as needed.

---

## License

This script is provided under the [MIT License](LICENSE). Feel free to use, modify, and distribute it as needed.

