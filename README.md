Download Link: https://assignmentchef.com/product/solved-data130008-01-project-3-blacjack
<br>
The search algorithms explored in the previous assignment work great when you know exactly the results of your actions. Unfortunately, the real world is not so predictable. One of the key aspects of an effective AI is the ability to reason in the face of uncertainty.

Markov decision processes (MDPs) can be used to formalize uncertain situations. In this homework, you will implement algorithms to find the optimal policy in these situations. You will then formalize a modified version of Blackjack as an MDP, and apply your algorithm to find the optimal policy.

In this problem, you will perform the value iteration updates manually on a very basic game just to solidify your intuitions about solving MDPs. The set of possible states in this game is {-2, -1, 0, 1, 2}. You start at state 0, and if you reach either -2 or 2, the game ends. At each state, you can take one of two actions: {-1, +1}.

If you’re in state $s$ and choose -1:

<ul>

 <li>You have an 80% chance of reaching the state $s-1$.</li>

 <li>You have a 20% chance of reaching the state $s+1$.</li>

</ul>

If you’re in state $s$ and choose +1:

<ul>

 <li>You have a 30% chance of reaching the state $s+1$.</li>

 <li>You have a 70% chance of reaching the state $s-1$.</li>

</ul>

If your action results in transitioning to state -2, then you receive a reward of 20. If your action results in transitioning to state 2, then your reward is 100. Otherwise, your reward is -5. Assume the discount factor $gamma$ is 1.




<ol class="problem">

 <li id="1a" class="writeup">Give the value of $V_text{opt}(s)$ for each state $s$ after 0, 1, and 2 iterations of value iteration. Iteration 0 just initializes all the values of $V$ to 0. Terminal states do not have any optimal policies and take on a value of 0.</li>

 <li id="1b" class="writeup">Based on $V_text{opt}(s)$ after the second iteration, what is the corresponding optimal policy $pi_text{opt}$ for all non-terminal states?</li>

</ol>

Equipped with an understanding of a basic algorithm for computing optimal value functions in MDPs, let’s gain intuition about the dynamics of MDPs which either carry some special structure, or are defined with respect to a different MDP.

