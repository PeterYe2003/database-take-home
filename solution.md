
## Solution 

### Approach & Analysis

This is supported by the charts in query_distribution.png show what seems to be a fairly quickly exponentially decreasing (the pattern is more clear after I increased each test to look at 5000 queries).
function where smaller numbers are queried more often than larger numbers.  The file generate_initial_data.py gives the distribution of queries which is:
exp_values = np.random.exponential(scale=1 / lambda_param, size=num_queries).
In constants.py, we also see that lambda_param is 0.1. Thus, we are sampling
from the distribution $0.1*e^{-0.1x}$. 

The ideal random walk would go from $0,1,2,...,499$ because that's the order of most likely to be queried to least likely to be queried.
However, we can't choose our starting node so probably the best we can do is $n, 0,1,2,...,499$ where $n$ is the starting node (though you might run into loops).

The probability of querying a node greater than n is approximately 
$\int_{n}^{500} 0.1e^{-0.1x}dx$. For $n > 100$, this integral is around $10^{-5}$, so we don't actually care too much
being able to reach nodes with large numbers as they're queried so rarely. If we start on a large number, we want to get
to the $0$ node as quickly as possible, so we want a lot of nodes pointing towards $0$ node.


### Optimization Strategy

I’m being assessed on two metrics being the success rate and the median path length, where
$$\text{path multiplier} = \log(1+ \frac{\text{random median}}{\text{optimized median}})$$
My combined score is just $optimized rate * (1+path_multiplier)$.

Thus, my goal is to optimize for the median path length while maintaining a high overall success rate. 



### Implementation Details

[Describe the key aspects of your implementation]

### Results

[Share the performance metrics of your solution]

### Trade-offs & Limitations

[Discuss any trade-offs or limitations of your approach]

### Iteration Journey

[Briefly describe your iteration process - what approaches you tried, what you learned, and how your solution evolved]

---

* Be concise but thorough - aim for 500-1000 words total
* Include specific data and metrics where relevant
* Explain your reasoning, not just what you did