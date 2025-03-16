
# Introduction

The Advent of Code[@wastl_adventofcode] is a programming contest held annually during the first 25 days of December.  A total of 25 puzzles are released, one a day. Each puzzle has two parts, except for the final day, which has just one part. The puzzles vary in difficulty. Early-day puzzles and weekday puzzles tend to be easier than later-day and weekend puzzles. The second part of a puzzle cannot be attempted until the first part is solved. The second part is sometimes significantly more difficult than the first part. Although the puzzles are typically solved using Python [@wastl_adventofcode_keynote_2022], contestants are permitted to use any programming language, or indeed any puzzle-solving technique of any kind, to solve the puzzles.

This paper measures how well the Google Gemini 2.0 Flash Thinking EXP 01-21 LLM can solve Advent of Code puzzles. To successfully solve an Advent of Code puzzle, an LLM must interpret potentially ambiguous instructions, select an algorithm, generate valid code, and debug any errors, timeouts, or incorrect answers in the resulting program.

# Methodology

## Models Tested

I tested the Google Gemini 2.0 Flash Thinking EXP 01-21 model. This model was chosen because it is representative of recent reasoning models, and Google provides a public API for accessing the model with a generous free quota.

## Programming Languages Tested

While Python is the most common language used to solve Advent of Code puzzles, many other languages are commonly used. I chose to test the following languages. In each case, the runtime for the language was installed on the test machine (a Macbook Pro running MacOS 15.3).


| Language    | Version                          |
| ----------- | ------------------------------ |
| C           | clang 16.0.0                   |
| C#          | dotnet 9.0.102                 |
| C++         | clang 16.0.0                   |
| Clojure     | 1.12.0.1495                    |
| Common Lisp | SBCL 2.5.0                     |
| Dart        | 3.6.1 (stable)                 |
| F#          | dotnet 9.0.102                 |
| Go          | go1.23.5                       |
| Haskell     | ghc 9.4.8                      |
| Java        | javac 23.0.1                   |
| JavaScript  | node v22.13.0                  |
| Kotlin      | kotlinc-jvm 2.1.0 (JRE 23.0.1) |
| Lua         | Lua 5.4.7                      |
| Objective-C | clang 16.0.00                  |
| OCaml       | 5.2.1                          |
| Perl        | v5.34.1                        |
| PHP         | 8.4.3                          |
| Python      | 3.13.1                         |
| Ruby        | 2.6.10p210                     |
| Rust        | cargo 1.84.0                   |
| Smalltalk   | GNU Smalltalk                  |
| Swift       | clang 16.0.0                   |
| TypeScript  | 5.7.3                          |
| Zig         | 0.13.0                         |

## Experimental Setup

I used the google.generativeai Python API to access the model.

All model parameters (such as temperature) were left at their default values.

