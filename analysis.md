# Analysis

## A quick note on how I tested this

Before getting into the results, I want to flag a real bug I ran into. This n8n build has a scheduling issue: when parts of the workflow run in parallel and one node needs to pull another node's data by name (instead of through the direct arrow connecting them), the referenced node sometimes hadn't finished yet, and the whole run failed. This happened consistently, not just once. I confirmed it wasn't a mistake in my own code by testing the same logic on its own, where it worked correctly every time.

Because of that, most of the numbers below come from running each of the five components on its own with realistic sample data pulled directly from the actual CSV files, rather than one single unbroken run through the whole pipeline. Every number is from a real API call with real output, just gathered branch by branch instead of in one continuous pass. I've marked below exactly which numbers were captured this way.

## Algorithm comparison

I compared two products: SKU-007 (Wool Gloves, currently going viral) and SKU-011 (Stainless Bottle, a stable, boring seller).

For the stable one, the math-based forecast said 75 units next month with a 97% confidence score. In plain terms, the recent past is a good guide here. The AI reviewer saw nothing unusual and left it alone. I would use that number as-is.

For the viral one, the math-based forecast said 354 units, but with only a 25% confidence score. The formula itself was saying it wasn't sure, it just couldn't explain why. That's the value of pairing a formula with an AI reviewer: the formula flags that something's off, and the AI is supposed to figure out what to do about it. In this case the AI had no outside signal to work with (no note about a viral video, no promotion), so it correctly left the number alone rather than guessing. If it had been told the product was picked up by an influencer last week, I'd expect it to revise the forecast upward. Without that signal, 354 is a reasonable floor, but I wouldn't bet the warehouse on it being the ceiling. This is the one case where I'd want a human to sanity check the number before it drives a purchase order.

## EOQ assumption analysis

I built three checks to catch situations where the standard reorder quantity formula shouldn't be trusted blindly.

- **Viral spike** (recent sales more than 2.5x the prior stretch): caught Wool Gloves. Recent months were running about 6x higher than the months before that.
- **Declining** (recent sales under half of where they started): caught Wired Earbuds, whose sales have been sliding for a year straight while the warehouse still holds nearly 7x its normal reorder level.
- **Long lead time plus unstable demand**: also caught Wool Gloves, since it takes 30 days to restock and demand is swinging wildly. That's a bad combination.
- **Low volume**: I built this check, but nothing in our current product list is slow moving enough to trigger it (the slowest seller still moves over 600 units a year). It never fired during testing, so I have not seen it in action, but I am leaving it in since it is still the right check to have.

When these flags fired, the AI recommended an emergency reorder for the viral item, sized off recent months' pace rather than the full year average, which is a smarter number than the plain formula gives you. For the declining item it recommended a markdown instead of a reorder. Both recommendations matched what I'd expect a human planner to do.

## Supplier rubric defense

I split the supplier score across four factors: reliability (35%), quality (30%), delivery speed (20%), and cost or payment terms (15%). I weighted reliability and quality highest because, for a company selling directly to customers, a late or defective shipment costs more than a slightly worse payment deadline. Our best and worst suppliers came out exactly where I'd expect from eyeballing their raw numbers. The top one has excellent on time delivery and almost no defects. The bottom one is weak on both despite offering the most generous payment terms in the group.

One catch: when I checked the AI's math by hand, its final scores did not always match what my stated weights would produce. The numbers were off by a few points each time. The category (preferred, watch, or deprioritize) was still always correct, so no real decision changed, but it shows that giving an AI a precise formula does not make it a calculator. For anything where the exact number matters, that arithmetic is better handled by regular code than by asking the AI to do it.

## Run metrics

Using real usage from actual test calls (Claude Sonnet 4.6 pricing: $3 per million words of input, $15 per million words of output):

| What ran | API calls | Cost | Time |
|---|---|---|---|
| Demand forecast check | 2 | $0.006 | 8.95 seconds, actually timed |
| Inventory reorder check | 2 | $0.009 | about 9 seconds, estimated (same shape as above, not separately timed) |
| Supplier scoring | 1 | $0.017 | not timed |
| Shipping and carrier decision | 13 | $0.082 | not timed |

The shipping branch cost the most by a wide margin. The reason is not that the shipping decisions themselves are hard. The workflow calls the AI once for every line in the shipping catalog, including the 10 catalog entries that are not orders at all, just reference options the AI has to look at and reject. Each of those calls still costs almost as much as a real one, for a one word answer. The number of calls matters as much as how hard each one is thinking.

## Reflection on the four primer questions

**Where does the formula catch the AI's mistake, or the other way around?** The formula's confidence score is what flagged the viral product as unreliable in the first place. The AI never would have known to be suspicious of its own forecast without that number. Going the other way, the AI's supplier scoring showed the reverse problem. It's good at judging suppliers by feel but not reliable at arithmetic, so a formula should double check its math.

**What happens if the planner names a step the system doesn't recognize?** It would just silently do nothing. The order would disappear from the process with no error, since the routing only checks for a few exact expected words. The fix is to clean up and standardize the AI's answer automatically before routing on it, instead of assuming it will always reply in exactly the expected format.

**Why does the reorder formula confidently get the declining product wrong?** Because the formula only looks at how much was sold on average across the whole year. It doesn't know the trend is heading downward. It has no concept of "yes, but less and less every month." The flag that catches this is the declining check, which specifically compares recent months to early months instead of just averaging everything together.

**Why is supplier scoring an AI's job, and reorder math a formula's job, and what would happen if you swapped them?** Supplier scoring is judgment heavy and qualitative. Whether 18 day delivery is good enough depends on business context an AI can reasonably weigh in on. Reorder math is pure arithmetic with one right answer. Swap them and you'd get an AI second guessing a calculation it should just trust, and a rigid formula trying to score suppliers on hard numbers when the real answer often depends on soft judgment calls the formula can't see.

## Demo video

https://www.loom.com/share/7c4be1c1a69443de8f6689baf791c87a
