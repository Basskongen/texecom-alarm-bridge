# Changelog

## 1.0.11

- Fixed: the area flags are 72, not 73; asking for one more made the panel refuse the
  request and then stop answering, so the area status stayed unknown with 1.0.9 and 1.0.10.
  Should a panel refuse a request, the flags are read in smaller requests.

## 1.0.10

- The area flags are read in smaller requests when the panel refuses a large one.

## 1.0.9

- Panels other than the Premier Elite 88 (for example the Elite 64) keep the keypad
  display, the clock and the area flags at other places in memory. On those models the
  app now takes area status, the display and the clock from the panel's Connect
  commands, so the alarm panel, the area sensors and the keypad card work. Outputs,
  system faults and the keypad LEDs stay unavailable there until mapped.
- Fixed: on such a panel the areas could show as armed while disarmed.

## 1.0.8

- Diagnostics: administrators can read panel memory through the integration
  (websocket command `texecom/read_memory`), to map panels with another memory layout.

## 1.0.7

- One diagnostics line in the log at start (panel model, raw display bytes, log pointer),
  to tell panels with another memory layout apart. When the display reads empty, the
  app looks for the display text in the panel's memory and logs where it is.

## 1.0.6

- A memory read the panel does not answer in time is sent once more before it counts
  as a failure, so a slow panel no longer drops the connection during start-up.
- The log says which command got no reply.
- The network search runs only when nothing answers on the panel's address.

## 1.0.5

- Fixed: with `host` left empty, Home Assistant could not create the panel's entities.
- After the panel is found on the network, the app connects within seconds instead of
  waiting a minute.

## 1.0.4

- Fixed: the app still refused to start without `host`.

## 1.0.3

- `host` is optional: left empty, the app looks for the panel on the network Home
  Assistant is on.

## 1.0.2

- Fixed a bug in the UDL passcode check.

## 1.0.1

- New option `find_panel` (on by default): if the panel does not answer on `host`, the
  app looks for it on the local network and remembers where it was found. Useful when
  the panel gets its address from DHCP.

## 1.0.0

- First release.
