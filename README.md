![cover](https://github.com/user-attachments/assets/b01e486e-b233-4ca1-9604-9214e8e910d3)

# How to use Ajax Security Alarm sensors as an Automation triggers in Home Assistant.
_Article on my project on how to use AJAX Security Alarm System sensors(IR) as movement detectors in Home Assistant. How to connect AJAX to HA using SIA protocol(Security Industry Association). How to trace SIA events sent to HA server, make sens of it and utilize for HA automations(Examples with Light Automations)_

## Table of Contents
- [🚪 Introduction / Hook](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-integrating-with-home-assistant-automations---------------%EF%B8%8F-back-to-table-of-contents)
- [🧠 Learning the AJAX System](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-learning-the-ajax-system---------------%EF%B8%8F-back-to-table-of-contents)
- [🔍 Tracing SIA Events in Home Assistant](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-tracing-sia-events-in-home-assistant---------------%EF%B8%8F-back-to-table-of-contents)
- [⚙️ Integrating with Home Assistant Automations](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-integrating-with-home-assistant-automations---------------%EF%B8%8F-back-to-table-of-contents)
- [💡 Example Automations & Use Cases](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-example-automations--use-cases---------------%EF%B8%8F-back-to-table-of-contents)
- [🌐 Conclusion & Bigger Picture](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-conclusion--bigger-picture---------------%EF%B8%8F-back-to-table-of-contents)
- [🪪 License](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-license---------------%EF%B8%8F-back-to-table-of-contents)
- [👨‍💻 Author and Inspiration](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-author-and-inspiration---------------%EF%B8%8F-back-to-table-of-contents)
- [🔗 Related Projects & Resources](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-related-projects--resources---------------%EF%B8%8F-back-to-table-of-contents)

---

### 🚪 Introduction / Hook               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Write an engaging intro about the closed nature of AJAX systems.  
Briefly recap your prior AJAX project (relay hack) and why this SIA automation was your next step. Mention why this can be useful to save up on extra IR/Motion sensors and how home security alarms are usually on standby doing nothing 80% of time. -->

Modern home security systems, like AJAX, are typically designed as **closed ecosystems**. They work great out of the box, but if you want to integrate them into broader smart home automations — [good luck with that](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant/blob/main/README.md#-why-ajax-doesnt-just-integrate)!

I’ve already tackled AJAX control via **relay hacking** in my previous project, heavily reffered to in this one, where I did a bit of reverse-engineering and wiring into the SpaceControl key fob to allow Arming and Disarming AJAX via Home Assistant. That solved the “control” problem — but I had another idea after that.

This article covers the next step: **harnessing the SIA Protocol** to make AJAX’s internal events fully visible and usable in Home Assistant. We will not cover [AJAX configuration](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant/blob/main/README.md#%EF%B8%8F-ajax-system---devices-groups-and-sia-monitoring-station-setup) or [how to set up SIA Integration](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant/blob/main/README.md#-sia-alarm-systems-configuration), assuming you have already done that yourself or with the help of my guides.

_Why bother?_ Because most of the time(in many cases, when at least someone is at home/in office/on site), security systems sit idle — **passively waiting to be used**. Meanwhile, those same sensors (motion detectors, door sensors, IR/cam triggers) could be put to work for:

- **Lighting control**
- **Presence-based automations**
- **Notifications or routines**

This approach allows you to maximize value from your existing security gear — **without buying extra door/window or IR motion sensors** — by using the system’s SIA event reporting as a trigger source. And regarding battery drain - from my > 3 year experience, devices that are active even 24/7 **do not drain energy as quick as I expected them to**, I actually barely noticed any difference. 

<sub>_Side note_, what actually does drain battery - is the case when your device is far away/behind thick walls from the HUB, and then **a lot** of energy goes to radio to keep up reporting status/establish and re-establish connections(also learned from experience).</sub>

---

### 🧠 Learning the AJAX System               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Describe the SIA Protocol briefly.  
Explain how AJAX uses it to send event notifications.  
Mention key event types: arm/disarm, intrusion, tamper, etc.  
Explain your goal: capture these SIA messages and turn them into Home Assistant automations. -->

AJAX Systems, like many professional-grade security solutions, use the **SIA Protocol** (Security Industry Association DC-09 Standard) to report events to monitoring stations or integrations.

In simple terms, AJAX sends out **SIA event messages** whenever something important happens:
- **Arming/Disarming** (System Armed, Disarmed, Partial Arm, etc.)
- **Intrusion Detection** (Motion detected, Door opened, etc.)
- **Tamper Events** (Device opened or tampered with)
- **System Health Checks** (Battery low, signal lost, etc.)

These messages can be intercepted by Home Assistant using its built-in **SIA Integration**.  
Once decoded, you can easily detect events like:
- Who armed or disarmed the system
- Whether an intrusion occurred
- If a specific sensor triggered an alarm

**My Goal:** Capture these SIA events inside Home Assistant and use them as automation triggers—turning AJAX from a passive security system into a powerful part of my smart home ecosystem.


---

### 🔍 Tracing SIA Events in Home Assistant               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Briefly explain HA Architecture(event bus conveyor which gathers and uses events to automate things). Explain how to use Developer Tools → Events in HA to monitor SIA events on the Event Bus.  
Document example SIA event payloads you captured, including arm/disarm/intrusion events.  
Map the key fields in the event payload and explain their meaning. Refer to SIA Codes document.   -->

To automate based on SIA events, you first need to **see them**.

In Home Assistant, everything revolves around the **[Event Bus](https://www.home-assistant.io/docs/configuration/events/)** — a sort of internal “conveyor belt” where all system events flow, and Integrations can fire their own custom events.  
It’s the **Core**—the beating heart of the Home Assistant Operating System.

If you go and listen to it using Developer Tools (as explained below) and simply monitor the most common event type — `state_changed` — you’ll notice something interesting:
- In a **fresh, young Home Assistant install**, it’s like a slow, peaceful heartbeat—some time-based triggers here and there, sun position updates, maybe a device checking in now and then.
- But in a **seasoned, mature, complex setup** — with 100+ devices and 100+ automations — it **roars**. Even at idle, it can easily hit **90+ events per minute**, just ticking over in its resting state. And when a cascade of actions is triggered…  
You’ll feel the storm.

Anyway, **Automations** listen to this Event Bus to act when specific conditions are met.

To inspect incoming SIA events:
1. Go to **Developer Tools → Events** in Home Assistant.
2. In the **Listen to Events** box, enter:
   ```plaintext
   sia_event_<port>_<account>
   ```
   Where port is the port and account is Account ID(Object number) set up in AJAX APP Monitoring station config, see [AJAX](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant/blob/main/README.md#%EF%B8%8F-ajax-system---devices-groups-and-sia-monitoring-station-setup) and [SIA](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant/blob/main/README.md#-sia-alarm-systems-configuration) configuration.
3. Click Start Listening.
4. Arm, disarm, or trigger your AJAX detectors(wave hands, open doors) and watch the events appear in real-time.

SIA Event structure and payload is described in [HA Documentation](https://www.home-assistant.io/integrations/sia/). Here are a few examples with comments:

Example of event payload (simplified for illustration)

<details>
<summary>📄 Example of event payload (simplified for illustration) (Click to Expand)</summary>

```yaml
{
  "event_type": "sia_event",
  "data": {
    "zone": "IR_Sensor_LivingRoom",
    "code": "BA",
    "message": "Burglary Alarm",
    "timestamp": "2025-07-08T14:53:00"
  }
}

```

</details>


Event payload from my HA/AJAX setup with comments:

<details>
<summary>📄 Event payload from my HA/AJAX setup with comments: (Click to Expand)</summary>

```yaml
{
event_type: sia_event_xxx_AAA34 ##where xxx is my port and AAA34 - user id
data:
  message_type: SIA-DCS ##Type of SIA message (usually "SIA-DCS" for standard events).
  receiver: null ##Receiver ID — usually the port or system receiving the event, not important
  line: L0 ##Communication line/channel (often unused in simple setups). All my events are L0
  account: AAA34 ##obvious
  sequence: "5277" ##	Message sequence number (for ordering).
  content: "#AAA34|Nri8/BA15]_12:06:34,07-08-2025" ##	Full raw SIA message content (undecoded text, rarely needed in automations).
  ti: null ##	Transmission Identifier—technical field from SIA message (rarely needed).
  id: null ## Identifier field from SIA message (may duplicate account or zone in some cases).
  ri: "8" ## THIS is important! Zone or sensor triggering the event. Critical for identifying which device caused the event - zones(Groups) are configured in AJAX App
  code: BA ## SIA Event Code (e.g., "BA" for Burglary Alarm, "CL" for Closing/Arming). Key for automations. Link to doc in article at the end.
  message: "15" ## Human-readable description of the event (mapped from code, like "Burglary Alarm").
  x_data: null ## Extended data included in some SIA messages (usually advanced or manufacturer-specific details).
  timestamp: "2025-07-08T12:06:34+00:00" ## 	Time of the event.
  event_qualifier: null ## SIA Qualifier (like "new event" vs. "restore")—used in some events for extra context.
  event_type: null ##Same as *code* in some cases; sometimes more generalized type info.
  partition: null ## Partition number—relevant for multi-partition systems (often unused in basic setups).
  extended_data: null ## Additional extended data (structured, list format)—rare in most home security SIA events.
  sia_code: ## This one is new, it used to be just the BA/BR/etc. codes, but now its explained! cool!
    code: BA
    type: Burglary Alarm
    description: Burglary zone has been violated while armed
    concerns: Zone or point ## What this event concerns (e.g., specific zone, device, or system-wide event). Useful for filtering.
origin: LOCAL ## Where the event originated. "LOCAL" means it was triggered from within Home Assistant itself (versus cloud or external sources).
time_fired: "2025-07-08T12:06:14.503616+00:00" ## Exact timestamp (UTC) when the event was fired inside Home Assistant.
context: ## Standard HA context object for event tracking, includes:
  id: 01JZMYVW77S9JC3NY8TWJX72Z1 ## Unique ID for this specific event in HA’s logs (useful for debugging and linking events). 
  parent_id: null ## ID of the parent event (if this event was triggered by another event/automation).
  user_id: null ## ID of the user (if applicable) who triggered the event (usually null for automated events).
}

```


</details>



By knowing this we can utilize these events to automate some things, like ligthting, for example.

---

### ⚙️ Integrating with Home Assistant Automations               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Show how to build automations based on captured SIA events. Explain your YAML structure step by step. Include a minimal, clean automation example with clear comments. -->

To use SIA events in HA Automations we need basic things from the payload, described in YAML example above: 
```plaintext
code
```
and 
```plaintext
ti
``` 

Where **code** gives us a clue of what kind of event was it and **ri** - what Zone/Group of Devices(room/space) this event originated in. See [SIA Codes document](https://docs.google.com/spreadsheets/d/1-N-RZVS8IiwM5zuw2u4gt8Bx_5xo_JOwuagHJgSJxUw/edit#gid=920971512) for description of alarm codes. Lets look at the example of simple Light Automation using those:


<details>
<summary>📄 Example of simple Light Automation (Click to Expand)</summary>

```yaml
alias: 'Room X Lights On for 1m30s' 
triggers:
  - platform: event
    event_type: sia_event ## set event type
    event_data:
      code: BA ## this is fired when AJAX MotionProtect device is armed(24/7 in our setup) and detects new motion/loud sound event
      ri: "8" ## this is Group(Zone) which AJAX MotionProtect device is attached to
conditions:
  - condition: state
    entity_id: light.room_x_lights 
    state: "off" ## lets not fire automation when light is already On in the room X
actions:
  - action: light.turn_on ##turn on the light
    target:
      entity_id: light.room_x_lights
  - delay:
       hours: 0
       minutes: 1
       seconds: 30 ## delay for 1m30s
  - action: light.turn_off
    target:
      entity_id: light.room_x_lights ## and turn off the light
mode: single
```

</details>


This is the simplest implementation of light automation and it WILL have a lot of issues in a real-world use-case scenario. We can expand and improve the automation from here to cover a lot of different scenarios - please check out my [Light Automations in Smart Home](https://github.com/AlexeiakaTechnik/My-experience-of-improving-Light-Automations-in-Home-Assistant) article on how to do it! 

---

### 💡 Example Automations & Use Cases               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Share your actual real-life automation use cases with AJAX + Home Assistant. Possible examples: - Auto-lights on intrusion - Notification to phone when alarm armed/disarmed - Mode switching based on alarm state Keep it clean and to the point. --> <!-- Also mention you’ve documented dashboard integrations elsewhere, with links to your dashboard article. -->

To expand usage of our AJAX system I have preapared a few examples of automations. Some of them may or may not be used by me in my setup at the moment. Those are just obvious examples, but you can integrate AJAX devices in your smart home setup however you like! Just treat them as individual/grouped(depends on group setup in AJAX) sensors with a number of states. Like binary door/windows sensors for DoorProtect or motion detectors for MotionProtect.

   * **Example 1 - ✨ Smart TTS Welcome Automation (with multi-sensor verification)**: 
   Assumes we have an Entryway with:
   - Smart speaker(media player)
   - AJAX DoorProtect sensor on the door in Group 1(ri: 7)
   - AJAX MotionProtect sensor inside the room in Group 2(ri: 8)

   This Automation will check if Entryway door was opened(BA code fired) and wait for Entryway IR sensor to also fire BA(someone has entered room, not just opened the door), and send TTS Message to speaker afterwards.

<details>
<summary>📄 Lets take a look at YAML(see comments): (Click to Expand)</summary>

```yaml
alias: 'Entryway Smart Welcome - Door & Motion Confirmed'
mode: single  ## Prevents overlapping executions
triggers: 
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BA' ## Burglary Alarm
      ri: 7  ## Group 1 with Entryway DoorProtect sensor (Door opened)
    id: door_opened ##lets use trigger IDs for conditions
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BA'
      ri: 8  ## Group 2 with Entryway MotionProtect sensor (Motion detected)
    id: motion_detected
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger ##only DoorProtect sensor can start automation
            id: door_opened 
        sequence: 
          - wait_for_trigger: ## this we will wait 60s for MotionProtect to also send BA code
              - platform: event
                event_type: sia_event
                event_data:
                  code: 'BA'
                  ri: 8
            timeout:
              seconds: 60
            continue_on_timeout: false
          - service: tts.speak
            metadata: {}
            data:
              cache: true ##lets add tts message to cache since it will be used a lot
              media_player_entity_id: media_player.entryway_speaker
              message: "Entryway door opened. Welcome home, brave traveller!"
            target:
              entity_id: tts.google_en_com ##I have learned that it's best to use googles tts
```

</details>
   
   * **Example 2 - ✨ HVAC Automatic Shutdown and Restart on Door/Window opened&closed**: 
   Assumes we have a room with Door and/or Window:
   - Smart speaker(media player)
   - HVAC System(air conditioner or climate control device(
   - AJAX DoorProtect sensor on the door in Group 1(ri: 7)
   - AJAX DoorProtect sensor on the window in Group 1(ri: 7)

   This Automation will check if door was opened(BA code fired) and wait for 5 minutes, if door was not closed in 5 minutes - the HVAC will shut down. If the door was closed and not opened again - the HVAC will be turned back on. Text-To-Speech audio messages will be added for help.

<details>
<summary>📄 Lets take a look at YAML(see comments): (Click to Expand)</summary>

```yaml
alias: 'HVAC Auto Pause/Resume - Door & Window with Grace Periods'
mode: restart  ## Only one instance runs, cancels prior ones if retriggered
triggers:
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BA' ##Burglary Alarm
      ri: 7  ## DoorProtect - Window or Door opened
    id: door_opened
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BR' ##Alarm Restored
      ri: 7  ## DoorProtect - Window or Door closed
    id: door_closed
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger
            id: door_opened
        sequence:
          - delay: "00:05:00"  # Wait 5 minutes to check door/window state
          - wait_for_trigger:
              - platform: event
                event_type: sia_event
                event_data:
                  code: 'BR'
                  ri: 2  ## Door/Window closed
            timeout: "00:00:01"  # Immediately check after delay
            continue_on_timeout: true
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ wait.trigger is none }}" ##check if wait period expired
                sequence:
                  - service: climate.set_hvac_mode
                    target:
                      entity_id: climate.living_room
                    data:
                      hvac_mode: 'off'
                  - service: tts.speak
                    metadata: {}
                    data:
                      cache: true
                      media_player_entity_id: media_player.living_room_speaker
                      message: "HVAC paused. Door or Window remained open for over 5 minutes."
                    target:
                      entity_id: tts.google_en_com

      - conditions:
          - condition: trigger
            id: door_closed
        sequence:
          - delay: "00:05:00"  ## Grace period after door closes
          - wait_for_trigger:
              - platform: event
                event_type: sia_event
                event_data:
                  code: 'BA'
                  ri: 2  ## Door opened again
            timeout: "00:00:01"
            continue_on_timeout: true
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ wait.trigger is none }}"
                sequence:
                  - service: climate.set_hvac_mode
                    target:
                      entity_id: climate.living_room
                    data:
                      hvac_mode: 'heat'  ## Adjust mode as needed
                  - service: tts.speak
                    metadata: {}
                    data:
                      cache: true
                      media_player_entity_id: media_player.living_room_speaker
                      message: "HVAC resumed. Door stayed closed for 5 minutes."
                    target:
                      entity_id: tts.google_en_com

```

</details>
   

   * **Example 3 - ✨ Visual and Audio Indication of ALARM State**: 
   Assumes we have any AJAX Setup and Smart Home with:
   - Smart speaker(media player)
   - Ambient RGB LED Lights

   This Automation will indicate if Alarm is in Armed/Disarmed/in Night Mode sates via RGB LED entity, with TTS messages indicating change of alarm state. If Alarm is Triggered - flashing Lights and repeating TTS until Alarm is Disarmed.

<details>
<summary>📄 Lets take a look at YAML(see comments): (Click to Expand)</summary>

```yaml
alias: 'Alarm State Visual & TTS Indication (No Helpers)'
mode: restart  ## Prevent overlapping runs
trigger:
  - platform: event
    event_type: sia_event
    event_data:
      code: 'CL'  ## Armed Away
      ri: 10  ## Device group(any will do for this automation)
    id: armed_away
  - platform: event
    event_type: sia_event
    event_data:
      code: 'OP'  ## Disarmed
      ri: 10
    id: disarmed
  - platform: event
    event_type: sia_event
    event_data:
      code: 'NL'  ## Night Mode
      ri: 10
    id: night_mode
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BA'  ## Triggered
      ri: 10
    id: triggered
variables:
  last_state: "{{ state_attr(alarm_control_panel.room_x_pir_motion_sensor, 'last_state') | default('disarmed') }}" ## We use this variable to track last state of the alarm to start flashing lights only if it went from Armed Away/Night to Triggered
action:
  - choose:
      - conditions:
          - condition: trigger
            id: armed_away
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.ambient_rgb_led
            data:
              brightness_pct: 70
              rgb_color: [255, 0, 0] ## Red color
          - service: tts.speak
            metadata: {}
            data:
              cache: true
              media_player_entity_id: media_player.house_speaker
              message: "Security Alarm was Armed Away"
            target:
              entity_id: tts.google_en_com
          - service: automation.set_state ##Automation self-stores its previous state using the undocumented but common HA trick of setting its own state (via automation.set_state).
            data:
              entity_id: automation.alarm_state_visual_tts_indication_no_helpers
              state: "armed_away"
      - conditions:
          - condition: trigger
            id: disarmed
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.ambient_rgb_led
            data:
              brightness_pct: 50
              rgb_color: [0, 255, 0] ## Green color
          - service: tts.speak
            metadata: {}
            data:
              cache: true
              media_player_entity_id: media_player.house_speaker
              message: "Alarm was Disarmed"
            target:
              entity_id: tts.google_en_com
          - service: automation.set_state
            data:
              entity_id: automation.alarm_state_visual_tts_indication_no_helpers
              state: "disarmed"
      - conditions:
          - condition: trigger
            id: night_mode
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.ambient_rgb_led
            data:
              brightness_pct: 25
              rgb_color: [128, 0, 128]
          - service: tts.speak
            metadata: {}
            data:
              cache: true
              media_player_entity_id: media_player.house_speaker
              message: "Night Mode engaged for Security Alarm"
            target:
              entity_id: tts.google_en_com
          - service: automation.set_state
            data:
              entity_id: automation.alarm_state_visual_tts_indication_no_helpers
              state: "night_mode"

      - conditions:
          - condition: trigger
            id: triggered
          - condition: template
            value_template: >
              {{ last_state in ['armed_away', 'night_mode'] }} ## Check if last state variable is in Armed Away/Night mode
        sequence:
          - repeat: ## repeat sequence will give us flashing red light and TTS untill alarm is disarmed
              while:
                - condition: template
                  value_template: >
                    {{ last_state in ['armed_away', 'night_mode'] }}
              sequence:
                - service: light.turn_on
                  target:
                    entity_id: light.ambient_rgb_led
                  data:
                    brightness_pct: 100
                    rgb_color: [255, 0, 0]
                - delay: "00:00:03" ## Delays for turning on and off LEDs
                - service: light.turn_off
                  target:
                    entity_id: light.ambient_rgb_led
                - delay: "00:00:03" ##
                - service: tts.speak
                  metadata: {}
                  data:
                    cache: true
                    media_player_entity_id: media_player.house_speaker
                    message: "Security Alarm Triggered!"
                  target:
                    entity_id: tts.google_en_com
                - delay: "00:00:15" ## delay for repeating TTS 


```

</details>

---

### 🌐 Conclusion & Bigger Picture               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Reflect on why this method matters, not only for home users but also in industrial or legacy system integrations. Reinforce your key message: "Sometimes you don’t need official APIs—you just need patience, observation, and smart automation." -->

Primary purpose of this guide was to explain that in your Smart Home or Office/other site you can still use devices which are standing by a lot of time. This is a good practice for saving on Smart Home system cost and optimizing load on your system - you have already paid for Security Alarm, AJAX or other vendor, why not put it to greater use and claim maximized value? The logic of using the security system while in  standby/disarmed will not contradict security concerns since additional automations are not needed once there is no one in the building. But it's important to also understand that such devices are limited in this use-case and are purposely built for security, not daily automations. Dedicated Motion/Presence or Opening sensors will have many advantages, like being independant, not limited by groups, having greater integration flexibility and probably lower cost if compared 1:1 with AJAX or other security system sensors. 

Personally - I am very satisfied with my own home-lab setup, and I find it very convinient to use AJAX's MotionProtect sensors for Light Automations in areas less demanding of precision and comfort such as hallways, courtyards, stairs, etc. For living areas, kitchens, bedrooms such sensors are simply too hardware-limited, however sophisticated your automation logic is. 

I hope you will find this article helpful and insightful and maybe try this out with other vendors/brands, since SIA protocol is a very common thing for security industry! If you have any ideas, questions or suggestions - I would gladly be of help, just email me or create Github issue! Best of luck with improving and optimizing your Smart Home!


---

### 🪪 License               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

This project is licensed under the [MIT License](LICENSE).

---

### 👨‍💻 Author and Inspiration               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

Created by [Alexei](https://github.com/AlexeiakaTechnik) — field integrator, systems tinkerer, and smart home architect in the making. 

---

### 🔗 Related Projects & Resources               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

- ✅ [AJAX Security Integration In Home Assistant](https://github.com/AlexeiakaTechnik/AJAX_security-integration-in-Home_Assistant)  
- ✅ [My experience of improving Light Automations in Smart Home powered by Home Assistant](https://github.com/AlexeiakaTechnik/My-experience-of-improving-Light-Automations-in-Home-Assistant)
- ✅ [Practial and stylish Home Assistant Dashboards for Tablets and Mobile Phones](https://github.com/AlexeiakaTechnik/Practial-and-stylish-Home-Assistant-Dashboards-for-Tablets-and-Mobile-Phones)
- 📄 [AJAX SIA Event Code Reference (Google Sheet)](https://docs.google.com/spreadsheets/d/1-N-RZVS8IiwM5zuw2u4gt8Bx_5xo_JOwuagHJgSJxUw/edit#gid=920971512)
