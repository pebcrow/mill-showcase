# Connecting MillSwitch to a mill system: high-level design

MillSwitch plugs into an existing mill or scale-house system through one adapter. This page describes what the app needs from that system and how an adapter is held to it. It names no specific vendor; everything below is a general assumption to confirm when a real integration is built.

## What the app needs from a mill system

1. **Membership through signed invites.** A mill hands out an invite, shown as a QR code, that carries only the mill, a one-time value, an expiry and a signature. It holds no secrets. A driver who redeems it becomes a member of that mill and receives a credential for it in one step.
2. **Load reporting and crediting.** The app can list a driver's delivered loads (ticket, delivery time, tons) and the credited total, which equals the sum of those loads.
3. **Per-mill history.** Each mill keeps its own history, so a driver who switches mills sees that mill's loads and nothing from the others.

## What an adapter assumes

- The system exposes the mills a driver app can join, each with a stable id and a display name.
- Invites are single-use and expire. A mill refuses reuse, expiry, tampering and a foreign signature.
- Credentials are issued per mill. One mill refuses another mill's credential, and a mill can check a credential and say who it belongs to.
- Loads and credited tons are available per driver.
- Every call is request and response, and can fail with a clear refusal.
- A real system may not let an app create invites at all. That step may exist only for testing.

```
 phone app --> port (what the app needs) --> adapter --> mill system
                       ^                         ^
                       |                         |
                contract tests          synthetic mills (today)
                                        your system's adapter (next)
```

## How contract tests keep any adapter honest

The app talks to a mill system only through a small port. A shared set of contract tests describes what any adapter must do: accept a good invite, refuse a reused, expired, tampered or foreign one, keep credentials separate per mill, and credit exactly the sum of the loads. Today these tests run against a synthetic adapter with invented mills. A new adapter is done when the same tests succeed against it. If an assumption proves false, the port and these notes change together.
