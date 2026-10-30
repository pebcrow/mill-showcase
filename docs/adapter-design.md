# Connecting MillSwitch to a mill system: high-level design

MillSwitch plugs into an existing mill or scale-house system through one adapter and a sign-in hook. This page describes what the app needs from that system and how an adapter is held to it. It names no specific vendor; everything below is a general assumption to confirm when a real integration is built.

## What the app needs from a mill system

1. **Driver sign-in.** The driver signs in once with the platform's own identity provider. The app never holds the driver's login secret or a client secret.
2. **Per-driver signed invites, issued by the mill.** The mill's back office or scale house issues a short-lived invite for a specific driver (or carrier), shown as a QR code. It carries only the mill, a key id, a one-time value, an expiry, the driver it is for and a signature. It holds no secrets. Issuing the invite is how the mill approves the driver, and the mill can revoke that driver later.
3. **Redeeming an invite.** The app sends the invite together with the driver's sign-in token. The mill takes the driver's identity from that token, never from an id the app supplies, and returns an access token and a refresh token scoped to that mill only.
4. **Loads as pending tickets.** A driver records a load (slip, truck, trailer, product, origin). It is a pending ticket until the mill's scale fills in gross, tare and net, which makes it scaled; the scaled ticket is authoritative. The mill can also reject a ticket.
5. **Credited weight.** Credited weight is the sum of scaled net weight only. Pending and rejected loads count for nothing. Each mill shows it in its own unit (for example short tons for a US mill).
6. **Safe replay after a dead zone.** Loads recorded with no signal are queued on the phone and sent when it reconnects. Each load carries a client-generated id (clientLoadId): a retry with the same load returns the original ticket, and a different load with the same id is refused.
7. **Per-mill history.** Each mill keeps its own loads, so a driver who switches mills sees that mill's loads and nothing from the others.

## What an adapter assumes

- The system has a public list of sites a driver may try to join, and, separately, the mills this driver holds tokens for. Neither is a cross-tenant listing of the driver's mills.
- Invites are single-use and expire. A mill refuses reuse, expiry, tampering, an unknown key and a foreign signature.
- Tokens are issued per mill, short-lived and refreshable. One mill refuses another mill's token, and a revoked driver's token is refused.
- A mill can check a token and say which mill, driver and expiry it belongs to.
- On the phone, a driver's per-mill tokens sit in secure storage, one token per mill. Switching mills means using a different token, with no cross-tenant query.
- Every call is request and response, and can fail with a clear refusal.
- Issuing invites, revoking drivers and scaling or rejecting loads are back-office calls. They never ship in the driver app.

```
 phone app --> port (what the app needs) --> adapter --> mill system
                       ^                         ^
                       |                         |
                contract tests          synthetic mills (today)
                                        your system's adapter (next)
```

## How contract tests keep any adapter honest

The app talks to a mill system only through a small port. A shared set of contract tests describes what any adapter must do: accept a good invite and refuse a reused, expired, tampered or foreign one; take identity from the sign-in token; keep tokens separate per mill and honour revocation; keep loads pending until scaled; return the original ticket on a replayed load and refuse a conflicting one; and credit exactly the sum of scaled net weight. Today these tests run against a synthetic adapter with invented mills. A new adapter is done when the same tests are green against it. If an assumption proves false, the port and these notes change together.
