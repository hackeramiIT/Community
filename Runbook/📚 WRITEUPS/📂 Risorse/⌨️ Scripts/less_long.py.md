Trasforma in piccoli chunck e concatena

```
s = "PAYLOAD"
n = 50
for i in range(0, len(s), n):
	chunk = s[i:i + n]
	print('Str = Str + "' + chunk + '"')
```