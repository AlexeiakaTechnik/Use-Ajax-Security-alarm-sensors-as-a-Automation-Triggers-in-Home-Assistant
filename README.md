[Insert intro image here]

# How to use Ajax Security Alarm sensors as an Automation triggers in Home Assistant.
_Article on my project on how to use AJAX Security Alarm System sensors(IR) as movement detectors in Home Assistant. How to connect AJAX to HA using SIA protocol(Security Industry Association). How to trace SIA events sent to HA server, make sens of it and utilize for HA automations(Examples with Light Automations)_

## Table of Contents
- [🚪 Introduction / Hook](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-introduction--hook)
- [🧠 Learning the AJAX System](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-learning-the-ajax-system)
- [🔍 Tracing SIA Events in Home Assistant](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-tracing-sia-events-in-home-assistant)
- [⚙️ Integrating with Home Assistant Automations](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-integrating-with-home-assistant-automations)
- [💡 Example Automations & Use Cases](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-example-automations--use-cases)
- [🛠️ Challenges & Lessons Learned](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#%EF%B8%8F-challenges--lessons-learned)
- [🌐 Conclusion & Bigger Picture](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-conclusion--bigger-picture)
- [📄 Appendix: YAML & SIA Configurations](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-appendix-yaml--sia-configurations)
- [🪪 License](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-license)
- [👨‍💻 Author and Inspiration](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-author-and-inspiration)
- [🔗 Related Projects & Resources](https://github.com/AlexeiakaTechnik/Use-Ajax-Security-alarm-sensors-as-a-Automation-Triggers-in-Home-Assistant/blob/main/README.md#-related-projects--resources)

---

### 🚪 Introduction / Hook               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Write an engaging intro about the closed nature of AJAX systems.  
Briefly recap your prior AJAX project (relay hack) and why this SIA automation was your next step.  
Mention why this method unlocks powerful automations for others. -->

---

### 🧠 Learning the AJAX System               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Describe the SIA Protocol briefly.  
Explain how AJAX uses it to send event notifications.  
Mention key event types: arm/disarm, intrusion, tamper, etc.  
Explain your goal: capture these SIA messages and turn them into Home Assistant automations. -->

---

### 🔍 Tracing SIA Events in Home Assistant               <sub>[⬆️ Back to Table of Contents](#table-of-contents)</sub>

<!-- Explain how to use Developer Tools → Events in HA to monitor SIA events on the Event Bus.  
Document example SIA event payloads you captured, including arm/disarm/intrusion events.  
Map the key fields in the event payload and explain their meaning. -->

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
