Five labs, one suite — do model families have personalities? (benchmark)
Discussion

Bench 3 from my 18 GB M3 Pro. Bench 2 was the 4B class, and the comments were right that fixed budgets were muddying the story for thinking-capable models, so this one is the methodology-hardened rerun: novel prompts, harder tasks, 4096-token budgets across the board, and `think:false` wherever Ollama supports it. Five labs, mixed 3-9B sizes, same hardware, same deterministic graders.

Lineup (sizes on disk): `gemma4:e4b` 9.6 GB (Google, 4B active MoE), `qwen3.5:9b` 6.6 GB (Alibaba), `granite4:3b` 2.1 GB (IBM), `olmo-3:7b-think-q8_0` 7.8 GB (AllenAI), `nemotron-3-nano:4b` 2.8 GB (NVIDIA).

25 tasks: 10 finance, 7 reasoning, 8 code. 3 trials per (model × task), median aggregation. `temp=0`, `seed=42`, `max_tokens=4096`, `think:false` where supported.

Headline: Gemma wins, and the bench-2 budget problem was real

| model | overall | finance | reasoning | code |
|---|---:|---:|---:|---:|
| `gemma4:e4b` | 72% | 70% | 71% | 75% |
| `qwen3.5:9b` | 68% | 80% | 43% | 75% |
| `granite4:3b` | 44% | 40% | 29% | 63% |
| `olmo-3:7b-think-q8_0` | 40% | 30% | 43% | 50% |
| `nemotron-3-nano:4b` | 36% | 30% | 14% | 63% |

Gemma takes the overall win, but the bigger story is methodological: once you stop grading thinking-capable models inside a tiny fixed budget, the bench 2 collapse mostly disappears. Qwen 3.5 9B no longer looks unusable; it goes 80% on finance, ties Gemma on code, and finishes only four points back overall.

That does not mean "just give every model infinite tokens." It means the old setup was mixing model quality with decoding-policy failure. Bench 2 was partly measuring "can this model finish hidden reasoning inside 1024 tokens?" Bench 3 is much closer to the thing people actually care about: with enough room to answer, what does each lab's small model family actually do well?

Lab personalities are real, but less clean than bench 2 made them look

The cleanest profile in this run is actually Gemma: 70% finance, 71% reasoning, 75% code. That's the most genuinely general-purpose line in the set, and it's why it wins the benchmark instead of just winning one category.

Qwen is the opposite kind of interesting. It wins finance at 80%, ties for best code score at 75%, and then drops to 43% on reasoning. So the "Qwen is broken" story from the last post was too simple. The better read is that Qwen has a real shape, but the old setup was punishing that shape so hard you couldn't see it.

Granite still looks like a coder pretending to be a generalist: 63% on code, 29% on reasoning. That's not quite the comically sharp split it had in bench 2, but the family resemblance is still there.

The Nemotron 3 Nano reversal

Bench 2's headline was "Nemotron won and it's not close." Bench 3 is almost the inverse. Nemotron is now last overall at 36%, and especially weak on reasoning at 14%.

What it still does have is efficiency. It is the fastest decoder in the run and the cheapest model per correct answer at 175 generated tokens per correct, versus OLMo at 7252 and Gemma at 1137. So Nemotron is still interesting, just in a different way: cheap when it works, not strong in absolute terms.

This is exactly why I think the eval ecosystem has a hidden-policy problem. A benchmark can accidentally crown the model that fits the measurement setup best rather than the model that is best overall.

The OLMo problem

OLMo is the one model here that still looks genuinely pathological under the hardened setup. 36 of its 75 trial rows produced no visible answer at all, even with `think:false` and a 4096-token budget. Answered rate was only 40%.

That's different from "it answered and got it wrong." It means the benchmark is still hitting a decoding or policy mismatch on this specific model family. I would not read the 40% overall as "OLMo is a 40% model" so much as "OLMo still does not play nicely with this Ollama setup even after the bench 2 fixes."

Methodology + repo

Apple M3 Pro, 18 GB, macOS 25.5, Ollama 0.21.2. `temp=0`, `seed=42`, `max_tokens=4096` across all models. 3 trials per task, median aggregation. All graders are deterministic numeric / regex / exec, no LLM-as-judge. `think:false` was sent wherever Ollama supports it.

Repo: https://github.com/joshuahickscorp/bench3
Raw JSONL with full responses + per-token timings: https://gist.github.com/joshuahickscorp/e35b38760b84460b0f99534043310ae1

Up next

Bench 4: the 18 GB cliff. Same machine, bigger models, less "who's smartest?" and more "what actually fits before macOS starts bleeding from the eyes?"
