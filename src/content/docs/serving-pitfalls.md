---
title: "Serving pitfalls (field notes)"
---

Practical lessons that don't fit a single term definition — each one learned the hard way while self-hosting. The vocabulary used here is defined in the rest of the glossary.

## Benchmark quants on your hardware — smaller is not always faster

Quant speed depends on how expensive the *dequantization kernels* are on your specific GPU architecture, not just on file size. On some architectures a larger Q8_0 decodes faster than a much smaller IQ quant, because Q8_0's dequant is nearly free while IQ's is heavy there. Treat quant choice as a per-hardware benchmark question, not a size ladder.

## mmap thrash looks like a hang

Loading a model file **larger than system RAM** with default memory-mapped loading can look like the server froze: the page cache thrashes and re-reads the file forever. Loading without mmap (llama.cpp: `--no-mmap`) reads it straight into memory and finishes in a predictable time. If a big-model load "hangs", compare file size against RAM before blaming the GPU.

## Benchmark drift: only trust interleaved A/B

Restart-to-restart throughput on the same setup can vary by ±20% (memory layout, clocks, cache state). Comparing a "before" number from one server start against an "after" from another mostly measures noise. Interleave both variants on a *single* running instance and compare those.

## Draft heads can be fine-tuned against the quantized model

The **MTP / nextn** draft head of a model can be fine-tuned without the original unquantized checkpoint: extract the head tensors from the served GGUF, capture the head's runtime inputs (token embeddings + final hidden states) by running a corpus through the full model, train the head, and re-quantize it back into the file in place. Two generalizable tricks: generate the training corpus *with the served model itself* so training matches the serving distribution, and validate the reimplemented head reproduces live acceptance before training. Result in one case: draft acceptance at 32k context up from 57% to 65–70%, long-context throughput up ~10%.

## Callback-based capture fires per micro-batch

Engine eval callbacks typically fire once per **micro-batch** (llama.cpp: per ubatch), not once per request or chunk. A capture/instrumentation tool that reads buffers per callback silently keeps only the *last* micro-batch's data for anything longer than the micro-batch size. Size the batch knobs (`-b`/`-ub`) to your chunk length and verify captured row counts against token counts.
