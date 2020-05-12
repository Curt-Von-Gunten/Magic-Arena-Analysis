# <center> Magic Arena Economy Analysis </center>


```python
import numpy as np
import pandas as pd
import math
import matplotlib.pyplot as plt
#import seaborn as sns
#%matplotlib inline
# from IPython.display import Image

# from IPython.display import HTML
# def hide_code():
# 	return HTML('''<script>
# 	code_show=true; 
# 	function code_toggle() {
# 	 if (code_show){
# 	 $("div.input").hide();
# 	 } else {
# 	 $("div.input").show();
# 	 }
# 	 code_show = !code_show
# 	} 
# 	$( document ).ready(code_toggle);
# 	</script>
# 	To toggle on/off the raw code, click <a href="javascript:code_toggle()">here</a>.''')
# hide_code()

from IPython.display import HTML
HTML('''<script>
code_show=true; 
function code_toggle() {
 if (code_show){
 $('div.input').hide();
 } else {
 $('div.input').show();
 }
 code_show = !code_show
} 
$( document ).ready(code_toggle);
</script>
<form action="javascript:code_toggle()"><input type="submit" value="Click here to toggle on/off the raw code."></form>''')
```




<script>
code_show=true; 
function code_toggle() {
 if (code_show){
 $('div.input').hide();
 } else {
 $('div.input').show();
 }
 code_show = !code_show
} 
$( document ).ready(code_toggle);
</script>
<form action="javascript:code_toggle()"><input type="submit" value="Click here to toggle on/off the raw code."></form>




```python
# =============================================================================
# Ultra function
# =============================================================================

def nCr(n,r):
    f = math.factorial
    return f(n) // (f(r) * f(n-r))

def draftFormatStruct(draftFormat):
    if draftFormat == 'quick' or draftFormat == 'premier' or draftFormat == 'sealed':
        winRange = list(range(0,8,1))
    if draftFormat == 'traditional':
        winRange = list(range(0,4,1))
    return(winRange)

def recordProbs(winRate, draftFormat):    
    recordProbList = []
    if draftFormat == 'quick' or draftFormat == 'premier' or draftFormat == 'sealed':
        for wins in draftFormatStruct(draftFormat):
            if wins < 7:
                prob = (winRate**wins)*(1-winRate)**2*(nCr(wins+2,2))*(1-winRate)
                recordProbList.append(prob)
            else:
                prob = (winRate**wins) + ((winRate**(wins-1))*((1-winRate)**1)*(nCr((wins-1)+1,1))*winRate) + ((winRate**(wins-1))*((1-winRate)**2)*(nCr((wins-1)+2,2))*winRate)  
                recordProbList.append(prob)
    elif draftFormat == 'traditional':
        matchWinRate = (winRate**2) + (2 * (winRate**2) * (1-winRate))
        for wins in draftFormatStruct(draftFormat):
            losses = 3 - wins
            prob = (matchWinRate**wins)*(1-matchWinRate)**losses*(nCr(3,wins))
            recordProbList.append(prob)
    return recordProbList

def rewardsCalculator(draftFormat):
    if draftFormat == 'quick':
        gems = [50, 100, 200, 300, 450, 650, 850, 950]
        gemToPacks = [50/200, 100/200, 200/200, 300/200, 450/200, 650/200, 850/200, 950/200]
        packs = [1+1*.20, 1+1*.22, 1+1*.24, 1+1*.26, 1+1*.30, 1+1*.35, 1+1*.40, 2]
        entryCost = 750/200
        packConvTot = np.array(gemToPacks) + np.array(packs)
        #packConvTot = np.array(gemToPacks) + np.array(packs) + (draftPackValue/1000)*3 
        packConvTotNet = np.array(gemToPacks) + np.array(packs) - entryCost
        return gems, packs
    
    elif draftFormat == 'traditional':
        gems = [0, 0, 1000, 3000]
        gemToPacks = [0/200, 0/200, 1000/200, 3000/200]
        packs = [200/200, 200/200, 800/200, 1200/200]
        entryCost = 1500/200
        packConvTot = np.array(gemToPacks) + np.array(packs)
        #packConvTot = np.array(gemToPacks) + np.array(packs) + (draftPackValue/1000)*3 
        packConvTotNet = np.array(gemToPacks) + np.array(packs) - entryCost
        return gems, packs

    elif draftFormat == 'premier':
        gems = [50, 100, 250, 1000, 1400, 1600, 1800, 2200]
        gemToPacks = [50/200, 100/200, 250/200, 1000/200, 1400/200, 1600/200, 1800/200, 2200/200]
        packs = [200/200, 200/200, 400/200, 400/200, 600/200, 800/200, 1000/200, 1200/200]
        entryCost = 1500/200
        packConvTot = np.array(gemToPacks) + np.array(packs)
        #packConvTot = np.array(gemToPacks) + np.array(packs) + (draftPackValue/1000)*3 
        packConvTotNet = np.array(gemToPacks) + np.array(packs) - entryCost
        return gems, packs

    elif draftFormat == 'sealed':
        gems = [200, 400, 600, 1200, 1400, 1600, 2000, 2200]
        gemToPacks = [200/200, 400/200, 600/200, 1200/200, 1400/200, 1600/200, 2000/200, 2200/200]
        packs = [3] * 8
        entryCost = 2000/200
        packConvTot = np.array(gemToPacks) + np.array(packs)
        #packConvTot = np.array(gemToPacks) + np.array(packs) + (draftPackValue/1000)*3 
        packConvTotNet = np.array(gemToPacks) + np.array(packs) - entryCost
        return gems, packs

def evGemCalculator(winRate, draftFormat):    
        rewardsSummed = 0
        for prob, reward in zip(recordProbs(winRate, draftFormat), rewardsCalculator(draftFormat)[0]):
            rewardsSummed += prob * reward 
        return(rewardsSummed)

def evPackCalculator(winRate, draftFormat):    
        rewardsSummed = 0
        for prob, reward in zip(recordProbs(winRate, draftFormat), rewardsCalculator(draftFormat)[1]):
            rewardsSummed += prob * reward 
        return(rewardsSummed)

def numbDrafts(gemsStart, draftType, winRate, goldEntry=False):
    drafts = 0
    if goldEntry == False:
        if draftType == 'traditional' or draftType == 'premier':
            cost = 1500
        elif draftType == 'quick':
            cost = 750
        elif draftType == 'quick':
            cost = 750
        elif draftType == 'sealed':
            cost = 2000
    elif goldEntry == True:
        if draftType == 'traditional' or draftType == 'premier':
            cost = 10000/5
        elif draftType == 'quick':
            cost = 5000/5
        # elif draftType == 'sealed':
        #     cost = 2000
    while gemsStart > cost and gemsStart < 100001:
        drafts += gemsStart/cost 
        gemsStart = (gemsStart/cost) * evGemCalculator(winRate, draftType) 
    return drafts         

def numbPacks(gemStart, draftType, winRate):
    drafts = numbDrafts(gemStart, draftType, winRate) 
    return drafts * evPackCalculator(winRate, draftType)

def getCards(draftType, numbOfPacks, numbOfDrafts, raresPerDraft):
    rareWild = numbOfPacks * ((1*(1/30)) + (4/30)) # Number of rare wildcards
    mythicWild = numbOfPacks * ((1*(1/30)) + (1/30)) # Number of mythic wildcards
    if draftType != 'sealed':
        randomRares = numbOfPacks * (7/8) - numbOfPacks * (1/30) + (numbOfDrafts * ((7/8) * raresPerDraft)) # Numbe of random rare/mythics
        randomMythics = numbOfPacks * (1/8) - numbOfPacks * (1/30)  + (numbOfDrafts * ((1/8) * raresPerDraft)) # Numbe of random rare/mythics
    elif draftType == 'sealed':
        randomRares = numbOfPacks * (7/8) - numbOfPacks * (1/30) + (numbOfDrafts * ((7/8) * 6)) # Numbe of random rare/mythics
        randomMythics = numbOfPacks * (1/8) - numbOfPacks * (1/30)  + (numbOfDrafts * ((1/8) * 6)) # Numbe of random rare/mythics
    return round(randomRares,1), round(randomMythics,1), round(rareWild,1), round(mythicWild,1)

def moneyToGems(money, unit='cash'):
    if unit == 'gems':
        return money
    elif unit == 'gold':
        return money
    elif unit == 'cash':
        if money == 5:
            gems = 750
        elif money == 10:
            gems = 1600
        elif money == 20: 
            gems = 3400
        elif money == 50:
            gems = 9200
        elif money == 100:
            gems = 20000
        elif money == 200:
            gems = 40000
        return gems

def finalFunction(money, draftType, winRate, raresPerDraft, unit='cash', goldEntry=False):
    gems = moneyToGems(money, unit=unit)
    return round(numbDrafts(gems, draftType, winRate, goldEntry=goldEntry), 0), getCards(draftType, numbPacks(gems, draftType, winRate), numbDrafts(gems, draftType, winRate, goldEntry=goldEntry), raresPerDraft)

def toDataframe (draftType, money, winRateList, raresPerDraft, unit='cash', goldEntry=False):
    results = []
    for winRate in winRateList:
        temp = finalFunction(money, draftType, winRate, raresPerDraft)
        results.append(temp)

    df = pd.DataFrame(results, columns=['# of drafts', 'numb of cards'])
    df['win rate'] = np.array(winRateList)[0:len(results)] * 100
    df[['rareRand','mythicRand','rareWild','mythicWild']]  = df['numb of cards'].astype(str).str.split(',', expand=True)
    df.drop('numb of cards', axis=1, inplace=True)
    df['rareRand'] = df['rareRand'].str.strip('(').astype(float)
    df['mythicRand'] = df['mythicRand'].str.strip('(').astype(float)
    df['rareWild'] = df['rareWild'].str.strip(')').astype(float)
    df['mythicWild'] = df['mythicWild'].str.strip(')').astype(float)
    return df


```

