# AFFINITY

Source code như sau

```
from Crypto.Util.number import getPrime
from random import randint

FLAG = b"KCSC{s0m3_r3ad4ble_5tr1ng_like_7his}"

assert FLAG.startswith(b"KCSC{") and FLAG.endswith(b"}")
   
def caesar_affine(msg, key, p):
    ct = []
    for m in msg:
        c = (m + key[0]) % p
        c = (key[1]*c + key[2]) % p
        ct.append(c)
    return ct

enc = list(FLAG)
n = getPrime(32)
for i in range(32):
    key = (randint(1,n), randint(1,n-1), randint(1,n))
    enc = caesar_affine(enc, key, n)

# print(n) In your dream! :)
print(n)

# [1910234182, 2771761218, 1048707146, 2771761218, 871449803, 617943628, 2163740357, 1302213321, 2302873284, 3886794429, 1086831562, 2752699010, 1517595080, 3886794429, 1498532872, 3240649152, 2321935492, 3690474878, 3886794429, 2379122116, 364437453, 3886794429, 3006205185, 852387595, 3905856637, 364437453, 364437453, 3886794429, 4064051772, 2809885634, 2379122116, 3240649152, 149055694, 3886794429, 2125615941, 206242318, 3886794429, 3671412670, 3886794429, 1283151113, 2321935492, 1086831562, 3886794429, 168117902, 2752699010, 1302213321, 3886794429, 2771761218, 2752699010, 1517595080, 579819212, 598881420, 3886794429, 1086831562, 2752699010, 1517595080, 3886794429, 1283151113, 2752699010, 3886794429, 579819212, 852387595, 2321935492, 3690474878, 2302873284, 2302873284, 3202524736, 3202524736, 2302873284, 656068044]
```

