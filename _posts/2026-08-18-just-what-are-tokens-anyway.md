# Just what are tokens, anyway?

Like my coworkers, I use AI for coding every day. Until recently, we haven't had to pay much attention to pricing. We were simply encouraged to experiment with the tools and adopt them as much as possible. Our usage didn't seem to have any limits whatsoever.

That's due to change, since software companies are realizing that overspending on AI is a real possibility. In the long term, AI companies are not going to offer an unreasonably cheap product. Remember how cheap Uber and Airbnb were when they first came along? Turns out this was because their cost of borrowing was essentially below zero in their rapid scaling phase. But over time, investors want a return on capital. So AI companies will need to turn a profit at some point.

In the future, the individual contributor will likely need to justify whether they are using AI efficiently, and this will come down to how they are using their tokens.

So this begs the question: just what are tokens, anyway?

## Not a word, not a character

A token is the atomic unit of text that a large language model works with. Roughly speaking, one token is about four characters, or three quarters of an English word.

Common words like _cat_ or _run_ are each a single token: `cat`, `run`.

Longer or less common words get split up, for example _tokenization_ becomes `token` and `ization`. Punctuation, spaces, and newlines are tokens too.

OpenAI has a [tokenizer playground](https://platform.openai.com/tokenizer) where you can see this in action. It's not very intuitive how the tokenizer breaks up words. This is because the process was designed for statistical efficiency, not human intuition.

So why go and abstract words into tokens? Why not just use words?

## A bit of history

Early NLP models did use words. Each word in the vocabulary got a unique ID. The problem is that natural language is always evolving. New words appear constantly, for example jargon, brand names, slang, and compound words. If a new word hadn't been seen during training, it was an "unknown" token and the information it represented was thrown away.

Character-level models solve that problem. If you only operate on individual characters, you'll never encounter an unknown token.

But characters make sequences very long. It's hard for a model to learn that "r-u-n-n-i-n-g" and "r-u-n" are related concepts.

The solution, now used in basically every major LLM, is **Byte Pair Encoding** (BPE). BPE was originally invented in the 1990s as a data compression algorithm and had nothing to do with language models at all. A [2016 paper](https://arxiv.org/abs/1508.07909) by Sennrich, Haddow, and Birch adapted it for neural machine translation and then the approach stuck.

## Byte-Pair Encoding

The BPE algorithm goes like this. Start with individual characters, then iteratively merge the most frequently occurring adjacent pair of tokens into a single new token. Repeat until you hit your target vocabulary size, usually somewhere in the tens of thousands. Common words end up as a single token. Rarer words get split into recognizable subword pieces. Random strings are handled character by character.

So BPE is a sort of happy medium between character-level and word-level encoding. It lets the data decide where the boundaries are. Frequent sequences get compressed into single tokens, and rare ones fall back to characters. The vocabulary stays manageable and nothing can ever truly be unknown.

The other nice property about BPE is that it's language-agnostic. You don't have to define what a "word" is — which is actually a hard problem in languages that don't use spaces as word separators, like Chinese or Japanese. BPE just looks at byte frequencies and figures it out.

## Why tokens drive pricing

So here's a practical question: why do API providers charge per token?

Well, the amount of compute required to process one token is roughly constant. Since each token goes through every layer of the transformer, if you double the token count, you roughly double the compute and thus roughly double the cost.

But there's a catch: **input tokens and output tokens are priced differently**, and output is almost always more expensive. Right now, Claude Sonnet 5 charges $2 per million input tokens and $10 per million output tokens. That's a 5x difference.

Why does this differential exist? The answer is that generating output is fundamentally more expensive than reading input.

When you send a prompt, the model processes all input tokens in one big batch. But generating a response is self-referencing: the model produces one token at a time, and each new token depends on all the ones before it. (In LLM jargon this is called _autoregressive generation_.) That means one full forward pass through the entire network _per output token_. It's slow and expensive in a way that prompt processing is not.

Some providers also offer **prompt caching**. If you send the same large system prompt with every request, you're paying to recompute it every time. With caching, the provider stores the intermediate computation and charges you a fraction of the normal rate on cache hits. For applications with large, repetitive context, the savings can add up.

## The gotchas

Token counts aren't always intuitive. Code tends to be more token-dense than prose because of punctuation, special characters, and camelCase identifiers. A line of Go code and a sentence of English might use the same number of tokens, but to a human the sentence has more meaning.

Non-English languages also tend to cost more. English dominates most training datasets, so it gets the most efficient tokenization. Languages with less training data representation require more tokens for the same semantic content. It's not a fundamental limitation, just an artifact of how the technology was trained.

It's a little funny to think that a compression algorithm from the mid-1990s is the foundation of the most sophisticated AI systems built today. But simple, elegant solutions to practical problems often outlast their original purpose.