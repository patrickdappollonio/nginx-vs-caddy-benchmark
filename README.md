# Performance of Caddy vs Nginx

This is a quick-and-dirty test to see how well Caddy vs Nginx works, based on a post from [Manish R Jain](https://manishrjain.com/reverse-proxy-caddy-nginx).

## Setup

You need FiloSottile's amazing `mkcert` tool: https://github.com/FiloSottile/mkcert

Then, run `mkcert -install` and `mkcert localhost` to generate a certificate for `localhost`. This should generate `localhost.pem` and `localhost-key.pem` in the current directory. Put them where the `docker-compose` files are.

For request testing, I recommend Vegeta: https://github.com/tsenart/vegeta

The simplest request you can make is:

```bash
echo "GET https://localhost/" | \
  vegeta attack --format=http --rate=500 --duration=10s | \
  vegeta report --type=text
```

## Running it

To run the Caddy environment, run:

```bash
docker-compose -f docker-compose.caddy.yaml up --remove-orphans
```

The Nginx version can be run with:

```bash
docker-compose -f docker-compose.nginx.yaml up --remove-orphans
```

## My results

These results are on my machine:

### Caddy

```text
Requests      [total, rate, throughput]  5000, 500.11, 500.09
Duration      [total, attack, wait]      9.998200887s, 9.99778805s, 412.837µs
Latencies     [mean, 50, 95, 99, max]    385.502µs, 325.001µs, 613.057µs, 811.483µs, 20.940099ms
Bytes In      [total, mean]              100000, 20.00
Bytes Out     [total, mean]              0, 0.00
Success       [ratio]                    100.00%
Status Codes  [code:count]               200:5000
Error Set:
```

### Nginx

```text
Requests      [total, rate, throughput]  5000, 500.11, 500.09
Duration      [total, attack, wait]      9.998173726s, 9.997755045s, 418.681µs
Latencies     [mean, 50, 95, 99, max]    337.36µs, 287.403µs, 523.78µs, 681.93µs, 19.36402ms
Bytes In      [total, mean]              100000, 20.00
Bytes Out     [total, mean]              0, 0.00
Success       [ratio]                    100.00%
Status Codes  [code:count]               200:5000
Error Set:
```

## Addendum

These tests were run on a laptop:

```bash
$ inxi
CPU: 6-core Intel Core i7-10710U (-MT MCP-) speed/min/max: 2448/400/4700 MHz
Kernel: 6.0.6-76060006-generic x86_64 Up: 4h 5m
Mem: 8390.5/15691.5 MiB (53.5%) Storage: 476.94 GiB (41.7% used) Procs: 424
Shell: Bash inxi: 3.3.13
```

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Pop
Description:    Pop!_OS 22.04 LTS
Release:        22.04
Codename:       jammy
```

Looking at the data, it seems that Caddy is a bit slower than Nginx.