<img src="Balance_v1.jpg" width="600" height="600" align="center"/>

<div class="alert alert-block alert-info">
<b>Note:</b> Many previous analyses use the average rewards (in gems or gold) per event as the primary outcome. This approach has some limitations. The following analysis:<br>1. Starts from X amount of money/gems.<br>2. Takes the gems you win from an event and inputs them into more events until you don't have enough gems to play again.<br>3. Accounts for the fact that quick draft entry is half the price of other draft formats.
</div>

# <center> 1. How many events can you play per amount of money invested</center>


```python
# =============================================================================
# Comparing number of events
# =============================================================================
money = 100
raresPerDraft = 3
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'sealed'
sealed = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)

###
fig = plt.figure(figsize=(12,8))
fig.add_subplot(111) # of drafts'

plt.plot(quick['win rate'][6:-3], quick['# of drafts'][6:-3], 'o-', color='#82AFD4', linewidth=3.0, markersize=12, label='Quick')
plt.plot(sealed['win rate'][6:-3], sealed['# of drafts'][6:-3], 'o-', color='#AE80B8', linewidth=3.0, markersize=12, label='Sealed')
plt.plot(traditional['win rate'][6:-3], traditional['# of drafts'][6:-3], 'o-', color='#87D1A1', linewidth=3.0, markersize=12, label='Traditional')
plt.plot(premier['win rate'][6:-3], premier['# of drafts'][6:-3], 'o-', color='#F86769', linewidth=3.0, markersize=12, label='Premier')

# plt.plot(quick['win rate'][6:-3], quick['# of drafts'][6:-3], 'o-', color='#8E87BE', linewidth=3.0, markersize=12, label='Quick')
# plt.plot(sealed['win rate'][6:-3], sealed['# of drafts'][6:-3], 'o-', color='#38BBC3', linewidth=3.0, markersize=12, label='Sealed')
# plt.plot(traditional['win rate'][6:-3], traditional['# of drafts'][6:-3], 'o-', color='#D07AA7', linewidth=3.0, markersize=12, label='Traditional')
# plt.plot(premier['win rate'][6:-3], premier['# of drafts'][6:-3], 'o-', color='#3987B8', linewidth=3.0, markersize=12, label='Premier')

# plt.plot(quick['win rate'][6:-3], quick['# of drafts'][6:-3], 'o-', color='#87074A', linewidth=3.0, markersize=12, label='Quick')
# plt.plot(sealed['win rate'][6:-3], sealed['# of drafts'][6:-3], 'o-', color='#550F3B', linewidth=3.0, markersize=12, label='Sealed')
# plt.plot(traditional['win rate'][6:-3], traditional['# of drafts'][6:-3], 'o-', color='#674374', linewidth=3.0, markersize=12, label='Traditional')
# plt.plot(premier['win rate'][6:-3], premier['# of drafts'][6:-3], 'o-', color='#CBA6C3', linewidth=3.0, markersize=12, label='Premier')

# plt.plot(quick['win rate'][6:-3], quick['# of drafts'][6:-3], 'o-', color='#356589', linewidth=3.0, markersize=12, label='Quick')
# plt.plot(sealed['win rate'][6:-3], sealed['# of drafts'][6:-3], 'o-', color='#3987B8', linewidth=3.0, markersize=12, label='Sealed')
# plt.plot(traditional['win rate'][6:-3], traditional['# of drafts'][6:-3], 'o-', color='#8E87BE', linewidth=3.0, markersize=12, label='Traditional')
# plt.plot(premier['win rate'][6:-3], premier['# of drafts'][6:-3], 'o-', color='#7F9CB2', linewidth=3.0, markersize=12, label='Premier')

# plt.plot(quick['win rate'][6:-3], quick['# of drafts'][6:-3], 'o-', color='#947E8D', linewidth=3.0, markersize=12, label='Quick')
# plt.plot(sealed['win rate'][6:-3], sealed['# of drafts'][6:-3], 'o-', color='#FBD1DD', linewidth=3.0, markersize=12, label='Sealed')
# plt.plot(traditional['win rate'][6:-3], traditional['# of drafts'][6:-3], 'o-', color='#27968B', linewidth=3.0, markersize=12, label='Traditional')
# plt.plot(premier['win rate'][6:-3], premier['# of drafts'][6:-3], 'o-', color='#EC8A9C', linewidth=3.0, markersize=12, label='Premier')

plt.xlabel('Win %', fontsize=22)
plt.ylabel('Number of events', fontsize=22)
plt.legend(fontsize=16)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
plt.title('Number of events from $100 (20,000 gems)', fontsize=26)
plt.show();
plt.close()

```


