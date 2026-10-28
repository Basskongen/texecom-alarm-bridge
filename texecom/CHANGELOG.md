# Changelog

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
