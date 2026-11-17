# Truck Pilot

A replacement controller for the HEXBUG / VEX Robotics **Offroad Truck**, whose official
"VEX Pilot" app is no longer in the app stores. It is a single web page that talks to the
truck's brain directly over Bluetooth LE.

## Run it

```sh
cd ~/code/hexbug
python3 -m http.server 8000
```

Then open <http://localhost:8000> in **Chrome or Edge** (Safari and Firefox have no Web Bluetooth).

1. Switch the truck on.
2. Press **Connect** and pick the truck from the browser's list.
   If the list stays empty, press **Show all devices** and pick it by name.
3. Drive: left pad is throttle, right pad is steering.
   Keyboard: `W`/`S` or `↑`/`↓` drive, `A`/`D` or `←`/`→` steer, `Space` stops. A game controller works too.

Steering trim, max speed and the two "reverse" switches are remembered between sessions.

### On a phone

The page needs `https://` (or localhost), so host `index.html` anywhere static (GitHub Pages, Netlify…).

- **Android**: open it in Chrome.
- **iPhone / iPad**: Safari can't do Bluetooth; open it in the free **Bluefy** browser.

## Protocol

Recovered from the last Android build of VEX Pilot (`com.vex.pilot.app`).

| What | UUID | Format |
| --- | --- | --- |
| Service | `6faa2911-9200-49a0-a353-6152c5930ec5` | |
| Controls (write without response, every 50 ms) | `6faa8824-…` | 6 bytes: `axis1 axis2 axis3 axis4 buttons1 buttons2`, axes are int8 −100…100 |
| Model config (write once after connecting) | `6faa635a-…` | 12 bytes, byte 8 = preset, rest zero |
| Status (notify) | `6faa5ec2-…` | byte 0 ÷ 20 = battery volts |
| Device info (read) | `6faabb19-…` | 4 version bytes, 4 port types, product type |

Offroad Truck mapping: `axis1` and `axis4` = drive motor, `axis2` = steering (trim is added, ±40).

Presets: 1 Truck, 2 Boxer, 3 V-Raptor, 4 Balancer, 5 Race Car, 6 Monkey, 7 Crane, 8 Alligator, 9 Pinch, 10 BiteForce.
The other models use the same brain; only the preset byte and axis mapping differ.

Advertising: manufacturer data starting `11 11`, `06 77` or `77 06`, then product type
(`0x4A`–`0x4E`), 4 version bytes, mode (`0xAA` pairing, `0x1F` connect, `0x2F` firmware update), 4 id bytes.

The original app also forced a firmware update when the brain was out of date; this page never touches firmware.
