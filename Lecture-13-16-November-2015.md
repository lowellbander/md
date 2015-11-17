## Lecture 13: 16 November 2015

### Converting to CNF for FOL 

In propositional logic (PL), resolution is guaranteed
to terminate, either at a contradiction or showing consistency. 

In first-order logic (FOL), resolution is NOT guaranteed to
terminate. It can end in contradiction, but it may not. 

NOTE: new conventions, following the homework example: 

* FOR ALL: A 
* THERE EXISTS: E 
* AND: AND // Too hard to read when using '&' and '|'
* OR: OR
* NOT: ~

Example for CNF conversion:

    A x [A y, Animal(y) => Loves(x,y)] => [E y, Loves(y, x)]

"Everyone who loves all animals is loved by someone"

Break it down into pieces for translation.

Okay, steps for converting to CNF: 

1. Eliminate =>, <=> 

    (A x) [~[(A y) ~Animal(y) OR Loves(x,y)]] OR [E y, Loves(y,x)]

Note where the negation goes: 
    
    A x, P => Q

Becomes: 

    A x, ~P OR Q

2. Move negation (~) inwards. 

Recall the rules for this (analogous to De Morgan's): 

* ~(A x, P) => (E x, ~P)
* ~(E x, P) => (A x, ~P)

Moving inward: 

    A x [E y, Animal(y) AND ~Loves(x,y)] OR [E y, Loves (y,x)]


3. Standardize variables (each quantifier gets a different variable)

    A x [E y, Animal(y) AND ~Loves(x,y)] OR [E z, Loves(z,x)]

4. Skolemization (gets rid of existential quantification, as we eventually
get rid of all quantifiers, assuming the result is universally quantified)> 

Example:

    (A x, E y) Loves(y,x)

Everyone has someone who loves them. But the the particular y who loves x is
rather unique to x. So, we define a function F(x) that gives back the person
that loves x, as that particular y is rather unique to x, almost a property of
it.

    A x, Loves(F(x), x)

Skolem showed that transforming a KB this way led to an equally satisfiable 
KB (only satisfiable if the original was, and vice versa). 

NOTE: F(x) must be unique to the knowledge base, or else we're setting up links
that may have unintended side effects. 

Note that skolemization can depend on more than one variable
(F(x,z)) or none at all (when we just have a constant), i.e., F(x) = c
for any x. 

What about something like: 

    E x, King(x)

Make up a **Skolem constant** that is unique to the KB: 

    King(G)

Back to our original example, replacing with functions and constants
to Skolemize our sentence: 

    A x [Animal(F(x)) AND ~Loves(x, F(x))] OR [Loves(G(x), x)]

And applying the rule that (A AND B) OR C => (A OR C) AND (B OR C): 

    A x [Animal(F(x)) OR Loves(G(x), x)] AND [~Loves(x, F(x)) OR Loves(G(x), x)]

5. Drop universal quantifiers, as they're assumed for Skolemized sentences.

    [Animal(F(x)) OR Loves(G(x), x)] AND [~Loves(x, F(x)) OR Loves(G(x), x)]

Notice that now have CNF: (P OR Q) AND (~R OR Q). Our literals are now 
predicates, functions, etc. instead of simple constant atoms. 


### Resolution on FOL KB 

English story: 

1. Jack owns a dog:
    
    E x Dog(x) AND Owns(Jack,x)

2. Every dog owner is an animal-lover. 
    
    A x [E y, Owns(x,y) AND Dog(y)] => Alover(x)

3. No animal-lover kills an animal. 
    
    A x Alover(x) => [A y, Animal(y) -> ~Kills(x,y)]

4. Either Jack or curiosity killed the cat Tuna. 

    Kills(Jack, Tuna) OR Kills(Curiosity, Tuna)
    Cat(Tuna)
    A x, Cat(x) => Animal(x)

That's our knowledge base delta. Now is alpha = Kills(Curiosty, Tuna)
implied by delta? Does delta |= alpha? 

Resolution with this added to our KB: delta AND ~alpha (which is in CNF). 
Recall that this is how we did this with proof by contradiction. 

Conversion to CNF: 

(1) requires just Skolemization, here introducing a constant: 
    
    Dog(D)
    Owns(Jack, D) // Recall that KB's are conjoined terms, so we can split them

(2) becomes: 

    ~Owns(x,y) OR ~Dog(y) OR Alover(x)

(3) becomes: 

    ~Alover(x) OR ~Animal(y) OR ~Kills(x,y)

(4) is already a clause: 

    Kills(Jack, Tuna) OR Kills(Curiosity, Tuna)

(4b) is already a unit-clause: 

    Cat(Tuna)

(4c) becomes: 

    ~Cat(x) OR Animal(x)


Our final KB in CNF form: 

1. Dog(D)
2. Owns(Jack, D)
3. ~Owns(x,y) OR ~Dog(y) OR Alover(x)
4. ~Alover(x) OR ~Animal(y) OR ~Kills(x,y)
5. Kills(Jack, Tuna) OR Kills(Curiosity, Tuna)
6. Cat(Tuna)
7. ~Cat(x) OR Animal(x)
8. ~Kills(Curiosity, Tuna) // Our negated query

Just like PL, we do resolution on terms that share
B and ~B (literal and its unification), but now we 
may need to do unification to match.

Look at (1) and (3). Dog(D) and ~Dog(y) don't quite 
mirror, unless we bind/unify y to D: theta = { y/D }.
Remember the result comes AFTER we substitute in D: 

1. Dog(D)
2. Owns(Jack, D)
3. ~Owns(x,y) OR ~Dog(y) OR Alover(x)
4. ~Alover(x) OR ~Animal(y) OR ~Kills(x,y)
5. Kills(Jack, Tuna) OR Kills(Curiosity, Tuna)
6. Cat(Tuna)
7. ~Cat(x) OR Animal(x)
8. ~Kills(Curiosity, Tuna) // Our negated query
9. ~Owns(x,D) OR Alover(x) // 1,3 where theta = { y/D } 
10. Alover(Jack) // 2,7 where theta = { x/Jack } 
11. Animal(Tuna) // 6,7 where theta = { x/Tuna } 
12. ~Animal(y) OR ~kills(Jack, y) // 4,9 where theta = { x/Jack }
13. ~Kills(Jack, Tuna) // 10,11
14. Kills(Curiosity, Tuna) // 5,12 => CONTRADICTION!


### Reduction to Propositional Logic (Propositionalization)

By converting to PL, we can turn it into a CSP and throw it at a SAT solver. 
This is cheating to avoid the complexity/sophistication of FOL. 

We want to take something that is FOL and convert it to PL. 

Example: 

    A x, King (x) AND Greedy(x) => Evil(x) 

And suppose we only have a PL reasoning engine. We can't have
functions, variables, quantifiers, etc. 

But what we can do is generate instances over the whole domain of x, 
as that gives us Boolean constants. 

    King(P1) AND Greedy(P1) => Evil(P1)
    ...
    King(P5) AND Greedy(P5) => Evil(P5) 

Now, for each one, we need only substitute a propositional variable 
for each term, e.g., x1 = King(P5), which is now true/false. 

    x1 AND y1 => z1
    ...
    x5 AND y5 => z5

Problem: notice how we could end up with a very large PL KB despite 
only a few variables. Sometimes, this approach actually wins out. 
We can also become more sophisticated: we can filter what to add
to our KB or not, based on our query. 

But we're not done propositionalizing just yet: we haven't converted 
functions, e.g., Father(x,y) means we'll need King(Father(P1)) AND ...
Now, instead of a linear relationship of possible variable values to 
number of sentences in our KB, we could possibly have infinitely many 
sentences, as we can introduce functions of arbitrary complexity 
and interconnectedness. No longer 5 values to 5 sentences; we could
have those 5, then accounting for father generates many more. 


#### Semi-Decidability 

Turing/Church theorem result: does delta |= alpha? We can convert to PL and
reason, but to avoid the infinity, we limit how much we nest (0 is no function
applications, 1 is one function application, 2 has function nesting, etc.).

Then, if we limit our testing, then we will generate a finite PL KB. If no
entailment at that level of nesting, then add another level, and keep going. If
alpha does follow, then we will terminate at some nesting level. But if alpha
does not, we could go on infinitely.

If it follows, we can tell you; if it does not follow, we cannot tell you.
Sometimes, FOL work is done without function symbols altogether, and/or 
with a finite domain (to avoid infinitely many values). 

Datalog (in databases) is apparently related to this. 

FOL courses also exist in graduate CS, philosophy, and math.


### Probabilistic Reasoning

~1979: McCarthy writes about how formal logic sucks at encoding 
common-sense knowledge. 

Example problem: 

    Bird(x) => fly(x) 

But if we try to write in exceptions, we contradict ourselves: 

    Bird(Tweety) AND ~Fly(Tweety) 

So we try to define that a bird can be abnormal and therefore it doesn't fly.
But that doesn't work either. 

How about assuming something is true until we know otherwise? That births
**non-monotonic logics**. They make assumptions based on existing knowledge, 
and can retract them with new knowledge. This is difficult mathematically. 

Another example: 

    A x, Quaker(x) AND ~Abnormal(x) => Pacifist(x)

    A x, Republican(x) AND ~Abnormal(x) => ~Pacifist(x) 

By default, if we learn someone is a Quaker, we assume there are a pacifist, 
as we initially assume that they are not abnormal.

But what if we learn: 

    Quaker(Nixon)
    Quaker(Republican)

The default conclusions clash: Pacifist and not Pacifist? The two normality
assumptions can't both be true. We have no way of managing our (possibly
conflicting) assumptions, which lead to **belief revision**.

We want to get to Bayesian networks, which will be built atop probability
calculus.





