# Lab Environment

## 1. Container Setup and Commands

Primeiro, começamos por dar setup ao Docker. Tivemos de criar três containers diferentes para conseguirmos simular três máquinas diferentes: o Attacker, o Host A e o Host B.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/562d7fa7-103f-4dd2-a027-2594ef6177e4/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/e686fa27-238d-4e44-af41-4dce44533c36/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/60674f77-d723-4625-ac23-3a795ef7b9c4/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/0cc34005-7e00-47fc-ad71-ab4e8d9b3039/Untitled.png)

## 2. About the Attacker Container

De seguida, procuramos o nome da interface da rede. Para isto, usamos o comando `ifconfig` , que nos mostra várias interfaces de rede. O nome da rede que procuramos é o que está associado ao endereço IP 10.9.0.1 (logo o primeiro que nos aparece) - `br-15d9ab541510`.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/b8e5146c-a443-4d2d-abba-c046f37fbe5c/Untitled.png)

# Task 1 - Using Scapy to Sniff and Spoof Packets

Para esta Task, vamos usar o Scapy. O Scapy é uma ferramenta usada para sniffing and spoofing, mas também funciona como bloco de construção de outras ferramentas de sniffing and spoofing.

Para usar esta ferramenta, tivemos deatravés de um programa de python fornecido no guião do lab.

```python
# python3
from scapy.all import*
a = IP()
a.show()
###[ IP ]
###version   = 4
ihl       = None
...
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/e8451173-c1c3-459b-afcb-a17dae226b49/Untitled.png)

## 1.1 Sniffing Packets

Para fazer o sniffing, vamos usar, mais uma vez, um script fornecido no lab, que exemplifica como fazer packet sniffing com o Scapy em programas de Python.

```python
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface='br-15d9ab541510', filter='icmp', prn=print_pkt)
```

O `iface` é o nome da rede que encontramos usando o comando `ifconfig` na preparação do ambiente para o lab.

### 1.1 A

Nesta alínea, analisamos

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/5919df30-047a-4a90-a33d-afd665dd7d9b/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/46998531-3970-45cc-be3d-983d284c5a32/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/026ba364-5daa-4861-b357-e7c5ace24bcf/Untitled.png)

tentei coisar em seed

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/48e34ba6-bae3-41ac-903d-1d5f2e12ab34/Untitled.png)

### 1.1 B

filtros 

icmp

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/96900e3e-3c76-4a29-bd39-e66c93e67596/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/516705ce-3cfc-446b-98d6-7b967f6e5f9a/Untitled.png)

tcp 23

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/c3b93268-85e6-47ed-8402-d6cdeed03dbd/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/f4aea3a1-9e5c-4b8d-b150-55e193065e57/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/3709ef8d-5229-4f76-b9f6-f87eeda33c1d/Untitled.png)

subnet

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/25b43e18-aaeb-481f-bd02-f3a1e5706a3c/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/8b1710ae-2033-4a94-9938-bebd95e06b32/Untitled.png)

## 1.2 Spoofing ICMP

```python
from scapy.all import*
a = IP()
ls(a)
a.src = '10.9.0.5'
a.dst = '10.9.0.6'
b = ICMP()
p = a/b
send(p)
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/482b8561-c990-4d11-b337-c148dbec0e52/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/770180ed-181e-45df-aea2-114c14a3ac23/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/011a2762-a47f-4b02-872c-f27526489f93/Untitled.png)

## 1.3 Traceroute

```python
from scapy.all import*
a = IP()
a.dst = '8.8.8.8'
a.ttl = 1
b = ICMP()
send(a/b)
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/d91ea282-7afb-4630-9e45-9e19ed31f4e8/Untitled.png)

temos de criar um programa para descobrir quantos hops é que ele tem de fazer

```python
import sys
from scapy.all import*
a = IP()
a.dst = '8.8.8.8'
a.ttl = 1
b = ICMP()

while True:
	rep = sr1(a/b, timeout=1)
	if rep == None or (rep[ICMP].type == 11 and rep[ICMP].code == 0):
		a.ttl +=1
		continue
	break
	
print(a.ttl)
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/85f6b931-7310-4af9-8548-c58844871d7e/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/09fb57e0-7025-489f-a28f-9b24f9d0494e/f2ed7af5-be4f-459b-b496-7581d20c3e45/Untitled.png)

## 1.4 Sniffing and-then Spoofing

```python
def snif_and_then_spoof(p):
    if p[ICMP].type != 8:
        return 0
    
    a = IP()
    a.src = p[IP].dst  # destino do packet é a origem de a
    a.dst = p[IP].src  # origem do packet é o destino de a

    b = ICMP()
    b.type = 0  # echo reply
    b.id = p[ICMP].id
    b.seq = p[ICMP].seq
    load = p[Raw].load

    send(a / b / load)

p = sniff(iface='br-15d9ab541510', filter='icmp', prn=snif_and_then_spoof)
```
