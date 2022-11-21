---
layout: default
title: Filter Rules
---

# Filter Rules

Filter rules only have a trigger. If the rules fires, the event will be dropped, and the processing of the following rules will be discarded. Events that are dropped in a rule engine associated to a connector will not be send to kafka and will therefor never be seen at the corresponding connector. A rule that is dropped in a rule engine associated to an adapter will not be send to the adapters target. For example the Helix ITSM adapter would not create an incident for this event.


Examle:
```YAML
- name: FilterPatrolInformation
  triggers: _classname == 'patrolInformation'
  type: Filter
```

If the classname of the event is patrolInformation, the event is going to be dropped.