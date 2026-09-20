# BroadcastAbsurdity
## Chall.py 
<img width="486" height="396" alt="image" src="https://github.com/user-attachments/assets/1bcdea7b-1a0b-4e56-b0ea-78380306a95a" />

## Phân tích bài toán
- Trong file chall.py ta thấy tác giả cố ý cho chạy vòng lặp 3 lần để mã hoá m (giá trị của **flag** sau khi bytes_to_long(m)). Ta thu được ba giá trị $n$ và $c$ trong 2 mảng $N,C$.
- Ta gọi mỗi giá trị trong $C$ là $c_1$, $c_2$, $c_3$ và trong $N$ là $n_1$, $n_2$, $n_3$.
  - Ta có:
    
    > $m^3 \equiv c_1 \pmod {n_1}$ <br>
    > $m^3 \equiv c_2 \pmod {n_2}$ <br>
    > $m^3 \equiv c_3 \pmod {n_3}$
  - Lúc này ta dùng thuật toán CRT để tìm ra $m^3$.
- Ta biết thêm một dữ kiện đó là $c = m^3 \bmod n \iff c = m^3$ (do $m^3 < n$), khi đó chỉ cần khai căn bậc 3 là ra $m = c^{1/3}$.
- Sau khi tìm ra m ta chỉ việc long_to_bytes(m) là ra **flag**.

## Python code
> [!CAUTION]
> - Hàm crt() và long_to_bytes() em lấy từ thư viện cá nhân.
> - Dùng hàm **integerr_nthroot()** trong thư viện **sympy** để khai căn bậc 3.
> - https://github.com/nguyenduongquoc8a6ntb/CryptoHack-Solutions/blob/main/Mathematics/Modular%20Math/Chinese_Remainder_Theorem.md

```python
N = [12077989369956056661925086906908222219056970917555067442667721635624020654631593427171278531123427154805529118942689, 19528264414437837871959809380154186932477506014280949005688200852470388683956542765267176057650652453185075909145911, 24311462873650297974522399850043255276984903463352412900415446293427154885548943973025088362699984280599136861226543]
C = [6254845900359533057633352066153345045271179999352043512492925739255834861550815163947666207341742038817547272393860, 5261392505662771660286278557121222271081784355974489892524678204457139656669927082251705427336115503245633492398488, 10393789593625980560037304573761244101414756621712010895377770346687060976771050156095847510599360908913736561130395]

def long_to_bytes(text):
    text = hex(text)[2:]
    text = bytes.fromhex(text)
    return text

def crt(a_list,n_list): # Chinese Remainder Theorem - [x ≡ a (mod N)]
    # N = n1 * n2 * n3...
    N = 1
    for i in n_list:
        N *= i
    
    # find x_i
    x_list = []
    for i in range(len(n_list)):
        N_i = N//n_list[i]
        M_i = pow(N_i,-1,n_list[i])
        x_i = a_list[i]*N_i*M_i
        x_list.append(x_i)

    return sum(x_list) % N

# Logic bài toán
from sympy import integer_nthroot
m_3 = crt(C,N)
m,b = integer_nthroot(m_3,3)
print(long_to_bytes(m).decode())
```




# PrivateMeansPrivate
## chall.py
<img width="520" height="405" alt="image" src="https://github.com/user-attachments/assets/010dedfd-ce97-4098-b52e-c3d4ad7af945" />

## Phân tích bài toán
- Đây là một bài RSA thông thường nhưng tác giả cho to biết thêm dữ kiện: $dp = d \bmod{(p-1)}$.
- Ta biến đổi:
  > $dp = d \bmod{(p-1)} \iff d = t.(p-1) + dp$
  - Nhân $e$ vào 2 vế:
    
    > $d.e = t_1.(p-1).e + dp.e$
  - Mà ta biết:
    
    > $d.e = t_2.\phi n + 1 \iff d.e = t_2(p-1)(q-1) + 1$
  - Kết hợp hai phương trình:
    
    > - $t_1(p-1)e + dp.e = t_2(p-1)(q-1) + 1$ <br>
    > $\iff dp.e -1 = t_2(p-1)(q-1) - t_1(p-1)e$ <br>
    > $\iff dp.e -1 = (p-1)[t_2.(q-1)-t_1.e]$ <br>
  - Với $t_2.(q-1)-t_1.e$ là hằng số ta đặt là k, Khi đó $dp.e - 1 = k.(p-1)$ thì $dp.e - 1$ chính là một bội của $(p-1)$.
- Chuyển vế ta thu được: $p = [(e.dp - 1)/k] + 1$.
- Ta biết $1 \le k < e$ vì:
  
  - Ta biết:
    
    > $dp < p - 1$ vì phần dư luôn nhỏ hơn số chia.
  - Nhân hai vế với e sau đó trừ 1 cho hai vế:
    
    > $dp.e - 1 < (p-1)e - 1$ <br>
    > $\iff k.(p-1) < e(p-1) - 1 < e(p-1)$
    > $\iff k<e$
- Để tìm ra **flag** ta brute-force giá trị k từ 1 đến $e$ để tìm $p$. Nếu $n /bmodp = 0$ thì đó là $p$ đúng.

## Python code
```python
n = 17933844014668288781101082296839404540185958114363900663980182726457867609572866270173248191587544410168818310433797
e = 65537
dp = 2104594675241281760675171840981611646923801292320309511153
c = 1986022583739945836122659941907174160804165451697456928488146621762051590120192187187103291037729656532513119385748

for k in range(1,e):
    p_temp = ((dp*e - 1)//k) + 1
    if n % p_temp == 0:
        p = p_temp
        break

q = n//p
phi_n = (p-1)*(q-1)
d = pow(e,-1,phi_n)
m = pow(c,d,n)
flag = bytes.fromhex(hex(m)[2:])
print(flag.decode())   
```
  
  
