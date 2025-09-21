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

### Part A: Partial Matching

**Question:**
In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

**Answer:**
This is teh case because the short string chosen is not connected to the target url. All that is required is that the short url base/domain cannot have repeat strings but there is no connection needed to generate the string from the target. The registration synchronization however is the one that links the 2 allowing one to be used as a proxy so it clearly must have both so that it can register them as linked. 

### Part B: Omitting Names

**Question:**
The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn't this convention used in every case?

**Answer:**
Because as is stated in the question ths is meant to shorten the spec while maintaining clarity. In cases whre the intention is not clear without specifying the arguments because operations will be done on them it is important to specify them. 

### Part C: Inclusion of Request

**Question:**
Why is the request action included in the first two syncs but not the third one?

**Answer:**
The request symbolizes the user intent. It is an action that the user requests or is driven by a user request. THe setting of expiratioin has nothing to do with what a user wants and is just done automaticaly. 

### Part D: Fixed Domain

**Question:**
Suppose the application did not support alternative domain names, and always used a fixed one such as "bit.ly." How would you change the synchronizations to implement this?

**Answer:**
If this was the case, I would not have to include the short base in teh generation of string because everythign would have the same shorturl. Other than that, however there would be no changes required because it is just a single case of the more complex implementation we already have. 

### Part E: Adding a Sync

**Question:**
These synchronizations are not complete; in particular, they don't do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

**Answer:**
sync cleanup
when ExpiringResource.expire (resource: shortURL)
then URL Shortening.delete(shortUrl)

---


## Exercise 3: Extending the Design

### Part A: New Concepts

**Question:**
Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.

**Answer:**
**concept** URLAnalytics

**purpose**
Track and provide access statistics for short URLs, allowing users to monitor usage of their shortened links.

**principle**
Each short URL has associated analytics data that tracks the number of times it has been accessed. Users can view analytics only for URLs they have created.

**state**
A set of AnalyticsRecords with
  • shortUrl: String
  • owner: User
  • accessCount: Integer
  • creationDate: DateTime

**actions**
createAnalytics(shortUrl: String, owner: User): (analytics: AnalyticsRecord)
  requires: no existing AnalyticsRecord for shortUrl
  effects: creates a new AnalyticsRecord with accessCount = 0

incrementAccess(shortUrl: String)
  requires: an AnalyticsRecord exists for shortUrl
  effects: increases accessCount by 1

getAnalytics(shortUrl: String, requester: User): (accessCount: Integer)
  requires: AnalyticsRecord exists for shortUrl and requester is the owner
  effects: returns the current accessCount for the shortUrl

### Part B: Essential Synchronizations

**Question:**
Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics.

**Answer:**

sync createAnalytics
when URLShortening.register (shortUrl, owner)
then URLAnalytics.createAnalytics (shortUrl, owner)

sync recordAccess
when URLShortening.lookup (shortUrl)
then URLAnalytics.incrementAccess (shortUrl)

sync viewAnalytics
when Request.getAnalytics (shortUrl, requester)
then URLAnalytics.getAnalytics (shortUrl, requester)

### Part C: Feature Assessment

**Question:**
As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:

1. Allowing users to choose their own short URLs
2. Using the "word as nonce" strategy to generate more memorable short URLs
3. Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL
4. Generate short URLs that are not easily guessed
5. Supporting reporting of analytics to creators of short URLs who have not registered as user

**Answer:**
1. Modify the NonceGeneration concept to accept user-provided strings as an optional parameter. Add validation to ensure the chosen string isn't already used.
Implementation:
Add generateWithCustom(shortUrl: String, context: String) action to NonceGeneration
Add sync: when Request.shortenUrlWithCustom(shortUrl, targetUrl) then NonceGeneration.generateWithCustom(shortUrl, context)
Good modularity - minimal changes to existing concepts seems like something that users would like.

2. Create a new concept WordNonceGeneration that extends NonceGeneration with a dictionary of words.
Implementation:
New concept with generateWord(context: String): (word: String) action
Replace NonceGeneration.generate with WordNonceGeneration.generateWord in syncs
Assessment: Excellent modularity - completely separate concept, easy to swap

3. Extend URLAnalytics state to include targetUrl field and add grouping actions.
Implementation:
Add targetUrl: String to AnalyticsRecords state
Add getAnalyticsByTarget(targetUrl: String, requester: User) action
Assessment: Good modularity - only changes URLAnalytics concept. Seems like something that users would like it is already in many products. 

4. Modify NonceGeneration to use cryptographically secure random generation instead of sequential.
Implementation:
Change the generation algorithm in NonceGeneration.generate
Assessment: Excellent modularity - no interface changes, just internal implementation. Im not sure this is valuable though it really just makes it harder to navigate to the site. The goal of the product is to make it easy to share links not make those links secure it is the responsibility of the webiste being navigated to to make sure getting there cannot cause a problem on its own. 

5. How to realize: This is undesirable and should not be included.
Why undesirable:
Security risk: No way to verify ownership without user registration
Privacy concerns: Anyone could claim to be the creator
Data integrity: No reliable way to track who created what
Abuse potential: Users could claim analytics for URLs they didn't create
Assessment: This feature would break the security model and should be rejected.

*Problem Set 2 completed on 9/21*
