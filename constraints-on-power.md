# Constraints on Power Workspace

A blog post series which will explore the Ethereum technology through the  
lense of constraints on power. This will likely include posts on:

* Plasma MVP constraint analysis
* Centralized blockchain risk
* AGI defense system
* Diverse stakeholder call to action

---

# Plasma MVP Constraint Analysis

The essence of decentralization is imposing constraints on central authority. These constraints limit actions of agents in a system. With cryptoeconomics we are able to design decentralized protocols which precisely define these constraints.

Plasma MVP is the first simple Plasma specification. It provides scalable payments using a central operator while maintaining strong security guarantees. In this post we will use payoff matrixes to identify central operators' poorly aligned incentives, address them by imposing constraints, and end up recreating the current Plasma MVP specification.

Let's begin!

## Unconstrained Central Payment Operator

One could naively scale payments with an unconstrained payment operator. In this scheme, users would first send ETH to a smart contract. This smart contract is completely controlled by a single operator. The operator is then tasked with storing each user's balance, updating balances when users want to send funds, and finally sending ETH to users who wish to withdraw--pretty close to how centralized exchanges operate today.

Let's take a look at a payoff matrix in this case. There will be two agents in our game: 1\) the payment operator, and 2\) a single user who want to send payments. The payouts are defined in the 2nd column and should be read as: { payout\_for\_player1, payout\_for\_player2 }. In this case player1 is the unconstrained operator, and player2 is any arbitrary user. The matrix might look something like this:

| ↓ _Unconstrained Operator _↓_ _\|\| →_ User _→ | **Passively Observe** |
| :--- | :--- |
| **Create valid blocks** | { tx\_fees, utility\_of\_fast\_transactions } |
| **Steal this user's money** | { user\_total\_eth − social\_cost, −user\_total\_eth } |
| **Steal another user's money** | { other\_user\_total\_eth - social\_cost, 0 } |
| **Steal or lose everyone's money** | { total\_eth − all\_future\_fees − social\_cost, −user\_total\_eth } |

Here I'm calling "social\_cost" the cost to the operator's reputation and any legal actions taken against them. If the operator is well known, lives in a jurisdiction with a good legal system, this cost will be extremely large. However, it is clear that you'd never trust an anonymous internet troll to be your operator.

This social cost is enough in some cases, but we can do way better by adding some cryptoeconomic constraints. These constraints greatly reduces how much trust you have to have in the social cost to an operator. This is great because with a lower barrier of entry we have more competition and with more competition we reduce monopolistic rent extraction. Plus everyone can sleep a little better knowing their money isn't under someone else's compete control.

## Adding Operator Constraints: Plasma Chain Exits

In our previous model, users had no recourse if the payment operator were to go rogue. Plasma exits were invented to solve this problem exactly. A Plasma exit is a mechanism by which users are able to reclaim their funds \(eg. ETH\) on the mainchain even if the central operator tries to steal their money. That means if users pay attention they will always have the upper-hand over a Plasma operator and keep their tokens safe. You can learn exactly how this can be implemented in this [Plasma MVP Overview video](https://www.youtube.com/watch?v=jTc_2tyT_lY).

| ↓_Unconstrained Operator_↓ \|\| → _User _→ | **Passively Observe** | Exit the Chain |
| :--- | :--- | :--- |
| **Create valid blocks** | { tx\_fees, utility\_of\_fast\_transactions } | { tx\_fees−this\_users\_future\_fees, exit\_benefit − exit\_cost } |
| **Attempt to steal another user's money** | { user\_total\_eth − social\_cost, −user\_total\_eth } | { −all\_future\_fees − social\_cost, −exit\_cost } |
| **Attempt to steal another user's money** | { other\_user\_total\_eth - social\_cost, −added\_risk } | { −all\_future\_fees − social\_cost, −exit\_cost } |
| **Attempt to steal or lose everyone's money** | { total\_eth − all\_future\_fees − social\_cost, −user\_total\_eth } | { −all\_future\_fees − social\_cost, −exit\_cost } |

Now that users have the ability to exit, the payout matrix is wildly different. If users exit the chain in response to a Plasma operator misbehaving, it is clear that the Plasma operator will suffer huge negative utility. The only conceivable reason for misbehaving now would be a hack \(incentive to hurt the operator\), or maybe the validator wanting to impose the `exit_cost` on its users. This is a huge step forward in the power balance between user and operator.

It is important to note the massive gain in "decentralization" which was achieved while not adding any new agents. This hints that decentralization is not the number of agents in a system, but instead the distribution of power within that system. By imposing a constraint on the operator, we were able to reduce how much we rely on the `social_cost` of an attack to secure the network. Instead we can rely on much more straightforward incentives, which eventually means we can trust more people to be operators. Democratizing trust like this is a pretty magical thing.



\[ FIRST list incentives if the plasma operator does not have any transparency\]

\[ NEXT allow users to verify plasma operator transactions \]

\[ FINALLY add exit mechanisms \]

There are a number of issues with this approach and we will use a simple payout matrix to expose one of the most obvious ones: stealing everyone's money.