![png](output_6_0.png)


>## Quick draft clearly beats the other event types. What happens above a 60% win rate?


```python
# =============================================================================
# Comparing infinite
# =============================================================================
money = 100
raresPerDraft = 3
winRateList = list(np.array(range(56,76,2))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'sealed'
sealed = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)

fig = plt.figure(figsize=(12,8))
fig.add_subplot(111) # of drafts'
plt.plot(quick['win rate'][:-1], quick['# of drafts'][:-1], 'o-', color='#82AFD4', linewidth=3.0, markersize=12, label='Quick')
plt.plot(sealed['win rate'][:-1], sealed['# of drafts'][:-1], 'o-', color='#AE80B8', linewidth=3.0, markersize=12, label='Sealed')
plt.plot(traditional['win rate'][:-3], traditional['# of drafts'][:-3], 'o-', color='#87D1A1', linewidth=3.0, markersize=12, label='Traditional')
plt.plot(premier['win rate'][:-4], premier['# of drafts'][:-4], 'o-', color='#F86769', linewidth=3.0, markersize=12, label='Premier')
plt.xlabel('Win %', fontsize=22)
plt.ylabel('Number of events', fontsize=26)
plt.legend(fontsize=16)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(0, 250)
plt.title('When can I practically draft infinitely?', fontsize=26)
plt.show()


```


![png](output_8_0.png)


>## Traditional draft takes off quickly once your win rate inches above 60%. Note that this analysis indirectly tells you how many gems you win relative to the cost of entry (expected value).

# <center> 3. What is the best limited event if you're mainly concerned with growing your collection for constructed </center>

>## One important parameter for this analysis is how many rares/mythics you get on average per draft. For now, I'll assume you aren't rare drafting.<br>Let's assume 3 rares/mythics per draft.


```python
# =============================================================================
# If I’m primary concerned with growing my collection for constructed, what is the best limited event?
# =============================================================================
money = 100
raresPerDraft = 3
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'sealed'
sealed = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)
buyDf = pd.DataFrame([getCards('buying', 100, 0, 0)] * 15, columns=['rareRand', 'mythicRand', 'rareWild', 'mythicWild']) 
buyDf['win rate'] = np.array(winRateList) * 100

plt.close()
fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(14,9), sharex=True)
ax1.plot(quick['win rate'][6:-3], quick['rareRand'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax1.plot(sealed['win rate'][6:-3], sealed['rareRand'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax1.plot(traditional['win rate'][6:-3], traditional['rareRand'][6:-3], 'o-', color='#87D1A1', label='Traditional')
ax1.plot(premier['win rate'][6:-3], premier['rareRand'][6:-3], 'o-', color='#F86769', label='Premier')
ax1.plot(buyDf['win rate'] [6:-3], buyDf['rareRand'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax1.set_ylabel('Number of cards', fontsize=14)
ax1.legend(fontsize=12)
ax1.set_title('Random Rare', fontsize=16)
# ax1.axvline(x=50, linewidth=.5, color='k')

ax2.plot(quick['win rate'][6:-3], quick['mythicRand'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax2.plot(sealed['win rate'][6:-3], sealed['mythicRand'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax2.plot(traditional['win rate'][6:-3], traditional['mythicRand'][6:-3], 'o-', color='#87D1A1', label='Traditional')
ax2.plot(premier['win rate'][6:-3], premier['mythicRand'][6:-3], 'o-', color='#F86769', label='Premier')
ax2.plot(buyDf['win rate'] [6:-3], buyDf['mythicRand'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax2.legend(fontsize=14)
ax2.set_title('Random Mythic', fontsize=16)
# ax2.axvline(x=50, linewidth=.5, color='k')

ax3.plot(quick['win rate'][6:-3], quick['rareWild'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax3.plot(sealed['win rate'][6:-3], sealed['rareWild'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax3.plot(traditional['win rate'][6:-3], traditional['rareWild'][6:-3], 'mo-', color='#87D1A1',label='Traditional')
ax3.plot(premier['win rate'][6:-3], premier['rareWild'][6:-3], 'o-', color='#F86769',label='Premier')
ax3.plot(buyDf['win rate'] [6:-3], buyDf['rareWild'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax3.set_ylim(1, 50)
ax3.set_xlabel('Win %', fontsize=14)
ax3.set_ylabel('Number of cards', fontsize=14)
ax3.legend(fontsize=12)
ax3.set_title('Rare Wildcard', fontsize=16)
# ax3.axvline(x=50, linewidth=.5, color='k')

ax4.plot(quick['win rate'][6:-3], quick['mythicWild'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax4.plot(sealed['win rate'][6:-3], sealed['mythicWild'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax4.plot(traditional['win rate'][6:-3], traditional['mythicWild'][6:-3], 'o-', color='#87D1A1',label='Traditional')
ax4.plot(premier['win rate'][6:-3], premier['mythicWild'][6:-3], 'o-', color='#F86769',label='Premier')
ax4.plot(buyDf['win rate'] [6:-3], buyDf['mythicWild'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax4.set_ylim(1, 19)
ax4.set_xlabel('Win %', fontsize=14)
ax4.legend(fontsize=14)
ax4.set_title('Mythic Wildcard', fontsize=16)
# ax4.axvline(x=50, linewidth=.5, color='k')

fig.suptitle('Number and type of cards for $100 (20,000 gems) \n 3 rares/mythics per draft', fontsize=22)
fig.tight_layout(rect=[0, 0.03, 1, 0.93]);

```


![png](output_12_0.png)


>## Quick draft results in the most random rares and mythics for win rates of 55% and below. But what about the fact that just using your gems to buy packs results in more rare and mythic wilcards at 50% and that premier and traditional draft results in more wildcards at 55% and above?

>## To account for this we can decide how many random rare/mythics are worth one wildcard rare/mythic, and then compare.


