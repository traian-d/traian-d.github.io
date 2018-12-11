---
layout: post
title:  "A toolbox for cellular automata"
date:   2018-12-11 23:13:41 +0200
---

I will come out and say it: I like engineering software. I still have a lot to learn, but I enjoy thinking about software architecture, about abstracting things out, generalizing algorithms, code reuse, testability etc. In my daily work this often puts me at odds with those who tend to think that everything beyond procedural-style code with endlessly nested if/else statements is over-engineering.

However, nobody can really prevent me from engineering things in my spare time :D
In particular, for a long time I've been interested in the potential for building a concise and comprehensive framework to express a wide range of [cellular automata](https://en.wikipedia.org/wiki/Cellular_automaton). 

I think the key insight to get things right is to realize that the whole cellular automaton can be split in three components: **the topology** of the environment, **the rules** by which cells interact and **the game state**. The topology would essentially describe the tessellation and shape of the game surface, e.g. a hexagonal grid on a flat surface, a square grid on a torus etc. The rules could be for example the rules of [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life); finally, the game state is simply the state (usually alive or dead) of all the game cells at a given time.

In turn, each component maps quite nicely to some simple structures:
 * The topology can be treated as a graph and stored for example as adjacency matrix (or, in my case, as an adjacency dictionary):
```python
class Topology:

    def __init__(self, n_rows, n_cols, topology_function):
        self.n_rows = n_rows
        self.n_cols = n_cols
        self.topology_function = topology_function

        self.has_neighbor_order, self.adj_dict = topology_function(n_rows, n_cols)
    .......
```
This adjacency dict can in turn be constructed by a function that gets passed to the topology:
```python
def rectangular_four_neighbors(n_rows, n_cols):
    adj_dict = {}

    for i in range(n_rows):
        for j in range(n_cols):
            loc = i * n_cols + j
            if loc not in adj_dict:
                adj_dict[loc] = set()

            if i > 0:
                adj_dict[loc].add((i - 1) * n_cols + j)
            if i < n_rows - 1:
                adj_dict[loc].add((i + 1) * n_cols + j)

            if j > 0:
                adj_dict[loc].add(i * n_cols + j - 1)
            if j < n_cols - 1:
                adj_dict[loc].add(i * n_cols + j + 1)

    return False, adj_dict
```
Finally, topologies can be made to be additive, so that you can combine different neighboring functions. For example, you can add a topology where each cell has for neighbors in positions N, S, E, W, to one with four diagonal neighbors (SE, SW, NE, NW). This yields the familiar square grid with eight neighbors:
```python
def eight_neighbor_grid(n_rows, n_cols):
    return Topology(n_rows, n_cols, rectangular_four_neighbors)\
           + Topology(n_rows, n_cols, rectangular_four_diagonal_neighbors)
```


 * The rules can be encapsulated in a function. In Python they can also be passed around easily since there functions are first-class objects:
 ```python
def conway_game_of_life(adj_dict, cell_values):
    new_cell_values = []
    for i in range(len(cell_values)):
        new_val = cell_values[i]
        live_neighbors = sum([cell_values[neighbor] for neighbor in adj_dict[i]])
        if live_neighbors < 2:
            new_val = 0
        elif live_neighbors > 3:
            new_val = 0
        elif live_neighbors == 3:
            new_val = 1
        new_cell_values.append(new_val)
    return new_cell_values
 ``` 

  * Finally, the game state can be stored as a numeric array, where each element stores a number corresponding to the state of a cell from the game grid:
```python
class Game:

    def __init__(self, topology):
        self.states = [[0] * (topology.n_rows * topology.n_cols)]

        self.topology = topology
    .......

    def apply_rule(self, rule):
        self.states.append(rule(self.topology.adj_dict, self.states[-1]))

    .......
```

The only slight catch is that you have to make sure that these components are compatible with each other, e.g. if a rule requires a cell to have six neighbors, you won't be able to run it on a grid with just four neighbors.

Once everything is in place, you can build some pretty nice animations of the running game using matplotlib.

To give you an example of a classic scenario, here is a Conway glider on a flat rectangular grid:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/compact_cell_automata/glider_simple.gif">
</div>

And this is the same glider, but on a toroidal grid:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/compact_cell_automata/glider_torus.gif">
</div>

For more spectacular visuals you can run a [methuselah](https://en.wikipedia.org/wiki/Methuselah_(cellular_automaton)) starting pattern, such as the Acorn:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/compact_cell_automata/acorn_game.gif">
</div>

With a bit of tweaking game states can be displayed additively. This is particularly useful for one-line topologies such as that used in [Rule 30](https://en.wikipedia.org/wiki/Rule_30):

<div style="text-align:center;margin:20px 0px">
<img src="/assets/compact_cell_automata/rule30.gif">
</div>

Of course you can also define your own game rules and topologies. Here is, for example, my own take on the [Sierpinski triangle](https://en.wikipedia.org/wiki/Sierpinski_triangle):

<div style="text-align:center;margin:20px 0px">
<img src="/assets/compact_cell_automata/sierpinski_triangle.gif">
</div>

My code is available on [github](https://github.com/traian-d/compact_cellular_automata). I still have to add some documentation and perhaps tests and a command-line interface. In the meantime however, please explore and enjoy the wonderful word of cellular automata!
