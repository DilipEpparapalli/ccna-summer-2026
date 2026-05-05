# OSI Upper Layers — Application, Presentation, Session

## The Layers Most People Gloss Over

These three layers sit at the top of the OSI model and are often hand-waved as "just the application stuff." That's a mistake. Each has a distinct job — and once you see them clearly, troubleshooting application-level problems becomes a lot more structured.

> **TCP/IP note:** In the TCP/IP model, all three collapse into a single "Application" layer. The functions don't disappear — they're just not separated. OSI gives you the vocabulary to talk about *which* function is failing.

---

## Layer 7 — Application

### Simple Explanation
The interface between the network and the software the user actually interacts with. This is where protocols like HTTP, HTTPS, DNS, FTP, and SSH live.

### What it does
- Initiates or receives network communication on behalf of an application
- Defines *what* is being requested — "give me this webpage", "resolve this domain", "transfer this file"

### Real-world example
Your browser types `nextcloud.yourdomain.com` → Layer 7 kicks off an HTTPS request. The browser doesn't care about packets, MACs, or routing. It just says "I want this resource."

---

## Layer 6 — Presentation

### Simple Explanation
Handles **what the data looks like** — format and encryption. It's the translator between the application and the network.

### Two jobs, always:
1. **Data formatting** — ensures both sides agree on how data is structured (JPEG, JSON, HTML, XML, MP4)
2. **Encryption/Decryption** — TLS lives here. Data gets encrypted before going down the stack, decrypted after coming up

### The mental model
```
Raw application data
        ↓
[ PRESENTATION ] ← "Let me format this as JSON and encrypt it with TLS"
        ↓
Transport layer receives clean, encrypted, formatted data
```

### Key distinction
- **Key exchange** happens during the handshake (Session layer territory)
- **Ongoing encrypt/decrypt of actual data** = Presentation layer

### Real-world examples
| Scenario | Presentation layer doing... |
|----------|----------------------------|
| HTTPS to Nextcloud | TLS encrypting your files before transit |
| Streaming a video | Encoding/decoding MP4 format |
| Receiving a webpage | Interpreting HTML so the browser can render it |
| WireGuard tunnel | ChaCha20 encrypting/decrypting data through the tunnel |

---

## Layer 5 — Session

### Simple Explanation
Manages the **lifetime of a conversation** between two applications. Opens it, keeps it alive, closes it cleanly.

### Three jobs, always:
1. **Establish** — sets up the session between two applications
2. **Maintain** — keeps the session alive, manages multiple streams if needed
3. **Terminate** — closes the session cleanly when done

### The mental model that makes it click
```
Every click on Nextcloud = a new HTTP request (new packet, maybe new TCP connection)

Without Session layer → you'd have to log in again every single click
With Session layer    → session token/cookie maintains "this is still the same user"
```

The Session layer keeps the **conversation's identity** alive across multiple individual requests.

---

## Sockets — What Actually Separates Sessions

Port numbers alone aren't enough to separate connections. A **socket** is the full 4-tuple that makes each connection unique:

```
Socket = Source IP + Source Port + Destination IP + Destination Port
```

Two SSH sessions to the same server:
```
Session 1: 192.168.1.10 : 54231  →  203.0.113.5 : 22
Session 2: 192.168.1.10 : 54232  →  203.0.113.5 : 22
```

Same destination IP, same destination port 22 — but different **ephemeral source ports** make each socket unique. Your OS tracks which data belongs to which application using this 4-tuple.

> A socket ≠ just a port number. A socket = the full 4-tuple combination.
