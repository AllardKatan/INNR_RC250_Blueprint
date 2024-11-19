# INNR_RC250_Blueprint
This blueprint creates an automation that translates the ZHA events which are sent by the INNR RC250 5 button remote (https://www.innr.com/en/product/remote-control/) to actions.


## Prerequisites
- The remote has to be paired with a Zigbee controller using the ZHA integration.
- Due to the use of modern syntax ('actions', 'triggers'), 2024.10 is the minimum HA core version supproted
- An input text helper unique to the remote device should be present. The remote has 5 buttons, which generate commands that are not always straightforward to use (e.g., the same command could be generated by a left or right arrow press). This blueprint uses a text helper to be able to distinguish those commands. You can create one in the 'helpers' tab of the 'devices &services' settings menu. See also https://www.home-assistant.io/integrations/input_text/ . Don't re-use the same text helper for different devices as that will lead to undesired results. If you make different automations with the same device you only need one helper.

## Configuration
The blueprint has 3 different modes of operation, of which the second and third are primarily designed to control lights:
- The first mode is the most versatile, it allows you to create an action for each type of button press. This can be any action you want. Any button you do not configure will trigger the automation but it won't send any action. The long presses can be assigned to actions but there is no looping, only a single instance of an action is done on a trigger. If you want looping here, you can make scripts that implement the looping and start/stop the scripts with the actions.
- The second mode stays close to the built-in way the remote works. It allows you to cycle through a fixed list of five scenes with the left and right buttons. The scenes can be any kind of scene, they don't need to be light scenes. The up and down buttons will increase or decrease brightness of one or more lights, the on/off button will turn the ligth(s) on and off. The scenes and light(s) can be selected in the blueprint dialog, they must be specified as entities. A seperate section for configuring a few parameters is present in the blueprint. 
- The third mode also controls one or more light(s). Left and right buttons will increase or decrease the color of this light. Whether 'color' refers to the color temperature (in K) of the light or the hue, is selectable. If you have multiple lights selected, all lights get sent the same commands, so mixing RGB (color) and CCT (warm/cool white) lights is not supported at the moment. Short presses will make jumps of color or brightness as set in the settings. The long presses will make smaller jumps (1/4 of the setting) every 1/8 of a second. After five seconds of holding, the automation will automatically stop, to prevent going into an infinite loop when the release command is missed.


## Background information: Buttons and ZHA commands
There are five buttons on the remote. Here is what they will send to ZHA (simplified explanation!):
1. The central button, with a 'power' icon on it, will send two different commands. Each time you press it, it will toggle between sending an 'on' command and an 'off' command. The toggling of commands is done by the remote, and independent of any actions you link in the automation. Long pressing (i.e. holding) or double pressing does not send different commands.
2. The 'up' and 'down' buttons, with sunshaped brightness control icons on them. When short pressed, they will give a command. These commands are specific to the button, i.e. from the command it is clear which button was pressed. You can also hold them. In that case, they send a command as soon as it is recognized that you are holding the button. Those commands are also specific to the button. When you release the button, another command is sent to signal the release. The release command is the same, independent of which button you release.
3. The 'left' and 'right' buttons, with arrows. Short-pressing these will cycle through a list of five commands that differ only in a number parameter. Which number is sent is tracked by the remote. The command will not tell you which button was pressed, this can only be derived by comparing it to the previous time this command was sent. The behavior on long-pressing is analog to the up and down buttons, in this case there are specific commands for each button, and again the release command is shared between the buttons, but distinct from the up-down release.
4. The up, down, left and right buttons actually send two separate ZHA events when pressed, with slightly different information. Only one event per press is used by the automation.

