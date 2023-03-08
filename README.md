# Highsignal
## Highest signal relay, ever.

**Goal:** To aggregate the cumulative intelligence of paid relay participants/shareholders such that a hierarchy of Nostr Events emerges, ranked by the value of the meaning embedded in each.

Nominal reactions such as *Likes* or *number of stars* are of limited value.

It's possible to rank Events comparatively against each other using the same types of algorithms used to rank sports players, and this should be far more meaningful than nominal upvotes/downvotes/stars/etc.

A RANKING is a Nostr Event of `Kind 641402` that indicates the signer's opinion on the relative value of one Nostr Event compared to that of another, when judged against exactly one (1) Judgment-Criteria.

* A Ranking SHOULD include the Event ID of a Judgment-Criteria to indicate the criteria by which the Events in this Ranking were judged.
* A Ranking MAY be deleted with a NIP-09 `Kind 5` Event.

A JUDGMENT-CRITERIA is a Nostr Event of `Kind 641404` that a Participant MAY use as a criteria by which to judge Nostr Events.
* A Judgment-Criteria MUST contain at least one [Rule](#) to limit the scope of what Participants SHOULD consider when producing Rankings under this Ranking Criteria.

A SCORING-FUNCTION is a function that takes an array of Rankings as an input and returns an array of Event IDs coupled with their Score.
* The simplest one to implement is Aarpad Elo's chess ranking algorithm.
* A Scoring-Function itself is a Nostr event.
* To instantiate a Scoring-Function, a Participant MUST create an Event of `Kind 641400` containing a short (`>=20` chars) name, and a description of how the Scoring-Function should work.
* To implement a Scoring-Function, a Participant MUST create a Patch to the relay implementing a function with the following signature:
    `func name(rankings []nostr.Event) map[string]int64 {}`

A SCORE is the value (an Integer) given to an Event by a Scoring-Function, and valid only within the Scoring-Pool in which it was calculated. Please read about the [Elo rating algorithm](https://en.wikipedia.org/wiki/Elo_rating_system) for an example on how a score could be calculated and what it means.

A SCORING-POOL is and independent set of Scores (of Events or Objects) produced by a Scoring-Function as calculated from a set of Rankings as input.
* The Scores within a Scoring-Pool have no meaning outside of the pool, the Score of an Event in a Scoring-Pool is relative to all other Events in the pool, and not relative to any Scores from any other Scoring-Pool.
* Meaning is derived by limiting the set of Rankings used as input to a specified criteria such as those of a particular Rating-Criteria and a particular set of pubkeys (e.g. only those in the Nostrocket Identity Tree, or shareholders in the relay weighted by Nostrocket type `Votepower`).
* The Scoring-Pool Type is defined as:
```
    type ScoringPool struct {
     	ScoringFunction  func([]nostr.Event) map[string]int64
     	JudgmentCriteria string //Event ID of the Judgment Criteria used to compute this pool
     	RankingEvents    map[string]nostr.Event //Rankings used to compute this pool
     	WeightedBy       int64 //Shares Held = 0, Votepower = 1, Account Age = 2, Number of Patches Merged = 3, ...
     	Results          map[string]int64
     }
```

Every Block, the relay SHOULD check for any Ranking events that it hasn't seen yet, and if there are new events then recalculate any impacted Scoring-Pools.

**Scoring-Functions and Judgment-Criteria can themselves be scored against each other.**

For example "Which of these two Scoring-Functions gives me the most meaningful result".

This allows us to find the best Judgment-Criteria and Scoring-Function for different types of content (e.g. politically controversial issues might be better approached with different Elo rating variables than pictures of kittens).

The **results of multiple Scoring-Pools can be combined** if some of the Events that have been Scored are common to all of them.

**Idea**: Make a number of Elo Scoring-Functions, randomly varying the K-factor, deviation, and starting score for each one.
See what happens, and score these Scoring-Functions to narrow in on what variables work best, then use this as the basis for the next generation.

**Problem**: Sybil attacks are cheap, fungible identity makes the rankings useless due to insufficient knowledge of who is being sampled.

**Solution**: Only subscribe to rankings from pubkeys that are in the Nostrocket Identity Tree. This makes identity non-fungible and Sybil attacks increase in cost until they are more expensive than the cost to detect and purge a bad actor, thus Sybil attacks are no longer economical.

Relay **Rule**: Participants who are [Excommunicated](#) MUST have their Rankings removed from all Scoring-Pools.

Some example Ranking Criteria:
* "Which of these two Events has the best signal to noise ratio?"
* "Which of these two Events is the most truthful statement?"
* "Which of these two Events best describes Bitcoin to a normie?"
* "Which of these two Events is the best haiku about Nostr?"
* "Which of these two Events is more likely to be Klaus' current location?"
* "Which of these two Events best explains the meaning of life?"

**Other apps can consume the scores** by sending a request to the relay.

The request should contain a list of Event IDs, the ID of a Ranking Criteria, and the ID of a Scoring-Function. If the request only contains a single Event ID, it is taken to mean all replies to that event ID, using the ranking criteria that has the most rankings for this set of events, and the scoring function that is most popular.

The relay will reply with an event containing an array of event IDs and their score, along with the ID of the Ranking Criteria, and the ID of the Scoring-Function.

Events with no rankings are simply given the starting value provided by the Scoring-Function.

These apps can also send Rankings, and if the user's pubkey is in the Identity Tree their Rankings will be counted.

**There is currently a Problem in the issue tracker: [can't find the best Nostr events](#todo).**

Some examples of how this could be used by Nostrocket projects:
* The relative **value of Problems** in terms of bringing in the *very next Participant* to a Nostrocket project, as ranked by Participants who have `Votepower > 0`, and scores weighted by Votepower.
* The relative **accuracy of the amount claimed in an Expense** (based on its Patch), as ranked by Participants who are Contributors (have sent a Patch).
* The relative **improvement in readability** that each Patch brings.
* Creating a **graph of trust**: "Which of these two Participants would I vouch for the most?" This type of system has a problem in that it is vulnerable to grinding attacks, but we've already solved this problem at the Identity Tree layer.


##### Charging for requests:
Anyone can of course run their own relay and query locally for free.

Participants who run a public facing relay can claim an Expense for their time and the cost of the VPS if this solves a Problem. Their implementation MUST follow the rules for the Expense to be valid.

One of those rules could be about giving priority to queries signed by pubkeys that have made a payment to the HighSignal Nostrocket Project, fulfilling some form of payment related requirement.