```python
### Intrepretting
quick['rareRandDff'] = quick['rareRand'] - buyDf['rareRand']
quick['rareWildDff'] = buyDf['rareWild'] - quick['rareWild']
quick['mythicRandDff'] = quick['mythicRand'] - buyDf['mythicRand']
quick['mythicWildDff'] = buyDf['mythicWild'] - quick['mythicWild']
quick['mythicWildCorr'] = quick['mythicWildDff'] * 4
quick['mythicDiff'] = quick['mythicRandDff'] - quick['mythicWildCorr']
quick['rareWildCorr'] = quick['rareWildDff'] * 6
quick['rareDiff'] = quick['rareRandDff'] - quick['rareWildCorr']
quick[quick['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(quick['win rate'] [6:-2], quick['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(quick['win rate'] [6:-2], quick['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from quick draft', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-25, 200)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Quick draft vs buying \n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();

```


![png](output_15_0.png)


>## You can see that when assuming that a rare wildcard is equal to 6 random rares and that a mythic wildcard is equal to 4 mythic wildcards, quick draft is the better option compared to buying packs directly

>## What about quick draft compared to traditional and premier at 55% and above?


```python
### Intrepretting
quick['rareRandDff'] = quick['rareRand'] - traditional['rareRand']
quick['rareWildDff'] = traditional['rareWild'] - quick['rareWild']
quick['mythicRandDff'] = quick['mythicRand'] - traditional['mythicRand']
quick['mythicWildDff'] = traditional['mythicWild'] - quick['mythicWild']
quick['mythicWildCorr'] = quick['mythicWildDff'] * 4
quick['mythicDiff'] = quick['mythicRandDff'] - quick['mythicWildCorr']
quick['rareWildCorr'] = quick['rareWildDff'] * 6
quick['rareDiff'] = quick['rareRandDff'] - quick['rareWildCorr']
quick[quick['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(quick['win rate'] [6:-2], quick['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(quick['win rate'] [6:-2], quick['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from quick draft', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-50, 100)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Quick draft vs permier draft \n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();

```


![png](output_18_0.png)


>## You're slightly ahead in premier draft at 55%. Because traditional results in more cards than premier (see earlier graph), we know that you'll also be better off in traditional than quick draft as well.

>## However, I think my assumption above that you get an average of 3 rares/wildcards per draft is too high. In my experience the AVERAGE is maybe around 1 (often 0; often 2; when you're lucky 3). Let's rerun the analysis at 1 instead of 3.


```python
# =============================================================================
# If I’m primary concerned with growing my collection for constructed, what is the best limited event?
# =============================================================================
money = 100
raresPerDraft = 1
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'sealed'
sealed = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)
buyDf = pd.DataFrame([getCards('buying', 100, 0, 0)] * 15, columns=['rareRand', 'mythicRand', 'rareWild', 'mythicWild']) 
buyDf['win rate'] = np.array(winRateList) * 100

plt.close()
fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(14,9), sharex=True)
ax1.plot(quick['win rate'][6:-3], quick['rareRand'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax1.plot(sealed['win rate'][6:-3], sealed['rareRand'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax1.plot(traditional['win rate'][6:-3], traditional['rareRand'][6:-3], 'o-', color='#87D1A1', label='Traditional')
ax1.plot(premier['win rate'][6:-3], premier['rareRand'][6:-3], 'o-', color='#F86769', label='Premier')
ax1.plot(buyDf['win rate'] [6:-3], buyDf['rareRand'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax1.set_ylabel('Number of cards', fontsize=14)
ax1.legend(fontsize=12)
ax1.set_title('Random Rare', fontsize=16)
# ax1.axvline(x=50, linewidth=.5, color='k')

ax2.plot(quick['win rate'][6:-3], quick['mythicRand'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax2.plot(sealed['win rate'][6:-3], sealed['mythicRand'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax2.plot(traditional['win rate'][6:-3], traditional['mythicRand'][6:-3], 'o-', color='#87D1A1', label='Traditional')
ax2.plot(premier['win rate'][6:-3], premier['mythicRand'][6:-3], 'o-', color='#F86769', label='Premier')
ax2.plot(buyDf['win rate'] [6:-3], buyDf['mythicRand'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax2.legend(fontsize=14)
ax2.set_title('Random Mythic', fontsize=16)
# ax2.axvline(x=50, linewidth=.5, color='k')

ax3.plot(quick['win rate'][6:-3], quick['rareWild'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax3.plot(sealed['win rate'][6:-3], sealed['rareWild'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax3.plot(traditional['win rate'][6:-3], traditional['rareWild'][6:-3], 'mo-', color='#87D1A1',label='Traditional')
ax3.plot(premier['win rate'][6:-3], premier['rareWild'][6:-3], 'o-', color='#F86769',label='Premier')
ax3.plot(buyDf['win rate'] [6:-3], buyDf['rareWild'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax3.set_ylim(1, 50)
ax3.set_xlabel('Win %', fontsize=14)
ax3.set_ylabel('Number of cards', fontsize=14)
ax3.legend(fontsize=12)
ax3.set_title('Rare Wildcard', fontsize=16)
# ax3.axvline(x=50, linewidth=.5, color='k')

ax4.plot(quick['win rate'][6:-3], quick['mythicWild'][6:-3], 'o-', color='#82AFD4', label='Quick')
ax4.plot(sealed['win rate'][6:-3], sealed['mythicWild'][6:-3], 'o-', color='#AE80B8', label='Sealed')
ax4.plot(traditional['win rate'][6:-3], traditional['mythicWild'][6:-3], 'o-', color='#87D1A1',label='Traditional')
ax4.plot(premier['win rate'][6:-3], premier['mythicWild'][6:-3], 'o-', color='#F86769',label='Premier')
ax4.plot(buyDf['win rate'] [6:-3], buyDf['mythicWild'][6:-3], 'o-', color='#FFA07A', label='Buying packs')
ax4.set_ylim(1, 19)
ax4.set_xlabel('Win %', fontsize=14)
ax4.legend(fontsize=14)
ax4.set_title('Mythic Wildcard', fontsize=16)
# ax4.axvline(x=50, linewidth=.5, color='k')

fig.suptitle('Number and type of cards for $100 \n 1 rare/mythic per draft', fontsize=22)
fig.tight_layout(rect=[0, 0.03, 1, 0.93]);

```


![png](output_21_0.png)


>## You can see that changing from 3 to 1 rare/mythic per draft does two things:<br>1. Sealed beats quick draft and the other draft formats at 50% and below.<br>2. Quick draft is penalized more than traditional and premier.

>## Let's use the conversion above to compare sealed to buying packs.


