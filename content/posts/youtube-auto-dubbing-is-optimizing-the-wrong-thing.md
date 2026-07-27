+++
title = "YouTube Auto-Dubbing Is Not Bad at Voice Cloning. It Is Optimizing the Wrong Thing."
date = "2026-07-19"
draft = true
description = "A case study in how an AI dub can preserve the speaker, match the video, translate nearly everything—and still be painful to listen to."
path = "youtube-auto-dubbing-is-optimizing-the-wrong-thing"
aliases = ["posts/youtube-auto-dubbing-is-optimizing-the-wrong-thing"]

[taxonomies]
tags = [ "ai", "audio", "youtube" ]

[extra]
featured = false
toc = true
source_status = "Draft recovered from the local antirez dubbing experiment"
+++

# YouTube Auto-Dubbing Is Not Bad at Voice Cloning. It Is Optimizing the Wrong Thing.

*A case study in how an AI dub can preserve the speaker, match the video, translate nearly everything—and still be painful to listen to.*

YouTube's automatic English dub of an Italian programming video sounds terrible.

Not merely synthetic. Not merely a little too clean or emotionally flat. It has the more disturbing quality of speech that is almost correct at every moment but repeatedly fails as a continuous performance. Phrases begin and end in unnatural places. Words blur into each other. English syntax is forced through an Italian rhythm. Pronunciations occasionally collapse into sounds that neither humans nor speech-recognition systems can confidently identify.

My first assumption was the obvious one: YouTube must be using a mediocre generic voice and doing a poor job of cloning the original speaker.

That assumption turned out to be wrong.

After downloading the original Italian track and YouTube's highest-quality English auto-dub, transcribing both, measuring the timing, comparing speech-recognition confidence, and running speaker embeddings, I reached a more interesting conclusion:

> YouTube's system is surprisingly good at preserving speaker identity and synchronization. The dub sounds bad because the system appears to prioritize source-language timing over natural target-language speech.

This is not primarily a voice-cloning failure. It is an objective-function failure.

## The video

The test case was Salvatore Sanfilippo's video [“Rivedere il codice automatico non è la soluzione neppure per gli junior”](https://www.youtube.com/watch?v=tHpFGv8DBl4), a roughly fifteen-minute Italian monologue about programming, artificial intelligence, and whether junior developers benefit from reading automatically generated code.

Sanfilippo—better known as antirez—is the creator of Redis. His delivery in the original video is informal and highly conversational. He pauses, restarts thoughts, emphasizes unexpected words, and occasionally leaves substantial gaps between phrases. This is ordinary human speech rather than a studio narration.

YouTube provides two audio tracks for the video:

- the Italian original;
- an English (US) auto-dub.

I downloaded the medium-quality Opus representation of each track, approximately 117–118 kbit/s, rather than comparing recordings captured from the player. The English track runs for 892.261 seconds. The Italian track runs for 892.181 seconds. At the whole-file level, synchronization is effectively exact.

I also produced a separate English version using a local Qwen3-TTS voice-cloning pipeline. That version became a useful comparison because it made almost the opposite engineering tradeoffs from YouTube's system.

## The first surprise: YouTube preserved the voice rather well

Voice similarity is not something that can be established reliably by looking at a waveform or asking a general-purpose language model whether two clips “sound alike.” So I used the 2,048-dimensional speaker encoder from the same Qwen model used for the custom clone.

For corresponding 16.8-second sections, the cosine similarity between YouTube's English dub and the Italian original was:

\[
0.986
\]

That is extremely high.

For context, the custom English output had a similarity of 0.991 to the dedicated English reference clip used to condition it.

These figures are not universal voice-quality scores. Speaker embeddings are affected by language, recording conditions, background sound, and the encoder's training. They measure speaker identity, not whether the performance is pleasant, expressive, or intelligible. Still, a score of 0.986 between YouTube's English and Italian tracks is strong evidence that the system is not simply substituting an unrelated American narrator.