<ol class="problem">

 <li id="2a" class="code">If we add noise to the transitions of an MDP, does the optimal value always get worse? Specifically, consider an MDP with reward function $text{Reward}(s,a,s’)$, states $text{States}$, and transition function $T(s,a,s’)$. Let’s define a new MDP which is identical to the original, except that on each action, with probability $frac{1}{2}$, we randomly jump to one of the states that we could have reached before with positive probability. Formally, this modified transition function is: $$T'(s,a,s’)= frac{1}{2} T(s,a,s’) + frac{1}{2} cdot frac{1}{|{ s” : T(s, a, s”) &gt; 0}|}.$$Let $V_1$ be the optimal value function for the original MDP, and $V_2$ the optimal value function for the modified MDP. Is it always the case that $V_1(s_text{start})geq V_2(s_text{start})$? If so, prove it in <code>blackjack.pdf</code> and put <code>return None</code> for each of the code blocks. Otherwise, construct a counterexample by filling out <code>CounterexampleMDP</code> in <code>submission.py</code>.</li>

 <li id="2b" class="writeup">Suppose we have an acyclic MDP for which we want to find the optimal value at each node. We could run value iteration, which would require multiple iterations — but it would be nice to be more efficient for MDPs with this acyclic property. Briefly explain an algorithm that will allow us to compute $V_text{opt}$ for each node with only a single pass over all the $(s, a, s’)$ triples.</li>

 <li id="2c" class="writeup">Suppose we have an MDP with states $text{States}$ and a discount factor $gamma &lt; 1$, but we have an MDP solver that can only solve MDPs with discount factor of $1$. How can leverage the MDP solver to solve the original MDP?Let us define a new MDP with states $text{States}’ = text{States} cup { o }$, where $o$ is a new state. Let’s use the same actions ($text{Actions}'(s) = text{Actions}(s)$), but we need to keep the discount $gamma’ = 1$. Your job is to define new transition probabilities $T'(s, a, s’)$ and rewards $text{Reward}'(s, a, s’)$ in terms of the old MDP such that the optimal values $V_text{opt}(s)$ for all $s in text{States}$ are equal under the original MDP and the new MDP.</li>

</ol>

Now that we have gotten a bit of practice with general-purpose MDP algorithms, let’s use them to play (a modified version of) Blackjack. For this problem, you will be creating an MDP to describe states, actions, and rewards in this game.

For our version of Blackjack, the deck can contain an arbitrary collection of cards with different face values. At the start of the game, the deck contains the same number of each cards of each face value; we call this number the ‘multiplicity’. For example, a standard deck of 52 cards would have face values $[1, 2, ldots, 13]$ and multiplicity 4. You could also have a deck with face values $[1,5,20]$; if we used multiplicity 10 in this case, there would be 30 cards in total (10 each of 1s, 5s, and 20s). The deck is shuffled, meaning that each permutation of the cards is equally likely.

<img decoding="async" data-src="blackjack_rule.png" class="float-right lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" class="float-right" src="blackjack_rule.png">

 </noscript>

The game occurs in a sequence of rounds. Each round, the player either (i) takes the next card from the top of the deck (costing nothing), (ii) peeks at the top card (costing <code>peekCost</code>, in which case the next round, that card will be drawn), or (iii) quits the game. (Note: it is not possible to peek twice in a row; if the player peeks twice in a row, then <code>succAndProbReward()</code> should return <code>[]</code>.)

The game continues until one of the following conditions becomes true:

<ul>

 <li>The player quits, in which case her reward is the sum of the face values of the cards in her hand.</li>

 <li>The player takes a card and “goes bust”. This means that the sum of the face values of the cards in her hand is strictly greater than the threshold specified at the start of the game. If this happens, her reward is 0.</li>

 <li>The deck runs out of cards, in which case it is as if she quits, and she gets a reward which is the sum of the cards in her hand. Make sure that if you take the last card and go bust, then the reward becomes 0 not the sum of values of cards.</li>

</ul>




In this problem, your state $s$ will be represented as a 3-element tuple:

<blockquote>

 <code>(totalCardValueInHand, nextCardIndexIfPeeked, deckCardCounts)</code>

</blockquote>

As an example, assume the deck has card values $[1, 2, 3]$ with multiplicity 1, and the threshold is 4. Initially, the player has no cards, so her total is 0; this corresponds to state <code>(0, None, (1, 1, 1))</code>. At this point, she can take, peek, or quit.

<ul>

 <li>If she takes, the three possible successor states (each of which has equal probability of $1/3$) are:

  <blockquote>

   <code>(1, None, (0, 1, 1))</code><code>(2, None, (1, 0, 1))</code><code>(3, None, (1, 1, 0))</code>

  </blockquote>She will receive a reward of 0 for reaching any of these states. (Remember, even though she now has a card in her hand for which she may receive a reward at the end of the game, the reward is not actually granted until the game ends.)</li>

 <li>If she peeks, the three possible successor states are:

  <blockquote>

   <code>(0, 0, (1, 1, 1))</code><code>(0, 1, (1, 1, 1))</code><code>(0, 2, (1, 1, 1))</code>

  </blockquote>She will receive (immediate) reward <code>-peekCost</code> for reaching any of these states. Things to remember about the states after a peek action:

  <ul>

   <li>From <code>(0, 0, (1, 1, 1))</code>, taking a card will lead to the state <code>(1, None, (0, 1, 1))</code> deterministically.</li>

   <li>The second element of the state tuple is not the face value of the card that will be drawn next, but the index into the deck (the third element of the state tuple) of the card that will be drawn next. In other words, the second element will always be between 0 and <code>len(deckCardCounts)-1</code>, inclusive.</li>

  </ul></li>

 <li>If she quits, then the resulting state will be <code>(0, None, None)</code>. (Remember that setting the deck to <code>None</code> signifies the end of the game.)</li>

</ul>

As another example, let’s say the player’s current state is <code>(3, None, (1, 1, 0))</code>, and the threshold remains 4.

<ul>

 <li>If she quits, the successor state will be <code>(3, None, None)</code>.</li>

 <li>If she takes, the successor states are <code>(3 + 1, None, (0, 1, 0))</code> or <code>(3 + 2, None, None)</code>. Note that in the second successor state, the deck is set to <code>None</code> to signify the game ended with a bust. You should also set the deck to <code>None</code> if the deck runs out of cards.</li>

</ul>

<ol class="problem">

 <li id="3a" class="code">Implement the game of Blackjack as an MDP by filling out the <code>succAndProbReward()</code> function of class <code>BlackjackMDP</code>.</li>

 <li id="3b" class="code">Let’s say you’re running a casino, and you’re trying to design a deck to make people peek a lot. Assuming a fixed threshold of 20, and a peek cost of 1, design a deck where for at least 10% of states, the optimal policy is to peek. Fill out the function <code>peekingMDP()</code> to return an instance of <code>BlackjackMDP</code> where the optimal action is to peek in at least 10% of states. Hint: Before randomly assinging values, think of the case when you really want to peek instead of blindly taking a card.</li>

</ol>

So far, we’ve seen how MDP algorithms can take an MDP which describes the full dynamics of the game and return an optimal policy. But suppose you go into a casino, and no one tells you the rewards or the transitions. We will see how reinforcement learning can allow you to play the game and learn its rules &amp; strategy at the same time!

<ol class="problem">

 <li id="4a" class="code">You will first implement a generic Q-learning algorithm <code>QLearningAlgorithm</code>, which is an instance of an <code>RLAlgorithm</code>. As discussed in class, reinforcement learning algorithms are capable of executing a policy while simultaneously improving that policy. Look in <code>simulate()</code>, in <code>util.py</code> to see how the <code>RLAlgorithm</code> will be used. In short, your <code>QLearningAlgorithm</code> will be run in a simulation of the MDP, and will alternately be asked for an action to perform in a given state (<code>QLearningAlgorithm.getAction</code>), and then be informed of the result of that action (<code>QLearningAlgorithm.incorporateFeedback</code>), so that it may learn better actions to perform in the future. We are using Q-learning with function approximation, which means $hat Q_text{opt}(s, a) = mathbb w cdot phi(s, a)$, where in code, $mathbb w$ is <code>self.weights</code>, $phi$ is the <code>featureExtractor</code> function, and $hat Q_text{opt}$ is <code>self.getQ</code>.We have implemented <code>QLearningAlgorithm.getAction</code> as a simple $epsilon$-greedy policy. Your job is to implement <code>QLearningAlgorithm.incorporateFeedback()</code>, which should take an $(s, a, r, s’)$ tuple and update <code>self.weights</code> according to the standard Q-learning update.</li>

 <li id="4b" class="writeup">Now let’s apply Q-learning to an MDP and see how well it performs in comparison with value iteration. First, call <code>simulate</code> using your Q-learning code and the <code>identityFeatureExtractor()</code> on the MDP <code>smallMDP</code> (defined for you in <code>submission.py</code>), with 30000 trials. How does the Q-learning policy compare with a policy learned by value iteration (i.e., for how many states do they produce a different action)? (Don’t forget to set the explorationProb of your Q-learning algorithm to 0 after learning the policy.) Now run <code>simulate()</code> on <code>largeMDP</code>, again with 30000 trials. How does the policy learned in this case compare to the policy learned by value iteration? What went wrong?</li>

 <li id="4c" class="code">To address the problems explored in the previous exercise, let’s incorporate some domain knowledge to improve generalization. This way, the algorithm can use what it has learned about some states to improve its prediction performance on other states. Implement <code>blackjackFeatureExtractor</code> as described in the code comments. Using this feature extractor, you should be able to get pretty close to the optimum on the <code>largeMDP</code>.</li>

 <li id="4d" class="writeup">Sometimes, we might reasonably wonder how an optimal policy learned for one MDP might perform if applied to another MDP with similar structure but slightly different characteristics. For example, imagine that you created an MDP to choose an optimal strategy for playing “traditional” blackjack, with a standard card deck and a threshold of 21. You’re living it up in Vegas every weekend, but the casinos get wise to your approach and decide to make a change to the game to disrupt your strategy: going forward, the threshold for the blackjack tables is 17 instead of 21. If you continued playing the modified game with your original policy, how well would you do? (This is just a hypothetical example; we won’t look specifically at the blackjack game in this problem.)To explore this scenario, let’s take a brief look at how a policy learned using value iteration responds to a change in the rules of the MDP.

  <ul>

   <li>First, run value iteration on the <code>originalMDP</code> (defined for you in <code>submission.py</code>) to compute an optimal policy for that MDP.</li>

   <li>Next, simulate your policy on <code>newThresholdMDP</code> (also defined for you in <code>submission.py</code>) by calling <code>simulate</code> with an instance of <code>FixedRLAlgorithm</code> that has been instantiated using the policy you computed with value iteration. What is the expected reward from this simulation? Hint: read the documentation (comments) for the <code>simulate</code> function in util.py, and look specifically at the format of the function’s return value.</li>

   <li>Now try simulating Q-learning directly on <code>newThresholdMDP</code> with <code>blackjackFeatureExtractor</code> and the default exploration probability. What is your expected reward under the new Q-learning policy? Provide some explanation for how the rewards compare, and why they are different.</li>

  </ul></li>

</ol>

5/5 - (1 vote)

Problem 1: Value Iteration

Problem 2: Transforming MDPs

Problem 3: Peeking Blackjack