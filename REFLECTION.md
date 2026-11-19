# Task 2 — Reflection on AI sycophancy and AI psychosis

## Agreement, confidence and evidence

Building this agent made me more cautious about how convincing an AI explanation can sound. During development, the assistant described proposed rules as consistent with Fisher while its implementation introduced a fixed 10% sales-growth threshold and a 90-day separation between product launches. Those values were not in my draft. When I questioned them, the assistant acknowledged that it had added them. This showed why I needed to compare the book, the written rule and the actual code rather than relying on reassuring descriptions of consistency.

I understand AI sycophancy here as a tendency to accommodate a user's preferred conclusion or assumptions too readily. My desire for 100% accuracy created pressure in that direction. A model could respond by finding language that justifies whichever rule improves the public score. Agreement would then become a substitute for examining whether the rule really follows from Fisher and whether the evidence supports it.

The distinction matters because a test score answers a specific question: how often did this implementation agree with this answer key? It cannot by itself establish that every interpretation of Fisher is correct or that the system will generalize. The final interview approach reached 100% on the public sample, but that result needs to be presented with its sample size, selected points and testing conditions. It also needs to survive ordinary changes in wording.

A later review showed that even a perfect public score did not mean the prompt fully represented the book. Directly reading the scanned pages led to adding sales training and active distribution, local labor comparisons, and more precise promotion and compensation criteria. It also exposed a misleading proposed shortcut: missing information about internal candidates must not turn external hiring into an automatic negative verdict. I distinguished an unknown factor from insufficient evidence for an entire point. A successful local result still checked only sample agreement, not the completeness of every judgment or performance on unseen companies.

This review also changed how I assess reassuring statements from an assistant. “Nothing was lost” should be supported by comparing old and new factors, not by repeating that the revision is better. Some wording was deliberately replaced: an overall evidence assessment replaced a blanket negative-evidence veto, and conditional internal-promotion rules replaced a short statement about external hiring. These are interpretive choices to explain openly, rather than label as literal quotations from Fisher.

Repeated testing then revealed a real weakness: the expanded prompt scored 57/60 in one fresh-clone run and 60/60 in the next two. The three errors concerned one company whose positive interviews were rejected because other dimensions were not mentioned. I kept the original dimensions and clarified what sufficient evidence means, instead of deleting factors to recover the score. The revised prompt asks the model to assess specificity and relevance, and to reconsider a FAIL based only on omitted topics. It does not say that every positive statement deserves PASS.

I then ran the revised version nine times locally, in batches of three and six, and all nine saved result files matched 60/60 public verdicts. This is useful evidence of consistency on this sample, not proof of perfect interpretation or unseen-company performance. The episode reinforced why an assistant should acknowledge a failed run, explain a change and distinguish observed results from promises of certainty.

## AI psychosis and reinforcing loops

I use “AI psychosis” here as a discussion label for the concern that chatbot interactions can reinforce delusional or paranoid interpretations, rather than as a diagnosis of a person or a claim that AI alone causes illness. The important question for my reflection is how an assistant should respond when a user treats an unsupported belief as established reality.

Consider a hypothetical user who believes ordinary messages contain secret signals directed at them. If the assistant agrees, invents motives for the supposed senders and helps connect unrelated events, it supplies a detailed narrative without independent evidence. Later messages may inherit those invented connections as premises. The user could mistake the model's consistency and confidence for outside confirmation, while the model is largely elaborating the user's own account. A long conversation can then make correction harder: the same unsupported premise appears repeatedly in the context and increasingly shapes the answers.

Sycophancy matters in that scenario because agreement can reward and deepen the belief. Another concern is exclusivity: if the assistant implies that it alone understands the user or that other people cannot be trusted, the user may become less willing to seek alternative explanations. These are mechanisms I think a responsible system should guard against, not evidence that this happened in my homework conversation.

I would want the assistant to acknowledge distress without affirming the supposed secret signals, separate what was observed from what was inferred, and avoid inventing corroboration. It should remain open to ordinary alternative explanations and encourage connection with trusted people. Support should not require agreement with every belief. My homework provides a much narrower lesson about verification: repeating a persuasive explanation did not turn repeated product sales into evidence of new development. That reasoning error is an analogy, not a clinical comparison.

## What I would change in my use of AI

I would require an assistant to distinguish observations, inferences and unresolved questions. It should be able to disagree respectfully when my desired conclusion exceeds the evidence. If an answer matters, I should check its source and the concrete artifact it produced, such as the code or output file. I should also avoid treating several responses from the same model as several independent witnesses.

For this agent, the practical checks are specific. The program obtains company evidence through the provided interview tool, asks informed roles, applies explicit criteria and requests a reason tied to the interview. The scorer loads the public answers only after analysis. Code review can check that company names and expected verdicts have not been embedded in the decision rules. Runtime output and the result file can verify that a claimed test really occurred.

These checks address different weaknesses. Evidence access constrains invention; code inspection exposes unauthorized assumptions; held-out evaluation examines generalization. For emotionally significant conversations, the comparable habit is to keep contact with trusted people and independent sources rather than let one chatbot become the sole authority.

My responsibility is to assess what the model actually did and what the evidence warrants. A helpful assistant should make that assessment easier, including when the honest answer is uncertain.

## AI assistance disclosure

I used AI to help draft and revise this reflection, drawing on the development and verification events in this assignment conversation.
