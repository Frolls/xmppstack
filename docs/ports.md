# Ports

- `22/tcp` SSH administration
- `80/tcp` public HTTP for redirect and ACME bootstrap
- `443/tcp` public HTTPS and WebSocket entrypoint
- `5222/tcp` XMPP client-to-server
- `5269/tcp` XMPP server-to-server when federation is enabled
- `5443/tcp` internal ejabberd HTTP/WebSocket/upload port
- `3478/tcp,udp` STUN/TURN
- `5349/tcp` TURN over TLS
- `49152-49200/udp` TURN relay range
