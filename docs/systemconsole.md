
The system console gives you a direct way to interact with Tabletop Simulator's settings.  It can be used to automate repetitive actions, and customise your TTS experience.


## Controls

The console accepts basic text input.  Additionally, you may hit `up arrow` and `down arrow` to cycle back and forth through the command history, and `tab` to autocomplete your current command (if more than one command is possible from your current prefix then hit it a couple of times to list all matching commands).  Hitting the `` ` `` key will activate the console.  Hitting it again will type the character; if you would rather it deactivated the console input as well as activating it, then you can make it do that by entering this command: `+console_hotkey_lock`


## Commands & Variables

You interact with the system console by typing commands into it.  When you hit enter the command will be perform its action, and then output its results to the console.

There is a subset of the commands which have an additional property; these are called `variables`, and each one stores a value (in addition to behaving like regular commands).  Typically these are used for the settings which govern TTS behaviour.  For example, the `color` command is a variable; it holds the seat color you currently occupy.  Typing it without any parameters will make it output its current value, while typing it with a parameter will let you set it; you could type `color red` to switch to the red seat, for instance.  You may also create your own variables with the `store_number`, `store_toggle`, and `store_text` commands; these are especially useful when writing scripts.

A lot of variables are toggles: they can be either `OFF` or `ON`.  For these you can set them in an additional way; by prefixing with `+`, `-`, or `!`.  `+` will turn it on, `-` will turn it off, and `!` will toggle it, changing it to the opposite of its current value.  For example: `+measure_in_metric` will set `measure_in_metric` to `ON`, and `!measure_logging` when `measure_logging` is `ON` will set it to `OFF` (and vice-versa).

Note that some variables are `persistent` - this means that they will remember their value even after you close the game.

There are three commands which are extremely valuable when getting to know the console, these are:

* `commands` - Lists all non-variable commands.
* `variables` - Lists all variable commands and their current values.
* `help` - Lists all commands, with a description of what they do.

You may specify a prefix when using these to have them filter to only commands which begin with it; for example you could type `variables spectator` to see the value of all commands which deal with the spectator window.
Additionally, if you ask for `help` on a specific command it will give you a detailed description of how to use it.

You may insert variables into commands by enclosing them in `<<` and `>>`.  For example, typing `hovered` will display the GUID of the object your pointer is hovering over; `spectator_camera_target <<hovered>>` will set the spectator camera target with it.

If you start a command with the `@` symbol then it will be silenced; it will not output anything to the console.


## Scripts

The `exec` command will execute a series of commands separated either by `;` or by being on seperate lines.  Alternatively, you may use the `-v` parameter to execute a text variable as a script.  For example, `exec -v bootexec` will execute the commands in the `bootexec` variable.  You may also pass `exec` a `-q` parameter to make it execute in quiet mode (commands being executed are not echoed to the console, but their output is).

There are two special variables: `autoexec` and `bootexec`; these are text variables which load the contents of those two files - `autoexec.cfg` and `bootexec.cfg`, respectively - in your TTS user folder (`C:\Users\<username>\Documents\My Games\Tabletop Simulator`).

`autoexec` will be executed every time you start a game within TTS, while `bootexec` is only executed once, when TTS first starts up.  Note that the game resets every time you go back to the main menu, so if you want to affect any settings / add bindings / etc you need to do it in `autoexec`

Example `bootexec`:
``` bash
# Host a game for 8 players using default server name and password
host_game 8

# Load game on row 4 slot 5
ui_games_click 4 5

# Switch to system console
chat_tab_system

# Activate spectator window
+spectator_window
```

Example `autoexec`:
``` bash
# Make easier to type versions of spectator_camera_ commands.
# i.e. cam_load instead of spectator_camera_load
alias cam_* spectator_camera_*

# Set some settings
+cam_stay_upright
-spectator_show_ui

# make right control have camera follow player while held
# @ makes it not output to console
bind +right_control @+cam_follow_player
bind -right_control @-cam_follow_player

# make semicolon look at object under pointer
# need to start line with > to stop <hovered> being evaluated,
# and exec to make it evaluate when activated
>bind semicolon exec cam_look_at <hovered>

# make period toggle object tracking, and comma set tracked object
bind period !cam_tracking
>bind comma exec cam_target <hovered>

# make right shift cycle through first 3 camera positions
alias next_camera add cam_load_zero 1 3
bind right_shift next_camera

# make some buttons to load specific camera positions
ui_button 1 600   0 cam_load 1
ui_button 2 600 -30 cam_load 2
ui_button 3 600 -60 cam_load 3
```


### Special characters

* `@` - Behaves as it does normally; prefix it to a command to silence that command.
* `@@` - Two `@` in a row silences the remainder of the script; each command will behave as if it had a `@` before it.  `@@` again will disable this effect.
* `#` - At the start of a line is used for comments; the line will be ignored.
* `:` - At the start of a line is used to specify a label, which may be skipped to with the `skip` command.
* `<` and `>` - These will be expanded by `exec` into `<<` and `>>`; this is so that you can set up aliases and bindings which refer to variables, without them being replaced at the point the script executes.  To suppress this behaviour, start the line with `>`.  You can see this used in the above `autoexec` in order to insert the `hovered` variable.


The `skip` command can be used inside a script to jump forward to a label.  It may not be used to jump backwards.  You may give it an optional `variable` and then further optional `comparison` and `value` parameters: if you do it will only skip if the variable is non-zero, or the result of the comparison is true.

Finally, the `exit` command will cause the script to stop executing.

