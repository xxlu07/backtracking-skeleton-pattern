# backtracking template: the choose/explore/unchoose skeleton, the 2 functions you fill in, and where pruning actually goes

Every backtracking problem looks different right up until you write the code. Then you notice you've written the same twelve lines for the fourth time, changing only two helper functions. That's the whole point of a template: the skeleton is fixed, the cells are what you swap.

Below is the version that shows up most often in interview prep material, the four questions you have to answer before it works, and the places where the template quietly breaks.

## The template, in about twelve lines

The skeleton is a DFS over a tree you build as you go:

python
def dfs(start_index, path):
    if is_leaf(start_index):
        report(path)
        return
    for edge in get_edges(start_index):
        path.append(edge)      # choose
        dfs(next_index, path)  # explore
        path.pop()             # unchoose


Walk through the loop body and you'll see the three named moves: **choose, explore, unchoose**. Upstream of the loop is the leaf check that reports an answer. That's the entire algorithm.

Two details matter more than they look.

`start_index` is not always `start_index + 1`. The recursion step should advance by however much of the input the edge consumed. In the letter-combination problem it's one character, so `+1`. In Word Break, the edge is a whole dictionary word, so you jump by `len(word)`. The online version of this chapter on AlgoMonster's site keeps `start_index + 1` in the main template and treats the variable-length jump as the second variant, which trips people up if they copy the first form everywhere.

The unchoose is where intuition usually fails. Forgetting `path.pop()` doesn't crash. It produces wrong answers that look almost right, which is worse.

## The four questions to answer before writing anything

The template is easy. Filling in `is_leaf` and `get_edges` correctly is the actual work. Four questions get you there:

1. **What counts as a complete solution?** That's your leaf condition. For subsets, every node of the tree qualifies. For combinations of size k, only nodes at depth k. For "does a path exist", you stop at the first success instead of enumerating everything.
2. **What are the edges out of a given state?** For permutations it's every unused index. For combination sum it's every candidate at or after the current index. For Word Search it's the four neighbouring cells that match the next letter.
3. **What state has to travel with the recursion?** A `path` list is the minimum. Permutations need a `used` array. Word Search needs an in-place visited marker on the grid. Combination problems need the running sum.
4. **Where does partial input get consumed?** This decides whether you pass `start_index`, a remaining target, or nothing at all, and it decides the recursion step size.

Write those four answers down as comments first. If you can't answer question 2 with a short sentence, you haven't drawn the tree yet, and no amount of code will save you.

## Draw the tree. Then draw it again.

The advice that shows up in nearly every serious treatment of this pattern is blunt: draw the state-space tree. It's the difference between backtracking feeling like magic and feeling like a loop.

Take the classic warm-up: given `n`, return all `n`-letter strings over `a` and `b`, lexicographic order. For `n = 2` the tree has a root, two children at depth 1 (`a`, `b`), and four leaves (`aa`, `ab`, `ba`, `bb`). The leaves are the answers. The path from root to leaf is the string you're accumulating. `path.append(edge)` extends the current path by one level; `path.pop()` hands the state back to the parent so the sibling branch starts clean.

Once you've traced that, the same picture covers N-Queens (edges are legal columns in the next row), Sudoku (edges are legal digits in the next empty cell), and Palindromic Partitioning (edges are palindrome prefixes of the remaining string). Different cells, identical skeleton.

Drawing the tree also tells you the complexity before you write a line. If every node has at most `W` children and the tree is `D` levels deep, you are looking at roughly `W^D` nodes. That's why input sizes for these problems are always small, and why a "can I do better than brute force" instinct is usually the wrong reflex here. The answer is to prune, not to replace the approach.

## Pruning: the one place to get aggressive

Pruning means cutting a branch before you spend time exploring it. It never changes the correct answer, only the work.

Three shapes cover most of what you'll be asked to do:

**Bound checks.** In combination sum, if the running sum already exceeds the target, return immediately. There's no completion below that node worth looking at. In combinations of size `k` from `1..n`, you can also stop the loop early: if you still need `k - len(path)` more elements, the last start index that can possibly work is `n - (k - len(path)) + 1`. Iterating past that point generates subtrees guaranteed to produce no leaves.

