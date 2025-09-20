# Problem Set 2: Cole Ruehle

## Exercise 1: Concept Questions

### Part A: Contexts

**Question:**
The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

**Answer:**
For URL shortening it will be used to be the domain. The context for each url is the domain upon which we append a string and we cannot share 2 same strings for a domain. 

### Part B: Storing Used Strings

**Question/Requirement:**
Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

**Answer:**
It is important because if we want to be able to remove and reuse strings. If that is the case then when a string is removed and no longer in use the counter will not reflect that. In the example that we only have 3 strings to chose from we assign all 3 then unasign the second one. The counter would roll over and try to assign 1 next, or think that there are none available. But the correct solution would be to know that the second was not assigned and use it. 

### Part C: Words as Nonces

**Question/Requirement:**
One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

**Answer:**
 A disadvangeage of this scheme is that it means you have fewer available strings to chose from. The set of words of x characters is less than the set of all strings. From the user's perspective this means you may need to wait to deal with quicker refreshing URLs as you have to refresh urls when you run out. But from a user perspective they can chose reasonable words that are relevant, like quiz for a quiz, or info for an informational form. These are more memorable and relevant.

---

## Exercise 2: Synchronizations

### Part A: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

### Part B: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

---

## Exercise 3: [Exercise Title]

### Part A: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

### Part B: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

---

## Exercise 4: [Exercise Title]

### Part A: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

### Part B: [Subsection if needed]

**Question/Requirement:**
[Your question or requirement here]

**Answer:**
[Your detailed answer here]

---

## Additional Notes

[Any additional thoughts, reflections, or notes about the problem set]

---

*Problem Set 2 completed on [Date]*
