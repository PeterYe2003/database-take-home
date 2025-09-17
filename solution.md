
## Solution 

### Approach & Analysis

The charts in query_distribution.png show the node queried frequency decreases at an exponential rate with respect to the
number on the node. The file generate_initial_data.py gives the distribution of queries which is:
exp_values = np.random.exponential(scale=1 / lambda_param, size=num_queries).
In constants.py, we also see that lambda_param is 0.1. Thus, we are sampling
from the distribution $0.1*e^{-0.1x}$. 

The ideal random walk would go from 0,1,2,...,499 because that's the order of most likely to be queried to least likely to be queried.
However, we can't choose our starting node so probably the best we can do is n,0,1,2,...,499 where $n$ is the starting node (though you might run into loops).

The probability of querying a node greater than n is approximately 
$\int_{n}^{500} 0.1e^{-0.1x}dx$. For $n > 100$, this integral is around $10^{-5}$, so we don't actually care too much
being able to reach nodes with large numbers as they're queried so rarely. If we start on a large number, we want to get
to the 0 node as quickly as possible, so we want a lot of nodes pointing towards 0 node.


### Optimization Strategy

I’m being assessed on two metrics being the success rate and the median path length, where
$\text{path multiplier} = \log(1+ \frac{\text{random median}}{\text{optimized median}})$
My combined score is just $\text{optimized rate} * (1+\text{path multiplier})$.

Thus, my goal is to optimize for the median path length while maintaining a high overall success rate. This was interesting
because optimizing for median performance is quite different from optimizing for average performance. In order to get a low median path length,
we need to start with the path n, 0, 1, ..., 7 frequently. This requires all $n$ to have a high edge weight to 0, but a tradeoff is that 
the random walk will have more loops because we could go n,0,1,...,n,0....

First optimization is to make sure we spend as little time as possible traversing large nodes. We do this by having only giving them outgoing
edges to 0, and no incoming edges. We say large nodes are nodes > 90, as they collectively have less than a 0.03% chance to be queried.
Second optimization is to connect the nodes in a chain $0 \to 1 \to 2 \to ... \to 90$. This way, after we hit node 0, we start on the optimal path.

For nodes > 7, we also give them a low chance to be sent back to 0. This is good, because 7,...,15 is strictly worse than
a walk that goes from 7,0,1,...,6. Also, the best possible median is 8 (will justify in results), so for any node > 7, looping back to 0
increases the chance of getting a short path, but also slightly decreases the chance we succeed in finding larger nodes greater than it
(as we will have more loops in our path, so we might not reach larger nodes before 10k steps).


### Implementation Details

The implementation was straightforward. I created edges from n to n+1 for $n \leq 90$, edges to 0
where the weight was linearly increasing with respect to $n$ for $8 \leq n \leq 90$. For $n > 90$, I had
only one edge going to the 0 node.

### Results

I ran my graph on 10,000 queries and I got a success rate of 100%, with a median path length of 8, giving an overall score of 530.
I also claim that it's not possible to get a median of 7 without having a success rate > 90%. Notice that the optimal walk (0,1,...,)
gets to the queried node in <= 7 steps only 53% of the time, as $\int_{0}^7 0.1e^{-0.1x} dx = 0.503$. However, because the first node is randomly chosen,
the best path becomes n,0,1,..., and the n start just wastes a step like, over 90% of the time (because P(n = query node) < 0.1). Thus, even if we can do the optimal path given a random starting node,
the entire distribution of random walk lengths would be increase by close to one step, so we can't get a median of 7 while having a high success rate. Thus, what I coded should be close to optimal
(though the success rate isn't truly 100% as my graph fails on large node queries which are rare. A better graph would have a higher bound than 90).

### Trade-offs & Limitations

The main trade-off was between success rate and median path length. I realized that the optimal median path length was 8,
so you want to try to do $n,0,1,2,3,4,5,6$ as much as possible (if $n \geq 7$), and otherwise some permutation of 
$0,1,2,3,4,5,6,7$. This would look like sending any node $n \geq 7$ to 0 with high probability. However, the trade-off was that
we would run into more loops, which would make it hard to find a node like 40 in 10k steps because we waste too many steps looping
back to 0. Thus, there's a balance for how often we can go back to 0.

On first-order, we should go back to 0 more frequently when we are on a larger node as the chance that the query node has
an even larger number is just smaller. Thus, we don't harm our success rate as much, and the increase in the chance we find a path 
with length less than 9 is larger (going to node 0 is better when your alternative is 101 vs 12). Thus, I went for a graph where the weight
that each node had of going back to 0 was larger as the node value increased (starting from 8).

The cap at 90 is also a limitation as my graph fails whenever a node > 90 is queried. The parameters for my weights can be further optimized
through some grid search, so maybe a better graph would have an upper bound of 150 or use a smarter increase in weights on the edges to 0 (instead of piecewise linear).

### Iteration Journey

My first iteration was just a simple cycle from 0 to 499. This ensured a 100% success rate and a median path length of 250
, but it did not take advantage of the fact that larger numbers are queried exponentially less frequently than smaller numbers.

My next method was to add more backward edges to 0, especially for larger node values. This improved my average path length to 15, but 
the success rate was low because I was getting caught in a lot of cycles.

I realized that the odds of a large node being queried was really low, so I just ensured that nodes greater than 100
always went to $0$. This was the biggest improvement in my method, as I had 100% accuracy and my
median path length decreased to 9. The only way to improve my graph was to decrease my median path length to 8.
For this to be a worthwhile effort, $\text{optimized rate} * (1+\log(1+\frac{560}{8})) > 100 * (1+\log(1+\frac{560}{9}))$, as the
naive random path length was around 560. Thus, the success rate must be greater than 98.22% (otherwise we’re better off 
just getting 100% success rate with a median path length of 9.0). I was able to achieve a good accuracy by just playing around with the bound I used
for when I only gave the nodes an edge to 0, and the weights on the backward edges on nodes 8+ to 0.

I also thought about some cluster structures, but they were all worse.

Overall this was a very fun problem!