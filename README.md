# Poker Engine, AI Poker 2025

## Overview
This project provides a framework for simulating poker games. It allows you to create custom poker players by extending a base Player class and overriding the action function. 

## Features
Standard Texas Hold 'Em poker with 52 cards (no jokers) except for some minor changes. The changes include:
- There is no side pot. This means that winning after going all in will also reward you with the raises made subsequently.
- There is only one blind.
- Aces are treated as either high or low depending on which gives a better hand.

## Getting Started

To get started with the Poker Engine, first clone the repository using:

```bash
git clone https://github.com/Tanish-0001/AI-Poker-2025.git
```

Then you can create a custom player by implementing a class that inherits from the base Player class, and implement the function ```action```. This function must return an action (of the type PlayerAction) and an amount. Your player instance must be created with the parameters name and stack. Note that if you are using ```__init__```, you must call ```super().__init__()``` with the parameters name and stack.

Optionally, if you would like to play poker yourself, you can use the ```InputPlayer``` player provided in baseplayers.py.
    

### Example

Here’s a simple example of how to create a custom player class:

```python
from player import Player, PlayerAction

class MyPlayer(Player):
    def __init__(self, name, stack, strategy="fold"):
        super().__init__(name, stack)
        self.strategy = strategy

    def action(self, game_state, action_history):
        # Implement your strategy here
        if self.strategy == "fold":
            return PlayerAction.FOLD, 0
        else:
            return PlayerAction.ALL_IN, self.stack
```

### Game State

The game state that ```action``` receives is structured in the following way:
1. Hole Cards' Index (suit order is spades, hearts, diamonds, clubs and rank order is 2, 3, ..., Q, K, Ace)
2. Community Cards' Index. 0 means not yet dealt
3. Pot
4. Current Raise Amount
5. Blind
6. Active Player Index
7. Number of players
8. Each player's stack
9. Hand number - Maintains a count of how many hands have been dealt at the table.

