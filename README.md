Intigriti October 2026 CTF — Coupon Reuse via Case-Sensitive Validation

Challenge: Intigriti October 2026 CTF
Challenge URL: https://challenge-1026.challenges.intigriti.io
Category: Business Logic / Coupon Validation Bypass
Status: Solved
Author: raffaele365

TL;DR

The challenge contained a coupon validation flaw: the application treated coupon codes as case-insensitive when applying discounts, but the duplicate-use validation distinguished uppercase and lowercase characters.

By submitting the same logical coupon codes with different capitalization, I was able to apply their discounts more than once, reduce the checkout total to €0, and obtain the CTF flag.

1. The Vulnerability

Coupon codes should have a consistent identity throughout the discount workflow.

In this challenge, however, two representations of the same coupon were treated differently by the validation logic:

* NIGHT20 and night20
* NEWS30 and news30

The application recognized these variations as valid discount codes, while the duplicate-use check failed to identify them as already applied.

This created a business logic vulnerability that allowed repeated application of discounts that should have been restricted.

2. Exploitation Steps

Step 1 — Add the target product

Open the challenge application and add the product midnight-vault to the cart.

Step 2 — Apply the original coupons

Apply the following coupon codes:

NIGHT20
NEWS30

Step 3 — Reuse the coupons with different capitalization

Submit the lowercase variants:

night20
news30

The application accepts the differently capitalized coupon strings instead of treating them as duplicates of the previously applied discounts.

Step 4 — Complete the checkout

After applying the coupon variations, the total reaches €0.

Proceed through the checkout flow to complete the challenge and retrieve the flag.

3. Proof of Concept

The essential payload sequence is:

NIGHT20
NEWS30
night20
news30

Observed result: The application allows the same logical discounts to be applied more than once, resulting in a zero-cost checkout.

Flag:

INTIGRITI{094638e6-ba2f-4c50-8bb4-41ced0d5debd}

4. Root Cause Analysis

The vulnerability arises from inconsistent coupon normalization.

The application effectively treats NIGHT20 and night20 as equivalent for discount application, but not for duplicate detection.

A secure implementation should normalize coupon identifiers consistently before performing both validation and discount application.

For example:

const normalizedCode = submittedCode.trim().toUpperCase();
if (appliedCoupons.has(normalizedCode)) {
  throw new Error("Coupon already applied");
}
appliedCoupons.add(normalizedCode);

This simplified example illustrates the required consistency. In a real application, coupon validity and usage limits should also be enforced server-side.

5. Security Impact

In a real e-commerce application, this flaw could allow an attacker to:

* Reuse single-use or non-stackable discounts.
* Combine discounts beyond the intended limits.
* Reduce an order total below the intended price.
* Cause financial losses if checkout accepts the manipulated total.

The key lesson is that business logic vulnerabilities do not always require complex injection payloads. A mismatch in how an application compares and processes user-controlled values can be enough to bypass important restrictions.

6. Remediation

The application should:

1. Normalize coupon codes consistently before validation and storage.
2. Enforce coupon uniqueness and usage restrictions on the server.
3. Recalculate the final payable amount using trusted server-side pricing rules.
4. Add regression tests for uppercase, lowercase, and mixed-case coupon submissions.

Conclusion

This challenge demonstrated how inconsistent case handling can turn a seemingly minor validation mistake into a complete checkout bypass.

The successful exploit relied on identifying the difference between coupon identity validation and discount application, then using alternative capitalization to apply the same logical discounts repeatedly.

Thanks to Intigriti for the October 2026 challenge!