server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no
    root-hints: "/var/lib/unbound/root.hints"
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no
    edns-buffer-size: 1232
    prefetch: yes
    qname-minimisation: yes
    cache-min-ttl: 3600
    hide-identity: yes
    hide-version: yes
    unwanted-reply-threshold: 10000
    val-clean-additional: yes
    so-rcvbuf: 1m
    so-sndbuf: 1m
    auto-trust-anchor-file: "/var/lib/unbound/root.key"

forward-zone:
    name: "."
    forward-addr: 9.9.9.9@853#dns.quad9.net
    forward-addr: 149.112.112.112@853#dns.quad9.net
    forward-tls-upstream: yes
