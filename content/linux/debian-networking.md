+++
title = "Debian Network Configuration"
date = 2026-11-10
weight = 1
+++

## Objective

Mempelajari konfigurasi jaringan Debian Linux.

## Environment

- OS: Debian 13
- Network: VirtualBox
- Interface: enp0s3

## 1. Check Network Interface

Jalankan:

```bash
ip addr show
```

## 2. Check Routing Table

```bash
ip route show
```

## 3. Check DNS Configuration

```bash
resolvectl status
```

## 4. Expected Output

```text
default via 10.0.2.2 dev enp0s3
```

## 5. Verification

Pastikan interface mendapatkan IP address
dan default gateway tersedia.
