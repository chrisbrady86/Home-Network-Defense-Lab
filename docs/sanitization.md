# Sanitization Checklist

Run through this before every commit that touches a config or log artifact.

- [ ] Public IP addresses removed or replaced with documentation ranges
- [ ] Internal hostnames replaced with generic labels
- [ ] MAC addresses removed
- [ ] Wireless SSIDs and PSKs removed
- [ ] API keys, tokens, and credentials removed
- [ ] Certificates and private keys removed
- [ ] Usernames replaced
- [ ] Packet captures not committed at all

`git log -p` after the fact does not undo a leaked secret. Check before, not after.