YouTube's newer auto-dubbing system includes what it calls [Expressive Speech](https://blog.youtube/news-and-events/youtube-auto-dubbing-expressive-speech/), intended to preserve aspects of a creator's emotion and energy. In this example, the identity-preservation part appears to be working.

And yet the result still sounds awful.

That distinction matters. “The voice is wrong” would suggest replacing the speech model. The actual problem is further upstream and downstream: translation adaptation, segmentation, duration constraints, and prosodic assembly.

## The intelligibility gap is measurable

I submitted YouTube's English track and the custom English track independently to Deepgram Nova-3. This was not used to decide whether the voices resembled antirez. It was used as a consistent proxy for acoustic intelligibility: how confidently could a strong English ASR system determine what each generated voice said?

The results were substantial:

| Measurement | YouTube English | Custom English |
|---|---:|---:|
| Words detected | 1,936 | 1,993 |
| Mean word confidence | 0.943 | **0.972** |
| Words below 0.8 confidence | 206 | **94** |
| Words below 0.6 confidence | 80 | **21** |
| Words below 0.4 confidence | 13 | **5** |

YouTube's dub produced:

- 2.2 times as many words below 0.8 confidence;
- 3.8 times as many words below 0.6 confidence;
- 2.6 times as many words below 0.4 confidence.

ASR confidence is not a substitute for a listening panel. A recognizer can be uncertain about a perfectly understandable proper noun, and it can confidently transcribe the wrong word. But across nearly two thousand words from the same source material, the difference is too large to dismiss as random variation.

The transcription also shows where intelligibility breaks down. Deepgram heard phrases such as:

> “will constantly scrutinize difficult produced by LLMs”

> “you to program x x Tidecam”

> “but then the issue opened up”

> “In The United States, in Pertho, in particular”

> “CanMed did do things with AI”

> “of a pain programming”

> “together together those skills”

These should not be presented as YouTube's intended written translation. Some are almost certainly ASR hallucinations triggered by unclear generated audio. But that does not absolve the dub. If a modern recognizer repeatedly cannot resolve the synthesized pronunciation, human listeners are likely experiencing the same ambiguity.

## YouTube solves the timing problem by making the English suffer

YouTube describes auto-dubbing as allowing viewers to follow a video with the same cadence as the original language. That sounds like an unqualified benefit. In practice, “same cadence” can become a trap.

Italian and English do not package information into speech in the same way. A natural English translation may require fewer words in one passage, more words in another, a pause in a different location, or a complete restructuring of the sentence. Even when the semantic content is equivalent, the spoken durations of corresponding fragments vary.

Professional dubbing deals with this through adaptation. A translator does not merely convert words. A dubbing writer rewrites the target sentence so that it:

- preserves the meaning;
- sounds natural in the target language;
- fits the available time;
- places emphasis appropriately;
- respects visible pauses and gestures;
- and, where necessary, approximates mouth movement.

An automatic system has to perform all of those jobs without a writer, director, actor, or editor.

The YouTube output in this case closely follows the Italian temporal structure. Its English begins in short fragments:

> “So yesterday,”

> “I wrote a follow-up on my blog that”

> “better specified.”

> “The ideas I had expressed…”

The meaning is recoverable, but the phrasing does not behave like a continuous English thought. It behaves like English content poured into Italian-shaped time boxes.

This is the central failure.

The system has several options whenever an English phrase does not fit its source window:

1. accelerate the phrase;
2. remove or compress pauses;
3. reduce articulation;
4. truncate or slur difficult sounds;
5. allow the timing to drift;
6. rewrite the translation and synthesize it again.

The sixth option is usually the best, but it is also the most expensive and operationally complicated. It requires iterative generation: translate, synthesize, measure, evaluate, rewrite, and synthesize again. It may also require a language-specific quality model capable of distinguishing concise natural English from merely shorter English.

