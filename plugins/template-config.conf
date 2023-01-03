# ------------------------------------------------------------------------
# OWASP ModSecurity Core Rule Set Plugin
# Copyright (c) 2021-2022 Core Rule Set project. All rights reserved.
#
# The OWASP ModSecurity Core Rule Set plugins are distributed under
# Apache Software License (ASL) version 2
# Please see the enclosed LICENSE file for full details.
# ------------------------------------------------------------------------

# OWASP CRS Plugin
# Plugin name: template-plugin
# Plugin description: Example plugin. Use and adopt this for your own plugins.
# Rule ID block base: 9,500,000 (range is 1000, thus ID block base +1000)
# Plugin version: 1.0.0

# Generic rule to disable the plugin
#
# Plugins are enabled by default.
#
# They become active by placing them in the plugin folder. It is possible to
# control plugin activation via setting a variable. This can be done in the
# plugin config file here.
#
# The predefined variable name is meant to be "<plugin name>-plugin_enabled".
# For the template-plugin, this means it can be disabled by setting
# tx.template-plugin_enabled=0.
#
# Note that a global setting of this variable overrides the setting here.
# That means the "enabled" variable is only set by this rule if it has not
# been set before.
#
# Feel free to set the variable unconditionally here by replacing the
# SecRule line with an unconditional SecAction statement.
#
#SecRule &TX:template-plugin_enabled "@eq 0" \
#  "id:9500010,\
#   phase:1,\
#   pass,\
#   nolog,\
#   setvar:'tx.template-plugin_enabled=0'"

# Example plugin configuration variable
#
# If your plugin is configurable with variables (such as limits, strictness, etc.)
# you can add your variables here. Leave them commented out, so that interested
# users can activate the variable by removing the comment chars.
#
# A good variable name follows the scheme "<pluginname>-plugin_variablename".
#
#SecAction \
#  "id:9500020,\
#   phase:1,\
#   nolog,\
#   pass,\
#   setvar:tx.template-plugin_foobar=1"