See [https://github.com/jackpal/aoc_conversation](TO BE RELEASED) for the client-side code.

In order to reproduce this paper, you will need to spend about 12 hours of wall clock time.

Each language takes between 49 and 245 conversation turns to test. 49 is the best case if every puzzle is solved correctly on the first try. 5 \* 49 == 245 is the worst case, if every puzzle takes 5 turns to solve.

Double-check current pricing and quota for Gemini, as the pricing may have changed since this document was written.

| Model                               |   Free Quota Limits |
| ----------------------------------- | ------------------: |
| gemini-2.0-flash-thinking-exp-01-21 | 1500 requests / day |

The experiment data was collected from 2025-01-31 to 2025-02-03. The start date was chosen to avoid polluting the contest leaderboard with LLM-generated results. Only year 2024 puzzles were tested to reduce the likelihood that the puzzles and their solutions would be in the training set of the LLM. Google reports that the knowledge cutoff date for Gemini Flash Thinking 2.0 is August 2024.

The [Advent of Code Data](https://pypi.org/project/advent-of-code-data/) library was used to manage a local cache of puzzle instructions, input, and answer data. This minimized the load on the Advent of Code contest infrastructure.

Puzzles were executed one at a time. For each (model, puzzle, language) tuple, the puzzle-solving process was a loop:

```python
if part == 2:
    preload_conversation(year,day,1) # preload the existing part 1 conversation
prompt = initial_prompt(puzzle_instructions(year,day,part))
for turn in range(5):
    response = conversation_turn(prompt)
    program = parse_program_(response)
    results = check_answer(run_program(program))
    if success(results):
        break
    else:
        prompt = incremental_prompt(results)
```

The `initial_prompt` function provided the puzzle instructions and requested the LLM to generate a program in the target language. The `incremental_prompt` function provided the results of the previous attempt (e.g., compiler errors, runtime errors, timeout status, or incorrect answer) and asked the LLM to correct the program. 

The limit of 5 conversation turns was chosen by trial and error during development. It was common for a LLM to need 3 to 5 turns to solve a puzzle. The first turn might result in a syntax error, the second in an input parsing runtime error, the third in a timeout, the fourth in an incorrect answer, and so on.

# Results

## Comparing Language Performance

When comparing model performance using different languages, it is useful to distinguish between the different ways that a program might fail to generate the correct answer:

| Status        | Meaning                                                                                   |
| ------------- | ----------------------------------------------------------------------------------------- |
| correct       | The program generated by the LLM produced the correct answer within 5 conversation turns. |
| incorrect     | On the 5th turn, the program generated by the LLM produced an incorrect answer.           |
| timeout       | On the 5th turn, the program generated by the LLM ran for more than 30 seconds.           |
| error         | On the 5th turn, the program failed with either a syntax error or a runtime error.        |
| not attempted | The puzzle was not attempted due to part 1 of the same puzzle not being solved correctly. |

Note that there are 49 possible puzzles, which is why the percentages shown are not even.

![Solvability by Language and Puzzle](figures/languages_heatmap.png)

![Solvability by Language](figures/languages_ranking.png)

## General Observations

The model was run at its default temperature, which introduced variations in results over repeated runs. The model might succeed in solving a given puzzle on one run but fail on another run.  The reported numbers could change +/- 2 puzzles if the same experiment were performed again.  The language ranking represents a single sample from a probabilistic distribution. A future version of the experiment loop could sample each case multiple times, to produce a more nuanced view.

## Puzzle-Specific Observations

Some puzzles were particularly hard for the model. The following puzzles were not solved by any language:

| Day-Part | Notes                                                                  |
| -------- | ---------------------------------------------------------------------- |
| 6-2      | Ambiguous instructions.                                                |
| 12-2     | Ambiguous instructions, domain knowledge of corner-counting algorithm. |
| 14-2     | Ambiguous instructions.                                                |
| 15-1     | Difficult simulation.                                                  |
| 15-2     | Not attempted because no experiment solved prerequisite 15-1.          |
| 17-2     | Couldn't brute force. Required thinking outside of the box.            |
| 21-1     | Ambiguous instructions, performance issues with naive solution.        |
| 21-2     | Not attempted because no experiment solved prerequisite 21-1.          |
| 24-2     | Required thinking outside of the box to solve in reasonable time.    

FWIW these puzzles were difficult for humans to solve, too.

## Language-Specific Observations

When interpreting the language rankings, keep in mind that the model has a non-zero temperature, so results vary from run to run.
Any two languages that are within a few percent of each other on the chart could swap positions in subsequent runs.

We see that a large number of languages have roughly the same solution rate. For these languages, the model is capable of rendering a given algorithm in that language. These include C#, Dart, Go, Java, JavaScript, Kotlin, Lua, Objective-C, PHP, Python, Ruby, Rust, Swift, and TypeScript.

Python and Rust are the two most popular languages used by Advent of Code participants. This may explain why Rust fares so well.

Many less popular languages suffer from a large number of "errors," which covers any compile-time or runtime error such as a syntax error, a type error, or a memory fault.

The C language suffers from memory issues. The model doesn't use dynamic data structures (even when prompted) and can't debug the resulting memory access errors. C++ fares better due to the standard library of common data structures.

Haskell suffers from a dialect problem, as the model tries to use language features without properly enabling them.

The Lisps (SBCL and Clojure) suffer from paren mis-matches and mistakes using standard library functions.

Smalltalk suffers from calling methods that are not available in the specific Smalltalk dialect being used.

Zig code generation suffers from confusion over whether variables should be declared const or non-const. The model has trouble interpreting the Zig compiler error messages, which report error line numbers relative to the function start rather than relative to the file start.

## In-Model Code Execution

The Gemini model supports code execution. When this feature is enabled, the model can construct and execute small Python programs during the conversation. I tested this feature in three different ways.

![Code Execution](figures/code_execution_heatmap.png)

The 4 variations of the model tested were:

| Suffix         | Solvability |
| -------------- | ----------: |
| none           |         69% |
| code_execution |         59% |
| ce_prompt      |         57% |
| ce_answer      |         16% |

The `code_execution` version simply enabled code execution, without any additional prompts.

The `ce_prompt` version enabled code execution, but also explicitly prompted the model to use code execution to test whether the generated program produced the correct answer on the example input.

The `ce_answer` version was a complete rewrite of the prompt, to include the actual puzzle input, and to request that the model produce the answer for the given input, rather than a program to be run.

The  `code_execution` and `ce_prompt` were similar enough that the difference may just be noise. Surprisingly, the results or using code execution was clearly worse than not using code execution. This may be because the model became too confident in its reasoning, given that it succeeded in solving the example input.

The `ce_answer` result was particularly poor. One reason appears to be that the model may have trouble properly parsing and using the actual puzzle input, which was often in the form of a very large batch of numbers or a large character grid.

## A digression: the effect of memorization on solvability

The Advent of Code contest fosters an [active open-source and open-discussion culture](https://www.reddit.com/r/adventofcode/). Past contest puzzle algorithms and solutions are widely available on the web. To discourage trivial cheating, participants are discouraged from publishing the actual input data and answers. What this means is that solutions to past years Advent of Code puzzles are widely available on the web and are probably present in most frontier LLM training data.

Looking at the solve rate for each year, we see that, in general, the model did better on older years than it did in 2024. But note that many years were not perfectly solved, despite the likelihood that the LLM did see the solutions for those years in the training data.

![AoC All Years Puzzle Solvability](figures/all_years_status.png)

![AoC All Years Puzzle Heatmap](figures/all_years_heatmap.png)

The model had trouble with 2019's IntCode puzzles, which were a series of simple virtual machine interpreter puzzles that built upon previous puzzles. This puzzle set was also difficult for humans.

# Discussion

Advent of Code puzzles are not necessarily representative of general coding problems. They are stylized, and their instructions are written with enjoyable but potentially distracting Christmas-themed details. On the other hand, the puzzle descriptions are extensively tested during development. This leads to specs that contain a lot of useful details.

This year's puzzles were weighted towards 2D grid problems. Model performance may have been affected by that.

## Directions for future work

The experiment could be continued, both to produce error bars for the existing measurements and to measure additional languages and models.

The experiment framework does not handle quota exhaustion as efficiently as it could. It currently throws away the entire conversation for the current puzzle. It would be more efficient to preserve the existing conversation, to be resumed when quota is available.

Would asking the models to generate typed Python help catch syntax errors? (Initial experiments showed no benefit to adding types, perhaps due to these problems not requring elaborate types.)

Would providing a library that implements abstractions common to Advent of Code problems help? Peter Norvig has developed [a promising library for Python](https://github.com/norvig/pytudes/blob/main/ipynb/AdventUtils.ipynb).

Would providing solution examples from previous AoC puzzles help?

When a model gets stuck in a rut, would asking the model to start over help?

It would be interesting to develop a heuristic for how many turns to give a model to solve a problem, perhaps based on the progress the model is making towards solving the problem, as measured by changes to the generated code and computed answer.

Most Advent of Code puzzles can be easily solved if you take the right approach. It might be possible to split the puzzle solving into a search for the right approach followed by a separate prompt series to implement the right approach.

Try solving puzzles using a cheap LLM first, and only use a more expensive LLM when the cheap LLM fails to solve the puzzle. This could reduce the overall costs of an evaluation run.

# Conclusion

Because this evaluation is relatively quick and inexpensive to run, and because it produces a wide range of scores for different languages and code execution modes, it could serve as a useful benchmark for evaluating model performance, now and for the next few years.

# References