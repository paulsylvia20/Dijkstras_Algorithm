# An Implementation of Dijkstra's Algorithm to a League of Legends Champion
### TL;DR
The present repository has painstakingly developed all of the shortest solutions to the Aphelios weapon swapping problem. These may be found along with a quick look-up tool in the excel file, [Aphelilos Methods.xlsx](https://github.com/paulsylvia20/Djikstras_Algorithm/blob/1a8f58013b5c6eedfc4ac46de77b1a841bf63032/Aphelios_Methods.xlsx). For example, if a player was interested in going from the initial weapon state (weapon queue = PBW) to the weapon cycle GPBRW, the lookup will return '(1) GP' meaning the player should deplete Green, then Purple. If you are interested in how I arrived at these solutions, please read-on and check out the Python implementation of Dijstra's Algorithm, ([Dijkstras_Algorithm.ipynb]()).

---
This project was born out of curiosity. As a casual League of Legends player, I became interested in a problem posed by game design. The problem goes like this: a champion, Aphelios, uses a rotating weapons system. Whenever he runs out of "ammo" with one weapon, it is replaced by the next weapon in a queue. There are N = 5 weapons total, two equipped in his mainhand/offhand and 3 that are queued (see the image below where each of his weapons are represented as a symbol.). At any moment, he has a choice of which weapon he should "deplete" and place at the end of the queue, then equip the weapon at the front of the queue in its place. Consequently, Aphelios can change the order in which his weapons arrive with certain orders being better than others. This repository seeks to find the shortest possible paths by applying a very general algorithm (Djikstra's algorithm). In the end we will identify every shortest path from every possible weapon state to every possible weapon cycle.

<div align="center">
  <img src="https://github.com/user-attachments/assets/4bb759fe-96fd-4a99-b330-263a28f9ea47" width="600">
</div>
<p align="center">Figure 1: a representation of Aphelios's 5 weapons. Left hand side (2 larger weapons) are his active weapons. 
  Right hand side (3 smaller weapons) are his weapon queue/inactive weapons. This particular weapon state could be either GRPBW or RGPBW. </p>
  
## Defining the Network Problem
The problem we seek to solve is best identified as a network problem wherein states of Aphelios's weapon system are nodes and choices about weapon depletion are edges. The first step will be to build the network by defining all of the nodes and edges. Here we have identified 24 *cycle nodes*, 60 *state nodes* and Y edges. To define nodes, we are faced with a a handful of problems the first of which is the distinction between nodes representing a particular *weapon state* and nodes that represent a *weapon cycle* which exist independently of which weapons are equipped or queued. I have flushed out these definitional and naming problems with Excel, but I will describe the logic I used in detail below.

#### Cycle Nodes
Aphelios has 5 weapons, each with its own color, Green (G), Red (R), Blue (B), Purple (P), White (W). One cycle node consists of the 5 weapons in a particular order (i.e. GRBPW) without replacement meaning there would be 5!= *120* possible of these orderings. However, a wrinkle which restrain this value is that these are, indeed, *cycles* (think of a circle) so we can ignore orderings that are phase shifts of the others. This means that GRBPW is equivalent to RBPWG, BPWGR, PWGRB, and WGRBP (notice that in each case, weapons were just shifted once to the left). For every ordering, there are 5 equivalent expressions meaning we are left with 5!/5 = 4! = 24 unique cycle nodes. In naming these, I defaulted to the convention of placing G first in the cycle which avoids any equivalency in the final node set.

#### State Nodes
Here we have to distinguish between weapon *cycles* and *states*. In the latter, active weapons matter and weapon queu matters. There is no circular structure and an interesting wrinkle is that the first two weapons can be flipped and still be equivalent. For brevity I will simply assert that the weapon queue contains all of the necessary information and we can simply ignore everything else. Since we have 5 weapons and 3 queue positions (order matters and there is no circular structure we need to consider), we are left with 5!/2! = 60 possible weapon states.

#### Base Network
While we are interested in weapon cycles as endpoints, they are mostly an abstract description. The weapon state is more literal. When path finding, we will primarily traverse the base network of weapon states, while checking associated weapon cycles at each stop along the way. In the base network, we find that each node has 2 outgoing edges (directed) and 2 incoming edges (also directed). This is because each weapon state contains a choice: Aphelios may deplete one of his 2 equipped weapons. Moreover, the network has a perfect, multidimensional symmetry that is challenging to depict visually, but guarentees each weapon state can be arrived at by exactly two other weapon states. The result is a network with 60*2 = 120 edges. Mathematically, a network like ours, having no starting or stopping nodes which can be traversed for an arbitrary number of steps, is known as *cyclical* and *directed*.

#### Associational Network
Next, we associate the 24 cycle nodes with the 60 state nodes. We should also begin discussing the notion of edge length. So far, edges have been treated as if they are 1 "step", however, Djikstra's algorithm works by finding the shortest distance between nodes, assuming edges have a measurable length (think gps navigation, finding the shortest route). In our case, the choice of length is arbitrary and we will select 1 'unit' as a distance between state nodes. Now we associate the state nodes with the cycle nodes. Think of weapon cycles as descriptors to the state nodes. When a weapon state is entered so to do we enter into its associated weapon states. So these associational edges have a length of 0. We identfiy 120 of these associational edges, 2 per weapon state (60 weapon states * 2 weapon cycles = 120). Alternatively, you can think of it as 5 possible weapon states per weapon cycle (24 weapon cycles * 5 weapon states = 120).

### The Whole Network
In sum, we have described a network composed of two parts, a base network (green) and an associational network (purple). The base network is both *directed* and *cyclical* with 60 state nodes, interconnected in duplicate via 120 directed edges of length 1. On top of this, there is an associational network that tags each weapon state node with its attendent weapon cycles. Each weapon state has 2 associated weapon cycles. In total, it consists of 120 additional *direct* edges of length 0 going from state nodes (inner circle) to cycle nodes (outer circle) where each weapon cycle recieves connection from exactly 5 state nodes.

<div align="center">
  <img src="https://github.com/user-attachments/assets/64c858cf-9b82-4928-9877-6b5d5fa80b3b" width="600">
</div>
<p align="center">Figure 2: the full network for Aphelios's weapon states and weapon cycles. The objective is to traverse from any weapon state (green node) to any weapon cycle (purple node) in the shortest number of steps where the green links are length 1 and the purple links are length 0. </p>

## The Search (Dijkstra's Algorithm)
