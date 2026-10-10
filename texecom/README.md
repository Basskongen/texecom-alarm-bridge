# Texecom Premier Elite

Connects a Texecom Premier Elite panel (via ComIP or SmartCom) to Home Assistant
over MQTT with automatic discovery.

Areas appear as alarm panels, zones as binary sensors, plus outputs, PC Control
switches, panel and battery voltage, tamper flags, LCD text and the event log.

Uses both the Connect protocol (live events, arm/disarm) and the Wintex/UDL
protocol (memory reads) on a single TCP connection. Only one client can talk to
the panel at a time: stop texecom2mqtt and Wintex before starting this app.
