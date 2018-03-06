# Constraints on Power Workspace

A blog post series which will explore the Ethereum technology through the  
lense of constraints on power. This will likely include posts on:

* Plasma MVP constraint analysis
* Centralized blockchain risk
* AGI defense system
* Diverse stakeholder call to action

---

# Plasma MVP Exit Constraint Analysis

A key part of decentralization is imposing constraints on authority. These constraints limit actions of agents in a system forcing them to engage in constructive behavior. When all agents are constrained in a way in which incentives are aligned, a strong sort of trust is established. With cryptoeconomics we are able to design decentralized protocols which precisely define these constraints.

Plasma MVP is the first simple Plasma specification. It provides scalable payments using a central operator while maintaining strong security guarantees. In this post we will use payoff matrixes to identify central operators' poorly aligned incentives, address them by imposing constraints, and end up recreating the current Plasma MVP specification.

Let's begin!

## Unconstrained Central Payment Operator

One could naively scale payments with an unconstrained payment operator. In this scheme, users would first send ETH to a smart contract. This smart contract is completely controlled by a single operator. The operator is then tasked with storing each user's balance, updating balances when users want to send funds, and finally sending ETH to users who wish to withdraw--pretty close to how centralized exchanges operate today.

Let's take a look at a payoff matrix in this case. There will be two agents in our game: 1\) the payment operator, and 2\) a single user who wants to send payments. The payouts are defined in the 2nd column and should be read as: { payout\_for\_player1, payout\_for\_player2 }. In this case player1 is the unconstrained operator, and player2 is any arbitrary user. The matrix might look something like this:

| ↓ _Unconstrained Operator _↓_ _→_ User _→ | **Passively Observe** |
| :--- | :--- |
| **Create valid block** | { tx\_fees, utility\_of\_fast\_transactions } |
| **Steal this user's money** | { user\_total\_eth − social\_cost, −user\_total\_eth } |
| **Steal another user's money** | { other\_user\_total\_eth - social\_cost, 0 } |
| **Steal or lose everyone's money** | { total\_eth − all\_future\_fees − social\_cost, −user\_total\_eth } |

Here I'm calling `social_cost` the cost to the operator's reputation and any legal actions taken against them. If the operator is well known, lives in a jurisdiction with a good legal system, this cost could be extremely large. However, it is clear that you'd never trust an anonymous internet troll to be your operator.

Social cost alone can get us a long way, but there are some serious limitations. The first of which is that the mechanisms which establish a high `social_cost` are slow and inefficient. Building a reputation and proving that you aren't some hacker trying to steal coins is a slow and ill-defined process full of difficult social signaling and establishing relationships. Plus even when you do establish this trust, there is no way to prevent the risk that you might make a mistake and get hacked yourself. There is a limit to how much an unconstrained central payment operator can be trusted. However, by imposing constraints using cryptoeconomics we can do much better.

## Adding Operator Constraints: Plasma Chain Exits

In our previous model, users had no recourse if the payment operator were to go rogue. Plasma exits were invented to solve this problem exactly. A Plasma exit is a mechanism by which users are able to reclaim their funds \(eg. ETH\) on the mainchain even if the central operator tries to steal their money. That means if users pay attention they will always have the upper-hand over a Plasma operator and keep their tokens safe. You can learn exactly how this can be implemented in this [Plasma MVP Overview video](https://www.youtube.com/watch?v=jTc_2tyT_lY).

| ↓_Unconstrained Operator_↓ → _User _→ | **Passively Observe** | Exit the Chain |
| :--- | :--- | :--- |
| **Create valid block** | { tx\_fees, utility\_of\_fast\_transactions } | { tx\_fees−this\_users\_future\_fees, exit\_benefit − exit\_cost } |
| **Attempt to steal user's money** | { user\_total\_eth − social\_cost, −user\_total\_eth } | { −all\_future\_fees − social\_cost, −exit\_cost } |
| **Attempt to steal another user's money** | { other\_user\_total\_eth - social\_cost, −added\_risk } | { −all\_future\_fees − social\_cost, −exit\_cost } |
| **Attempt to steal or lose everyone's money** | { total\_eth − all\_future\_fees − social\_cost, −user\_total\_eth } | { −all\_future\_fees − social\_cost, −exit\_cost } |

Now that users have the ability to exit, the payout matrix is wildly different. If users exit the chain in response to a Plasma operator misbehaving, it is clear that the Plasma operator will suffer huge negative utility. The only conceivable reason for misbehaving now would be a hack \(incentive to hurt the operator\), or maybe the validator wanting to impose the `exit_cost` on its users. This is a huge step forward in the power balance between user and operator.

It is important to note the large gain in "decentralization" which was achieved while not adding any new agents. This hints that decentralization is not the number of agents in a system, but instead the distribution of power within that system. By imposing a constraint on the operator, we were able to reduce how much we rely on the `social_cost` of an attack to secure the network. Instead we can rely on much more straightforward incentives, which eventually means we can trust more people to be operators. This process of establishing trust through designing constraints is, to me, what is truly exciting about blockchains.

## Fixing More Imbalanced Incentives

In the Plasma MVP spec, there are still some wacky incentives which reduce how much trust users can place in the system. The major risk is that an operator, or someone who's hacked the operator, can still force all users to exit the chain. If the Plasma operator tries to steal someone's money by creating an invalid block, everyone's funds are immediately put at risk. This risk is so significant that everyone must immediately exit the chain and get their money back. Unfortunately, that means the operator can grief its users by imposing the `exit_cost` on everyone.  This `exit_cost` largely consists of the gas cost of submitting an exit transaction on the mainchain plus the capital lockup cost of waiting a week for the dispute period to end.

Once again we can address this problem by imposing constraints on the operator. In a recent Plasma architecture, a change is made which gives each coin a unique serial number instead of pooling coins in a single contract. With unique coin IDs, a Plasma operator can only attempt to steal a single coin at a time. This means that the large risk of block invalidity which previously would have resulted in a mass exit, now has little to no effect. Each coin must be stolen individually, and can be challenged individually. This is opposed to the Plasma operator stealing all coins, forcing everyone to exit. So for every dollar the operator spends to steal funds, a user spends an equal number of dollars to counter the attack. In cryptoeconomic terms, this reduces the griefing factor from `1:numberof_users_who_need_to_exit` to `1:1` --a significant improvement.

We can now adjust our model to take this change into consideration.

| ↓_Unconstrained Operator_↓ → _User _→ | **Passively Observe** | Challenge Operator |
| :--- | :--- | :--- |
| **Create valid block** | { tx\_fees, utility\_of\_fast\_transactions } | XXX |
| **Attempt to steal a user's money** | { users\_coin\_eth − social\_cost, −user\_total\_eth } | { −challenge\_bond − social\_cost, +challenge\_reward } |

This is especially exciting. Notice how simple this payout matrix has become. The only way for a user to lose is the `{Attempt to steal a user's money, Passively Observe}` box. However, this outcome is easy to avoid and therefore extremely unlikely. Any user is able to challenge an invalid exit, and each invalid exit when found to be invalid is forced to pay a reward to the challenger. This means that if you try to steal and get challenged you'll be paying a fine which incentivizes the challenger.

With such restrictive constraints, we can now trust almost anyone to become an operator. Operators are rendered quite harmless and users are in control. We started in a world where your money is protected by someone's word and a vague threat of force. Then we moved into a world in which every user is able to retrieve their funds when an operator goes rogue. Then finally, with progressively more restrictive cryptographic and economic constraints, we can guarantee that even a rogue operator is unable to do harm.

## 









