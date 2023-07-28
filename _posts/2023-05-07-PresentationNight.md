---
layout: single
title:  "Random Walks in Products of Free Groups"
header:
  image: "images/UGRPS-Summer-22.jpg"
date: 2023-05-07
mathjax: true
---

On April 20th 2023, I had the opportunity to share with the Department of Mathematics at Purdue University some of the ideas I worked on during the previous Summer 2022 with [Dr. Thomas Sinclair](https://www.math.purdue.edu/~tsincla/) and other peers in the field, [Colton Griffin](https://www.linkedin.com/in/colton-griffin-9b1b3b191/), [Sanchita Chakraborty](https://www.linkedin.com/in/sanchita-chakraborty-8107a9192/), and Yuxiao Wang. I will try to summarize the presentation, as well as our preleminary results from when we worked on the project.

# Purpose
The main goal was to explain the underlying concepts of the study in a brief 10 minute presentation to a wide audience without feeling limited or confused by the complexity of notation and the mathematics used.

# Introduction
## Algebraic Groups
Firstly, we refreshed the concepts of groups in Algebra. Most importantly, we must remember that the main properties of groups are:
1. an operation that allows elements to interact 
2. an identity element exists
3. the existence of inverse elements
For example, the integers modulo n groups has the addition operation, 0 as the identity element and every non-zero element in the set has an element which added to itself returns to zero, e.g., ![mod_example](https://latex.codecogs.com/svg.image?(1&plus;3)\mod&space;4=0.)

I am intentionally utilizing the modulo group as it is also a special case of groups, an abelian group. Abelian groups are special because the results are independent of the order in which the operation is done, i.e., it possesses the commutative property. However, our study focuses in a group that does not have the commutative property, free groups.

## Free Groups

### Group Properties
Free Groups do not promise to be abelian. Meaning, the common case is that it is non-commutative. We can represent the group to be geometric so that the elements represent possible directions of movement. More specifically, we have an identity element, which we denote as `e`, representing the origin which happens by not moving. In this group, the operation is concatenation, this way, we can interpret the string generated from the operation to represent a history of movements starting from the origin. Every direction has an inverse direction, thus satisfying all the requirements. Most importantly, free groups, although non-abelian, are a special group, this is because they possess generator elements, the same way I intentionally snuck the integers modulo n example. These special elements generate all other possible elements in the group. This way we can interpret visually two cases: the commutative and the non-commutative, even though they have the same number of generators, for example, up and right direction elements.

### Examples of Two Generators
1. In the commutative case, consider the image below which is just a simple cartesian grid. If we consider `(2, 2)` to be the origin, right up is equivalent to up right, i.e., `(3, 3)`.

<div align="center">
    <img src="/images/grid.png" alt="Grid Example">
    <figcaption>Grid Figure taken from Jarutatsanangkoon, P., Mohammed, W. S., & Pijitrojana, W. (2018). Transformation optics based on unitary vectors and Fermat’s principle for arbitrary spatial transformation design. Applied Optics, 57(29), 8632-8639.</figcaption>
</div>

2. On the other hand, a non-commutative case would be the Cayley Graph underneath which is more analogous to Free Groups. Although we still have two generators a and b, being right and up respectively, like before but still the `ab` string is not the same location as `ba`.

<div align="center">
    <img src="/images/Cayley tree.png" alt="Cayley Graph">
    <figcaption>Cayley Graph Figure taken from Wikipedia.</figcaption>
</div>

### Leinert Property

Informally, we can express the Leinert property to be a very self-evident fact about the Free Groups. That is, the only way to return back to the origin with any given string is by undoing every step of the original string. In other words, for a given string, the only inverse string is to flip the elements so that the last element is the first and the first is the last, as well as inverting them to their group inverses. Namely, for

![stringx](https://latex.codecogs.com/svg.image?x=ab^{-1}ab^{-1}ab)

the string is first flipped left to right,

![stringflip](https://latex.codecogs.com/svg.image?bab^{-1}ab^{-1}a&space;)

then, flip by inverting the elements, as follows,

![stringy](https://latex.codecogs.com/svg.image?y=b^{-1}a^{-1}ba^{-1}ba^{-1}.)

I will let you try to convince yourself that
![yisinvertx](https://latex.codecogs.com/svg.image?y=x^{-1})
because
![equal](https://latex.codecogs.com/svg.image?&space;xy=yx=e.)
However, we can see that is not true in the commutative case as we can undo any step at any time so there is no unique inverse.


## Definitions & Notation
1. The probability of moving in the direction n is denoted as
![mu](https://latex.codecogs.com/svg.image?\mu(n).)

2. The probability of moving in direction inverse to n so that 
![muhat](https://latex.codecogs.com/svg.image?\hat\mu(n)=\mu(n^{-1}),)
is denoted as
![muhat](https://latex.codecogs.com/svg.image?\hat\mu(n).)

3. The probability of staying in place is denoted as
![sigma](https://latex.codecogs.com/svg.image?\sigma=\mu*\hat\mu.)

4. The probability of returning to the origin after infinitely many steps, named the spectral norm, is given by the following equation,
![spectralnorm](https://latex.codecogs.com/svg.image?\left||\sigma\right||=lim_{n\to\infty}\sqrt[n]{\sigma^{(n)}(e)}.)

# Our study
## Goal
We should mention first that [Woess (1986)][#woess] managed to prove that the spectral norm for the random walks in groups with the Leinert Property to be the inverse of the radius of convergence for the following function:

![G(z)](https://latex.codecogs.com/svg.image?&space;G(z)=\sum_{n=0}^\infty{\mu^{(n)}(e)z^{2n}}.)

Is there a middle ground between the commutative and non-commutative case? Or let me better rephrase the question as, is there Almost-Leinert cases and how would these behave? Well, analogically, we can visuallize the problem with the following daily circumstance:

<div align="center">
    <img src="/images/Almost_leinert_analogy.png" alt="Almost Leinert Analogy">
    <figcaption>This figure shows how an Almost Leinert Property might behave like with a dressing analogy.</figcaption>
</div>

When dressing up, we are not able to change the order in which we dress the shirt and tie, or the socks and shoes. However, we still have the freedom to dress as, (sock, shirt, shoes, tie) or (sock, shoes, shirt, tie). It is sufficient to dress the shirt before the tie and the socks before the shoes, anything else is possible. So what might the spectral norm of groups which do not strictly follow the Leinert Property be? What is an example of such a case? 


## Methods

## Results

# Conclusion
## Summary
## Takeaways
## References
1. [Akemann, C. A., & Ostrand, P. A. (1976). Computing norms in Group C ∗ -algebras. American Journal of Mathematics, 98(4), 1015. https://doi.org/10.2307/2374039](#akemann)

2. [Hastings, M. B. (2007). Random unitaries give quantum expanders. Physical Review A, 76(3). https://doi.org/10.1103/physreva.76.032315](#hastings)

3. [Woess, W. (1986). A short computation of the norms of free convolution operators. Proceedings of the American Mathematical Society, 96(1), 167–170. https://doi.org/10.1090/s0002-9939-1986-0813831-3](#woess)


# Additional information

* [The slides](/files/Random%20Walks%20on%20Products%20of%20Free%20Groups_Undergrad%20Presentation%20Night.pdf) used for the short presentation at the Presentation Night.
* [The poster](/files/Poster_Almost_Leinert.pdf) used for the Purdue Summer Undergraduate Research Poster Symposium and [Louis Stokes Alliance for Minority Participation (LSAMP)](https://www.purdue.edu/gradschool/diversity/programs/louis-stokes-alliance-for-minority-participation/about/summer-program.php) Research Conference of Summer 2022.

# Acknowledgements

I would like to thank Dr. Thomas Sinclair for his support of the team and allowing me to be a part of such an exciting experience and being an incredible mentor/teacher for both the Presentation Night and the study's development. My team members, Colton, Sanchita and Yuxiao for being an incredible team with insightful ideas and discussions. In addition to [Dr. Jonathan Peterson](https://www.math.purdue.edu/~peterson/) for hosting and organizing such an incredible and insightful event. Also, [Dr. Ignacio Camarillo](https://www.bio.purdue.edu/People/profile/ignacio.html), director of LSAMP, for teaching, organizing, and funding my personal involvement in the project. As well as, the National Science Foundation for funding Dr. Sinclair's work as it was partially funded by the grant DMS-2055155. Lastly, I would like to thank the audience for the attention throughout the presentation and the engagement received in the QA section. I really appreciated all the opportunities I was granted and will forever be in debt to all of you as I thank you all for making my first research experience in Academia one I could never forget.
