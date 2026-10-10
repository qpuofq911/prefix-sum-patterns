# prefix sum problems: how to spot the pattern, build the prefix array without off-by-one bugs, and work through 16 interview classics

You know the shape of the problem. Given an array, count the subarrays that sum to `k`. Or answer a thousand range-sum queries. Or find the largest rectangle in a matrix with a target total.

The straightforward solution is two nested loops, and it works fine on the sample input. Then it hits the real constraints — 10⁵ elements, 10⁵ queries — and times out at roughly 10¹⁰ operations. The fix is almost always the same fifteen lines of code. The hard part isn't the code. It's recognizing that this is a prefix sum problem before you've burned twenty minutes on a sliding window that keeps breaking because the array has negative numbers in it.

This is a walkthrough of the pattern, the specific places it goes wrong, and the LeetCode problems where it actually shows up.

## The trap that makes prefix sum problems feel harder than they are

Recomputing a range sum from scratch costs O(n) per query. Do that q times and you're at O(q × n). Two loops over every subarray is O(n²) or O(n³) depending on whether you re-sum or reuse.

Prefix sum makes a trade: you spend O(n) once to precompute running totals, and afterward every range sum is a single subtraction. No loop, no dependence on how wide the range is. Two elements or two million — same cost.

That trade is only worth it when the same array gets queried more than a couple of times. For a single one-off sum, a plain loop is simpler and uses less memory. Knowing when *not* to reach for the pattern matters as much as knowing the pattern.

## Everything reduces to one formula

Build an array one cell longer than the input:

python
prefix = [0] * (n + 1)          # prefix[k] = sum of arr[0..k-1]
for i in range(n):
    prefix[i + 1] = prefix[i] + arr[i]

# sum of arr[l..r], inclusive
sum_lr = prefix[r + 1] - prefix[l]


`prefix[0] = 0` is the sum of zero elements. That leading zero is not decoration — it's what lets `l = 0` work without a special case.

Three facts cover almost every bug people hit:

1. `prefix` has length `n + 1`, not `n`.
2. `prefix[0]` is always 0.
3. `prefix[j] - prefix[i]` gives you `arr[i..j-1]` — the right index is exclusive. If a problem hands you an inclusive right index `r`, translate it to `prefix[r + 1] - prefix[i]`.

The off-by-one is where most wrong answers come from. Writing `prefix[i] = prefix[i-1] + arr[i]` with a zero-indexed array is the classic version of this mistake, and it produces correct-looking answers on the sample and wrong ones on the hidden tests.

The idea generalizes past addition: prefix products, prefix XOR, and 2D grids where you precompute rectangle sums. The 2D version uses inclusion–exclusion, which is the same subtraction idea with four terms instead of two:


sum(r1,c1 → r2,c2) = P[r2+1][c2+1] - P[r1][c2+1] - P[r2+1][c1] + P[r1][c1]


## The three jobs prefix sum actually does

Most tutorials flatten these into one pattern. They're distinct enough to separate, because they fail in different ways.

### 1. Precompute once, answer many

Static array, many range queries. Build the prefix array in the constructor, answer each query in O(1). That's LeetCode 303, and it's the cleanest version of the pattern.

### 2. Prefix sum plus a hash map

This is the version that carries the most interview weight. If `prefix[j] - prefix[i] = k`, then the subarray between `i` and `j` sums to `k` — which means for every `j`, you want to know how many earlier prefix sums equal `prefix[j] - k`.

python
counts = {0: 1}          # seed: an empty prefix has sum 0
running = 0
answer = 0
for num in nums:
    running += num
    answer += counts.get(running - k, 0)   # lookup first
    counts[running] = counts.get(running, 0) + 1   # then store


Order matters. Look up before you store, or you'll count the empty subarray ending at the current index. And the `{0: 1}` seed is mandatory — without it you silently drop every valid subarray that starts at index 0.

Two things this handles that sliding window can't: negative numbers, and counts instead of existence. A sliding window needs a monotonic property to know which direction to move. Negatives destroy that, which is exactly why "subarray sum equals k" is a prefix sum problem and not a two-pointer problem.

### 3. Difference array — the pattern run backwards

