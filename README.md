# Guardian Automated Style Guide

> Checking your style so you don't have to.

## How it works

A `styleguide` service checks over body of text and applies predetermined rules. A list of rule violations are returned.

The style guide itself makes no changes to the document, it just enables the user to make potentially complex edits in an automated fashion. This model decouples style suggestions from document editing meaning future collabaration features in Composer should not interact badly with the style guide.

```
            Makes edits to document               Pushes Text to...
  ( ͡° ͜ʖ ͡°)  -------------------> +----------+ -----------------> +------------+
    User                         | Composer |                    | styleguide |
            <------------------- +----------+ <----------------- +------------+
           Suggests/Warns to user           Lists violations to...
```

## Violations

When you check a body you get sets of violations which can be either warnings or suggestions.

Some examples:

> User wrote 'the Ukraine'.

Detect bad style and suggest replacing with just 'Ukraine'

> User spelled a name differently a several times: 'Yanukovych', 'Yanukovuych', 'Yanukovich'

Use fuzzy string matching algorithms (Jaro, Levenshtein, etc.) to find similar words which are not common dictionary words and suggest a merge.

## Potential Issues

A list of issues we're likely to come across.

### Document models

The document model of Composer articles/live blogs/etc. is different and important for the style guide to distinguish disperate blocks of text so that it can provide accurate location information.

In order to get around this I suggest we send up the structured JSON as it appears in the Composer document model along with a list of json paths which style guide can use to find the text it considereds the 'body'. This is important since live blogs for example have their text spread across several structured JSON objects but we still want to make sure spelling is consistent across all blocks.

Additionally this means in the future (if composer gets a new document model) it's composers responsibility to update the location mappings for the style guide to continue working.

### Ignoring suggestions or warnings

In order to ignore a suggestion we need to be able to follow a piece of text as it moves about the document during editing. How do we track a given piece of text as the position changes? This is more complex compared to a standard text editing packages 'ignore' feature as we might want to have a style guide violation allowed in the case of a direct '[sic]' quote but maintain good style outside of that quote. So it's vital that we're able to distinguish the same rule violation in different contexts.

I propose a violation ID system which combines the suggested transform or warning (e.g. `the Ukraine -> Ukraine` or `Prefer refugee to migrant`) with a 'context hash'. The context hash will represent the context of the violation without coupling it with the position in our document. An initial idea: the context hash is a simple text hash of the parent sentence. There are obvious downsides to this, changes in the sentence which don't fix the detected violation will cause the violation to be retriggered. There is a careful balancing act to be found here where we don't allow our context to be too large or too small. Suggestions welcome!

## Prospective Endpoints

The following is a list of the endpoints and if they're an objective of the hackday. Since dynamically adding/removing rules is non-functional, but quite a lot of work, it's considered a later task if the project is formally adopted.

| Endpoint          | Verb   | Function                                                                                                                                                                            | Hackday? |
|-------------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| /check?disable=*  | POST   | Send a body of text to be checked, a set of rule violations are returned. Optional `disable` list of rule IDs which informs the service these rules have been deliberately ignored. | Yes      |
| /rules            | POST   | Add a rule to your global rule list. * Rule format to be determined (totes want s-expressions) *                                                                                    | Later    |
| /rules/:id        | DELETE | Delete a rule from the global rule list.                                                                                                                                            | Later    |



## Rule Examples

Rules created by users have some common options such as case sensitivity and a plain text description to give the user.

### Find and Suggest Replacement

> Find a string, and suggest a replacement.

When referring to a proper noun spelling should be in the local variety of English. An example might be 'Department of Defense' which should use the American spelling of defence when referring to the branch of the US government.

### Choice of Words

> Find a string, suggest alternate wording.

Similar to find and replace, but without a concrete 'right' answer, useful in cases where understanding the context of the phrase might be difficult.

### Associated Word Requirements

> If a string is found but another is not, raise a warning.

An example of this might be an article discussing a recent suicide which does not contain contact details for the Samaritans, etc.
