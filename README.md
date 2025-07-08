[Insert intro image here]

# How to use Ajax Security Alarm sensors as an Automation triggers in Home Assistant.
_Article on my project on how to use AJAX Security Alarm System sensors(IR) as movement detectors in Home Assistant. How to connect AJAX to HA using SIA protocol(Security Industry Association). How to trace SIA events sent to HA server, make sens of it and utilize for HA automations(Examples with Light Automations)_

## Table of Contents
- [🚪 Introduction / Hook](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-integrating-with-home-assistant-automations---------------%EF%B8%8F-back-to-table-of-contents)
- [🧠 Learning the AJAX System](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-learning-the-ajax-system---------------%EF%B8%8F-back-to-table-of-contents)
- [🔍 Tracing SIA Events in Home Assistant](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-tracing-sia-events-in-home-assistant---------------%EF%B8%8F-back-to-table-of-contents)
- [⚙️ Integrating with Home Assistant Automations](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-integrating-with-home-assistant-automations---------------%EF%B8%8F-back-to-table-of-contents)
- [💡 Example Automations & Use Cases](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-example-automations--use-cases---------------%EF%B8%8F-back-to-table-of-contents)
- [🛠️ Challenges & Lessons Learned](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-challenges--lessons-learned---------------%EF%B8%8F-back-to-table-of-contents)
- [🌐 Conclusion & Bigger Picture](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-conclusion--bigger-picture---------------%EF%B8%8F-back-to-table-of-contents)
- [📄 Appendix: YAML & SIA Configurations](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-appendix-yaml--sia-configurations---------------%EF%B8%8F-back-to-table-of-contents)
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

Example event payload (for illustration):
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

---

### ⚙️ Integrating with Home Assistant Automations               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Show how to build automations based on captured SIA events. Explain your YAML structure step by step. Include a minimal, clean automation example with clear comments. -->

Example Automation - YAML

```yaml
alias: 'Intrusion Alarm Lights On'
trigger:
  - platform: event
    event_type: sia_event
    event_data:
      code: 'BA'
action:
  - service: light.turn_on
    target:
      entity_id: light.entryway_lights
mode: single
```

---

### 💡 Example Automations & Use Cases               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Share your actual real-life automation use cases with AJAX + Home Assistant. Possible examples: - Auto-lights on intrusion - Notification to phone when alarm armed/disarmed - Mode switching based on alarm state Keep it clean and to the point. --> <!-- Also mention you’ve documented dashboard integrations elsewhere, with links to your dashboard article. -->

---

### 🛠️ Challenges & Lessons Learned               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Document the common problems you faced: - Event noise / duplicate events - Timing issues / latency - Failsafe considerations - How you solved or mitigated them -->

---

### 🌐 Conclusion & Bigger Picture               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Reflect on why this method matters, not only for home users but also in industrial or legacy system integrations. Reinforce your key message: "Sometimes you don’t need official APIs—you just need patience, observation, and smart automation." -->

---

### 📄 Appendix: YAML & SIA Configurations               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>
<!-- Include your full working SIA integration YAML here (with sensitive info redacted). Provide several automation snippets as ready-to-use examples. -->

📄 Example SIA Configuration:

```yaml
sia_accounts:
  - account: '1234'
    encryption_key: 'ABCDEF1234567890ABCDEF1234567890'
    ping_interval: 3600
    allowed_time_diff: 30
```

📄 Example Automation:

```yaml
alias: 'AJAX System Armed Notification'
trigger:
  - platform: event
    event_type: sia_event
    event_data:
      code: 'CL'
action:
  - service: notify.mobile_app
    data:
      message: 'AJAX Security System Armed'
mode: single
```

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
