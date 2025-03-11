## Solution 

### Approach & Analysis

When looking at query_distribution, I noticed that all of the frequent numbers being queried were oddly small. Furthermore, I noticed that all numbers in the queries were below 50, so my approach redirects every node above 50 straight to node 0 to abuse this fact.

My first strategy was to create a cycle that essentially just had the random walk go through all the numbers, but upon further inspection, I noticed that since the extremely small numbers (0-14) appear more frequently than the semi-small numbers (15-49), two separate cycles might be more effective.

Nevertheless, I still want to maintain an 100% success rate, so I want these two cycles to interact with some probability, therefore inspiring me to add two more edges. This will be covered more in detail soon, but these are essentially "redirection" nodes that have a chance to send a random walk to the other cycle. Overall, I believe that since the queries are heavily skewed towards smaller numbers, my algorithm uses that as the crux for all of its heuristics.

### Optimization Strategy

First, we draw edges from nodes 50-499 to node 0, as the nodes with label at least 50 are almost never queried with the current query distribution, so I want to instantly remove them to guarantee fast convergence of the random walk. I chose to send them always to node 0 because node 0 was the most common query, and also the "beginning" of the small number cycle mentioned below.

Now, we create two cycles based on threshold values, with the specific values being 15 and 50 respectively. This means that one cycle consists of the nodes 0-14, and one cycle consists of the nodes 15-49. To create these cycles, we draw edges from node i to node i + 1 in each cycle (making careful to draw an edge from 14 -> 0 and 49 -> 15).

Lastly, we draw two extra edges to account for an edge case, which is the uncommon chance that we start in a cycle but our target is in the other cycle. One of the edges is from 14 -> 15, and the other edge is from 49 -> 0, which are essentially the ends of both cycles. The reason why I chose to link 14 -> 15 and 49 -> 0 was because 14 and 49 are the least likely to be queried in their specific cycles, and 0 and 15 are the most likely to be queried.

Note that we choose to connect node i to node i + 1 and not the other way around. This is intentional because the nodes are less likely to be queried as their index increases, so I start with the smallest index and increment upwards. Overall, the optimization strategy looks like two cycles with two edges connecting them, and many edges (the nodes from 50-499) going to node 0.

Furthermore, every edge in the cycles and the edges from nodes 50-499 to node 0 have weight 1. This is because they are either the only edge (so weight doesn't matter), or there is only one more edge from that node (the "redirection" edges) which have weights that can be toggled instead of those weights. I specifically chose to redirect from small -> medium with weight 1 and medium -> small with weight 4 because the small cycle has numbers that appear more, and therefore should have higher priority in terms of weights.

### Implementation Details

My implementation is not too difficult to follow, it essentially sets the two threshold values and then generates the cycles using a for loop that carefully considers the edge case with the modulo operator.

Note that the extra two edges are manually added afterward, so the overall implementation is quite clean and nice.

### Results

My solution has a success rate of 100%, a median path length of 8.0, and a combined score of 540.06.

### Trade-offs & Limitations

One big limitation of my optimal graph is that it does not exploit the fact that nodes can have a maximum of three edges, as I considered creating three cycles - small, medium, and large - to accurately capture the exponential distribution better, and utilize the extra edges to connect the cycles from all three directions. Additionally, I don't use many of the allowed edges (almost half), as I thought that having more edges would just confuse the random walk and lead to a worse score, but I think it could be possible that adding some more edges could be helpful - potentially between nodes in the same cycle.

Another potential tradeoff is the success rate vs. average path length consideration, as I wanted my optimal graph to maintain its perfect success rate, which may have forced my average path length to stay above a certain bound. If given more time, I could potentially try to sacrifice more success rate to ensure that the paths that work are even shorter. Additionally, if I knew the specific score function, I could potentially figure the optimal balance between success rate and average path length. Overall, I am pretty satisfied with my performance.

### Iteration Journey

Looking at the detailed results from the initial_results file, I realized that many nodes were getting stuck in loops, which decreased the success rate and increased the average path length. Thus, my first idea was to ensure that most nodes only have one edge, which guarantees which node they travel to next, therefore ensuring that although I decrease variability, the algorithm will be more consistent overall.

This led me to my first optimized graph, which was essentially just a large cycle, where each node i was connected to just node i+1 with weight 1. When evaluating this algorithm, I noticed that it had a success rate of 100% and an average path length of 252.0, which was quite the improvement from the initial graph, but I realized that I could potentially sacrifice marginal success rate for a much lower path length.

Now, I decided to analyze the query patterns and determine which queries were more common, so that I could determine methods to essentially "guide" my algorithm towards those more frequent nodes. From the query_distribution file, I noticed that the nodes with smaller indices were queried much more often than the nodes with larger indices, and from the queries.json file, I noticed that there was no target node with index greater than 50. This led me to my second optimized graph, which used the cycle idea from before, but just cycled through the nodes 0-49, and had every other node 50-499 have one edge straight to node 0. This made the algorithm much better, maintaining the success rate of 100% while decreasing the average path length down to 8.0, with a combined score of 519.86, which is much better than before.

Lastly, with my remaining time, I sought to find methods to make the cycle idea better, keeping the fact that for all nodes with label 50-499, they should have one edge straight to node 0 because they almost never appear in the queries. My idea was to somehow construct multiple cycles, and have one node in each cycle that could "redirect" the random walk to other cycles, weighted based on the frequencies of the queries. I had ideas for implementing three cycles - small, medium, and large - but I ran out of time and I think that the improvement would be marginal, ending with a combined score of 540.06 - even though the average path length is still 8.0.

I considered many things, such as changing the amount of nodes in the small vs. larger cycle, changing the weights on the "redirection" nodes themselves, and even considering if one of the cycles should be reversed (i + 1 -> i). Nevertheless, I kept testing them and observing the optimized_results file, deducing how feasible my ideas were based on the average path length (since all of these optimal graphs had 100% success). I would consider the bottleneck targets - which was essentially the "worst-cases" in my graph, and I realized that the overall goal was to not only reach small numbers quickly, but to also allow for enough variation that you can jump between cycles efficiently. Additionally, each time my idea performed badly (with a lower score than previous optimization graphs), I always tried to figure out why and see if there was a potential speedup, even from a graph that wasn't feasible. Nevertheless, implementing something that used this insight was quite challenging, but I believe my final solution essentially represents my entire iteration process, and it uses ideas from every step of the way - specifically by trying new things and merging the ideas that worked together to create a final product. Overall, this project was very interesting and I enjoyed creating an optimal graph for a problem that can be continuously optimized.