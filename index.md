## Fair Partitioning of Convex Polygons
An investigation by Owen Shriver for CSCI 716 (Computational Geometry)

### The Problem

[TOPP #67](https://topp.openproblem.net/p67#Problem.67): Consider a convex polygon P and a positive integer n. Is it always possible to partition P into n convex pieces such that each piece has the same area and the same perimeter?

## Past Results

Nandakumar and Rao proved this for n = 2 and n = 4. Aronov and Hubard proved it for all prime powers. Nandakumar also proposed a proof for a weaker conjecture, where the pieces need not be convex. However, there has been no full proof of the conjecture for all n, though it is believed to probably be true.

## What I Did

I did not have any particular goals for this project; instead, I just wanted to experiment and approach the problem from several different angles in order to understand it better. My work is therefore organized into a series of attempts, some of which may be related, but which do not consititute a complete study of the problem. They were simply my ideas for how to put my own spin on the problem

## Attempt 1: Calculus-Based Approximator

One immediate thing I noticed was that the approaches to this problem that I read had all been from a proof standpoint, trying to prove the conjecture for certain values of n. I wondered if there might be some value in creating an approximator for the problem; that is, an algorithm that starts with some unfair partition and tries to minimize the "unfairness".

To measure unfairness, I chose to compute the variance in area and the variance in perimiter and sum them together, like so:

> Unfairness Score U = Σ(area - mean area)^2 + Σ(perimeter - mean perimeter)^2

For a perfectly fair partition, U = 0; for all other partitions, U > 0.

Then, my plan was to choose one point on the partition and compute the derivative of U with respect to x and y, in order to locate a local minimum of U and move the point to that minimum. Repeated application of this process with different points would hopefully be able to find a state where no further progress could be made, or reach an asymptote. 

The derivative of the area variance was easy to calculate, because area is linear with respect to x and y. But the perimeter formula depends on the distance formula, which involves radicals, and so solving for its derivative ends up looking something like this:

![Image](https://raw.githubusercontent.com/ofs9424/FairPartitioning/gh-pages/formula.png)

So I was unable to accomplish anything with this attempt.

## Attempt 2: Random Approximator

Even though it's not possible to use calculus to find the optimal location to move a point to, it is still possible to find a location that decreases U through trial and error. This ends up looking a lot like a hill climbing algorithm-- except with one important distance. With hill climbing, the number of possible mutations at any particular point are finite, so it is possible to determine with certainty that no improvement is possible and thus end the algorithm. With my iterative random approximator, I need to give it a particular point to end. I chose to measure this by counting iterations, where one iteration consists of:

> * Select one point in the partition
> * Choose a random new location for that point, within the bound created by the three points it is connected to
> * Check if the new point keeps all of the polygons in the partition convex
> * Check if the new point decreases unfairness
> * If both checks pass, update the partition with the new point

Here is an example of the algorithm in progress:
![Image](https://raw.githubusercontent.com/ofs9424/FairPartitioning/main/diagram%20animated.gif)

So this algorithm is successful in doing roughly what I want it to do. But, I'd like to actually get somewhere with this, rather than just producing a toy model that doesn't correspond to the actual problem at all.

An optimization algorithm like this will eventually converge onto a particular state. Since it's not a true hill climbing algorithm, the program cannot detect the convergence directly, but I can look at the trends as the number of iterations increases.

There were two things I wanted to look at. First, the total unfairness (which, remember, is the area variance + perimeter variance). If this value seemed to hit an asymptote, then it likely means that this approximation method can't get arbitrarily close to a true partition. Second, the point of last improvement, in other words the last iteration where a change was made to the partition, normalized by the total iteration count to a 0-1 scale. If this value remains constant, then it would suggest that the algorithm would be able to continue finding improvements indefinitely and not get stuck.

Here were the results, for the same problem shown before (irregular quadrilateral, n = 7):

![Image](https://raw.githubusercontent.com/ofs9424/FairPartitioning/gh-pages/variance.png)
![Image](https://raw.githubusercontent.com/ofs9424/FairPartitioning/gh-pages/poli.png)

These results are both fairly encouraging. The average variance (aka unfairness) seems to continue to decrease as iteration count increases, and the average point of last improvement stays roughly constant.

So this attempt ended up being a moderate success-- I achieved my goal, got some results, and the results do suggest that the problem might be solvable this way, though I don't have the time or the computing power to do it myself.

## Attempt 3: Extending the Backtracking Algorithm

After finishing my approximator, I figured I'd make at least some attempt to attack the problem theoretically. Nandakumar and Rao gave, as a demonstration, an unsuccessful attempt at a backtracking algorithm for the problem:

> * From the polygon, carve out a piece such that: the piece has the proper area, the piece has the same perimeter as all previous pieces, the piece is convex, and the remaining polygon is convex.
> * Repeat until the partition is complete or this is impossible; if it's impossible, backtrack and try a different piece.


They showed that this algorithm fails on cases such as a 120-degree rotationally symmetric circle-like polygon for n = 3, where the partition is clearly three ways down the center like a pie, but the backtracking algorithm can't find that because it leaves the polygon non-convex after the first piece is carved.

I wondered if this method could be fixed by lifting the requirement that the polygon remain convex at all steps, and instead introducing a "concavity index" which would track how many divisions are needed to ensure that the pieces are all convex. A convex polygon has a concavity index of 0, and a concave polygon's index is the fewest number of convex pieces that it can be broken into, minus 1. That way, I could ensure that a division never happened that would bring the concavity index above the number of cuts left to make, which could in theory salvage the algorithm. But when I tried to calculate the concavity index, I wasn't able to-- I believe it is capped at the number of vertices of the piece that was just carved out, but I can't prove that, and even if I could, it wouldn't really help me here.

## Attempt 4: Extending the n = 4 Argument

Nandakumar and Rao also proved n = 2 and n = 4. For n = 2, the proof goes as follows:

> * Let P be a point on the polygon's edge. For each such point, there is a unique Q such that PQ divides the polygon into two pieces of equal area.
> * Move P around the perimeter of the polygon. Eventually, P will equal the original Q, and Q will equal the original P.
> * When this happens, the two halves will have changed places, so if one half had a greater perimeter, now the other half does.
> * Since perimeter is continuous with respect to deformation of the polygon, there must exist a P-Q pair where the perimeters are equal.


They then go on to show n = 4 by using a similar argument, except instead of just comparing the perimeters on each side, they run the n = 2 algorithm on each side and compare the resulting perimeters.

This immediately raises the question-- why can't this apply to all numbers? The problem's been solved for n = 3, but not n = 6, so why not just walk a P and Q around the perimeter and run the n = 3 algorithm on each side, comparing the results?

The key is in the last step. The algorithm run on each side must be continuous with respect to deformation of the polygon. Nandakumar and Rao went to some lengths to show that the n = 2 case was continuous, allowing it to be used in the n = 4 proof. The n = 3 proof is much more complicated, and therefore I won't explain it here, but it is not known to be continuous. But at least I was able to determine that if the conjecture can be proved for n using a method that is continuous with respect to deformation of the polygon, then it is also true for 2n.

## Conclusion

At this point, I felt that I had attacked the problem from enough different angles. Although I did not really get any results (other than the vague suggestion from Attempt #2 that a solution might exist), I certainly understand the problem better, and I hope that everyone reading this can get a better understanding of the problem as well from my work.