When you need to apply many range *updates* and read the final array once, you invert the logic: add `v` at the start of the range, subtract `v` one past the end, then take a prefix sum at the very end. Each update is O(1) instead of O(n). Car Pooling and Corporate Flight Bookings are the standard examples, and both look like simulation problems until you notice the boundaries.

## 16 prefix sum problems worth your time

Not all of these are labeled "prefix sum" anywhere. That's the point of the table — the pattern is the recognition skill being tested.

| # | Problem | Difficulty | Variant | The bit that goes wrong |
| --- | --- | --- | --- | --- |
| 1480 | Running Sum of 1d Array | Easy | Plain build | Nothing. It's the build loop, bare |
| 303 | Range Sum Query – Immutable | Easy | Precompute + O(1) query | Right index exclusive in the prefix array |
| 724 | Find Pivot Index | Easy | Left total vs. right total | Checking before adding the current element |
| 560 | Subarray Sum Equals K | Medium | Prefix + hash map counting | Missing the `{0: 1}` seed |
| 930 | Binary Subarrays With Sum | Medium | Counting form on 0/1 input | Same seed issue, disguised |
| 974 | Subarray Sums Divisible by K | Medium | Prefix mod k, count remainders | Negative remainders in Java/C++ |
| 523 | Continuous Subarray Sum | Medium | Earliest index per remainder | Minimum length of 2 |
| 525 | Contiguous Array | Medium | 0 → −1, store first index | Storing the latest index instead of the first |
| 238 | Product of Array Except Self | Medium | Prefix product, both directions | Reaching for division |
| 53 | Maximum Subarray | Medium | Running sum minus minimum prefix | Updating the minimum in the wrong order |
| 304 | Range Sum Query 2D – Immutable | Medium | 2D inclusion–exclusion | Forgetting the `+ P[r1][c1]` term |
| 1074 | Number of Submatrices That Sum to Target | Hard | 2D prefix + hash map per row pair | Collapsing rows correctly |
| 1590 | Make Sum Divisible by P | Medium | Prefix mod + earliest index | Handle the "remove nothing" case |
| 862 | Shortest Subarray with Sum at Least K | Hard | Prefix + monotonic deque | Knowing why a hash map isn't enough |
| 1094 | Car Pooling | Medium | Difference array | Putting the subtraction one index too early |
| 1109 | Corporate Flight Bookings | Medium | Difference array | Same, with 1-indexed input |

Work these roughly in order. Problems 1–3 are about the mechanical build; 4–8 are the counting form, which is where most interview questions land; 9–14 push into products, 2D, and deques; 15–16 flip you into the difference array so you see the pattern from the other side.

If you only have time for four: 303, 560, 304, and 1094. That set covers precompute-and-query, hash map counting, 2D inclusion–exclusion, and range updates. Everything else is a variation on one of those.

> Two more failure modes worth flagging up front. In Java and C#, prefix sums overflow 32-bit integers fast — use `long` whenever element values can be large. And when you take a prefix sum modulo `k`, negative values produce negative remainders in some languages, which breaks the counting unless you normalize with `((x % k) + k) % k`.

## Why people plateau on this pattern

The usual failure isn't understanding prefix sums. It's that the pattern is taught as a formula, and then the practice problems are labeled by topic — so you already know a prefix sum is coming before you read the question. Real interviews don't label anything.

The gap shows up when a problem mentions "contiguous subarray," the values include negatives, and there's no obvious window. Nothing in the statement says "prefix sum." You have to make that jump on your own.

One practical way to train it: after solving each problem in the table, write down the specific phrase in the problem statement that should have triggered the pattern — "subarray sum equals," "divisible by k," "range sum query," "range updates with one final read." That mapping from wording to pattern is the actual skill. The code is memorization.

## What AlgoMonster does with this pattern

AlgoMonster organizes its material around interview patterns rather than problem tags, and prefix sum is one of the patterns it builds out properly rather than mentioning in passing. Its prefix sum section is structured to run theory first: an intro page that builds the running-totals array and walks through the range-sum-as-one-subtraction idea with an interactive widget where you build the array cell by cell and then switch to query mode and slide the range boundaries.

From there, the section hands you practice problems rather than a wall of links — a target-sum subarray search that uses the running-sum-plus-hash-map form, Range Sum Query – Immutable for the precompute-once setup, and Product of Array Except Self for applying the same idea to products in both directions.

