# DNSCat pcap analysis


#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is a **dark-web** fueled search engine that offers **free** functionalities to check if a company or its customers have been **compromised** by **stealer malwares**.

Their primary goal of WhiteIntel is to combat account takeovers and ransomware attacks resulting from information-stealing malware.

You can check their website and try their engine for **free** at:

{% embed url="https://whiteintel.io" %}

***

If you have pcap with data being **exfiltrated by DNSCat** (without using encryption), you can find the exfiltrated content.

You only need to know that the **first 9 bytes** are not real data but are related to the **C\&C communication**:

```python
from scapy.all import rdpcap, DNSQR, DNSRR
import struct 

f = ""
last = ""
for p in rdpcap('ch21.pcap'):
	if p.haslayer(DNSQR) and not p.haslayer(DNSRR):

		qry = p[DNSQR].qname.replace(".jz-n-bs.local.","").strip().split(".")
		qry = ''.join(_.decode('hex') for _ in qry)[9:]
		if last != qry:
			print(qry)
			f += qry
		last = qry

#print(f)
```

For more information: [https://github.com/jrmdev/ctf-writeups/tree/master/bsidessf-2017/dnscap](https://github.com/jrmdev/ctf-writeups/tree/master/bsidessf-2017/dnscap)\
[https://github.com/iagox86/dnscat2/blob/master/doc/protocol.md](https://github.com/iagox86/dnscat2/blob/master/doc/protocol.md)

There is a script that works with Python3: [https://github.com/josemlwdf/DNScat-Decoder](https://github.com/josemlwdf/DNScat-Decoder)

```
python3 dnscat_decoder.py sample.pcap bad_domain
```

