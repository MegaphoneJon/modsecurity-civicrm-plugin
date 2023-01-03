# OWASP ModSecurity Core Rule Set - Template Plugin

The OWASP Core Rule Set (CRS) comes with a plugin structure that allows
to add official and third-party plugins to work with the existing
baseline CRS installation.

This repository contains a template-plugin that you can use as a base
to start writing your own plugin. And it illustrates the idea behind
the plugin mechanism.

The template plugin in this repository is an offical CRS plugin.

The CRS plugin documentation can be found on the [website](https://coreruleset.org/docs/configuring/plugins/).

## How do CRS plugins work?

Plugins are rules like ordinary rules and they integrate with an existing
CRS installation. Starting with CRS 4.0, there are four rules includes,
three of which are meant for plugins:

```
Include crs/crs-setup.conf

Include modsecurity.d/owasp-modsecurity-crs/plugins/*-config.conf
Include modsecurity.d/owasp-modsecurity-crs/plugins/*-before.conf

Include modsecurity.d/owasp-modsecurity-crs/rules/*.conf

Include modsecurity.d/owasp-modsecurity-crs/plugins/*-after.conf
```

The plugin-Folder in the standard CRS distribution is empty. Yet you can
copy your plugin files into that folder. Files with a filename of
the pattern *-before.conf are loaded before all CRS files are being
loaded in the first include line above. Then all the normal CRS
rules are loaded and then finally there is the third Include statement
that loads all the files which follow the filename pattern *-after.conf
from the plugins folder.

A plugin typically has the following files (explained with the
example of this template-plugin):

```
plugins/template-config.conf
plugins/template-before.conf
plugins/template-after.conf
```

All three files may contain CRS rules.

CRS plugins are enabled by default. Yet they can be disabled by setting a
variable. See below or in the template-plugin's config file for more information.

## What Rule IDs do the plugins have

Each CRS plugin gets a range of 1,000 rule IDs, from 9,500,000 onwards.
The template plugin runs from 9,500,000 to 9,500,999. There is a separate
plugin rule ID registration repo where new plugins can be registered.

https://github.com/coreruleset/plugin-registry

The rule range is meant to be used as follows:

* 9,5XX,000 - 9,5XX,099 : Initialization
* 9,5XX,100 - 9,5XX,499 : Request Rules
* 9,5XX,500 - 9,5XX,999 : Response Rules

## How does this Template Plugin work?

The Template Plugin has two active rules and three rules that are commented
out:

Rule 9500010 in template-config.conf checks for the existence of the
TX variable `template-plugin_enabled`. If the variable is not set,
it is being created and set to 0.
The rule is commented out. That means the plugin is enabled
by default. Uncomment this rule to disable the plugin.
This rule is mandatory for CRS plugins.

Rule 9500020 in template-config.conf sets the TX variable `template-plugin_foobar`
to 1. It is commented out. Uncomment it to set the variable.

Rule 9500100 in template-config.conf checks for the request parameter
template-plugin-test-parameter. If is present and not empty, then
a rule alert is triggered and the anomaly score is raised with
a severity CRITICAL score.

Rule 9500099 in template-before.conf checks for the 
TX variable `template-plugin_enabled`. If the variable is existing and it is
set to 0, then the subsequent rules of the plugin are removed for the
current transaction. This effectively disables the plugin for the
current transaction.
This rule is mandatory for CRS plugins. When writing your own plugins,
it is important to make sure the control statement in this rule removes
the right rule range.

Rule 9500100 in template-before.conf checks for the existence of the
request parameter `template-plugin-test-parameter`. If the parameter exists
and is non-empty, a critical alert is thrown.

Rule 9500500 in template-after.conf looks in the response body for a 
supposed token (that is probably not existing there).
This rule is commented out and serves as an example for a response rule.

## How to adopt the Template Plugin to your needs?

Feel free to take this template plugin and adopt it to your needs.

The following steps are necessary to create a new plugin that
follows the established standard for CRS plugins:

* Pick a plugin name. Example: template-plugin.
* Go to https://github.com/coreruleset/plugin-registry and look for an empty rule range.
* Rename the plugins files to the plugin-name you picked above.
* Write your rules.
* Adjust the rule IDs to fit the rule range you picked above.
* Make sure the control statement in 9500099 is correct.
* Test your rule set throughly, namely the interaction with CRS.
* Document your rule set. Please make sure that every rule has at
  least a brief description.
* Set the CAPEC tag for all the rules that are meant to detect
  attacks and log it.
* Publish your plugin rule set
* Register your plugin rule set at https://github.com/coreruleset/plugin-registry
  and enter the link to the website of your plugin; probably where you
  published it.

## Order of rules with plugins and existing CRS rules

The rules run in the order described in the following sections.

### Phase 1

As you know, ModSecurity rules can run in multiple phases. The 
engine will first execute all rules in phase 1, then all the
rules in phase 2. If a phase 2 rule is listed in a rules file
before a phase 1 rule, then the phase 1 rule is executed first.

* modsec-recommended-rules (rule id 200000 and friends)
* CRS configuration based on rules in crs-setup.conf
* plugins *-config-before.conf
* plugins *-before.conf
* CRS init rules in REQUEST-901-INITIALIZATION.conf
* CRS rules running in phase 1
* CRS rule 949111: blocking early
* plugins *-after.conf

Please note that plugin rules in the *-before.conf file will
run before the formal initialization of CRS in the 901 rules
file. If you do anomaly scoring in phase 1 before the
901 rules are executed, the scores will be overwritten.

If you want to score in phase 1, you need to put your rules
in the plugin *-after.conf file. Yet the rules in this file
are being executed after the early blocking is being executed.

In conclusion: Phase 1 anomaly scoring via plugins and early
blocking are incompatible. This is a weakness of
the CRS plugin architecture that is very hard to improve.


### Phase 2

After phase 1 is over, phase 2 is executed.

We assume that the plugins-config file does not contain any
phase 2 rules.

* modsec-recommended-rules (rule id 200000 and friends)
* plugins *-before.conf
* CRS rules running in phase 2
* CRS rule 949110: normal blocking
* plugins *-after.conf

### Phase 3

* plugins *-before.conf
* CRS rules running in phase 3
* CRS rule 959191: blocking early
* plugins *-after.conf

### Phase 4

* plugins *-before.conf
* CRS rules running in phase 4
* CRS rule 959190: normal blocking
* plugins *-after.conf

### Phase 5

* plugins *-before.conf
* CRS rules running in phase 5
* plugins *-after.conf


## Additional information about plugins

* You do not necessarily need to follow the scoring policy
  of the CRS rule set. If you deem it necessary, then feel
  free to issue a deny directly.
* If you do anomaly scoring, then please score in a way that 
  fits into the existing CRS anomaly scoring infrastructure.
  That means rule 949110 is meant to block requests and
  rule 959100 is meant to block responses.
* You can also work with data files. We suggest you use
  your plugin name as prefix for the data files.
* Please make sure that the files in your plugin-config
  are all running in phase 1.


## How to contribute the Template Plugin to CRS?

Plugins can be inofficial 3rd party plugins. Or they can be official CRS plugins
in our github repository.

If you want to have your plugin hosted in our official repo, then
please register your plugin at https://github.com/coreruleset/plugin-registry.
You can then get in touch with the CRS project and we will review your plugin for inclusion
in our repository. Before you apply, please make sure you
have followed the guidelines above. If there are things that you
can not follow, then please document it throughly and tell us about
it.

## Testing

Plugins can also be shipped with their own tests. Any `ftw` test yaml file that you add below the `tests/regression` directory will be run by the integrated pipeline.
We suggest you to use a new subdirectory for your specific plugin's test, e.g. `tests/regression/<your-plugin>`, and create one file per rule you want to test.

Many plugins target rule exclusions, so it will be rather common to see tests matching a REQUEST_URI but matching for `no_log_contains`, as we want to test that we are not being blocked.

## License

Copyright (c) 2021-2022 OWASP ModSecurity Core Rule Set project. All rights reserved.

The OWASP ModSecurity Core Rule Set and its official plugins are
distributed under Apache Software License (ASL) version 2.
Please see the enclosed LICENSE file for full details.
