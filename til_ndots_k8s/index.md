# Kubernetes DNS performance - ndots


# Intro
TIL a DNS config that exists in many kubernetes setups, including yours.

## Context
Today I learned about ndots.
ndots is a config parameter from `/etc/resolv.conf` which configures a threshold for the number of dots which must appear in a name given to res_query

## Scenario

Depending on your Kubernetes setup, pods may contain the following content inside `/etc/resolv.conf`:
```
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Notice that ndots is set to 5
Now, imagine there's a payments service (apiv1/service) and that other pods (N replicas) are sending requests to this payments service using the internal service url.
i.e `payments-svc.payments.svc.cluster.local:3000` (4 dots)

The dns resolver, will then say "Hey, 4 dots is less than 5!" and then will build name queries with each of the domains below, try them and return the one that resolves.
```
default.svc.cluster.local
svc.cluster.local
cluster.local
```

Now, thing that this other pods are sending 5k requests per second to the payments service.
Sucks to be kube-dns (or core-dns), right? CPU throttling will skyrocket and services depending on payments will start to be slow.

An immediate solution would be to add a dot (.) at the end of the URL<br>
`payments-svc.payments.svc.cluster.local.:3000` <br>

This will tell the DNS resolver to look for the name as it is.

## Testing
If you have prometheus running you can easily test this.

Create a couple pods in namespace X and a service in namespace Y, then write a simple script to force DNS resolution and compare the results.
i.e
`for i in {1..5000}; do nslookup app.y.svc.cluster.local}; done`<br>
and then<br>
`for i in {1..5000}; do nslookup app.y.svc.cluster.local.}; done`



## References
https://pracucci.com/kubernetes-dns-resolution-ndots-options-and-why-it-may-affect-application-performances.html <br>
https://man7.org/linux/man-pages/man5/resolv.conf.5.html
