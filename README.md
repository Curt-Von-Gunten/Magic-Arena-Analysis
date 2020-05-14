# <center> Magic Arena Economy Analysis </center>

<img src="Images/Balance_v1.jpg" width="600" height="600" align="center"/>

<div class="alert alert-block alert-info">
<b>Note:</b> Many previous analyses use the average rewards (in gems or gold) per event as the primary outcome. This approach has some limitations. The following analysis:<br>1. Starts from X amount of money/gems.<br>2. Takes the gems you win from an event and inputs them into more events until you don't have enough gems to play again.<br>3. Accounts for the fact that quick draft entry is half the price of other draft formats.
</div>

# <center> 1. How many events can you play per amount of money invested</center>

![png](Figures/output_6_0.png)

>## Quick draft clearly beats the other event types. What happens above a 60% win rate?

![png](Figures/output_8_0.png)

>## Traditional draft takes off quickly once your win rate inches above 60%. Note that this analysis indirectly tells you how many gems you win relative to the cost of entry (expected value).

# <center> 3. What is the best limited event if you're mainly concerned with growing your collection for constructed </center>

>## One important parameter for this analysis is how many rares/mythics you get on average per draft. Let's first consider the case where you don't rare draft at all. You only take a rare/mythic that's naturally fits your deck. Based on feedback from Reddit users, let's assume that the AVERAGE number is 1.5 rares/mythics per draft.

![png](Figures/output_12_0.png)

>## At 50% and below, it is clear that Sealed and Quick Draft are preferable to Premier and Tradition. But above 50% it is hard to interpret because Quick Draft and Sealed result in more random rares/mythics, but less wildcards than Premier and Limited.There's also the fact that just using your gems to buy packs results in more rare and mythic wilcards at 50% but less random rare/mythics.

>## To account for this we can decide how many random rare/mythics are worth one wildcard rare/mythic, and then compare. <br>

![png](Figures/output_15_0.png)

>## Later in the season once you start getting duplicates, wildcards will be more valuable. Let's change the conversion rate so that wildcards are more valuable.

![png](Figures/output_17_0.png)

>## Buying packs is surprisingly good at 50% and below.

# <center> 3. Rare drafting </center>

>## Above, we assumed you were trying to build the best deck and getting 1.5 rares/mythics per draft. What happens when we disregard the quality of our draft deck and pick every rare/mythic that we see?

>## There are two main parameters for this analysis:<br>1. How many more rares/mythics you get per draft when rare drafting.<br>2. How much your win rate decreases.<br><br> This is tricky because rares and mythics are more commonly passed in some formats than others. Luckily MTGGoldfish just published these numbers (https://www.mtggoldfish.com/articles/collecting-mtg-arena-ikoria-edition):<br><br>Quick Draft - you see 3.66 Rares and 0.42 Mythics per draft <br>Premier Draft - you see 6.7 Rares and 0.47 Mythics per draft <br>Traditional Draft - you see 7.6 Rares and 0.60 Mythics per draft

<div class="alert alert-block alert-info">
<b>Note:</b> Below, a win rate of 50% is your win rate when not rare drafting (purple line). If rare drafting decreases your win rate by 5%, the blue lines at 50% would be the rewards at 50% - 5% (i.e., 45%).
</div>

![png](Figures/output_23_0.png)

<div class="alert alert-block alert-info">
<b>Note:</b> To use this graph correctly, use your win rate when NOT rare drafting; the graph corrects for you.
</div>

>## This is a ridiculous wall of information. Let's reduce it with our conversion method above.

![png](Figures/output_26_0.png)

# <center> 4. Should I spend my surplus gold directly on buying packs, or get cards indirectly by entering limited events with the marked up gold entry fee? </center>

![png](Figures/output_28_0.png)

![png](Figures/output_28_1.png)

# <center> Thanks for reading. Please provide suggestions and corrections. </center>

<img src="Images/Merchant_v1.jpg" width="750" height="750" align="center"/>

