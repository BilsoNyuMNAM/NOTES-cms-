## When studyig something

when i ask you something ie. "what is useMemo and useCallback for performance optimization", follow this pattern: first start with explaining the problems (in details) we face without the given concept (userMemo, useCallback) ie. unnecessary api calls on re render, etc and then boil down the root cause of the problem,ie "so the root cuase is we are making calls when we don't need (on rerenders)"and then ask "so how can we solve this problem?" and then introduce the cocept (ie, useMemo and useCallback) and how it solves the problem. follow this natural chain of thought order while explaining things. 


## when learning a new concept

as a human, what questions should i ask to learn [XYZ]


## learning something from the first principles

When explaining [SOMETHING SPECIFIC YOU WANNA LEARN or just add "anything"], use a natural progression of thoughts where each step leads logically to the next. Start with the core challenge, then walk through the reasoning process step by step, showing how each insight builds on the previous one. For example, when explaining the minimum difference problem: "We need to find the minimum difference between any two elements in an array. When is this difference smallest? When two numbers are as close as possible to each other on the number line. How can we easily identify adjacent numbers? By arranging all elements in order. What's the most efficient way to arrange elements? By sorting the array. Once sorted, we just need to check differences between consecutive elements to find the minimum." Please apply this cause-and-effect reasoning to any problem I ask about. Connect the dots in a way that feels like a natural thought process, where each insight flows from the previous one until we reach the complete solution. and emphasize more on "why" aspect


## Refined version

when i ask you something ie. "what is useMemo and useCallback for performance optimization", follow this pattern: first start with explaining the problems (in details) we face without the given concept (userMemo, useCallback) ie. unnecessary api calls on re render, etc and then boil down the root cause of the problem,ie "so the root cuase is we are making calls when we don't need (on rerenders)"and then ask "so how can we solve this problem?" and then introduce the cocept (ie, useMemo and useCallback) and how it solves the problem.

then walk through the reasoning process step by step, showing how each insight builds on the previous one. For example, when explaining the minimum difference problem: "We need to find the minimum difference between any two elements in an array. When is this difference smallest? When two numbers are as close as possible to each other on the number line. How can we easily identify adjacent numbers? By arranging all elements in order. What's the most efficient way to arrange elements? By sorting the array. Once sorted, we just need to check differences between consecutive elements to find the minimum." Please apply this cause-and-effect reasoning to any problem I ask about. Connect the dots in a way that feels like a natural thought process, where each insight flows from the previous one until we reach the complete solution. and emphasize more on "why" aspect

keep the format of whole chat based on first priciple thinking: where we ask the natural, human like question that leads to the other piece and so on. this we we reach the truth why following the human curiosity. ie. so what we used to use before these hooks? okay, so what were the problems in those methods? what is the root cause/s of the problem/s? how does [hooks (or the given)] concept fix it?. ASK natural, human like questions to yourself wherever needed and then explain the concept.

also remember, you are explaining this to an absolute beginner so keep the words, sentences and tone easy, simple, digestable and fun (explaining with fun examples or analogies would be awesome).  (don't create response for any example given in this prompt, it's only for your understanding)

# make smth readable

When I give you a piece of text and ask you to make it “readable,” transform it into a visually readable format while keeping the original wording, meaning, and structure as much as possible.

Follow these rules strictly:

1. Keep the original wording
- Do not rewrite, summarize, simplify, or paraphrase the content.
- Only make minor changes if absolutely necessary for readability.
- Preserve technical terms, names, examples, quotes, and emphasis.

2. Use sentence-based formatting
- Each sentence should be treated as its own visual unit.
- After every complete sentence, add a blank line before the next sentence.
- A complete sentence normally ends with `.`, `?`, or `!`.
- Do NOT add blank lines after commas, colons, semicolons, dashes, or other punctuation that does not end a sentence.

Example:

This is the first sentence.

This is the second sentence.

This is the third sentence.

3. Keep each sentence together
- Do not arbitrarily split one sentence into multiple separate paragraphs.
- Keep a sentence on one line when it is reasonably short.
- If a sentence is extremely long and the interface naturally wraps it visually, that is fine, but do not manually break it into separate lines just for formatting.

For example, keep this together:

“The ‘developer experience’ bait-and-switch” by Alex Russell is a great example.

Do NOT format it like this:

“The ‘developer experience’ bait-and-switch”
by Alex Russell is a great example.

4. Capitalization
- Keep necessary capitalization for proper nouns, names, brands, technologies, acronyms, titles, and sentence beginnings where appropriate.
- Otherwise, prefer lowercase / smallcase.
- Do not unnecessarily capitalize words.

For example:

the frontend is changing because of AI.

React is heavily overrepresented in the training weights.

5. Preserve paragraphs and headings
- Keep headings as headings.
- Keep numbered lists and bullet points where they are part of the original structure.
- Do not turn everything into one giant block of text.
- However, within normal prose, use the sentence-by-sentence blank-line format described above.

6. Do not add commentary
- Do not explain what you changed.
- Do not introduce the output with phrases like “here’s the readable version.”
- Just give me the transformed text.

7. The desired visual style is:

first sentence.

second sentence.

third sentence.

fourth sentence.

This should feel spacious and easy to read, with a clear blank line between every sentence.

The most important rule is:

EVERY COMPLETE SENTENCE → BLANK LINE → NEXT SENTENCE.

Do not add extra blank lines within a sentence.