The pulled track suggests that YouTube frequently accepts acoustic and linguistic degradation instead. It preserves the timing, but the voice pays the price.

## A likely pipeline—and where errors compound

YouTube does not publicly document every internal stage or model used for this particular track. The following architecture is therefore a high-level inference, not a claim about proprietary implementation details:

```text
Italian audio
    ↓
Speech recognition
    ↓
English translation
    ↓
Source-derived timing constraints
    ↓
Expressive speech synthesis / voice transfer
    ↓
Mixing with the original audio environment
```

Every stage can be individually good while the complete dub is bad.

An ASR error becomes a translation error. A literal but technically correct translation becomes an awkward dubbing line. An awkward line becomes harder to fit into its allotted duration. Duration compression damages articulation. Damaged articulation makes the final speech harder to understand.

This is a classic cascaded-system problem: errors do not merely add together; they interact.

The most important contract in such a system is the timing representation passed between stages. If the translation model receives hundreds of narrow, fixed windows, it is encouraged to optimize each local fragment rather than the coherence of the entire thought. If the speech model is then required to honor those windows, it cannot repair the linguistic decisions without violating timing.

The end product can be locally synchronized and globally unnatural.

## What the custom pipeline did differently

The custom version started from the same Italian subtitle timeline but deliberately relaxed the granularity.

First, the Italian transcript was reconstructed as coherent English prose. The goal at this stage was not synchronization. It was to produce something an English-speaking person might naturally say.

Second, the voice was conditioned using a clean 16.8-second clip of antirez actually speaking English, together with the clip's transcript. This is a major advantage over cloning an English voice exclusively from Italian speech. The reference provides direct evidence of his English vowels, consonants, accent, rhythm, and phrasing.

Third, the completed narration was divided into 26 semantic blocks and mapped back onto the original 14-minute, 52-second timeline.

Only four blocks exceeded their available source windows. Their required corrections were small:

- 3.2 percent;
- 2.2 percent;
- 3.7 percent;
- 4.0 percent.

All other blocks retained their generated speed and pitch. Silence was allowed where the English thought finished before the next source topic began.

This does not produce perfect phonetic lip synchronization. It is phrase-level or semantic synchronization. English topics begin and end near their Italian counterparts, but every English phoneme does not match every Italian mouth shape.

That compromise is precisely why the narration remains intelligible.

YouTube is attempting a harder and, in one sense, more impressive task: tighter local correspondence with no manual intervention. But tight synchronization is not automatically better synchronization if achieving it destroys the target-language performance.

## The custom system also had advantages YouTube may be unable—or unwilling—to use

It would be unfair to compare the outputs without acknowledging the asymmetry.

The custom workflow included multiple editorial and technical passes:

1. obtaining and correcting the source transcript;
2. reconstructing a coherent English narrative;
3. locating a separate English recording of the speaker;
4. extracting a clean reference segment;
5. enabling audio-and-text in-context voice cloning;
6. generating and reviewing the narration;
7. removing a mistakenly spoken title;
8. mapping semantic blocks to the source timeline;
9. measuring every block;
10. applying limited local correction;
11. validating the final audio and video containers.

YouTube's feature has to run automatically across millions of channels and many target languages. Its economic and product requirements are radically different:

- no dubbing editor;
- no per-video direction;
- no manual search for clean reference speech;
- bounded compute cost;
- acceptable processing latency;
- safety and impersonation controls;
- broad language coverage;
- and a quality floor that works across wildly different source material.

There are also legitimate reasons not to build persistent cross-video voice profiles for every creator. Doing so would raise questions about consent, voice rights, privacy, deletion, impersonation, and whether an upload in one language grants permission to synthesize that person's speech in every other language.

