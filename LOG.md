# Debug log


## CC-06: "I ordered more than they had"

**Reproduced:** The counter said onyly 3 Dal Tadka were left but it let me order 4 Dal Tadka Thali at Rs. 73 each, and the order went through fine.

**Cause:** In validateLine in backend/logic/validation.js, the stock check only rejected dishes with zero stock or less. It never compared the quantity ordered against the stock left.

**Fix:** I added a check that rejects a line when qty is greater than the dish's stock, with a message saying how many are left. I put it in validateLine because that is where every order line is validated, so the rule applies no matter which screen the order comes from.

**Checked:** At checkout a message showing the current stock of item and the quantity asked for is displayed at the bottom.

**Time:** Around 15 mins, mostly of it locating the exact place of code execution and then simply writing a line of code.

## CC-07: "Cancelling makes it worse"

**Reproduced:** I ordered 4 Cold Coffee when stock was 25, which correctly dropped it to 21. Then I cancelled the order from My Orders. Stock went down to 17 instead of back up to 25.

**Cause:** releaseStock in backend/logic/validation.js did dish.stock - line.qty. It subtracted the cancelled quantity a second time instead of adding it back.

**Fix:** I changed the minus to a plus. I fixed it in releaseStock because the cancel route calls it for every cancellation, so one change covers all orders.

**Checked:** Ordered 4 Cold Coffee (25 to 21), cancelled it, and stock returned to 25.

**Time:** about 10 minutes, mostly spent on the reproduction, since the cause was obvious once I read releaseStock.

## CC-08: "An old coupon still works"

**Reproduced:** I added Chicken Manchurian worth Rs. 119.50 to the cart and applied FRESHERS24, whose expiry date is 30 Sept 2024 and whose description says it is expired. The cart took Rs. 30 off and the total came to Rs. 99.23.

**Cause:** applyCoupon in backend/logic/pricing.js never checked the coupon's expiresAt date. It only checked that the coupon exists, has uses left, and meets the minimum order, veg and slot rules. The coupons route hid expired coupons from the public list, but a code typed in by hand skipped that filter.

**Fix:** I added an expiry check inside applyCoupon, right after the uses-left check, so it rejects any coupon whose expiresAt is in the past. I fixed it there because both the cart quote and order placement go through applyCoupon, so one check covers both.

**Checked:** FRESHERS24 is now rejected with "This coupon has expired" and no discount. BYTE10 on the same cart still applies its discount.

**Time:** about 20 minutes, most of it finding that the expiry check was missing from pricing.js while the coupons list route had one.

## CC-10: "Sorting by price is backwards"

**Reproduced:** On the Menu page I picked "Price: low to high" from the sort dropdown. The cheapest dishes were at the bottom of the list instead of the top. "Price: high to low" was reversed the same way.

**Cause:** In the SORTERS object in backend/logic/search.js, the comparators for price-asc and price-desc were swapped. price-asc used b.price - a.price (high to low) and price-desc used a.price - b.price (low to high). The dropdown values in index.html were correct.

**Fix:** I swapped the two comparators so price-asc is a.price - b.price and price-desc is b.price - a.price. I fixed it in SORTERS because the menu route uses it for every price sort, so the frontend needed no change.

**Checked:** Low to high now starts with the cheapest dish and high to low starts with the most expensive. The sort checks in backend/smoke.js pass, including "sort by price works", which failed before the fix.

**Time:** about 10 minutes, mostly spent looking for where sorting happens before spotting that the two comparators were just swapped.


## CC-09: "The menu shows more dishes than it should"

**Reproduced:** I opened the Menu and scrolled to the bottom. More dishes loaded automatically and same dish appeared more than once. The "Showing" number in the header kept growing past the real total of 37 (I saw 74 on the first look).

**Cause:** In paginate in backend/logic/search.js, the page slice was calculated correctly but the function returned items: list, the full list, instead of the slice. Every page request therefore returned all dishes. total and hasMore were still right, which hid the problem.

**Fix:** I changed items: list to items so the function returns only the current page. I fixed it in paginate because every menu request goes through it, so no frontend change was needed.

**Checked:** The menu now loads 5 dishes at a time with no repeats, and "Showing" stops at 37. The smoke test check "pages differ" now passes.

**Time:** about 25 minutes, mostly spent working out whether the fault was in the frontend scroll loading or the backend, before seeing that paginate returned the whole list.

## CC-01 : "The search suggestions are behind everything"

**Reproduced:** I start typing a dish name and the list of suggestions comes up, but it's stuck behind the rest of the page. I can only click the very top bit of it.

**Cause:** The z-index of search bar is lower than the panel below it, so it is shown behind the panel

**Fix:** I just increased the z-index of a function of the search bar to a distinguishable number of 50, therefore, it doesn't clashes with any other function's z-index

**Time:** less than 10 minutes, mostly finding the point of source of error

## Could not fix

## CC-02: "Can't read anything in dark mode"
## CC-03: "The menu is wider than my phone"
## CC-04: "The buttons don't work on my tablet"
## CC-05: "The category bar scrolls away on my phone"

## Extra credit

**How I noticed it:** I placed an order, got the confirmation, then opened My Orders and it said "No orders found". The browser still had the order token saved, so the order existed.

**Cause:** In GET /api/orders in backend/routes/orders.routes.js, the member filter checked req.query.userId, but the frontend sends memberId. The comparison was always against undefined, so every order was filtered out.

**Fix:** I changed req.query.userId to req.query.memberId.

**Checked:** After a fresh order, My Orders lists it. Orders that I placed earlier were also visible.

**Time:** about 25 minutes, most of it reading the frontend and backend code side by side to see that the page asked for memberId while the server filtered on userId.