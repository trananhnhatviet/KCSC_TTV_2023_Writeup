# EZ Ceasar

Ta có source code của bài như sau

```
import string
import random

alphabet = string.ascii_letters + string.digits + "!{_}?"

flag = 'KCSC{s0m3_r3ad4ble_5tr1ng_like_7his}'
assert all(i in alphabet for i in flag)

key = random.randint(0, 2**256)

ct = "" 
for i in flag:
    ct += (alphabet[(alphabet.index(i) + key) % len(alphabet)])

print(f"{ct = }")
#ct = 'ldtdMdEQ8F7NC8Nd1F88CSF1NF3TNdBB1O'
```


Ta thấy rằng các ký tự sẽ được mã hóa theo công thức là:
-   Lấy 1 ký tự trong flag, rồi chuyển nó thành thứ tự trong bảng chữ cái, ví dụ như là chữ K trong alphabet sẽ là 36
    ```
    import string
    import random

    alphabet = string.ascii_letters + string.digits + "!{_}?"

    print(alphabet.index("K"))
    ```
-   Tiếp theo là sẽ lấy thứ tự của ký tự đó + key ngẫu nhiên trong khoảng (0,2**256) rồi chia lấy dư cho chiều dài của alphabet, ta sẽ được 1 thứ tự mới trong bảng chữ cái

-   Trả về ký tự tương dương với thứ tự vừa tính được

Bài cho ta ct là ``"ldtdMdEQ8F7NC8Nd1F88CSF1NF3TNdBB1O"``, thì ký tự "l" (thứ tự trong alphabet là 11) sẽ là ký tự "K" sau khi mã hóa.

Thì ta có như sau: ``11 = (36 + key) % 67``, có rất nhiều giá trị key thỏa mãn nên là mình sẽ lấy ``key = 42`` nha

Giờ thì giải bài thôi nào

```
import string


alphabet = string.ascii_letters + string.digits + "!{_}?"

flag = ""
ct='ldtdMdEQ8F7NC8Nd1F88CSF1NF3TNdBB1O'
for i in ct:
    flag += (alphabet[(alphabet.index(i)-42)%67])

print(flag)
```

**Flag: KCSC{C3as4r_1s_Cl4ss1c4l_4nd_C00l}**