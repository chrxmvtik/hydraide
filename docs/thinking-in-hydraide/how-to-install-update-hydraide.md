# 🚀 Install & Update HydrAIDE – No Pain. Just Power.

Welcome to **HydrAIDE installation** – where setup is no longer a burden, and updates are no longer a risk.

HydrAIDE wasn’t just built to perform.
It was built to arrive like a storm.

- No build steps.
- No dependency hell.
- No environment mismatch.

Just a Docker container.
And magic happens. 💥

---

## 🧠 Philosophy – Shipping Without Regret

We’ve all been there:

> "It worked on my machine."

Or worse:

> "Why did the update break *everything*?"

HydrAIDE was engineered to **never put you in that position**. From day one, we made a commitment:

> 🔐 "Every version of HydrAIDE ships as a fully encapsulated container – with no build step required."

There’s no compiling. No dependency checks. No need to match OS versions.

HydrAIDE runs **everywhere Docker runs**. That’s it.

And because each release is tagged, verified, and immutable, you get:

- 🕊️ Clean upgrades.
- 🧰 Effortless rollbacks.
- 📦 Predictable deployment.

You’re not setting up a database.
You’re **plugging in a power source**.

---

## 📦 The Container Stack – Ready for Anything

HydrAIDE ships as a multi-container system – each instance tuned for a specific purpose (live, test, crawler, etc).
All using the exact same core image.

Your `docker-compose.yml` defines how HydrAIDE is run – securely, efficiently, and repeatably.

```yaml
services:
  hydraide-server:
    image: ghcr.io/HydrAIDE/hydraserver:VERSION
    ports:
      - "4900:4444"
    environment:
      - GRAYLOG_SERVER=your.graylog.server:5140
      - GRAYLOG_SERVICE_NAME=hydraide-service
      - HYDRA_MAX_MESSAGE_SIZE=5368709120  # Max message size HydrAIDE accepts via gRPC. Useful when inserting massive datasets across thousands of Swamps. Can be raised to multiple GB.
      - GRPC_SERVER_ERROR_LOGGING=true  # Logs gRPC connection errors (e.g., certificate mismatch) to Graylog. Useful in dev. Turn off in production to avoid extra overhead.
    volumes:
      - /path/to/cert:/hydraide/certificate
      - /path/to/settings:/hydraide/settings
      - /path/to/data:/hydraide/data
    stop_grace_period: 10m
```