```python
### Intrepretting
sealed['rareRandDff'] = sealed['rareRand'] - buyDf['rareRand']
sealed['rareWildDff'] = buyDf['rareWild'] - sealed['rareWild']
sealed['mythicRandDff'] = sealed['mythicRand'] - buyDf['mythicRand']
sealed['mythicWildDff'] = buyDf['mythicWild'] - sealed['mythicWild']
sealed['mythicWildCorr'] = sealed['mythicWildDff'] * 4
sealed['mythicDiff'] = sealed['mythicRandDff'] - sealed['mythicWildCorr']
sealed['rareWildCorr'] = sealed['rareWildDff'] * 6
sealed['rareDiff'] = sealed['rareRandDff'] - sealed['rareWildCorr']
sealed[sealed['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(sealed['win rate'] [6:-2], sealed['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(sealed['win rate'] [6:-2], sealed['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from sealed', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-50, 100)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Buying vs sealed \n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();
```


![png](output_24_0.png)


>## Sealed comes out ahead at 50% and greater, but not by as much as you might have expected.

>## Lets change the converstion so that a rare wildcard is worth 8 random rares (say you really just want to craft a specific deck quickly)


```python
### Intrepretting
sealed['rareRandDff'] = sealed['rareRand'] - buyDf['rareRand']
sealed['rareWildDff'] = buyDf['rareWild'] - sealed['rareWild']
sealed['mythicRandDff'] = sealed['mythicRand'] - buyDf['mythicRand']
sealed['mythicWildDff'] = buyDf['mythicWild'] - sealed['mythicWild']
sealed['mythicWildCorr'] = sealed['mythicWildDff'] * 4
sealed['mythicDiff'] = sealed['mythicRandDff'] - sealed['mythicWildCorr']
sealed['rareWildCorr'] = sealed['rareWildDff'] * 8
sealed['rareDiff'] = sealed['rareRandDff'] - sealed['rareWildCorr']
sealed[sealed['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(sealed['win rate'] [6:-2], sealed['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(sealed['win rate'] [6:-2], sealed['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from sealed', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-50, 100)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Buying vs sealed \n rareWild = 8 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();
```


![png](output_27_0.png)


>## In this case, buying packs gives you the same value in cards.

# <center> 3. Rare drafting </center>

>## Above, we saw that the number of rares/mythics per draft matters a lot. What happens when we disregard the quality of our draft deck and pick every rare/mythic that we see?

>## There are two main parameters for this analysis:<br>1. How many more rares/mythics you get per draft when rare drafting.<br>2. How much your win rate decreases.<br><br>For the first round, I'll assume you get 1 when not rare drafting and 5 when rare drafting, and that your win rate only decreases 5%.

<div class="alert alert-block alert-info">
<b>Note:</b> Below, a win rate of 50% is your win rate when not rare drafting (purple line). If rare drafting decreases your win rate by 5%, the blue lines at 50% would be the rewards at 50% - 5% (i.e., 45%).
</div>


```python
# =============================================================================
# Rare drafting
# =============================================================================
money = 100
raresPerDraft = 1
winRateList = list(np.array(range(35,61,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)

money = 100
raresPerDraft = 5
winRateList = list(np.array(range(30,56,5))/100)
draftType = 'quick'
quick_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier_rare = toDataframe(draftType, money, winRateList, raresPerDraft)

# # Lagging variables
# quick_rare['win rate_lag'] = quick_rare['win rate'].shift(periods=2, fill_value=0)
# premier_rare['win rate_lag'] = premier_rare['win rate'].shift(periods=2, fill_value=0)
# traditional_rare['win rate_lag'] = traditional_rare['win rate'].shift(periods=2, fill_value=0)

quick_rare['win rate_lag'] = quick_rare['win rate'] + 5
premier_rare['win rate_lag'] = premier_rare['win rate'] + 5
traditional_rare['win rate_lag'] = traditional_rare['win rate'] + 5

# Plotting
fig, ((ax1, ax2, ax3),(ax4, ax5, ax6),(ax7, ax8, ax9),(ax10, ax11, ax12)) = plt.subplots(4, 3, figsize=(14,12), sharex=True)

ax1.plot(quick_rare['win rate_lag'] [:], quick_rare['rareRand'][:], 'co-', label='Rare draft: yes')
ax1.plot(quick['win rate'] [:], quick['rareRand'][:], 'mo-', label='Rare draft: no')
ax1.legend()
ax1.set_title('Quick draft', fontsize=16)
ax1.set_ylabel('Random rare', fontsize=12)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax4.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicRand'][:], 'cs--', label='Rare draft = yes')
ax4.plot(quick['win rate'] [:], quick['mythicRand'][:], 'ms--', label='Rare draft = no')
ax4.set_ylabel('Random mythic', fontsize=12)
ax7.plot(quick_rare['win rate_lag'] [:], quick_rare['rareWild'][:], 'cv:', label='Rare draft = yes')
ax7.plot(quick['win rate'] [:], quick['rareWild'][:], 'mv:', label='Rare draft = no')
ax7.set_ylabel('Rare wildcard', fontsize=12)
ax10.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicWild'][:], 'cx-.', label='Rare draft = yes')
ax10.plot(quick['win rate'] [:], quick['mythicWild'][:], 'mx-.', label='Rare draft = no')
ax10.set_ylabel('Mythic wildcard', fontsize=12)

ax2.plot(traditional_rare['win rate_lag'] , traditional_rare['rareRand'], 'co-', label='Rare draft: yes')
ax2.plot(traditional['win rate'] , traditional['rareRand'], 'mo-', label='Rare draft: no')
#ax2.legend()
ax2.set_title('Traditional draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax5.plot(traditional_rare['win rate_lag'] , traditional_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax5.plot(traditional['win rate'] , traditional['mythicRand'], 'ms--', label='Rare draft = no')
ax8.plot(traditional_rare['win rate_lag'], traditional_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax8.plot(traditional['win rate'] , traditional['rareWild'], 'mv:', label='Rare draft = no')
ax11.plot(traditional_rare['win rate_lag'], traditional_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax11.plot(traditional['win rate'] , traditional['mythicWild'], 'mx-.', label='Rare draft = no');

ax3.plot(premier_rare['win rate_lag'], premier_rare['rareRand'], 'co-', label='Rare draft: yes')
ax3.plot(premier['win rate'], premier['rareRand'], 'mo-', label='Rare draft: no')
#ax3.legend()
ax3.set_title('Premier draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax6.plot(premier_rare['win rate_lag'] , premier_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax6.plot(premier['win rate'] , premier['mythicRand'], 'ms--', label='Rare draft = no')
ax9.plot(premier_rare['win rate_lag'] , premier_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax9.plot(premier['win rate'] , premier['rareWild'], 'mv:', label='Rare draft = no')
ax12.plot(premier_rare['win rate_lag'] , premier_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax12.plot(premier['win rate'] , premier['mythicWild'], 'mx-.', label='Rare draft = no')

fig.suptitle('Rare drafting:\n4 additional rares/mythics\n5% win rate decrease', fontsize=22);
```


![png](output_33_0.png)


