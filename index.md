# Rubiks cube solver 
The challenge itself: 
[link](./rubik.html)

When applying hte processing power of computers to a real-life application, the process requires several conditions be satisfied: the real-life can be translated to some form of data; the data might be manipulated

Satisifying both of these conditions simultainously to some degree of success is the largest obstacle for this problem. A cube has ~43 quintilliion unique permutations - can't use a lookup table. The pieces on a cube have both a permutation and an orientation - can't use a simple number based system or describe it with some hashign algorithm. The cube needs to be encoded into a manner that permits quick and efficient maniuplation for additional search hueristics or other puprposes. 

To tackle the first lemma, we must realise that a cube cannot be represented holistally on a computer in a way that one might represent a cube for a grahpical interface - a series of cubies in a 3d space with orientation and position data; we cannot represent it by it's "stickers" or facelets as that's simply too inefficient. Realising this, we might conclude that we can represent a cube by both the **identity** of the pieces and their orientation.

Again, to allow for efficient manipulation, we decide to represent the identity and orientation of some different classes of pieces, such as corners or edges, as required. 

Assumign some such systems exist, we can therefore conclude that an application data-stcuture for representing a cube is a system of coordinates that can each represent a cube. In it's simplist form, a cube can be represented by 4 coordinates corncering the orientaiton and position of both the edges and the corners. We have satisfied the first lemma. 

The second lemma warrents that the data strcuture we have created can be easily manipulated: seperating both the orientation and identiy of the pieces on the cube we can quite easily manipulate them simply by exchanging the identiy of the pieces in specified locations, and "applying" some additional orientation factor to the pieces as they are manipulated. We can store the aforementioned locations and change-in-orientation as pre-defined "moves" that we can apply to the move through a slightly m ore complicated applicaiton of the above method. 

## The two phase algorithm

Each cube can be represented by a series of coordinates that each describe an attribute of that cube. For instance, the orientation of the corners and the edges, or the position of some pieces. 

Certain turns of the cube's faces do not effect some of these attributes. For instance, double turns of any face but the cube's relative top and bottom will not change the orientation of the corners or edges.

With this information, it's possible to split the cube's various states into different groups or *co-sets* that are achievable whilst a constraint has been placed on the moves you can apply to the cube.

# Phase One
The first group, you only consider the orientation of the corners and edges, as well as the position of the edges in
the *UD-slice* - the positions sandwiched between the U and D faces. You do not consider how these edges are orientated
or positioned, just which positions are occupied by these middle edges.

This effectively limits both the quantity of data you have to keep tract of when manipulating the cube (for instance,
the position of edges and corners are only later considered), and the distance you have to search through to reach the "
solved" state of this group, or the identity element - a cube where the edges and corners are orientated correctly, and
where the UD-slice edges - the edges that originate from the UD-slice of a solved cube - are in their home slice.

The aforementioned coordinates are represented in various ways that are each optimised for their uses:

### The corner orientation coordinate

A corner can be orientated one of three ways which; a ternary number system is perhaps the most fitting method of
representing each corner's orientation.

- **0** implies that the corner is not rotated from its default state (as defined in some basic setup)
- **1** that the corner has been twisted once clockwise
- **2** that the conner has been twisted twice clockwise - this is synonymous with an anti-clockwise rotation.

It follows that we can store the orientation of each corner as `ori = ori mod 3` to ensure that the orientation
doesn't "grow" as it changes throughout the solve.

To minimise the size of the ternary number, we can exclude the last corner, now only using 7 bits, as for a cube to be
reachable through valid means the sum of all the orientations of the corners (modulo 3) must satisfy `sum modulo 3 = 0`.
Intuitively this means that there is not a singular corner twisted individually, as a turn of the face will always
impact 4 corners.

The following function reduces the orientation of the corners to a ternary number within the
range `3 pow 7 - 1 = 2186 <---> 0`:

`[0, 2, 2, 0, 2, 2, 1, 0] --> 673`

```python
def corner_orientation_coordinates(self):
    co = reduce(lambda variable_base, total: 3 * variable_base + total, self.co[:7])

    return co
```

The following does the opposite, taking the index of the element in the corner orientation co-set of the cube group and
computing the orientation of the corners:

```python
def Ocorner_coords(self, index):
    parity = 0
    self.co = [-1] * 8
    for _ in range(6, -1, -1):
        parity += index % 3
        self.co[_] = index % 3
        index //= 3

    parity %= 3
    parity = self.__Ocorner_parity_value[parity]
    self.co[-1] = parity
```

### The edge orientation coordinate
The orientation of the edges can be defined in much the same way - a binary system is used where *0* implies an unflipped edge and *1* the contrary.

Again, a similar method can be used to minimise the size of said number: the orientation of the final edge is omitted and calculated such that `sum mod 2 = 0`.
```python
@property
def Oedge_coords(self):
    eo = reduce(lambda variable_base, total: 2 * variable_base + total, self.eo[:11])

    return eo
```
The index of the element has range `2 pow 11 - 1 = 2047 <---> 0` and the following function reverses it in a similar manner to the corners:
```python
@Oedge_coords.setter
def Oedge_coords(self, index):
    parity = 0
    self.eo = [-1] * 12
    for _ in range(10, -1, -1):
        parity += index % 2
        self.eo[_] = index % 2
        index //= 2

    self.eo[-1] = parity % 2
```
