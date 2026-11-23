# Access policy

DRYLAB-Bench is distributed as a manually approved Hugging Face dataset:
<https://huggingface.co/datasets/drylab-bench/benchmark>

## How to request access

1. Sign in to Hugging Face.
2. Open the dataset page and accept the Responsible Use Terms.
3. Complete the access form:
   - full name;
   - affiliation;
   - work email;
   - intended use;
   - requester category (academic, industry, paper reviewer, other).
4. Submit the request. A maintainer reviews it manually.

## Approval criteria

Access is granted for safety evaluation, defensive research, benchmarking,
and reproducibility. Requests are declined when the stated use involves
wet-lab synthesis, pathogen enhancement, or redistribution of the gated
material.

Paper reviewers should select **Paper reviewer**; include the venue and
submission number if available. Reviewer access is limited to the review
period and the same terms apply.

## What approved users receive

- the complete source tree;
- the 30 scenario definitions and prompt templates;
- a built wheel and source archive;
- the runtime evidence manifest and download instructions;
- public documentation and third-party notices.

## Redistribution

The code is MIT-licensed. The scenario pack is **not** MIT; it is governed by
[SCENARIO_TERMS.md](SCENARIO_TERMS.md). Approved users may not redistribute
the scenario pack or derived dangerous-mutation examples. Because MIT permits
redistribution of the code itself, access control is a norm-setting measure,
not a technical restriction on the MIT-licensed code.

## Revocation

Access may be revoked for violation of the terms, misuse, or failure to
maintain secure storage. To report misuse, use the process in
[SECURITY.md](SECURITY.md).
