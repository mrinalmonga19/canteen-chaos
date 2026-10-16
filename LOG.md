# Debug log

Your notes. One entry per bug you fixed, using the template below.

This file is read as carefully as your code. A correct fix you cannot
explain counts for little; a bug you could not fix but investigated
honestly still counts for something.

Delete the example before you submit.



## Example — delete this

### CC-99 — "The cart total is wrong"

**Reproduced:** Added 2 dosas at Rs. 60 each. The cart showed
Rs. 119.99999 instead of Rs. 130. Happened every time, on any dish with
a price ending in .50.

**Cause:** The total was being added up with plain floating point and
never rounded, so 0.1 + 0.2 style errors showed up on screen. The
rounding helper existed but this one place was not using it.

**Fix:** Ran the total through the existing rounding helper instead of
adding a new one, so every price on screen goes through the same path.

**Checked:** Cart, checkout and the order screen all show Rs. 130 now.
Prices without decimals still show without a trailing .00.

**Time:** about 40 minutes, most of it working out that the cart and the
order screen round in different places.



## CC-0X — "<the complaint, in short>"

**Reproduced:**

**Cause:**

**Fix:**

**Checked:**

**Time:**

## CC-06: "I ordered more than they had"

**Reproduced:** The counter said onyly 3 Dal Tadka were left but it let me order 4 Dal Tadka Thali at Rs. 73 each, and the order went through fine.

**Cause:** In validateLine in backend/logic/validation.js, the stock check only rejected dishes with zero stock or less. It never compared the quantity ordered against the stock left.

**Fix:** I added a check that rejects a line when qty is greater than the dish's stock, with a message saying how many are left. I put it in validateLine because that is where every order line is validated, so the rule applies no matter which screen the order comes from.

**Checked:** At checkout a message showing the current stock of item and the quantity asked for is displayed at the bottom.

**Time:** Around 15 mins, mostly of it locating the exact place of code execution and then simply writing a line of code.

## CC-07: Cancelling an order lowers stock instead of restoring it

**Reproduced:** I ordered 4 Cold Coffee when stock was 25, which correctly dropped it to 21. Then I cancelled the order from My Orders. Stock went down to 17 instead of back up to 25.

**Cause:** releaseStock in backend/logic/validation.js did dish.stock - line.qty. It subtracted the cancelled quantity a second time instead of adding it back.

**Fix:** I changed the minus to a plus. I fixed it in releaseStock because the cancel route calls it for every cancellation, so one change covers all orders.

**Checked:** Ordered 4 Cold Coffee (25 to 21), cancelled it, and stock returned to 25.

**Time:** about 10 minutes, mostly spent on the reproduction, since the cause was obvious once I read releaseStock.

## Could not fix

For anything you investigated but did not solve. Say what you tried and
where you got to. This is worth marks — leaving it blank when you got
stuck is not.

### CC-0X — "<the complaint>"

**What I tried:**

**Where I got to:**

**What I would try next:**



## Extra credit

Anything not on the bug log: a problem you found yourself, a test you
wrote, or a fix you are unsure about. Same format, plus one line on how
you noticed it.

**How I noticed it:** I placed an order, got the confirmation, then opened My Orders and it said "No orders found". The browser still had the order token saved, so the order existed.

**Cause:** In GET /api/orders in backend/routes/orders.routes.js, the member filter checked req.query.userId, but the frontend sends memberId. The comparison was always against undefined, so every order was filtered out.

**Fix:** I changed req.query.userId to req.query.memberId.

**Checked:** After a fresh order, My Orders lists it. Orders that I placed earlier were also visible.

**Time:** about 25 minutes, most of it reading the frontend and backend code side by side to see that the page asked for memberId while the server filtered on userId.