YouTube allows creators to review, unpublish, or replace automatic dubs, and its [documentation](https://support.google.com/youtube/answer/15569972?hl=en) acknowledges that fast source speech can make a listenable dub impossible. The product is explicitly a scalable automatic baseline, not a substitute for a directed localization process.

## Loudness is not the main problem, but it does not help

Both downloaded YouTube tracks reached 0.0 dBFS when decoded. The custom track peaked at -4.1 dBFS.

Peak level alone does not prove audible clipping, especially with a perceptual codec, and the original Italian track was mastered similarly. Still, a dense synthetic voice with no headroom can feel more fatiguing. Aggressive mastering makes timing discontinuities and hard consonants more conspicuous rather than hiding them.

The YouTube dub also retains continuous background or environmental sound, whereas the custom replacement is clean narration with deliberate silence. Preserving ambience is normally desirable, but it complicates analysis: amplitude-based silence detection cannot reliably reveal speech boundaries because the underlying audio bed never becomes truly silent.

## The broader lesson: define quality at the system level

AI media pipelines are often evaluated component by component:

- Was the transcript accurate?
- Was the translation semantically correct?
- Did the voice resemble the speaker?
- Did the audio match the video's duration?

This example can score reasonably well on all four questions and still fail as a listening experience.

The missing metric is whether a human would voluntarily continue listening.

Automatic dubbing needs a system-level objective that balances at least four competing properties:

\[
Q = f(M, N, I, S)
\]

where:

- \(M\) is semantic meaning;
- \(N\) is target-language naturalness;
- \(I\) is intelligibility;
- \(S\) is audiovisual synchronization.

You cannot maximize all four independently. Overweighting synchronization can reduce naturalness and intelligibility. Overweighting naturalness can allow unacceptable visual drift. Optimizing semantic similarity alone can produce translations that cannot be spoken within the scene.

The correct balance also depends on the material. A talking head with clearly visible lips needs tighter timing than a screen recording. An off-screen lecture can tolerate substantial relaxation. A technical monologue needs terminology accuracy more than phonetic lip matching. Comedy may prioritize delivery above literal translation.

A mature dubbing system should therefore vary its constraints by scene and should be willing to regenerate lines rather than forcing every first-pass translation into a fixed box.

## YouTube does not lack the technology

Google has world-class speech recognition, translation, speech synthesis, audiovisual understanding, and generative models. The existence of a poor auto-dub does not demonstrate that Google cannot build a better one.

It demonstrates that a zero-cost, zero-touch product operating at YouTube scale has different priorities from a carefully produced dub.

The current optimization appears to favor:

\[
\text{scale} + \text{automation} + \text{timing fidelity} + \text{coverage}
\]

over:

\[
\text{natural English} + \text{editorial adaptation} + \text{intelligibility}
\]

That may be rational for the product. It is not pleasant for the listener.

The most revealing result of this experiment is that YouTube's voice is not obviously the weakest component. The system preserves antirez's identity surprisingly well. It keeps the audio attached to the original timeline. It communicates most of the content.

Then it ruins the performance by insisting that English behave too much like timed Italian.

YouTube solves synchronization by making the voice suffer. A better pipeline lets the translation and the timeline negotiate.

---

## Methodology and limitations

- The YouTube Italian and English tracks were downloaded as their medium-quality Opus representations, approximately 117–118 kbit/s.
- Durations and levels were measured with FFmpeg/FFprobe.
- English intelligibility was compared using Deepgram Nova-3 word confidences.
- Speaker similarity was estimated with the Qwen3-TTS speaker encoder used by the custom cloning pipeline.
- The custom track used a 16.8-second English reference recording, natural English adaptation, 26 semantic timing blocks, and four local tempo corrections of approximately 2–4 percent.
- ASR confidence is an intelligibility proxy, not a human mean-opinion score.
- Speaker-embedding cosine similarity measures identity as represented by one model; it does not measure acting quality, emotion, accent accuracy, or listening comfort.
- The observations come from one Italian-to-English video and should not be treated as a benchmark of every YouTube auto-dub.
- Descriptions of YouTube's undocumented internal stages are informed inferences from the output and public product documentation, not claims of access to its proprietary implementation.

