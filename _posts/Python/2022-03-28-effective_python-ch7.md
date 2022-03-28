---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 7. 동시성과 병렬성"
category: Python
tag: Python
---

## 52. 자식 프로세스를 관리하기 위해 subprocess를 사용하라
subprocess는 python에서 child process를 관리하기 위해 사용하는 built-in 모듈.  
책에서는 `subprocess.Popen`을 사용해 관리할 것을 권유하고 있으며 기본적인 사용방법은 <https://docs.python.org/ko/3/library/subprocess.html> 링크에 있음.

한 가지 유의할 점은 `communicate`를 사용하기 전 buffer를 close하면 바로 프로그램이 실행이 된다는 점을 주의. stdin은 왠만하면 직접 건드리지 않는 것이 좋을듯.  

**- Code**
```python
import subprocess
import os

def run_encrpy(data):
    # 여기서 env는 환경변수를 뜻함.
    env = os.environ.copy()
    env['password'] = 'zf7ShyBhgZ0raQDdE/FiZpm/m/8f9X+M1'

    proc = subprocess.Popen(
        ['openssl', 'enc', '-des3', '-pass', 'env:password'],
        env=env, 
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)

    proc.stdin.write(data)
    proc.stdin.flush()
    return proc

def run_hash(input_stdin):
    return subprocess.Popen(
        ['openssl', 'dgst', '-whirlpool', '-binary'],
        stdin=input_stdin,
        stdout=subprocess.PIPE)


encrpyt_procs = []
hash_procs = []

for _ in range(3):
    data = os.urandom(100)

    encrpyt_proc = run_encrpy(data)
    encrpyt_procs.append(encrpyt_proc)

    hash_proc = run_hash(encrpyt_proc.stdout)
    hash_procs.append(hash_proc)

    encrpyt_proc.stdout.close()
    encrpyt_proc.stdout = None

for proc in encrpyt_procs:
    proc.communicate()
    assert proc.returncode == 0

for proc in hash_procs :
    # communicate는 stdout, stderr를 반환한다.
    # 이는 앞서 stdout을 subprocess.PIPE로 설정해서 그런 것임.
    out, _ = proc.communicate()
    print(out[-10:])

assert proc.returncode == 0
```

**- Result**
```text
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
b'b\x05\xa4\x8eT1\x19\x18\x14\xd0'
b'\xb9\xa4\xa7*\x8e\xee\x10\x80\\\xbe'
b'\xd0\xe9\xa1?\xc3[\xf4d\xbe\xda'
```