> ⚠️ **Important:** The public HydrAIDE server is **not yet released**. Please do not attempt to use this image.
> Instead: ⭐ [Star the GitHub repo](https://github.com/hydraide/hydraide) and subscribe to be notified when the server is live.

Until then, the container above is for demonstration only.

---

## 🛡️ Volumes Matter – Do This Right

HydrAIDE’s performance and durability depend heavily on how you handle volumes. Here’s what we recommend:

- Use **external volumes** mounted into your container.
- Choose **ZFS** file systems with snapshot support.
- Enable **RAID-like redundancy** (1 or 2 disk fault tolerance).
- Never store your HydrAIDE data on the same disk as your OS.

Why?
Because when you use:

> 🔗 ZFS + External Volumes + Isolated Disks

You get:

- Instant backup via `zfs snapshot`
- Fast, atomic recovery
- Rock-solid disk I/O without choking your OS

And this matters.
Because HydrAIDE is a system that writes **constantly, intentionally, and at scale**.

---

## 🧬 Multi-Device Strategy – Let Your Disks Specialize

HydrAIDE supports running **multiple instances**, each writing to different disks.

Want to store cold data (logs, archives) on HDD and hot data (caches, indexes) on SSD?

> Easy. Just run two containers.

Each one points to a different volume. Your app can choose **where to send data** by controlling which client writes to which Swamp.

That’s not just flexibility. That’s **architectural power**.

---

## 🔐 Security First – TLS and Certificates

HydrAIDE only speaks gRPC over **secure connections**.

You must mount a certificate folder inside the container. This ensures encrypted communication with all clients.

In your compose file:

```yaml
volumes:
  - /your/cert/folder:/hydraide/certificate
```

If you don’t provide valid certs, the server won’t start. That’s by design.

HydrAIDE doesn’t trust unencrypted traffic. Neither should you.

---

## 🧾 Settings vs Data

HydrAIDE uses two critical paths:

- `/hydraide/settings` → where your pattern registry lives.
- `/hydraide/data` → where your Swamps and Treasures are stored.

Keep them separate. Mount them externally. Back them up often.

If you ever lose the `/data` folder, you’ve lost your world – unless you have a snapshot. In that case, recovery is as fast as a `zfs rollback`.

If you ever lose the `/settings` folder, **don’t panic** – your application will re-register every Swamp pattern on startup. That’s not a bug. That’s a feature.

> HydrAIDE’s philosophy: the map is not the territory – and it can always be redrawn.

---

## 🧊 Graceful Shutdowns – Don’t Pull the Plug

HydrAIDE performs **multi-layered memory + disk flushing** on shutdown.

Once shutdown is initiated:

- New connections are refused.
- Existing connections are completed.
- All Swamps are flushed to disk, even if their `writeInterval` wasn’t due yet.

Add this to your compose file:

```yaml
stop_grace_period: 10m
```

This gives HydrAIDE time to:

- Finalize all writes
- Flush memory safely
- Release internal locks cleanly

If you shut down HydrAIDE without this window?
You risk partial writes.
You risk corruption.

HydrAIDE is graceful. But only if you let her finish.

---

## 🔄 Updates – Immutable by Default

Each release of HydrAIDE is tagged by version and published to a public registry. Updates are:

- Atomic
- Reversible
- Fully testable in parallel

Just change the image tag. Re-deploy. Done.

Want to test a new version before going live?

> Run two containers side-by-side. One on `:4900`, one on `:4901`. Compare. Decide. Switch.

Your data stays safe.
Your system stays online.

HydrAIDE gives you the power to upgrade **without fear**.

---

## 🖥️ Minimum Requirements – Or Rather, the Lack of Them

What are HydrAIDE’s minimum system requirements?

> None. Not really.

HydrAIDE was designed from day one to operate in **silence until summoned**. That means:

- No background processes.
- No idle memory leaks.
- No constant CPU churn.

It does absolutely *nothing* until you tell it to. And when you do?

> It only uses the memory needed to load your Swamp – nothing more.

Practically speaking:

- HydrAIDE runs on **a single-core CPU** just fine.
- It can operate with **as little as 512 KB of memory** (as long as your OS allows it).
- It doesn’t preload. It doesn’t buffer unless told to.

This makes HydrAIDE ideal for:

- ⚙️ Tiny edge devices
- 📦 Docker containers with ultra-low limits
- 🔬 Raspberry Pi–level deployments

The only real factor that affects memory usage is:

- The size of your **largest Swamp**
- The temporary **in-memory indexes** (Go maps) created during queries

That’s it.

Compared to other systems?

- Traditional databases run daemons.
- They preload schemas.
- They cache aggressively.
- They consume RAM just *because they exist*.

HydrAIDE does none of that.

> HydrAIDE always exists – patiently waiting, silently listening.
> It’s the Swamps that don’t exist until you call their name.
> And when you do, they wake — and consume only what they need. Nothing more.

This isn’t minimalism. This is **intentional invisibility**.

---

## 🧠 Final Words

Thank you for joining us on this journey through HydrAIDE's installation and deployment philosophy.

By now, you’ve seen how HydrAIDE removes friction from every part of the setup process – from container launch to graceful shutdown.

And here’s the best part:

> **You’re ready.**

You’ve learned the mindset.
You’ve explored the architecture.
You’ve understood the rhythm of how HydrAIDE thinks, stores, and scales.

Now, it’s time to bring it to life.

With the upcoming SDKs, you’ll be able to integrate HydrAIDE into your first project in **less than a day** – and you’ll know *exactly* what you’re doing.

Because now, you don’t just install HydrAIDE.

> You think like HydrAIDE.

Welcome to the future of data.

---

## 🧭 Navigation

← [Back to 🌐 Distributed Architecture](./distributed-architecture.md)