Ta thấy rằng hàm mã hóa có chữ affine, mình google thì thu được cách mã hóa của nó [tại đây](https://en.wikipedia.org/wiki/Affine_cipher)

Ta thử lấy ví dụ là ``E1(x) = (5x + 8) % 256`` và ``E2(x) = (4x + 3) % 256``. Ta thử lấy giá trị K là 75, sau khi E1(75) = 127, E2(127) = 255. Suy ra ``E2(E1(x)) = (20x + 4*8 + 3) % 256``.

Ta thấy rằng sau 2 lần mã hóa affine thì sẽ ra một loại affine khác với key mới. ``Key(E1) = (5,8)`` và ``Key(E2) = (4,3)`` thì sẽ thu được ``Key(E) = (20,35)``.

Tiếp tục với chall này, ta thấy rằng ban đầu ký tự sẽ được mã hóa với ``key(E1) = (1, key[0])``, ``key(E2) = (key[1], key[2])`` Thì ta sẽ thu được Key mới là ``Key(E) = (key[1], key[0]*key[1] + key[2])``

Bây giờ ``E(x) = (key[1]*m + key[1]*key[0] + key[2]) % p``, ta sẽ tổng quát nó thành ``E(x) = (A*m + B) % p``

Ta có các ký tự biết trước là ``KCS{}``[75, 67, 83, 67, 123, 125] tương ứng với [1910234182, 2771761218, 1048707146, 871449803, 656068044]

Ta có như sau

```
c0 = (A*75 + B) % p
c1 = (A*67 + B) % p
c2 = (A*83 + B) % p
c3 = (A*123 + B) % p
c4 = (A*125 + B) % p
```

```
c2 - c1 = (A*16) % p
c3 - c2 = (A*40) % p

--> 40*(c2 - c1) = (A*16*40) % p
    16*(c3 - c2) = (A*40*16) % p

---->  40*(c2 - c1) - 16*(c3 - c2) = 0 % p

-----> 40*(c2 - c1) - 16*(c3 - c2) chia hết cho p
```

Tương tự như thế

```
c3 - c2 = (A*40) % p
c4 - c3 = (A*2) % p

--> 2*(c3 - c2) = (A*40*2) % p 
    40 * (c4 - c3) = (A*2*40) % p

---->  2*(c3 - c2) - 40*(c4 - c3) = 0 % p

-----> 2*(c3 - c2) - 40*(c4 - c3) chia hết cho p


========> Nếu gcd(40*(c2 - c1) - 16*(c3 - c2), 2*(c3 - c2) - 40*(c4 - c3)) là số nguyên tố thì p = gcd(40*(c2 - c1) - 16*(c3 - c2), 2*(c3 - c2) - 40*(c4 - c3))

```

Giờ ta sẽ tìm p bằng đoạn code như sau
```
from Crypto.Util.number import*

def gcda(a,b):
        remainder=a%b
        while remainder>0:
            a=b
            b=remainder
            remainder=a%b
        return b

c0 = 1910234182
c1 = 2771761218
c2 = 1048707146
c3 = 871449803
c4 = 656068044

print(gcda(40*(c2 - c1) - 16*(c3 - c2), 2*(c3 - c2) - 40*(c4 - c3)))
```

Output sẽ là ``8260755674`` và factor ra sẽ là ``2 * 4130377837`` --> ``p = 4130377837``

Giờ ta đã có được p, giờ ta sẽ tìm lại A bằng cách

```
c2 - c1 = (A*16) % p

Giờ ta sẽ inverse(16,p) = 2839634763 rồi nhân với (c2 - c1)

<=> (A * 16 * 2839634763) % p = A
```

Tìm A bằng đoạn code này

```
p = 4130377837

A = (c2-c1)*inverse(16,p)%p
```

A = 1957498039

Tìm B thì oke hơn rồi, ``c0 = (A*75 + B) % p`` --> ``B = (c0 - 75*A) % p``

B = 3791483389

Sau khi có được các key cần thiết rồi, ta sẽ tìm lại các ký tự bằng công thức ``m = (c - B)*inverse(A,p) % p`` theo như wiki đã hướng dẫn.

Solution sẽ như sau

```
from Crypto.Util.number import*

c = [1910234182, 2771761218, 1048707146, 2771761218, 871449803, 617943628, 2163740357, 1302213321, 2302873284, 3886794429, 1086831562, 2752699010, 1517595080, 3886794429, 1498532872, 3240649152, 2321935492, 3690474878, 3886794429, 2379122116, 364437453, 3886794429, 3006205185, 852387595, 3905856637, 364437453, 364437453, 3886794429, 4064051772, 2809885634, 2379122116, 3240649152, 149055694, 3886794429, 2125615941, 206242318, 3886794429, 3671412670, 3886794429, 1283151113, 2321935492, 1086831562, 3886794429, 168117902, 2752699010, 1302213321, 3886794429, 2771761218, 2752699010, 1517595080, 579819212, 598881420, 3886794429, 1086831562, 2752699010, 1517595080, 3886794429, 1283151113, 2752699010, 3886794429, 579819212, 852387595, 2321935492, 3690474878, 2302873284, 2302873284, 3202524736, 3202524736, 2302873284, 656068044]

def gcda(a,b):
        remainder=a%b
        while remainder>0:
            a=b
            b=remainder
            remainder=a%b
        return b

c0 = 1910234182
c1 = 2771761218
c2 = 1048707146
c3 = 871449803
c4 = 656068044

print(gcda(40*(c2 - c1) - 16*(c3 - c2), 2*(c3 - c2) - 40*(c4 - c3)))

p = 4130377837

A = (c2-c1)*inverse(16,p)%p

B = (c0 - 75*A)%p

print(B)

flag = ""
for i in c:
      flag += chr((i-B)*inverse(A,p)%p)

print(flag)
```

**Flag: KCSC{Wow!_y0u_be4t_m3_Thr33_7ime5_In_a_d4y_H0w_C0u1D_y0u_d0_1h4t!!??!}**
