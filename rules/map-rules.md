---
layout: default
title: Map Rules
---

Map rules contain a Map of expressions. An entry in the map is a property name of the event and an expression is going to be set for that property when the event triggers. One map rule can have multiple map entries.

Example:

```YAML
- name: UpdateMessageText
  triggers: _classname == 'patrolWarning' || _classname == 'patrolAlarm'
  type: Map
  classname: EVENT
  attributes:
    msg: patrolTrapText + ' ' + patrolTrapOrigin + ' ' + patrolTrapExtra
    serverity: "'critical'"
```

If the classname is patrolWarning or the classname is patrolAlarm the msg and the severity are going to be set.

The Map rule can have an attribute called classname, which will change the class of the event. In the example above the classname is going to be set to EVENT when the rule triggers. Be aware, that the following rules will now no longer trigger if they test on the classname which is not EVENT.

The expressions can be very simple from just assigning a constant or another slot value up to complex expressions. For more information see the chapter script language.
