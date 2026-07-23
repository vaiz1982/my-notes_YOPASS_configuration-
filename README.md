
For each secret - need to open new forward ! // looks like safety stuff here ! 
#############

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












###################################################

** Let's clarify the actual architecture, because "inside the cluster" doesn't mean "only reachable from inside the cluster."

**How it actually works:**
- Kubernetes/k3s is just *how* yopass runs (the process manager, essentially) — it's not a barrier that traps the app inside some sealed box.
- The pod runs the yopass server + memcached.
- The NodePort service exposes that pod's port on the **node itself** (your EC2 box's OS-level network stack).
- From there, *you* control who can reach it — via your SSH tunnel, via opening a port publicly, via Tailscale, whatever you choose.

**So who can you actually send secrets to?**
Anyone, as long as they can reach the URL you send them. Right now, your setup only lets *you* reach it (via SSH tunnel from your own device) — so practically, the recipient right now is just... you, testing with yourself in a second browser.

**To actually share with someone else (a colleague, a friend, anyone outside this box), you have real options:**

1. **Open the app port properly with TLS**, so it's a normal public website anyone can visit (this is what `yopass.se`, the public demo, does). Requires a domain + Let's Encrypt cert — more setup, but genuinely shareable with anyone.

2. **Put both of you on the same private network** (Tailscale, WireGuard, or your existing VPN) — then the recipient just needs to also be on that network to reach the link. This matches your very first question at the start of this whole conversation about Tailscale.

3. **SSH tunnel for each recipient** — technically possible but impractical; you'd need to give them SSH access to your box, which defeats the purpose.

**Why run it in a cluster (k3s) at all, then, versus just plain Docker?**
Honestly — for this use case (single small app, single user), Kubernetes doesn't buy you much. Its real value is things like: automatic restarts, scaling to multiple replicas, rolling updates, managing many services together. For one lightweight app like yopass, plain `docker-compose` (which we discussed early on) gets you the same result with way less complexity — which is part of why today's troubleshooting was so much heavier than it needed to be.

**Bottom line:** the cluster was a good learning exercise, but for *actually* sharing secrets with other people going forward, you'd want either (a) a properly TLS'd public instance, or (b) put it behind Tailscale so anyone you invite to your tailnet can reach it — the k3s vs Docker choice is separate from that and matters less.
