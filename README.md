# 🏆 Intigriti October 2026 CTF — Coupon Reuse Bypass

![CTF Solved](https://img.shields.io/badge/CTF-SOLVED-brightgreen)
![Category](https://img.shields.io/badge/Category-Business%20Logic-blue)
![Impact](https://img.shields.io/badge/Impact-Zero--Cost%20Checkout-orange)

**Author:** raffaele365  
**Challenge:** Intigriti October 2026 CTF  
**Target:** https://challenge-1026.challenges.intigriti.io  
**Category:** Business Logic / Coupon Validation Bypass  
**Status:** Solved

---

## 🎯 TL;DR

The October 2026 Intigriti challenge featured a checkout system with promotional coupons and a product priced at **€666.00**.

I discovered that the application accepted certain lowercase variations of already-applied coupon codes, allowing the same logical promotions to be reused.

By exploiting inconsistent case handling in the coupon validation workflow, I reduced the checkout total from **€666.00 to €0.00**, completed the order, and retrieved the CTF flag.

**The vulnerability was a business logic bypass, not an injection vulnerability.**

---

## 🛒 1. Understanding the Challenge

The application presented an online shopping interface with a product called **The Midnight Collection**, priced at **€666.00**.

The checkout page included:

- A shopping cart
- A promotional code input
- An order summary
- A discount calculation
- A Complete Order button

The goal was to exploit a vulnerability in the application and retrieve a flag in the format:

```text
INTIGRITI{...}
```

The interesting attack surface was the promotion-code workflow, particularly how the application identified and tracked previously applied coupons.

---

## 🔍 2. Discovery and Investigation

### Step 1 — Identifying valid coupons

During testing, I identified two valid promotion codes:

| Coupon | Discount |
|---|---|
| `NIGHT20` | 20% |
| `NEWS30` | 30% |

Both codes were accepted by the checkout application.

However, simply applying valid discounts was not enough to solve the challenge.

The next step was to investigate how the application handled coupon reuse.

### Step 2 — Investigating case sensitivity

I tested whether the application considered uppercase and lowercase versions of a coupon to be identical.

The key question was:

> Would a coupon already applied as `NIGHT20` be recognized as the same coupon if submitted as `night20`?

I applied the original code and then tested its lowercase variation.

```text
NIGHT20
night20
```

**Observed result:** The lowercase variation was accepted and the discount was applied again.

The order summary showed **NIGHT20 × 2**.

This was the first indication of a coupon-reuse validation flaw.

### Step 3 — Testing another capitalization variation

I also tested:

```text
Night20
```

This variation was rejected.

That observation was important: the application was not simply accepting every possible capitalization. Its handling of coupon representations was inconsistent.

### Step 4 — Repeating the test with NEWS30

I repeated the approach using the second promotion:

```text
NEWS30
news30
```

The lowercase variation was accepted.

The order summary showed **NEWS30 × 2**.

A further mixed-case variation, `News30`, was rejected.

The behavior supported the finding that specific differently capitalized representations could bypass the intended coupon-reuse restriction.

---

## 💥 3. Exploitation

The complete successful payload sequence was:

```text
NIGHT20
NEWS30
night20
news30
```

### Reproduction steps

1. Open the challenge application.
2. Navigate to the shopping cart containing The Midnight Collection.
3. Apply `NIGHT20`.
4. Apply `NEWS30`.
5. Submit `night20`.
6. Submit `news30`.
7. Observe the duplicate promotion counts and zero checkout total.
8. Click **Complete Order**.
9. Retrieve the challenge flag.

### Observed checkout result

| Item | Result |
|---|---:|
| Original product price | €666.00 |
| NEWS30 promotion | Applied × 2 |
| NIGHT20 promotion | Applied × 2 |
| Total discount | €666.00 |
| **Final checkout total** | **€0.00** |

The screenshot confirmed that both promotion codes were recorded twice and the final checkout total was zero.

---

## 📸 4. Proof of Concept

### Successful zero-cost checkout

![Coupon reuse — NEWS30 x2, NIGHT20 x2, total EUR 0.00](Screenshot%202026-10-25%20171753_2.png)

**Screenshot observations:**

- Product: The Midnight Collection
- Price: €666.00
- Promotion NEWS30: 30% off × 2
- Promotion NIGHT20: 20% off × 2
- Final total: **€0.00**
- Complete Order button available

> **Note:** The image will appear after the screenshot has been uploaded to the root of this GitHub repository with the exact filename shown in the Markdown link.

---

## 🚩 5. Challenge Flag

The checkout flow was completed successfully.

**Order reference:**

```text
BLE-228A7990F1
```

**Flag:**

```text
INTIGRITI{094638e6-ba2f-4c50-8bb4-41ced0d5debd}
```

The flag was submitted to Intigriti, and the challenge solution was confirmed as correct.

---

## 🧠 6. Root Cause Analysis

The observed vulnerability was inconsistent handling of coupon identity during validation and reuse checks.

A secure coupon system should consistently determine whether two submitted representations refer to the same promotion.

For example:

```text
NIGHT20
night20
```

If both representations identify the same promotion, they must also share the same reuse restrictions.

In this challenge, the application accepted certain alternative capitalizations as valid promotions without enforcing the intended duplicate-use restriction.

### Why this matters

The problem was not that lowercase coupon codes existed.

The problem was that changing the capitalization allowed an already-used logical promotion to be applied again.

This distinction is essential: **input normalization and authorization of coupon usage must be consistent throughout the entire checkout workflow.**

### Important limitation

The server-side source code was not available for inspection.

Therefore, inconsistent coupon normalization is an explanation consistent with the observed behavior, not a claim about a specific backend implementation.

---

## 🛡️ 7. Security Impact

In a real e-commerce application, a similar flaw could allow an attacker to:

- Reuse restricted or single-use promotional codes
- Bypass discount stacking restrictions
- Apply discounts more times than intended
- Reduce payable order totals below their intended values
- Potentially complete purchases without the expected payment

In this challenge, the observed impact was a successfully completed checkout with a final total of **€0.00**.

---

## 🔧 8. Recommended Remediation

### Consistent coupon normalization

Coupon identifiers should be normalized consistently before validation, duplicate detection, and storage.

Example:

```javascript
function normalizeCoupon(code) {
  return code.trim().toUpperCase();
}

const normalizedCode = normalizeCoupon(submittedCode);

if (appliedCoupons.has(normalizedCode)) {
  throw new Error("Coupon already applied");
}

appliedCoupons.add(normalizedCode);
```

This is an illustrative example, not extracted challenge source code.

### Server-side enforcement

The backend should independently enforce:

1. Coupon eligibility
2. Per-order usage limits
3. Discount stacking restrictions
4. Duplicate promotion prevention
5. Trusted calculation of the final payable amount

### Regression testing

Tests should include:

```text
NIGHT20
night20
Night20
nIgHt20
```

All equivalent representations should follow the same coupon identity and usage rules.

---

## 🏁 9. Conclusion

The October 2026 Intigriti CTF demonstrated how a seemingly minor inconsistency in string handling can lead to a significant business logic vulnerability.

The successful attack did not require SQL injection, XSS, or complex exploitation tools.

Instead, it required examining how the application processed valid input, testing alternative representations, and observing a discrepancy in coupon-reuse enforcement.

**Final result: €666.00 → €0.00 → Flag captured.**

Thanks to **Intigriti** for organizing the challenge!

---

**Researcher:** raffaele365  
**Challenge:** Intigriti October 2026 CTF  
**Result:** Solved
