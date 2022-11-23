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

For the 256K test, you can use:

```bash
echo "GET https://localhost/data.bin" | \
  vegeta attack --format=http --rate=500 --duration=10s | \
  vegeta report --type=text
```

You can generate your own custom 256K file by running:

```bash
truncate -s 256K data.bin
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

For the 256K tests, run the `-alt.yaml` files instead. Make sure your test points to the download of the `data.bin` file.

## My results

These results are on my machine:

### Caddy (hello-docker container)

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

### Nginx (hello-docker container)

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

### Caddy (256K container)

```text
Requests      [total, rate, throughput]  5000, 500.11, 500.06
Duration      [total, attack, wait]      9.998832175s, 9.99773652s, 1.095655ms
Latencies     [mean, 50, 95, 99, max]    1.310211ms, 1.208517ms, 1.670566ms, 2.506097ms, 36.576825ms
Bytes In      [total, mean]              1280000000, 256000.00
Bytes Out     [total, mean]              0, 0.00
Success       [ratio]                    100.00%
Status Codes  [code:count]               200:5000
Error Set:
```

### Nginx (256K container)

```text
Requests      [total, rate, throughput]  5000, 500.08, 499.97
Duration      [total, attack, wait]      10.000685239s, 9.998377302s, 2.307937ms
Latencies     [mean, 50, 95, 99, max]    1.536594ms, 990.49µs, 4.165685ms, 7.359351ms, 26.088002ms
Bytes In      [total, mean]              1280000000, 256000.00
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

Looking at the data, here are the back-of-the-napkin calculations:

| Test                    | hello-docker         | 256K                 |
| ----------------------- | -------------------- | -------------------- |
| Caddy (mean)            | 385.502µs            | 1.310211ms           |
| Nginx (mean)            | 337.36µs             | 1.536594ms           |
| Delta                   | Nginx is 114% faster | Caddy is 117% faster |
| Caddy (95th percentile) | 613.057µs            | 1.670566ms           |
| Nginx (95th percentile) | 523.78µs             | 4.165685ms           |
| Delta                   | Nginx is 117% faster | Caddy is 249% faster |
| Caddy (99th percentile) | 811.483µs            | 2.506097ms           |
| Nginx (99th percentile) | 681.93µs             | 7.359351ms           |
| Delta                   | Nginx is 119% faster | Caddy is 293% faster |