>## The value of rare drafting depends on the draft format. You gain a ton from rare drafting in quick draft regardless of how good you are when not rare drafting. The other formats are hard to discern at 55%.

>## Before you read too much into the results above, I think the above assumptions don't penalize you enough for rare drafting. Let's now say that you get 1 rare/mythic when not rare drafting, 6 when rare drafting, and that your win rate decreases 10% (which is the lowest win rate that would be noticeable in a single draft event)


```python
# =============================================================================
# Rare drafting
# =============================================================================
money = 100
raresPerDraft = 1
winRateList = list(np.array(range(35,61,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)

money = 100
raresPerDraft = 6
winRateList = list(np.array(range(25,51,5))/100)
draftType = 'quick'
quick_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier_rare = toDataframe(draftType, money, winRateList, raresPerDraft)

# # Lagging variables
# quick_rare['win rate_lag'] = quick_rare['win rate'].shift(periods=2, fill_value=0)
# premier_rare['win rate_lag'] = premier_rare['win rate'].shift(periods=2, fill_value=0)
# traditional_rare['win rate_lag'] = traditional_rare['win rate'].shift(periods=2, fill_value=0)

quick_rare['win rate_lag'] = quick_rare['win rate'] + 10
premier_rare['win rate_lag'] = premier_rare['win rate'] + 10
traditional_rare['win rate_lag'] = traditional_rare['win rate'] + 10

# Plotting
fig, ((ax1, ax2, ax3),(ax4, ax5, ax6),(ax7, ax8, ax9),(ax10, ax11, ax12)) = plt.subplots(4, 3, figsize=(14,12), sharex=True)

ax1.plot(quick_rare['win rate_lag'] [:], quick_rare['rareRand'][:], 'co-', label='Rare draft: yes')
ax1.plot(quick['win rate'] [:], quick['rareRand'][:], 'mo-', label='Rare draft: no')
ax1.legend()
ax1.set_title('Quick draft', fontsize=16)
ax1.set_ylabel('Random rare', fontsize=12)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax4.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicRand'][:], 'cs--', label='Rare draft = yes')
ax4.plot(quick['win rate'] [:], quick['mythicRand'][:], 'ms--', label='Rare draft = no')
ax4.set_ylabel('Random mythic', fontsize=12)
ax7.plot(quick_rare['win rate_lag'] [:], quick_rare['rareWild'][:], 'cv:', label='Rare draft = yes')
ax7.plot(quick['win rate'] [:], quick['rareWild'][:], 'mv:', label='Rare draft = no')
ax7.set_ylabel('Rare wildcard', fontsize=12)
ax10.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicWild'][:], 'cx-.', label='Rare draft = yes')
ax10.plot(quick['win rate'] [:], quick['mythicWild'][:], 'mx-.', label='Rare draft = no')
ax10.set_ylabel('Mythic wildcard', fontsize=12)

ax2.plot(traditional_rare['win rate_lag'] , traditional_rare['rareRand'], 'co-', label='Rare draft: yes')
ax2.plot(traditional['win rate'] , traditional['rareRand'], 'mo-', label='Rare draft: no')
#ax2.legend()
ax2.set_title('Traditional draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax5.plot(traditional_rare['win rate_lag'] , traditional_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax5.plot(traditional['win rate'] , traditional['mythicRand'], 'ms--', label='Rare draft = no')
ax8.plot(traditional_rare['win rate_lag'], traditional_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax8.plot(traditional['win rate'] , traditional['rareWild'], 'mv:', label='Rare draft = no')
ax11.plot(traditional_rare['win rate_lag'], traditional_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax11.plot(traditional['win rate'] , traditional['mythicWild'], 'mx-.', label='Rare draft = no');

ax3.plot(premier_rare['win rate_lag'], premier_rare['rareRand'], 'co-', label='Rare draft: yes')
ax3.plot(premier['win rate'], premier['rareRand'], 'mo-', label='Rare draft: no')
#ax3.legend()
ax3.set_title('Premier draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax6.plot(premier_rare['win rate_lag'] , premier_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax6.plot(premier['win rate'] , premier['mythicRand'], 'ms--', label='Rare draft = no')
ax9.plot(premier_rare['win rate_lag'] , premier_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax9.plot(premier['win rate'] , premier['rareWild'], 'mv:', label='Rare draft = no')
ax12.plot(premier_rare['win rate_lag'] , premier_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax12.plot(premier['win rate'] , premier['mythicWild'], 'mx-.', label='Rare draft = no')

fig.suptitle('Rare drafting:\n5 additional rares/mythics\n10% win rate decrease', fontsize=22);
```


![png](output_36_0.png)


>## It seems that it's hard to go wrong rare drafting in quick draft. It's also clear that it's a disaster to rare draft in other formats if you're at or above 60%.

>## To be thorough let's do one more, where you get 1 rare/mythic when not rare drafting, 5 when rare drafting, and your win rate drops 25%, which may be closer to the truth in a competitive format like traditional.


