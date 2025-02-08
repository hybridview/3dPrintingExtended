

# 3dPrintingExtended

```
#####################################################################
#  Preparation 
#####################################################################
# copy this file in the same directory as your printer.cfg
# add 
#    [include pressure_advance.cfg]
# to your printer.cfg
#
# A [save_variables] block is needed since a printer save variable needs to be used to have it available after power up.
# You can skip this if you already have an [save_variables] config block
# e.g: 
#    [save_variables]
#    filename: /home/pi/klipper_config/.variables.stb
# I like to hide that file as there is nothing in that should be modified by the user.
# Do a klipper restart after adding the stuff above


[gcode_macro PRESSURE_ADVANCE_LIST]
description: List all filament pressure advance settings 
gcode:
  ## {% if not printer.save_variables.variables.pressure_advance %}
    {action_respond_info("PRESSURE ADVANCE: No filament defined ABORDED")}
  {% else %}
    {% set pa_dic = printer.save_variables.variables.pressure_advance %}
    {% set out = ["PRESSURE ADVANCE: Defined filaments"] %}
    {% for filament in pa_dic|sort(attribute='id') %}
      {% set _dummy = out.append("%s" % filament.id) %}
      {% for setup in filament.val|sort(attribute='nozzle') %}
        {% set _dummy = out.append("Nozzle: %1.02f | Pressure Advance: %1.03f | Smooth Time: %1.03f" % 
           (setup.nozzle, setup.pa, setup.st)) %}
      {% endfor %}
    {% endfor %}
    {action_respond_info(out|join("\n"))}
  {% endif %}

[gcode_macro PRESSURE_ADVANCE_ADD]
description: Add or change pressure advance settings
gcode:
  {% if 'FILAMENT' not in params|upper %}
    {action_respond_info("PRESSURE ADVANCE: FILAMENT must be defined use \"PRESSURE_ADVANCE_ADD FILAMENT=id\" as a minimum")}
  {% else %}
    {% set cfg = printer.configfile.settings.extruder %}
    {% set id = params.FILAMENT|string %}
    {% set nozzle = params.NOZZLE|default(0.40)|float|round(2) %}
    {% if not printer.save_variables.variables.pressure_advance %} # add first entry
      {action_respond_info("PRESSURE ADVANCE: Initialize with Filament %s" % (id))}
      {% set pa_dic = [{'id' : id, 
                        'val': [{'nozzle': nozzle, 
                        'pa'    : params.PRESSURE_ADVANCE|default(cfg.pressure_advance)|float|round(3), 
                        'st'    : params.SMOOTH_TIME|default(cfg.pressure_advance_smooth_time)|float|round(3)}]}] %}
    {% else %}
      {% set pa_dic = printer.save_variables.variables.pressure_advance %}
      {% for filament in pa_dic %}
        {% if id == filament.id %}
          {% set id_index = loop.index0 %}
          {% for setup in filament.val %}
            {% if nozzle == setup.nozzle %} # change value of an existing nozzle st an existing filament
              {% set change_txt = [] %}
              {% if 'PRESSURE_ADVANCE' in params|upper %}
                {% set _dummy = change_txt.append("PRESSURE ADVANCE") %}
                {% set _dummy = pa_dic[id_index].val[loop.index0].update({'pa': params.PRESSURE_ADVANCE|float|round(3)}) %}
              {% endif %}
              {% if 'SMOOTH_TIME' in params|upper %}
                {% set _dummy = change_txt.append("SMOOTH TIME") %}
                {% set _dummy = pa_dic[id_index].val[loop.index0].update({'st': params.SMOOTH_TIME|float|round(3)}) %}
              {% endif %}
              {% if change_txt|length > 0 %}
                {action_respond_info("PRESSURE ADVANCE: Changed %s at Filament %s Nozzle %s" % (change_txt|join(" and "),id,nozzle))}
              {% else %}
                {action_respond_info("PRESSURE ADVANCE: Nothing changed at Filament %s Nozzle %s" % (id,nozzle))}
              {% endif %}
            {% elif loop.last %} # add a new nozzle to an existing filament
              {action_respond_info("PRESSURE ADVANCE: Add setup for Nozzle %s at Filament %s" % (nozzle,id))}
              {% set _dummy = pa_dic[id_index].val.append({'nozzle': nozzle, 
                                                           'pa'    : params.PRESSURE_ADVANCE|default(cfg.pressure_advance)|float|round(3), 
                                                           'st'    : params.SMOOTH_TIME|default(cfg.pressure_advance_smooth_time)|float|round(3)}) %}
            {% endif%}
          {% endfor %}
        {% elif loop.last %} # add a new filament
          {action_respond_info("PRESSURE ADVANCE: Add setup for Filament %s" % (id))}
          {% set _dummy = pa_dic.append({'id' : id, 
                                         'val': [{'nozzle': nozzle, 
                                         'pa'    : params.PRESSURE_ADVANCE|default(cfg.pressure_advance)|float|round(3), 
                                         'st'    : params.SMOOTH_TIME|default(cfg.pressure_advance_smooth_time)|float|round(3)}]}) %}
        {% endif %}
      {% endfor %}
    {% endif %}
    ### SAVE_VARIABLE VARIABLE=pressure_advance VALUE="{pa_dic}"
  {% endif %}


.variables.stb

[Variables]
ercf = {'version': 1.2, 'static': {'unload_modifier': 9.0, 'min_temp_extruder': 180, 'extruder_eject_temp': 240, 'timeout_pause': 72000, 'disable_heater': 600}, 'clog_detection': False, 'endless_spool_mode': False, 'min_bowden_length': 750.0, 'selector': {'pos': [2.4, 24.0, 44.8, 71.2, 92.0, 113.6, 139.2, 160.8, 181.6], 'servo_angle': {'up': 30, 'down': 140}}, 'toolhead': {'end_of_bowden_to_sensor': 27.0, 'sensor_to_nozzle': 60.5}, 'calib': {'ref': 500, 'ratio': [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0]}}
filament_loaded = 'true'
filament_sensor = {'toolhead_runout': 0, 'runout': 1}
plates = {'array': [{'name': 'Mueller', 'offset': 0.0}, {'name': 'Energetics', 'offset': 0.0}, {'name': 'Texture', 'offset': -0.1}, {'name': 'En_Thick', 'offset': 0.0}], 'index': 0}
pressure_advance = [{'id': 'ESUN_ABS+_Black', 'val': [{'nozzle': 0.4, 'pa': 0.05, 'st': 0.04}, {'nozzle': 0.6, 'pa': 0.055, 'st': 0.04}]}, {'id': 'KVP_ABS_FL_Blue', 'val': [{'nozzle': 0.4, 'pa': 0.05, 'st': 0.04}]}]
print_stats = {'filament': 2779399.9760404243, 'time': {'filter': 99334, 'total': 3274949, 'service': 965487}}

```