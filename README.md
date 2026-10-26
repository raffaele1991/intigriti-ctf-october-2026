# 🏆 Intigriti October 2026 CTF — Coupon Reuse Bypass

![Status](https://img.shields.io/badge/CTF-SOLVED-brightgreen)
![Category](https://img.shields.io/badge/Category-Business%20Logic-blue)

**Author:** raffaele365  
**Challenge:** Intigriti October 2026 CTF  
**Target:** https://challenge-1026.challenges.intigriti.io  
**Vulnerability:** Coupon reuse through inconsistent case handling

---

## 🎯 TL;DR

The challenge contained a business logic flaw in coupon validation.

The application accepted differently capitalized versions of the same coupon codes, allowing discounts to be reused and the checkout total to reach **€0**.

## 🔍 Vulnerability

The coupon system did not consistently treat uppercase and lowercase versions as the same logical coupon.

| Original coupon | Alternative representation |
|---|---|
| `NIGHT20` | `night20` |
| `NEWS30` | `news30` |

This inconsistency allowed the intended coupon usage restrictions to be bypassed.

## 💥 Exploitation

### Step 1 — Open the challenge

Navigate to:

```text
https://challenge-1026.challenges.intigriti.io
```

### Step 2 — Add the target product

Add the `midnight-vault` product to the cart.

### Step 3 — Apply the original coupon codes

```text
NIGHT20
NEWS30
```

### Step 4 — Reapply using lowercase variants

```text
night20
news30
```

The application accepted the differently capitalized codes instead of rejecting them as duplicates.

### Step 5 — Complete checkout

The combined discounts reduced the total to **€0**. Completing the checkout revealed the flag.

## 🚩 Flag

```text
INTIGRITI{094638e6-ba2f-4c50-8bb4-41ced0d5debd}
```

## 🧠 Root Cause

The observed behavior is consistent with inconsistent coupon-code normalization between validation and discount application.

For example, a system may consider `NIGHT20` and `night20` different during duplicate detection while recognizing both as the same discount.

The exact server-side implementation was not available, so this is a root-cause hypothesis based on the observed behavior.

## 🛡️ Remediation

- Normalize coupon identifiers consistently before validation and storage.
- Enforce coupon uniqueness and usage limits server-side.
- Recalculate checkout totals on the server.
- Add regression tests for uppercase, lowercase and mixed-case coupon codes.

## 📌 Impact

In a real e-commerce system, a similar flaw could allow users to reuse restricted discounts, bypass stacking rules and purchase products for less than their intended price.

## 🏁 Conclusion

This challenge demonstrates that a small inconsistency in string handling can lead to a significant business logic bypass.

**Thanks to Intigriti for the challenge!**