The broader curriculum is pitched at the same level: 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials, with the written explanations produced by people who worked at Google and competitive programmers. It's text and diagram based, not video — which is either the selling point or the dealbreaker depending on how you learn. There's also a decision flowchart for pattern selection where prefix sums appears as a branch answer, which is a decent proxy for the recognition training described above.

Two things it ships beyond the pattern library: unlimited Speedrun (timed practice) and an unlimited AI assistant for when you're stuck at 1 a.m. Subscribers also get the Foundation Course covering data structures and algorithms from scratch, and a "Build an LLM" course, at no extra cost. Quant Interview Prep is developed with an external partner and sold separately, so don't count on it as part of the bundle.

👉 [See the prefix sum lessons and the full pattern library](https://algo.monster?fpr=li69)

## Pricing: what's actually on offer

There's no monthly option and no team tier — it's annual or lifetime, individual only. Current promotional pricing:

| Plan | What you get | Price | Billing |
| --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | — |
| Pro – Yearly | All patterns, lessons and problems unlocked; 500+ lessons across 6 courses; 3,000+ editorials; unlimited AI assistant; unlimited Speedrun; real interview questions updated monthly; member discount on live bootcamps | $99/year ($8.25/mo billed annually) | Annual, renews |
| Pro – Lifetime | Everything in the annual plan plus lifetime updates, paid once | $189 | One-time |

Purchase links:

- 👉 [Start with the free tier](https://algo.monster?fpr=li69)
- 👉 [Get Pro Yearly — $99/year](https://algo.monster?fpr=li69)
- 👉 [Get Pro Lifetime — $189 one-time](https://algo.monster?fpr=li69)

Two honest caveats on those numbers. They move — partner pages have tracked different figures at different times as sales start and end, and the annual and lifetime prices are both currently shown discounted from higher list prices. Check the page before you decide, because the price you see at checkout is the one that counts.

The free tier exists and is genuinely usable for evaluating the teaching style. It isn't enough to prep for interviews on its own.

## Which plan makes sense for you

Do the arithmetic on your own timeline rather than the marketing framing.

If you have interviews in the next two to three months, $99 for the year is the obvious pick — you'd spend more than that on coffee during the prep period, and the lifetime plan's extra $90 buys you access you won't use before the offer lands.

Lifetime at $189 breaks even after two renewals. It makes sense if you expect to interview repeatedly over the next several years: engineers who jump companies for promotions, or anyone who wants a reference they can reopen before an internal loop. Over a thirty-year career the annual-versus-lifetime gap stops being interesting.

There's also no formal trial and no refund window advertised, so the free tier is your preview. Use it on the prefix sum section specifically — you'll know within twenty minutes whether prose plus diagrams is how you learn algorithms, or whether you need someone talking over a whiteboard.

One thing to weigh against the price: 3,000+ editorials and a structured pattern library don't replace a problem bank. If you already know which pattern applies and just want volume, you want a larger question set, not a curriculum. AlgoMonster's value is concentrated in the recognition step — the part that makes you reread the problem statement twice and then say "oh, prefix sums."

That's also why prefix sum is a good test case. If the section on it gets you from "here's a problem about subarrays" to "here's the hash map version and here's why a sliding window fails" without you having to hunt through forum threads, the platform is doing what it claims.

👉 [Try the free tier and see how the prefix sum walkthrough lands](https://algo.monster?fpr=li69)

## Short answers

**Do I need prefix sums for interviews?** For arrays and strings questions, yes — it's one of the few patterns that shows up across easy, medium, and hard problems, and it's the standard answer to any "count the subarrays where..." phrasing.

**Is prefix sum ever the wrong answer?** Yes. If the array gets updated between queries, the precomputed totals are invalidated and you want a Fenwick tree or a segment tree instead. If the problem needs range min or max rather than a sum, there's no prefix trick that works the same way. And if all values are positive and you want the longest or shortest window meeting a condition, a sliding window is simpler and uses less memory.

**What's the hardest part?** Not the code. It's deciding fast enough that the pattern applies, before you've sunk time into the wrong approach. That's what the problem list above is for.