**Validity checks.** N-Queens and Sudoku test each candidate against cached constraint sets (columns, diagonals, box indices). One failing test kills the subtree.

**Duplicate skipping.** This is the one people get wrong most often. Sort the input first, then inside the loop skip an element when `i > start` and `nums[i] == nums[i-1]`. Sorting puts equal values adjacent, and the index comparison means you only skip duplicates at the same level of the tree, not across levels. Permutations handle it slightly differently because there's no `start` index; you check whether the previous identical element is currently in the `used` set.

Pruning is also the honest answer to the "when do I not use this template" question. Backtracking with a good bound beats a clever-sounding alternative in most interview settings, and the interviewer can see why it works.

## Backtracking 1 vs Backtracking 2

The skeleton above (call it Backtracking 1) reports at leaves and returns nothing. It's right for anything that enumerates solutions into a result list: subsets, permutations, combinations, N-Queens, Sudoku.

The second variant is for problems that **aggregate an answer back up the recursion instead of collecting solutions**. Word Break is the canonical example: can the target string be assembled from a dictionary? The recursion doesn't gather strings, it returns a boolean, and the parent combines child results with `or`. You still get choose/explore/unchoose; you just replace "report and return" with "aggregate and return".

python
def dfs(start_index):
    if start_index == len(target):
        return True                # leaf: consumed everything
    ans = False
    for word in words:
        if target.startswith(word, start_index):
            ans = ans or dfs(start_index + len(word))   # edge consumes len(word)
    return ans


Notice `start_index + len(word)`. If you templated that as `+1`, the recursion breaks immediately. This is the single most common mechanical error when people learn the template from one example and apply it to another.

There's a third pattern worth knowing, usually presented as a separate chapter: **backtracking with a grid**. Word Search is the textbook case. The distinction is that you mark and unmark cells in place instead of pushing and popping a path, because the question is about paths through a grid, not about sequences. Generic graph reachability marks cells permanently; word search has to restore them, since a different ordering may legitimately revisit a cell the current path already left.

## Complexity, without the hand-waving

Time complexity is proportional to the number of nodes in the state-space tree, so state it in those terms first, then reduce to a closed form if you can:

| Approach | Time | Space (recursion) |
| --- | --- | --- |
| Subsets | O(n · 2^n) | O(n) |
| Permutations | O(n! · n) | O(n) |
| Combinations of size k | O(C(n,k) · k) | O(k) |
| Word Search on `m × n` grid, word length L | O(m · n · 4^L) | O(L) |
| N-Queens / Sudoku | Exponential worst case, tiny in practice | depth of the tree |

The multiplier in the time column is copy cost. Every time you append `path[:]` to the result list you pay for a copy of the current path, and interviewees forget to say it out loud. Space is the height of the tree, because that's where the call stack peaks.

## The bugs that actually cost offers

- **Missing `pop()`.** Wrong answers, not errors. Trace a two-element example by hand and it shows up immediately.
- **Missing `copy()`.** Appending `path` instead of `path[:]` gives you a result list full of empty lists, because every stored reference points at the same mutating object.
- **Wrong base case.** Reporting at the wrong depth, or returning after the first child instead of after the loop. In the "find one solution" variant, returning `true` inside the loop is correct and returning `false` after one failed child is not.
- **Pruning with the wrong condition.** A bound check that's one index too tight silently drops valid answers.
- **Copying the template without re-deriving the edge set.** The `+1` problem above.

None of these are conceptual failures. They're what happens when you learn the shape without drawing the tree.

## Which problems map to which cell

This is roughly how the pattern organises itself once you've seen enough of it. The middle column is the only thing that changes.

