



ssh -i some1 -L 1337:localhost:30663 ubuntu@<ipNode> // where yopass live!!!


##########################



```bash
### 1. Clone repo
git clone https://github.com/jhaals/yopass.git
cd yopass

### 2. Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

### 3. Install k3s
curl -sfL https://get.k3s.io | sh -
sudo k3s kubectl get nodes

mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
kubectl get nodes

### 4. Only if node hangs / RAM tight (skip on t3.small+)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo systemctl restart k3s

### 5. Deploy yopass
kubectl apply -f deploy/yopass-k8.yaml
kubectl get pods -w

### 6. Expose via NodePort (internal only, no public port)
kubectl get pods
kubectl expose pod <pod-name> --type=NodePort --port=1337 --name=yopass-nodeport
kubectl get svc yopass-nodeport

### 7. Access via SSH tunnel (only port 22 needs to be open in security group)
ssh -i <your-key> -L 1337:localhost:<NODEPORT> ubuntu@<server-public-ip>
# then open http://localhost:1337/ in your browser

### 8. Test
# - paste a secret, encrypt, wait ~5-10s
# - copy the one-click link
# - open it in a different browser/incognito -> confirm it decrypts
# - reload same link -> should say secret is gone (one-time proof)
```

**Key reminders:**
- Minimum instance: **t3.small** (2GB RAM) — t3.micro isn't enough
- Never expose the app port publicly — SSH tunnel only
- Public IP changes on stop/start unless you attach an Elastic IP
























#####################################






https://canyouseeme.org/



https://canyouseeme.org/

https://eruda.liriliri.io/




####################################







YOPASS DEPLOYMENT — WHAT WE DID & ACHIEVED
============================================

GOAL
----
Deploy a self-hosted instance of yopass (open-source, end-to-end encrypted,
one-time-view secret sharing tool) on AWS EC2, using Kubernetes (k3s), and
verify it actually works end-to-end.


WHAT WE BUILT
-------------
1. Installed kubectl and k3s (lightweight single-node Kubernetes) on an EC2 
   Ubuntu instance.
2. Deployed yopass + memcached into the cluster using the repo's official
   deploy/yopass-k8.yaml manifest.
3. Exposed the yopass pod internally via a Kubernetes NodePort service.
4. Set up secure access via SSH tunnel/port-forward (instead of exposing
   the app port to the public internet) — so only SSH (port 22) needs to
   be open in the AWS security group.
5. Verified the full secret-sharing flow actually works:
   - Encrypted a test secret client-side (OpenPGP, in-browser)
   - Generated a one-time URL + separate decryption key
   - Opened the link in a second/incognito browser (simulating a recipient)
   - Confirmed decryption worked
   - Confirmed the link died after one view (true one-time-use behavior)


PROBLEMS WE HIT & FIXED
------------------------
- t3.micro (1GB RAM) was too small — k3s alone used ~900Mi just idling,
  causing timeouts and hung kubectl commands.
  FIX: added swap as a temporary patch, then properly resized the
  instance to t3.small (2GB RAM) — the real fix.

- kubectl port-forward was unstable under real browser load (multiple
  parallel asset requests caused dropped connections).
  FIX: switched to a Kubernetes NodePort service instead, which is more
  robust for this kind of traffic.

- Public-IP + VPN combination caused inconsistent/failed connections,
  even after all AWS-side configuration (security groups, iptables,
  Kubernetes service wiring) was confirmed correct.
  FIX: switched to accessing yopass via an SSH tunnel instead of a
  public port — this bypassed whatever the VPN was doing, and is also
  simply the more secure long-term pattern (no app port ever exposed
  to the internet).


KEY TAKEAWAYS FOR NEXT TIME
-----------------------------
- Minimum instance size for k3s + a small app: t3.small (2GB RAM), not
  t3.micro.
- Always access self-hosted tools like this via SSH tunnel or a private
  network (Tailscale, VPN you control) — never expose the app port
  directly to 0.0.0.0/0.
- NodePort > port-forward for anything a real browser will hit with
  multiple simultaneous requests.
- Elastic IP recommended if reusing the same server across sessions,
  since the public IP otherwise changes on every stop/start.


RESULT
------
A working, self-hosted, end-to-end-encrypted, one-time-view secret
sharing service, running on Kubernetes, accessed securely, and proven
to work with a real encrypt → share → decrypt → self-destruct test.
