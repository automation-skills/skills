# Diagnostic tree

```
Failure reported
├─ Hard error on node? → read message + params + credential + typeVersion
├─ Empty output?
│  ├─ IF/Switch routed elsewhere?
│  ├─ HTTP 2xx with empty body?
│  ├─ Code returned [] or wrong shape?
│  └─ Expression undefined filtered downstream?
├─ Wrong values?
│  ├─ `$json` after merge/context clear?
│  └─ Off-by-one Merge input?
└─ Flaky?
   ├─ Rate limit / partial pagination?
   └─ Race on sub-workflow async?
```