| Problem | Template | Edge set | What travels with the recursion |
| --- | --- | --- | --- |
| Letter combinations of a phone number | Backtracking 1 | Letters mapped to the current digit | `path` |
| Subsets (with/without duplicates) | Backtracking 1 | Every index from `start` | `start_index`, `path` |
| Permutations | Backtracking 1 | Every unused index | `used` array, `path` |
| Combinations / Combination Sum I–III | Backtracking 1 | Candidates from `start` (or from `i` if reuse allowed) | running sum, `path` |
| Palindrome Partitioning | Backtracking 1 | Palindrome prefixes of the remainder | `start_index`, `path` |
| N-Queens, Sudoku Solver | Backtracking 1 + constraint sets | Legal columns / legal digits at the next position | column, diagonal, box sets |
| Word Search | Grid backtracking | 4-adjacent cells matching the next letter | in-place cell marker, index into word |
| Word Break | Backtracking 2 | Dictionary words matching at `start_index` | `start_index` only |

One pattern, eight rows, and the differences concentrated in two columns. That's the argument for learning it this way instead of grinding problems one at a time and hoping the connection appears.

## Where to drill the template

Free resources for this exist and they're decent. GitHub repos, LeetCode discussion threads and university lecture slides all publish some version of the skeleton. What they don't give you is the trace you can step through, or the curated problem set in the right order.

AlgoMonster's backtracking chapter is one of the clearer written treatments of the pattern online, and it's free to read. It walks through the state-space tree for a concrete input, gives the template, then drills N-Queens, Palindrome Partitioning, Word Search, Sudoku and Combination Sum using that same skeleton — which is the point. 👉 [See the full backtracking template walkthrough on AlgoMonster](https://algo.monster?fpr=li69)

The platform is built by ex-Google engineers around the premise that interviews recycle a small set of patterns and that learning the templates explicitly beats grinding thousands of problems. It's entirely text and diagrams, so if you need video walkthroughs this isn't it. It does carry 3,000+ editorials, an AI assistant you can query when you're stuck on a specific recursion, and a Speedrun mode for timed review before an interview. The material is structured as a 4–8 week curriculum rather than an open problem bank.

If you're specifically stuck on backtracking, the useful move is to read the template chapter, then solve the paired problems without looking at the solutions, then compare. Reading the template doesn't teach you to derive the edge set. Solving five problems against that template does.

## Plans and pricing

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of the curriculum, enough to judge whether the text-and-diagram format works for you | $0 | — | [Start on AlgoMonster's free tier](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems unlocked; unlimited AI assistant; unlimited Speedrun; company-sourced questions updated monthly; 3,000+ editorials; member discount on live bootcamps; 3-day free trial then 12 months of access | $99/year (shown as $8.25/mo, about $0.28/day) | Billed annually, cancel or upgrade anytime | [Check the current AlgoMonster yearly plan](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in the annual plan plus lifetime updates, and an internal referral to a top tech company for candidates who qualify | $189 | One-time payment | [Get AlgoMonster lifetime access](https://algo.monster?fpr=li69) |

Two things worth weighing before you pick. The lifetime plan costs about $90 more than a single year, so it pays for itself the second time you prep for interviews — which, for most engineers, happens every two or three years. And if you're prepping for a senior role, budget separately for system design: the coverage here is introductory, not enough for an L5+ loop on its own. Several reviews also flag the lack of a live interview simulation as the main gap, which is fair. Practising recursion is not the same as performing it with someone watching. 👉 [Compare the plans and current pricing](https://algo.monster?fpr=li69)

## FAQ

**Do I need to memorise the template?** Memorise the shape, not the code. The three moves and the leaf check are the shape; `start_index + len(edge)` versus `+1` is derived, not memorised.

**Is backtracking the same as DFS?** It's a DFS over a tree you construct while running. In a plain tree or graph problem the edges are given to you. Here you generate them, which is why the template needs a `get_edges` step.

**When should I prune instead of looking for a better algorithm?** When the problem asks for all solutions, the first answer is almost always a pruned search. Interviewers ask for pruning because they want to see whether you understand where valid subtrees stop being possible.

**Do I need `used[]` for subsets?** No. Passing `start_index` forward is enough to prevent reordering, and reordering is what `used[]` exists to allow. If your subsets solution has a `used` array, you've probably built a permutations solution by accident.

The template isn't a shortcut past understanding the problem. It's what's left once you've understood it, and it's the reason a well-prepared candidate can go from reading a problem to writing a correct recursive function in a couple of minutes instead of fifteen.
