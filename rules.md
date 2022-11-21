---
layout: default
title: About
---

# Rules

The rule engine is used to process events. Every endpoint (connector or adapter) has its own rule engine. The rule engine associated with the adapter contains rules to normalize events. This means that it sets the events to a uvuyo common event format connectors can rely on. This allows it in more complex environments to send events from one connector to multiple adapters of different type. 

A rule engine loads a list of modules. Every module contains a list of event rules. An event rule usually has a trigger. A trigger is a Boolean expression. If the expression is true the rule is fired, If the expression is false the rule is ignored. If a rule does not have a trigger, it is always fired.

Rules are triggered in the order they are loaded. The rules are loaded module by module and within the module rule by rule. A rule in a module loaded later in the list will be executed after a rule of a module loaded earlier in the list. A rule following another rule in the same module will be executed after that rule.

The rule engine supports three types of rules. The filter rule, the map rule and the script rule.