For example, here is a sample game_state:
```python
[16, 7, 0, 0, 0, 0, 0, 20, 20, 20, 3, 4, 1000, 1000, 980, 1000, 1]
```
Which means that the player has the cards with index 16 and 7 (which corresponds to the 4 of hearts and 7 of spades), the community cards are all unrevealed, the pot has $20, the current raise is $20 (which includes the blind), the blind is $20, the active player index is 3 (which means its the 4th player's turn), the number of players is 4 and they have $1000, $1000, $980, $1000 resepctively, and the game number is 1.

### Action History

The action history is a list of tuples consisting of the following data: (phase, name, action, amount). Here is a sample action history:
```python
[('pre-flop', 'David', 'call', 20),
('pre-flop', 'Alice', 'raise', 60),
('pre-flop', 'Bob', 'fold', 0),
('pre-flop', 'Charlie', 'fold', 0),
('pre-flop', 'David', 'call', 40),
('flop', 'David', 'check', 0),
('flop', 'Alice', 'check', 0)]
```
Note that the action history does not include posting of the blind. The above action history follows a game where Charlie posts a blind of 20, followed by David calling 20, Alice raising to 60, Bob folding, Charlie folding, David calling 40, David checking, Alice checking.

Note: If you find any bugs, please e-mail them to ieee.sb@pilani.bits-pilani.ac.in
## Strategic Report: The AllInCounterBot
### Overall Goal:  
The AllInCounterBot is designed with a primary mission: to effectively counter opponents who frequently go all-in, especially before any community cards are dealt (pre-flop). Beyond this specialty, it aims to play a reasonably solid and adaptive game after the community cards appear, with a particular eye for spotting and responding to aggressive betting.  
#### How it Works & What it Considers:  
The bot's decision-making process is divided into two main phases: Pre-Flop (before community cards) and Post-Flop (after community cards).
I. Pre-Flop Strategy (Before Community Cards):  
At this stage, the bot only knows its own two starting cards.  
* Assessing Its Starting Hand:  
The bot first evaluates the raw strength of its two cards. It has an internal "ranking" system that considers:  
* Pairs: High pairs (like Aces, Kings) are considered very strong, medium pairs (like Eights, Nines) good, and low pairs (like Twos, Threes) decent.  
* Suited Cards: Two cards of the same suit (e.g., Ace-King of hearts) are more valuable than unsuited cards, especially if they are high-ranking or can form straights.  
* Connected Cards: Cards close in rank (e.g., Ten-Jack) are good because they can make straights.  
* High Cards: Generally, higher-ranking cards (Ace, King, Queen) are better.  
#### Responding to Opponent Actions:  
THE CORE: Facing an All-In: This is where the bot's special programming shines.  
It has a wider-than-usual range of hands it's willing to call an all-in with. It's looking to catch players who might be shoving with weaker hands than they should.  
If its starting hand meets a specific "good enough" threshold (e.g., any pair, strong Aces like Ace-Ten or better, good suited connectors like King-Queen suited), it will call the all-in.  
If its hand is below this threshold, it will fold.  
#### Facing a Normal Bet or Raise (Not an All-In):  
With its strongest hands (e.g., premium pairs like Queens, Kings, Aces; Ace-King): It will likely re-raise, trying to build the pot or get weaker hands to fold. If it can't re-raise effectively but has more chips than the call, it might go all-in itself.  
With good, but not premium, hands (e.g., medium pairs, suited Aces, good connectors): It will typically just call the bet.  
With weak hands: It will fold.  
If No One Has Bet Yet (It's the Bot's Turn to Act First, or Others Checked):  
With good to strong hands: It will make a standard opening bet (usually 3-4 times the big blind).  
With weaker hands: It will check (bet nothing, passing the action).  
II. Post-Flop Strategy (After Community Cards are Dealt):  
Once the Flop, Turn, or River cards are out, the bot combines its two starting cards with the community cards to see what poker hand it has.
Evaluating Its Current Hand Strength:  
The bot uses a hand evaluator to determine its best five-card hand (e.g., Pair, Two Pair, Flush, Straight, etc.). It assigns a "confidence" score to this hand, considering both the rank (e.g., a Flush is better than a Pair) and the specific cards (e.g., a Pair of Aces is better than a Pair of Twos).  
#### Detecting Opponent Aggression:  
This is a key adaptive feature post-flop. The bot looks at how an opponent bets:  
Is the bet very large compared to the size of the big blind?  
Is the bet very large compared to the current size of the pot?  
Did the opponent go all-in?  
If the answer to any of these is "yes," the bot flags the opponent's bet as "aggressive." Interestingly, what it considers "aggressive" can also depend slightly on its own hand strength – if the bot has a weak hand itself, even a moderately sized bet from an opponent might seem more aggressive.  
#### Responding to Opponent Actions (Post-Flop):  
With Very Strong Hands (e.g., Two Pair or better):  
If no one has bet: The bot will bet, usually a good fraction of the pot, trying to get value. The stronger its hand, the more it might bet. It might even go all-in.  
If an opponent bets: The bot will usually raise. If the opponent's bet was deemed "aggressive" and the bot has an exceptionally strong hand (like a Full House or better), it might just call to "trap" the opponent, hoping they bet more on the next street.  
With Medium Strength Hands (e.g., a decent pair, a good draw to a straight or flush):  
If no one has bet: The bot will usually check.  
If an opponent bets: Its decision depends heavily on the perceived aggression and the cost of calling.  
If the bet is deemed "aggressive," the bot is more likely to call, as it might be getting good odds or suspecting a bluff.  
If the bet is not overly aggressive and the call amount is a reasonable portion of its stack, it might call.  
If the bet is large and not particularly aggressive, or its hand is on the weaker side of "medium," it will likely fold.  
With Weak Hands (e.g., just a high card, a very weak pair):  
If no one has bet: The bot will check.  
If an opponent bets: The bot will usually fold.  
#### SPECIAL "HERO CALL" CONDITION:  
There's one specific exception. If an opponent goes all-in, the bot only has Ace-high (no pair or better), AND the amount to call is a small percentage (e.g., 15% or less) of its current chip stack, it might make the call. This is a "hero call," banking on the chance the opponent is bluffing with something even weaker.  
#### Randomness (Minor Element):  
If the bot has an extremely strong hand post-flop (like a Full House or better), it will introduce a tiny bit of randomness into its bet sizing. This makes it slightly less predictable to very observant opponents.  
#### How it Adapts:  
The bot's adaptation primarily occurs within a single hand based on the immediate game situation and opponent actions:
* Pre-Flop All-In Adaptation: Its primary "adaptation" here is its pre-programmed wider calling range specifically against all-ins. It doesn't learn which specific opponent is aggressive over time, but it's always ready to counter an all-in with a broader set of hands than it would use for a normal call.  
* Post-Flop Aggression Detection: As described above, it dynamically assesses if an opponent's bet is aggressive. Its response then adapts:  
* More willing to call with medium hands against aggression.  
* More likely to raise (or trap) with strong hands against aggression.  
* Post-Flop Hero Call: This is a specific situational adaptation, changing its default "fold weak hands" behavior when narrow conditions (all-in, Ace-high, small call relative to stack) are met.  
Bet Sizing based on Hand Strength: Post-flop, its bet amounts with strong hands scale with its perceived hand strength – stronger hands lead to bigger bets/raises.  
The bot does not learn or build profiles of specific opponents over multiple hands in this particular script. Its "adaptation" is reactive to the current betting round and hand dynamics.  
#### Summary:
The AllInCounterBot is a specialist designed to punish pre-flop over-aggression. It achieves this by calling all-ins with a relatively wide range of starting hands. Post-flop, it plays a more standard game but incorporates a system to detect and react to aggressive bets, making it more willing to continue with drawing or medium-strength hands if it suspects an opponent is trying to bully it. It also has a specific, risky "hero call" maneuver for certain all-in situations. Its actions are primarily driven by its own hand strength and the immediate actions of its opponents in the current hand.
