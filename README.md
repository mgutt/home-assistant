Der folgende YAML-Code saldiert den Ertrag / Verbrauch eines 3-phasigen Stromzählers, der mit einem Shelly 3EM gemessen wird.

Hinweis: Alle "shelly3main" sind entsprechend gegen den von dir gewählten Sensornamen zu ersetzen:

```
  - trigger:
      - platform: time_pattern
        minutes: "/1"
        seconds: "59"
    sensor:
    - name: Netzbezug
      unique_id: netzbezug
      unit_of_measurement: "kWh"
      state: >-
        {%- set keep_last_state = true -%}
        {#- on reboot some sensors do not return a number (instead they return "unknown" or "unavailable") -#}
        {%- if is_number(states("sensor.phasenbezug")) and is_number(states("sensor.phaseneinspeisung")) -%}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy")) and is_number(states("sensor.shelly3main_channel_b_energy")) and is_number(states("sensor.shelly3main_channel_c_energy")) -%}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy_returned")) and is_number(states("sensor.shelly3main_channel_b_energy_returned")) and is_number(states("sensor.shelly3main_channel_c_energy_returned")) -%}
          {#- obtain balanced energy value of household -#}
          {%- set phases_current = (states("sensor.shelly3main_channel_a_energy") | float + states("sensor.shelly3main_channel_b_energy") | float + states("sensor.shelly3main_channel_c_energy") | float) | round(2) -%}
          {%- set phases_diff = (phases_current - states("sensor.phasenbezug") | float) | round(2) -%}
          {%- set phases_returned_current = (states("sensor.shelly3main_channel_a_energy_returned") | float + states("sensor.shelly3main_channel_b_energy_returned") | float + states("sensor.shelly3main_channel_c_energy_returned") | float) | round(2) -%}
          {%- set phases_returned_diff = (phases_returned_current - states("sensor.phaseneinspeisung") | float) | round(2) -%}
          {%- set phases_balance = (phases_diff - phases_returned_diff) | round(2) -%}
          {#- update this sensor only if balance is positive -#}
          {%- if phases_balance > 0 -%}
            {{ (states("sensor.netzbezug") | float(0) + phases_balance | abs) | round(2) }}
            {%- set keep_last_state = false -%}
          {%- endif -%}
        {%- endif -%}
        {%- endif -%}
        {%- endif -%}
        {%- if keep_last_state -%}{{ states("sensor.netzbezug") | float(0) | round(2) }}{%- endif -%}
      device_class: energy
      state_class: total_increasing
  - trigger:
      - platform: time_pattern
        minutes: "/1"
        seconds: "59"
    sensor:
    - name: Netzeinspeisung
      unique_id: netzeinspeisung
      unit_of_measurement: 'kWh'
      state: >-
        {%- set keep_last_state = true -%}
        {#- on reboot some sensors do not return a number (instead they return "unknown" or "unavailable") -#}
        {%- if is_number(states("sensor.phasenbezug")) and is_number(states("sensor.phaseneinspeisung")) -%}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy")) and is_number(states("sensor.shelly3main_channel_b_energy")) and is_number(states("sensor.shelly3main_channel_c_energy")) -%}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy_returned")) and is_number(states("sensor.shelly3main_channel_b_energy_returned")) and is_number(states("sensor.shelly3main_channel_c_energy_returned")) -%}
          {#- obtain balanced energy value of household -#}
          {%- set phases_current = (states("sensor.shelly3main_channel_a_energy") | float + states("sensor.shelly3main_channel_b_energy") | float + states("sensor.shelly3main_channel_c_energy") | float) | round(2) -%}
          {%- set phases_diff = (phases_current - states("sensor.phasenbezug") | float) | round(2) -%}
          {%- set phases_returned_current = (states("sensor.shelly3main_channel_a_energy_returned") | float + states("sensor.shelly3main_channel_b_energy_returned") | float + states("sensor.shelly3main_channel_c_energy_returned") | float) | round(2) -%}
          {%- set phases_returned_diff = (phases_returned_current - states("sensor.phaseneinspeisung") | float) | round(2) -%}
          {%- set phases_balance = (phases_diff - phases_returned_diff) | round(2) -%}
          {#- update this sensor only if balance is negative -#}
          {%- if phases_balance < 0 -%}
            {{ (states("sensor.netzeinspeisung") | float(0) + phases_balance | abs) | round(2) }}
            {%- set keep_last_state = false -%}
          {%- endif -%}
        {%- endif -%}
        {%- endif -%}
        {%- endif -%}
        {%- if keep_last_state -%}{{ states("sensor.netzeinspeisung") | float(0) | round(2) }}{%- endif -%}
      device_class: energy
      state_class: total_increasing
  - trigger:
      - platform: time_pattern
        minutes: "/1"
        seconds: "0"
    sensor:
    - name: Phasenbezug
      unique_id: phasenbezug
      unit_of_measurement: "kWh"
      state: >-
        {#- on reboot some sensors do not return a number (instead they return "unknown" or "unavailable") -#}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy")) and is_number(states("sensor.shelly3main_channel_b_energy")) and is_number(states("sensor.shelly3main_channel_c_energy")) -%}
          {{ (states("sensor.shelly3main_channel_a_energy") | float + states("sensor.shelly3main_channel_b_energy") | float + states("sensor.shelly3main_channel_c_energy") | float) | round(2) }}
        {%- else -%}
          unavailable
        {%- endif -%}
      device_class: energy
      state_class: total_increasing
  - trigger:
      - platform: time_pattern
        minutes: "/1"
        seconds: "0"
    sensor:
    - name: Phaseneinspeisung
      unique_id: phaseneinspeisung
      unit_of_measurement: "kWh"
      state: >-
        {#- on reboot some sensors do not return a number (instead they return "unknown" or "unavailable") -#}
        {%- if is_number(states("sensor.shelly3main_channel_a_energy_returned")) and is_number(states("sensor.shelly3main_channel_b_energy_returned")) and is_number(states("sensor.shelly3main_channel_c_energy_returned")) -%}
          {{ (states("sensor.shelly3main_channel_a_energy_returned") | float + states("sensor.shelly3main_channel_b_energy_returned") | float + states("sensor.shelly3main_channel_c_energy_returned") | float) | round(2) }}
        {%- else -%}
          unavailable
        {%- endif -%}
      device_class: energy
      state_class: total_increasing
  - trigger:
      - platform: time_pattern
        seconds: "/30"
    sensor:
    - name: Netzleistung
      unique_id: netzleistung
      unit_of_measurement: 'W'
      state: >-
        {#- on reboot some sensors do not return a number (instead they return "unknown" or "unavailable") -#}
        {%- if is_number(states("sensor.shelly3main_channel_a_power")) and is_number(states("sensor.shelly3main_channel_b_power")) and is_number(states("sensor.shelly3main_channel_c_power")) -%}
          {{ (states("sensor.shelly3main_channel_a_power") | float + states("sensor.shelly3main_channel_b_power") | float + states("sensor.shelly3main_channel_c_power") | float) | round(2) }}
        {%- else -%}
          unavailable
        {%- endif -%}
      device_class: power
      state_class: measurement
```

Es hat lange gebraucht bis der Code das gewünschte Ergebnis lieferte. Insbesondere der Else-Teil, der ein "unavailable" zurückgibt, wenn der Sensor nicht erreichbar ist, hat geholfen, dass das Werte bei einem Neustart von Home Assistant nicht einfach genullt werden.
