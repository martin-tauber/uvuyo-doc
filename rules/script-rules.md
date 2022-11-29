---
layout: default
title: Script Rules
---
Script rules allow you to create scripts to manipulate events. They are the most complex rule type, but also give you the most power. Script rules have a script which is executed when the trigger expression is true. The script takes the event that should be processed as an input.

Example:
```yaml

- name: LogEvent
  type: Script
  script: |
    // write event to log file in json format ...
    logger.info(Util.json(event));

    // set the default message for the event
    if ( _classname =~ "patrol.*" ) {
      event.details=`Event was send from origin '${event.patrolTrapOrigin}'.`;
    } else {
      event.details="we have nothing more to say ...";
    }
    
```

Script rules provide you with the full power of a scripting language as “if then else” 

The uvuyo gateway uses the Jexl engine as the driver for the event engine. 
Environment