Example `autoexec`:
``` bash
@@ # silence script


## Set up a `private_room` variable to govern whether server is private or public

# Create scripts for each mode

store_text private_game_settings
  host_name Members Only!
  host_password foobar
end private_game_settings

store_text public_game_settings
  host_name All Are Welcome!
  host_password ""
end public_game_settings

# create variable and assign scripts
store_toggle private_room
alias +private_room exec -q -v private_game_settings
alias -private_room exec -q -v public_game_settings

# bind to key, and set to private by default
bind KeypadMinus !private_room
+private_room


## Set up smart chat keybinding:
# Push to activate Team chat input if in team,
# or Game chat input if not.

store_text smart_chat
  skip :teamchat team
  chat_tab_game
  skip :activate
:teamchat
  chat_tab_team
:activate
  chat_input
end smart_chat

bind y @exec -v smart_chat


## Display GUID of currently held object,
# or if nothing held then currently hovered object.

store_text echo_guid_script
  skip :held grabbed
  >exec echo <hovered>
  exit
:held
  >exec echo <grabbed>
end echo_guid_script

alias echo_guid exec -q -v echo_guid_script
bind KeypadEnter echo_guid
```

## Some useful commands

As noted above, `help`, `commands`, and `variables` will let you find out everything you can do with the system console.  Having said that, here is a selection of some of the more useful commands available:

* `add`, `subtract`, and `multiply` will let you do simple arithmetic on a variable.  `add` is useful for cycling a modal variable (it has an optional third parameter which sets a modulus), while `subtract` subtracts the variable *from* the value, so is useful for ping-ponging between two numbers.
* `alias` will create a new name for another command, while retaining any parameters you type in. Good for making shorter names for commands you use a lot.  Use with `store_text` and `exec` to make your own commands from scripts.  Can also be used to attach commands to each value of a toggle variable, which will run when the variable is set to that value.  Finally, if there are a bunch of commands which share a prefix which you want to make short versions of you can use `*` to do so (e.g. `alias cam_* spectator_camera_*`)
* `bind`, `unbind` control attaching commands to keypresses.  You may use `+` and `-` before the keycode to specify if you want it to trigger on *press* or *release*, respectively. [(list of Unity keycodes)](https://docs.unity3d.com/ScriptReference/KeyCode.html)
* `broadcast` will broadcast the provided message.
* `chat_font_size` sets the size of the font in the chat / console window.
* `clear` will clear a text variable.
* `color` reports/sets your player color.
* `console_hotkey_lock` When enabled, locks whichever key is bound to toggling the system console, so that hitting it always toggles the console (this makes the key untypeable in text input boxes).
* `default_host_name` and `default_password` set those values.
* `dice_roll_height_multiplier` sets how high dice go when randomized.
* `displays` outputs information on currently connected monitors.
* `drawing_erase_all` erases all drawings.
* `drawing_render_fully_visible` will cause drawings to render fully in 3d space (which is neat if you have VR)
* `echo` displays its parameters in the system console.
* `edit` allow you to edit a text variable with the in-game GUI (you may also do this by passing the variable the `-e` parameter, e.g. `autoexec -e`)
* `escape` will display a text variable in the console, and will  escape all the formatting characters (i.e. all the `[` and `]` characters).
* `find` finds a component on the table.
* `grabbed`, `hovered` output the GUID of the component you are interacting with.
* `highlight` a component.
* `host_game` creates a table; you can specify single player, multiplayer, or hotseat.
* `host_name` and `host_password` set those values for current game.
* `last` is a special variable which holds the value returned from the most recent command.
* `mirror_all` will mirror all text displayed in every other chat tab into the system console.  This means you can always be in the system console without missing any messages.
* `quiet_mode` will, when enabled, stop command names being echoed in the console.  Unlike silencing commands with `@`, this will still display the commands' output.
* `reset` will reset a persistent variable to its default value.
* `sendkey` will emulate a keypress.  Primarily useful for binding things to VR controllers.
* `status` will display some key information about the current game. 
* `stats_monitor` will display some graphs/info which update in real time.
* `team` reports/sets your team.
* `ui_button` will add a button to the screen which performs a command when clicked.
* `ui_dialog_input` will display the text entry UI, and store the typed text in `last`.
* `ui_games_click` will click on a button on the game select UI, if it is open.  
* `ui_games_hide` will hide the game select UI without clicking on anything.
* `wait` is a command which can be inserted into a script to cause it to pause for some amount of time.  Useful for fudging a script to work with asynchronous commands which take some time to execute.


Commands are named with their topic first, so commands which affect the same part of the game have the same prefix.  The following are some useful groups of commands: get more info on them by running `help prefix_`.  For instance, `help camera_`

* `camera_` commands control the position and behaviour of the player camera (i.e. your point-of-view)
* `chat_` commands control the chat window.
* `component_` commands deal with the game components on the table.  Notably the `component_default_` commands let you specify what toggles new component are created with, which is useful if you are creating a lot of similar components.
* `errors_` commands handle how Lua errors are displayed.
* `jigsaw_` commands let you mess with jigsaws.
* `log_` commands control formatting of lua `log` calls.
* `mirror_` commands govern mirroring text from other tabs into the system console.
* `mod_` commands control various performance settings when loading mods.
* `say_` commands let you output messages to the chat channels.
* `spectator_` commands control the spectator view.
* `timestamp_` commands let you add timestamps to the chat channels / console.
* `tool_` commands let you check and set the currently used tool (grab, draw, etc).
* `ui_` commands deal with the game's User Interface.
* `vr_` commands perform all VR related tasks.