```python
# =============================================================================
# Rare drafting
# =============================================================================
money = 100
raresPerDraft = 1
winRateList = list(np.array(range(35,61,5))/100)
draftType = 'quick'
quick = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier = toDataframe(draftType, money, winRateList, raresPerDraft)

money = 100
raresPerDraft = 7
winRateList = list(np.array(range(10,36,5))/100)
draftType = 'quick'
quick_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'traditional'
traditional_rare = toDataframe(draftType, money, winRateList, raresPerDraft)
draftType = 'premier'
premier_rare = toDataframe(draftType, money, winRateList, raresPerDraft)

# # Lagging variables
# quick_rare['win rate_lag'] = quick_rare['win rate'].shift(periods=2, fill_value=0)
# premier_rare['win rate_lag'] = premier_rare['win rate'].shift(periods=2, fill_value=0)
# traditional_rare['win rate_lag'] = traditional_rare['win rate'].shift(periods=2, fill_value=0)

quick_rare['win rate_lag'] = quick_rare['win rate'] + 25
premier_rare['win rate_lag'] = premier_rare['win rate'] + 25
traditional_rare['win rate_lag'] = traditional_rare['win rate'] + 25

# Plotting
fig, ((ax1, ax2, ax3),(ax4, ax5, ax6),(ax7, ax8, ax9),(ax10, ax11, ax12)) = plt.subplots(4, 3, figsize=(14,12), sharex=True)

ax1.plot(quick_rare['win rate_lag'] [:], quick_rare['rareRand'][:], 'co-', label='Rare draft: yes')
ax1.plot(quick['win rate'] [:], quick['rareRand'][:], 'mo-', label='Rare draft: no')
ax1.legend()
ax1.set_title('Quick draft', fontsize=16)
ax1.set_ylabel('Random rare', fontsize=12)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax4.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicRand'][:], 'cs--', label='Rare draft = yes')
ax4.plot(quick['win rate'] [:], quick['mythicRand'][:], 'ms--', label='Rare draft = no')
ax4.set_ylabel('Random mythic', fontsize=12)
ax7.plot(quick_rare['win rate_lag'] [:], quick_rare['rareWild'][:], 'cv:', label='Rare draft = yes')
ax7.plot(quick['win rate'] [:], quick['rareWild'][:], 'mv:', label='Rare draft = no')
ax7.set_ylabel('Rare wildcard', fontsize=12)
ax10.plot(quick_rare['win rate_lag'] [:], quick_rare['mythicWild'][:], 'cx-.', label='Rare draft = yes')
ax10.plot(quick['win rate'] [:], quick['mythicWild'][:], 'mx-.', label='Rare draft = no')
ax10.set_ylabel('Mythic wildcard', fontsize=12)

ax2.plot(traditional_rare['win rate_lag'] , traditional_rare['rareRand'], 'co-', label='Rare draft: yes')
ax2.plot(traditional['win rate'] , traditional['rareRand'], 'mo-', label='Rare draft: no')
#ax2.legend()
ax2.set_title('Traditional draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax5.plot(traditional_rare['win rate_lag'] , traditional_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax5.plot(traditional['win rate'] , traditional['mythicRand'], 'ms--', label='Rare draft = no')
ax8.plot(traditional_rare['win rate_lag'], traditional_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax8.plot(traditional['win rate'] , traditional['rareWild'], 'mv:', label='Rare draft = no')
ax11.plot(traditional_rare['win rate_lag'], traditional_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax11.plot(traditional['win rate'] , traditional['mythicWild'], 'mx-.', label='Rare draft = no');

ax3.plot(premier_rare['win rate_lag'], premier_rare['rareRand'], 'co-', label='Rare draft: yes')
ax3.plot(premier['win rate'], premier['rareRand'], 'mo-', label='Rare draft: no')
#ax3.legend()
ax3.set_title('Premier draft', fontsize=16)
#ax1.set_ylim(1, 300)
#ax1.axvline(x=50, linewidth=.5, color='k')
ax6.plot(premier_rare['win rate_lag'] , premier_rare['mythicRand'], 'cs--', label='Rare draft = yes')
ax6.plot(premier['win rate'] , premier['mythicRand'], 'ms--', label='Rare draft = no')
ax9.plot(premier_rare['win rate_lag'] , premier_rare['rareWild'], 'cv:', label='Rare draft = yes')
ax9.plot(premier['win rate'] , premier['rareWild'], 'mv:', label='Rare draft = no')
ax12.plot(premier_rare['win rate_lag'] , premier_rare['mythicWild'], 'cx-.', label='Rare draft = yes')
ax12.plot(premier['win rate'] , premier['mythicWild'], 'mx-.', label='Rare draft = no')

fig.suptitle('Rare drafting:\n5 additional rares/mythics\n25% win rate decrease', fontsize=22);
```


![png](output_39_0.png)


>## Even if you can manage a 60% win rate in quick draft when not rare drafting, you still come out very ahead if you drop to a 35% win rate when rare drafting. But if you're that good you should probably be trying to maintain a high win rate in traditional and not rare drafting.

# <center> 4. Should I spend my surplus gold directly on buying packs, or get cards indirectly by entering limited events with the marked up gold entry fee? </center>

>## Given the results so far, and assuming you have a roughly average win rate, we should compare buying packs to quick draft when rare drafting.

>## But for competitive people like myself who can't allow themselves to rare draft, let's assume you aren't rare drafting and that you get 1 rare/mythic per draft.


```python
# =============================================================================
# If I’m primarily concerned with growing my collection for constructed, should I (a) spend my surplus gold directly on buying packs, or (b) get cards indirectly by entering limited events with the marked up gold entry fee?
# =============================================================================

money = 100
raresPerDraft = 1
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
unit = 'gold'
goldEntry = True
quickGold = toDataframe(draftType, money, winRateList, raresPerDraft, unit=unit, goldEntry=goldEntry)

### All together
# fig = plt.figure(figsize=(18,10))
# fig.add_subplot(111) # of drafts'
# plt.plot(buyDf['win rate'] [6:-2], buyDf['rareRand'][6:-2], 'co-', label='Buying packs: rareRand')
# plt.plot(buyDf['win rate'] [6:-2], buyDf['mythicRand'][6:-2], 'cs--', label='Buying packs: mythicRand')
# plt.plot(buyDf['win rate'] [6:-2], buyDf['rareWild'][6:-2], 'cv:', label='Buying packs: rareWild')
# plt.plot(buyDf['win rate'] [6:-2], buyDf['mythicWild'][6:-2], 'cx-.', label='Buying packs: mythicWild')

# plt.plot(quickGold['win rate'] [6:-2], quickGold['rareRand'][6:-2], 'mo-', label='Quick Drafting: rareRand')
# plt.plot(quickGold['win rate'] [6:-2], quickGold['mythicRand'][6:-2], 'ms--', label='Quick Drafting: mythicRand')
# plt.plot(quickGold['win rate'] [6:-2], quickGold['rareWild'][6:-2], 'mv:', label='Quick Drafting: rareWild')
# plt.plot(quickGold['win rate'] [6:-2], quickGold['mythicWild'][6:-2], 'mx-.', label='Quick Drafting: mythicWild')

# plt.xlabel('Win %', fontsize=22)
# plt.ylabel('Cards', fontsize=26)
# plt.legend(fontsize=16)
# plt.xticks(fontsize=16, rotation=0)
# plt.yticks(fontsize=16, rotation=0)
# #plt.xlim(60, 76)
# plt.ylim(0, 150)
# plt.title('Buying vs drafting', fontsize=26)
# plt.show();

### Separate
fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(13,8), sharex=True)

ax1.plot(buyDf['win rate'] [6:-2], buyDf['rareRand'][6:-2], 'co-', label='Buying packs')
ax1.plot(quickGold['win rate'] [6:-2], quickGold['rareRand'][6:-2], 'mo-', label='Quick Drafting')
ax1.legend(fontsize=12)
ax1.set_title('Randon Rare', fontsize=16)
ax1.axvline(x=50, linewidth=.5, color='k')
#ax1.grid(b=True, which='major', axis='y')

ax2.plot(buyDf['win rate'] [6:-2], buyDf['mythicRand'][6:-2], 'cs--', label='Buying packs')
ax2.plot(quickGold['win rate'] [6:-2], quickGold['mythicRand'][6:-2], 'ms--', label='Quick Drafting')
ax2.legend(fontsize=12)
ax2.set_title('Randon Mythic', fontsize=16)
ax2.axvline(x=50, linewidth=.5, color='k')
#ax2.grid(b=True, which='major', axis='y')

ax3.plot(buyDf['win rate'] [6:-2], buyDf['rareWild'][6:-2], 'cv:', label='Buying packs')
ax3.plot(quickGold['win rate'] [6:-2], quickGold['rareWild'][6:-2], 'mv:', label='Quick Drafting')
#ax3.set_ylim(1, 50)
ax3.legend(fontsize=12)
ax3.set_title('Rare Wildcard', fontsize=16)
ax3.axvline(x=50, linewidth=.5, color='k')
#ax3.grid(b=True, which='major', axis='y')

ax4.plot(buyDf['win rate'] [6:-2], buyDf['mythicWild'][6:-2], 'cx-.', label='Buying packs')
ax4.plot(quickGold['win rate'] [6:-2], quickGold['mythicWild'][6:-2], 'mx-.', label='Quick Drafting')
ax4.legend(fontsize=12)
ax4.set_title('Mythic Wilcard', fontsize=16)
ax4.axvline(x=50, linewidth=.5, color='k')
#ax4.grid(b=True, which='major', axis='y')

fig.suptitle('Buying packs vs. drafting with gold entry', fontsize=22)
fig.tight_layout(rect=[0, 0.03, 1, 0.95]);

### Intrepretting
quickGold['rareRandDff'] = quickGold['rareRand'] - buyDf['rareRand']
quickGold['rareWildDff'] = buyDf['rareWild'] - quickGold['rareWild']
quickGold['mythicRandDff'] = quickGold['mythicRand'] - buyDf['mythicRand']
quickGold['mythicWildDff'] = buyDf['mythicWild'] - quickGold['mythicWild']
quickGold['mythicWildCorr'] = quickGold['mythicWildDff'] * 4
quickGold['mythicDiff'] = quickGold['mythicRandDff'] - quickGold['mythicWildCorr']
quickGold['rareWildCorr'] = quickGold['rareWildDff'] * 6
quickGold['rareDiff'] = quickGold['rareRandDff'] - quickGold['rareWildCorr']
quickGold[quickGold['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(quickGold['win rate'] [6:-2], quickGold['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(quickGold['win rate'] [6:-2], quickGold['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from draft', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-25, 200)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Conversion \n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();


```


