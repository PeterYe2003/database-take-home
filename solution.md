
## Solution 

### Approach & Analysis

This is supported by the charts in query_distribution.png show what seems to be a fairly quickly exponentially decreasing (the pattern is more clear after I increased each test to look at 5000 queries).
function where smaller numbers are queried more often than larger numbers.  The file generate_initial_data.py gives the distribution of queries which is:
exp_values = np.random.exponential(scale=1 / lambda_param, size=num_queries).
In constants.py, we also see that lambda_param is 0.1. Thus, we are sampling
from the distribution $0.1*e^{-0.1x}$. 

The ideal random walk would go from 0,1,2,...,499 because that's the order of most likely to be queried to least likely to be queried.
However, we can't choose our starting node so probably the best we can do is n,0,1,2,...,499 where $n$ is the starting node (though you might run into loops).

The probability of querying a node greater than n is approximately 
$\int_{n}^{500} 0.1e^{-0.1x}dx$. For $n > 100$, this integral is around $10^{-5}$, so we don't actually care too much
being able to reach nodes with large numbers as they're queried so rarely. If we start on a large number, we want to get
to the $0$ node as quickly as possible, so we want a lot of nodes pointing towards $0$ node.


### Optimization Strategy

I’m being assessed on two metrics being the success rate and the median path length, where
$\text{path multiplier} = \log(1+ \frac{\text{random median}}{\text{optimized median}})$
My combined score is just $\text{optimized rate} * (1+\text{path multiplier})$.

Thus, my goal is to optimize for the median path length while maintaining a high overall success rate. This was interesting
because optimizing for median performance is quite different from optimizing for average performance. In order to get a low median path length,
we need to start with the path n, 0, 1, ..., 499 frequently. This requires $n$ to have a high edge weight to $0$, but this will
result in a lot of loops because we could go $n,0,1,...,n,0...$.

First optimization is to make sure we spend as little time as possible traversing large nodes. We do this by having only giving them outgoing
edges to $0$, and no incoming edges. We say large nodes are nodes > 80, as they collectively have less than a 0.03% chance to be queried.
Second optimization is to connect the nodes in a chain $0 \to 1 \to 2 \to ... \to 80$. This way, after we hit node 0, we start on the optimal path.

For nodes > 7, we also give them a low chance to be sent back to 0. This is good, because 7,...,15 is strictly worse than
a walk that goes from 7,0,1,...,6. Also, the best possible median is 8 (will justify in iteration journey), so we want to
maximize P(path length <= 8), and overall success ratio. Looping to $0$ increases the chance of getting a short path, but decreases 
the chance we succeed in finding the node (as we will have more loops in our path, so we might not reach node 80 before 10k steps).


### Implementation Details

The implementation was extremely straightforward. I just did edges from n to n+1 for $ n \leq 80$, some edges to $0$
where the weight is linearly increasing with respect to $n$ for $8 \leq n \leq 80$, and for $n > 80$, I had
only one edge going to $0$.

### Results

I ran my graph on 5,000 queries and I got a success rate of 100%, with a median path length of 8, giving an overall score of 527.
I also claim that it's not possible to get a median of 7 without having a success rate > 80%. Notice that the optimal walk (0,1,...,)
gets to the queried node in <= 7 steps only 53% of the time, as $int_{0}^7 0.1e^{-0.1x} dx = 0.503$. However, because the first node is randomly chosen,
the best path becomes n,0,1,..., and the n start just wastes a step over half the time. Thus, even if we can do the next best path, the entire distribution is basically shifted up
by one step, so we can't get a median of 7 while having a high success rate. Thus, what I coded should be close to optimal
(though the success rate isn't truly 100% as my graph fails. A better graph would have a higher bound than 80).

### Trade-offs & Limitations

[Discuss any trade-offs or limitations of your approach]

### Iteration Journey

[Briefly describe your iteration process - what approaches you tried, what you learned, and how your solution evolved]

---

* Be concise but thorough - aim for 500-1000 words total
* Include specific data and metrics where relevant
* Explain your reasoning, not just what you did