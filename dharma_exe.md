# Dharma.exe

## Description :
Un candidat nous a remis son CV sur une clé USB. Mais la clé USB comporte également un fichier binaire mystérieux. Est-ce un malware, ou alors le candidat essaie de nous challenger ?

## Solution :
Après une analyse appronfondie et refléchie, on peut observer que le programme, `dharma`, nous demande un mot de passe (exprimé par la sortie `Password: `).  
Après plusieurs essais n'ayant pas aboutis à cause d'un MYSTERIEUX (hahaha) `SIGTRAP` (newbie clue: "anti-debug"), on parvient à lancer `strace dharma`, voilà la sortie (les parties moins intéressantes sont tronquées) :
```
memfd_create("2O3naSbh", MFD_CLOEXEC)   = 3
write(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \22\0\0\0\0\0\0"..., 16936) = 16936
...
getpid()                                = 2158
...
openat(AT_FDCWD, "/proc/2158/fd/3", O_RDONLY|O_CLOEXEC) = 4
read(4, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \22\0\0\0\0\0\0"..., 832) = 832
...
mmap(NULL, 16544, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 4, 0) = 0x7f2b01794000
mmap(0x7f2b01795000, 4096, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 4, 0x1000) = 0x7f2b01795000
mmap(0x7f2b01796000, 4096, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 4, 0x2000) = 0x7f2b01796000
mmap(0x7f2b01797000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 4, 0x2000) = 0x7f2b01797000
close(4)                                = 0
...
close(3)                                = 0
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
...
mmap(NULL, 95984, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f2b00d76000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\0203\0\0\0\0\0\0"..., 832) = 832
...
mmap(NULL, 103496, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f2b00d5c000
mmap(0x7f2b00d5f000, 69632, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x3000) = 0x7f2b00d5f000
mmap(0x7f2b00d70000, 16384, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x14000) = 0x7f2b00d70000
mmap(0x7f2b00d74000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17000) = 0x7f2b00d74000
close(3)                                = 0
mprotect(0x7f2b00d74000, 4096, PROT_READ) = 0
...
clone(child_stack=0x7f2b00d5afb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tid=[2160], tls=0x7f2b00d5b700, child_tidptr=0x7f2b00d5b9d0) = 2160
...
write(1, "Password: \n", 11Password: 
)
```
Un simple `strace` et nous voilà déjà si loin dans l'épreuve... Pour faire court, `dharma` créer un fichier dans la ram (`memfd_create`), y écrit en premier ce qu'il semblerait être un ELF, puis créer des mappings (`mmap`), avant de lire le contenu de la lib dynamique de la libc qu'il a trouvé dans `/etc/ld.so.cache`, pour finir par `clone`.  
On sait à peu près ce qui nous attend, so let's get debugging.  
