---
layout: post
title:  "Making ugly imperative pretty functional in Java"
date:   2020-00-26 22:00:00 +0100
categories: entry
---

I decided to write this article after a refactor today at work which has stuck with me. To give you some context, I was facing a challenge
today where I had a fairly short amount of time to hotfix some extra functionality into a component in order to not miss out on revenue opportunities which
were fluttering by. The component, without going into too much detail, originally consisted of creating simple escape-character-safe regex expressions
from an input phrase. The new functionality required was that component had to be able to create an expression from the same phrase which could recognise certain synonims.
For example, where we might have treated `Link & Zelda` and `Link and Zelda` as two different phrases, we now needed a catch-all regex solution.

The intuitive solution was that I simply iterate over the string for each special case synonym, and replace it with a regex OR clause containing each of the possible synonims.
Ideally, the result of some method would look like this:
```
makeMyPhraseSmartRegex("Link and Zelda"); // -> returns "Link (and|&) Zelda"
makeMyPhraseSmartRegex("Link & Zelda"); // -> returns "Link (and|&) Zelda"
```
My first implementation was fairly clean, although looking back a `reduce` may have been prettier here (but that's spoilers for later!):
```java
public class RegexComponent{
    // configured known special synonims. (By the way, it may be better to make this environmentally configurable, if you have a similar problem ;) )
    private Map<String, List<String>> knownSynonims = Map.of("&", List.of("and"), " and ", List.of(" & "));

    public String makeMyPhraseSmartRegex(final String string){
        ...
        // already existing code
        String loopString;
        for(var entry : knownSynonims){
            loopString = loopString.replace(entry.getKey, convertEntryToRegexOrCriteria(entry));
        }
    }

    private String convertEntryToRegexOrCriteria(Entry<String, List<String>> entry){
        // functionality for producing "( and | & )" from " and " etc.
    }
}
```
But my implementation fell at the first hurdle. Instead of the desired output, I got something like:
```
makeMyPhraseSmartRegex("Link and Zelda"); // -> returns "Link (and|( and | & )) Zelda"
```
Since a lot of these synonims are going to be reversible, with the above implementation, theres a very good chance that every time you replace a word with a 
regex bracket set, **one of the words within those brackets are going to match something else when you iterate through the remainder.** This means we ended up with
the nested replacement you see there.

Obviously this was no good.

So then I had to come up with a solution. The problem then became being able to match many things all at once, so that you don't catch the other ends of synonim pairs.
There are probably a number of approaches one could take; maybe going with something that checks that a match isn't already inside another block. But these kind of logic
checks quickly spiral out of control, and I ferverrently believe that the best logic is no logic, and the next best is dumb logic. Harder to mess it up that way.

The solution I came up with was to avoid iterating over the string while replacing, instead splitting the string up around all known special characters. Theoretically, once we had
split the original string into a stream of string chunks, it would then be a simple matter of going through each chunk, seeing if a chunk is a special character, mapping it with
`convertEntryToRegexOrCriteria` if it is, doing nothing if not, and collecting all the chunks into a single string when getting to the end. This solves the concern about accidentally
replacing synonim pairs since every element in the stream is only going to be mapped at most once.

I was fairly happy with my approach, but as I began to type it out, I could see my code becoming imperative and, well, for want of a better word, shit. The first draft looked something like this:

```java
public class RegexComponent{
    private Map<String, List<String>> knownSynonims = Map.of("&", List.of("and"), " and ", List.of(" & "));

    public String makeMyPhraseSmartRegex(final String word) {
        if (EMPTY_STRING.equals(word)) {
            return word;
        }
        final var regexSafeString = REGEX_METACHARACTERS.matcher(word).replaceAll("\\\\$0");
        Stream<String> stringStream = Stream.of(regexSafeString);
        for (Entry<String, List<String>> entry : regexCaptureProperties.getComputedCachedCharacterMap().entrySet()) {
            stringStream = stringStream.map(str -> splitByWordKeepingWord(str, entry.getKey())).flatMap(Function.identity());
        }
        final var specialWordReplacedString = stringStream.map(str -> {
            final var value = regexCaptureProperties.getComputedCachedCharacterMap().get(str);
            if (value == null) {
                // Doesn't need to be replaced
                return str;
            } else {
                return convertEntryToRegexOrCriteria(Map.entry(str, value));
            }
        });
        return addWordBoundaries(specialWordReplacedString);
    }

    private String addWordBoundaries(final String input) {
        return String.format(BOUNDARY_ENCLOSURE_FORMAT, input);
    }

    private String convertEntryToRegexOrCriteria(Entry<String, List<String>> entry){
        // functionality for producing "( and | & )" from " and " etc.
    }

    private Stream<String> splitByWordKeepingWord(String string, String splitWord) {
        final var splitStringList = Arrays.asList(string.split(splitWord));
        if (splitStringList.size() == 1) {
            return Stream.of(string);
        }
        return addWordBoundaries(specialWordReplacedString);
        final var resultantList = new ArrayList<String>();
        // Add string back
        splitStringList.forEach(str -> {
            resultantList.add(str);
            resultantList.add(splitWord);
        });
        // Remove last added splitWord
        resultantList.remove(resultantList.size() - 1);
        return resultantList.stream();
    }
}
```
There's a feeling of dread I feel when I start to write code like this. It takes me back to my first few weeks of CodeWars Kata. You get addicted to thinking Imperically, and start
to produce all manner of arcane solutions to obscure algorithm problems. Once you get used to a professional enterprise coding environment, where maintanability and stability rign supreme, 
this kind of reckless code screeching makes you sweat. I was on the clock, and had a gut feeling I was overengineering it. I knew this code would need comments to make heads or tails of it.

Recently, a collegue (and very good friend) give me a piece of advice. He said that, where possible, method names should serve as documentaion. They are automatically maintained (unlike code comments),
and incidentally, make a _really_ good litmus test for when you need to go back to your CleanCode :tm: fundamentals. So the first refactor I did was just that: break the imperative up into abstract chunks
which keep the mental thread easier to keep ahold of:
```java
public class RegexComponent {
    private Map<String, List<String>> knownSynonims = Map.of("&", List.of("and"), " and ", List.of(" & "));

    public String makeMyPhraseSmartRegex(final String word) {
        if (EMPTY_STRING.equals(word)) {
            return word;
        }
        final var regexSafeString = REGEX_METACHARACTERS.matcher(word).replaceAll("\\\\$0");
        final var stringStream = splitStringIntoStreamBySpecialWords(regexSafeString);
        final var stringStreamWithWordsReplaced = replaceAllSpecialCharactersInStream(regexSafeString);
        return addWordBoundaries(stringStreamWithWordsReplaced.collect(Collectors.joining("")));
    }

    private String makeStringRegexSafe(final String string) {
        return REGEX_METACHARACTERS.matcher(string).replaceAll("\\\\$0");
    }

    private Stream<String> splitStringIntoStreamBySpecialWords(final String string) {
        Stream<String> stringStream = Stream.of(string);
        for (Entry<String, List<String>> entry : knownSynonims.entrySet()) {
            stringStream = stringStream.map(str -> splitByWordKeepingWord(str, entry.getKey())).flatMap(Function.identity());
        }
        return stringStream;
    }

    private Stream<String> replaceAllSpecialCharactersInStream(final Stream<String> stream) {
        return stream.map(str -> {
            final var value = regexCaptureProperties.getComputedCachedCharacterMap().get(str);
            if (value == null) {
                // Doesn't need to be replaced
                return str;
            } else {
                return convertEntryToRegexOrCriteria(Map.entry(str, value));
            }
        });
    }

    private Stream<String> splitByWordKeepingWord(final String string, final String splitWord) {
        final var splitStringList = Arrays.asList(string.split(splitWord));
        if (splitStringList.size() == 1) {
            return Stream.of(string);
        }
        final var resultantList = new ArrayList<String>();
        // Add string back
        splitStringList.forEach(str -> {
            resultantList.add(str);
            resultantList.add(splitWord);
        });
        // Remove last added splitWord
        resultantList.remove(resultantList.size() - 1);
        return resultantList.stream();
    }

    private String addWordBoundaries(final String input) {
        // functionality for wrapping regex phrase in word boundaries
    }

    private String convertEntryToRegexOrCriteria(Entry<String, List<String>> entry){
        // functionality for producing "( and | & )" from " and " etc.
    }
}
```

I hope you'll agree that while there's certainly room for improvement, this is much nicer than before. The arcane bits of implementation are in small chunks with (kinda?) descriptive names.
But this is all pretty fundamental stuff, which most bootcamp grads can tell you a week out of graduation. So why did I bother writing about it? Well, I was particularly irriated by having to come
up with those variable names in `makeMyPhraseSmartRegex`. They seemed arbitrarty, obvious (the method describes it), and just ugly. It then became apparanet to me that since I had already broken
down all the operations into unary functions, I could just replace all the ugly variables with a nice `Optional` pipeline.

```java
public class RegexComponent {
    private Map<String, List<String>> knownSynonims = Map.of("&", List.of("and"), " and ", List.of(" & "));

    public String makeMyPhraseSmartRegex(final String word) {
        if (EMPTY_STRING.equals(word)) {
            return word;
        }

        final var result = Optional.of(word)
                .map(this::makeStringRegexSafe)
                .map(this::splitStringIntoStreamBySpecialWords)
                .map(this::replaceAllSpecialCharactersInStream)
                .map(stringStream -> stringStream.collect(Collectors.joining("")))
                .map(this::addWordBoundaries);

        return result.orElseThrow();
    }

    private String makeStringRegexSafe(final String string) {
        return REGEX_METACHARACTERS.matcher(string).replaceAll("\\\\$0");
    }

    private Stream<String> splitStringIntoStreamBySpecialWords(final String string) {
        Stream<String> stringStream = Stream.of(string);
        for (Entry<String, List<String>> entry : knownSynonims.entrySet()) {
            stringStream = stringStream.map(str -> splitByWordKeepingWord(str, entry.getKey())).flatMap(Function.identity());
        }
        return stringStream;
    }

    private Stream<String> replaceAllSpecialCharactersInStream(final Stream<String> stream) {
        return stream.map(str -> {
            final var value = regexCaptureProperties.getComputedCachedCharacterMap().get(str);
            if (value == null) {
                // Doesn't need to be replaced
                return str;
            } else {
                return convertEntryToRegexOrCriteria(Map.entry(str, value));
            }
        });
    }

    private Stream<String> splitByWordKeepingWord(final String string, final String splitWord) {
        final var splitStringList = Arrays.asList(string.split(splitWord));
        if (splitStringList.size() == 1) {
            return Stream.of(string);
        }
        final var resultantList = new ArrayList<String>();
        // Add string back
        splitStringList.forEach(str -> {
            resultantList.add(str);
            resultantList.add(splitWord);
        });
        // Remove last added splitWord
        resultantList.remove(resultantList.size() - 1);
        return resultantList.stream();
    }

    private String addWordBoundaries(final String input) {
        // functionality for wrapping regex phrase in word boundaries
    }

    private String convertEntryToRegexOrCriteria(Entry<String, List<String>> entry){
        // functionality for producing "( and | & )" from " and " etc.
    }
}
```

I cant tell you how satisfying this was. The ugly imperative implementation had now been broken down into a number of concise operations, with the 
method invoking those operations doing so in a clear, and concise way:
```java
        final var result = Optional.of(word)
                .map(this::makeStringRegexSafe)
                .map(this::splitStringIntoStreamBySpecialWords)
                .map(this::replaceAllSpecialCharactersInStream)
                .map(stringStream -> stringStream.collect(Collectors.joining("")))
                .map(this::addWordBoundaries);
```
We bypass all need for naming variables, passing one as an argument to the function of the next and instead make the pipeline an explicit action. 
We also get to use Java's method reference syntax which is a _huge_ win. (Not for performance reasons, It's just really pretty).

I'm sure people who are more familiar with hands on experience of functional programming are looking at me like i've just said "wow, the sky is blue!", 
but coming to an intuitive appreciation for a "pretty" functional appraoch (see what I did there) really opened my eyes.

Ofcouse, I'm sure purists can point out a number of reasons why this isn't proper functional programming, but there's a really nice takeaway here for 
a junior Java developer. Breaking your imperative methods into smaller abstract chunks is already a good practice for making code maintaible, testable, 
understandable, simpler... the list goes on, but an added side effect is that it puts a lot of implementations following this pattern in a strong position
to utilise Java's functional programming capabilities. Just keep your methods unary if you can... anything else looks gross! ;)