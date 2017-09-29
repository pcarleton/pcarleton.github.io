
I want to revisit what I am trying to get out of cashcoach in order to focus my efforts going forward.

The first question I wanted to answer was: How much money did I spend last month?

For a couple reasons involving splitting rent with my girlfriend and splitting a credit card with her, it was actually a fairly difficult question to answer.  I was about to leave my job and have no income, so the question I was really trying to answer was: How long do I have before I run out of money?

Now that I have income again, it's more related to: Am I saving enough money?

The first question, "How much did I spend last month?" is a question that has a more or less objective answer.  The second question is fairly objective as well, although it ignores the knobs you can turn about reducing spending, or dipping more into savings.

This specific situation didn't apply going forward, but I was hopeful there were general purpose Ideas I could extract from it.

The third question gets more abstract.  Am I saving enough money for what?  I had vague goals of retiring at some point.  A down payment on a house would be a cool thing so I don't keep burning money on rent.  However, neither of these were very specific.

My girlfriend gave me trouble because she asked if I would be willing to chip in for a new lamp for our front hall, and I ended up spending a weekend estimating my life expectancy, retirement age and tax burden.  Is this going to happen every time I want to buy something?

The answer is, no, I would not like to re-do these calculations each time I buy something.  I want to know if I am on track or off track of reaching my goals.  I want to know that in a broad sense.

The next question is what does that tool look like?  

A related question is how is this tool different than what already exists? Could I get it through existing tools?

- Existing tools (Mint) have trouble with my split rent and split credit card
- I generally find myself frustrated with the amount of upkeep required to massage my financial data into a format that works for my spending analysis.  Often a few large miscategorized purchases throw everything off.
  * It would be nice if I could write a script to fix these, so I don't end up recategorizing anything.
  * Having tags so I could see how much money a particular trip cost in order to forecast future trips would be useful too, although, I think just simple tagging would work because I would probably want to itemize the trip myself.  This one is a bit blurry at the moment.
- I liked Level Money's simplistic approach: How much have you spent etc.
- Spreadsheets are general purpose and what I end up using, so feeding a spreadsheet seems like a simple way to dovetail into my existing workflow
- Having models of taxes, Life insurance, and other financial instruments would be neat, but not immediately necessary to be useful.
- If I want broader adoption, having fallbacks for uploading CSV's or manually entering income etc. would be useful for people that don't want to enter their financial data.
-

Cont'd on Tuesday

I went out in the weeds on Monday.  My problem is not a shortage of ideas of what to do with this data, my problem is determining which idea is what I actually want and will use.  To some extent, just building and experimenting will surface some of that, but seeing as how I have limited time to work on this, some careful thought on what the underlying thing I want to get out of it seems worth it.

So back to my scenario where I was not comfortable spending money on a lamp.  What are the components of that discomfort?

- Not knowing how much I was spending, both in absolute terms, and in % terms of my salary
- Not knowing how much was too much in order to save enough for retirement (or other goals)

So there are two pieces here.  Working backwards, I first need some way of calculating what my goals are.  I made a spreadsheet doing some calculations about interest and retirement age etc. which gave me more confidence in what the ballpark of goals should be.  This was like the base of the pyramid of my financial confidence.

The next piece to the puzzle would be tracking those goals against my actual spending.  However, having a simple on/off track indicator would only be good if it always showed on-track, because if it showed off-track, I would just get anxiety about spending too much with out having concrete steps to get back on-track.

That might be getting ahead of myself though.  An on-track/off-track meter would be pretty powerful.  It could say tracking above, below or in range for my goals.  If it's tracking above, i.e. I'm saving too much towards a goal, that would be a useful signal as well because it would tell me I should come up with a new goal for that money.  I want to be deliberate with what I do with my money, so having a plan for what I do with the extra seems like a good idea.  If I want to pump it into retirement to achieve financial independence sooner, that seems reasonable, but it also seems reasonable to want to buy a house.

So the above/on/below meter would lead to several actions:

- Above: auxiliary goals like saving for downpayment on a house
- On: No action? Except maybe to also come up with auxiliary goals and then reduce spending
- Below: Look in to ways to reduce spending

Creating auxiliary goals would probably involve a custom spreadsheet/model that would dig into the variables, like how expensive of a house, what the mortgage would be, how that would factor into spending long term, etc.  It would require a good amount of research.  This research/modeling would be the part that would be helpful to share amongst other users of this platform if I eventually got to it.

So we definitely want a goal setting piece.  And ideally a tracking piece to measure how we're looking against the goals in real time.

The tracking piece could go a number of ways.  The source of truth would be the $ in a particular account, or group of accounts.  I think this is why Mint does it this way.  Its unambiguous.  However, the balance in an account fluctuates if it's invested in the market, so there would be a fair amount of noise.

So there would be several measures of tracking the goal.  One would be the actual balance, the other would be the #/size of deposits to the account.

The third would be the overall spending rate.  This is sort of the flip side of the savings rate.  If the spending rate is high, its the most actionable way to adjust savings.


So back to the overall structure of the project, we have the ability to set goals.  These goals have an ongoing tracking component in terms of balance, deposits, and spending.  Only one of those would suffice for the initial version.  Probably spending since its the easiest to track with CC statements and most in my control.

So we're back to the original question of how do I track my spending.  Start out with the balances on the CC accounts in full, then worry about adding in filters etc. to massage them.
