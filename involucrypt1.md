# Involucrypt1

## Description :
Pendant une investigation numérique sur un disque dur, vous trouvez un fichier chiffré en utilisant un algorithme propriétaire.

Heureusement, le script utilisé pour chiffrer et déchiffrer se trouve dans le même dossier, mais vous ne disposez pas de la clé. Bon courage !

## Solution :
Modifier le programme crypt.py pour bruteforce la clé de déchiffrage du fichier involucrypt1.

```python
import itertools
import operator
import sys

import binascii

BLOCK = 150


class mersenne_rng(object):
    def __init__(self, seed=5489):
        self.state = [0] * 624
        self.f = 1812433253
        self.m = 397
        self.u = 11
        self.s = 7
        self.b = 0x9D2C5680
        self.t = 15
        self.c = 0xEFC60000
        self.l = 18
        self.index = 624
        self.lower_mask = (1 << 31) - 1
        self.upper_mask = 1 << 31

        # update state
        self.state[0] = seed
        for i in range(1, 624):
            self.state[i] = self.int_32(
                self.f * (self.state[i - 1] ^ (self.state[i - 1] >> 30)) + i
            )

    def twist(self):
        for i in range(624):
            temp = self.int_32(
                (self.state[i] & self.upper_mask)
                + (self.state[(i + 1) % 624] & self.lower_mask)
            )
            temp_shift = temp >> 1
            if temp % 2 != 0:
                temp_shift = temp_shift ^ 0x9908B0DF
            self.state[i] = self.state[(i + self.m) % 624] ^ temp_shift
        self.index = 0

    def get_random_number(self):
        if self.index >= 624:
            self.twist()
        y = self.state[self.index]
        y = y ^ (y >> self.u)
        y = y ^ ((y << self.s) & self.b)
        y = y ^ ((y << self.t) & self.c)
        y = y ^ (y >> self.l)
        self.index += 1
        return self.int_32(y)

    def get_rand_int(self, min, max):
        return (self.get_random_number() % (max - min)) + min

    def int_32(self, number):
        return int(0xFFFFFFFF & number)

    def shuffle(self, my_list):
        for i in range(len(my_list) - 1, 0, -1):
            j = self.get_rand_int(0, i + 1)
            my_list[i], my_list[j] = my_list[j], my_list[i]


def keystream(seeds, length, base=None):
    key = base if base else []
    for seed in seeds:
        random = mersenne_rng(seed)

        for _ in range(BLOCK):
            if len(key) == length:
                break
            key.append(random.get_rand_int(0, 255))
            random.shuffle(key)
        if len(key) == length:
            break
    return key

def encrypt(string, key, hex = False):
    key = keystream(map(ord, key), len(string))
    
    if hex:
        stream = itertools.starmap(operator.xor, zip(key, string))
    else:        
        stream = itertools.starmap(operator.xor, zip(key, map(ord, string)))


    return bytearray(stream)

def iteration(key):
    print(key)

hexdata = bytearray(open(sys.argv[1], 'rb').read())
charset = "abcdefghijklmnopqrstuvwxyz"

def solve_password(maxrange):
    for i in range(3,maxrange+1):
        print(i)
        for attempt in itertools.product(charset, repeat=i):
            keybf = ''.join(attempt)
            resp = encrypt(hexdata.decode('latin-1'), keybf)

            try:
                print(keybf + ": " + resp.decode('utf-8'))
            except:
                False

print(solve_password(10))
```

Après quelques minutes on tombe sur `ane` qui nous donne le flag :
```
The flag is: TheKeyIsTooDamnWeak!

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud
exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure
dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
```

Flag : `TheKeyIsTooDamnWeak!`


#### Explication :
Avant même d'expliquer pourquoi l'algorithme utilisé est vulnérable, il est important de dire qu'utiliser un MT (Mersenne Twister) pour générer des clefs cryptographiques n'est pas à faire dû à la facilité d'en prédire le comportement.
L'entièreté de la classe `mersenne_rng` n'est pas intéressante pour résoudre le challenge. A vrai dire, le fait qu'un MT soit utilisé pour générer la clef n'est ici même pas pertinent.
Concentrons nous sur la fonction `keystream` :
```python
def keystream(seeds, length, base=None):
    key = base if base else []
    for seed in seeds:
        random = mersenne_rng(seed)

        for _ in range(BLOCK):
            if len(key) == length:
                break
            key.append(random.get_rand_int(0, 255))
            random.shuffle(key)
        if len(key) == length:
            break
    return key
```
Le paramètre `seeds` correspond à un array contenant les codes des caractères saisis en argv 1, `length` correspond à la taille de la chaîne à chiffrer.
Dans la mesure où la fonction `keystream` ajoute 150 (variable `BLOCK`) octets "aléatoires" par caractère saisi (sauf pour le dernier bloc si la taille de la chaîne à chiffrer modulo 150 est différente de 0), il est important que le chaîne à chiffrer ait une taille supérieure ou égale à `150 * len(argv[1])`, sinon seule une partie des caractères rentrés  seront utilisés dans la génération de la clef.
Comme le fichier à chiffré, `involucrypt1`, a une taille de 370 octets, seuls les 3 premiers caractères (`plafond(370 / 150)`) de l'argv 1 seront utilisés, d'où la facilité du bruteforce.
