# Q-SYS Plugin - Microsoft 365 Calendar

## Overview

The **Microsoft 365 Calendar Q-SYS Plugin** retrieves calendar information from a Microsoft 365 mailbox and presents room availability inside Q-SYS.

The plugin uses Microsoft Graph to read the configured mailbox calendar and provides a simple room-status view for meeting room and signage workflows.

It is designed for Q-SYS systems that need to know whether a room is currently free, occupied, or about to be booked.

---

## Features

- Read Microsoft 365 calendar events through Microsoft Graph
- Authenticate with Microsoft Entra ID using client credentials
- Monitor a configured user or room mailbox
- Show current room availability
- Indicate when the next meeting starts soon
- Display current or upcoming meeting information
- Poll calendar data automatically every 60 seconds
- Expose key configuration and status controls as Q-SYS pins
- Provide internal UI feedback with room status text and color

---

## Plugin Information

| Property | Value |
| --- | --- |
| Name | Microsoft-365-Calendar |
| Version | 1.0.0.0 |
| Author | Jens Claerebout |
| Protocol | HTTPS / Microsoft Graph |
| Authentication | Microsoft Entra ID app registration |
| Grant Type | Client Credentials |

---

## Microsoft 365 Requirements

The plugin requires a Microsoft Entra ID app registration that can read calendar data through Microsoft Graph.

Required information:

- Tenant ID
- Client ID
- Client Secret
- Mailbox address to monitor

The app registration must have permission to read the target mailbox calendar. For room calendars, use the room mailbox email address as the `Mailbox` value.

---

## Configuration

### Properties

This plugin does not currently expose configurable plugin properties.

---

### Control Pins

#### Inputs

| Control | Type | Description |
| --- | --- | --- |
| `Tenant_ID` | Text | Microsoft Entra tenant ID |
| `Client_ID` | Text | Application/client ID from the app registration |
| `Client_Secret` | Text | Client secret for the app registration |
| `Mailbox` | Text | User or room mailbox address to read |

#### Outputs

| Control | Type | Description |
| --- | --- | --- |
| `Status` | Status Indicator | Plugin connection / Microsoft Graph request status |
| `NextSoon` | Toggle Button | Turns on when the room is free but the next booking starts within 15 minutes |

---

### Internal / UI Controls

These controls are used inside the plugin UI and are not exposed as user pins:

| Control | Type | Description |
| --- | --- | --- |
| `RoomStatusText` | Text Indicator | Human-readable room availability and meeting information |
| `RoomStatusColor` | Text Indicator | Color feedback for room state |

---

## UI Layout

The plugin UI contains a single configuration and status page.

It includes:

- Tenant ID
- Client ID
- Client Secret
- Mailbox
- Room status text
- Room status color feedback

Room status colors:

| State | Color |
| --- | --- |
| Free | Green |
| Next meeting soon | Orange |
| Occupied | Red |
| Loading / Error | Grey |

---

## Communication

The plugin communicates with Microsoft 365 using HTTPS requests to Microsoft identity and Microsoft Graph endpoints.

### Endpoints Used

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token` | POST | Request an access token |
| `https://graph.microsoft.com/v1.0/users/{mailbox}/calendarView` | GET | Read calendar events for the configured mailbox |

### Calendar Query

The plugin reads events from the current time until the end of the current day.

Requested fields:

- subject
- start
- end
- organizer
- isAllDay
- showAs

The calendar view is ordered by event start time.

---

## Behavior

### Authentication

The plugin requests a Microsoft Graph access token using:

- tenant ID
- client ID
- client secret
- `https://graph.microsoft.com/.default` scope
- client credentials grant

If Microsoft Graph returns an authentication error, the plugin clears the token and retries with a new token on the next request.

### Polling

The plugin checks the configured mailbox calendar:

- when the script starts
- when settings change
- every 60 seconds while running

### Room Status

The plugin compares calendar events against the current time and updates the room state:

- If a meeting is active, the room is shown as booked until the meeting end time
- If the room is free and the next meeting starts within 15 minutes, `NextSoon` turns on
- If the room is free and another meeting exists later today, the next meeting time is displayed
- If there are no more meetings today, the room is shown as free for the rest of the day

---

## Installation

1. Add the plugin to your Q-SYS design
2. Create or use an existing Microsoft Entra ID app registration
3. Grant the app permission to read the target mailbox calendar
4. Enter the following values in the plugin:
   - Tenant ID
   - Client ID
   - Client Secret
   - Mailbox email address
5. Deploy to the Core
6. Verify the plugin status changes to **OK** and room status text updates

---

## Notes

- The configured mailbox can be a user mailbox or a room mailbox
- Calendar data is read through Microsoft Graph
- The plugin currently uses the `Europe/Brussels` Outlook timezone preference
- The room status check is based on events from now until the end of the current day
- Meeting subjects are displayed in the room status text
- Protect the client secret like any other credential

---

## Known Limitations

- No automatic mailbox discovery
- No support for multiple rooms in one plugin instance
- No configurable polling interval
- No configurable warning threshold for `NextSoon`
- Timezone is currently fixed to `Europe/Brussels`
- Calendar write actions are not implemented

---

## Future Improvements

- Add configurable timezone
- Add configurable polling interval
- Add configurable next-meeting warning threshold
- Add support for multiple rooms
- Add optional meeting subject privacy mode
- Add richer error messages for Microsoft Graph responses

---

## License

MIT License

---

## Author

Jens Claerebout
