---
layout: page
title: "Rubiks Cube Vlog"
permalink: /rubiks_cube
---

# Rubiks cube solver 
The challenge itself: 

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