![png](output_44_0.png)



![png](output_44_1.png)


>## You're worse off drafting at 50%. Let's say you get 2 rare/mythics per draft instead. Let's also remove the first set of graphs and just look at the conversion graph.


```python
# =============================================================================
# If I’m primarily concerned with growing my collection for constructed, should I (a) spend my surplus gold directly on buying packs, or (b) get cards indirectly by entering limited events with the marked up gold entry fee?
# =============================================================================
money = 100
raresPerDraft = 2
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
unit = 'gold'
goldEntry = True
quickGold = toDataframe(draftType, money, winRateList, raresPerDraft, unit=unit, goldEntry=goldEntry)

### Intrepretting
quickGold['rareRandDff'] = quickGold['rareRand'] - buyDf['rareRand']
quickGold['rareWildDff'] = buyDf['rareWild'] - quickGold['rareWild']
quickGold['mythicRandDff'] = quickGold['mythicRand'] - buyDf['mythicRand']
quickGold['mythicWildDff'] = buyDf['mythicWild'] - quickGold['mythicWild']
quickGold['mythicWildCorr'] = quickGold['mythicWildDff'] * 4
quickGold['mythicDiff'] = quickGold['mythicRandDff'] - quickGold['mythicWildCorr']
quickGold['rareWildCorr'] = quickGold['rareWildDff'] * 6
quickGold['rareDiff'] = quickGold['rareRandDff'] - quickGold['rareWildCorr']
quickGold[quickGold['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(quickGold['win rate'] [6:-2], quickGold['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(quickGold['win rate'] [6:-2], quickGold['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from draft', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-25, 200)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Buying packs vs. drafting with gold entry \n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();
```


![png](output_46_0.png)


>## With the additional 1 rare/mythic you come out slightly ahead drafting.

>## What about when rare drafting in quick draft, which is what you should be doing if you're primary interested in growing your collection and are fine with losses.<br>Let's say 5 rare/mythics when rare drafting, and a 25% decrease in win rate to be extreme and test the limit.


```python
# =============================================================================
# If I’m primarily concerned with growing my collection for constructed, should I (a) spend my surplus gold directly on buying packs, or (b) get cards indirectly by entering limited events with the marked up gold entry fee?
# =============================================================================
money = 100
raresPerDraft = 5
winRateList = list(np.array(range(5,76,5))/100)
draftType = 'quick'
unit = 'gold'
goldEntry = True
quickGold = toDataframe(draftType, money, winRateList, raresPerDraft, unit=unit, goldEntry=goldEntry)

quickGold['win rate_lag'] = quickGold['win rate'] + 25

### Intrepretting
quickGold['rareRandDff'] = quickGold['rareRand'] - buyDf['rareRand']
quickGold['rareWildDff'] = buyDf['rareWild'] - quickGold['rareWild']
quickGold['mythicRandDff'] = quickGold['mythicRand'] - buyDf['mythicRand']
quickGold['mythicWildDff'] = buyDf['mythicWild'] - quickGold['mythicWild']
quickGold['mythicWildCorr'] = quickGold['mythicWildDff'] * 4
quickGold['mythicDiff'] = quickGold['mythicRandDff'] - quickGold['mythicWildCorr']
quickGold['rareWildCorr'] = quickGold['rareWildDff'] * 6
quickGold['rareDiff'] = quickGold['rareRandDff'] - quickGold['rareWildCorr']
quickGold[quickGold['win rate'] == 50]

fig = plt.figure(figsize=(10,6))
fig.add_subplot(111) # of drafts'
plt.plot(quickGold['win rate'] [6:-2], quickGold['rareDiff'][6:-2], 'co-', linewidth=2, markersize=12, label='Rare difference')
plt.plot(quickGold['win rate'] [6:-2], quickGold['mythicDiff'][6:-2], 'mo-', linewidth=2, markersize=12, label='Mythic difference')
plt.xlabel('Win %', fontsize=18)
plt.ylabel('How many more cards from draft', fontsize=18)
plt.legend(fontsize=18)
plt.xticks(fontsize=16, rotation=0)
plt.yticks(fontsize=16, rotation=0)
#plt.xlim(60, 76)
plt.ylim(-25, 200)
plt.axhline(y=0, linewidth=.5, color='k')
plt.title('Buying packs vs. Gold entry rare drafting\n rareWild = 6 randRares \n mythicWild = 4 randMythics', fontsize=20, y=1.03)
plt.show();
```


![png](output_49_0.png)


<div class="alert alert-block alert-info">
<b>Note:</b> To use this graph correctly, use your win rate when NOT rare drafting; the graph corrects for you.
</div>

>## Apparently you should take that gold and snag every rare you see with reckless abandon, regardles of the loss of dignity that comes with losing.

# <center> Thanks for reading. Please provide suggestions and corrections. </center>

<img src="Merchant_v1.jpg" width="750" height="750" align="center"/>
