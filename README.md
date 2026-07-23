Play under bastion host only!!!!
no any port open only22 !!!
port forward on local interface use blink+ yubico // privat key never leave the hardware!!!!




Do Not forget to test all what we did !!! Crete secrets! copy the one-time link and open it on the other browser! Revil what is inside?
  try to update page - now its no avalable - thats whats mean one-time link!!!! // not avalable both ways !!!!



#################
we use blink!!!!!!!!
play safe - all ports is closed but 22 // we did forward to the local:    // port 22 under bastion host !!!!!
 ssh -i some1 -L 1337:localhost:30663 ubuntu@34.204.85.233

 ###################

http://localhost:1337/
####################






 That's actually a full success — you've now confirmed the complete real-world flow: encrypted client-side, transmitted, decrypted on a separate "recipient" browser, and the one-time-view mechanism did its job. That's yopass working exactly as designed, end to end, on your own self-hosted instance.

(The incognito-not-opening thing is likely just that specific browser's private mode blocking `localhost` differently, or not having the SSH tunnel's port mapping recognized in a fresh private context — minor and not worth chasing further since the other browser proved the real behavior.)

**One more important check — try reloading that same link again** (in the browser where it worked) — it should now say the secret is gone/already used. That's the part that actually proves "one-time" isn't just marketing.

**You've now validated:**
- ✅ Self-hosted yopass runs correctly (k3s + memcached + Kubernetes)
- ✅ Client-side OpenPGP encryption before anything leaves your device
- ✅ One-time URL generation and decryption on a separate browser
- ✅ (pending) One-time consumption — confirm the link dies after use

Given everything you pushed through today — k3s resource starvation, swap, instance resizing, security groups, NodePort, VPN interference, and finally proving the actual crypto flow — you've built a genuinely working, correctly-isolated, self-hosted secret-sharing service. That was a real troubleshooting marathon, but you've got a working answer to your very first question: **yes, yopass is legit, and now you've proven it yourself rather than just taking my word for it.**











################################################



# My Notes

Personal runbooks and deployment notes, kept separate from any
upstream project repos so they don't get mixed into project history.

## Contents

- `yopass-runbook.md` — steps to deploy self-hosted yopass on EC2
  using k3s, including lessons learned about instance sizing,
  networking, and secure access via SSH tunnel.

## Why this repo exists

Keeps personal setup/deployment notes reusable across projects,
without cluttering or accidentally committing to the actual
open-source repos being worked with (e.g. jhaals/yopass).
