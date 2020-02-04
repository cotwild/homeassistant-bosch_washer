# homeassistant-bosch_washer

IMPORTANT!!!!!
Creating a module for each device causes too many requests on the HOME CONNECT server and returns error 409. I therefore created a combined sensor, actually working with Oven (Steamer & normal oven), Dishwasher, Washer and Dryer.

USE THIS INSTEAD: https://github.com/cotwild/home_connect
.
.
.
.
.
.
.
.
!!! Adoption for Home Connect Washer !!! (works with SIEMENS & BOSCH devices)
All credits to the original author "gkreitz" of the code!

This is a quick n' dirty component for Home Assistant to read the state of a Bosch Washer using Home Connect.

This will give you four sensors for each Home Connect washer you have:
- **door**: `open`, `close`, `locked`, or `unknown`. https://developer.home-connect.com/docs/status/door_state
- **program**: `cotton`, `synthetic`, ..., `unknown`. https://developer-staging.home-connect.com/docs/dryer/supported_programs_and_options
- **remaining**: time remaining in seconds, or unknown.
- **state**: `inactive`, `ready`, `run`, `finished`, ..., or `unavailable`. https://developer.home-connect.com/docs/status/operation_state

If the washer is off/not connected to wifi, you'll get a **state** of `unavailable`` and the rest as ``unknown``.

## Installation
- Ensure your washer is set up and working in the Home Connect app.
- Copy this folder to `<config_dir>/custom_components/bosch_washer/`.
- Create an account on https://developer.home-connect.com/.
- Register an application. Pick `Device flow` for OAuth flow.
- Once you start this sequence, you have 5 minutes to complete it (or you'll have to restart from here):
  - `export CLIENT_ID="YOUR_CLIENT_ID"`
  - `curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "client_id=${CLIENT_ID}" https://api.home-connect.com/security/oauth/device_authorization | tee tmp.json`
  - Go to `verification_uri` in a browser, type in `user_code`. Log in using your (end user, not developer) Home Connect account and approve.
  - `export DEVICE_CODE=$(jq -r .device_code tmp.json)`
  - `curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=urn:ietf:params:oauth:grant-type:device_code&device_code=${DEVICE_CODE}&client_id=${CLIENT_ID}" https://api.home-connect.com/security/oauth/token | tee access_token.json`
  - `jq .refresh_token access_token.json`

Put the following in your home assistant config:
```
sensor:
  - platform: bosch_washer
    refresh_token: "YOUR_REFRESH_TOKEN"
```

## Remarks on the API
This is built using the Home Connect API, documented on https://developer.home-connect.com/. There is plenty in the API that is not exposed via this component. Using the API, one can also remote control the dryer, but I haven't figured out a use case for that yet. The API is a straightforward REST API with Oauth authentication. There's also a server-side event feed giving pretty quick updates. Originally, this module was just polling, but I figured it'd be fun to test out asyncio, so I rewrote the module to be async and cloud-push.

The API is a bit flakey, and tends to time out/return 504 during European evenings. Currently, this module retries forever, with an exponential backoff. I'll fix to something a tad better if/when I get sufficiently annoyed.
